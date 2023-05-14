# Generating Function

## 介紹

對於一個數列 \\( a_0, a_1, \dots \\)，我們定義他的一般生成函數 (Ordinary Generating Function, OGF) 為 \\( A(x) = \sum_{i = 0}^{\infty} a_i x^i \\)。舉例來說，費氏數列 (\\( f = \{0, 1, 1, 2, 3, 5, 8, \dots\} \\)) 的 OGF 為 \\( F(x) = 0 + x + 1x^2 + 2x^3 + 3x^4 + 5x^5 + 8x^6 \dots \\)。指數生成函數 (Exponential Generating Function, EGF) 的定義與 OGF 類似，只是每一項多除以 \\( i! \\)，也就是 \\( A(x) = \sum_{i = 0}^{\infty} \frac{a_i}{i!} x^i \\)。

現在來看看數列 \\( a = \{1, 1, 1, \dots\} \\)，顯然 \\( a \\) 的 OGF 為 \\( A(x) = 1 + x + x^2 + \dots \\)。根據等比級數的公式 \\( A(x) = \frac{1}{1 - x} \\)，在 \\( |x| \geq 1 \\) 時發散。然而在生成函數中我們大多只在乎多項式的係數，並不會代入數值，因此在大部分的情況下可以忽略函數的收斂與發散性，但這也代表函數不能任意代入數值。這種不考慮收斂性的無窮級數叫做形式冪級數 (Formal Power Series, FPS)。

## Common series of OGF/EGF

在計算時，我們會希望能夠把複雜的多項式化簡，變成容易觀察係數的多項式。常見的生成函數有：

\\[
\frac{1}{1 - x} = \sum_{n \geq 0} x^n \\\\
(1 + x)^n = \sum_{k = 0}^n \binom{n}{k} x^k \\\\
\frac{1}{(1 - x)^k} = \sum_{n \geq 0} \binom{n + k - 1}{k - 1} x^n = \sum_{n \geq 0} \binom{n + k - 1}{n} x^n \\\\
e^x = \sum_{n \geq 0} \frac{x^n}{n!}
\\]

第三個或許沒有那麼直觀，我們會在後面證明。

## 符號

為了方便，我們先定義一些符號：

\\( [x^n]A(x) \\)：\\( A(x) \\) 的 \\( x^n \\) 項係數。以下是一些範例：

\\[
[x^2](1 + 7x + 5x^2) = 5 \\\\
[x^2 y^3](2x^2 + 5x^2 y^3 + 3xy^4) = 5 \\\\
[x^1](1 + x^2) = 0 \\\\
[x^k](x^a f(x)) = [x^{k - a}]f(x) \\\\
\\]

\\( \deg A \\)：\\( A(x) \\) 的最高次方數。\\( \deg (5 + 11x^2 + 6x^3) = 3 \\)

\\( \binom{n}{k} = \frac{n!}{k!(n - k)!} = \frac{n \cdot (n - 1) \cdots (n - k + 1)}{k!} \\)

在這裡的二項式係數的 \\( n \\) 可以為虛數，但 \\( k \\) 必須是 \\( \geq 0 \\) 的整數。這叫做廣義二項式定理。

## Operations on Formal Power Series

如同一般的多項式，我們可以對 FPS 進行運算。這些運算與一般的多項式相同。細節和原理請參考其他篇文章，像是多項式乘法可以參考 FFT/NTT。

五則運算：\\( f(x) \pm g(x), f(x)g(x), \frac{f(x)}{g(x)}, f(x) \% g(x) \\)

微分：\\( f^\prime(x) \\)

積分：\\( \int f(x)dx \\)

逆元：\\( f(x)g(x) = 1 \\)，記 \\( g(x) = f^{-1}(x) \\)

\\( \log \\)：\\( \log f(x) = \int \frac{f^\prime(x)}{f(x)} \\)

\\( \exp \\)：\\( \exp f(x) = e^{f(x)} \\)

次方：\\( f^k(x) = e^{k \log f(x)} \\)

開根號：\\( \sqrt{f(x)} \\)

複合函數：\\( f(g(x)) \\)

## Operations on Genertaing Function

生成函數作為數列的另一種表達方式，我們可以透過改變 GF 來改變其代表的數列。

