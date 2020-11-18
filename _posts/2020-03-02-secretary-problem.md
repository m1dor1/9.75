---
layout: post
title: 秘书问题（Secretary problem）
tags: 
- "算法 / Algorithm"
---

> Life can only be understood going backwards, but it must be lived going forwards.
> 
> – Søren Aabye Kierkegaard (1813-1855)
> {:align="right"}

<!-- more -->

## 0. 一碗鸡汤

> 有一天，柏拉图问老师苏格拉底：什么是爱情？
> 
> 苏格拉底就让他先到麦田去，摘一颗全麦田里最大最金黄的麦穗来。期间只能摘一次，并且只能向前走不能回头。
> 
> 柏拉图按照老师说的去做了，结果两手空空的走出了田地。老师问他为什么摘不到？他说：因为只能摘一次，又不能走回头路，期间即便看到又大又金黄的麦穗，因为不知道前面是否还会有更好的，所以就没有摘。
> 
> 继续往前走时，又发觉前面的麦穗总是不如之前见到的好，原来最大最金黄的麦穗早就已经错过了，所以我什么都没有摘。
> 
> 这时苏格拉底说：这就是“爱情”。
> 
> —— 佚名
> {:align="right"}

## 1. 显而易见的策略

柏拉图要使用什么策略才能让自己选择到最好麦穗的概率最大化呢？一个显而易见的策略是把前面 k 个麦穗作为“训练集”，一旦出现比这 k 个麦穗都好的麦穗，就作出选择。

我们不妨来计算一下这一策略选择到最好麦穗的概率。事件「第一个比前 k 个都好的麦穗恰好为 N 个中最好的麦穗」好像并不好处理，但它可以分解为「最好的麦穗不在前 k 个里」和「前 k 个麦穗里最好的麦穗，恰好是最好的麦穗前面所有麦穗里最好的麦穗」这两个事件的交。假设最好的麦穗在第 i 个，则 i <= k 时，取得最好麦穗的概率为 0；i > k 时，概率自然就是 k/(i - 1)。[^1]于是：

<p align="center"><img src="https://m1dor1.github.io/9.75/svg/-2358441750626973461.svg" alt="P (k) = \frac{1}{N} \left( \sum^k_{i = 1} 0 + \sum^N_{i = k + 1} \frac{k}{i - 1} \right) = \frac{k}{N} \left( \frac{1}{k} + \frac{1}{k + 1} + \cdots + \frac{1}{N - 1} \right)"></p>

## 2. 朴实无华的算法

很自然会想，如何使 P(k) 最大化呢？由于 k 只能取 1 到 N，只要一个一个取，选择使 P(k) 最大的那个 k 就好了。更进一步，注意到：

<p align="center"><img src="https://m1dor1.github.io/9.75/svg/-516020830953166857.svg" alt="\begin{array}{ccl}
  P (k - 1) &amp; = &amp; \dfrac{k - 1}{N} \left( \dfrac{1}{k - 1} + \dfrac{1}{k} + \cdots + \dfrac{1}{N - 1} \right)\\
  &amp; = &amp; \dfrac{1}{N} + \dfrac{k - 1}{N} \left( \dfrac{1}{k} + \dfrac{1}{k + 1} + \cdots + \dfrac{1}{N - 1} \right)
\end{array}"></p>

从而：

<p align="center"><img src="https://m1dor1.github.io/9.75/svg/1552475870197919153.svg" alt="\nabla P = P (k) - P (k - 1) = \frac{1}{N} \left( \frac{1}{k} + \frac{1}{k + 1} + \cdots + \frac{1}{N - 1} - 1 \right)"></p>

因此 ∇P 是单调递减的，故 P(k) 一定会在某个 k 值达到最大值，随后一直下降[^2]。换而言之，我们希望找到能满足条件 ∇P >= 0 的最大的 k：

<p align="center"><img src="https://m1dor1.github.io/9.75/svg/-5165671164038390625.svg" alt="P (k) - P (k - 1) \geqslant 0 \Rightarrow \frac{1}{k} + \frac{1}{k + 1} + \cdots + \frac{1}{N - 1} \geqslant 1"></p>

