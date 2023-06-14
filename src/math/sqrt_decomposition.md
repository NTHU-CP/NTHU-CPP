# 數論分塊（整除分塊）

## 介紹

### 例題

> 求 \\( \displaystyle\sum_{i=1}^n \left\lfloor\frac n i\right\rfloor \\) 的值。（\\( n\le 10^{10} \\)）

如果一個一個求的話，很明顯會導致 TLE。

### 數論分塊

然而，我們先定義一個集合 \\( R(n) \\)：

> \\( \displaystyle R(n)=\left\\{\left\lfloor\frac n i\right\rfloor :\ i=1,2,\cdots, n\right\\} \\)

可以知道，在例題中所有會出現的「值」，都是 \\( R(n) \\) 中的某一個元素。

實際上，我們會有以下定理：

> \\( \displaystyle |R(n)| = \mathcal{O}(\sqrt{n}) \\)

證明：

對於 \\( i=1\cdots \left\lfloor\sqrt{n}\right\rfloor \\)，\\( \displaystyle\left\lfloor\frac n i\right\rfloor \\) 的取值顯然只有 \\( \mathcal{O}(\sqrt{n}) \\) 種。

對於 \\( i=\left\lfloor\sqrt{n}+1\right\rfloor\cdots n \\)，可以知道

\\[ \displaystyle 1\le\left\lfloor\frac n i\right\rfloor\le \left\lfloor\sqrt{n}\right\rfloor \\]

換言之取值也只有 \\( \mathcal{O}(\sqrt{n}) \\) 種。

兩者合併下來，可以知道 \\( \displaystyle\left\lfloor\frac n i\right\rfloor \\) 的取值只有 \\( \mathcal{O}(\sqrt{n}) \\) 種，也就是 \\( |R(n)|=\mathcal{O}(\sqrt{n}) \\)。

<p align="right">\( \blacksquare \)</p>

由此定理可知，上述例題的 \\( \sum \\) 中僅會有 \\( \mathcal{O}(\sqrt{n}) \\) 種不同的值。

因此可以換個角度想，枚舉每個數值出現的次數，最後將每個數值乘上出現次數後，即可得到答案。

舉例來說，\\( n = 10 \\) 的情況如下表所示：

| \\( i \\)                             | 1   | 2   | 3   | 4   | 5   | 6   | 7   | 8   | 9   | 10  |
| ------------------------------------- | --- | --- | --- | --- | --- | --- | --- | --- | --- |:---:|
| \\( \left\lfloor n/i\right\rfloor \\) | 10  | 5   | 3   | 2   | 2   | 1   | 1   | 1   | 1   |  1  |

因此，

\\[ \displaystyle\sum_{i=1}^{10} \left\lfloor\frac{10}{i}\right\rfloor = 10 + 5 + 3 + 2\times 2 + 1\times 5=27 \\]

### 右端點位置

觀察到，數值相同的數字會是連續的，我們稱呼極大的連續相同數字為一「塊」。

舉例來說：\\( [1, 1] \\)、\\( [2, 2] \\)、\\( [3, 3] \\)、\\( [4, 5] \\)、\\( [6, 10] \\) 各是一塊。

而我們會有以下定理：

> \\( i \\) 所在的塊（值為 \\( \left\lfloor\frac n i\right\rfloor \\) 的塊）的右端點為 \\( \left\lfloor\frac{n}{\left\lfloor\frac n i\right\rfloor}\right\rfloor \\)。

舉例來說：\\( n=10 \\) 的情況下，\\( 7 \\) 所在的塊的右端點是 \\( \left\lfloor\frac{10}{\left\lfloor\frac{10}{7}\right\rfloor}\right\rfloor=\left\lfloor\frac{10}{1}\right\rfloor=10 \\)。

證明：

給定 \\( i \\)，令 \\( \left\lfloor\frac n i\right\rfloor = k \\)。

現在我們要找一個最大的 \\( j \\) 使得 \\( \left\lfloor\frac n j\right\rfloor =k \\)。

\begin{align}
\left\lfloor\frac n j\right\rfloor =k &\Leftrightarrow k\le \frac n j< k+1 \\\\
&\Leftrightarrow \frac{n}{k+1}< j \le\frac{n}{k}
\end{align}

注意到 \\( i \\) 是滿足 \\( \left\lfloor\frac n i\right\rfloor =k \\) 的，因此至少有個正整數 \\( j \\) 滿足 \\( \frac{n}{k+1}< j \le\frac{n}{k} \\)。

取最後一式的右半部分可知 \\( j\le\frac{n}{k} \\)。