### 加法 (\\( c_n = a_n + b_n \\))

對於 OGF 和 EGF，\\( C(x) = A(x) + B(x) \\) 會產生數列 \\( c_n = a_n + b_n \\)，證明為比對係數。

### 平移 (\\( c_n = a_{n \pm k} \\))

\\( c_n = a_{n + k} \\)：把 \\( a \\) 的前面 \\( k \\) 項移除，因此在 OGF 中 \\( C(x) = \frac{A(x) - \sum_{i = 0}^{k - 1} a_i x^i}{x^k} \\)。EGF 則是直接微分 \\( k \\) 次，階乘的係數會在次方乘下來的時候對齊。

\\( c_n = a_{n - k} \\)：把 \\( a \\) 往後移動 \\( k \\) 項，前面用 \\( 0 \\) 補齊。對於 OGF 來說 \\( C(x) = x^k A(x) \\)，EGF 則是透過積分 \\( k \\) 次來對齊階乘的係數。

### 每項乘以 \\( n \\) (\\( c_n = n a_n \\))

係數的 \\( n \\) 可以透過微分拿走 \\( x^n \\) 的次方乘到係數，但次方會減少 \\( 1 \\)，所以需要再乘以 \\( x \\)。因此 \\( C(x) = x A^{\prime}(x) \\)。這對於 OGF 和 EGF 都相同。

### 卷積 (\\( C(x) = A(x)B(x) \\))

根據卷積的定義，我們可以列出 \\( [x^n]C(x) = \sum_{k = 0}^n [x^k]A(x) \cdot [x^{n - k}]B(x)\\)。

因此對於 OGF，\\( c_n = \sum_{k = 0}^n a_k b_{n - k} \\)，這與普通的多項式乘法相同。

對於 EGF，\\( \frac{c_n}{n!} = \sum_{k = 0}^n \frac{a_k}{k!} \frac{b_{n - k}}{(n - k)!} = \sum_{k = 0}^n \frac{1}{k! (n - k)!} a_k b_{n - k} \\)，把 \\( n! \\) 移到右邊後得到 \\( c_n = \sum_{k = 0}^n \frac{n!}{k! (n - k)!} a_k b_{n - k} = \sum_{k = 0}^n \binom{n}{k} a_k b_{n - k} \\)。利用這個性質，EGF 適合用來處理有二項式係數的遞迴關係式。

### 前綴和 (\\( c_n = \sum_{i = 0}^n a_i \\))

這個技巧只對 OGF 有用。令 \\(C(x) = \frac{1}{1 - x} A(x) \\)。我們把 \\( \frac{1}{1 - x} \\) 以等比級數展開，得到 \\( C(x) = (1 + x + x^2 + x^3 + \cdots)A(x) \\)。因為右邊是多項式相乘，我們可以把他展開，然後觀察 \\( [x^n] \\) 項的係數，得到

\begin{aligned}
[x^n]C(x) &= [x^n](1 + x + x^2 + \cdots)A(x) \\\\
\Longrightarrow c_n &= [x^n] \sum_{n = 0}^{\infty} \sum_{k = 0}^n x^i (a_{n - i} x^{n - i}) \\\\
\Longrightarrow c_n &= [x^n] \sum_{n = 0}^{\infty} \sum_{i = 0}^n a_{n - i} x^n \\\\
\Longrightarrow c_n &= \sum_{i = 0}^n a_{n - i} = \sum_{i = 0}^n a_i \\\\
\end{aligned}

故得證。

## Solving recurrence with GF

我們可以用生成函數解遞迴關係式的一般項。舉例來說，如果我們有一個數列

\begin{aligned}
a_0 &= 0 \\\\
a_{n + 1} &= 3 a_n + 2, n \geq 0 \\\\
\end{aligned}

把數列的前面幾項列出來，\\( a = 0, 2, 8, 26, 80, 242, \dots\\) 我們猜測數列的一般項有可能是 \\( 3^n - 1 \\)，但要怎麼證明呢？

要讓他變成多項式的形式，我們把等號兩邊同時乘以 \\( x^n \\)，並對定義的範圍做加總。

