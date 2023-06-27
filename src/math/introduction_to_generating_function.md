# Generating Function

## 介紹

Generating function 在競程比賽中有許多用途，包含計算排列數、組合數，也可以解數列的一般項。計算過程中雖然會涉及許多多項式的運算，但也不用因爲數學不好而卻步，可以把多項式運算相關的模板當黑盒使用，將重點放在把問題轉換成手邊工具能夠解決的。

## 基本定義

生成函數是藉由冪級數 (Power Series) 的係數來代表一個數列。在比賽中比較常見的生成函數類型為一般生成函數 (Ordinary Generating Function, OGF) 和指數生成函數 (Exponential Generating Function, EGF)。

對於數列 \\( a_0, a_1, \dots \\)，我們定義他的 OGF 為 \\( A(x) = \sum\limits_{i = 0}^{\infty} a_i x^i \\)，也就是把數列的第 \\( i \\) 項放到 \\( x^i \\) 的係數。舉例來說，費氏數列 (\\( f = \{0, 1, 1, 2, 3, 5, 8, \dots\} \\)) 的 OGF 為 \\( F(x) = 0 + x + x^2 + 2x^3 + 3x^4 + 5x^5 + 8x^6 \dots \\)。

EGF 的定義與 OGF 類似，只是每一項多除以 \\( i! \\)，也就是 \\( A(x) = \sum\limits_{i = 0}^{\infty} a_i \frac{x^i}{i!} \\)。在計算組合數的時候，EGF 的 \\( \frac{1}{i!} \\) 係數會發揮很大的作用。

在生成函數中，我們關注的點是數列與生成函數之間的關係，也就是無窮級數的係數，因此如果我們忽略 \\( x \\) 的數值，只作為形式存在，那麼我們就可以利用他解決一些運算的問題。因為無窮級數難以計算，我們可以先把他變成有限的函數，完成運算後再把結果回復到無窮級數，寫成流程圖大概長這樣：

\\[
數列 \xrightarrow{\text{放在函數的係數}} 無窮級數 \xrightarrow{\text{通式}} 有限函數 \rightarrow 有限函數的運算 \xrightarrow{\text{展開}} 無窮級數 \xrightarrow{\text{觀察係數}} 新的數列
\\]

