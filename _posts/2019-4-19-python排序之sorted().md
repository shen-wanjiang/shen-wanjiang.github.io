---
layout:     post
title:      python排序之sorted()
subtitle:   使用sorted()对字典列表等排序
date:       2019-04-19
author:     WJ
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - python
---

# 前言
> python中使用功能强大的sorted()函数,对序列排序

# sorted()介绍
与map,filter,reduce一样,sorted也是一个高阶函数 
语法:
`sorted(iterable[, key[, reverse]])`
参数说明:
- iterable -- 可迭代对象(列表,字典,字符串等)
- key -- 接受一个函数来实现自定义的排序(key指定的函数将作用于list的每一个元素上，并根据key函数返回的结果进行排序).
- reverse -- 排序规则，reverse = False 升序（默认）, reverse = True 降序

针对参数key进行补充:
    可以自定义一个函数,将可迭代对象中的每个元素依次放入函数,例如元素A经过函数后得到一个新的结果元素B(关于如何转换就可以根据我们需要自定义),按照新的结果B进行排序,但是展示,还是按照该结果对应的元素A进行展示.
```py
>>> sorted([3, 2, -4, -10, 5], key=abs)  # abs函数,为取绝对值函数,  abs(-3) --> 3
[2, 3, -4, 5, -10]
```
对比原始的list和经过`key=abs`处理过的list:
```py
list = [3, 2, -4, -10, 5]
keys = [3, 2, 4, 10, 5]
```
按照keys进行排序后, 将其对应的原始数据展示出来  
**用`sorted()`排序的关键在于实现一个映射函数。**

# 实际使用例子
下面展示几个常见的例子

#### 1. 字典排序: 按照字典的value值进行排序(从大到小)
```py
dic = {'a':31, 'bc':5, 'c':3, 'asd':4, 'aa':74, 'd':0}
dict= dict(sorted(dic.items(), key=lambda d:d[1], reverse = True))
print(dict)  # {'aa': 74, 'a': 31, 'bc': 5, 'asd': 4, 'c': 3, 'd': 0}
```
#### 2. 字典排序: 按照字典的key值进行排序(从小到大)
```py
dic = {'a':31, 'bc':5, 'c':3, 'asd':4, 'aa':74, 'd':0}
dict= dict(sorted(dic.items(), key=lambda d:d[0]))
print(dict)  # {'a': 31, 'aa': 74, 'asd': 4, 'bc': 5, 'c': 3, 'd': 0}
```
#### 3. 对列表中的字典进行排序
- 直接根据字典中的value排序
    ```py
    a = [{'a': 1, 'g': 19}, {'a': 2, 'g': 19}, {'a': 1, 'g': 11}]
    b = sorted(a, key=lambda x: x['a'])
    print(b)
    # 也可以这样写,python中一切皆对象
    alist_sort = sorted(alist,key=lambda e: e.__getitem__('a'),reverse=True)
    ```
- 当字典元素中value数值相同的情况下,根据第二个value进行排序
    ```py
    # 先根据a的值进行排序,如果a的值相同,就根据'g'的值进行排序
    a = [{'a': 1, 'g': 19}, {'a': 2, 'g': 19}, {'a': 1, 'g': 11}]
    b = sorted(a, key=lambda x: (x['a'], x['g']))  # 外层还可以用 reverse = True 表示倒叙
    print(b)
    ```
# 面试题
> 之前遇到的一个面试题目
#### 给定一个任意长度数组，实现一个函数
让所有奇数都在偶数前面，而且奇数升序排列，偶数降序排序，如字符串'1982376455',变成'1355798642'
```python
# 方法一
def func1(l):
    if isinstance(l, str):
        l = [int(i) for i in l]
    l.sort(reverse=True)
    for i in range(len(l)):
        if l[i] % 2 > 0:
            l.insert(0, l.pop(i))
    print(''.join(str(e) for e in l))

# 方法二
def func2(l):
    print("".join(sorted(l, key=lambda x: int(x) % 2 == 0 and 20 - int(x) or int(x))))
```
**重点说一下方法二. python的编程宗旨就是简洁**
- `"".join()`
    ```py
    # "".join()  将一个列表按照前方元素拼接为一个字符串
    >>> ''.join(['a', 'b', 'c'])
    'abc'
    ```
- sorted中的key
    ```py
    sorted(l, key=lambda x: int(x) % 2 == 0 and 20 - int(x) or int(x))
    
    int(x) % 2 == 0 and 20 - int(x) or int(x)
    # 先判断and部分: int(x) % 2 是否为偶数,如果为偶数则返回 20 - int(x). 如果为奇数返回False
    # 然后判断ｏｒ部分: 如果前方表达式为False(即:为奇数),返回结果 int(x), 如果前方表达式不为False,返回该数值(即:20-int(x))
    # 这里为什么用20,只是为了区分开了奇数和偶数,单个数最大为8(针对偶数),20-8=12也比单个数大, 所以,你可以设置该数值为比8+9=17(包含17)大的任何数字,都可以.
    ```

    **这里说明的是,后面的逻辑运算符的运算. 请参考我的另一个文章: python中的逻辑运算符**