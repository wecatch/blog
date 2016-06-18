title: 你真得理解 python 的浅拷贝和深拷贝吗？
date: 2016-06-18 17:00:28
tags:
- python
- 对象状态
categories:
- 技术
---

[三月沙](http://sanyuesha.com/about/) [原文链接](http://sanyuesha.com/2016/06/18/python-copy-deepcopy/)


为了让一个对象发生改变时不对原对象产生副作用，此时，需要一份这个对象的拷贝，python 提供了 copy 机制来完成这样的任务，对应的模块是 [copy](https://docs.python.org/2/library/copy.html)。

## 浅拷贝:shadow copy

在 `copy` 模块中，有 `copy` 函数可以完成浅拷贝。
```python
from copy import copy
```
在 python 中，标识一个对象唯一身份的是：对象的`id`(内存地址)，对象类型，对象值，而浅拷贝就是创建一个具有相同类型，相同值但不同`id`的新对象。

对可变对象而言，对象的值一样可能包含有对其他对象的引用，浅拷贝产生的新对象，虽然具有完全不同的`id`，但是其值若包含可变对象，这些对象和原始对象中的值包含同样的引用。

```python
>>> import copy
>>> l = {'a': [1,2,3], 'b':[4,5,6]}
>>> c = copy.copy(l)
>>> id(l) == id(c)
False
>>> l['a'].append('4')
>>> c['b'].append('7')
>>> l
{'a': [1, 2, 3, '4'], 'b': [4, 5, 6, '7']}
>>> c
{'a': [1, 2, 3, '4'], 'b': [4, 5, 6, '7']}
>>> 
```
可见浅拷贝产生的新对象中可变对象的值在发生改变时会对原对象的值产生副作用，因为这些值是同一个引用。

<!-- more -->


浅拷贝仅仅对对象自身创建了一份拷贝，而没有在进一步处理对象中包含的值。因此使用浅拷贝的典型使用场景是：对象自身发生改变的同时需要保持对象中的值完全相同，比如 list 排序。

```
>>> def sorted_list(olist, key=None):
...     copied_list = copy.copy(olist)
...     copied_list.sort(key=key)
...     return copied_list
... 
>>> a = [3,2,1]
>>> b = sorted_list(a)
>>> a
[3, 2, 1]
>>> b
[1, 2, 3]
```

## 深拷贝:deep copy

在 `copy` 模块中，有 `deepcopy` 函数可以完成深拷贝。
```python
from copy import deepcopy
```

深拷贝不仅仅拷贝了原始对象自身，也对其包含的值进行拷贝，它会递归的查找对象中包含的其他对象的引用，来完成更深层次拷贝。因此，深拷贝产生的副本可以随意修改而不需要担心会引起原始值的改变。

```python
>>> import copy
>>> l = {'a': [1,2,3], 'b':[4,5,6]}
>>> c = copy.deepcopy(l)
>>> id(l) == id(c)
False
>>> l['a'].append('4')
>>> c['b'].append('7')
>>> l
{'a': [1, 2, 3, '4'], 'b': [4, 5, 6]}
>>> c
{'a': [1, 2, 3], 'b': [4, 5, 6, '7']}
>>> 
```

值得注意的是，深拷贝并非完完全全递归查找所有对象，因为一旦对象引用了自身，完全递归可能会导致无限循环。一个对象被拷贝了，python 会对该对象做个标记，如果还有其他需要拷贝的对象引用着该对象，**它们的拷贝其实指向的是同一份拷贝**

```python
>>> a = [1,2]
>>> b = [a,a]
>>> b
[[1, 2], [1, 2]]
>>> c = deepcopy(b)
>>> id(b[0]) == id(c[0])
False
>>> id(b[0]) == id(b[1])
True
>>> c
[[1, 2], [1, 2]]
>>> c[0].append(3)  #c list 中包含的两份拷贝指向同一处
>>> c
[[1, 2, 3], [1, 2, 3]]
>>> 
```

## 自定义拷贝机制

使用 `_copy_` 和 `__deepcopy__` 可以完成对一个对象拷贝的定制。这里不展开了，有机会再探讨自定义拷贝。
