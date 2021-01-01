# 正则表达式详解（下）


书接上文，在下篇中主要介绍如何在 python 中使用正则表达式实现文本的匹配和替换工作。

<!--more-->


## 匹配字符串

python 的 re 模块支持正则表达式，基本步骤是使用 compile 编译正则表达式为 pattern 实例，之后使用实例匹配目标文本（一般使用方法 match 和 search）。先来看一个简单的例子，

```python
# encoding: UTF-8
import re

string = "Tutorial.  0123456789 _ + -., !@  # $%^&*();\/|<>\"\'12345 - 98.7 3.141 .6180 9,000 + 42 55.123.4567 + 1 - (800) - 555 - 2468 foo@demo.net ar.ba@test.co.uk www.demo.com"
pattern = re.compile(r"\d+", re.M)
sear = re.search(pattern, string)
print('sear', sear)

mat = re.match(pattern, string)
print('mat', mat)

fin = re.findall(pattern, string)
print('fin', fin)
```

代码第 4 行，使用正则表达式`r"\d+"`compile 编译一个 pattern 的实例。

> 为什么要在正则表达式之前加个`r`呢？

此处的`r`表示 raw，即原始输入。因为`\`在 python 的字符串中也是转义字符，此处的功能与正则表达式中的功能有冲突。`compile`中的表达式从输入到真正的表达式的意思执行需要经过两次解读，**python 本身解读之后将字符串表达的意思传递给正则表达式，正则引擎再解读之后执行**，问题就出在两次解读上。

以最终匹配`\section`为例，如果不加`r`使用 python 第 5 行的参数就该是`\\\\section`，即`pattern = re.compile("\\\\section", re.M)`。第一次解读，python 将其理解为`\\section`传递给正则引擎，正则理解为要匹配`\section`，如下表所示

| Characters    | Stage                                                        |
| ------------- | ------------------------------------------------------------ |
| `\section`    | 希望匹配的字符串                                             |
| `\\section`    | 正则引擎添加反斜线 [`re.compile()`](https://bugxch.github.io/library/re.html#re.compile) |
| `"\\\\section"` | python 字符串为表示`\`字符再添加反斜线                       |

所以为了匹配 1 个`\`最后需要写 4 个`\`，实在麻烦，**为了便利，因此使用r在字符串之前，表示去掉 Python 的解读，仅保留正则引擎的解读。**比如下表中的参数写法就能省掉很多不必要的`\`，十分方便。

| Regular String | Raw string    |
| -------------- | ------------- |
| `"ab*"`        | `r"ab*"`      |
| `"\\section"`   | `r"\section"` |
| `"\\w+\\s+\\1"`   | `r"\w+\s+\1"` |

> `re.M`是个什么东东？

上面的代码中，compile 还包含了`re.M`作为一个 flag，它是`MULTILINE`的缩写，表示匹配多行模式，除了多行模式之外（具体含义见下表），python 还有其他的 flag 影响表达式匹配的规则。

| Flag                            | Meaning                                           |
| ------------------------------- | ------------------------------------------------- |
| `ASCII`, `A`                    | 使得 `\w`, `\b`, `\s` 和 `\d` 仅仅匹配 ASCII 文本 |
| `DOTALL`, `S`                   | 使得 `.` 匹配换行符在内的所有字符                 |
| `IGNORECASE`, `I`               | 忽略大小写                                        |
| `LOCALE`, `L`                   | 使用 locale 匹配                                  |
| `MULTILINE`, `M`                | 多行匹配，影响 `^` 和 `$`                         |
| `VERBOSE`, `X` (for ‘extended’) | 忽略表达式中的空格，允许插入注释                  |

需要说明的是`X`模式，使用方法如下

```python
charref = re.compile(r"""
 &[#]                # Start of a numeric entity reference
 (
     0[0-7]+         # Octal form
   | [0-9]+          # Decimal form
   | x[0-9a-fA-F]+   # Hexadecimal form
 )
 ;                   # Trailing semicolon
""", re.VERBOSE)
```

表达式内部加入了注释，而且可以多行显示，可以使用它表示复杂的表达式，非常方便。

> 匹配正则表达式的方法

pattern 的对象建立之后，就是可以使用 re 的方法匹配字符串了。最重要的方法有下表中的四个。

| 方法或者属性 | 目的                                                         |
| ------------ | ------------------------------------------------------------ |
| `match()`    | 表达式是否匹配字符串的开头                                   |
| `search()`   | 浏览整个字符串，匹配其中任意位置                             |
| `findall()`  | 匹配所有的字符子串，返回列表                                 |
| `finditer()` | 匹配所有的字符子串，返回 [iterator](https://bugxch.github.io/glossary.html#term-iterator). |

运行上面的代码如下所示

```python
>>>
sear <_sre.SRE_Match object; span=(11, 21), match='0123456789'>
mat None
fin ['0123456789', '12345', '98', '7', '3', '141', '6180', '9', '000', '42', '55', '123', '4567', '1', '800', '555', '2468']
```

sear 匹配成功，返回匹配的对象。mat 匹配失败，fin 匹配成功并且返回了字符列表。

匹配成功之后对象有如下方法

| 方法      | 目的                                                |
| --------- | --------------------------------------------------- |
| `group()` | 返回匹配的字符串                                    |
| `start()` | 返回匹配字符串的开始位置                            |
| `end()`   | 返回匹配字符串的结束位置                            |
| `span()`  | 返回匹配字符串开始位置和结束位置的元组 (start, end) |

`group()`在上篇中讲过，它在字符捕获时候非常有用，字符的分组从 0 开始，其中第 0 个代表所有匹配的内容，1 代表匹配的第一组，以此类推。比如下面的代码

```
>>> p = re.compile('(a(b)c)d')
>>> m = p.match('abcd')
>>> m.group(0)
'abcd'
>>> m.group(1)
'abc'
>>> m.group(2)
'b'
```

可以将`group(0)`理解成没有分组符号匹配的所有字符，而之后的编号代表分组依次匹配的子串。

## 修改字符串

模块级别的字符串修改方法如下表所示。

| 方法和属性 | 目的                                                     |
| ---------- | -------------------------------------------------------- |
| `split()`  | 使用 pattern 将目标字符串分割，并返回列表                |
| `sub()`    | 查找所有匹配的子串，并替换之，返回替换之后的字符串       |
| `subn()`   | 查找所有匹配的子串，并替换之，返回替换之后的字符串和次数 |

- `.split`(*string*[, *maxsplit=0*]) 将字符串分割，其中有参数最大分块，默认将分割成最多的块。

```python
>>> p = re.compile(r'\W+')
>>> p.split('This is a test, short and sweet, of split().')
['This', 'is', 'a', 'test', 'short', 'and', 'sweet', 'of', 'split', '']
>>> p.split('This is a test, short and sweet, of split().', 3)
['This', 'is', 'a', 'test, short and sweet, of split().']
```

## 替换字符串

- `.sub`(*replacement*, *string*[, *count=0*]) 将 string 中所有符合 pattern 模式替换成 replacement，count 表示替换的次数。看如下的例子

```python
>>> p = re.compile( '(blue|white|red)')
>>> p.sub( 'colour', 'blue socks and red shoes')
'colour socks and colour shoes'
>>> p.sub( 'colour', 'blue socks and red shoes', count=1)
'colour socks and red shoes'
```

方法`subn()`与`sub()`类似，但是它返回一个元组，其中包含替换之后的字符串，以及替换的次数，下面是另一个例子

```python
>>> p = re.compile( '(blue|white|red)')
>>> p.subn( 'colour', 'blue socks and red shoes')
('colour socks and colour shoes', 2)
>>> p.subn( 'colour', 'no colours at all')
('no colours at all', 0)
```

特别地，替换可以使用分组，这在交换字符串的位置时非常有用。举例如下

```python
>>> strin = 'This is part1 and part2'
>>> p = re.compile('(.*)(part1)(.*)(part2)')
>>> p.sub(r'\1\4\3\2', strin)
'This is part2 and part1'
```

我们还可以使用之前的名字分组，除了使用`\number`指代第几个分组之外，还可以使用`\g<number>`指代已经命名的分组。一个典型的例子如下所示

```python
>>> p = re.compile('section{ (?P<name> [^}]* ) }', re.VERBOSE)
>>> p.sub(r'subsection{\1}','section{First}')
'subsection{First}'
>>> p.sub(r'subsection{\g<1>}','section{First}')
'subsection{First}'
>>> p.sub(r'subsection{\g<name>}','section{First}')
'subsection{First}'
```

## 使用正则表达式

考虑使用正则表达式之前，先想想是否可以使用字符串方法就能解决问题。如果仅仅匹配和替换固定字符串，python 自带的字符串方法速度更快，效率更高，不是每个字符串的替换都需要使用正则表达式。

## 未尽事宜

相信以上介绍的内容能够解决大部分的字符处理问题，但是对于专业大数据处理可能还需要了解正则表达式更多的内容，比如不同语言的正则表达式语法，以及复杂正则表达式的匹配效率。这些比较深入的知识可以参考 Jeffrey E.F. Friedl 的[《精通正则表达式》](http://datamining.xmu.edu.cn/documentation/books/mastering_regular_expressions.pdf)，里面有非常专业的解释，相信它可以帮你解决正则表达式的绝大部分疑问。

![封面](http://image-7.verycd.com/ff45820ed025353061ccb475dc49c57d109161/%E7%B2%BE%E9%80%9A%E6%AD%A3%E5%88%99%E8%A1%A8%E8%BE%BE%E5%BC%8F.jpg)
