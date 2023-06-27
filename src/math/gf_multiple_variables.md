# GF with multiple variables

對於多變數的函數，我們一樣可以用生成函數求一般項。

> \\( f(n, k) = \binom{n}{k} \\)，\\( f \\) 可以以遞迴的形式表達：
> \begin{aligned} f(n, k) =
\begin{cases}
1, n \geq 0, k = 0 \\\\
0, n = 0, k \geq 1 \\\\
f(n - 1, k) + f(n - 1, k - 1), n, k \geq 1 \\\\
\end{cases}
\end{aligned}
>
> 求 \\( f \\) 的 OGF。

\\( f \\) 是線性遞迴，按照套路一樣先對定義的範圍加總，並將每一項乘以 \\( x^n y^k \\)：

\begin{aligned}
\sum\limits_{n = 1}^{\infty} \sum\limits_{k = 1}^{\infty} f(n, k) x^n y^k &= \sum\limits_{n = 1}^{\infty} \sum\limits_{k = 1}^{\infty} (f(n - 1, k) + f(n - 1, k - 1)) x^n y^k \\\\
&= x \sum\limits_{n = 1}^{\infty} \sum\limits_{k = 1}^{\infty} f(n - 1, k) x^{n - 1} y^k + xy \sum\limits_{n = 1}^{\infty} f(n - 1, k - 1) x^{n - 1} y^{k - 1} \\\\
\Longleftrightarrow F(x, y) - \sum\limits_{n = 0}^{\infty} x^n &= x(F(x, y) - \sum\limits_{n = 0}^{\infty} x^n) + xy F(x, y) & \text{by f(0, n) = 0} \\\\
\Longleftrightarrow F(x, y) - \frac{1}{1 - x} &= (x + xy)F(x, y) - \frac{x}{1 - x} & \text{by 等比級數，整理括號} \\\\
\Longleftrightarrow (1 - x - xy)F(x, y) &= 1 \\\\
\Longleftrightarrow F(x, y) &= \frac{1}{1 - x - xy} \\\\
\end{aligned}

我們得到 \\( f(n, k) \\) 的 OGF 為 \\( F(x, y) = \frac{1}{1 - x - xy} \\)。可以觀察 OGF 在單一變數的變化，比如固定 \\( n \\)，也就是求 \\( f_n(k) = \binom{n}{k} \\) 的 OGF，就會是

\begin{aligned}
f_n(k) &= [x^n]F(x, y) \\\\
&= [x^n]\frac{1}{1 - x - xy} \\\\
&= [x^n]\frac{1}{1 - x(1 + y)} \\\\
&= [x^n]\sum\limits_{n = 0}^{\infty} (1 + y)^n x^n & \text{將等比級數展開，公比為 \\( x(1 + y) \\)} \\\\
&= (1 + y)^n \\\\
&= \sum\limits_{k = 0}^n \binom{n}{k} y^k & \text{二項式定理} \\\\
\end{aligned}

這與二項式定理告訴我們的結論相同。

再比方說固定 \\( k \\)，也就是求 \\( f_k(n) = \binom{n}{k} \\) 的 OGF，可以列出

\begin{aligned}
f_k(n) &= [y^k]F(x, y) \\\\
&= [y^k]\frac{1}{1 - x - xy} \\\\
&= [y^k]\frac{1}{(1 - x) - xy} \\\\
&= [y^k]\frac{\frac{1}{1 - x}}{1 - \frac{xy}{1 - x}} & \text{分子分母同時除以 \\( \frac{1}{1 - x} \\)} \\\\
&= [y^k]\frac{1}{1 - x} \sum\limits_{k = 0}^{\infty} (\frac{x}{1 - x})^k y^k & \text{將等比級數展開，公比為 \\( \frac{xy}{1 - x} \\)} \\\\
&= \frac{x^k}{(1 - x)^{k + 1}} \\\\
\Longleftrightarrow \binom{n}{k} &= [x^n] \frac{x^k}{(1 - x)^{k + 1}} \\\\
&= [x^{n - k}] \frac{1}{(1 - x)^{k + 1}} & \text{次方往左平移 \\( k \\) 項，係數不變} \\\\
\Longleftrightarrow \binom{n + k}{k} &= [x^n] \frac{1}{(1 - x)^{k + 1}} \\\\
\end{aligned}

這個形式的 OGF 會在 \\( k \\) 固定 \\( n \\) 改變的情況下用到。

## References

- [Tutorial] Generating Functions in Competitive Programming [Part 1](https://codeforces.com/blog/entry/77468) [Part 2](https://codeforces.com/blog/entry/77551)
- [generatingfunctionology](https://www2.math.upenn.edu/~wilf/gfologyLinked2.pdf)
