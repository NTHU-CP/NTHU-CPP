# 再探數論函數

## Dirichlet 捲積

兩個數論函數可以透過捲積操作來產生新的函數。

> 定義：Dirichlet 捲積 \\(f\ast g(n)=\sum_{d\mid n}f(d)g(\dfrac n d)\\)。

Dirichlet 捲積滿足交換律、結合律、對於一般加法的分配律。

這些性質留給讀者自行驗證（交換律應該是顯然的）。

舉例來說：

> \\( d = 1\ast 1 \\)

換句話說，因數和函數是 \\( 1 \\) 函數與自身做捲積得到的。

寫成式子的話：

\\[ \displaystyle d(n)=\sum_{d\mid n} 1=\sum_{d\mid n}1(d)\times 1\left(\frac n d\right) \\]

> \\( \sigma = id\ast 1 \\)

寫成式子的話：

\\[ \displaystyle \sigma(n)=\sum_{d\mid n} d=\sum_{d\mid n}id(d)\times 1\left(\frac n d\right) \\]

## 引理

強烈推薦拿筆動手跟著推導一遍。

> \\( \displaystyle\sum_{d\mid n} f(d)=\sum_{d\mid n} f\left(\frac n d\right) \\)

<details><summary>證明</summary>

LHS 中的 \\( d \\) 會跑過所有 \\( n \\) 的所有因數。

RHS 中的 \\( \frac n d \\) 實際上也會跑過 \\( n \\) 的所有因數。

因此兩邊枚舉的數字的集合是一樣的，因此自然相等。

<p align="right">\( \blacksquare \)</p>

</details>

> \\( \displaystyle\sum_{d\mid n}\mu(d)=[n=1] \\)
>
> 也就是 \\( \mu\ast 1=\epsilon \\)

<details><summary>證明</summary>

根據莫比烏斯函數 \\( \mu \\) 的定義，只要 \\( n \\) 的因數中有平方數，則函數值為 \\( 0 \\)。

因此對於 \\( n=p_1^{\alpha_1}p_2^{\alpha_1}\cdots p_m^{\alpha_m} \\)，我們只需考慮 \\( \alpha_i = 1 \\) 的情況，也就是 \\( n=p_1\times p_2\cdots p_m \\)。

再根據 \\( \mu \\) 的定義，對於每個質數我們可以將其視作貢獻一個 \\( -1 \\)。

若是 \\( m \\) 個質數的連乘，由於每個質數貢獻一個 \\( -1 \\)，因此函數值就會是 \\( (-1)^m \\)。

對於 \\( \displaystyle\sum_{d\mid p_1 p_2\cdots p_m} \mu(d)\\)，我們分別考慮 \\( d \\) 是 \\( 0\sim m \\) 個質數的連乘的取法。

* \\( d \\) 是 \\( 0 \\) 個質數的連乘的取法有 \\( \binom{m}{0} \\) 種，函數值是 \\( (-1)^0 \\)。
* \\( d \\) 是 \\( 1 \\) 個質數的連乘的取法有 \\( \binom{m}{1} \\) 種，函數值是 \\( (-1)^1 \\)。
* \\( \vdots \\)
* \\( d \\) 是 \\( m \\) 個質數的連乘的取法有 \\( \binom{m}{m} \\) 種，函數值是 \\( (-1)^m \\)。

因此

\\[ \displaystyle\mu(n) = \sum_{d\mid n}\mu(d) = \sum_{i=0}^m \binom{m}{i}\times (-1)^i \\]

若 \\( m=0 \\)，則結果為 \\( 1 \\)；否則根據二項式定理，結果為 \\( 0 \\)。

<p align="right">\( \blacksquare \)</p>

</details>

> 莫比烏斯反演：
>
> \\( \displaystyle f(n)=\sum_{d\mid n}g(d)\Longleftrightarrow g(n)=\sum_{d\mid n}\mu\left(\frac n d\right)f(d) \\)
>
> 也就是 \\( f=g\ast 1\Leftrightarrow g=\mu\ast f \\)

