# Combination Problem with GF

## 介紹

生成函數也可以用來解組合數問題。

> \\( f(x) \\) 為用 \\( 1, 5, 10, 50 \\) 元硬幣湊出 \\( x \\) 元的方法數。求 \\( f(x) \\) 的 OGF \\( F(x) \\)。

\begin{aligned}
F(x) &= (1 + x + x^2 + \cdots)(1 + x^5 + x^{10} + \cdots)(1 + x^{10} + x^{20} + \cdots)(1 + x^{50} + x^{100} + \cdots) \\\\
&= \frac{1}{1 - x} \frac{1}{1 - x^5} \frac{1}{1 - x^{10}} \frac{1}{1 - x^{50}} & \text{每個括號內的都是等比級數} \\\\
&= \frac{1}{(1 - x)(1 - x^5)(1 - x^{10})(1 - x^{50})} \\\\
\end{aligned}

等號右邊每個多項式的 \\( [x^k] \\) 項係數代表使用該種硬幣湊出 \\( k \\) 元的方法數，因此只有在次方為幣值倍數的情況係數會是 \\( 1 \\)。根據 OGF 卷積的定義，\\( [x^n]F(x) = \sum\limits_{a + b + c + d = n} [x^a]\frac{1}{1 - x} [x^b]\frac{1}{1 - x^5} [x^c]\frac{1}{1 - x^{10}} [x^d]\frac{1}{1 - x^{50}} \\)，也就會是使用這 \\( 4 \\) 種硬幣湊出 \\( n \\) 元的方法數。

## 例題

> [ABC303 Ex - Constrained Tree Degree](https://atcoder.jp/contests/abc303/tasks/abc303_h)
>
> 給定正整數 \\( N \\) 和集合 \\( S \\)，求 \\( N \\) 個節點且每個節點的 degree \\( \in S \\) 的樹數量 \\( \bmod 998244353 \\)。
>
> - \\( 2 \leq N \leq 2 \cdot 10^5 \\)
> - \\( 1 \leq s_1 < s_2 < \dots < s_{p} < N - 1 \\)

本題是 EGF 的應用。

我們知道樹與 [prufer sequence](https://zh.wikipedia.org/zh-tw/%E6%99%AE%E5%90%95%E5%BC%97%E5%BA%8F%E5%88%97) 有一對一的關係，\\( N \\) 個節點的樹可以對應到唯一長度為 \\( N - 2 \\) 的 prufer sequence，且對於樹中每個節點的 degree \\( d_i \\) 會剛好在 sequence 中出現 \\( d_i - 1 \\) 次。

根據以上的性質，題目可以化簡為：

> 求長度為 \\( N - 2 \\)，每個數字的範圍 \\( 1 \leq a_i \leq N \\) 且 \\( d_i + 1 \in S \\) 的數列數量

將 \\( N \\) 種數字分開考慮，如果出現的次數合法就會對答案的方法數貢獻 \\( 1 \\)，但同時要除以出現次數 (排列數)。因此非常適合套用 EGF！

定義 EGF \\( f(x) = \sum\limits_{k = 0}^{N} [k + 1 \in S] \frac{x^k}{k!}\\)，其中 \\( [\text{proposition}] \\) 為[艾佛森括號](https://zh.wikipedia.org/zh-tw/%E8%89%BE%E4%BD%9B%E6%A3%AE%E6%8B%AC%E5%8F%B7)。

令 \\( g_k \\) 為長度為 \\( k \\) 的合法數列數量。我們要求 \\( g \\) 的 EGF，可以列出

\\[ [x^k]g(x) = \sum\limits_{cnt_1 + cnt_2 + \dots + cnt_n = k} [x^{cnt_1}]f(x) \cdot [x^{cnt_2}]f(x) \cdots [x^{cnt_n}]f(x) \\]

不難看出 \\( g(x) = f(x)^N \\)，又因為 EGF \\( [x^k] g(x) = \frac{g_k}{k!} \\)，\\( g_k = k! \cdot [x^k] f(x)^N \\)。

因此答案就是 \\( g_{N - 2} = (N - 2)! \cdot [x^{N - 2}] f(x)^N \\)。

計算 \\( f(x)^N \\) 可以用快速冪 + NTT 做到 \\( O(N \log^2 N) \\) 或是運用 \\( f(x)^N = e^{N \log f(x)} \\) 的性質做到 \\( O(N \log N) \\)。

後者的做法一樣可以參考這三篇文章 [1](https://codeforces.com/blog/entry/56422) [2](https://cp-algorithms.com/algebra/polynomial.html) [3](https://www.luogu.com.cn/blog/MoYuFang/sheng-cheng-han-shuo-di-shuo-xue-ji-chu)。
