# 數論函數

## 線性篩

```cpp
for (int i = 2; i <= n; ++i) {
  if (isPrime[i])
    primes.push_back(i);
  for (auto p : primes) {
    if (i * p > n)
      break;
    isPrime[i * p] = false;
    if (i % p == 0)
      break;
  }
}
```

注意到，在線性篩中，每個數字只會被其「最小的質因數」篩到一次。

## 數論函數

定義：\\(f:\mathbb{Z}\rightarrow\mathbb{C}\\)，則稱 \\(f\\) 是數論函數。

定義：若對於任意互質的 \\(n\\)、\\(m\\) 都有 \\(f(nm)=f(n)f(m)\\)，則稱 \\(f\\) 是積性的。

推論：積性函數的取值取決於在質數的冪次上的取值。

定義：Dirichlet 捲積 \\(f\ast g(n)=\sum_{d\mid n}f(d)g(\dfrac n d)\\)。

性質：Dirichlet 捲積滿足交換律、結合律、對於一般加法的分配律。

定義：艾佛森括號 \\([P]=\cases{1&: If \(P\) is true \\\\ 0&: Otherwise}\\)；特別的，\\([P=1]\\) 可以記作 \\(\iota(P)\\)。

定義：莫比烏斯函數 \\(\mu(n)=\cases{1&: \\(n = 1\\) \\\\ (-1)^k&: \\(n = p_1p_2\cdots p_k\\) \\\\ 0&: Otherwise}\\)。

---
只要是積性函數就能夠使用線性篩。

一般來說，會分成三種情況來分析，其中 \\(p\\) 是任意（在線性篩中通常是最小）的質因數：

\\(n=\cases{1 \\\\ pk & \\(p \mid k\\) \\\\ pk & \\(p\nmid k\\)} \\)


## 引理

1. \\( \sum_{d\mid n}\mu(d)=[n=1] \\)
2. \\( f(n)=\sum_{d\mid n}g(d)\Longleftrightarrow g(n)=\sum_{d\mid n}\mu(\frac n d)f(d) \\)
3. \\( f(n)=\sum_{n\mid d}g(d)\Longleftrightarrow g(n)=\sum_{n\mid d}\mu(\frac d n)f(d) \\)
4. \\( \sum_{d\mid n}\varphi(d)=n \\)，\\( \varphi(n)=\sum_{d\mid n}\mu(\frac n d)\times d \\)

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

## 杜教篩

題目：給定函數 \\( f \\)，要求 \\( \sum_{i=1}^n f(n) \\)。

定理：杜教篩 \\( \displaystyle g(1)S(n)=\sum_{i=1}^n (g\ast f)(i) - \sum_{i=2}^n g(i)S(\lfloor\frac n i\rfloor) \\)。

定義：\\( R(n)=\{\frac n i\mid i=1,2,\cdots, n\} \\)。

定理：若 \\( m\in R(n) \\)，則 \\( R(m)\subseteq R(n) \\)。

若 \\( g\ast f \\) 還有 \\( g \\) 的前綴和都很好算，則可以使用杜教篩加速，複雜度能到 \\( O(n^{2/3}) \\)。

## 例題

1. \\( \sum_{i=1}^n \mu(i) \\)
2. \\( \sum_{i=1}^n \varphi(i) \\)
3. \\( \sum_{i=1}^n \sum_{j=1}^n ij\times \gcd(i,j) \\)

其中 \\( n\le 10^{10} \\)。

## 其他

* 給定一棵邊帶權樹，一個路徑的價值是路徑上所有權重的 \\( \gcd \\)，求所有相異路徑的價值總和
    * （點數、權重 \\( \le 10^5 \\) ）
* 給定 \\( a_1,\cdots,a_n\\) （\\( n\le 10^5 \\) ），求 \\( \sum_{i=1}^n \sum_{i=1}^n \gcd(a_i,a_j)\times\gcd(i,j) \\)