又因為 \\( j \\) 是正整數，因此分數的部分可以加上下高斯，由此得到 \\( j\le\left\lfloor\frac n k\right\rfloor \\)。

\\( j \\) 要盡量大，因此取 \\( j=\left\lfloor\frac n k\right\rfloor \\)，再將 \\( k=\left\lfloor\frac n i\right\rfloor \\) 代入即可得

\\[ \displaystyle j=\left\lfloor\frac{n}{\left\lfloor\frac n i\right\rfloor}\right\rfloor \\]

<p align="right">\( \blacksquare \)</p>

### 計算答案

有了這個定理，要計算 \\( \sum_{i=1}^n \left\lfloor\frac n i \right\rfloor \\)，就只需要從 \\( i = 1 \\) 開始，每次計算當前塊的右端點在哪。

統計完當前塊的答案後，再跳到右端點再往右一格，就能到達新的一塊。重複這樣的步驟即可。

Code 如下：

```cpp
for (int i = 1, r = 0; i <= n; i = r + 1) {
  r = n / (n / i);
  ans += 1LL * (r - i + 1) * (n / i);
}
```

由於不同數值只有 \\( \mathcal{O}(\sqrt{n}) \\) 個，而每個不同數值也只會被算到一次。

因此時間複雜度從 \\(\mathcal{O}(n)\\) 降為 \\(\mathcal{O}(\sqrt{n})\\)。

## 數論分塊的應用

### 捲積的前綴和

接下來我們來研究下方這種問題：

> 給定兩個函數 \\( f \\)、\\( g \\)，求 \\( \sum_{i=1}^n (f\ast g)(i) \\) 的值。

換句話說，給定兩個函數，要求其捲積的前綴和。

我們可以將式子做點轉換：

\begin{align}
\displaystyle\sum_{i=1}^n (f\ast g)(i) &= \sum_{i=1}^n \sum_{d\mid i} g(d) \times f\left(\frac n d\right) & \text{捲積的定義} \\\\
&= \sum_{d=1}^n \sum_{i=1}^{\lfloor\frac{n}{d}\rfloor} g(d)\times f(i) & \text{交換 sigma 的順序} \\\\
&= \sum_{d=1}^n g(d)\sum_{i=1}^{\lfloor\frac{n}{d}\rfloor} f(i) & g(d) \text{ 和 } i \text{ 無關，放到前面} \\\\
&= \sum_{d=1}^n g(d)\times S_f\left(\left\lfloor\frac{n}{d}\right\rfloor\right) & \text{假設 } S_f(j)=\sum_{i=1}^j f(i)
\end{align}

注意到 \\( S_f(\lfloor\frac{n}{d}\rfloor) \\) 的至多只有 \\( \mathcal{O}(\sqrt{n}) \\) 種不同取值。

令 \\( S_g(j)=\sum_{i=1}^j g(i) \\)。

我們知道，對於數論分塊的左端點 \\( i \\)，其右端點是 \\( \left\lfloor\frac{n}{\left\lfloor\frac n i\right\rfloor}\right\rfloor \\)。

因此：

\begin{align}
\displaystyle\sum_{j=i}^{\left\lfloor\frac{n}{\left\lfloor\frac n i\right\rfloor}\right\rfloor} g(j)\times S_f\left(\left\lfloor\frac{n}{j}\right\rfloor\right)
&= S_f\left(\left\lfloor\frac{n}{i}\right\rfloor\right)\times\sum_{j=1}^{\left\lfloor\frac{n}{\left\lfloor\frac n i\right\rfloor}\right\rfloor} g(j) \\\\
&= S_f\times\left(S_g\left( \left\lfloor\frac{n}{\left\lfloor\frac{n}{i}\right\rfloor}\right\rfloor \right) - S_g(i - 1)\right)
\end{align}

可以將 \\( f \\)、\\( g \\) 的前綴和進行預處理，這樣就能夠在 \\( \mathcal{O}(1) \\) 的時間內得到 \\( S_f \\) 和 \\( S_g \\) 了。

因此捲積的前綴和就能夠利用數論分塊的方法在 \\( \mathcal{O}(\sqrt{n}) \\) 的時間內算出。

Code 如下：

```cpp
for (int i = 1, r = 0; i <= n; i = r + 1) {
  r = n / (n / i);
  ans += 1LL * (Sg(r) - Sg(i - 1)) * Sf(n / i);
}
```

取 \\( f=g=1 \\)，就得到我們最一開始講的例題了。

另外，題目也可能會直接給轉換後的型式，也就是說：

