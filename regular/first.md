## 常用正则

`.* ` 匹配任意数量不包括换行的字符

### 单个字符的重复

匹配电话号码，如：`010-12345678` 匹配它，我们可以写这样的正则 `0\d\d-\d\d\d\d\d\d\d\d` ，为了避免重复我们可以这样写这个正则 `^0\d{2}-\d{8}$` , `\d{2}` 这样的形式我们可以实现连续匹配。

匹配输入的是否是QQ号，`^\d{5, 12}$`

匹配网址，`^w{3}\.\w+\.com$`

例如上面的匹配网址，我们在开发中经常会需要对元字符进行匹配，所以，我们要经常使用转义字符 `\`.

### 字符集合

正则的语法可以让我们很轻松的查找数字，字母，空白，因为正则已经定义了查找这些字符集合的元字符，但我们要查找未定义的字符集合，我们可以使用 `[]` ，里面包含要定义的元素集合的字符即可，例如：查找英文元音字母，可以定义 `[aeiou]`。

同时，它也代表字符的范围，例如：`[0-9]` 等同于 `\d` ，`[a-zA-Z0-9_]` 等同于 `\w` .

几种格式的电话号码，如：(010)88886666，022-22334455，02912345678。我们可以使用 `\(?\d{3}[) -]?\d{8}` 来匹配

研究过像bootstrap-vue这样的模板库的同学，应该都知道该模板库可以将这些`.md` 文件中 两个 ``` 之间的内容提取出来。因为其中牵涉到换行所以 `.` 元字符不行，由于我们要找到所有的字符，一般我们使用 `/```[\s\S]```/gm` 这样的用法，但是网上还有 `/^```[^]+?^```/gm` 这样的写法。

### 多个字符的重复

正则给我们提供了分组的概念帮助我们实现多个字符的重复，在 `()` 中假如要重复的多个字符就可以了。

`(\d{3}\.){3}` 可以实现三个任意数字加一个英文句号的多字符集合重复3次

每个分组都有一个组号，顺序从左到右，组号为1，2，3 ...

### 后向引用

`\b(\w+)\b\s+\1\b` 可以用来匹配重复的单词，像go go, 或者kitty kitty

