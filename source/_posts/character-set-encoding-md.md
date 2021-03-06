---
title: JAVA中的字符集和字符编码
date: 2018-08-10 23:19:41
tags:
    - ASCII
    - Unicode
    - UTF-8
    - UTF-16
    - UTF-32
categories:
    - 编程
---

> 本文参考了很多优秀的文章,终于理清了JAVA中的字符集和编码问题,为后续的lucene分析奠定基础.

#### 什么是字符集和字符编码

- 字符集: 
    - 字符的集合,常见的有ASCII,Unicode字符集
- 字符编码: 
    - codepoint码点是字符的数字形式,而计算机中都是二进制,对于一个二进制串如何判断是ASCII字符还是Unicode字符?
    - 显然直接用这种码点的二进制存储和传输是不行的,便有各种编码方式,常见流行的有UTF-8这种编码
    - ***实际存储,传输的数据***是对码点的二进制进行转换/编码后的数据(也就是按编码方案)

#### ASCII字符集及编码

- 我们知道，计算机内部，所有信息最终都是一个二进制值。每一个二进制位（bit）有0和1两种状态，因此八个二进制位就可以组合出256种状态，这被称为一个字节（byte）。也就是说，一个字节一共可以用来表示256种不同的状态，每一个状态对应一个符号，就是256个符号，从00000000到11111111。上个世纪60年代，美国制定了一套字符编码，对英语字符与二进制位之间的关系，做了统一规定。这被称为 ASCII 码，一直沿用至今。
- ASCII 码一共规定了128个字符的编码，比如空格SPACE是32（二进制00100000），大写的字母A是65（二进制01000001）。这128个符号（包括32个不能打印出来的控制符号），只占用了一个字节的后面7位，最前面的一位统一规定为0。

#### Unicode字符集

- Unicode 当然是一个很大的集合，现在的规模可以容纳100多万个符号。每个符号的编码都不一样，比如，U+0639表示阿拉伯字母Ain，U+0041表示英语的大写字母A，U+4E25表示汉字严。具体的符号对应表，可以查询unicode.org，或者专门的汉字对应表。
- Unicode的编码空间从U+0000到U+10FFFF,共有1112064个码位(code point)可用来映射字符,码位就是字符的数字形式
- 这部分编码空间可以划分为17个平面(plane),每个平面包含2^16(65536)个码位
- 第一个平面称为基本多语言平面(Basic Multilingual Plane,BMP),或称第零平面(Plane 0)
- 其他平面称为辅助平面(Supplementary Planes)
- 基本多语言平面内,从U+D800到U+DFFF之间的码位区块是永久保留不映射到Unicode字符
- UTF-16就利用保留下来的0xD800-0xDFFF区段的码位来对辅助平面的字符的码位进行编码
- 最常用的字符都包含在BMP中,用2个字节表示

#### Unicode字符集编码方式

- Unicode具有多种编码方案
- Unicode字符集规定的标准编码方案是UCS-2(UTF-16)用两个字节表示一个Unicode字符
- UCS-4(UTF-32)用4个字节表示一个Unicode字符
- 另外一个常用的Unicode编码方案–UTF-8用1到4个变长字节来表示一个Unicode字符,并可以从一个简单的转换算法从UTF-16直接得到
- UTF-16不兼容ASCII编码导致不流行,而UTF-8兼容ASCII编码

#### UTF-8

