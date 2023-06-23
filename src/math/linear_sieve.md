# 線性篩

在先前的章節中，我們知道可以利用埃氏篩法在 \\( \mathcal{O}(n\log{\log{n}}) \\) 的時間內做質數篩法。

而在此章節，我們將會介紹 \\( \mathcal{O}(n) \\) 的演算法，也就是線性篩。

## 質數篩法

直接看 code。

```cpp
vector<bool> LinearPrimeSieve(int n) {
  vector<int> primes;
  vector<bool> notPrime(n + 1, false);
  notPrime[0] = notPrime[1] = true;
  for (int i = 2; i <= n; ++i) {
    if (notPrime[i] == false)
      primes.push_back(i);
    for (auto p : primes) {
      if (i * p > n)
        break;
      notPrime[i * p] = true;
      if (i % p == 0)
        break;
    }
  }
  return notPrime;
}
```

執行完以上程式碼後，``notPrime`` 為 ``true`` 的話代表是合數，為 ``false`` 代表是質數。

而 ``primes`` 裡則會是 \\( 1\sim n \\) 的質數由小到大的排列。

接下來我們會證明其正確性與時間複雜度。

證明：

> 對於所有質數 \\( i \\)，經過上述計算後，``notPrime[i]`` 等於 ``false``。

注意到 ``notPrime[i * p]`` 中的 ``i`` 和 ``p`` 都大於 \\( 1 \\)，因此只有合數的 ``notPrime`` 值會被設為 ``true``。

> 對於所有合數 \\( i \\)，經過上述計算後，``notPrime[i]`` 等於 ``true``。

對於任何合數 \\( x \\)，假設其質因數分解為 \\( p_1^{\alpha_1}p_2^{\alpha_2}\cdots p_m^{\alpha_m} \\)，其中 \\( p_1 \\) 是最小質因數。

當 \\( i = \frac{x}{p_1} \\) 時，由於 \\( p_1 < i \\)，因此在進入 ``for (auto : primes)`` 之前 \\( p_1 \\) 就會被放到 ``primes`` 裡。

所有小於 \\( p_1 \\) 的質數都無法整除 \\( i \\)，因此會順利執行到 ``notPrime[i * p] = true`` 這行而將 \\( x \\) 設成合數。

> 對於所有合數 \\( i \\)，在上述計算中，``notPrime[i] = true`` 恰好會被執行一次。

對於並非最小的質數 \\( p_2 \\)，當 \\( i = \frac{x}{p_2} \\) 的時候，因為 \\( i\bmod p_1 = 0 \\) 而且 \\( p_1 < p_2 \\)，因此會在 \\( p = p_1 \\) 時提早 ``break``。

所以所有合數僅會被其最小的質因數篩到一次。

<br>

前面兩項證明了此演算法的正確性、最後一項則證明了此演算法的時間複雜度是 \\( \mathcal{O}(n) \\)。

<p align="right">\( \blacksquare \)</p>

## 最小質因數篩法

如同埃氏篩法一樣，我們可以順手維護每個數字的最小質因數。

具體做法就是將 ``notPrime`` 改成 ``primeFactor`` 並且把布林值改成記錄最小質因數。

根據前述證明可知，記錄下來的數字恰好會是最小質因數。

```cpp
vector<int> LinearPrimeFactorSieve(int n) {
  vector<int> primes;
  vector<int> primeFactor(n + 1, 0);
  for (int i = 2; i <= n; ++i) {
    if (primeFactor[i] == 0) {
      primes.push_back(i);
      primeFactor[i] = i;
    }
    for (auto p : primes) {
      if (i * p > n)
        break;
      primeFactor[i * p] = p;
      if (i % p == 0)
        break;
    }
  }
  return primeFactor;
}
```

## 數論函數篩法

許多的數論函數都能夠透過線性篩進行打表。

一般而言，給定函數 \\( f \\) 要求 \\( f(n) \\)，假設 \\( p \\) 是 \\( n \\) 的最小質因數。

我們會分成三個情況討論：

\begin{cases}
n=1\\\\
p^2\mid n\\\\
p\mid n
\end{cases}

\\( n = 1 \\) 的情況單獨處理。