<details><summary>證明一</summary>

已知左邊，推導右邊：

\begin{align}
\displaystyle\sum_{d\mid n}\mu\left(\frac n d\right)f(d) &= \sum_{d\mid n}\mu\left(\frac n d\right)\sum_{t\mid d}g(t) & \text{將 LHS 代入} \\\\
&=\sum_{t\mid n}\sum_{t\mid d\mid n} g(t)\times\mu\left(\frac n d\right) & \text{交換 } \sum \text{ 的順序} \\\\
&=\sum_{t\mid n}\sum_{d\mid \frac n t} g(t)\times\mu\left(\frac{n}{dt}\right) & \text{將枚舉的 } d \text{ 除 } t \text{，並在式子中補回去}\\\\
&=\sum_{t\mid n}g(t) \left(\sum_{d\mid \frac n t} \mu\left(\frac{\left(\frac{n}{t}\right)}{d}\right)\right) & g(t) \text{ 與 } d \text{無關，可以移到前面} \\\\
&=\sum_{t\mid n}g(t) \left(\sum_{d\mid \frac n t} \mu(d)\right) & \text{注意到括號裡就是枚舉 } \frac n t \text{ 的因數} \\\\
&=\sum_{t\mid n}g(t) \left[\frac{n}{t} =1\right] & \text{根據上一條引理} \\\\
&=g(n) & \text{根據定義可知只有 } t=n \text{ 時才有值}
\end{align}

注意到交換 \\( \sum \\) 的這一個步驟，此時的 \\( t \\)、\\( d \\)、\\( n \\) 之間滿足 \\( t\mid d\mid n \\)。

因此在已知 \\( n \\) 的條件下，

* 先枚舉 \\( d \\) 再枚舉 \\( d \\) 的因數作為 \\( t \\)
* 先枚舉 \\( t \\) 再枚舉其小於等於 \\( n \\) 的倍數作為 \\( d \\)

兩種枚舉方法其實是一樣的。

已知右邊，推導左邊：

\begin{align}
\displaystyle\sum_{d\mid n}g(d)&=\sum_{d\mid n}\sum_{t\mid d}\mu\left(\frac d t\right)f(t) & \text{將 RHS 代入} \\\\
&= \sum_{t\mid n}\sum_{t\mid d\mid n} f(t)\times\mu\left(\frac d t\right) & \text{交換 } \sum \text{ 的順序} \\\\
&= \sum_{t\mid n}\sum_{d\mid\frac n t} f(t)\times\mu(d) & \text{將枚舉的 } d \text{ 除 } t \text{，並在式子中補回去}\\\\
&= \sum_{t\mid n}f(t)\times\left( \sum_{d\mid\frac n t}\mu(d) \right) & f(t) \text{與 } d \text{無關，可以移到前面} \\\\
&= \sum_{t\mid n}f(t)\times \left[\frac{n}{t} =1\right] & \text{根據上一條引裡} \\\\
&= f(n) & \text{根據定義可知只有 } t=n \text{ 時才有值}
\end{align}

至此兩個方向可以互相推導，因此證明完畢。

<p align="right">\( \blacksquare \)</p>

</details>

<details><summary>證明二</summary>

以捲積的角度來看的話，將 LHS 的兩邊對 \\( \mu \\) 做捲積：

\\[ \mu\ast f=g\ast 1\ast\mu=g\ast\epsilon=g \\]

將 RHS 的兩邊對 \\( 1 \\) 做捲積：

 \\[ g\ast 1=\mu\ast 1\ast f=\epsilon\ast f=f \\]

因此兩個方向能夠互相推導，得證。

<p align="right">\( \blacksquare \)</p>

> \\( \displaystyle f(n)=\sum_{n\mid d}g(d)\Longleftrightarrow g(n)=\sum_{n\mid d}\mu\left(\frac d n\right)f(d) \\)

