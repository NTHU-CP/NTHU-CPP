# Lagrange Inversion Theorem

## 介紹

Lagrange Inversion Theorem 是一個解 GF 很方便的工具。如果對證明沒有興趣的話可以跳過直接看應用。

> 定理：兩個 FPS \\( F(x) \\) 和 \\( G(x) \\) 滿足 \\( G(F(x)) = x \\)，且 \\( [x^0] F(x) = [x^0] G(x) = 0 \\)、\\( [x^1] F(x) \neq 0, [x^1] G(x) \neq 0 \\)，則
>
> \\[
[x^n] F(x) = \frac{1}{n} [x^{n - 1}] (\frac{x}{G(x)})^n
\\]

## 證明

在講解定理的證明之前，我們先來證明一個等下會用到的引理。

> 引理：對於任何 FPS 滿足 \\( F(0) = 0, F^{\prime}(0) \neq 0 \\)，和任何 \\( k \in \mathbb{Z} \\)，
>
> \\[
[x^{-1}] F^{\prime}(x) F(x)^k = [k = -1]
\\]
>
> 其中 \\( [k = -1] \\) 為[艾佛森括號](https://zh.wikipedia.org/zh-tw/%E8%89%BE%E4%BD%9B%E6%A3%AE%E6%8B%AC%E5%8F%B7)。

引理證明：

我們分開討論 \\( k = -1 \\) 和 \\( k \neq -1 \\) 的情況，

如果 \\( k \neq -1 \\)，則 \\(F^{\prime}(x)F(x)^k = \frac{1}{k + 1}(F(x)^{k + 1})^{\prime} \\) (by 微分 chain rule)，因此 \\( x^{-1} \\) 項的係數為 \\( 0 \\)。

如果 \\( k = -1 \\)，則

\begin{aligned}
F^{\prime}(x)(F(x))^{-1} &= \frac{F^{\prime}(x)}{F(x)} \\\\
&= \frac{a_1 + 2 a_2 x + 3 a_3 x^2 + \dots}{a_1 x + a_2 x^2 + a_3 x^3 + \dots} & \text{展開 FPS} \\\\
&= \frac{1 + 2 \frac{a_2}{a_1} x + 3 \frac{a_3}{a_1} x^2 + \dots}{x + \frac{a_2}{a_1} x^2 + \frac{a_3}{a_1} x^3 + \dots} & \text{分子分母同時除以 \\( a_1 \\)} \\\\
&= x^{-1} \frac{1 + 2 \frac{a_2}{a_1} x + 3 \frac{a_3}{a_1} x^2 + \dots}{1 + \frac{a_2}{a_1} x + \frac{a_3}{a_1} x^2 + \dots} & \text{提出 \\( x^{-1} \\)} \\\\
\Longleftrightarrow [x^{-1}] F^{\prime}(x)(F(x))^{-1} &= [x^0] \frac{1 + 2 \frac{a_2}{a_1} x + 3 \frac{a_3}{a_1} x^2 + \dots}{1 + \frac{a_2}{a_1} x + \frac{a_3}{a_1} x^2 + \dots} & \text{比對係數} \\\\
&= [x^0] (\frac{1}{1 + \frac{a_2}{a_1} x + \frac{a_3}{a_1} x^2 + \dots} + \frac{2 \frac{a_2}{a_1} x}{1 + \frac{a_2}{a_1} x + \frac{a_3}{a_1} x^2 + \dots} + \dots) & \text{把分子拆開} \\\\
&= [x^0] (\frac{1}{1 - (-\frac{a_2}{a_1} x - \frac{a_3}{a_1} x^2 - \dots)} + \frac{2 \frac{a_2}{a_1} x}{1 - (-\frac{a_2}{a_1} x - \frac{a_3}{a_1} x^2 - \dots)} + \dots) & \text{等比級數形式} \\\\
\end{aligned}

可以觀察到括號內為無窮個等比級數的加總，其中只有第一個等比級數的首項為常數 \\( 1 \\)，而公比都有帶 \\( x \\)，因此後面不會對 \\( x^0 \\) 係數有貢獻，故引理得證。

證明完引理，我們回過頭證明 Lagrange Inversion Theorem。

因為 \\( F(x) \\) 和 \\( G(x) \\) 滿足上述引理的條件，

\begin{aligned}
F(G(x)) &= x \\\\
\Longleftrightarrow F^{\prime}(G(x)) \cdot G^{\prime}(x) &= 1 & \text{等號兩邊同時微分} \\\\
\end{aligned}

因為 \\( F^{\prime}(x) = \sum\limits_{i = 1}^{\infty} i ([x^i] F(x)) x^{i - 1} \\) (微分的定義)，以 \\( x = G(x) \\) 代入後得到

\begin{aligned}
\sum\limits_{i = 1}^{\infty} i ([x^i] F(x)) G(x)^{i - 1} \cdot G^{\prime}(x) &= 1 \\\\
\Longleftrightarrow \sum\limits_{i = 1}^{\infty} i ([x^i] F(x)) G(x)^{i - 1 - n} \cdot G^{\prime}(x) &= G(x)^{-n} & \text{等號兩邊同時乘以 \\( G(x)^{-n} \\)} \\\\
\Longleftrightarrow [x^{-1}] \sum\limits_{i = 1}^{\infty} i ([x^i] F(x)) G(x)^{i - 1 - n} \cdot G^{\prime}(x) &= [x^{-1}] G(x)^{-n} & \text{提取 \\( x^{-1} \\) 項係數} \\\\
\Longleftrightarrow \sum\limits_{i = 1}^{\infty} i ([x^i] F(x)) [x^{-1}] G(x)^{i - 1 - n} \cdot G^{\prime}(x) &= [x^{-1}] G(x)^{-n} \\\\
\end{aligned}

根據引理，\\( [x^{-1}] G(x)^{i - 1 - n} \cdot G^{\prime}(x) \\) 只有在 \\( i - 1 - n = -1 \\) 時為 \\( 1 \\)，其他時候為 \\( 0 \\)，此時 \\( i = n \\)，因此

\begin{aligned}
n [x^n] F(x) &= [x^{-1}] G(x)^{-n} \\\\
\Longleftrightarrow n [x^n] F(x) &= [x^{n - 1}] (x^n G(x)^{-n}) & \text{比對係數} \\\\
\Longleftrightarrow [x^n] F(x) &= \frac{1}{n} [x^{n - 1}] (\frac{x}{G(x)})^n \\\\
\end{aligned}

定理證明完畢。

## 應用

> 給定 \\( n \\) 個左括號和右括號，定義 \\( c_n \\) 為合法的括號匹配數量，求 \\( c_n \\) 的一般項？([卡特蘭數](https://zh.wikipedia.org/zh-tw/%E5%8D%A1%E5%A1%94%E5%85%B0%E6%95%B0))

\\( c_0 = 1 \\)。因為任何一個合法的括號匹配 \\( s \\) 可以分解成 \\( s = (s_1)s_2 \\)，枚舉 \\( s_1 \\) 的長度，可以列出遞迴式：

\\[
c_{n + 1} = \sum\limits_{i = 0}^n c_i c_{n - i}
\\]

定義 \\( C(x) \\) 為 \\( c_n \\) 的 OGF，可以列出

\begin{aligned}
\sum\limits_{n = 0}^{\infty} c_{n + 1} x^n &= \sum\limits_{n = 0}^{\infty} \sum\limits_{i = 0}^n c_i c_{n - i} x^n & \text{兩邊同乘 \\( x^n \\) 並用 \\( \sum \\) 加起來} \\\\
\Longleftrightarrow \frac{C(x) - c_0}{x} &= \sum\limits_{n = 0}^{\infty} \sum\limits_{i = 0}^n c_i x^i c_{n - i} x^{n - i} & \text{將 \\( x^n \\) 拆開} \\\\
\Longleftrightarrow \frac{C(x) - 1}{x} &= C(x)^2 \\\\
\Longleftrightarrow C(x) - 1 &= x C(x)^2 \\\\
\end{aligned}

化簡到這裡就可以套用 Lagrange Inversion Theorem 了。因為定理要求 \\( F(x) \\) 的常數項為 \\( 0 \\)，因此我們定義 \\( F(x) = C(x) - 1 \\)，代入後得到

\begin{aligned}
F(x) &= x(F(x) + 1)^2 \\\\
\Longleftrightarrow \frac{F(x)}{(F(x) + 1)^2} &= x
\end{aligned}

我們要找 \\( G(F(x))) = x \\) 的 \\( G(x) \\)，不難發現 \\( G(x) = \frac{x}{(x + 1)^2} \\)。因此，

\begin{aligned}
\text{[}x^n\text{]} F(x) &= \frac{1}{n} [x^{n - 1}] (\frac{x}{G(x)})^n \\\\
&= \frac{1}{n} [x^{n - 1}] (x + 1)^{2n} & \text{代入 \\( G(x) \\)} \\\\
&= \frac{1}{n} [x^{n - 1}] \sum\limits_{i = 0}^{2n} \binom{2n}{i} x^i 1^{2n - i} & \text{二項式定理} \\\\
&= \frac{1}{n} \binom{2n}{n - 1} \\\\
&= \frac{1}{n + 1} \binom{2n}{n} \\\\
\end{aligned}

因此，\\( c_0 = 1 \\)，\\( c_n = \frac{1}{n + 1} \binom{2n}{n} \\) 為卡特蘭數的一般項。

## References

- [Lagrange Inversion Theorem](https://atcoder.jp/contests/abc222/editorial/2765)