\\( n\neq 1 \\) 的時候分成兩種情況，一種是 \\( n \\) 的最小質因數 \\( p \\) 在 \\( n \\) 上的冪次為二次以上，也就是第二條式子，或者是只有一次，也就是第三條式子。

方便起見，我們將第三條式子註記成前兩條式子的 \\( Otherwise \\)，如下方例題所示。

### 莫比烏斯函數線性篩

以莫比烏斯函數 \\( \mu \\) 舉例來說：

> \\[ \mu(n)=\cases{ 1 & \\(n=1\\) \\\\ 0 & \\(p^2\mid n\\) \\\\ -\mu(\frac n p) & Otherwise} \\]

證明：

\\( \mu(1) = 1 \\) 顯然。

\\( p^2\mid n \\) 的時候，根據定義可知 \\( \mu(n) = 0 \\)。

Otherwise 的情況，假設 \\( n \\) 有其他的平方以上的質因數。

那在分解到那個數字時會回傳 \\( 0 \\)，因此這裡乘上 \\( -1 \\) 並無影響。

假設 \\( n \\) 沒有其他質因數，根據定義可以看做每個質因數各貢獻一個 \\( -1 \\)，因此符合此處的遞迴式。

<p align="right">\( \blacksquare \)</p>

寫成 code 如下：

```cpp
vector<int> LinearMuSieve(int n, vector<int> &primeFactor) {
  vector<int> mu(n + 1, 0);
  mu[1] = 1;
  for (int i = 2; i <= n; ++i) {
    if (i % (1LL * primeFactor[i] * primeFactor[i]) == 0)
      mu[i] = 0;
    else
      mu[i] = -mu[i / primeFactor[i]];
  }
  return mu;
}
```

### 歐拉函數線性篩

以歐拉函數（\\( \varphi(n)= n \text{以下與其互質的數字數量} \\)）來說：

> \\[ \varphi(n)=\cases{1 & \\( n=1 \\) \\\\ \varphi(\frac n p)\times p & \\( p^2\mid n \\) \\\\ \varphi(\frac n p)\times (p-1) & Otherwise } \\]

證明：

\\( n = 1 \\) 的情況顯然。

\\( p^2\mid n \\) 的時候，假設 \\( p \\) 在 \\( n \\) 的質因數分解中的次方為 \\( \alpha \\)，觀察一下 \\( \varphi(n) \\) 和 \\( \varphi(\frac n p) \\)：

\begin{cases}
\varphi(n) = (p^\alpha - p^{\alpha - 1})\times \text{其他質數的影響} \\\\
\varphi(\frac n p) = (p^{\alpha - 1} - p^{\alpha - 2})\times \text{其他質數的影響}
\end{cases}

比較後即可得 \\( \varphi(n) = \varphi(\frac n p)\times p \\)。

Otherwise 的情況，同樣觀察一下 \\( \varphi(n) \\) 和 \\( \varphi(\frac n p) \\)：

\begin{cases}
\varphi(n) = (p - 1)\times \text{其他質數的影響} \\\\
\varphi(\frac n p) = \text{其他質數的影響}
\end{cases}

比較可知 \\( \varphi(n) = \varphi(\frac n p) \times (p - 1) \\)。

<p align="right">\( \blacksquare \)</p>

寫成 code 如下：

```cpp
vector<int> LinearPhiSieve(int n, vector<int> &primeFactor) {
  vector<int> phi(n + 1, 0);
  phi[1] = 1;
  for (int i = 2; i <= n; ++i) {
    if (i % (1LL * primeFactor[i] * primeFactor[i]) == 0)
      phi[i] = phi[i / primeFactor[i]] * primeFactor[i];
    else
      phi[i] = phi[i / primeFactor[i]] * (primeFactor[i] - 1);
  }
  return phi;
}
```

## 結語

或許有人會認為埃氏篩法的複雜度 \\( \mathcal{O}(n\log{\log{n}}) \\) 已經夠接近線性，因此不需要弄懂線性篩。

然而在未來許多更強力的篩法（比如杜教篩）中，很多時候是需要結合線性篩一起使用的。

因此這微小的差距很可能會導致其他進階篩法的複雜度變差許多。

總結來說，埃氏篩法能夠解決大多數問題，但是對於有意精進數論的人來說，線性篩依舊是十分重要的。