\begin{aligned}
\sum_{n = 0}^{\infty} a_{n + 1} x^n &= \sum_{n = 0}^{\infty} (3 a_n + 2) x^n \\\\
\Longrightarrow \frac{A(x) - a_0 x^0}{x} &= 3 \sum_{n = 0}^{\infty} a_n x^n + 2 \sum_{n = 0}^{\infty} x^n \\\\
\Longrightarrow \frac{A(x) - a_0 x^0}{x} &= 3A(x) + \frac{2}{1 - x}
\end{aligned}

將 \\( a_0 = 0 \\) 代入，

\begin{aligned}
\Longrightarrow A(x) &= 3xA(x) + \frac{2x}{1 - x} \\\\
\Longrightarrow (1 - 3x)A(x) &= \frac{2x}{1 - x} \\\\
\Longrightarrow A(x) &= \frac{2x}{(1 - x)(1 - 3x)}
\end{aligned}

因此，\\( A(x) = \frac{2x}{(1 - x)(1 - 3x)} \\) 為此數列的 OGF。找到 OGF 之後，我們可以求數列的一般項。先將 \\( A(x) \\) 寫成 partial fraction 的形式，

\\[
A(x) = 2x(\frac{C}{1 - x} + \frac{D}{1 - 3x})
\\]

解出 \\( C = -\frac{1}{2}, D = \frac{3}{2} \\)。
換成 partial fraction 形式的好處是他可以展開成等比級數，因此

\begin{aligned}
\Longrightarrow A(x) &= 2x(-\frac{1}{2} \sum_{n = 0}^{\infty} x^n + \frac{3}{2} \sum_{n = 0}^{\infty} 3^n x^n ) \\\\
\Longrightarrow A(x) &= 2x \sum_{n = 0}^{\infty} (-\frac{1}{2} + \frac{3^{n + 1}}{2}) x^n \\\\
\Longrightarrow A(x) &= 2x \sum_{n = 0}^{\infty} \frac{3^{n + 1} - 1}{2} x^n \\\\
\Longrightarrow A(x) &= \sum_{n = 0}^{\infty} (3^{n + 1} - 1) x^{n + 1} \\\\
\Longrightarrow A(x) &= \sum_{n = 1}^{\infty} (3^n - 1) x^n \\\\
\Longrightarrow A(x) &= 0 + (3^1 - 1) x^1 + (3^2 - 1) x^2 + (3^3 - 1) x^3 + \cdots \\\\
\Longrightarrow a_n &= [x^n]A(x) = 3^n - 1
\end{aligned}

因此我們證明了 \\( a_n = 3^n - 1 \\)。

回顧上面的過程，找遞迴關係式的一般項大致可以分成四個步驟：

1. 列出遞迴式和定義的範圍
2. 等號兩邊用 \\( \sum_{n} \\) 將遞迴定義的範圍加起來並將每一項乘以 \\( x^n \\)
3. 化簡算式求出 OGF
4. 將 OGF 寫成 partial fraction 的形式後變成等比級數和，\\( a_n \\) 就是 \\( x^n \\) 的係數

## Combination Problem with GF

生成函數也可以用來解組合數問題。

> \\( f(x) \\) 為用 \\( 1, 5, 10, 50 \\) 元硬幣湊出 \\( x \\) 元的方法數。求 \\( f(x) \\) 的 OGF \\( F(x) \\)。

\\[
F(x) = (1 + x + x^2 + \cdots)(1 + x^5 + x^{10} + \cdots)(1 + x^{10} + x^{20} + \cdots)(1 + x^{50} + x^{100} + \cdots) \\\\
= \frac{1}{1 - x} \frac{1}{1 - x^5} \frac{1}{1 - x^{10}} \frac{1}{1 - x^{50}} \\\\
= \frac{1}{(1 - x)(1 - x^5)(1 - x^{10})(1 - x^{50})} \\\\
\\]

等號右邊每個多項式的 \\( [x^k] \\) 項係數代表使用該種硬幣湊出 \\( k \\) 元的方法數，因此只有在次方為幣值倍數的情況係數會是 \\( 1 \\)。根據卷積的定義，\\( [x^n]F(x) = \sum_{a + b + c + d = n} [x^a]\frac{1}{1 - x} [x^b]\frac{1}{1 - x^5} [x^c]\frac{1}{1 - x^{10}} [x^d]\frac{1}{1 - x^{50}} \\)，也就會是使用這 \\( 4 \\) 種硬幣湊出 \\( n \\) 元的方法數。

