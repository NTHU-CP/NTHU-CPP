# Generating Function

## 介紹

生成函數是藉由冪級數 (Power Series) 的係數來代表一個數列。在比賽中比較常見的生成函數類型為一般生成函數 (Ordinary Generating Function, OGF) 和指數生成函數 (Exponential Generating Function, EGF)。

對於數列 \\( a_0, a_1, \dots \\)，我們定義他的 OGF 為 \\( A(x) = \sum_{i = 0}^{\infty} a_i x^i \\)，也就是把數列的第 \\( i \\) 項放到 \\( x^i \\) 的係數。舉例來說，費氏數列 (\\( f = \{0, 1, 1, 2, 3, 5, 8, \dots\} \\)) 的 OGF 為 \\( F(x) = 0 + x + 1x^2 + 2x^3 + 3x^4 + 5x^5 + 8x^6 \dots \\)。

EGF 的定義與 OGF 類似，只是每一項多除以 \\( i! \\)，也就是 \\( A(x) = \sum_{i = 0}^{\infty} \frac{a_i}{i!} x^i \\)。在計算組合數的時候，EGF 的 \\( \frac{1}{i!} \\) 係數會發揮很大的作用。

現在來看看數列 \\( a = \{1, 1, 1, \dots\} \\)，顯然 \\( a \\) 的 OGF 為 \\( A(x) = 1 + x + x^2 + \dots \\)。根據等比級數的公式 \\( A(x) = \frac{1}{1 - x} \\)，在 \\( |x| \geq 1 \\) 時發散。然而在生成函數中我們大多只在乎多項式和數列之間的關係，並不會代入數值，因此在大部分的情況下可以忽略函數的收斂與發散性，但這也代表函數不能任意代入數值。這種不考慮收斂性的冪級數叫做形式冪級數 (Formal Power Series, FPS)。

## Common series of OGF/EGF

在計算時，我們會希望能夠化簡為容易觀察係數的多項式。常見的生成函數有：

1. \\( \frac{1}{1 - x} = \sum_{k = 0}^{\infty} x^k \\)，在 OGF 中 \\( a_k = 1 \\)，等比級數的公式
2. \\( (1 + x)^n = \sum_{k = 0}^n \binom{n}{k} x^k \\)，在 OGF 中 \\( a_k = \binom{n}{k} \\)，二項式定理
3. \\( \frac{1}{(1 - x)^k} = \sum_{n = 0}^{\infty} \binom{n + k - 1}{k - 1} x^n = \sum_{n = 0}^{\infty} \binom{n + k - 1}{n} x^n \\)，\\( a_k = \binom{n + k - 1}{k - 1} = \binom{n + k - 1}{n} \\)
4. \\( e^x = \sum_{k = 0}^{\infty} \frac{x^k}{k!} \\)，在 EGF 中 \\( a_k = 1 \\)，指數函數的泰勒展開式

第三個或許沒有那麼直觀，我們會在後面證明。

## 符號

為了方便，我們先定義一些符號：

\\( [x^n]A(x) \\)：\\( A(x) \\) 的 \\( x^n \\) 項係數。以下是一些使用範例：

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

如同一般的多項式，我們可以對 FPS 進行運算。這些運算與一般的多項式相同。細節和原理請參考 Reference 的文章。

- 五則運算：\\( f(x) \pm g(x), f(x)g(x), \frac{f(x)}{g(x)}, f(x) \text{%} g(x) \\)

- 微分：\\( f^\prime(x) \\)

- 積分：\\( \int f(x)dx \\)

- 逆元：\\( f(x)g(x) = 1 \\)，記 \\( g(x) = f^{-1}(x) \\)

- \\( \log \\)：\\( \log f(x) = \int \frac{f^\prime(x)}{f(x)} \\)

- \\( \exp \\)：\\( \exp f(x) = e^{f(x)} \\)

- 次方：\\( f^k(x) = e^{k \log f(x)} \\)

- 開根號：\\( \sqrt{f(x)} \\)

- 複合函數：\\( f(g(x)) \\)

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
2. 等號兩邊用 \\( \sum_{n} \\) 將遞迴定義的範圍加起來並將每一項乘以 \\( x^n \\) (從數列轉換成 FPS)
3. 化簡算式求出 OGF，\\( a_n \\) 就是 \\( x^n \\) 的係數

