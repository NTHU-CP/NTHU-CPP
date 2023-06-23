# 數論函數介紹

> 定義：若 \\(f:\mathbb{Z}\rightarrow\mathbb{C}\\)，則稱 \\(f\\) 是數論函數。

翻成白話就是，若一個函數的值域是整數（被定義在整數上）、對應域是複數，則其為數論函數。

舉例來說：

* 是否為質數：\\( p(n)=\cases{1 &: \\( n \\) 是質數 \\\\ 0 &: \\( n \\) 不是質數} \\)
* 歐拉函數：\\( \varphi(n)=n\text{以下與其互質的數字數量} \\)
* 因數和：\\( \sigma(n)=\sum_{d\mid n} d \\)
* GCD 函數：\\( f_{7122}(n) = \gcd(7122, n) \\)

## 積性函數

> 定義：若對於任意互質的 \\(n\\)、\\(m\\) 都有 \\(f(nm)=f(n)f(m)\\)，則稱 \\(f\\) 是積性的。

舉例來說：

> \\( \sigma(n) \\) 是積性的。

證明：

令 \\( n \\) 的質因數分解為 \\( p_1^{\alpha_1}p_2^{\alpha_1}\cdots p_m^{\alpha_m} \\)。

其因數和為 \\( \displaystyle\sigma(n)=\prod_{i=1}^m(1+p_i+\cdots +p_i^{\alpha_i}) \\)，考慮展開後的每一項即可知其正確性。

而對於任何質數的冪次 \\( p^\alpha \\)，也容易知道 \\( \sigma(p^\alpha)=1+p+\cdots + p^\alpha \\)。

因此，\\( \displaystyle\sigma(n)=\prod_{i=1}^m \sigma(p_i^{\alpha_i}) \\)，從這條式子可以知道 \\( \sigma \\) 符合積性函數的定義。

<p align="right">\( \blacksquare \)</p>

以上是一個由公式中可以看出其積性的例子。

> 推論：積性函數的取值可以由其在質數的冪次上的取值來求出。

寫成式子來說：\\( f(n)=f(p_1^{\alpha-1})f(p_2^{\alpha_2})\cdots f(p_m^{\alpha_m})=\prod_{i=1}^m f(p_i^{\alpha_i}) \\)。

上述的因數和函數就是其中一個例子。

> 定理：歐拉函數是積性的，也就是說若 \\( n \\)、\\( m \\) 互質，則 \\( \varphi(nm)=\varphi(n)\times\varphi(m) \\)。

當然，如果知道歐拉函數的公式，就可以輕易說明其為積性，但我們先忘掉公式。

證明：

將 \\( 1\sim nm \\) 擺成 \\( n\times m \\) 的樣子。

\\[
\left[\begin{array}{cccc}
1 & 2 & \cdots & m \\\\
m + 1 & m + 2 & \cdots & 2m \\\\
\vdots & \vdots & \ddots & \vdots \\\\
(n-1)m+1 & (n-1)m+2 & \cdots & nm
\end{array}\right]
\\]

注意到，每個橫排在模 \\( m \\) 之後，恰好會對應到 \\( 0\sim m-1 \\) 的每個數字。

所以對於第 \\( i \\)（\\( i=1\sim m \\)）個直排，若 \\( \gcd(i, m)\neq 1 \\)，則其上所有數字都不會和 \\( nm \\) 互質。

相反的，若 \\( \gcd(i, m)=1 \\)，則其上所有數字都和 \\( nm \\) 互質，這樣的直排共有 \\( \varphi(m) \\) 個。

另一方面因為 \\( \gcd(n,m)=1 \\)，因此 \\( m\times (0\sim n-1) \\) 在模 \\( n \\) 後會是 \\( 0\sim n-1 \\)。

所以每個直排在模 \\( n \\) 之後，恰好會對應到 \\( 0\sim n-1 \\) 的每個數字。

由此可知每個直排上恰有 \\( \varphi(n) \\) 個數字與 \\( n \\) 互質。

考慮到恰有 \\( \varphi(m) \\) 個直排與 \\( m \\) 互質，且這些直排上恰有 \\( \varphi(n) \\) 個數字與 \\( n \\) 互質。

綜合起來就是整個表格上恰有 \\( \varphi(m)\times\varphi(n) \\) 個數字與 \\( n\times m \\) 互質。

根據定義，這也就是 \\( \varphi(nm) \\)。

<p align="right">\( \blacksquare \)</p>

> 推論：歐拉函數 \\( \displaystyle\varphi(n)=n\times\prod_{i=1}^m \frac{p_i - 1}{p_i} \\)，其中 \\( p_i \\) 是 \\( n \\) 的質因數。

證明：

我們已經知道歐拉函數是積性的，因此只需要討論其在質數的冪次上的取值即可。

對於 \\( p^\alpha \\)，其因數只有 \\( p \\)，因此在其之下與其不互質的數字有 \\( p^{\alpha -1} \\) 個。

因此 \\( \varphi(p^\alpha)=p^\alpha -p^{\alpha - 1} \\)。

所以對於 \\( n=p_1^{\alpha_1}p_2^{\alpha_1}\cdots p_m^{\alpha_m} \\)，\\( \displaystyle\varphi(n)=\prod_{i=1}^m p_i^{\alpha_i} - p_i^{\alpha_i - 1}\\)。

整理後即可得 \\( \displaystyle\varphi(n) = n\times\prod_{i=1}^m \frac{p_i - 1}{p_i} \\)。

<p align="right">\( \blacksquare \)</p>

由此可知，若已知一個函數是積性的，那麼我們就可以藉由討論其在質數冪次上的取值來得到其公式。

## 許多數論函數的例子

> * 艾佛森括號 \\([P]=\cases{1&: If \(P\) is true \\\\ 0&: Otherwise}\\)
>
> * \\( \epsilon(n)=[n=1] \\)
>
> * \\( id(n)=n \\)
>
> * \\( 1(n)=1 \\)
>
> * 因數數量：\\( d(n)=\sum_{d\mid n} 1 \\)
>
> * 因數總和：\\( \sigma(n)=\sum_{d\mid n} d \\)
>
> * 歐拉函數：\\( \displaystyle\varphi(n) = n\times\prod_{i=1}^m \frac{p_i - 1}{p_i} \\)
>
> * 莫比烏斯函數：\\(\mu(n)=\cases{1&: \\(n = 1\\) \\\\ (-1)^k&: \\(n = p_1p_2\cdots p_k\\) \\\\ 0&: Otherwise}\\)。

## 結語

在這一章節中我們暫且點到為止，比較艱澀的內容會放在再探數論函數的章節中。