## GF with multiple variables

對於多變數的函數，我們一樣可以用生成函數求一般項。以 \\( f(n, k) = \binom{n}{k} \\) 舉例，\\( f \\) 可以以遞迴的形式表達：

\\[
f(n, k) =
\begin{cases}
1, n \geq 0, k = 0 \\\\
0, n = 0, k \geq 1 \\\\
f(n - 1, k) + f(n - 1, k - 1), n, k \geq 1 \\\\
\end{cases}
\\]

一樣先對定義的範圍加總，並將每一項乘以 \\( x^n y^k \\)。

\\[
\sum_{n = 1}^{\infty} \sum_{k = 1}^{\infty} f(n, k) x^n y^k = \sum_{n = 1}^{\infty} \sum_{k = 1}^{\infty} (f(n - 1, k) + f(n - 1, k - 1)) x^n y^k
\\]

經過一些化簡後得到 \\( f(n, k) \\) 的 OGF 為 \\( F(x, y) = \frac{1}{1 - x - xy} \\)。我們可以觀察 OGF 在單一變數的變化。比如固定 \\( n \\)，也就是求 \\( f_n(k) = \binom{n}{k} \\) 的 OGF，就會是

\begin{aligned}
f_n(k) &= [x^n]F(x, y) \\\\
&= [x^n]\frac{1}{1 - x - xy} \\\\
&= [x^n]\frac{1}{1 - x(1 + y)} \\\\
&= [x^n]\sum_{n = 0}^{\infty} (1 + y)^n x^n \\\\
&= (1 + y)^n \\\\
&= \sum_{k = 0}^n \binom{n}{k} y^k
\end{aligned}

這與二項式定理告訴我們的結論相同。

再比方說固定 \\( k \\)，也就是求 \\( f_k(n) = \binom{n}{k} \\) 的 OGF，可以列出

\begin{aligned}
f_k(n) &= [y^k]F(x, y) \\\\
&= [y^k]\frac{1}{1 - x - xy} \\\\
&= [y^k]\frac{1}{(1 - x) - xy} \\\\
&= [y^k]\frac{\frac{1}{1 - x}}{1 - \frac{xy}{1 - x}} \\\\
&= [y^k]\frac{1}{1 - x} \sum_{k = 0}^{\infty} (\frac{x}{1 - x})^k y^k \\\\
&= \frac{x^k}{(1 - x)^{k + 1}}
\end{aligned}

我們可以從 \\( f_k(n) \\) 的 OGF 得到一些性質，因為

\begin{aligned}
\binom{n}{k} &= [x^n][y^k]F(x, y) \\\\
&= [x^n]\frac{x^k}{(1 - x)^{k + 1}} \\\\
&= [x^{n - k}]\frac{1}{(1 - x)^{k + 1}}
\end{aligned}

因此，\\( \binom{n + k - 1}{k - 1} = [x^n]\frac{1}{(1 - x)^k} \\)。這個性質會很常用到。

## Problems

