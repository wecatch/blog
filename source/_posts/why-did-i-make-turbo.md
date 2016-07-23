title: 我为什么创造了 turbo 这个后端的轮子
date: 2016-07-23 19:05:01
tags:
- python  
categories:
- 技术
---

[三月沙](http://sanyuesha.com/about/) [原文链接](http://sanyuesha.com/2016/07/23/why-did-i-make-turbo/)


> turbo 与其说是一个 framework ，不如说是一个后端的解决方案。

## turbo 诞生的背景

tornado 是异步非阻塞的 web 服务器，同时也是一个 web framework，功能简单但完全够用。

原东家的技术栈是：tornado、mongodb、redis、openresty，最大的服务每天服务的独立用户有上百万。早期大部分项目完全使用 tornado 最原始的方式构建，一个 main 文件包含几百个路由，所有的 handler 分布在十几个大大小小的文件夹下面，项目基本的文件结构包含：

- 一个抽象的 dal 层(数据抽象)
- 数据库连接(db)
- 配置(conf)
- 工具(utils)
- 缓存(cache)
- 第三方库(lib)

随着项目的代码越来越多，多个特性开发常常同时进行，开发人员经常要非常小心地应对代码的更改，即使如此，冲突依然时常发生，代码的可维护性和健壮性日益变差。

创业公司由于业务面临飞速增长，多条业务线齐头并进，开发团队往往要维护很多个项目，有的项目生命周期很短，一次活动就结束了，有的项目要对外合作，随合作的结束而结束，有的项目服务新的产品，需要快速构建。不同的项目由不同的开发人员创建，创建项目的方式基本是 copy 旧项目，比较原始，容易出错。项目结构在不同的项目之间的差异随着业务的不同，差别渐渐凸显，重复的功能性代码和工具性代码也逐渐变多。


<!-- more -->


在这样的背景之下，影响后端开发效率和开发质量问题主要可以抽象为：

- 大项目路由多，不易管理，不易变更
- 项目目录分层不够，协同开发困难
- 构建没有标准，混乱不堪，可维护性差
- 重复性代码可共享性差，不易插拔
- tornado 天生原始，没有 Session、数据库连接、配套的 mongodb ORM、缓存管理、RESTFul等丰富特性辅以提高开发效率。

turbo 就是为解决这些问题而创建的，因而 turbo 更像是一个解决方案。


## turbo 具有什么特性

- 项目构建工具，一键生成项目目录和 app 结构
- Session，提供灵活的接口，可以自定义存储方式
- 简单的 mongodb ORM，和 mongoose 相比，非常简单甚至粗糙，但足够用了
- 丰富且高度统一的目录划分，项目再大也不怕
- 代码可插拔，易移植，易维护
- 可以方便实现 RESTFul 风格的 api
- 灵活的路由组织结构，非常方便重构和查找

#### 一键构建

目录结构一键生成，不再是从旧项目拷贝，结构一致，高度统一，可维护性变强。

#### Session

自带 Session，存储方式可以自由变更，灵活性强。

#### 简单的 mongodb ORM 

ORM 提供最基本的 collection 结构到代码的映射，表结构清晰明了，增删改均可灵活定义数据一致性检查。

#### 丰富的目录结构划分

严格定义了各个目录的结构的作用以及层次，提供了一般大型项目所需的各种目录划分

```bash
├── README.md
├── app-server # 业务 server
├── config # 全局配置
├── db # db 连接
├── helpers # 全局业务逻辑
├── lib # 第三方依赖库
├── models # 表结构
├── script # 一次性脚本
├── service # 业务性工具
├── store # 全局状态管理
├── task # 异步任务
├── utils # 非业务性工具

```

#### 代码可插拔, 易移植

```bash
apps
├── __init__.py
├── app
│   ├── __init__.py
│   ├── app.py
│   ├── base.py
│   ├── setting.py
├── base.py
├── settings.py

```

每个业务 app 都具有一样的结构，配置和业务代码随着业务所需分布在各个不同的层次，抽象程度越高的代码，所在层次越高，比如全局合法性检查可以放置在 `apps/base.py` 中，越具体的业务所在的层次越低，比如具体的注册登录逻辑可以放置在 `apps/app/app.py` 中。如果业务发生变更，业务目录 `app` 可完整移植。

所有业务 `app` 都必须在 `apps/settings.py` 下注册才能生效，上下线 so easy.

#### RESTFul api

http 协议所支持的方法都可以在 turbo 中以大小写的方式区分来实现普通页面 render 和接口 api


```python
# render 
def get(self):
    pass
    
# api  
def GET(self):
    self._data = {'username': 'zhyq0826'}

```

#### 灵活的路由

路由紧随业务代码，提供了

- register_group_urls
- register_url

两种方式来实现路由的注册

```python

register.register_group_urls('', [
    ('/', app.HomeHandler),
    ('/index', app.HomeHandler, 'index'),
    ('/hello', app.HomeHandler, 'home'),
])

register.register_url('/v1/hello', app.ApiHandler)

```

可以十分方便的更改路由规则。

#### 集成很多实用的功能特性

在使用mongodb 和 tornado 的实际开发中遇到了各式各样的问题，因而 turbo 自身在发展过程中集成了很多小功能，非常实用。


1.可以使用命令行一键生成所有 collection 的索引
```bash
turbo-admin index <model_name>
```

2.类似 flux 的状态和事件分发机制，可以非常方便管理和更改全局变量状态，此特性可以用以请求统计，函数调用统计等方面

```python
#user.py

from turbo.flux import Mutation, register, State

mutation = Mutation(__file__)
state = State(__file__)

@register(mutation)
def increase_rank(rank):
    return rank+1


@register(mutation)
def dec_rank(rank):
    return rank-1
    
#actions.py

from turbo.flux import Mutation, register, dispatch, register_dispatch

import mutation_types

@register_dispatch('user', mutation_types.INCREASE)
def increase(rank):
    pass


def decrease(rank):
    return dispatch('user', mutation_types.DECREASE, rank)


@register_dispatch('metric', 'inc_qps')
def inc_qps():
    pass

```

3.mongodb 读写调用 hook，利用该 hook ，可以统计 mongodb 读写调用状况

```python


def collection_method_call(turbo_connect_ins, name):
    def outwrapper(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            if name in turbo_connect_ins._write_operators:
                turbo_connect_ins._model_ins.write_action_call(name, *args, **kwargs)

            if name in turbo_connect_ins._read_operators:
                turbo_connect_ins._model_ins.read_action_call(name, *args, **kwargs)

            return func(*args, **kwargs)

        return wrapper

    return outwrapper


class Model(object):

    def write_action_call(self, name, *args, **kwargs):
        """
        execute when write action occurs, note: in this method write action must be called asynchronously
        """
        pass

    def read_action_call(self, name, *args, **kwargs):
        """
        execute when read action occurs, note: in this method read action must be called asynchronously
        """
        pass

```

4.日志可以根据文件所在路径生成 logger 名称，也可以自定义 logger 路径、级别、名称等
```python

from turbo.util import getLogger

logger = getLogger(__file__)

logger = getLogger('feed', log_level=logging.DEBUG, log_path='/var/log/feed.log')

```

5.字符串化 mongodb 数据

```python

from turbo.util import to_str, to_list_str, to_dict_str, json_encode

data  = json_encode(to_str(db.collection.find()))

```
6.快捷的参数提取和转换

```python

    _get_params = {
        'need': [
            ('skip', int),
            ('limit', int),
        ],
        'option': [
            ('did', basestring, None)
        ]
    }


```

可调用 self.parameters 完成参数提取工作，必须的参数不存在或转换错误时，值为 `None`，可选的参数可以指定默认值。

```python
assert self.parameters['skip'] == 0
assert self.parameters['did'] == None
```


## 适时而建的轮子 turbo 

turbo 在生产环境中应用了近一年，最大的项目日独立用户过百万，还有十多个大大小小的项目的后端使用了 turbo。turbo 经过了最初的考验。 

后端的轮子对比前端来说，远不如后者的来的快、来的猛，后端更需要长久的磨练和雕琢，一点一滴都需要经过业务的锤炼和数据的洗礼，才能渐渐成品，成为解决特定问题的利器。而轮子的创造更要把握时机，顺势而发。过早，轮子不够健全，解决问题不够彻底，很难推广使用，甚至有可能成了绊脚石，过晚，很多问题已经导致代码积劳成疾，想要完全治愈，必然代价不菲。

turbo 正是为了解决 tornado 和 mongodb 类应用在面对代码规模庞大、多人协同所遭遇的种种问题而创造的，它的立足点非常实际，没有炫技，不浮夸，切中要害，直面问题。

turbo 的实用就是它的华丽所在。


- 仓库地址：[Github](https://github.com/wecatch/app-turbo) 
- QQ群交流: 387199856

