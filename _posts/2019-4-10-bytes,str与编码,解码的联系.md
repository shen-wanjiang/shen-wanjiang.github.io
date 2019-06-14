---
layout:     post
title:      编码与解码
subtitle:   python中的bytes,str类型, 以及编码解码
date:       2019-4-10
author:     WJ
header-img: img/post-bg-android.jpg
catalog: true
tags:
    - python
    - python开发基础
---

## python中的bytes,str类型, 以及编码解码
1. python3中有bytes和string类型
    - bytes主要是给在计算机看的，string主要是给人看的
    - 中间有个桥梁就是编码规则，现在大趋势是utf8
    - bytes对象是二进制，很容易转换成16进制，例如\x64
    - string就是我们看到的内容，例如'abc'
    - string经过编码encode，转化成二进制对象，给计算机识别, 也就是bytes类型
    - bytes经过反编码decode，转化成string，但是注意反编码的编码规则是有范围,\xc8就不是utf8识别的范围
	```py
	>>> '€20'.encode('utf-8')
	b'\xe2\x82\xac20'  # bytes对象,二进制
	>>> b'\xe2\x82\xac20'.decode('utf-8')
	'€20'
	>>>'hello'.encode('utf-8')
	b'hello'
	>>>b'hello'.decode('utf-8')
	'hello'
	```