也可以自己指定子表达式的组名。要指定一个子表达式的组名，请使用这样的语法：(?<Word>\w+)(或者把尖括号换成'也行：(?'Word'\w+)),这样就把\w+的组名指定为Word了。要反向引用这个分组捕获的内容，你可以使用\k<Word>,所以上一个例子也可以写成这样：\b(?<Word>\w+)\b\s+\k<Word>\b


### 贪婪与懒惰

当正则表达式中包含能接受重复的限定符时，通常的行为是（在使整个表达式能得到匹配的前提下）匹配尽可能多的字符。以这个表达式为例：`a.*b`，它将会匹配最长的以a开始，以b结束的字符串。如果用它来搜索aabab的话，它会匹配整个字符串aabab。这被称为贪婪匹配。
有时，我们更需要懒惰匹配，也就是匹配尽可能少的字符。前面给出的限定符都可以被转化为懒惰匹配模式，只要在它后面加上一个问号 `?` 。这样`.*?`就意味着匹配任意数量的重复，但是在能使整个匹配成功的前提下使用最少的重复。现在看看懒惰版的例子吧：
`a.*?b` 匹配最短的，以a开始，以b结束的字符串。如果把它应用于aabab的话，它会匹配aab（第一到第三个字符）和ab（第四到第五个字符）


### 平衡组/递归匹配

有时我们需要匹配像( 100 * ( 50 + 15 ) )这样的可嵌套的层次性结构，这时简单地使用\(.+\)则只会匹配到最左边的左括号和最右边的右括号之间的内容(这里我们讨论的是贪婪模式，懒惰模式也有下面的问题)。假如原来的字符串里的左括号和右括号出现的次数不相等，比如( 5 / ( 3 + 2 ) ) )，那我们的匹配结果里两者的个数也不会相等。有没有办法在这样的字符串里匹配到最长的，配对的括号之间的内容呢？
为了避免(和\(把你的大脑彻底搞糊涂，我们还是用尖括号代替圆括号吧。现在我们的问题变成了如何把xx <aa <bbb> <bbb> aa> yy这样的字符串里，最长的配对的尖括号内的内容捕获出来？

这里需要用到以下的语法构造：

```
(?'group')  		把捕获的内容命名为group,并压入堆栈(Stack)
(?'-group') 		从堆栈上弹出最后压入堆栈的名为group的捕获内容，如果堆栈本来为空，则本分组的匹配失败
(?(group)yes|no)	如果堆栈上存在以名为group的捕获内容的话，继续匹配yes部分的表达式，否则继续匹配no部分
(?!)				零宽负向先行断言，由于没有后缀表达式，试图匹配总是失败
```

我们需要做的是每碰到了左括号，就在压入一个"Open",每碰到一个右括号，就弹出一个，到了最后就看看堆栈是否为空－－如果不为空那就证明左括号比右括号多，那匹配就应该失败。正则表达式引擎会进行回溯(放弃最前面或最后面的一些字符)，尽量使整个表达式得到匹配。

```
<                         #最外层的左括号
    [^<>]*                #最外层的左括号后面的不是括号的内容
    (
        (
            (?'Open'<)    #碰到了左括号，在黑板上写一个"Open"
            [^<>]*       #匹配左括号后面的不是括号的内容
        )+
        (
            (?'-Open'>)   #碰到了右括号，擦掉一个"Open"
            [^<>]*        #匹配右括号后面不是括号的内容
        )+
    )*
    (?(Open)(?!))         #在遇到最外层的右括号前面，判断黑板上还有没有没擦掉的"Open"；如果还有，则匹配失败

>                         #最外层的右括号

```

平衡组的一个最常见的应用就是匹配HTML,下面这个例子可以匹配嵌套的<div>标签：<div[^>]*>[^<>]*(((?'Open'<div[^>]*>)[^<>]*)+((?'-Open'</div>)[^<>]*)+)*(?(Open)(?!))</div>.
> 元字符

```
.	匹配除换行符以外的任意字符
\w	匹配字母或数字或下划线或汉字
\s	匹配任意的空白符
\d	匹配数字
\b	匹配单词的开始或结束
^	匹配字符串的开始
$	匹配字符串的结束
```

> 重复

```
*		重复零次或更多次
+		重复一次或更多次
?		重复零次或一次
{n}		重复n次
{n,}	重复n次或更多次
{n,m}	重复n到m次
```

> 反义

```
\W			匹配任意不是字母，数字，下划线，汉字的字符
\S			匹配任意不是空白符的字符
\D			匹配任意非数字的字符
\B			匹配不是单词开头或结束的位置
[^x]		匹配除了x以外的任意字符
[^aeiou]	匹配除了aeiou这几个字母以外的任意字符
```

> 分组语法
```
捕获		(exp)			匹配exp,并捕获文本到自动命名的组里
			(?<name>exp)	匹配exp,并捕获文本到名称为name的组里，也可以写成(?'name'exp)
			(?:exp)			匹配exp,不捕获匹配的文本，也不给此分组分配组号
零宽断言	(?=exp)			匹配exp前面的位置
			(?<=exp)		匹配exp后面的位置
			(?!exp)			匹配后面跟的不是exp的位置
			(?<!exp)		匹配前面不是exp的位置
注释		(?#comment)		这种类型的分组不对正则表达式的处理产生任何影响，用于提供注释让人阅读
```

> 懒惰限定符

```
*?		重复任意次，但尽可能少重复
+?		重复1次或更多次，但尽可能少重复
??		重复0次或1次，但尽可能少重复
{n,m}?	重复n到m次，但尽可能少重复
{n,}?	重复n次以上，但尽可能少重复
```


> 其它正则语法

```
\a	报警字符(打印它的效果是电脑嘀一声)
\b	通常是单词分界位置，但如果在字符类里使用代表退格
\t	制表符，Tab
\r	回车
\v	竖向制表符
\f	换页符
\n	换行符
\e	Escape
\0nn	ASCII代码中八进制代码为nn的字符
\xnn	ASCII代码中十六进制代码为nn的字符
\unnnn	Unicode代码中十六进制代码为nnnn的字符
\cN	ASCII控制字符。比如\cC代表Ctrl+C
\A	字符串开头(类似^，但不受处理多行选项的影响)
\Z	字符串结尾或行尾(不受处理多行选项的影响)
\z	字符串结尾(类似$，但不受处理多行选项的影响)
\G	当前搜索的开头
\p{name}	Unicode中命名为name的字符类，例如\p{IsGreek}
(?>exp)	贪婪子表达式
(?<x>-<y>exp)	平衡组
(?im-nsx:exp)	在子表达式exp中改变处理选项
(?im-nsx)	为表达式后面的部分改变处理选项
(?(exp)yes|no)	把exp当作零宽正向先行断言，如果在这个位置能匹配，使用yes作为此组的表达式；否则使用no
(?(exp)yes)	同上，只是使用空表达式作为no
(?(name)yes|no)	如果命名为name的组捕获到了内容，使用yes作为表达式；否则使用no
(?(name)yes)	同上，只是使用空表达式作为no
```