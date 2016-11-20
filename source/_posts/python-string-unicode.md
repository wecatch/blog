title: 如何理解 python UnicodeEncodeError ：python 的 string 和 unicode
date: 2016-11-06 20:48:08
tags:
- python
- 编码
categories:
- 技术
---

[三月沙](http://sanyuesha.com/about/) [原文链接](http://sanyuesha.com/2016/11/06/python-string-unicode/)


> 文中 python 皆为 2.x 版本

初学 python 的人基本上都有过如下类似经历:

**UnicodeDecodeError**
```python
Traceback (most recent call last):
  File "<input>", line 1, in <module>
UnicodeDecodeError: 'ascii' codec can't decode byte 0xe4 in position 0: ordinal not in range(128)
```
**UnicodeEncodeError**
```python
Traceback (most recent call last):
  File "<input>", line 1, in <module>
UnicodeEncodeError: 'ascii' codec can't encode characters in position 0-1: ordinal not in range(128)
```
这两个错误在 python 中十分常见，一不留神就碰上了。如果你写过c、c++ 或者 java，对比之下一定会觉得 python 这个错误真让人火大。事实也确实如此，我也曾经很火大🔥。

这两个错误究竟意味着什么？可以先从 python 的基本数据类型 string 和 unicode 开始。

# string 
字符串(string)其实就是一段文本序列，是由一个或多个字符组成(character)，字符是文本的最小构成单元，在 python 中可以用以下方式表示字符串:

```bash
>>> s1 = 'abc'
>>> s2 = "abc"
>>> s3 = """
  abc
  """
>>> s4 = '中文'
>>> for i in [s1, s2, s3, s4]:
        print type(i)
<type 'str'>
<type 'str'>
<type 'str'>
<type 'str'>
```

这些变量在 python shell 中对应输出是:

```bash
s1 --> 'abc'
s2 --> 'abc'
s3 --> '\nabc\n'
s4 --> '\xe4\xb8\xad\xe6\x96\x87'
```

s4 的输出和其它变量明显不同，字面上是一个 16 进制序列，但是 s4 和其它字符串一样，在 python 内部都是用同样方式进行存储的: 字节流(byte stream)，即字节序列。

字节是计算机内部最小的可寻址的存储单位(对大部分计算机而言)，一个字节是由 8 bit 组成，也就是对应 8 个二进制位。其实可以更进一步解释说，python 不仅用字节的方式存储着变量中的字符串文本，python 文件中的所有信息在计算机内部都是用一个个字节表示的，计算机是用这样的方式存储文本数据的。

## 字符串用字节如何表示？

答案就是**编码**。计算机是只能识别 0 或 1 这样的二进制信息，而不是 a 或 b 这样对人类有意义的字符，为了让机器能读懂这些字符，人类就发明字符到二进制的映射关系，然后按照这个映射规则进行相应地编码。[ascii](https://en.wikipedia.org/wiki/ascii) 就是这样背景下诞生的一种编码规则。**ascii 也是 python 2.x 默认使用的编码规则。**

ascii 规定了常用的字符到计算机是如何映射的，编码范围是 0~127 共 128 个字符。简单来说它就是一本字典，规定了不同字符的对应的编码值(code point，一个整数值)，这样一来计算机就能用二进制表示了。比如字符 a 的编码是 97，对应的二进制是 1100001，**一个字节**就足够存储这些信息。字符串 "abc" 最终存储就是 `[97] [98] [99]` 三个字节。python 默认情况下就是使用这个规则对字符进行编码，对字节进行解码(反编码)。

```python
>>> ord('a')
97
>>> chr(97)
'a'
>>> 
```

由于 ascii 的编码范围非常有限，对超过 ascii 范围之外的字符，python 是如何处理的？很简单，抛错误出来，这就是 `UnicodeEncodeError` 和 `UnicodeDecodeError` 的来源。那 python 会在什么时候抛出这样的错误，也就是说 python 进行编码和解码的操作发生在何时？

# unicode 对象
unicode 对象和 string 一样，是 python 中的一种字符对象(python 中一切皆对象，string 也是)。先不要去想 unicode 字符集、unicode 编码或者 utf-8 这些概念，在此特意加了`对象`就是为了和后面提到的 unicode 字符集进行区分。这里说的 unicode 就是 python 中的 unicode 对象，构造函数是 `unicode()`。

在 python 中创造 unicode 对象也很简单:

```python
>>> s1 = unicode('abc')
>>> s2 = u'abc'
>>> s3 = U'abc'
>>> s4 = u'中文'
```

这些变量在 python shell 中对应输出是:
```bash
s1 --> u'abc'
s2 --> u'abc'
s3 --> u'abc'
s4 --> u'\u4e2d\u6587'
```
同样的，s4 的输出和其它变量不同，这些就是unicode 字符。由于 ascii 能够表示的字符太少，而且不够通用(扩展 ascii 的话题，就是把 ascii 没有利用的剩下大于 127 的位置利用了，在不同的字符集里代表不同的意思)，[unicode 字符集](https://zh.wikibooks.org/wiki/Unicode) 就被造出来了，一本更大的字典，里面有更多的编码值。

## unicode 字符集

unicode 字符集解决了：

- ascii 表达能力不够的问题
- 扩展 ascii 不够通用的问题

虽然 unicode 字符集表达能力强，又能够统一字符编码规则，但是它并没有规定这些字符在计算机中是如何表示的。它和 ascii 不同，很多字符(编码值大于 255 )没有办法用一个字节就搞定。怎样做到高效快捷地存储这些编码值？于是就有了 unicode 字符集的编码规则的实现：utf-8、utf-16等。

到这里可以简单理清 ascii、unicode 字符集、utf-8等的关系了：ascii 和 unicode 字符集都是一种编码集，由字符和字符对应的整数值(code point)组成，ascii 在计算机内部用一个字节存储，utf-8 是 unicode 字符集存储的具体实现，因为 unicode 字符集没有办法简简单单用一个字节搞定。

回到 s4 对应的输出，这个输出就是 unicode 字符集对应的编码值(code point)的 16 进制表示。

unicode 对象是用来表示 unicode 字符集中的字符，这些字符(实际是那个编码值，一个整数) 在 python 中又是如何存储的？有了前文的分析，也许可以猜到，python 依然是通过编码然后用字节的方式存储，但是这里的编码就不能是 ascii 了，而是对应 unicode 字符集的编码规则: utf-8、utf-16等。

## unicode 对象的编码

unicode 对象想要正确的存储就必须指定相应的编码规则，这里我们只讨论使用最广泛的 utf-8 实现。

在 python 中对 unicode 对象编码如下：

```python
>>> s=u'中文'
>>> s.encode('utf-8')
'\xe4\xb8\xad\xe6\x96\x87'
>>> type(s.encode('utf-8'))
<type 'str'>
```

编码之后输出的是个 string 并以字节序列的方式进行存储。有了编码就会有解码，python 正是在这种编码、解码的过程使用了错误的编码规则而发生了 `UnicodeEncodeError` 和 `UnicodeDecodeError` 错误，因为它默认使用 ascii 来完成转换。

# string 和 unicode 对象的转换

unicode 对象可以用 utf-8 编码为 string，同理，string 也可以用 utf-8 解码为 unicode 对象

```python
>>> u=u'中文'
>>> s = u.encode('utf-8')
>>> s
'\xe4\xb8\xad\xe6\x96\x87'
>>> type(s)
<type 'str'>
>>> s.decode('utf-8')
u'\u4e2d\u6587'
>>> type(s.decode('utf-8'))
<type 'unicode'>
```

错误的编码规则就会导致那两个常见的异常

```python
>>> u.encode('ascii')
Traceback (most recent call last):
  File "<input>", line 1, in <module>
UnicodeEncodeError: 'ascii' codec can't encode characters in position 0-1: ordinal not in range(128)
>>>
>>> s.decode('ascii')
Traceback (most recent call last):
  File "<input>", line 1, in <module>
UnicodeDecodeError: 'ascii' codec can't decode byte 0xe4 in position 0: ordinal not in range(128)
```

这两个错误在某些时候会突然莫名其妙地出现就是因为 python 自动地使用了 ascii 编码。

## python 自动解编码

1.stirng 和 unicode 对象合并

```python
>>> s + u''
Traceback (most recent call last):
  File "<input>", line 1, in <module>
UnicodeDecodeError: 'ascii' codec can't decode byte 0xe4 in position 0: ordinal not in range(128)
>>> 
```

2.列表合并

```python
>>> as_list = [u, s]
>>> ''.join(as_list)
Traceback (most recent call last):
  File "<input>", line 1, in <module>
UnicodeDecodeError: 'ascii' codec can't decode byte 0xe4 in position 0: ordinal not in range(128)
```

3.格式化字符串

```bash
>>> '%s-%s'%(s,u)
Traceback (most recent call last):
  File "<input>", line 1, in <module>
UnicodeDecodeError: 'ascii' codec can't decode byte 0xe4 in position 0: ordinal not in range(128)
>>> 
```

4.打印 unicode 对象

```python
#test.py
# -*- coding: utf-8 -*-
u = u'中文'
print u

#outpt
Traceback (most recent call last):
  File "/Users/zhyq0826/workspace/zhyq0826/blog-code/p20161030_python_encoding/uni.py", line 3, in <module>
    print u
UnicodeEncodeError: 'ascii' codec can't encode characters in position 0-1: ordinal not in range(128)
```

5.输出到文件

```python
>>> f = open('text.txt','w')
>>> f.write(u)
Traceback (most recent call last):
  File "<input>", line 1, in <module>
UnicodeEncodeError: 'ascii' codec can't encode characters in position 0-1: ordinal not in range(128)
>>>
```

---

1，2，3 的例子中，python 自动用 ascii 把 string 解码为 unicode 对象然后再进行相应操作，所以都是 `decode` 错误， 4 和 5 python 自动用 ascii 把 unicode 对象编码为字符串然后输出，所以都是 `encode` 错误。

只要涉及到 unicode 对象和 string 的转换以及 unicode 对象输出、输入的地方可能都会触发 python 自动进行解码/编码，比如写入数据库、写入到文件、读取 socket 等等。

到此，这两个异常产生的真正原因了基本已经清楚了: unicode 对象需要**编码**为相应的 string(字符串)才可以存储、传输、打印，字符串需要**解码**为对应的 unicode 对象才能完成 unicode 对象的各种操作，`len`、`find` 等。

```python
string.decode('utf-8') --> unicode
unicode.encode('utf-8') --> string
```

# 如何避免这些的错误

1.理解编码或解码的转换方向

无论何时发生编码错误，首先要理解编码方向，然后再针对性解决。

2.设置默认编码为 utf-8

在文件头写入 

```python
# -*- coding: utf-8 -*-
```

python 会查找: coding: name or coding=name，并设置文件编码格式为 `name`，此方式是告诉 python 默认编码不再是 ascii ，而是要使用声明的编码格式。

3.输入对象尽早解码为 unicode，输出对象尽早编码为字节流

无论何时有字节流输入，都需要尽早解码为 unicode 对象。任何时候想要把 unicode 对象写入到文件、数据库、socket 等外界程序，都需要进行编码。

4.使用 codecs 模块来处理输入输出 unicode 对象

[codecs](https://docs.python.org/2/library/codecs.html) 模块可以自动的完成解编码的工作。

```python
>>> import codecs
>>> f = codecs.open('text.txt', 'w', 'utf-8')
>>> f.write(u)
>>> f.close()
```

> 参考文献
- https://zh.wikibooks.org/wiki/Unicode
- https://zh.wikibooks.org/wiki/ascii
- http://www.unicode.org/
- https://docs.python.org/2/howto/unicode.html
- http://www.ruanyifeng.com/blog/2007/10/ascii_unicode_and_utf-8.html

> 注意：转载请注明出处和文章链接