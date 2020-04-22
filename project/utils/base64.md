# base64

## base64的原理

base64的编码是基于64个字符A-Z, a-z, 0-9, +, /的编码方式，因为2的6次方正好是64,所以就用6 bit就可以表示出64个字符。

```
eg:
000000  --> A
000001  --> B
```

base64编码会以每3个字符(1Byte = 8 bit)为一组，然后针对每组，首先获取每个字符的ASCII编码(字符'a'=97=01100001)，然后将 ASCII 编码转换成 8 bit 的二进制，得到一组 3 * 8=24 bit 的字节。然后再将这 24 bit 划分为 4 个 6 bit 的字节，并在每个 6 bit 的字节前面都填两个高位 0，得到 4 个 8 bit 的字节，然后将这 4 个 8 bit 的字节转换成十进制，对照 BASE64 编码表 ，得到对应编码后的字符。

|           | L        | u        | c        | y        |          |          |        |        |
| --------- | -------- | -------- | -------- | -------- | -------- | -------- | ------ | ------ |
| ASCII     | 76       | 117      | 99       | 121      |          |          |        |        |
| 8 bit字节 | 01001100 | 01110101 | 01100011 | 01111001 | 00000000 | 00000000 |        |        |
| 6 bit字节 | 010011   | 000111   | 010101   | 100011   | 011110   | 010000   | 000000 | 000000 |
| 十进制    | 19       | 7        | 21       | 35       | 30       | 16       |        |        |
| 对应编码  | T        | H        | V        | j        | e        | Q        | =      | =      |



**Base64编码本质**上是一种将二进制数据转成文本数据的方案。对于非二进制数据，是先将其转换成二进制形式，然后每连续6比特（2的6次方=64）计算其十进制值，根据该值在A--Z,a--z,0--9,+,/ 这64个字符中找到对应的字符，最终得到一个文本字符串。



1. 标准Base64只有64个字符（英文大小写、数字和+、/）以及用作后缀等号；

2. Base64是把3个字节变成4个可打印字符，所以Base64编码后的字符串一定能被4整除（不算用作后缀的等号）；

3. 等号一定用作后缀，且数目一定是0个、1个或2个。这是因为如果原文长度不能被3整除，Base64要在后面添加\0凑齐3n位。为了正确还原，添加了几个\0就加上几个等号。显然添加等号的数目只能是0、1或2；

4. 严格来说Base64不能算是一种加密，只能说是编码转换。



## 应用

1. Base64编码的主要的作用不在于安全性，而在于让内容能在各个网关间无错的传输，这才是Base64编码的核心作用

2. 实现简单的数据加密，使用户一眼望去完全看不出真实数据内容，base64算法的复杂程度要小，效率要高相对较高

3. 在计算机中任何数据都是按ascii码存储的，而ascii码的128～255之间的值是不可见字符。而在网络上交换数据时，比如说从A地传到B地，往往要经过多个路由设备，由于不同的设备对字符的处理方式有一些不同，这样那些不可见字符就有可能被处理错误，这是不利于传输的。所以就先把数据先做一个Base64编码，统统变成可见字符，这样出错的可能性就大降低了。

4. Base64编码可用于在HTTP环境下传递较长的标识信息。

   标准的Base64并不适合直接放在URL里传输，因为URL编码器会把标准Base64中的“/”和“+”字符变为形如“%XX”的形式，而这些“%”号在存入数据库时还需要再进行转换，因为ANSI SQL中已将“%”号用作通配符



示例：

```
var str = 'javascript';

window.btoa(str)
//转码结果 "amF2YXNjcmlwdA=="

window.atob("amF2YXNjcmlwdA==")
//解码结果 "javascript"
```

对于转码来说，Base64转码的对象只能是字符串，因此来说，对于其他数据还有这一定的局限性，在此特别需要注意的是对Unicode转码。

```
var str = "China，中国"
window.btoa(str) // Failed to execute 'btoa' on 'Window': The string to be encoded contains characters outside of the Latin1 range.

```

我们需要window.encodeURIComponent和window.decodeURIComponent来支持汉字

```
var str = "China，中国";

window.btoa(window.encodeURIComponent(str))
//"Q2hpbmElRUYlQkMlOEMlRTQlQjglQUQlRTUlOUIlQkQ="

window.decodeURIComponent(window.atob('Q2hpbmElRUYlQkMlOEMlRTQlQjglQUQlRTUlOUIlQkQ='))
//"China，中国"
```



注意：BASE64Encoder每编码76个字符后，都会在后面加上一个回车换行。所以当超过76个字符的字符串被编码后，再解码时会提示RuntimeError；如果除去编码后出现的所有换行符，如使用Base64.NO_WRAP，则字符串能正常解码。



参考文章

1. https://juejin.im/post/5bd00c0251882577787451f4
2. https://juejin.im/post/5b7d50106fb9a019d7475785