因此对给定的 N，只要从 1/(N-1) 倒过来开始加，总和刚好大于等于 1 时，就得到所求的 k。这个算法被称为 [Odds algorithm](https://en.wikipedia.org/wiki/Odds_algorithm)，使用 Python 实现只要几行：

```py
def stop_time(N):
    if N <= 1:
        return 0
    S = 0
    while S < 1:
        S = S + 1 / (N - 1)
        N = N - 1
    return N
# [stop_time(i) for i in range(1, 21)]
```

更进一步，注意到 P(k) = S * k / N，简单改写一下就可以同时输出 k 和 P(k)：

```py
def stop_time(N):
    S, k = 0, N
    if N <= 1:
        return (0, 1)
    while S < 1:
        S = S + 1 / (k - 1)
        k = k - 1
    return (k, S * k / N)
# [stop_time(i) for i in range(1, 21)]
```

可以看到 P(k) 随着 N 的增长一直下降，并且 N 趋于正无穷时 P(k) 似乎趋近 0.37，而且 k / N 也在 0.37 附近上下波动，这是巧合吗？

## 3. 无聊的分析

回顾微积分知识，我们可以使用积分近似代替调和级数的求和[^3]：

<p align="center"><img src="https://m1dor1.github.io/9.75/svg/1320412632942422181.svg" alt="\frac{1}{k} + \frac{1}{k + 1} + \cdots + \frac{1}{N - 1} &gt; \int_{n = k}^N
\frac{1}{n} = \ln \frac{N}{k}"></p>

直觉告诉我们，N 很大的时候，两者会非常接近，因此 ln N/k 会在 1 附近，故 k/N 会在 1/e = 0.36787944117144233... 附近上下波动，从而 P(k) = S * k / N = 1 * 1/e = 1/e。但是直觉毕竟只是直觉～

首先我们来证明，N <- N + 1 时使 P(k, N + 1) 最大的 k 不是 k 就是 k + 1。使用反证法，S >= 1 => S + 1/N > 1，因此不可能是比 k 小的数；而如果是 k + 2，根据上面的推论显然有：

<p align="center"><img src="https://m1dor1.github.io/9.75/svg/1735573652853424295.svg" alt="\begin{array}{cl}
  &amp; \dfrac{1}{k + 1} + \cdots + \dfrac{1}{N - 1} &lt; 1\\
  \Rightarrow &amp; \dfrac{1}{k + 2} + \cdots + \dfrac{1}{N} &lt; 1 - \left( \dfrac{1}{k + 1} - \dfrac{1}{N} \right) &lt; 1
\end{array}"></p>

故也不可能是比 k + 1 大的数。其次我们证明 P(k, N) >= P(k, N + 1)：

<p align="center"><img src="https://m1dor1.github.io/9.75/svg/4732486028916466384.svg" alt="\begin{array}{cl}
  &amp; \dfrac{k}{N} \left( \dfrac{1}{k} + \cdots + \dfrac{1}{N - 1} \right) - \dfrac{k}{N + 1} \left( \dfrac{1}{k} + \cdots + \dfrac{1}{N} \right)\\
  = &amp; \dfrac{k}{N (N + 1)} \left( \dfrac{1}{k} + \cdots + \dfrac{1}{N - 1} \right) - \dfrac{k}{N (N + 1)}\\
  = &amp; \dfrac{k}{N (N + 1)} \left( \dfrac{1}{k} + \cdots + \dfrac{1}{N - 1} - 1 \right) \geqslant 0
\end{array}"></p>

最后我们证明 P(k, N) > P(k + 1, N + 1)：

<p align="center"><img src="https://m1dor1.github.io/9.75/svg/-5330250605919793952.svg" alt="\begin{array}{ll}
  &amp; \dfrac{k}{N} \left( \dfrac{1}{k} + \cdots + \dfrac{1}{N - 1} \right) - \dfrac{k + 1}{N + 1} \left( \dfrac{1}{k + 1} + \cdots + \dfrac{1}{N} \right)\\
  = &amp; \dfrac{k - N}{N (N + 1)} \left( \dfrac{1}{k + 1} + \cdots + \dfrac{1}{N - 1} \right) + \dfrac{1}{N} - \dfrac{k + 1}{N (N + 1)}\\
  = &amp; \dfrac{k - N}{N (N + 1)} \left( \dfrac{1}{k + 1} + \cdots + \dfrac{1}{N - 1} \right) + \dfrac{N - k}{N (N + 1)}\\
  = &amp; \dfrac{k - N}{N (N + 1)} \left( \dfrac{1}{k + 1} + \cdots + \dfrac{1}{N - 1} - 1 \right) &gt; 0
\end{array}"></p>

因此最大的 P(k) 随着 N 递增单调递减，而 P(k) 显然有下界 0。回顾微积分知识，单调有界数列必有极限，我们来尝试证明这个极限就是 1/e，首先留意到 S 和 P(k) 可以联系起来：

<p align="center"><img src="https://m1dor1.github.io/9.75/svg/-8634867378899963437.svg" alt="\begin{array}{rl}
  S = &amp; \dfrac{1}{k} + \dfrac{1}{k + 1} + \cdots + \dfrac{1}{N - 1}\\
  = &amp; \left( \dfrac{k + 1}{k} - 1 \right) + \left( \dfrac{k + 2}{k + 1} - 1 \right) + \cdots + \left( \dfrac{N}{N - 1} - 1 \right)\\
  P = &amp; S \cdot \dfrac{k}{N} = S \cdot \dfrac{k}{k + 1} \cdot \dfrac{k + 1}{k + 2} \cdot \cdots \cdot \dfrac{N - 1}{N}
\end{array}"></p>

由几何均值不等式：

<p align="center"><img src="https://m1dor1.github.io/9.75/svg/8186406151852659599.svg" alt="\sqrt[N - k]{\prod_{n = k}^{N - 1} \frac{n + 1}{n}} \leqslant \frac{\sum_{n = k}^{N - 1} \frac{n + 1}{n}}{N - k} = \frac{S + N - k}{N - k} = 1 + \frac{S}{N - k}"></p>

去根号求倒数，于是有：

<p align="center"><img src="https://m1dor1.github.io/9.75/svg/-400933789699923878.svg" alt="P = S \cdot \prod_{n = k}^{N - 1} \frac{n}{n + 1} \geqslant S / \left( 1 + \frac{S}{N - k} \right)^{N - k} &gt; S \cdot e^{- S}"></p>

当 N 趋于正无穷时，S 趋于 1，N - k 趋于正无穷，故 P 的极限为 S / e^S = 1/e，lim k / N = P / S = 1/e。~~上一节挖的坑终于填上了（苦笑）~~

### 证明的补充

虽然「当 N 趋于正无穷时，S 趋于 1，N - k 趋于正无穷」是显然的，但为了证明的完整性，还是有必要进行补充说明。首先我们可以证明 N > 2 时，N - k > k：

<p align="center"><img src="https://m1dor1.github.io/9.75/svg/1688490370974642601.svg" alt="1 \leqslant \frac{1}{k} + \frac{1}{k + 1} + \cdots + \frac{1}{N - 1} &lt; \frac{1}{k} \cdot (N - k)"></p>

其次我们证明 N 趋于正无穷时，k 也趋于正无穷。还是使用放缩法，注意到 N > 4, N' = 2N 的时候 k' >= k + 1：

<p align="center"><img src="https://m1dor1.github.io/9.75/svg/-9061068061483126859.svg" alt="\frac{1}{k} + \cdots + \frac{1}{N - 1} + \cdots + \frac{1}{2 N - 1} &gt; 1 + \frac{N}{2 N - 1} &gt; 1 + \frac{1}{2}"></p>

上面我们已经证明 N' = N + 1 时，k' 只能为 k 或者 k + 1，所以 k(N) 单调递增，当 N 趋于正无穷时，k 也趋于正无穷，而由 N > N - k > k，N - k 也趋于正无穷。

最后回顾数列极限的定义，要证明「当 N 趋于正无穷时，S 趋于 1」，等于证明「对任意的 ε，总存在一个 N，使得当 n > N 时 S - 1 < ε 总能成立」。对任意的 ε，我们总能找到一个正整数 k，使得 1/k < ε，并能找到 k 对应的一个 N，从而有：

<p align="center"><img src="https://m1dor1.github.io/9.75/svg/-8811312657620901806.svg" alt="S = \frac{1}{k} + \frac{1}{k + 1} + \cdots + \frac{1}{N - 1} = 1 + \varepsilon_1 &gt; 1 \Rightarrow \varepsilon_1 &lt; \frac{1}{k} &lt; \varepsilon"></p>

因此对任意的 n > N，k(n) >= k，S = 1 + ε(n) < 1 + 1/k(n) <= 1 + 1/k < 1 + ε 总能成立，证毕。

## 4. 休息一下

看到这里你大概会觉得这篇文章快要结束了吧，毕竟问题得到了完美的解决~~（还开了透视锁头挂😂）~~。可惜的是，前面只是引子，正文才刚刚开始。

仔细回想一下，其实还有两个问题没有解决：

1. 这种策略是最优解吗？如果是，为什么？
2. 如果想不到这个算法，怎么办？

刷过算法题的人，看到「最」字，多少会意识到，很可能是动态规划题！

## 5. 最短路径问题

作为对动态规划的回顾，我们来重温最短路径问题。

得益于科技的进步，导航 App 的出现让我们不再迷路，并且能在没人指路的情况下顺利到达目的地。然而，你是否曾好奇，导航 App 里的最佳路线，是怎么算出来的呢？

<p align="center"><img src="https://m1dor1.github.io/9.75/svg/-7302496205316364015.svg" alt="\begin{tikzpicture}[line cap=round,line join=round,&gt;=triangle 45,x=1cm,y=1cm]
\clip(-7.46,-9.84) rectangle (7.18,4.54);
\draw [line width=0.8pt] (-5,0)-- (-3,2);
\draw [line width=0.8pt] (-5,0)-- (-3,-2);
\draw [line width=0.8pt] (-3,2)-- (-1,2);
\draw [line width=0.8pt] (-1,2)-- (1,2);
\draw [line width=0.8pt] (-3,2)-- (-1,0);
\draw [line width=0.8pt] (-1,0)-- (1,2);
\draw [line width=0.8pt] (-3,-2)-- (-1,2);
\draw [line width=0.8pt] (-1,2)-- (1,0);
\draw [line width=0.8pt] (-3,-2)-- (-1,-2);
\draw [line width=0.8pt] (-1,-2)-- (1,2);
\draw [line width=0.8pt] (-1,-2)-- (1,0);
\draw [line width=0.8pt] (-1,-2)-- (1,-2);
\draw [line width=0.8pt] (1,-2)-- (3,0);
\draw [line width=0.8pt] (1,2)-- (3,0);
\draw [line width=0.8pt] (1,0)-- (3,0);
\draw (-4.4,1.45) node[anchor=north west] {3};
\draw (-4.4,-0.9) node[anchor=north west] {4};
\draw (-2.2,2.56) node[anchor=north west] {1};
\draw (-0.2,2.56) node[anchor=north west] {1};
\draw (2.2,1.45) node[anchor=north west] {1};
\draw (2.2,-0.8) node[anchor=north west] {-2};
\draw (1.8,0.56) node[anchor=north west] {0};
\draw (-0.7,1.05) node[anchor=north west] {1};
\draw (-0.7,1.45) node[anchor=north west] {2};
\draw (-2.6,1.22) node[anchor=north west] {-1};
\draw (-2.6,0) node[anchor=north west] {2};
\draw (-0.6,0) node[anchor=north west] {2};
\draw (0.2,-0.8) node[anchor=north west] {-2};
\draw (-2.2,-1.5) node[anchor=north west] {5};
\draw (-0.2,-1.5) node[anchor=north west] {2};
\begin{scriptsize}
\draw [fill=black] (-5,0) circle (2.5pt);
\draw[color=black] (-5,0.43) node {$A$};
\draw [fill=black] (-3,2) circle (2.5pt);
\draw[color=black] (-3,2.43) node {$B$};
\draw [fill=black] (-3,-2) circle (2.5pt);
\draw[color=black] (-3,-1.57) node {$C$};
\draw [fill=black] (-1,2) circle (2.5pt);
\draw[color=black] (-1,2.43) node {$D$};
\draw [fill=black] (-1,0) circle (2.5pt);
\draw[color=black] (-1,0.43) node {$E$};
\draw [fill=black] (-1,-2) circle (2.5pt);
\draw[color=black] (-1,-1.57) node {$F$};
\draw [fill=black] (1,2) circle (2.5pt);
\draw[color=black] (1,2.43) node {$G$};
\draw [fill=black] (1,0) circle (2.5pt);
\draw[color=black] (1,0.43) node {$H$};
\draw [fill=black] (1,-2) circle (2.5pt);
\draw[color=black] (1,-1.57) node {$I$};
\draw [fill=black] (3,0) circle (2.5pt);
\draw[color=black] (3,0.43) node {$J$};
\end{scriptsize}
\end{tikzpicture}"></p>

如图所示，我们希望求从 A 点到 J 点的最短路径（所有权重加起来最小）。虽然不知道哪一条路径才是最短的，但是我们知道，如果这条最短的路径经过 G 点，那这条路径从 A 到 G 段，一定也是从 A 点到 G 点的最短路径。假如 f(X) 表示从 A 点到 X 点最短路径的总权重大小，那么：

<p align="center"><img src="https://m1dor1.github.io/9.75/svg/-5702516332670911375.svg" alt="f (J) = \min \{ f (G) + 1, f (H) + 0, f (I) - 2 \}"></p>

因此我们可以从 A 点开始，一层一层计算下去，最后就能得到 A 点到 J 点的最短路径：

| 层数 | 点 X | f(X) | 最短路径 |
| :-: | :-: | --- | --- |
| 0 | A | f(A) = 0 | A |
| 1 | B | f(B) = f(A) + 3 = 3 | A --- B |
|   | C | f(C) = f(A) + 4 = 4 | A --- C |
| 2 | D | f(D) = min{f(B) + 1, f(C) + 2} = f(B) + 1 = 4 | A --- B --- D |
|   | E | f(E) = f(B) - 1 = 2 | A --- B --- E |
|   | F | f(F) = f(C) + 5 = 9 | A --- C --- F |
| 3 | G | f(G) = min{f(D) + 1, f(E) + 1, f(F) + 2} = f(E) + 1 = 3 | A --- B --- E --- G |
|   | H | f(H) = min{f(D) + 2, f(F) - 2} = f(D) + 2 = 6 | A --- B --- D --- H |
|   | I | f(I) = f(F) + 2 = 11 | A --- C --- F --- I |
| 4 | J | f(J) = min{f(G) + 1, f(H) + 0, f(I) - 2} = f(G) + 1 = 4 | A --- B --- E --- G --- J |

「导航正式开始！」

<p align="center"><img src="https://m1dor1.github.io/9.75/svg/-6621267061444958900.svg" alt="\definecolor{ccqqqq}{rgb}{0.8,0,0}
\begin{tikzpicture}[line cap=round,line join=round,&gt;=triangle 45,x=1cm,y=1cm]
\clip(-7.46,-9.84) rectangle (7.18,4.54);
\draw [line width=1.2pt, color=ccqqqq] (-5,0) -- (-3,2);
\draw [line width=0.8pt] (-5,0) -- (-3,-2);
\draw [line width=0.8pt] (-3,2) -- (-1,2);
\draw [line width=0.8pt] (-1,2) -- (1,2);
\draw [line width=1.2pt, color=ccqqqq] (-3,2) -- (-1,0);
\draw [line width=1.2pt, color=ccqqqq] (-1,0) -- (1,2);
\draw [line width=0.8pt] (-3,-2) -- (-1,2);
\draw [line width=0.8pt] (-1,2) -- (1,0);
\draw [line width=0.8pt] (-3,-2) -- (-1,-2);
\draw [line width=0.8pt] (-1,-2) -- (1,2);
\draw [line width=0.8pt] (-1,-2) -- (1,0);
\draw [line width=0.8pt] (-1,-2) -- (1,-2);
\draw [line width=0.8pt] (1,-2) -- (3,0);
\draw [line width=1.2pt, color=ccqqqq] (1,2) -- (3,0);
\draw [line width=0.8pt] (1,0) -- (3,0);
\draw (-4.4,1.45) node[anchor=north west] {3};
\draw (-4.4,-0.9) node[anchor=north west] {4};
\draw (-2.2,2.56) node[anchor=north west] {1};
\draw (-0.2,2.56) node[anchor=north west] {1};
\draw (2.2,1.45) node[anchor=north west] {1};
\draw (2.2,-0.8) node[anchor=north west] {-2};
\draw (1.8,0.56) node[anchor=north west] {0};
\draw (-0.7,1.05) node[anchor=north west] {1};
\draw (-0.7,1.45) node[anchor=north west] {2};
\draw (-2.6,1.22) node[anchor=north west] {-1};
\draw (-2.6,0) node[anchor=north west] {2};
\draw (-0.6,0) node[anchor=north west] {2};
\draw (0.2,-0.8) node[anchor=north west] {-2};
\draw (-2.2,-1.5) node[anchor=north west] {5};
\draw (-0.2,-1.5) node[anchor=north west] {2};
\begin{scriptsize}
\draw [fill=black] (-5,0) circle (2.5pt);
\draw[color=black] (-5,0.43) node {$A$};
\draw [fill=black] (-3,2) circle (2.5pt);
\draw[color=black] (-3,2.43) node {$B$};
\draw [fill=black] (-3,-2) circle (2.5pt);
\draw[color=black] (-3,-1.57) node {$C$};
\draw [fill=black] (-1,2) circle (2.5pt);
\draw[color=black] (-1,2.43) node {$D$};
\draw [fill=black] (-1,0) circle (2.5pt);
\draw[color=black] (-1,0.43) node {$E$};
\draw [fill=black] (-1,-2) circle (2.5pt);
\draw[color=black] (-1,-1.57) node {$F$};
\draw [fill=black] (1,2) circle (2.5pt);
\draw[color=black] (1,2.43) node {$G$};
\draw [fill=black] (1,0) circle (2.5pt);
\draw[color=black] (1,0.43) node {$H$};
\draw [fill=black] (1,-2) circle (2.5pt);
\draw[color=black] (1,-1.57) node {$I$};
\draw [fill=black] (3,0) circle (2.5pt);
\draw[color=black] (3,0.43) node {$J$};
\end{scriptsize}
\end{tikzpicture}"></p>

## 6. 应用到秘书问题

要使用动态规划解决问题，我们需要先确定所有可能的状态，并确定状态之间的联系，最后把问题分解为子问题开始处理。回顾上面的选择过程，可以看到所有的状态，由三个变量构成：

- 当前麦穗的序号 k
- 当前麦穗是否为（已经看到的所有麦穗中的）最好的麦穗
- 是否已经做出了选择

我们这里沿用 [Venkat Anantharam](https://people.eecs.berkeley.edu/~ananth/223Spr07/lec4_ee223_spr07.pdf) 老师使用的记号，如果已作出选择，就记为 ∆，如果是最优，则记为 1，否则记为 0。于是在第 k 个麦穗时，一共有四种状态：

- (k, 1) 表示当前的第 k 个麦穗是目前最好的麦穗，且还没有作出选择；
- (k, 0) 表示当前的第 k 个麦穗并不是目前最好的麦穗，且还没有作出选择；
- (∆, k, 1) 表示当前是第 k 个麦穗，已经做出了选择，且 **选择的麦穗是目前最好的麦穗**；
- (∆, k, 0) 表示当前是第 k 个麦穗，已经做出了选择，且 **选择的麦穗并不是目前最好的麦穗**。

k = 1 时必然是最好的，故 (k, 0) 和 (∆, k, 0) 并不存在。

接下来我们确定状态之间的联系。

- 由于我们希望选到最好的麦穗，(k, 0) 时我们并不会做出选择，因此接下来的状态可能是 (k + 1, 0) 和 (k + 1, 1)。由于我们不知道所有麦穗的大小分布情况，k + 1 个麦穗中，最好的麦穗在前 k 个中的概率显然是 k / (k + 1)，因此转移到 (k + 1, 0) 的概率是 k / (k + 1)，转移到 (k + 1, 1) 的概率是 1 / (k + 1)。
- 而对于 (k, 1) 的情况，**可以做出选择**：做出选择会跳转到 (∆, k, 1)，不做出选择的话和 (k, 0) 一样，转移到 (k + 1, 0) 的概率是 k / (k + 1)，转移到 (k + 1, 1) 的概率是 1 / (k + 1)。
- (∆, k, 1) 也是类似，转移到 (∆, k + 1, 1) 的概率是 k / (k + 1)，转移到 (∆, k + 1, 0) 的概率是 1 / (k + 1)。
- (∆, k, 0) 显然是传递性的，因此只会转移到 (∆, k + 1, 0)。

我们以 N = 4 为例，画出示意图（部分转移概率未标出，使用蓝色和紫色表示此时可以做出选择或者不做）：

<p align="center"><img src="https://m1dor1.github.io/9.75/svg/-329896808788267486.svg" alt="\definecolor{ppl}{rgb}{0.6,0.2,1}
\definecolor{blu}{rgb}{0,0,1}
\begin{tikzpicture}
\draw [line width=0.8pt, color=blu] (-6,-1.5) -- (-6,1.5);
\draw [line width=0.8pt, color=ppl] (-6,-1.5) -- (-3,-1.5);
\draw [line width=0.8pt, color=ppl] (-6,-1.5) -- (-3,-4.5);
\draw [line width=0.8pt] (-6,1.5) -- (-3,4.5);
\draw [line width=0.8pt] (-6,1.5) -- (-3,1.5);
\draw [line width=0.8pt] (-3,4.5) -- (0,4.5);
\draw [line width=0.8pt] (0,4.5) -- (3,4.5);
\draw [line width=0.8pt] (3,4.5) -- (6,4.5);
\draw [line width=0.8pt, color=blu] (-3,-1.5) -- (-3,1.5);
\draw [line width=0.8pt, color=blu] (0,-1.5) -- (0,1.5);
\draw [line width=0.8pt, color=blu] (3,-1.5) -- (3,1.5);
\draw [line width=0.8pt, color=ppl] (-3,-1.5) -- (0,-1.5);
\draw [line width=0.8pt, color=ppl] (-3,-1.5) -- (0,-4.5);
\draw [line width=0.8pt, color=ppl] (0,-1.5) -- (3,-1.5);
\draw [line width=0.8pt, color=ppl] (0,-1.5) -- (3,-4.5);
\draw [line width=0.8pt] (-3,1.5) -- (0,1.5);
\draw [line width=0.8pt] (0,1.5) -- (3,1.5);
\draw [line width=0.8pt] (3,1.5) -- (6,1.5);
\draw [line width=0.8pt] (-3,1.5) -- (0,4.5);
\draw [line width=0.8pt] (0,1.5) -- (3,4.5);
\draw [line width=0.8pt] (3,-4.5) -- (6,-4.5);
\draw [line width=0.8pt] (-3,-4.5) -- (0,-4.5);
\draw [line width=0.8pt] (0,-4.5) -- (3,-1.5);
\draw [line width=0.8pt] (0,-4.5) -- (3,-4.5);
\draw [line width=0.8pt] (-3,-4.5) -- (0,-1.5);
\draw [line width=0.8pt, color=ppl] (3,-1.5) -- (6,-4.5);
\begin{scriptsize}
\draw [fill=black] (-6,1.5) circle (2.5pt);
\draw[color=black] (-5.3,1.1) node {($\triangle$, 1, 1)};
\draw [fill=black] (-6,-1.5) circle (2.5pt);
\draw[color=black] (-5.5,-1.2) node {(1, 1)};
\draw [fill=black] (-3,4.5) circle (2.5pt);
\draw[color=black] (-2.3,4.1) node {($\triangle$, 2, 0)};
\draw [fill=black] (-3,1.5) circle (2.5pt);
\draw[color=black] (-2.3,1.1) node {($\triangle$, 2, 1)};
\draw [fill=black] (-3,-1.5) circle (2.5pt);
\draw[color=black] (-2.5,-1.2) node {(2, 1)};
\draw [fill=black] (-3,-4.5) circle (2.5pt);
\draw[color=black] (-2.3,-4.1) node {(2, 0)};
\draw [fill=black] (0,4.5) circle (2.5pt);
\draw[color=black] (0.7,4.1) node {($\triangle$, 3, 0)};
\draw [fill=black] (0,1.5) circle (2.5pt);
\draw[color=black] (0.7,1.1) node {($\triangle$, 3, 1)};
\draw [fill=black] (0,-1.5) circle (2.5pt);
\draw[color=black] (0.5,-1.2) node {(3, 1)};
\draw [fill=black] (0,-4.5) circle (2.5pt);
\draw[color=black] (0.5,-4.1) node {(3, 0)};
\draw [fill=black] (3,4.5) circle (2.5pt);
\draw[color=black] (3.7,4.1) node {($\triangle$, 4, 0)};
\draw [fill=black] (3,1.5) circle (2.5pt);
\draw[color=black] (3.7,1.1) node {($\triangle$, 4, 1)};
\draw [fill=black] (3,-1.5) circle (2.5pt);
\draw[color=black] (3.5,-1.2) node {(4, 1)};
\draw [fill=black] (3,-4.5) circle (2.5pt);
\draw[color=black] (3.5,-4.1) node {(4, 0)};
\draw [fill=black] (6,4.5) circle (2.5pt);
\draw[color=black] (6,4.9) node {0};
\draw [fill=black] (6,1.5) circle (2.5pt);
\draw[color=black] (6,1.9) node {1};
\draw [fill=black] (6,-4.5) circle (2.5pt);
\draw[color=black] (6,-4.1) node {0};
\draw[color=black] (-4.5,3.5) node {1/2};
\draw[color=black] (-4.5,2) node {1/2};
\draw[color=black] (-1.5,2) node {2/3};
\draw[color=black] (1.5,2) node {3/4};
\draw[color=ppl] (-4.5,-1) node {1/2};
\draw[color=ppl] (-1.5,-1) node {1/3};
\draw[color=ppl] (1.5,-1) node {1/4};
\draw[color=ppl] (-4.5,-3.5) node {1/2};
\draw[color=black] (-1.5,-4) node {2/3};
\draw[color=black] (1.5,-4) node {3/4};
\end{scriptsize}
\end{tikzpicture}"></p>

可以看到和上面的最短路径问题基本一致，虽然还是有点区别：

- 最短路径问题是最小化权重，选麦穗问题是最大化可能；
- 最短路径问题是确定的，选麦穗问题带有随机性，需要计算期望（均值）；
- 最短路径问题是正向的，选麦穗问题是反向进行的。

最后自然是各状态可能性大小的计算（从右到左反向进行），令 J(X) 表示在 X 状态下选到最好的麦穗的最大概率，则显然有：

<p align="center"><img src="https://m1dor1.github.io/9.75/svg/-7250911767254105552.svg" alt="\begin{array}{ccl}
  J (\triangle, k, 0) &amp; = &amp; J (\triangle, k + 1, 0) = \cdots = 0\\
  J (\triangle, k, 1) &amp; = &amp; \frac{1}{k + 1} J (\triangle, k + 1, 0) +
  \frac{k}{k + 1} J (\triangle, k + 1, 1)\\
  &amp; = &amp; \frac{k}{k + 1} \cdot \left( \frac{k + 1}{k + 2} \left( \ldots \left(
  \frac{N - 1}{N} J (\triangle, N, 1) \right) \right) \right) = \frac{k}{N}\\
  J (k, 1) &amp; = &amp; \max \left\{ J (\triangle, k, 1), \frac{1}{k + 1} J (k + 1,
  1) + \frac{k}{k + 1} J (k + 1, 0) \right\}\\
  &amp; = &amp; \max \left\{ \frac{k}{N}, J (k, 0) \right\}\\
  J (k, 0) &amp; = &amp; \frac{1}{k + 1} J (k + 1, 1) + \frac{k}{k + 1} J (k + 1, 0)
\end{array}"></p>

因此只计算 J(k, 1) 与 J(k, 0) 就足够了，依然是分层计算：

| 层数 | 状态 X | J(X) | 是否做出选择 |
| :-: | :-: | --- | --- |
| 0 | (4, 0) | J(4, 0) = 0 | --- |
|   | (4, 1) | J(4, 1) = max{4/4, J(4, 0)} = 1 | 是 |
| 1 | (3, 0) | J(3, 0) = 1/4 J(4, 1) + 3/4 J(4, 0) = 1/4 | --- |
|   | (3, 1) | J(3, 1) = max{3/4, J(3, 0)} = 3/4 | 是 |
| 2 | (2, 0) | J(2, 0) = 1/3 J(3, 1) + 2/3 J(3, 0) = 5/12 | --- |
|   | (2, 1) | J(2, 1) = max{2/4, J(2, 0)} = 1/2 | 是 |
| 3 | (1, 1) | J(1, 1) = max{1/4, 1/2 J(2,0) + 1/2 J(2,1)} = 11/24 | 否 |

竟然和 Odds algorithm 的结果完全一致！

<p align="center"><img src="https://m1dor1.github.io/9.75/svg/-4200998759319817544.svg" alt="S = 1 + \frac{1}{2} + \frac{1}{3} = \frac{11}{6} &gt; 1 \Rightarrow k = 1, P = S \cdot \frac{k}{N} = \frac{11}{24}"></p>

## 7. 两个算法的等价性

没错，这两个算法确实是等价的，因此开头的策略，确实是使选到最好的麦穗概率最大化的最优策略。

首先显然 J(x, 0) 从 x = N 开始（倒过来看）是单调递增的，且最小值为 1/N，而 x / N （倒过来看）单调递减，且最后会降到 1/N，必然存在临界点 k，当 x <= k 时，x/N <= J(x, 0)，因此不作出选择；当 x > k 时，x/N > J(x, 0)，因此每次见到最好的麦穗都会做出选择。这和 Odds algorithm 对应的策略完全一致。

接下来我们来证明这个临界点 k 就是 Odds algorithm 里的 k。注意到：

<p align="center"><img src="https://m1dor1.github.io/9.75/svg/-4222438951552325156.svg" alt="\begin{array}{cl}
  &amp; J (k, 0) = \frac{1}{k + 1} J (k + 1, 1) + \frac{k}{k + 1} J (k + 1, 0)\\
  \Rightarrow &amp; \frac{1}{k + 1} J (k + 1, 0) - \frac{1}{k} J (k, 0)\\
  = &amp; - \frac{1}{k (k + 1)} J (k + 1, 1) = - \frac{1}{k (k + 1)} \cdot \frac{k + 1}{N} = - \frac{1}{k} \cdot \frac{1}{N}
\end{array}"></p>

因此：

<p align="center"><img src="https://m1dor1.github.io/9.75/svg/-7150690618258797425.svg" alt="\begin{array}{rll}
  \frac{1}{N} J (N, 0) - \frac{1}{N - 1} J (N - 1, 0) &amp; = &amp; - \frac{1}{N - 1} \cdot \frac{1}{N}\\
  \frac{1}{N - 1} J (N - 1, 0) - \frac{1}{N - 2} J (N - 2, 0) &amp; = &amp; - \frac{1}{N - 2} \cdot \frac{1}{N}\\
  \ldots &amp;  &amp; \\
  \frac{1}{k + 1} J (k + 1, 0) - \frac{1}{k} J (k, 0) &amp; = &amp; - \frac{1}{k} \cdot \frac{1}{N}\\
  \text{(Sum)}\Rightarrow \frac{1}{N} J (N, 0) - \frac{1}{k} J (k, 0) &amp; = &amp; - \frac{1}{N} \left( \frac{1}{k} + \frac{1}{k + 1} + \ldots + \frac{1}{N - 1} \right)\\
  (J(N, 0) = 0)\Rightarrow J (k, 0) &amp; = &amp; \frac{k}{N} \left( \frac{1}{k} + \frac{1}{k + 1} + \ldots + \frac{1}{N - 1} \right)\\
  \Rightarrow J (k, 0) \geqslant \frac{k}{N} &amp; \Leftrightarrow &amp; \frac{1}{k} + \frac{1}{k + 1} + \ldots + \frac{1}{N - 1} \geqslant 1
\end{array}"></p>

Q.E.D.

## 8. 今回はここまで

{% include img.html url="/pic/end.jpg" alt="到这里就结束啦～" %}

本来打算 2 月 14 号写完的，🐦到了现在……

写正经的文章果然很累，不过 EE 真有意思，如果重新选专业大概会选 EE 吧～~~不过也是叶公好龙😂~~

### 8.1 历史与变形

这个选麦穗的故事有很多版本（苏格拉底与徒弟、禅师与失恋青年、……），还登上了小学语文教科书，但是真实性显然是无法考证的。根据 Thomas S. Ferguson 的 [考证](https://www.math.upenn.edu/~ted/210F10/References/Secretary.pdf)，最早有记载的「秘书问题」，是 1875 年 Cayley 给 Educational Times 投稿的一个数学问题：

> 有一种彩票玩法如下：每次可以取一把奖励大小不同的彩票，只能领取最后一张刮开的彩票的奖励。如果采用最优的策略，拿到奖励的期望是多少？

Cayley 使用了动态规划，并以 N = 4 为例进行了说明。此后这个问题出现了很多种不同的形式：

- 秘书问题：一位经理要从 N 位候选人中雇佣一位秘书，见面之后决定是否录用，拒绝了之后就不再召见了。经理要怎么做才能使得选中最好的一位的概率最大？
- 相亲问题
- 卖房问题：稍微进行了一点变形，房子卖出后立刻买入利率为 r% 的理财产品。可以使用动态规划解决。
- 群面问题：把所有面试者分成 N 组面试，只留下每一组的最优者，然后依次面试，拒绝后不再召见。被证明与秘书问题等价。
- ……

直到 2000 年，F. Thomas Bruss 研究「最后一次成功是第几次」（最优停时问题）的时候，[Odds theorem](https://doi.org/10.1214%2Faop%2F1019160340) 才被发现，并被应用到秘书问题上（秘书问题的“成功”，自然是超过前面的所有人）。

### 8.2 わたし、气になります！

{% include img.html url="/pic/kininari.jpg" alt="<del>好奇会秃头的！</del>" %}

欢迎来到随机控制（Stochastic control）与优化理论的世界！很多大学都有相应的课程，比如 [Stanford EE365](https://web.stanford.edu/class/ee365/)、[MIT OCW](https://ocw.mit.edu/courses/electrical-engineering-and-computer-science/6-231-dynamic-programming-and-stochastic-control-fall-2015/lecture-notes/)、[Berkeley EECS](https://people.eecs.berkeley.edu/~ananth/223Spr07)、[ETH Zurich](https://idsc.ethz.ch/education/lectures/optimal-control.html)……

教材方面自然是厚厚两卷的 Dynamic Programming & Optimal Control by [Dimitri P. Bertsekas](https://www.mit.edu/~dimitrib/home.html)。书里有非常多的习题，而且都很有意思，然而并没有解答😂（[MIT OCW](https://ocw.mit.edu/courses/electrical-engineering-and-computer-science/6-231-dynamic-programming-and-stochastic-control-fall-2015/assignments/) 上能找到一部分解答）。可惜 Dimitri P. Bertsekas 教授现在转教 [Reinforcement learning](https://www.mit.edu/~dimitrib/Dynamic_Prog_Videos.html) 了……

[^1]: 学过概率论应该会发现，这里实际上是求随机变量的期望。

[^2]: 其实 ∇P / 1 就是一阶（后向）差商（类似导数）。

[^3]: 一种方法是使用 Lagrange 中值定理，ln(k + 1) - ln(k) = 1/(k + ε) < 1/k。
