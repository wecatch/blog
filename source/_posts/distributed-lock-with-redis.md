title: 如何用 redis 造一把分布式锁
date: 2016-08-20 12:37:42
tags:
- redis
- lock
- distributed
categories:
- 技术
---

[三月沙](http://sanyuesha.com/about/) [原文链接](http://sanyuesha.com/2016/08/20/distributed-lock-with-redis/)


## 基本概念

**锁**

> [wiki](https://en.wikipedia.org/wiki/Lock_(computer_science):In computer science, a lock or mutex (from mutual exclusion) is a synchronization mechanism for enforcing limits on access to a resource in an environment where there are many threads of execution. A lock is designed to enforce a mutual exclusion concurrency control policy.
> 在计算机科学中，锁或互斥量是一种同步机制，用于在多线程执行环境中，强行限制对资源访问。锁常被用于同步并发控制。

简单来讲，锁是用来控制多线程执行对资源的并发访问的。比如当一个资源只允许在任意时刻只有一个执行线程对其进行写操作，那当其他线程要访问资源时，就必须要检查该该资源上是否存在`写操作锁`，如果存在，必须要等待锁的释放并获得锁之后才能对资源进行访问。

**悲观锁**

悲观锁假设在一个完整事务发生的过程中，总是会有其他线程会更改所操作的资源，因此线程总是对资源加锁之后才会对其做更改。

**乐观锁**

乐观锁假设在一个完整事务发生的过程中，不一定会有其他线程会更改资源，一旦发现资源被更改，则停止当前事务回滚所有操作。

**分布式锁**

常见的锁有多线程锁、数据库事务中的行级锁、表级锁等，这些锁的特点都是发生在单一系统环境中，如果需要在不同的进程、不同机器和系统等分布式环境中控制对资源的访问，这时候我们需要一把分布式锁。

不言而喻，分布式锁就是为了解决分布式环境中对资源的访问限制而诞生的。

## 如何设计一把分布式锁

<!-- more -->

我们用 redis 来实现这把分布式的锁，redis 速度快、支持事务、可持久化的特点非常适合创建分布式锁。


### 分布式环境中如何消除网络延迟对锁获取的影响

锁，简单来说就是存于 redis 中一个唯一的 key。一般而言，redis 用 `set` 命令来完成一个 key 的设置(加锁)，使用 `get` 命令获取 key 的信息(检查锁)。由于网络延迟的存在，简单的使用 `set` 和 `get` 命令可能会带来如下问题：

线程 A 检查锁是否存在(get)-->否-->加锁(set)，在 A 发起加锁命令但是还没有加锁成功的时候，可能线程 B 已经完成了 `set` 操作，锁被 B 获得，但是 A 也发起了加锁请求，由于 `set` 命令并不检查 key 的存在，B 的锁很可能会被 A 的 `set` 操作破坏。

幸运的是，redis 提供了另一个命令 `setx` : 当指定的 key 不存在时，设置 key 的值为指定 value，如果存在，不做任何操作，成功则返回 1，失败则返回 0。也就是只要命令返回成功，线程就能正确获得锁，不需要再做类似 `get` 检查操作。

使用 `setx` 可以消除网络延迟对锁设置的影响。

### 加锁的客户端发生 crash 导致锁不能被正确释放应该怎么处理？

加锁成功并操作完成之后，就需要加锁线程对锁进行释放，以让出资源的控制权。释放锁，就是删除 redis 中这个唯一的 key，但是一定要保证删除的这个 key 是该线程创建的，因而锁创建时必须携带执行线程的唯一特征以标示创建者的身份，在这里就是这个唯一 key 对应的 value。

如果加锁的线程出现异常 crash 了而不能及时删除锁，则会导致锁一直无法被正确释放，资源处于一直被占有，别的线程处于一直等待的状态。为了避免这样的情况，锁一定要在异常发生之后可以自己释放，以让出资源的控制权，可以使用 redis 的超时机制来达到这个目的。超时时间视不同的业务场景而定，一般是最大允许等待时间。需要注意的是，只有在加锁成功之后才可以对 key 设置 TTL，否则很容易导致 key 被多个线程不断设置 TTL 而无法过期。

```python
  if CONN.setnx(lockname, identifier):
    CONN.expire(lockname, timeout)
```


### 加锁之后如何有效监测锁是否被篡改？

redis 提供了 pipeline 和事务操作来保证多个命令可以在一个事务内全部完成从而减少多次网络请求带来的开销，watch  命令又可以在事务开始执行之前对所要操作的 key 执行监测，从而保证了事务的完整性和一致性。因此，为了防止锁篡改，可以在加锁完成之后对锁进行 watch 操作，一旦锁发生变化，则终止事务，回滚操作。

```python
pipe = CONN.pipeline(True)
pipe.watch(lock)
```

### 提供锁的宿主机( redis 服务器) crash 导致锁不能被正确建立和释放该如何处理？


不论是通信故障或是服务器故障而导致的锁服务器无法响应，此时都会导致客户端加锁和释放锁的请求无法完成，因此一定要有相应的应急处理，以确保程序流程的完整体验，加强客户端的健壮性。比如相应的超时提示，异常告警等。


## 哪些边界需要注意

1.只有锁正确释放才算是整个事务的完整结束，如果锁释放失败，比如被篡改、锁服务器异常等，不同的业务可以根据自己的需求进行变动和调整。

2.设置 TTL 一定要在加锁成功之后，否则所有获取锁的客户端都会尝试 TTL 导致锁无法过期。

3.锁的过期时间也就是获取锁的客户端的最大等待时间，这个时间以执行的事务能够容忍的最长时间为限


## 一个简单的 python 实现

{% gist 5e474f42c8167ab32a6627ddd837fc83 redis_lock.py %}


在上面这个实现中，如果锁释放(release_lock)失败，客户端可以尝试对当前的操作进行回滚或自定义处理方式。

如果业务的事务是在 redis 中执行的，完全可以用 redis 的 pipeline 和事务(开始前对锁键进行 watch )来完成所有操作，执行完成之后正确删除锁即可，这样 release_lock 的操作就可以不需要单独进行了。

```python
try:
    pipe.watch(lock)
    pipe.multi()
    #TODO transaction with redis command
    pipe.delete(lock)
    pipe.unwatch(lock)
except:
    #TODO transaction failes
    pass
else:
    #TODO transaction success
    pass
```
