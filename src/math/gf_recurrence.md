# Solving recurrence with GF

## 介紹

我們可以用生成函數解遞迴關係式的一般項。

> 給定線性遞迴關係式，滿足
> \begin{aligned}
\begin{cases}
a_0 &= 0 \\\\
a_{n + 1} &= 3 a_n + 2, n \geq 0 \\\\
\end{cases}
\end{aligned}
>
> 求數列 \\( a \\) 的一般項

首先我們先把數列轉換成生成函數的形式，這樣比較方便運算。把等號兩邊同時乘以 \\( x^n \\)，並對定義的範圍做加總，得到

\begin{aligned}
\sum\limits_{n = 0}^{\infty} a_{n + 1} x^n &= \sum\limits_{n = 0}^{\infty} (3 a_n + 2) x^n \\\\
\Longrightarrow \frac{A(x) - a_0 x^0}{x} &= 3 \sum\limits_{n = 0}^{\infty} a_n x^n + 2 \sum\limits_{n = 0}^{\infty} x^n \\\\
\Longrightarrow \frac{A(x) - a_0 x^0}{x} &= 3A(x) + \frac{2}{1 - x}
\end{aligned}

將 \\( a_0 = 0 \\) 代入，

\begin{aligned}
\Longrightarrow A(x) &= 3xA(x) + \frac{2x}{1 - x} \\\\
\Longrightarrow (1 - 3x)A(x) &= \frac{2x}{1 - x} \\\\
\Longrightarrow A(x) &= \frac{2x}{(1 - x)(1 - 3x)}
\end{aligned}