- JVM规范中明确说明了java的char类型使用的编码方案是UTF-16
- UTF-8 最大的一个特点，就是它是一种变长的编码方式。它可以使用1~4个字节表示一个符号，根据不同的符号而变化字节长度。
- UTF-8 的编码规则很简单，只有二条：
    - 对于单字节的符号，字节的第一位设为0，后面7位为这个符号的 Unicode 码。因此对于英语字母，UTF-8 编码和 ASCII 码是相同的。
    - 对于n字节的符号（n > 1），第一个字节的前n位都设为1，第n + 1位设为0，后面字节的前两位一律设为10。剩下的没有提及的二进制位，全部为这个符号的 Unicode 码。
    - 下表总结了编码规则，字母x表示可用编码的位。
    Unicode符号范围     |               UTF-8编码方式
    (十六进制)          |              （二进制）
    ----------------------+---------------------------------------------
    0000 0000-0000 007F | 0xxxxxxx
    0000 0080-0000 07FF | 110xxxxx 10xxxxxx
    0000 0800-0000 FFFF | 1110xxxx 10xxxxxx 10xxxxxx
    0001 0000-0010 FFFF | 11110xxx 10xxxxxx 10xxxxxx 10xxxxxx
    - 跟据上表，解读 UTF-8 编码非常简单。如果一个字节的第一位是0，则这个字节单独就是一个字符；如果第一位是1，则连续有多少个1，就表示当前字符占用多少个字节。
    - 下面，还是以汉字严为例，演示如何实现 UTF-8 编码。
    - 严的 Unicode 是4E25（100111000100101），根据上表，可以发现4E25处在第三行的范围内（0000 0800 - 0000 FFFF），因此严的 UTF-8 编码需要三个字节，即格式是1110xxxx 10xxxxxx 10xxxxxx。然后，从严的最后一个二进制位开始，依次从后向前填入格式中的x，多出的位补0。这样就得到了，
    - 严的 UTF-8 编码是11100100 10111000 10100101，转换成十六进制就是E4B8A5。

#### 传输,存储的顺序是否和编码顺序一致(大小端)

- UCS-2 格式可以存储 Unicode 码（码点不超过0xFFFF）
    - 以汉字严为例，Unicode 码是4E25，需要用两个字节存储，一个字节是4E，另一个字节是25
    - 存储的时候，4E在前，25在后，这就是 Big endian 方式；25在前，4E在后，这是 Little endian 方式
- 第一个字节在前，就是"大头方式"（Big endian），第二个字节在前就是"小头方式"（Little endian）。
- 那么很自然的，就会出现一个问题：计算机怎么知道某一个文件到底采用哪一种方式编码？
- Unicode 规范定义
    - 每一个文件的最前面分别加入一个表示编码顺序的字符,这个字符的名字叫做"零宽度非换行空格",用FEFF表示,这正好是两个字节,而且FF比FE大1
    - 如果一个文本文件的头两个字节是FE FF,就表示该文件采用大头方式
    - 如果头两个字节是FF FE,就表示该文件采用小头方式

#### 实例

- 打开"记事本"程序notepad.exe，新建一个文本文件，内容就是一个严字，依次采用ANSI，Unicode，Unicode big endian和UTF-8编码方式保存。
- 然后，用文本编辑软件UltraEdit 中的"十六进制功能"，观察该文件的内部编码方式。
    1）ANSI：文件的编码就是两个字节D1 CF，这正是严的 GB2312 编码，这也暗示 GB2312 是采用大头方式存储的。
    2）Unicode：编码是四个字节FF FE 25 4E，其中FF FE表明是小头方式存储，真正的编码是4E25。
    3）Unicode big endian：编码是四个字节FE FF 4E 25，其中FE FF表明是大头方式存储。
    4）UTF-8：编码是六个字节EF BB BF E4 B8 A5，前三个字节EF BB BF表示这是UTF-8编码，后三个E4B8A5就是严的具体编码，它的存储顺序与编码顺序是一致的。

#### UTF-16

- JVM规范中明确说明了java的char类型使用的编码方案是UTF-16，所以先来了解下UTF-16。
- 基本多语言平面内，从U+D800到U+DFFF之间的码位区块是永久保留不映射到Unicode字符。UTF-16就利用保留下来的0xD800-0xDFFF区段的码位来对辅助平面的字符的码位进行编码。
- 最常用的字符都包含在BMP中，用2个字节表示。辅助平面中的码位，在UTF-16中被编码为一对16比特长的码元，称作代理对（surrogate pair），具体方法是：
    - 将码位减去0x10000,得到的值的范围为20比特长的0~0xFFFFF。
    - 高位的10比特的值（值的范围为0~0x3FF）被加上0xD800得到第一个码元或称作高位代理（high surrogate），值的范围是0xD800~0xDBFF.由于高位代理比低位代理的值要小，所以为了避免混淆使用，Unicode标准现在称高位代理为前导代理（lead surrogates）。
    - 低位的10比特的值（值的范围也是0~0x3FF）被加上0xDC00得到第二个码元或称作低位代理（low surrogate），现在值的范围是0xDC00~0xDFFF.由于低位代理比高位代理的值要大，所以为了避免混淆使用，Unicode标准现在称低位代理为后尾代理（trail surrogates）。