> [CF 1832E - Combinatorics Problem](https://codeforces.com/contest/1832/problem/E)
> \\( b_i = (\sum_{j = 0}^i \binom{i - j + 1}{k} \cdot a_j) \bmod 998244353 \\)，給定 \\( a_0, \dots, a_{n - 1} \\)，求 \\( b_0, \dots, b_{n - 1} \\)

本題的範圍是 \\( n \leq 10^7, 1 \leq k \leq 5 \\)，因此有 DP 的做法，但是用生成函數可以做到 \\( k = n \\) 量級。

我們先將二項式係數拆開，得到

\\[
b_i = \sum_{j = 0}^i \frac{(i - j + 1)!}{k!(i - j + 1 - k)!} a_j
\\]

按照之前的套路，將等號兩邊定義的範圍加總，並對每一項乘以 \\( x^n \\)，得到

\\[
\sum_{n \geq 0} b_n x^n = \frac{1}{k!} \sum_{n \geq 0} \sum_{j = 0}^n \frac{(n - j + 1)!}{(n - j + 1 - k)!} x^{n - j} a_j x^j
\\]

可以發現等號右邊可以拆成兩個多項式的卷積，

\begin{aligned}
f &= \sum_{n \geq 0} a_n x^n \\\\
g &= \sum_{n \geq 0} \frac{(n + 1)!}{(n + 1 - k)!} x^n \\\\
\Longrightarrow \frac{1}{k!}(f \cdot g) &= \frac{1}{k!} \sum_{n \geq 0} \sum_{j = 0}^n \frac{(n - j + 1)!}{(n - j + 1 - k)!} x^{n - j} a_j x^j
\end{aligned}

我們可以用 NTT 快速計算 \\( f \\) 和 \\( g \\) 的卷積，\\( b_i \\) 就會是多項式係數。

> [CF 438E - The Child and Binary Tree](https://codeforces.com/problemset/problem/438/E)
> 對於 \\( S = 1, \dots, m \\)，求點權和為 \\( S \\)，且所有點權 \\( \in C \\) 的二元樹數量 \\( \bmod 998244353 \\)。
> \\( 1 \leq m \leq 10^5 \\)
> \\( 1 \leq c_i \leq 10^5 \\)

定義 \\( f_s \\) 為權重和為 \\( s \\) 的合法二元樹數量，\\( f_0 = 1 \\)。枚舉二元樹樹根的權重 \\( w \\)，我們可以列出遞迴式，

\\[
f_s = \sum_{w \in C} \sum_{i = 0}^{s - w} f_i f_{s - w - i}, s \geq 1
\\]

按照套路，將等號兩邊定義的範圍加總，並對每一項乘以 \\( x^n \\)，得到

\begin{aligned}
\sum_{n \geq 1} f_n x^n &= \sum_{n \geq 1} \sum_{w \in C} x^w \sum_{i = 0}^{n - w} f_i x^i f_{n - w - i} x^{n - w - i} \\\\
\Longrightarrow F(x) - f_0 x^0 &= \sum_{w \in C} x^w \sum_{n \geq 1} \sum_{i = 0}^{n - w} f_i x^i f_{n - w - i} x^{n - w - i} \\\\
\end{aligned}

可以發現最右邊其實就是 \\( F(x)^2 \\)。我們定義 \\( C(x) = \sum_{w \in C} x^w \\)，可以再化簡為

\begin{aligned}
F(x) - 1 &= C(x)F(x)^2 \\\\
\Longrightarrow F(x) &= \frac{1 \pm \sqrt{1 - 4C(x)}}{2C(x)}
\end{aligned}

由於 \\( f_0 = F(0) = 1 \\)，在 \\( x = 0 \\) 收斂，因此要取負號，得到 \\( F(x) = \frac{1 - \sqrt{1 - 4C(x)}}{2C(x)} \\)。但是由於 \\( C(x) \\) 的常數項為 \\( 0 \\)，\\( C^{-1}(x) \\) 不存在，因此我們要對他有理化，

\begin{aligned}
F(x) &= \frac{1 - \sqrt{1 - 4C(x)}}{2C(x)} \\\\
&= \frac{(1 - \sqrt{1 - 4C(x)})(1 + \sqrt{1 - 4C(x)})}{2C(x)(1 + \sqrt{1 - 4C(x)})} \\\\
&= \frac{1^2 - (1 - 4C(x))}{2C(x)(1 + \sqrt{1 - 4C(x)})} \\\\
&= \frac{4C(x)}{2C(x)(1 + \sqrt{1 - 4C(x)})} \\\\
&= \frac{2}{1 + \sqrt{1 - 4C(x)}} \\\\
\end{aligned}

化簡到這裡就可以了，因為分母的常數項不為 \\( 0 \\)。

## References

- https://codeforces.com/blog/entry/77468
- https://codeforces.com/blog/entry/77551
- https://codeforces.com/blog/entry/56422
- https://cp-algorithms.com/algebra/polynomial.html
- https://www.luogu.com.cn/blog/MoYuFang/sheng-cheng-han-shuo-di-shuo-xue-ji-chu
- https://www2.math.upenn.edu/~wilf/gfologyLinked2.pdf