以數列 \\( a = \{1, 1, 1, \dots\} \\) 舉例，顯然 \\( a \\) 的 OGF 為 \\( A(x) = 1 + x + x^2 + \dots \\)，有限函數就是 \\( \frac{1}{1 - x} \\)。對 \\( \frac{1}{1 - x} \\) 進行運算就簡單了許多。如上面所說，因為我們只關注函數的係數，並沒有要實際代入數值，\\( x \\) 只作為形式存在，因此我們可以忽略函數的收斂與發散性。這種只以形式存在的冪級數叫做形式冪級數 (Formal Power Series, FPS)。將無限轉化為有限運算的操作可以參考[這篇](https://web.math.sinica.edu.tw/math_media/d222/22209.pdf)。

## Common series of OGF/EGF

在計算時，我們會希望能夠化簡為容易觀察係數的多項式。常見的生成函數有：

1. \\( \frac{1}{1 - x} = \sum\limits_{k = 0}^{\infty} x^k \\)，在 OGF 中 \\( a_k = 1 \\)，等比級數的公式，作為 FPS 我們可以忽略函數的收斂與發散性
2. \\( (1 + x)^n = \sum\limits_{k = 0}^n \binom{n}{k} x^k \\)，在 OGF 中 \\( a_k = \binom{n}{k} \\)，固定 \\( n \\) 的二項式係數
3. \\( \frac{1}{(1 - x)^{k + 1}} = \sum\limits_{n = 0}^{\infty} \binom{n + k}{k} x^n \\)，\\( a_n = \binom{n + k}{k} \\)，固定 \\( k \\) 的二項式係數
4. \\( e^x = \sum\limits_{k = 0}^{\infty} \frac{x^k}{k!} \\)，在 EGF 中 \\( a_k = 1 \\)，指數函數的泰勒展開式

第三個或許沒有那麼直觀，我們會在後面證明。

## 符號

為了方便，我們先定義一些符號：

- \\( [x^n] A(x) \\)：\\( A(x) \\) 的 \\( x^n \\) 項係數。以下是一些使用範例：

\\[
[x^2] (1 + 7x + 5x^2) = 5 \\\\
[x^2 y^3] (2x^2 + 5x^2 y^3 + 3xy^4) = 5 \\\\
[x^1] (1 + x^2) = 0 \\\\
[x^k] (x^a f(x)) = [x^{k - a}] f(x) \\\\
\\]

- \\( \deg A \\)：\\( A(x) \\) 的最高次方數。\\( \deg (5 + 11x^2 + 6x^3) = 3 \\)

- \\( \binom{n}{k} = \frac{n!}{k!(n - k)!} = \frac{n \cdot (n - 1) \cdots (n - k + 1)}{k!} \\)

在這裡的二項式係數的 \\( n \\) 可以為虛數，但 \\( k \\) 必須是 \\( \geq 0 \\) 的整數。這叫做廣義二項式定理。

## Operations on Genertaing Function

生成函數作為數列的另一種表達方式，我們可以透過改變 GF 來改變其代表的數列。在以下的範例中，我們希望透過數列 \\( a \\) 和 \\( b \\) 構造出數列 \\( c \\)。
定義 \\( a_i, b_i, c_i \\) 分別為數列 \\( a, b, c \\) 中的第 \\( i \\) 項，\\( A(x), B(x), C(x) \\) 分別為數列 \\( a, b, c \\) 的生成函數。

### 加法 (\\( c_n = a_n + b_n \\))

對於 OGF 和 EGF，\\( C(x) = A(x) + B(x) \\) 會產生數列 \\( c_n = a_n + b_n \\)，證明為比對係數。

### 平移 (\\( c_n = a_{n \pm k} \\))

- \\( c_n = a_{n + k} \\)：把 \\( a \\) 的前面 \\( k \\) 項移除。
  - OGF：\begin{aligned}
C(x) &= a_k x^0 + a_{k + 1} x^1 + \dots \\\\
\Longleftrightarrow x^k C(x) &= a_k x^k + a_{k + 1} x^{k + 1} & \text{等號兩邊同時乘以 \\( x^k \\)} \\\\
&= \sum\limits_{i = k}^{\infty} a_i x^i \\\\
&= A(x) - \sum\limits_{i = 0}^{k - 1} a_i x^i & \text{相當於 A(x) 缺少最前面的 \\( k \\) 項} \\\\
\Longleftrightarrow C(x) &= \frac{A(x) - \sum\limits_{i = 0}^{k - 1} a_i x^i}{x^k} \\\\
\end{aligned}

  - EGF：\begin{aligned}
A(x) &= \sum\limits_{i = 0}^{\infty} a_i \frac{x^i}{i!} \\\\
\Longleftrightarrow A^{\prime}(x) &= a_0 \cdot 0 \cdot \frac{x^{0 - 1}}{0!} + a_1 \cdot 1 \cdot \frac{x^{1 - 1}}{1!} + a_2 \cdot 2 \cdot \frac{x^{2 - 1}}{2!} + \dots & \text{微分的定義} \\\\
&= a_1 \frac{x^0}{0!} + a_2 \frac{x^1}{1!} + a_3 \frac{x^2}{2!} \\\\
&= \sum\limits_{i = 0}^{\infty} a_{i + 1} \frac{x^i}{i!} & \text{微分 \\( 1 \\) 次後數列向左平移 \\( 1 \\) 項} \\\\
\Longleftrightarrow A^{(k)}(x) &= \sum\limits_{i = 0}^{\infty} a_{i + k} \frac{x^i}{i!} = C(x) & \text{微分 \\( k \\) 次就會是數列向左平移 \\( k \\) 項} \\\\
\end{aligned}

- \\( c_n = a_{n - k} \\)：把 \\( a \\) 往後移動 \\( k \\) 項，前面用 \\( 0 \\) 補齊。對於 OGF 來說 \\( C(x) = x^k A(x) \\)，EGF 則是透過積分 \\( k \\) 次來對齊階乘的係數，過程與上述的微分大同小異。

### 每項乘以 \\( n \\) (\\( c_n = n a_n \\))

係數的 \\( n \\) 可以透過微分拿走 \\( x^n \\) 的次方乘到係數，但次方會減少 \\( 1 \\)，所以需要再乘以 \\( x \\)。因此 \\( C(x) = x A^{\prime}(x) \\)。這對於 OGF 和 EGF 都相同。

### 卷積 (\\( C(x) = A(x)B(x) \\))

根據卷積的定義，我們可以列出 \\( [x^n]C(x) = \sum\limits_{k = 0}^n [x^k]A(x) \cdot [x^{n - k}]B(x)\\)。

因此對於 OGF，\\( c_n = \sum\limits_{k = 0}^n a_k b_{n - k} \\)，這與普通的多項式乘法相同。

對於 EGF，

\begin{aligned}
\frac{c_n}{n!} &= \sum\limits_{k = 0}^n \frac{a_k}{k!} \frac{b_{n - k}}{(n - k)!} \\\\
&= \sum\limits_{k = 0}^n \frac{1}{k! (n - k)!} a_k b_{n - k} \\\\
\end{aligned}

把 \\( n! \\) 移項到右邊後得到

\begin{aligned}
c_n &= \sum\limits_{k = 0}^n \frac{n!}{k! (n - k)!} a_k b_{n - k} \\\\
&= \sum\limits_{k = 0}^n \binom{n}{k} a_k b_{n - k}
\end{aligned}

利用這個性質，EGF 適合用來處理有二項式係數的遞迴關係式。

### 前綴和 (\\( c_n = \sum\limits_{i = 0}^n a_i \\))

這個技巧只對 OGF 有用。令 \\(C(x) = \frac{1}{1 - x} A(x) \\)。我們把 \\( \frac{1}{1 - x} \\) 以等比級數形式展開，得到 \\( C(x) = (1 + x + x^2 + x^3 + \cdots)A(x) \\)。因為右邊是多項式相乘，我們可以把他展開，然後觀察 \\( [x^n] \\) 項的係數，得到

\begin{aligned}
\text{[} x^n \text{]} C(x) &= [x^n] (1 + x + x^2 + \cdots)A(x) \\\\
\Longrightarrow c_n &= [x^n] \sum\limits_{n = 0}^{\infty} \sum\limits_{k = 0}^n x^i (a_{n - i} x^{n - i}) \\\\
\Longrightarrow c_n &= [x^n] \sum\limits_{n = 0}^{\infty} \sum\limits_{i = 0}^n a_{n - i} x^n \\\\
\Longrightarrow c_n &= \sum\limits_{i = 0}^n a_{n - i} = \sum\limits_{i = 0}^n a_i \\\\
\end{aligned}

因此數列 \\( c \\) 為數列 \\( a \\) 的前綴和。

## References

- [Tutorial] Generating Functions in Competitive Programming [Part 1](https://codeforces.com/blog/entry/77468) [Part 2](https://codeforces.com/blog/entry/77551)
- [等比級數](https://web.math.sinica.edu.tw/math_media/d222/22209.pdf)