2. 转换
字符串和字节符之间划分界线是必然的。下面这个图解要牢记于心：
![图片](https://images2017.cnblogs.com/blog/764761/201708/764761-20170828175555827-950234753.png)

	字符串(string)由字符组成，字符也是抽象的实体且与任何二进制表示无关。
	当操纵字符串的时候，很多细节是不用了解的。我们可以分割、切片和拼接字符串，在字符串内部进行搜索。但并不在乎内部是如何表示的，也不用在意底层一个字符要花费多少byte。
	只有在需要将string编码(encode)成byte的时候，比如：通过网络传输数据；或者需要将byte解码(decode)成string的时候，我们才会关注string和byte的区别。

3. 之前的桥梁: 编码方式
传入encode和decode的参数是编码方式。编码是一种用二进制数据表示抽象字符的方式。目前有很多种编码,常用的编码格式无非有：ASCII、Unicode、UTF-8， 默认的是utf-8。 
	```py
	>>> '€20'.encode('iso-8859-15')
	b'\xa420'
	>>> b'\xa420'.decode('iso-8859-15')
	'€20'
	```
	**编码是这个转换过程中至关重要的一部分。若不编码，bytes对象b'\xa420'只是一堆比特位而已。编码赋予其含义。采用不同的编码，这堆比特位的含义就会大不同：**
	```py
	>>> b'\xa420'.decode('windows-1255')  # 解释含义就不同
	'₪20'
	```
4. 代码在ide中的展示问题
	```py
	>>>'hello'.encode('utf-8')
	b'hello'
	```
	这里ide中用b''来标示出来,为了方便我们阅读,关于,ide怎么流程将二进制数据转变给我们展示,我们并不需要关心,我们只需要知道你现在获得了一个bytes的对象

5. unicode, utf-8
    1. 编码-作为动词
    「编码」在汉语里可以作动词使用，编码就是把一个字符（严格一点说是字符在字符集中的编号 code point）转换成一个字节序列，以便在网络传输或者存储到文本中。比如「好」在 Unicode 中的编号是 U+597d，经过 UTF-8 编码后会转换成二进制序列是 '\xe5\xa5\xbd' 。
	    ```py
	    >>> a = u"好"
	    >>> a
	    u'\u597d'  # 如果你打印出的是 "好",说明已经被转换后给你展示出来了. 所有,这里 u'好' 和 '好'  是一样的, 你输入'好',也就是输入了unicode表中的'好'对应的编号
	    >>> b = a.encode("utf-8")  # 所以,这里就相当于字符串的编码
	    >>> b
	    '\xe5\xa5\xbd'
	    >>>
	    ```

    2. 编码-作为名词
    「编码」还可以做名词使用，作为名词使用时，就是指一种具体的编码实现方式，比如 ASCII 编码，GBK 编码，UTF-8 编码，都叫做编码，是一种具体的实实在在的编码格式。

    3. Unicode定义
    把编码概念弄清楚了之后，我们就可以来定义 Unicode 了。其实**Unicode 是一个囊括了世界上所有字符的字符集，其中每一个字符都对应有唯一的编码值（code point），然而它并不是一种什么编码格式，仅仅是字符集而已**。 Unicode 字符要存储要传输怎么办，它不管，具体怎么编码，你们可以自己去实现，可以用 UTF-8、UTF-16、甚至用 GBK 来编码也是可以的。比如：
	    ```py
	    >>> a = u"好"
	    >>> a
	    u'\u597d'
	    >>> b = a.encode("gbk")
	    >>> b
	    '\xba\xc3'  # 这是二进制对象,这才是存储到你机器上的数据. 所有二进制数据,只有赋予了编码才有它的意义
	    ```
	    「好」用 GBK 编码转后就是用两个字节表示，用 UTF-8 编码就是 3 个字节，同一个字符用不同的编码方式占用的字节长度不一。

    4. 把 Unicode 和 ASCII 码作为字符集来理解时，需要区分二者吗？
    不需要，因为 Unicode 是在 ASCII 的基础之上建立的，比如 ASCII 码字符集中的 「A」对应的 code ponit 是 0x41，Unicode 码是 U+0041，两者是一样的。至于中日韩文字用 ASCII 根本没法表示，所以也不存在混淆的说法。

    5. Unicode规定每个字符用多少个字节来表示?
    Unicode 并没有统一规定每个符号用三个或者四个字节表示。Unicode 只规定了每个字符对应到唯一的代码值（code point），代码值 从 0000 ~ 10FFFF 共 1114112 个值 ，真正存储的时候需要多少个字节是由具体的编码格式决定的。比如：字符 「A」用 UTF-8 的格式编码来存储就只占用1个字节，用 UTF-16 就占用2个字节，而用 UTF-32 存储就占用4个字节。
    如下图:
    ![图片](https://raw.githubusercontent.com/shen-wanjiang/save_picture/master/markdown_pic/unicode.png)

    6. 当然,你也可以理解为每一种编码方式都对应一张表.
    Ascll用8位二进制表示所有的字母，unicode用16位二进制表示所有的符号，utf-8动态的用不同位数的二进制表示所有的符号。
    **注意这里,平时大家说的unicode编码,其实是不对的,它的含义值的是utf-16,即采用utf-16来编码**

    7. Python中的数据展示
    Python中的展示的数据都是在内存中，所以，不管以什么样的形式存储在内存中（unicode），我们来表示一个内容时候，可以用不同的标示来展示，如果是一个二进制，用b'xxx'，或者是一个字符串.
    decode()编码  encode()解码 bytes()转换为二进制  
        ```py
        cc= 'hello'.encode('utf-8') -->  c  -->  b'hello'    # 将字符串转换为byte类型
        cc.decode('utf-8') --> 'hello'                       # 将byte类型转换为字符串
        dd = bytes('hello', 'unicode_escape')  -->  dd  -->  b'hello'  #　将字符串转换为byte类型。  
        ```
        **bytes（）  和 encode()  采用相同的编程它们是等价的**  
    
    	展示问题
	    ```py
	    # 注意，这里python中展示二进制，用的直接就是b’hello’ 如果用汉字(你好)编码为二进制，那么在展示的时候，就不会直接用b’你好’来显示了，而是
	    bytes('你好', 'utf-8')  -->  b'\xe4\xbd\xa0\xe5\xa5\xbd'   # 用这样的二进制展示，其中\x表示16进制。 \xe4 = 11100100 相当于一个字节，八位二进制
	    bytes('你好', 'gbk')  -->  b'\xc4\xe3\xba\xc3'  # 可以看到不同的编码方式，得到的对应二进制数据不一样
	    bytes('你好', 'unicode_escape') -->  b'\\u4f60\\u597d'  # unicode-escape编码集，他是将unicode内存编码值直接存储,转换为二进制就可以存储了
    ```
