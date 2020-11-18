---
layout: post
title: λ 演算（Lambda calculus）
tags: 
- 编程 / Programming
---

> If you have the name of the spirit, you get control over it.
> 
> – Harold Abelson[^1]
> {:align="right"}

[^1]: [SICP, Lecture 2B: Compound Data (1984)](https://ocw.mit.edu/courses/electrical-engineering-and-computer-science/6-001-structure-and-interpretation-of-computer-programs-spring-2005/video-lectures/2b-compound-data/#vid_playlist)

<!-- more -->

## 0. 起因

如果使用过任意一门稍微「现代」一点的编程语言，你一定不会对“匿名函数”（lambda expression）感到陌生。比如在 JavaScript 里，你可以这样定义一个函数，然后进行求值：

```js
(function(x){return x + 1;})(3) // 4
```

而在 2015 年制定的 ES6 标准里，又添加了一种 [更简便的语法](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions)：

```js
((x) => (x + 1))(3) // 4
```

然而，除了少数情况外，匿名函数似乎没什么用，Python 的创始人 Guido van Rossum 甚至一度想移除 lambda，最后 [以失败告终](https://mail.python.org/pipermail/python-dev/2006-February/060415.html)。

那么 λ 真是「食之无味，弃之可惜」吗？曾经我也这么认为，直到最近看了 SICP……😂

### TL:DR

所有别的编程语言能做的事，只用 λ 也可以做到，包括但不限于：

- 基本的数学计算
- 赋值、逻辑判断
- 循环、递归、死循环
- 实现各种数据结构
- 实现求值器、编译器、~~求值器上的求值器~~……

~~λ 就是世界的主宰！~~

## 1. 基础

要继续探索的旅程，首先回顾一下函数的定义。

一个函数是指一个集合到另一个集合的对应关系，比如上面的函数 λ := (x) => (x + 1)，可以用表格来表示：

|x|λ(x)|
|:-:|:-:|
|0| 1 |
|1| 2 |
|2| 3 |
|3| 4 |
|...|...|

当函数被调用时（比如上面的 λ(3)），λ 会查询这个表，找到 3 右边对应的数 4，并返回这个数[^2]。

我们希望 λ 函数能满足下面的性质：

### 1.1 闭包（Closure）

这里闭包的定义是~~套娃~~，λ 函数是自封闭的，λ 函数既可以作为 λ 函数的输入（参数），又可以作为 λ 函数的输出（返回值），也可以两者同时出现。

闭包的一个著名应用是柯里化（Currying）。如果一个函数需要同时输入多个参数，比如加法：

```js
add1 = (x, y) => (x + y)
```

然而（传统的）λ 函数只能接受一个参数的话，就可以用这个技巧拆开参数：

```js
add2 = (x) => ((y) => (x + y))
```

再依次传入即可计算：

```js
add1(4, 3) // 7
add2(4)(3) // 7
```

这里的 add2 函数以一个数作为输入，返回一个 λ 函数。比如上面执行了 add2(4)，把 4 带入定义，会返回 λ := (y) => (4 + y)，再输入 (3) 就有 λ(3) = 4 + 3 = 7，输出结果 7。

以下均假设 λ 函数可以接受多个参数。

### 1.2 静态作用域（Lexical scope）

我们从一个最简单的函数开始：

```js
I = (x) => (x)
```

这个函数会把输入原样输出。现在来定义一个输出这个函数的函数：

```js
J = (x) => I
```

按照代换的想法，我们用 I 的定义替换掉 I：

```js
J = (x) => ((x) => (x))
```

那么这两个 J 相等吗？具体而言，对任一给定的求值过程（比如 J(1)(4)），得到的结果一致吗？

也许你已经意识到了问题，我们希望这两个结果是一致的（都是 4），可是回顾 Currying 的过程，返回的 λ 函数里 x 的值又不可避免会被 J 影响到，这两个函数不可能完全独立……

我们自然可以用别的变量名来代替 J 函数的输入 x，比如 y：

```js
J = (y) => ((x) => (x))
```

可是写 J 函数的时候，我们不一定知道 I 函数的输入变量是 x，更不可能知道 I 函数里调用的其他函数所使用的所有输入变量名，再小心翼翼地避免变量名冲突……

为了解决这一矛盾，静态作用域诞生了，大致而言：

- 外层的函数使用的参数，会形成一个“初值表”传给内层的函数作为函数的执行环境；
- 如果内层的函数直接使用（自由变量，free variable），则直接进行代换；
- 如果内层的函数修改了这些值（约束变量，bound variable），会覆盖（shadow）该函数的环境变量再进行代换，不会影响外层函数对应变量的值。内层的函数执行结束回到外层函数时该变量的值恢复。

现在再回头看 J(1)(4) 的执行过程：

1. J(1) 返回 λ 函数 (x) => (x)，同时设置（或修改）环境变量 x = 1；
2. λ(4) 覆盖掉环境里 x 的值，环境变量 x 被修改为 4；
3. 计算出 λ(4) 的输出为 4，返回父层函数 J，环境恢复 x = 1；
4. J 函数执行结束，返回父层函数，环境再次恢复。

可以看到，在引入静态作用域后，无论以哪种方式对 λ 函数进行求值（从外到内、从内到外），得到的结果都是一致的，这就是 [Church–Rosser 定理](https://en.wikipedia.org/wiki/Church%E2%80%93Rosser_theorem)，证明可以参考 [这里](http://pages.cs.wisc.edu/~horwitz/CS704-NOTES/1.LAMBDA-CALCULUS.html)。

## 2. 数据结构

在 SICP 里，作者反复强调“数据和程序（procedure）没有任何区别”。“程序即数据”这点很好理解，毕竟多厉害的程序，本质都是一串二进制数据，可是“数据即程序”该怎么理解呢？

在 λ 演算里，我们没有定义任何的数据结构，能用的只有 λ 函数。如何用 λ 函数来表示数据呢？

> If it looks like a duck, swims like a duck, and quacks like a duck, then it probably is a duck.

回顾数据结构的使用，我们并不清楚数据结构的内部是如何实现的，只要调用对应的函数能返回所需要的值就可以。换言之，数据结构对我们而言，其实是一个黑箱（black box）。只要能用 λ 函数实现相同的操作，我们就可以认为构造了这一数据结构，这就是「公理化方法」。

### 2.1 序对（Pair）与列表（List）

我们从最简单的数据结构——~~贴贴~~序对开始。序对是指一个有序二元组 (a, b)，可调用以下函数[^3]：

- 构造：pair(a, b) 返回生成的序对 p
- 前部：head(p) 返回序对 p 在前的元素 a
- 后部：tail(p) 返回序对 p 在后的元素 b

怎么实现序对呢？方法并不惟一，这里是一种比较容易理解的方法。

我们先从前部和后部开始。如果不需要序对的话，head 和 tail 很容易实现：

```js
head = (x, y) => x
tail = (x, y) => y
```

然而现在 head 接受的参数为序对 p，容易想到，只要改写为：

```js
head = (p) => p((x, y) => x)
tail = (p) => p((x, y) => y)
```

那么对应的 pair 又如何实现呢？也不难推出：

```js
pair = (x, y) => ((f) => f(x, y))
```

就这样，我们「无中生有」定义出了序对，不妨打开 JS 控制台试试；

```js
p = pair(1, 2); // f => f(1, 2)
head(p) // 1
tail(p) // 2
```

更有趣的是，由于 λ 函数的闭包性质，序对也是闭包，因此元素 a 与 b 也可以是序对。在 LISP 里，列表就是使用序对定义的，前部为元素，后部为下一个序对，比如列表 (a, b, c)：

| 序对 | 前部 | 后部 |
| --- | --- | --- |
| p1 | a | p2 |
| p2 | b | p3 |
| p3 | c | nil |

只要定义了 nil，我们就可以实现列表了，方法自然很多，此处从略。

### 2.2 布尔型（Boolean）

我们来尝试定义一下“对与错”，有了对错，我们就可以实现程序里常见的 if 条件判断了。和上面类似，我们不知道对错是怎么实现的，但是可以调用三个函数：

| a | b | AND(a, b) | OR(a, b) | NOT(a) |
| - | - | --------- | -------- | ------ |
| TRUE | TRUE | TRUE | TRUE | FALSE |
| TRUE | FALSE | FALSE | TRUE | FALSE |
| FALSE | TRUE | FALSE | TRUE | TRUE |
| FALSE | FALSE | FALSE | FALSE | TRUE |

好像还是没有头绪，那还是从 if 开始吧！我们来定义函数 IFTHENELSE(cond, then, else):

- 如果 cond == TRUE，那么返回 then；
- 如果 cond == FALSE，那么返回 else。

回想起上面使用的技巧，很容易写出：

```js
IFTHENELSE = (a, b, c) => a(b, c)
```

于是 TRUE 和 FALSE 也立刻能写出来：

```js
TRUE = (x, y) => x
FALSE = (x, y) => y
```

最后再用 IFTHENELSE 来定义 AND, OR, NOT[^4]：

```js
AND = (x, y) => IFTHENELSE(x, y, FALSE)
OR = (x, y) => IFTHENELSE(x, TRUE, y)
NOT = (x) => IFTHENELSE(x, FALSE, TRUE)
```

虽然有点怪怪的，但我们确实成功定义了布尔型，不妨打开控制台试试：

```js
NOT(TRUE) // (x, y) => y
AND(TRUE, TRUE) // (x, y) => x
OR(FALSE, FALSE) // (x, y) => y
```

Well done!🎉

### 2.3 丘奇数（Church numeral）

现在我们来定义自然数！数学里的自然数定义方法不止一种，但原理相似，从 0 开始一个一个向后推，并且确保不会出现重复：

- 1 = next(0)
- 2 = next(1) = next(next(0))
- 3 = next(2) = next(next(next(0)))
- ...

这里我们使用 λ 演算的作者丘奇的定义：

```js
zero = (f) => ((s) => s)
one = (f) => ((s) => f(s))
two = (f) => ((s) => f(f(s)))
three = (f) => ((s) => f(f(f(s))))
...
```

那么如何定义 next 呢？我们先来找找规律：

```js
zero(f)(s) = s
one(f)(s) = f(s)
two(f)(s) = f(f(s))
...
(next(n))(f)(s) = f(n(f)(s))
```

因此：

```js
next = (n) => (f) => (s) => f((n(f))(s))
```

到这里浏览器控制台已经帮不了我们了，只好手工计算：

```js
next(zero)
== (f) => (s) => f((zero(f))(s))
== (f) => (s) => f((((f) => ((s) => s))(f))(s))
== (f) => (s) => f(((s) => s)(s))
== (f) => (s) => f(s)
== one
```

当然，也许你已经发现，只要给 Church numeral 带入自增函数 (x) => (x + 1) 和起点 0，就能得到对应的自然数：

```js
one((x) => (x + 1))(0) // 1
next(zero)((x) => (x + 1))(0) // 1
```

这样，我们成功推导出了丘奇数的后继（successor）。有了 next，很自然想到，有没有 prev 呢？Church 曾经找了很久也没找到，怀疑可能不存在，后来被他的学生 Kleene 构造了出来（据说花了三个月🥶）。

Kleene 使用的方法是用相邻的两个数构造一个 pair，再定义 pair 的后继 TRANS：

```js
TRANS
== pair(prevn, n) => pair(n, next(n))
== (p) => pair(tail(p), next(tail(p)))
```

为了保持封闭性，我们设 prev(0) = 0。要求 prev(N)，我们首先创建一个 pair(0, 0)，然后进行 N 次 TRANS 操作，得到 pair(N - 1, N)，最后再使用 head 即可：

```js
TRANS = (p) => pair(tail(p), next(tail(p)))
prev = (n) => head(n(TRANS)(pair(zero, zero))))
```

同样，我们来手工验算一下：

```js
prev(one)
== head(one(TRANS)(pair(zero, zero))))
== head(((f) => (s) => f(s))(TRANS)(pair(zero, zero)))
== head(((s) => TRANS(s))(pair(zero, zero)))
== head(pair(zero, one))
== zero
```

除了 Kleene 的序对，~~知名自媒体（误）~~[王垠的博客](https://www.yinwang.org/blog-cn/2012/07/04/dan-friedman) 提供了另一种更简洁巧妙的构造方法[^5]，大致的想法是这样的（看不懂可以跳过）：

只使用 λ 我们很难从 f(f(s)) 反推回 f(s)，那么能不能找到一个 t，使得 f(t) = s 呢？这样的 t 也很难找😂但如果换过来变成 t_(f) = s，那还是很简单的：

```js
t_ = (w) => (s)
```

我们希望可以定义一个“箱子”，把具体的值封装起来📦，并在输入一个函数的时候，返回箱子里的数被函数作用的结果，即满足下面的定义：

```js
// b = (w) => (s)
// b(f) = s
b0 = box(s)
b0(f) = box(s)(f) == f(s)
b1 = box(f(s))
b1(f) = box(f(s))(f) == f(f(s))
...
```

这样从 (w) => (s) 开始重复 N 次 (b) => box(b(f))，就能得到 box(prev(N)(f)(s))，最后再拆开箱子即可。不难看出：

```js
box = (s) => (g) => g(s)
```

接下来我们定义箱子的自增函数 inc = (b) => box(b(f))，也不难得出：

```js
inc = (b) => ((s) => (g) => g(s))(b(f))
== (b) => (g) => g(b(f))
```

最后是拆开箱子 box(prev(N)(f)(x))：

```js
box(x)(I) = I(x) == x // I = (x) => (x)
```

组合起来就得到了 prev(N)：

```js
prev // = (n) => (f) => (s) => n(inc)((w) => (s))(I) // I = (x) => (x)
= (n) => (f) => (s) => n((b) => ((g) => g(b(f))))((w) => (s))((x) => (x))
```

同样可以使用控制台来验证：

```js
prev(zero)((x) => (x + 1))(0) // 0
prev(one)((x) => (x + 1))(0) // 0
prev(two)((x) => (x + 1))(0) // 1
```

~~呼，终于结束了😂~~

### 2.4 运算与判断

现在我们来定义四则运算！首先是加法 add(m, n)，显然有：

```js
add = (m, n) => (n(next)(m))
// add(three, two)((x)=>(x + 1))(0) // 5
```

减法也是一样：

```js
minus = (m, n) => (n(prev)(m))
// minus(three, two)((x)=>(x + 1))(0) // 1
```

乘法和加法类似：

```js
// mul(m, n) = add(add(..., m), m)
mul = (m, n) => n((x) => add(x, m))(zero)
// mul(three, two)((x)=>(x + 1))(0) // 6
```

只要把 add 换成 mul，zero 换成 one 就是求幂：

```js
pow = (m, n) => n((x) => mul(x, m))(one)
// pow(three, two)((x)=>(x + 1))(0) // 9
```

除法（整除）和求余需要先定义大小关系，因此要先定义 iszero(n)，n=zero 时返回 TRUE，n 为其他丘奇数时返回 FALSE。也许你留意到 zero 和 FALSE 很相似，尝试定义 (z) => IFTHENELSE(g(z), FALSE, TRUE)。然而，虽然 iszero(zero) 会返回 TRUE，但 n 为其他丘奇数的时候，会没有结果🥶

首先我们留意到丘奇数需要两个参数，iszero 需要一个丘奇数，如果 iszero 有最简单的形式，一定会是这样：

```js
iszero = (z) => z(...)(...)
```

而回顾 zero 的定义：zero = (f) => (s) => (s)，也就是说：

```js
iszero = (z) => z(...)(TRUE)
```

最后我们希望 ...(TRUE) 会返回 FALSE，...(FALSE) 也会返回 FALSE（为什么？），方法自然有很多，比如 AND(FALSE, bool) 永远会返回 FALSE，但还有更简单的方法：

```js
iszero = (z) => z((x) => FALSE)(TRUE)
```

有了 iszero，剩下的判断全都解决了：

```js
leq = (x, y) => iszero(minus(x, y)) // Less than or equal to
geq = (x, y) => iszero(minus(y, x)) // Greater than or equal to
lt = (x, y) => NOT(leq(y, x)) // Less than
gt = (x, y) => NOT(leq(x, y)) // Greater than
eq = (x, y) => AND(leq(x, y), leq(y, x)) // Equal to
neq = (x, y) => NOT(eq(x, y)) // Not wqual to
```

我们现在可以大致写出整除和求余的伪代码了：

```js
div = (x, y) => IFTHENELSE(geq(x, y), 1 + div(x - y, y), 0)
mod = (x, y) => IFTHENELSE(geq(x, y), mod(x - y, y), x)
```

然而 λ 函数是匿名函数，递归好像不可能实现？

## 3. 递归

这大概是 λ 演算里最有趣，也最难以置信的部分：没有名字的函数，依然可以实现递归，进而实现“死循环”！因此 λ 演算这门只有 λ 的编程语言，是图灵完备的；所有其他编程语言可以实现的程序，只用 λ 也可以实现[^6]。

我们以阶乘函数 n! 为例，很容易写出递归版：

```js
fact = (x) => (x == 1)? 1 : x * fact(x - 1)
fact(6) // 720
```

而由于 λ 函数没有名字，我们还需要把这个函数包起来，传入一个 fact：

```js
f = (fact) => (x) => (x == 1)? 1 : x * fact(x - 1)
```

最后我们需要把 fact 作为参数传入求值：

```js
((fact) => (x) => (x == 1)? 1 : x * fact(x - 1))((x) => (x == 1)? 1 : x * fact(x - 1))
```

就实现了递归！打开 JS 控制台试试：

```js
((fact) => (x) => (x == 1)? 1 : x * fact(x - 1))((x) => (x == 1)? 1 : x * fact(x - 1))(6) // 720
```

那么怎么用 λ 函数来表示这个变换呢？

留意到 f(fact) = fact，fact 是我们要求的函数，而 f 是我们已知的。也就是说，给定了 f，如何寻找 f 的一个点 x，使得 f(x) = x？更进一步，如果能找到一个通用的 y 使得 x = y(f)，递归起来就很方便了～

符合条件的 y [不止一个](https://en.wikipedia.org/wiki/Fixed-point_combinator)，不过最著名的，还是 Curry 发明的 Y Combinator：

```js
y = (f) => ((x) => f(x(x)))((x) => f(x(x)))
```

虽然看着很奇怪，但可以验算一下，代入任意的函数 f：

```js
y(f)
== ((x) => f(x(x)))((x) => f(x(x)))
== f((x) => f(x(x)))((x) => f(x(x)))
== f(y(f))
== f(f(...f(y(f))))
```

这样只要定义出函数 f，就可以实现递归。不过注意原版的 Y Combinator 不能直接应用到惰性求值的 JavaScript 上（会变成死循环），需要进行变形[^7]:

```js
y = (f) => ((x) => f((v) => (x(x))(v)))((x) => f((v) => (x(x))(v)));
y((fact) => (x) => (x == 1)? 1 : x * fact(x - 1))(6) // 720
```

于是整除和求余也完成了～🎉

### 3.1 附：Y Combinator 的简单推导[^7]

大意是从 f(f) 变换出 f，再把 f 作为参数提取出来。还是以阶乘函数为例：

```js
f = (fact) => (x) => (x == 1)? 1 : x * fact(x - 1)
```

我们希望直接 f(fact)，但是手上只有 f，于是尝试 f(f)：

```js
((fact) => (x) => (x == 1)? 1 : x * fact(x - 1))((fact) => (x) => (x == 1)? 1 : x * fact(x - 1))
```

但是要使 f(f)(x) 可以求值，还需要把 fact 修改为 fact(fact)（因为 f 需要先接受一个函数作为参数）：

```js
((fact) => (x) => (x == 1)? 1 : x * fact(fact)(x - 1))((fact) => (x) => (x == 1)? 1 : x * fact(fact)(x - 1))
```

简化：

```js
((u) => u(u))((fact) => (x) => (x == 1)? 1 : x * fact(fact)(x - 1))
```

继续简化，把 fact(fact) 替换成 g：

```js
((u) => u(u))((fact) => ((g) => (x) => (x == 1)? 1 : x * g(x - 1))(fact(fact)))
```

注意到 (g) => (x) => (x == 1)? 1 : x * g(x - 1) 就是 f，作为参数提取出来，就得到了 Y Combinator：

```js
(f) => ((u) => u(u))((fact) => f(fact(fact)))
```

认不出来？把右边函数代入 u，并用 x 替换掉 fact，就得到了熟悉的形式：

```js
(f) => ((x) => f(x(x)))((x) => f(x(x)))
```

完结撒花🎉🎉🎉

## 4. 今回はここまで

{% include img.html url="/pic/end.jpg" alt="到这里就结束啦～" %}

> I think that it's extraordinarily important that we in computer science keep fun in computing... I hope the field of computer science never loses its sense of fun... What's in your hands, I think and hope, is intelligence: the ability to see the machine as more than when you were first led up to it, that you can make it more.
> 
> – Alan J. Perlis (April 1, 1922-February 7, 1990)
> {:align="right"}

### 4.1 起因

写这篇文章纯属偶然，无聊时点开了 SICP 的视频，然后就入魔了～~~后来被第四章劝退了😂~~

魔法如此奇妙，30 多年前的课程录像，现在来看竟然还不过时，好多困扰很久的问题被两位作者用最简单的方式解释的一清二楚。久违地有了记笔记的冲动，结果这篇笔记也没记录下多少东西。很多有趣的内容，比如列表的操作、map、实现 OOP，本来想加进来的，结果有心无力……

因为 λ-calculus 的资料很多很多[^8]，本以为会写得很轻松，但把最基本的内容复述一遍，花费的时间却远超预料，刚接触时的兴奋感也消失殆尽，~~大概是届不到了～~~

### 4.2 接下来可以看什么？

- SICP！SICP！SICP！不过 SICP 还是太难了🥶
  - 小提示：先看视频再看书，会比只看书看得更快～
  - 除了 MIT 外，还有 UC Berkeley 的 [Python 版](https://cs61a.org/) 和国立新加坡大学的 [JavaScript 版](https://sicp.comp.nus.edu.sg/)。
  - 习题虽然很多，但都能找到答案。
- 王垠推荐了好多有趣的书，比如：
  - Daniel P. Friedman 的一系列“小人书”，如 The Little Schemer，还有 Essentials of Programming Languages。
  - Matthias Felleisen 和 Matthew Flatt 的 [Programming Languages and Lambda Calculi](https://www.cs.utah.edu/~mflatt/past-courses/cs7520/public_html/s06/notes.pdf)。
  - Neil Jones 的 [Computability and Complexity](http://www.diku.dk/~neil/Comp2book.html)。
- Bonus: [可以用 λ 函数实现数组（Array）吗？](https://surface.syr.edu/cgi/viewcontent.cgi?article=1080&context=eecs_techreports)

~~最后，小心秃头😂~~

[^2]: 之后这个表格可能会更新。如果表格保持不变，则称 λ 为纯函数（pure function）。

[^3]: 在 LISP 里由于历史原因，这三个函数分别叫 CONS, CAR, CDR。

[^4]: NOT 有两个版本，对应不同的求值策略。这里的是正常次序（normal order）版，只适用于惰性求值（lazy evaluation）的语言，比如 JavaScript。另一个版本是 NOT = (p) => (a, b) => p(b, a)，只适用于按应用次序（applicative order）求值的语言，比如 Scheme。

[^5]: 感谢 [维基百科](https://en.wikipedia.org/wiki/Church_encoding#Derivation_of_predecessor_function) 详细的解释。

[^6]: 虽说如此，但纯 λ 实现的程序，有人能看懂吗……可以访问 [这个页面](http://www.jsfuck.com/) 感受一下😂

[^7]: 感谢 [王垠的 Y Combinator 推导 PPT](https://www.slideshare.net/yinwang0/reinventing-the-ycombinator)！这个简洁的推导源自 The Little Schemer, Chap. 9。

[^8]: Google 找到了好多 20 年前的页面，感觉像在考古……这里是一个 [传送门](http://www.cs.unc.edu/~stotts/723/Lambda/scheme.html) 。