因此，\\( A(x) = \frac{2x}{(1 - x)(1 - 3x)} \\) 為此數列的 OGF。找到 OGF 之後，我們可以求數列的一般項。先將 \\( A(x) \\) 寫成 [partial fraction](https://en.wikipedia.org/wiki/Partial_fraction_decomposition#Examples) 的形式，

\\[
A(x) = 2x(\frac{C}{1 - x} + \frac{D}{1 - 3x})
\\]

解出 \\( C = -\frac{1}{2}, D = \frac{3}{2} \\)。換成 partial fraction 形式的好處是他可以展開成等比級數，因此

\begin{aligned}
\Longrightarrow A(x) &= 2x(-\frac{1}{2} \sum\limits_{n = 0}^{\infty} x^n + \frac{3}{2} \sum\limits_{n = 0}^{\infty} 3^n x^n ) \\\\
\Longrightarrow A(x) &= 2x \sum\limits_{n = 0}^{\infty} (-\frac{1}{2} + \frac{3^{n + 1}}{2}) x^n \\\\
\Longrightarrow A(x) &= 2x \sum\limits_{n = 0}^{\infty} \frac{3^{n + 1} - 1}{2} x^n \\\\
\Longrightarrow A(x) &= \sum\limits_{n = 0}^{\infty} (3^{n + 1} - 1) x^{n + 1} \\\\
\Longrightarrow A(x) &= \sum\limits_{n = 1}^{\infty} (3^n - 1) x^n \\\\
\Longrightarrow A(x) &= 0 + (3^1 - 1) x^1 + (3^2 - 1) x^2 + (3^3 - 1) x^3 + \cdots \\\\
\Longrightarrow a_n &= [x^n]A(x) = 3^n - 1
\end{aligned}

因此我們得到 \\( a_n = 3^n - 1 \\)。

> 給定費氏數列 \begin{aligned}
\begin{cases}
f_0 &= 0 \\\\
f_1 &= 1 \\\\
f_n &= f_{n - 1} + f_{n - 2}, n \geq 2 \\\\
\end{cases}
\end{aligned}
>
> 求 \\( f \\) 的 OGF。

同樣先把數列轉換成多項式的形式：

\begin{aligned}
\sum\limits_{n = 2}^{\infty} f_n x^n &= x \sum\limits_{n = 2}^{\infty} f_{n - 1} x^{n - 1} + x^2 \sum\limits_{n = 2}^{\infty} f_{n - 2} x^{n - 2} \\\\
\Longleftrightarrow F(x) - f_0 x^0 - f_1 x^1 &= x(F(x) - f_0 x^0) + x^2 F(x) & \text{減掉缺少的項} \\\\
\Longleftrightarrow F(x) - x &= (x + x^2) F(x) & \text{\\( f_0 = 0, f_1 = 1 \\) 代入} \\\\
\Longleftrightarrow F(x)(1 - x - x^2) &= x \\\\
\Longleftrightarrow F(x) &= \frac{x}{1 - x - x^2} \\\\
\end{aligned}

我們求出 \\( f \\) 的 OGF。多項式 \\( 1 - x - x^2 = 0 \\) 的兩個根分別為 \\( \phi_{+} = \frac{1 + \sqrt{5}}{-2} \\) 和 \\( \phi_{-} = \frac{1 - \sqrt{5}}{-2} \\)，因此 \\( F(x) = \frac{x}{(x - \phi_{+})(x - \phi_{-})}\\)，將 \\( F(x) \\) 以 partial fraction 展開就能求出一般項，這部分留給讀者自行練習。

回顧上面的過程，找線性遞迴關係式的一般項大致可以分成三個步驟：

1. 列出遞迴式和定義的範圍
2. 等號兩邊用 \\( \sum\limits_{n} \\) 將遞迴定義的範圍加起來並將每一項乘以 \\( x^n \\) (從數列轉換成 FPS)
3. 化簡算式求出 OGF，\\( a_n \\) 就是 \\( x^n \\) 的係數

再來看一道題目。

> 給定非線性遞迴關係式，滿足
> \begin{aligned}
\begin{cases}
a_0 &= 1 \\\\
a_n &= \sum\limits_{k = 1}^n k a_k a_{n - k} = a_1 a_{n - 1} + 2 a_2 a_{n - 2} + \dots + (n - 1) a_{n - 1} a_1 + n a_n a_0, n \in \mathbb{N} \\\\
\end{cases}
\end{aligned}
>
> 求數列的一般項

遞迴關係的等號右邊每一項看起來都很眼熟，實際上我們可以運用目前學會的操作將他分解。

分別定義 \\( b_0 = 0, b_n = \sum\limits_{k = 1}^n a_k a_{n - k} \\)，\\( A(x), B(x) \\) 分別為數列 \\( a, b \\) 的 OGF。可以發現 \\( b_n \\) 與卷積的定義只相差在 \\( \sum \\) 起始的 index，我們可以將 \\( k = 0 \\) 的 case 單獨扣掉，得到：

\begin{aligned}
b_n &= \sum\limits_{k = 1}^n a_k a_{n - k} \\\\
&= \sum\limits_{k = 0}^n a_k a_{n - k} - a_0 a_n & \text{將 \\( k = 0 \\) 的 case 單獨扣掉} \\\\
\Longleftrightarrow \sum_{n = 1}^{\infty} b_n x^n &= \sum\limits_{n = 1}^{\infty} (\sum\limits_{k = 0}^n a_k a_{n - k} - a_0 a_n) x^n & \text{兩邊同時乘上 \\( x^n \\) 把數列轉換成生成函數} \\\\
\Longleftrightarrow B(x) - b_0 x^0 &= \sum\limits_{n = 1}^{\infty} \sum\limits_{k = 0}^n a_k x^k a_{n - k} x^{n - k} - \sum\limits_{n = 1}^{\infty} a_0 a_n x^n & \text{等號左邊缺少 \\( b_0 \\) 項，右邊展開括號} \\\\
\Longleftrightarrow B(x) - b_0 x^0 &= (A(x)^2 - a_0^2) - \sum\limits_{n = 1}^{\infty} a_0 a_n x^n & \text{等號右邊的第一項其實就是 \\( A(x)^2 \\) 缺少 \\( n = 0 \\)，單獨減掉} \\\\
\Longleftrightarrow B(x) &= (A(x)^2 - 1) - \sum\limits_{n = 1}^{\infty} a_n x^n & \text{代入 \\( a_0 = 1, b_0 = 0 \\)} \\\\
&= (A(x)^2 - 1) - (A(x) - a_0 x_0) \\\\
&= A(x)^2 - A(x) \\\\
\end{aligned}

我們得到了數列 \\( b \\) 的生成函數。回到原本的題目，可以發現我們要求的 \\( a_n = n b_n \\)，可以透過先前提過的技巧 \\( A(x) = x B^{\prime}(x) \\) 求得。剩下的化簡留給讀者自行練習。

## 例題

> [CF 1832E - Combinatorics Problem](https://codeforces.com/contest/1832/problem/E)
>
> \\( b_i = (\sum\limits_{j = 0}^i \binom{i - j + 1}{k} \cdot a_j) \bmod 998244353 \\)，給定 \\( a_0, \dots, a_{n - 1} \\)，求 \\( b_0, \dots, b_{n - 1} \\)
>
> - \\( 1 \leq n \leq 10^7 \\)
> - \\( 1 \leq k \leq 5 \\)

本題有 \\( O(nk) \\) 的 DP 解。這邊介紹生成函數 \\( O(n \log n) \\) 的解法，可以做到 \\( k = n \\) 量級。

我們先將二項式係數拆開，得到

\\[
b_i = \sum\limits_{j = 0}^i \frac{(i - j + 1)!}{k!(i - j + 1 - k)!} a_j
\\]

按照之前的套路，將等號兩邊定義的範圍加總，並對每一項乘以 \\( x^n \\)，得到

\\[
\sum\limits_{n \geq 0} b_n x^n = \frac{1}{k!} \sum\limits_{n \geq 0} \sum\limits_{j = 0}^n \frac{(n - j + 1)!}{(n - j + 1 - k)!} x^{n - j} a_j x^j
\\]

可以發現等號右邊可以拆成兩個多項式的卷積，

\begin{aligned}
f &= \sum\limits_{n \geq 0} a_n x^n \\\\
g &= \sum\limits_{n \geq 0} \frac{(n + 1)!}{(n + 1 - k)!} x^n \\\\
\Longrightarrow \frac{1}{k!}(f \cdot g) &= \frac{1}{k!} \sum\limits_{n \geq 0} \sum\limits_{j = 0}^n \frac{(n - j + 1)!}{(n - j + 1 - k)!} x^{n - j} a_j x^j
\end{aligned}

我們可以用 NTT 快速計算 \\( f \\) 和 \\( g \\) 的卷積，\\( b_i \\) 就會是 OGF 的係數。

由於本題 \\( n \\) 很大，再加上 NTT 的複雜度帶 \\( \log \\)，可能要優化 NTT 才能 AC (本題時限 4s，筆者的 NTT 跑 3.3s)。

[Solution Code](https://codeforces.com/contest/1832/submission/211226494)

> [CF 438E - The Child and Binary Tree](https://codeforces.com/problemset/problem/438/E)
>
> 給定集合 \\( C \\)。令 \\( a_i \\) 為節點 \\( i \\) 的點權。對於 \\( S = 1, \dots, m \\)，求點權和為 \\( S \\)，且 \\(a_i \in C \\) 的二元樹數量 \\( \bmod 998244353 \\)。
>
> - \\( 1 \leq m \leq 10^5 \\)
> - \\( 1 \leq c_i \leq 10^5 \\)

定義 \\( f_s \\) 為權重和為 \\( s \\) 的合法二元樹數量，\\( f_0 = 1 \\)。枚舉二元樹樹根的權重 \\( w \\)，我們可以列出遞迴式，

\\[
f_s = \sum\limits_{w \in C} \sum\limits_{i = 0}^{s - w} f_i f_{s - w - i}, s \geq 1
\\]

按照套路，將等號兩邊定義的範圍加總，並對每一項乘以 \\( x^n \\)，得到

\begin{aligned}
\sum\limits_{n \geq 1} f_n x^n &= \sum\limits_{n \geq 1} \sum\limits_{w \in C} x^w \sum\limits_{i = 0}^{n - w} f_i x^i f_{n - w - i} x^{n - w - i} \\\\
\Longrightarrow F(x) - f_0 x^0 &= \sum\limits_{w \in C} x^w \sum\limits_{n \geq 1} \sum\limits_{i = 0}^{n - w} f_i x^i f_{n - w - i} x^{n - w - i} \\\\
\end{aligned}

可以發現最右邊其實就是 \\( F(x)^2 \\)。我們定義 \\( C(x) = \sum\limits_{w \in C} x^w \\)，可以再化簡為

\begin{aligned}
F(x) - 1 &= C(x)F(x)^2 \\\\
\Longrightarrow F(x) &= \frac{1 \pm \sqrt{1 - 4C(x)}}{2C(x)} & \text{把 \\( F(x) \\) 當作變數解二元一次方程式}
\end{aligned}

由於 \\( f_0 = F(0) = 1 \\)，\\( F(x) \\) 在 \\( x = 0 \\) 要收斂至 \\( 1 \\)，因此要取負號，得到 \\( F(x) = \frac{1 - \sqrt{1 - 4C(x)}}{2C(x)} \\)。一般來說化簡到這裡就可以了，但是在 \\( \bmod 998244353 \\) 下，\\( C(x) \\) 的逆元 \\( C^-1(x) \\) 存在若且唯若 \\( C(x) \\) 的常數項不為 \\( 0 \\)[^note-1]。因此我們要對他有理化，

\begin{aligned}
F(x) &= \frac{1 - \sqrt{1 - 4C(x)}}{2C(x)} \\\\
&= \frac{(1 - \sqrt{1 - 4C(x)})(1 + \sqrt{1 - 4C(x)})}{2C(x)(1 + \sqrt{1 - 4C(x)})} \\\\
&= \frac{1^2 - (1 - 4C(x))}{2C(x)(1 + \sqrt{1 - 4C(x)})} \\\\
&= \frac{4C(x)}{2C(x)(1 + \sqrt{1 - 4C(x)})} \\\\
&= \frac{2}{1 + \sqrt{1 - 4C(x)}} \\\\
\end{aligned}

化簡到這裡就可以了，因為分母的常數項不為 \\( 0 \\)。對於 \\( S = k \\) 的答案就會是 \\( f_k = [x^k] F(x) \\)。

本題中對 FPS 做開根號和逆元的詳細做法可以參考這三篇文章 [1](https://codeforces.com/blog/entry/56422) [2](https://cp-algorithms.com/algebra/polynomial.html) [3](https://www.luogu.com.cn/blog/MoYuFang/sheng-cheng-han-shuo-di-shuo-xue-ji-chu)。

> [ABC222 H - Beautiful Binary Tree](https://atcoder.jp/contests/abc222/tasks/abc222_h)
>
> 每個節點上寫著一個數字 \\( h_i \\)。定義一個 degree 為 \\( n \\) 的漂亮二元樹為：
>
> - \\( h_i \in \text{\{ \\( 0, 1 \\) \}} \\)
> - 葉節點的 \\( h_i = 1 \\)
> - 執行以下的操作至多 \\( n - 1 \\) 次，使得根節點為 \\( n \\) 且其他節點為 \\( 0 \\)
>   - 選擇兩個節點 \\( u, v \\)，滿足 \\( v \\) 是 \\( u \\) 的兒子或是孫子，更新為 \\( h_u \leftarrow h_u + h_v, h_v \leftarrow 0 \\)
> 求 degree 為 \\( n \\) 的漂亮二元樹數量 \\( \bmod 998244353 \\)
>
> - \\( 1 \leq n \leq 10^7 \\)，\\( O(n) \\)

觀察：

因為每次操作都要把一個 \\( h_i = 1 \\) 的往上移動，且經過 \\( n - 1 \\) 次操作後根節點要為 \\( n \\)，因此根節點的初始 \\( h_i = 1 \\)。又因為每次往上移動時最多可以跳過一個節點 \\( (u, v) \\) 可以為父子或爺孫關係)，因此樹上不能有相鄰的 \\( 0 \\)，否則無法在 \\( N - 1 \\) 次內完成。

有了上述的觀察，我們可以列出 DP 式：

- \\( a_i \\)：degree 為 \\( i \\) 的漂亮二元樹數量。
- \\( b_i \\)：degree 為 \\( i \\) 且 root 為 \\( 0 \\) 的漂亮二元樹數量。

列出關係式：

\begin{aligned}
b_i = 2 a_i + \sum\limits_{j = 1}^{i - 1} a_j a_{i - j}
\end{aligned}

\\( 2 a_i \\) 為只有一個子孫的情況，可以為左或右子樹。\\( \sum\limits_{j = 1}^{i - 1} a_j a_{i - j} \\) 為枚舉左子樹的 degree。因為不能有相鄰的 \\( 0 \\)，因此不能接上 \\( b \\) 類型的根作為子樹。

定義 \\( a_0 = b_0 = 0 \\)，將 \\( a, b \\) 轉換為 OGF \\( A(x), B(x) \\) 得到

\\[
B(x) = 2A(x) + A(x)^2
\\]

同樣 \\( a_i \\) 也可以列出遞迴式：

\begin{cases}
a_1 = 1 & \text{base case} \\\\
a_i = 2 (a_{i - 1} + b_{i - 1}) + \sum\limits_{j = 1}^{i - 1} (a_j + b_j)(a_{i - 1 - j} + b_{i - 1 - j}), (i > 1) & \text{根節點為 \\( 1 \\)，因此子樹 degree 加起來為 \\( i - 1 \\)} \\\\
\end{cases}

因此，

\begin{aligned}
A(x) - a_0 x^0 - a_1 x^1 &= 2x(A(x) + B(x)) + x(A(x) + B(x))^2 & \text{按照套路轉成 OGF} \\\\
\Longleftrightarrow A(x) &= x + 2x(A(x) + B(x)) + x(A(x) + B(x))^2 & \text{代入 \\( a_0 = 0, a_1 = 1 \\)} \\\\
&= x(1 + A(x) + B(x))^2 \\\\
&= x(1 + 3A(x) + A(x)^2)^2 & \text{代入 \\( B(x) \\)} \\\\
\end{aligned}

到這邊就能夠在 \\( O(n \log n) \\) 求出了。但因為題目要求 \\( O(n) \\)，我們需要進一步化簡。

根據 [Lagrange Inversion Theorem](lagrange_inversion_theorem.md)，我們定義 \\( F(x) = A(x) = x(1 + 3F(x) + F(x)^2)^2 \\)，因此 \\( \frac{F(x)}{(1 + 3F(x) + F(x)^2)^2} = x \\)。不難看出滿足 \\( G(F(x)) = x \\) 的 \\( G(x) = \frac{x}{(1 + 3x + x^2)^2} \\)。根據定理，我們可以列出

\begin{aligned}
\text{[} x^n \text{]} F(x) &= \frac{1}{n} [x^{n - 1}] (\frac{x}{G(x)})^n \\\\
&= \frac{1}{n} [x^{n - 1}] (1 + 3x + x^2)^{2n} & \text{代入 \\( G(x) \\)} \\\\
&= \frac{1}{n} [x^{n - 1}] ((1 + 3x) + x^2)^{2n} \\\\
&= \frac{1}{n} [x^{n - 1}] \sum\limits_{i = 0}^{2n} \binom{2n}{i} (1 + 3x)^{2n - i} (x^2)^{i} & \text{二項式定理} \\\\
&= \frac{1}{n} \sum\limits_{i = 0}^{2n} \binom{2n}{i} [x^{n - 1 - 2i}] (1 + 3x)^{2n - i} \\\\
&= \frac{1}{n} \sum\limits_{i = 0}^{2n} \binom{2n}{i} [x^{n - 1 - 2i}] \sum\limits_{j = 0}^{2n - i} \binom{2n - i}{j} (3x)^j 1^{2n - i - j} & \text{二項式定理} \\\\
&= \frac{1}{n} \sum\limits_{i = 0}^{2n} \binom{2n}{i} \binom{2n - i}{n - 1 - 2i} 3^{n - 1 - 2i} \\\\
\end{aligned}

因此我們可以在 \\( O(n) \\) 求出 \\( [x^n] F(x) = [x^n] A(x) = a_n \\)。

[^note-1]: 假設 \\( A(x)B(x) = 1 \\)，我們稱 \\( B(x) \\) 為 \\( A(x) \\) 的乘法逆元，記做 \\( B(x) = \frac{1}{A(x)} = A(x)^{-1} = A^{-1}(x) \\)。
定義 \\( C(x) = 1 = A(x)B(x) = \sum\limits_{n = 0}^{\infty} (\sum\limits_{i + j = n} a_i b_j) x^n \\)。因此如果 \\( a_0 = 0 \\)，則 \\( [x^0] C(x) = a_0 b_0 = 0 \\)，\\( A(x) \\) 的逆元不存在。

## References

- [Tutorial] Generating Functions in Competitive Programming [Part 1](https://codeforces.com/blog/entry/77468) [Part 2](https://codeforces.com/blog/entry/77551)
- FPS 相關運算，有興趣可以自行研究
  - [CF - Operations on Formal Power Series](https://codeforces.com/blog/entry/56422)
  - [CP algorithms - Operations on polynomials and series](https://cp-algorithms.com/algebra/polynomial.html)
  - [生成函数的数学基础](https://www.luogu.com.cn/blog/MoYuFang/sheng-cheng-han-shuo-di-shuo-xue-ji-chu)
- [generatingfunctionology](https://www2.math.upenn.edu/~wilf/gfologyLinked2.pdf)
- [non-linear-recursion-generating-functions](https://math.stackexchange.com/questions/1262413/non-linear-recursion-generating-functions)
- [Lagrange Inversion Theorem](https://atcoder.jp/contests/abc222/editorial/2765)