不同於莫比烏斯反演，這裡的 \\( d \\) 是枚舉 \\( n \\) 的倍數。

不用懷疑，它就是會無止盡的枚舉下去。

證明的方法與莫比烏斯反演的證明一雷同，這裡不再贅述。

</details>

> \\( \displaystyle\sum_{d\mid n}\varphi(d)=n \\)
>
> \\( \displaystyle\varphi(n)=\sum_{d\mid n}\mu\left(\frac n d\right)\times d \\)

<details><summary>證明</summary>

我們先證明上式。

固定 \\( n \\) 的因數 \\( d \\)，我們想計算 \\( 1\sim n \\) 中與 \\( n \\) 的 \\( \gcd \\) 為 \\( d \\) 的數字有幾個。

注意到：

\\[ \gcd(i, n) = d\Longleftrightarrow \gcd\left(\frac{i}{d}, \frac{n}{d}\right)=1 \\]

因此 \\( 1\sim n \\) 中與 \\( n \\) 的 \\( \gcd \\) 為 \\( d \\) 的數字共有 \\( \varphi\left(\frac n d\right) \\) 個。

另一方面，所有 \\( 1\sim n \\) 的數字和 \\( n \\) 的 \\( \gcd \\) 一定是 \\( n \\) 的某個因數。

所以自然地，\\( \displaystyle n=\sum_{d\mid n}\varphi\left(\frac n d\right)=\sum_{d\mid n}\varphi(d) \\)。

<p align="right">\( \blacksquare \)</p>

接著我們由上式來證明下式。

證明：

注意到上式的捲積寫法為：\\( \varphi\ast 1=id \\)。

由莫比烏斯反演即可得到下式 \\( \varphi = id\ast\mu \\)。

也因為同樣的原因，因此上下式其實是能夠互相推導的。

<p align="right">\( \blacksquare \)</p>

</details>

## 例題

1. \\( \sum_{i=1}^N\sum_{j=1}^M [\gcd(i,j)=1] \\)
2. \\( \sum_{i=1}^N\sum_{j=1}^M [\gcd(i,j)\ \mbox{is prime}] \\)
3. \\( \sum_{i=1}^N\sum_{j=1}^M \gcd(i,j) \\)
4. \\( \sum_{i=1}^N\sum_{j=1}^M ij\times \gcd(i,j) \\)

## 套路

可以發現，使用到的技巧不外乎是：

* 看到 \\(\gcd\\) 就枚舉他
* 看到艾佛森括號就把他換掉
* 原本枚舉因數的，就改成枚舉倍數
* 原本枚舉倍數的，就改成枚舉因數
* 把枚舉的倍數提出原本的數
* 看到長得像 Lemma 的就套套看
* 看到兩個 \\(\Sigma\\) 的變數乘在一起，就改成枚舉兩者乘積和其因數

## 習題

1. \\( \sum_{i=1}^N\sum_{j=1}^M [\gcd(i,j)\ \mbox{is a perfect number}] \\)
2. \\( \sum_{i=1}^N\sum_{j=1}^M lcm(i,j) \\)
3. \\( \sum_{i=1}^N\sum_{j=1}^M \sigma_0(ij) \\)
4. \\( \prod_{i=1}^N\prod_{i=1}^M F_{\gcd(i,j)} \\)

其中 \\( \sigma_0 \\) 是因數數量；\\( F \\) 是費氏數列。

每一題都有 \\( Q \\) 筆詢問，\\( N、M、Q\le 1e5 \\)

## 其他

* 給定一棵邊帶權樹，一個路徑的價值是路徑上所有權重的 \\( \gcd \\)，求所有相異路徑的價值總和
  * （點數、權重 \\( \le 10^5 \\)）
* 給定 \\( a_1,\cdots,a_n\\)（\\( n\le 10^5 \\)），求 \\( \sum_{i=1}^n \sum_{i=1}^n \gcd(a_i,a_j)\times\gcd(i,j) \\)
