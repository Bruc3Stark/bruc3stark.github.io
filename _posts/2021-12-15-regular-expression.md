---
layout: mypost
title: 正则表达式
categories: [其他]
---

### 正则表达式

正则表达式(Regular Expression)是一种文本模式，包括普通字符（例如，a 到 z 之间的字母）和特殊字符（称为"元字符"）。

正则表达式使用单个字符串来描述、匹配一系列匹配某个句法规则的字符串。

### 常用规则

| 字符           | 描述  |
|:-------------:| :-----|
|\d|匹配一个数字字符。等价于 [0-9]|
|\D|匹配一个非数字字符。等价于 [^0-9]|
|\w|匹配包括下划线的任何单词字符。等价于[A-Za-z0-9_]|
|\W|匹配任何非单词字符。等价于 '[^A-Za-z0-9_]'|
|[xyz]|字符集合，匹配所包含的任意一个字符|
|`x|y`|匹配 x 或 y，xy为多个字符，建议使用`(ab|cd)`的形式来区分边界|
|[^xyz]|字符取反集合,匹配未包含的任意字符|
|[a-z]|字符范围。匹配指定范围内的任意字符|
|.|匹配除 "\n" 之外的任何单个字符。要匹配包括 '\n' 在内的任何字符可以用"(.|\n)"|

### 限定符

限定符用来指定正则表达式的一个给定组件必须要出现多少次才能满足匹配

| 字符           | 描述  |
|:-------------:| :-----|
|`*`|匹配前面的子表达式零次或多次，等价于{0,}|
|`+`|匹配前面的子表达式一次或多次，等价于 {1,}|
|?|匹配前面的子表达式零次或一次，等价于 {0,1}|
|{n}|n是一个非负整数。匹配确定的n次|
|{n,}|n是一个非负整数。至少匹配n次|
|{n,m}|m,n为非负整数n <= m,最n-m次,在逗号和两个数之间不能有空格|
|()|把括号内的作为一个整体，比如(ab){2},匹配abab|

### 定位符

定位符用来描述字符串或单词的边界

| 字符           | 描述  |
|:-------------:| :-----|
|^|匹配输入字符串开始的位置。如果设置了RegExp对象的 Multiline 属性，还会与 \n 或 \r 之后的位置匹配|
|$|匹配输入字符串结尾的位置。如果设置了 RegExp 对象的 Multiline 属性，$ 还会与 \n 或 \r 之前的位置匹配|
|\b|匹配一个字边界，即字与空格间的位置|
|\B|非字边界匹配|

### 非打印字符

非打印字符也可以是正则表达式的组成部分。下表列出了表示非打印字符的转义序列

| 字符           | 描述  |
|:-------------:| :-----|
|\cx|匹配由x指明的控制字符。例如， \cM 匹配一个 Control-M 或回车符。x 的值必须为 A-Z 或 a-z 之一。否则，将 c 视为一个原义的 'c' 字符|
|\f|匹配一个换页符。等价于 \x0c 和 \cL|
|\n|匹配一个换行符。等价于 \x0a 和 \cJ|
|\r|匹配一个回车符。等价于 \x0d 和 \cM|
|\s|匹配任何空白字符，包括空格、制表符、换页符等等。等价于 [ \f\n\r\t\v]|
|\S|匹配任何非空白字符。等价于 [^ \f\n\r\t\v]|
|\t|匹配一个制表符。等价于 \x09 和 \cI|
|\v|匹配一个垂直制表符。等价于 \x0b 和 \cK|

### 图解

盗的一张图

![regexp](regexp.png)

### 贪婪非贪婪

*、+和?限定符都是贪婪的，因为它们会尽可能多的匹配文字

默认是贪婪模式匹配,在量词后面直接加上一个问号？就是非贪婪模式。

比如有一段文字`<p>6666</p>`

如果使用`<.+>`是默认的贪婪匹配，会匹配到`<p>6666</p>`

而使用`<.+?>`就变成了非贪婪匹配，会匹配到`<p>`和`</p>`

### 在Java中使用正则

在Java 正则表达式很方便

主要使用java.util.regex包下的几个类

### Pattern

Pattern对象是一个正则表达式的编译表示。Pattern类没有构造方法。要创建一个 Pattern 对象，你必须首先调用其公共静态的compile编译方法要么传入一个正则表达式或者传入一个表达式和规则，比如不区分大小写....