> 給定 \\( f \\)、\\( g \\) 兩個函數。令 \\( S(n) = \sum_{i=1}^n f(i) \\)。
>
> 有 \\( T \\) 筆詢問。（\\( T\le 10^4 \\)）
>
> 每筆詢問給定 \\( n \\)，求 \\( \sum_{i=1}^n g(i)\times S\left(\left\lfloor\frac{n}{i}\right\rfloor\right) \\)。（\\( n\le 10^5 \\)）

舉例來說，\\( g = \mu \\)、\\( f(n) = n \\)，那麼：

\\[ \sum_{i=1}^n g(i)\times S\left(\left\lfloor\frac{n}{i}\right\rfloor\right) = \sum_{i=1}^n \mu(i)\times\frac{\left\lfloor\frac n i\right\rfloor\times\left( \left\lfloor\frac n i\right\rfloor +1 \right)}{2} \\]

只要先對 \\( \mu \\) 做線性篩預處理，即可在 \\( \mathcal{O}(\sqrt{n}) \\) 的時間內得到答案。

總時間複雜度為 \\( \mathcal{O}(C + T\sqrt{n}) \\)，其中 \\( C\le 10^5 \\)。

### 複合數論分塊

考慮下述問題：

> 有 \\( T\le 10^4 \\) 筆測資，每筆給定 \\( n \\)、\\( m \\)。
>
> 求 \\( \displaystyle\sum_{i=1}^m \left\lfloor\frac n i\right\rfloor\times\left\lfloor\frac m i\right\rfloor \\) 的值。（\\( n\le m\le 10^{5} \\)）

在此題中，如果遮住 \\( \left\lfloor\frac m i\right\rfloor \\) 或者 \\( \left\lfloor\frac n i\right\rfloor \\) 二者其一，就是我們熟悉的數論分塊的型式。

但現在我們必須同時考慮兩者。

首先我們知道 \\( i=n+1\sim m \\) 的部分因為 \\( \left\lfloor\frac n i\right\rfloor =0 \\) 而不必考慮。

接著讓我們先看個 \\( n=8 \\)、\\( m=10 \\) 例子。

| \\( i \\)                             | 1   | 2   | 3   | 4   | 5   | 6   | 7   | 8   |
| ------------------------------------- | --- | --- | --- | --- | --- | --- | --- |:---:|
| \\( \left\lfloor n/i\right\rfloor \\) | 8   | 4   | 2   | 2   | 1   | 1   | 1   | 1   |
| \\( \left\lfloor m/i\right\rfloor \\) | 10  | 5   | 3   | 2   | 2   | 1   | 1   | 1   |

延續前面我們對數論分塊中「塊」的定義，我們可以定義 \\( \left( \left\lfloor\frac n i\right\rfloor , \left\lfloor\frac m i\right\rfloor \right) \\) 相同的是一塊。

以上述例子來看，\\( [1, 1]、[2, 2]、[3, 3]、[4, 4]、[5, 5]、[6, 8] \\) 會是塊。

具體來說，我們知道單看 \\( \left\lfloor\frac n i\right\rfloor \\) 會有 \\( \mathcal{O}(\sqrt{n}) \\) 塊，單看 \\( \left\lfloor\frac m i\right\rfloor \\) 則會有 \\( \mathcal{O}(\sqrt{m}) \\) 塊。

把它們的端點的端點聯集起來，就會形成 \\( \mathcal{O}(\sqrt{n} + \sqrt{m}) \\) 塊。

根據數論分塊的精神，將同一塊的放在一起算即可。

而對於當前的左端點 \\( i \\) 要找右端點的時候，我們先分別找到對於 \\( n \\) 和對於 \\( m \\) 的右端點。

也就是 \\( \left\lfloor\frac{n}{\left\lfloor\frac n i\right\rfloor}\right\rfloor \\) 還有 \\( \left\lfloor\frac{m}{\left\lfloor\frac m i\right\rfloor}\right\rfloor \\)。

跳到其中比較近的那個即可。

```cpp
for (int i = 1, r = 0; i <= n; i = r + 1) {
  r = min(n / (n / i), m / (m / i));
  ans += 1LL * (r - i + 1) * (n / i) * (m / i);
}
```

## 練習題

* [Atcoder ABC 230 E](https://atcoder.jp/contests/abc230/tasks/abc230_e)
* [Codeforces 1263 C](https://codeforces.com/problemset/problem/1263/C)
* [Codefroces 616 E](https://codeforces.com/problemset/problem/616/E)
* [Codeforces 1780 E](https://codeforces.cc/contest/1780/problem/E)
  * 這題偏難