- 例如U+10437编码:
    - 0x10437减去0x10000,结果为0x00437,二进制为0000 0000 0100 0011 0111。
    - 分区它的上10位值和下10位值（使用二进制）:0000000001 and 0000110111。
    - 添加0xD800到上值，以形成高位：0xD800 + 0x0001 = 0xD801。
    - 添加0xDC00到下值，以形成低位：0xDC00 + 0x0037 = 0xDC37。
- 由于前导代理、后尾代理、BMP中的有效字符的码位，三者互不重叠，搜索时一个字符编码的一部分不可能与另一个字符编码的不同部分相重叠。所以可以通过仅检查一个码元（构成码位的基本单位，2个字节）就可以判定给定字符的下一个字符的起始码元。
- java中的codepoint相关
    - 对于一个字符串对象，其内容是通过一个char数组存储的。char类型由2个字节存储，这2个字节实际上存储的就是UTF-16编码下的码元。我们使用charAt和length方法的时候，返回的实际上是一个码元和码元的数量，虽然一般情况下没有问题，但是如果这个字符属于辅助平面字符，以上2个方法便无法得到正确的结果。正确的处理方式如下：
```java
int character = aString.codePointAt(i);
int length = aString.codePointCount(0, aString.length())；
需要注意codePointAt的返回值，是int而非char,这个值就是Unicode码。
codePointAt方法调用了codePointAtImpl：
static int codePointAtImpl(char[] a, int index, int limit) {
    char c1 = a[index];
    if (isHighSurrogate(c1) && ++index < limit) {
        char c2 = a[index];
        if (isLowSurrogate(c2)) {
            return toCodePoint(c1, c2);
        }
    }
    return c1;
}
isHighSurrogate方法判断下标字符的2个字节是否为UTF-16中的前导代理（0xD800~0xDBFF）：
public static boolean isHighSurrogate(char ch) {
    // Help VM constant-fold; MAX_HIGH_SURROGATE + 1 == MIN_LOW_SURROGATE
    return ch >= MIN_HIGH_SURROGATE && ch < (MAX_HIGH_SURROGATE + 1);
}
public static final char MIN_HIGH_SURROGATE = '\uD800';
public static final char MAX_HIGH_SURROGATE = '\uDBFF';
然后++index,isLowSurrogate方法判断下一个字符的2个字节是否为后尾代理（0xDC00~0xDFFF）：
public static boolean isLowSurrogate(char ch) {
    return ch >= MIN_LOW_SURROGATE && ch < (MAX_LOW_SURROGATE + 1);
}
public static final char MIN_LOW_SURROGATE  = '\uDC00';
public static final char MAX_LOW_SURROGATE  = '\uDFFF';
toCodePoint方法将这2个码元组装成一个Unicode码：
public static int toCodePoint(char high, char low) {
    // Optimized form of:
    // return ((high - MIN_HIGH_SURROGATE) << 10)
    //         + (low - MIN_LOW_SURROGATE)
    //         + MIN_SUPPLEMENTARY_CODE_POINT;
    return ((high << 10) + low) + (MIN_SUPPLEMENTARY_CODE_POINT
                                    - (MIN_HIGH_SURROGATE << 10)
                                    - MIN_LOW_SURROGATE);
}
这个过程就是以上将一个辅助平面的Unicode码位转换成2个码元的逆过程。
所以，枚举字符串的正确方法：
for (int i = 0; i < aString.length();) {
	int character = aString.codePointAt(i);
	//如果是辅助平面字符，则i+2
	if (Character.isSupplementaryCodePoint(character)) i += 2;
	else ++i;
}
将codePoint转换为char[]可调用Character.toChars方法，然后可进一步转换为字符串：	
new String(Character.toChars(codePoint));
toChars方法所做的就是以上将Unicode码位转换为2个码元的过程,这2个码元最后就成了字符串对象包含的char数组中的2个元素。
```

#### References

- [字符编码笔记：ASCII，Unicode 和 UTF-8](http://www.ruanyifeng.com/blog/2007/10/ascii_unicode_and_utf-8.html)
- [聊聊java中codepoint和UTF-16相关的一些事](https://vinoit.me/2016/10/07/codePoint-in-java-and-utf16/)

<!--more-->