> [CF 1832E - Combinatorics Problem](https://codeforces.com/contest/1832/problem/E)
>
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

我們可以用 NTT 快速計算 \\( f \\) 和 \\( g \\) 的卷積，\\( b_i \\) 就會是 OGF 的係數。

> [CF 438E - The Child and Binary Tree](https://codeforces.com/problemset/problem/438/E)
>
> 對於 \\( S = 1, \dots, m \\)，求點權和為 \\( S \\)，且所有點權 \\( \in C \\) 的二元樹數量 \\( \bmod 998244353 \\)。
> 
> - \\( 1 \leq m \leq 10^5 \\)
> - \\( 1 \leq c_i \leq 10^5 \\)

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

## Combination Problem with GF

生成函數也可以用來解組合數問題。

> \\( f(x) \\) 為用 \\( 1, 5, 10, 50 \\) 元硬幣湊出 \\( x \\) 元的方法數。求 \\( f(x) \\) 的 OGF \\( F(x) \\)。

\\[
F(x) = (1 + x + x^2 + \cdots)(1 + x^5 + x^{10} + \cdots)(1 + x^{10} + x^{20} + \cdots)(1 + x^{50} + x^{100} + \cdots) \\\\
= \frac{1}{1 - x} \frac{1}{1 - x^5} \frac{1}{1 - x^{10}} \frac{1}{1 - x^{50}} \\\\
= \frac{1}{(1 - x)(1 - x^5)(1 - x^{10})(1 - x^{50})} \\\\
\\]

等號右邊每個多項式的 \\( [x^k] \\) 項係數代表使用該種硬幣湊出 \\( k \\) 元的方法數，因此只有在次方為幣值倍數的情況係數會是 \\( 1 \\)。根據 OGF 卷積的定義，\\( [x^n]F(x) = \sum_{a + b + c + d = n} [x^a]\frac{1}{1 - x} [x^b]\frac{1}{1 - x^5} [x^c]\frac{1}{1 - x^{10}} [x^d]\frac{1}{1 - x^{50}} \\)，也就會是使用這 \\( 4 \\) 種硬幣湊出 \\( n \\) 元的方法數。

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

定義 EGF \\( f(x) = \sum_{k = 0}^{N} [k + 1 \in S] \frac{x^k}{k!}\\)，其中 \\( [\text{proposition}] \\) 為[艾佛森括號](https://zh.wikipedia.org/zh-tw/%E8%89%BE%E4%BD%9B%E6%A3%AE%E6%8B%AC%E5%8F%B7)。

令 \\( g_k \\) 為長度為 \\( k \\) 的合法數列數量。我們要求 \\( g \\) 的 EGF，可以列出

\\[ [x^k]g(x) = \sum_{cnt_1 + cnt_2 + \dots + cnt_n = k} [x^{cnt_1}]f(x) \cdot [x^{cnt_2}]f(x) \cdots [x^{cnt_n}]f(x) \\]

不難看出 \\( g(x) = f^N(x) \\)，又因為 EGF \\( [x^k]g(x) = \frac{g_k}{k!} \\)，\\( g_k = k! \cdot [x^k]f^N(x) \\)。

因此答案就是 \\( g_{N - 2} = (N - 2)! \cdot [x^{N - 2}]f^N(x) \\)。

計算 \\( f^N(x) \\) 可以用快速冪 \\( O(N \log^2 N) \\) 或是 \\( e^{N \log f(x)} \\) 做到 \\( O(N \log N) \\)。

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
f_n(k) &= [x^n] F(x, y) \\\\
&= [x^n] \frac{1}{1 - x - xy} \\\\
&= [x^n] \frac{1}{1 - x(1 + y)} \\\\
&= [x^n] \sum_{n = 0}^{\infty} (1 + y)^n x^n \\\\
&= (1 + y)^n \\\\
&= \sum_{k = 0}^n \binom{n}{k} y^k
\end{aligned}

這與二項式定理告訴我們的結論相同。

再比方說固定 \\( k \\)，也就是求 \\( f_k(n) = \binom{n}{k} \\) 的 OGF，可以列出

\begin{aligned}
f_k(n) &= [y^k] F(x, y) \\\\
&= [y^k] \frac{1}{1 - x - xy} \\\\
&= [y^k] \frac{1}{(1 - x) - xy} \\\\
&= [y^k] \frac{\frac{1}{1 - x}}{1 - \frac{xy}{1 - x}} \\\\
&= [y^k] \frac{1}{1 - x} \sum_{k = 0}^{\infty} (\frac{x}{1 - x})^k y^k \\\\
&= \frac{x^k}{(1 - x)^{k + 1}}
\end{aligned}

我們可以從 \\( f_k(n) \\) 的 OGF 得到一些性質，因為

\begin{aligned}
\binom{n}{k} &= [x^n] [y^k] F(x, y) \\\\
&= [x^n] \frac{x^k}{(1 - x)^{k + 1}} \\\\
&= [x^{n - k}] \frac{1}{(1 - x)^{k + 1}}
\end{aligned}

因此，\\( \binom{n + k - 1}{k - 1} = [x^n] \frac{1}{(1 - x)^k} \\)。

## References

- [[Tutorial] Generating Functions in Competitive Programming (Part 1)](https://codeforces.com/blog/entry/77468)
- [[Tutorial] Generating Functions in Competitive Programming (Part 2)](https://codeforces.com/blog/entry/77551)
- [CF - Operations on Formal Power Series](https://codeforces.com/blog/entry/56422)
- [CP algorithms - Operations on polynomials and series](https://cp-algorithms.com/algebra/polynomial.html)
- [生成函数的数学基础](https://www.luogu.com.cn/blog/MoYuFang/sheng-cheng-han-shuo-di-shuo-xue-ji-chu)
- [generatingfunctionology](https://www2.math.upenn.edu/~wilf/gfologyLinked2.pdf)