```
Pattern pattern = Pattern.compile("<.+?>");
Pattern pattern = Pattern.compile("<.+?>",Pattern.CASE_INSENSITIVE);
```

### Matcher

Matcher对象是对输入字符串进行解释和匹配操作的引擎。与Pattern 类一样，Matcher 也没有公共构造方法。你需要调用 Pattern 对象的 matcher 方法来获得一个 Matcher 对象。

matches：整个匹配，只有整个字符序列完全匹配成功，才返回True，否则返回False。但如果前部分匹配成功，将移动下次匹配的位置

lookingAt：部分匹配，总是从第一个字符进行匹配,匹配成功了不再继续匹配，匹配失败了,也不继续匹配

find：部分匹配，从当前位置开始匹配，找到一个匹配的子串，将移动下次匹配的位置

groupCount：返回正则表达式中组的个数，和要匹配的字符串无关

group:以正则表达是中的某个组来匹配，find之后才可以使用

start,end:find之后返回为真后才可以用，返回的是匹配到的开始和结束，可用group来取出结果

reset:清除find后保存的开始结束位置

```
Pattern pattern = Pattern.compile("<.+?>");

Matcher matcher = pattern.matcher("<p>6666</p>123");

// 多个匹配位置
while(matcher.find()){
    //当前匹配位置的组matcher.group()
    System.out.println(matcher.start()+","+matcher.end()+matcher.group());
}
```

```
Pattern pattern = Pattern.compile("(iPhone; CPU|Android.*;)\\s(.*?)\\s(Build/|OS)");

Matcher matcher = pattern.matcher("Mozilla/5.0 (Linux; Android 6.0.1; Redmi 4 Build/MMB29M; wv) AppleWebKit/537.36 (KHTML, like Gecko) Version/4.0 Chrome/57.0.2987.132 Mobile Safari/537.36");

if (matcher.find()) {
    System.out.println(matcher.group(1));
}
```

### 其他

在字符串上可以直接使用正则表达式

```
String pat = "<.+?>";

System.out.println("123<p>6666</p>123".matches(pat));//整串匹配，false

for(String str:"123<p>6666</p>123".split(pat)){
    System.out.println(str);
}

false
123
6666
123
```

### 在JavaScript中使用正则

### 创建

正则表达式实例的的创建方式有两种，字面量创建方式和实例创建方式

注意的是实例创建方式传出的pattern是个字符串，要对\做转义` /\d/ = > "\\d" `

```
// 字面量创建方式
var reg = /pattern/flags
// 实例创建方式
var reg = new RegExp(pattern,flags);

pattern:正则表达式
flags:标识(修饰符)
标识主要包括：
1. i 忽略大小写匹配
2. m 多行匹配，即在到达一行文本末尾时还会继续寻常下一行中是否与正则匹配的项
3. g 全局匹配 模式应用于所有字符串，而非在找到第一个匹配项时停止
```

### 常用方法

正则实例上的方法：test、exec

字符串上的方法：match、replace、search、split

```
patt.test(str)//测试字符串是否符合模式

str.split(patt)//分割字符串

分组

var re = /^(\d{3})-(\d{3,8})$/;
re.exec('010-12345'); // ['010-12345', '010', '12345']
re.exec('010 12345'); // null

var s = 'JavaScript, VBScript, JScript and ECMAScript';
var re=/[a-zA-Z]+Script/g;

// 使用全局匹配:
re.exec(s); // ['JavaScript']
re.lastIndex; // 10

re.exec(s); // ['VBScript']
re.lastIndex; // 20

re.exec(s); // ['JScript']
re.lastIndex; // 29

re.exec(s); // ['ECMAScript']
re.lastIndex; // 44

re.exec(s); // null，直到结束仍没有匹配到

str.match(pat)//返回数组，数组的内容依赖于 regexp 是否具有全局标志g，否则是第一个匹配的

"<p>666</p>"
s.match(/<.+?>/);
["<p>"]
s.match(/<.+?>/g);
["<p>", "</p>"]
```

### 参考

[正则表达式 - 教程](http://www.runoob.com/regexp/regexp-tutorial.html)

[廖雪峰的官方网站](https://www.liaoxuefeng.com/wiki/001434446689867b27157e896e74d51a89c25cc8b43bdb3000/001434499503920bb7b42ff6627420da2ceae4babf6c4f2000)

[chenermeng/blog](https://github.com/chenermeng/blog)
