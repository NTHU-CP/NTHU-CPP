# 杜教篩

## 例題

我們先來看一道題目。

> 求 \\( S(n) = \displaystyle\sum_{i=1}^n \mu(i) \\) 的值。（\\( 1\le n\le 10^{10} \\)）

由於 \\( n \\) 太大了，即使用線性篩也會 TLE。但是先回想一個引理：

> \\( \mu\ast 1 = \sum_{d\mid n}\mu(d)=[n=1] \\)

對它取前綴和，會發現它很好算，也就是 \\(\sum_{i=1}^n \mu\ast 1 = \sum_{i=1}^n [i=1] = 1\\)。

因此若可以找到 \\(\mu\ast 1\\) 的前綴和，還有 \\(\mu\\) 的前綴和之間的關係，說不定就能夠加速計算 \\(\mu\\) 的前綴和。

而這就是大名鼎鼎的杜教篩。

## 杜教篩

杜教篩適用於以下情況：

> 給定函數 \\( f \\)，要求 \\( \displaystyle S(n)=\sum_{i=1}^n f(n) \\)。

定理：

> \\( \displaystyle g(1)\times S(n)=\sum_{i=1}^n (g\ast f)(i) - \sum_{i=2}^n g(i)\times S\left(\left\lfloor\frac n i\right\rfloor\right) \\)

證明如下：

\begin{align}
\displaystyle \sum_{i=1}^n (g\ast f)(i)
&= \sum_{i=1}^n \sum_{d\mid i} g(d)\times f\left(\frac{i}{d}\right) & \text{根據 Dirichlet 捲積的定義展開} \\\\
&= \sum_{d=1}^n \sum_{i=1}^{\left\lfloor\frac{n}{d}\right\rfloor} g(d)\times f(i) & \text{套路，將 sigma 的順序交換，並改變枚舉的變數} \\\\
&= \sum_{d=1}^n g(d) \sum_{i=1}^{\left\lfloor\frac{n}{d}\right\rfloor} f(i) & g(d) \text{ 與 } i \text{ 無關，將其移到前面} \\\\
&= \sum_{d=1}^n g(d)\times S(\left\lfloor\frac{n}{d}\right\rfloor) & \text{將 } \sum f \text{ 換成前綴和} \\\\
&= g(1)\times S(n) + \sum_{d=2}^n g(d)\times S(\left\lfloor\frac{n}{d}\right\rfloor) & \text{將 } d = 1 \text{ 的情況獨立出來}
\end{align}

移項後，即可得到：

\begin{align}
\displaystyle g(1)\times S(n)=\sum_{i=1}^n (g\ast f)(i) - \sum_{i=2}^n g(i)\times S\left(\left\lfloor\frac n i\right\rfloor\right)
\end{align}

<p align="right">\( \blacksquare \)</p>

至此可以發現，若我們有個快速計算 RHS 的方法，那只要再除掉 \\(g(1)\\)，就能夠快速得到 \\(S(n)\\) 了。

當然的，\\( g(1) \\) 必須是非零，這點後面不再贅述。

<br>

---

<br>

寫成 pseudo code 可以得到：

```py
def Find_S(n):
    ans = sum of convolution(g, f)(1...n)
    for i in range(2, n):
        ans = ans - g(i) * Find_S(n / i)
    return ans / g(1)
```

注意到，```for``` 迴圈的部分，只要我們能夠快速求出 \\( g \\) 的前綴，那麼就可以數論分塊。

令 \\( Sg \\) 是 \\( g \\) 的前綴和。

```py
def Find_S(n):
    ans = sum of convolution(g, f)(1...n)
    i = 1
    while i <= n:
        r = n / (n / i)
        ans = ans - (Sg(r) - Sg(i - 1)) * Find_S(n / i)
        i = r + 1
    return ans / g(1)
```

因此，杜教篩的流程其實就是不斷透過上述定理還有數論分塊，來進行遞迴計算。

<br>

---

<br>

```Find_S``` 是一個遞迴函式，接下來我們會討論在遞迴中，實際會走訪的狀態數是多少。

首先我們需要一個引理：

> 對任意正整數 \\( n、a、b \\)，都有 \\( \displaystyle\left\lfloor\frac{\left\lfloor\frac{n}{a}\right\rfloor}{b}\right\rfloor =\left\lfloor\frac{n}{ab}\right\rfloor \\)。

證明：

令 \\( \frac n a = x + y \\)，其中 \\( x \\) 是非負整數、\\( y \\) 是小數部分。

則 \\[ \text{LHS} = \left\lfloor\frac x b\right\rfloor \, \text{RHS} = \left\lfloor\frac{x + y}{b}\right\rfloor \\]

假設 \\( \text{LHS}\neq\text{RHS} \\)，那麼 \\( \text{LHS} < \text{RHS} \\)。

因為兩者皆是非負整數，所以在相異的假設下，至少相差一，由此可得：

\\[ \left\lfloor\frac x b\right\rfloor \le \left\lfloor\frac{x + y}{b}\right\rfloor - 1 = \left\lfloor\frac{x + y - b}{b}\right\rfloor \Rightarrow x \le x+y-b \\]

然而 \\( y \\) 是小數部分、\\( b \\) 是正整數，也就是 \\( y < 1 \Rightarrow y-b < 0\\)，得到 \\( x \\) 比自己還要小，產生矛盾。

因此由反證法可知引理是正確的。

<p align="right">\( \blacksquare \)</p>

接著我們需要定義一種集合：

> \\( \displaystyle R(n)=\left\\{\left\lfloor\frac n i\right\rfloor :\ i=1,2,\cdots, n\right\\} \\)

翻譯成白話，\\( R(n) \\) 就是 \\( n \\) 除以所有比其小的正整數並取下高斯後，那些數字們所組成的集合。

而我們在數論分塊那章節時就知道，這個集合的大小是 \\(\mathcal{O}(\sqrt{n})\\)。

有了這個定義，我們就可以來看下面這個定理：

> 若 \\( m\in R(n) \\)，則 \\( R(m)\subseteq R(n) \\)。

證明：

根據定義，\\( m\in R(n) \\) 代表存在某個正整數 \\( i \\) (\\( 1\le i\le n \\)) 使得 \\( \left\lfloor\dfrac n i\right\rfloor = m \\)。

因此

\\[ R(m)=\left\\{\left\lfloor\frac{\left\lfloor\frac{n}{i}\right\rfloor}{j}\right\rfloor :\ j=1, 2, \cdots, m \right\\} \\]

利用前述引理，可以簡化成：

\\[ R(m)=\left\\{\left\lfloor\frac{n}{ij}\right\rfloor :\ j=1, 2, \cdots, m \right\\} \\]

由此可見 \\( R(m)\subseteq R(n) \\)。

<p align="right">\( \blacksquare \)</p>

有了這份定理，我們回顧 ```Find_S```。

```Find_S``` 是遞迴函數，要計算 ```Find_S(n)``` 會需要遞迴到所有 \\( i = 2\cdots n \\) 的 ```Find_S(n / i)```。

注意到，也就是所有 \\( i\in R(n)\backslash\\{1\\} \\) 的 ```Find_S(n / i)```。

令 \\( m = \frac n i \\)。

計算 ```Find_S(m)``` 時也會需要用到所有 \\( j\in R(m)\backslash\\{1\\} \\) 的 ```Find_S(m / j)```。

根據定理可知，所需要的那些 \\( j \\)，其實也會屬於 \\( R(n)\backslash\\{1\\} \\)。

也就是說，在計算 ```Find_S(n)``` 的遞迴中，所遇到的任何 ```Find_S(x)```，\\( x \\) 都會屬於 \\( R(n) \\)。

因為 \\( |R(n)|=\mathcal{O}(\sqrt{n}) \\)，所以遞迴時所需的不同狀態數只有 \\( \mathcal{O}(\sqrt{n}) \\) 個，可以記憶化來計算。

```py
def Find_S(n):
    if Find_S(n) has been visited: 
        return ans[n]
    
    ans[n] = sum of convolution(g, f)(1...n)
    i = 1
    while i <= n:
        r = n / (n / i)
        ans[n] = ans[n] - (Sg(r) - Sg(i - 1)) * Find_S(n / i)
        i = r + 1
    ans[n] = ans[n] / g(1)
    return ans[n]
```

## 時間複雜度

上個章節討論了杜教篩的原理，並證明了會遞迴到的狀態只有 \\( \mathcal{O}(\sqrt{n}) \\)，這裡我們要來討論時間複雜度。

在此我們有兩個假設：

* \\( g\ast f \\) 的前綴和很好計算
* \\( g \\) 的前綴和很好計算

方便起見，我們把它們當成可以 \\( \mathcal{O}(1) \\) 得到。

那麼問題就只剩下計算 \\( \sum_{i=2}^n g(i)\times S\left(\left\lfloor\frac n i\right\rfloor\right) \\) 的時間複雜度了。

我們需要所有 \\( i\in R(n) \\) 的 ```Find_S(n / i)```。

而計算 ```Find_S(x)``` 會需要用到所有 \\( j\in R(x) \\) 的 ```Find_S(x / j)```，也就是需要 \\( \mathcal{O}(\sqrt{x}) \\) 個狀態。

合併下來可知時間複雜度是：

\begin{align}
\sum_{x\in R(n)} \sqrt{x} &= \sum_{x=1}^{\sqrt{n}} \left(\sqrt{x} + \sqrt{\frac{n}{x}}\right) & \text{按照 } \sqrt{n} \text{ 分成兩邊} \\\\
&=\int_1^\sqrt{n} \left(\sqrt{x} + \sqrt{\frac{n}{x}}\right) dx & \text{使用微積分來估算} \\\\
&=\left(\left.\frac{2}{3}\times x^{\frac{3}{2}} \right|_1^\sqrt{n}\right) + \left(\left.2\times\sqrt{n}\times x^{\frac{1}{2}}\right|_1^\sqrt{n}\right) & \text{透過微積分運算得到} \\\\
&=\mathcal{O}(n^{\frac{3}{4}}) &
\end{align}

因此，純粹使用杜教篩的話，時間複雜度為 \\( \mathcal{O}(n^{\frac{3}{4}}) \\)。

<br>

---

<br>

更進一步，我們可以選定某個數字 \\( k\ge \sqrt{n} \\)，將 \\( S[1]\sim S[k] \\) 使用線性篩來得到。

因此在 ```Find_S``` 中，我們只需要計算 \\( R(n) \\) 中大於 \\( k \\) 的元素即可。

因此時間複雜度會降為：

\begin{align}
\sum_{x\in R(n) \\\\ x > k} \sqrt{x} &= \sum_{x=1}^{\frac{n}{k}} \sqrt{\frac{n}{x}} \\\\
&= \int_1^\frac{n}{k} \sqrt{\frac{n}{x}} dx \\\\
&= \left.2\times \sqrt{n} \times x^{\frac{1}{2}}\right|_1^\frac{n}{k} \\\\
&= \mathcal{O}\left(\frac{n}{\sqrt{k}}\right)
\end{align}

結合線性篩，總時間複雜度是：\\( \mathcal{O}\left(k + \frac{n}{\sqrt{k}}\right) \\)。

最佳情況發生在 \\( k = \frac{n}{\sqrt{k}} \\)，也就是 \\( k = n^{\frac{2}{3}} \\) 的時候。

因此杜教篩的時間複雜度，就進一步降低為 \\( \mathcal{O}(n^\frac{2}{3}) \\) 了。

## 例題

現在我們重新回顧最開始所講的例題。

> 求 \\( S(n) = \displaystyle\sum_{i=1}^n \mu(i) \\) 的值。（\\(1\le n\le 10^{10}\\)）

由杜教篩的定理可知，我們需要找到一個函數 \\( g \\)，使得：

* \\( g\ast\mu \\) 的前綴和很好計算
* \\( g \\) 的前綴和很好計算

而我們知道 \\( (\mu\ast 1)(n) = [n=1]\\)，其前綴和很顯然，就是 \\( \sum_{i=1}^n [i=1] = 1 \\)。

\\( 1(n) \\) 的前綴和也是顯然，就是 \\( \sum_{i=1}^n 1(i) = n \\)。

因此可以將 \\( g \\) 函數選為 \\( 1 \\) 函數，因此可得：

\\[ S(n) = 1 - \sum_{i=2}^n S\left(\left\lfloor\frac{n}{i}\right\rfloor\right) \\]

利用這個遞迴式，寫成程式碼如下：

```cpp
#include <bits/stdc++.h>
using namespace std;

using ll = long long;

ll n;
ll lim;                  // n^(2/3)
unordered_map<ll, ll> s; // prefix sum of mu
vector<ll> psm;          // prefix sum of mu

void LinearSieve() {
  vector<int> primes;
  vector<int> pf(lim); // prime factor
  for (int i = 2; i < lim; ++i) {
    if (pf[i] == 0) {
      primes.push_back(i);
      pf[i] = i;
    }
    for (auto p : primes) {
      if (i * p >= lim)
        break;
      pf[i * p] = p;
      if (i % p == 0)
        break;
    }
  }

  vector<int> mu(lim);
  psm.resize(lim);
  mu[1] = psm[1] = 1;
  for (int i = 2; i < lim; ++i) {
    if (i % (pf[i] * pf[i]) != 0)
      mu[i] = -mu[i / pf[i]];
    psm[i] = psm[i - 1] + mu[i];
  }
}

ll Find_S(ll n) {
  if (n < lim)
    return psm[n];
  if (s.find(n) != s.end())
    return s[n];
  s[n] = 1;
  for (ll i = 2, r = 0; i <= n; i = r + 1) {
    r = n / (n / i);
    s[n] -= (r - i + 1) * Find_S(n / i);
  }
  return s[n];
}

int main() {
  cin >> n;
  while ((__int128)lim * lim * lim <= (__int128)n * n)
    ++lim;
  LinearSieve();
  cout << Find_S(n) << "\n";
}
```

或者也可以使用迭代來實作：

```cpp
#include <bits/stdc++.h>
using namespace std;

using ll = long long;

int lim;
unordered_map<ll, ll> s; // prefix sum of mu
vector<ll> psm;          // prefix sum of mu

void LinearSieve() {
  vector<int> primes;
  vector<int> pf(lim); // prime factor
  for (int i = 2; i < lim; ++i) {
    if (pf[i] == 0) {
      primes.push_back(i);
      pf[i] = i;
    }
    for (auto p : primes) {
      if (i * p >= lim)
        break;
      pf[i * p] = p;
      if (i % p == 0)
        break;
    }
  }

  vector<int> mu(lim);
  psm.resize(lim);
  mu[1] = psm[1] = 1;
  for (int i = 2; i < lim; ++i) {
    if (i % (pf[i] * pf[i]) != 0)
      mu[i] = -mu[i / pf[i]];
    psm[i] = psm[i - 1] + mu[i];
  }
}

int main() {
  ll n;
  cin >> n;
  while ((__int128)lim * lim * lim <= (__int128)n * n)
    ++lim;
  LinearSieve();

  vector<ll> v;
  for (ll i = 1, r = 0; i <= n; i = r + 1) {
    r = n / (n / i);
    v.push_back(i);
  }
  reverse(v.begin(), v.end());
  for (auto i : v) {
    if (n / i < lim)
      continue;
    ll m = n / i;
    s[m] = 1;
    for (ll j = 2, r = 0; j <= m; j = r + 1) {
      r = m / (m / j);
      if (m / j < lim)
        s[m] -= (r - j + 1) * psm[m / j];
      else
        s[m] -= (r - j + 1) * s[m / j];
    }
  }
  if (n == 1)
    cout << psm[1] << "\n";
  else
    cout << s[n] << "\n";
}
```

<br>

---

<br>

接著來看第二道例題：

> 求 \\( \displaystyle S(n) = \sum_{i=1}^n \varphi(i) \\) 的值。（\\(1\le n\le 10^{10}\\)）

同樣的，要使用杜教篩，就必須找到一個 \\( g \\) 使得 \\( g\ast\varphi \\) 和 \\( g \\) 本身的前綴都很容易計算。

回想數論函數章節所講的引理，選擇 \\( 1 \\) 當作 \\( g \\) 的話，可以得到：

* \\( 1\ast\varphi = id \\)，\\( \displaystyle\sum_{i=1}^n id(i)=\frac{n\times (n+1)}{2} \\)。
* \\( \displaystyle\sum_{i=1}^n 1(i) = n \\)。

兩者的確都很容易得到。

代入杜教篩的定理中，可以得到：

\\[ \displaystyle S(n) = \frac{n\times (n+1)}{2} - \sum_{i=2}^n S\left(\left\lfloor\frac{n}{i}\right\rfloor\right) \\]

寫成程式碼如下：

```cpp
#include <bits/stdc++.h>
using namespace std;

using ll = long long;

ll n;
int lim;              // n^(2/3)
vector<__int128> psp; // prefix sum of phi

void LinearSieve() {
  vector<int> primes;
  vector<int> pf(lim); // prime factor
  for (int i = 2; i < lim; ++i) {
    if (pf[i] == 0) {
      primes.push_back(i);
      pf[i] = i;
    }
    for (auto p : primes) {
      if (i * p >= lim)
        break;
      pf[i * p] = p;
      if (i % p == 0)
        break;
    }
  }

  vector<int> phi(lim);
  psp.resize(lim);
  phi[1] = psp[1] = 1;
  for (int i = 2; i < lim; ++i) {
    int now = 1, res = i;
    while (res % pf[i] == 0) {
      res /= pf[i];
      now *= pf[i];
    }
    phi[i] = phi[res] * now / pf[i] * (pf[i] - 1);
    psp[i] = psp[i - 1] + phi[i];
  }
}

unordered_map<ll, __int128> s;
__int128 Find_S(ll n) {
  if (n < lim)
    return psp[n];
  if (s.find(n) != s.end())
    return s[n];
  s[n] = (__int128)n * (n + 1) / 2;
  for (ll i = 2, r = 0; i <= n; i = r + 1) {
    r = n / (n / i);
    s[n] -= (r - i + 1) * Find_S(n / i);
  }
  return s[n];
}

int main() {
  cin >> n;
  while ((__int128)lim * lim * lim <= (__int128)n * n)
    ++lim;
  LinearSieve();
  __int128 x = Find_S(n);
  string ans = "";
  while (x)
    ans.push_back(x % 10 + '0'), x /= 10;
  reverse(ans.begin(), ans.end());
  cout << ans << "\n";
}
```

或者寫成迭代的樣子：

```cpp
#include <bits/stdc++.h>
using namespace std;

using ll = long long;

ll n;
int lim;                       // n^(2/3)
unordered_map<ll, __int128> s; // prefix sum of phi
vector<__int128> psp;          // prefix sum of phi

void LinearSieve() {
  vector<int> primes;
  vector<int> pf(lim); // prime factor
  for (int i = 2; i < lim; ++i) {
    if (pf[i] == 0) {
      primes.push_back(i);
      pf[i] = i;
    }
    for (auto p : primes) {
      if (i * p >= lim)
        break;
      pf[i * p] = p;
      if (i % p == 0)
        break;
    }
  }

  vector<int> phi(lim);
  psp.resize(lim);
  phi[1] = psp[1] = 1;
  for (int i = 2; i < lim; ++i) {
    int now = 1, res = i;
    while (res % pf[i] == 0) {
      res /= pf[i];
      now *= pf[i];
    }
    phi[i] = phi[res] * now / pf[i] * (pf[i] - 1);
    psp[i] = psp[i - 1] + phi[i];
  }
}

int main() {
  ll n;
  cin >> n;
  while ((__int128)lim * lim * lim <= (__int128)n * n)
    ++lim;
  LinearSieve();

  vector<ll> v;
  for (ll i = 1, r = 0; i <= n; i = r + 1) {
    r = n / (n / i);
    v.push_back(i);
  }
  reverse(v.begin(), v.end());

  for (auto i : v) {
    if (n / i < lim)
      continue;
    ll m = n / i;
    s[m] = (__int128)m * (m + 1) / 2;
    for (ll j = 2, r = 0; j <= m; j = r + 1) {
      r = m / (m / j);
      if (m / j < lim)
        s[m] -= (r - j + 1) * psp[m / j];
      else
        s[m] -= (r - j + 1) * s[m / j];
    }
  }

  __int128 x;
  if (n == 1)
    x = psp[1];
  else
    x = s[n];

  string ans = "";
  while (x)
    ans.push_back(x % 10 + '0'), x /= 10;
  reverse(ans.begin(), ans.end());
  cout << ans << "\n";
}
```

<br>

---

<br>

另解，注意到：

\\[ \sum_{i=1}^n \phi(i) = \sum_{i=1}^n\sum_{j=1}^i [\gcd(i, j)=1]= \frac{1}{2}\times(\sum_{i=1}^n\sum_{j=1}^n [\gcd(i, j)=1]+1) \\]

第一個等式是根據 \\( \varphi \\) 的定義得到。

第二個等式是因為 \\( (a, b) \\)、\\( (b, a) \\) 這樣的 pair 會被重複算到兩次，加一是因為 \\( i=1 \\)、\\( j=1 \\) 的情況。

而對於右式：

\begin{align}
\sum_{i=1}^n\sum_{j=1}^n [\gcd(i, j)=1] &= \sum_{i=1}^n\sum_{j=1}^n\sum_{d\mid \gcd(i, j)} \mu(d) & \text{套用在數論函數章節講過的引理} \\\\
&= \sum_{i=1}^n\sum_{j=1}^n\sum_{d\mid i \\\\ d\mid j} \mu(d) & \text{將 } \gcd \text{ 按照定義拆開} \\\\
&= \sum_{d=1}^n\sum_{i=1}^{\lfloor\frac n d\rfloor}\sum_{j=1}^{\lfloor\frac n d\rfloor} \mu(d) & \text{套路，交換 sigma 的順序} \\\\
&= \sum_{d=1}^n \mu(d) \sum_{i=1}^{\lfloor\frac n d\rfloor}\sum_{j=1}^{\lfloor\frac n d\rfloor} 1 & \mu(d) \text{ 和 } i \text{、} j \text{ 無關，移到前面}\\\\
&= \sum_{d=1}^n \mu(d) \left\lfloor\frac n d\right\rfloor^2 & \text{將後面兩個 sigma 整理好}
\end{align}

這是典型數論分塊的形式，為此我們需要求 \\( \mu \\) 的前綴和。

回想數論分塊：

```cpp
for (int i = 1, r = 0; i <= n; i = r + 1) {
  r = n / (n / i);
  ans += 1LL * (S[r] - S[i - 1]) * (n / i);
}
```

可以知道，要求的前綴和，會是每一塊的右界（\\( i - 1 \\) 是上一塊的右界），也就是 \\( \sum_{i=1}^r \mu(i) \\)。

注意到，\\( r = \left\lfloor\frac{n}{\left\lfloor n / i \right\rfloor}\right\rfloor \\)，將 \\( \lfloor n / i \rfloor \\) 看成另一個整數 \\( k \\)。

這樣一來，\\( r = \left\lfloor\frac n k \right\rfloor \\)，因此 \\( r\in R(n) \\)。

也就是說，要求的 \\( \sum_{i=1}^r \mu(i) \\)，恰好和杜教篩中會求到的相同。

因此可以結合例題一還有數論分塊，來求出 \\( \varphi \\) 的前綴和。

Bottleneck 在於求 \\( \mu \\) 的前綴和，時間複雜度依然是 \\( \mathcal{O}(n^\frac{2}{3}) \\)。

<br>

---

<br>

[洛谷 P213](https://www.luogu.com.cn/problem/P4213) 即是上面兩道練習的模板題。

在此附上 AC code。

在這道題目中要注意，\\( n + 1 \\) 可能會導致 overflow。

附註：我只有測試過以下的 code，上面的 code 並沒有丟到 OJ 上測試過。

```cpp
#include <bits/stdc++.h>
using namespace std;

using ll = long long;

int lim = (1 << 21);
int primeFactor[(1 << 21)];
vector<int> primes;
int mu[(1 << 21)];
ll phi[(1 << 21)];
unordered_map<int, int> mu_s;
unordered_map<int, ll> phi_s;

signed main() {
  ios::sync_with_stdio(0);
  cin.tie(0);

  for (int i = 2; i < lim; ++i) {
    if (!primeFactor[i]) {
      primes.push_back(i);
      primeFactor[i] = i;
    }
    for (auto p : primes) {
      if (i * p >= lim)
        break;
      primeFactor[i * p] = p;
      if (i % p == 0)
        break;
    }
  }

  mu[1] = 1;
  phi[1] = 1;
  for (int i = 2; i < lim; ++i) {
    if (i % (1LL * primeFactor[i] * primeFactor[i]) == 0) {
      mu[i] = 0;
      phi[i] = phi[i / primeFactor[i]] * primeFactor[i];
    } else {
      mu[i] = -mu[i / primeFactor[i]];
      phi[i] = phi[i / primeFactor[i]] * (primeFactor[i] - 1);
    }
  }
  for (int i = 2; i < lim; ++i) {
    mu[i] += mu[i - 1];
    phi[i] += phi[i - 1];
  }

  int T;
  cin >> T;
  while (T--) {
    int n;
    cin >> n;

    if (n < lim) {
      cout << phi[n] << " " << mu[n] << "\n";
      continue;
    }

    vector<int> R;
    for (ll i = 1, r = 0; lim <= n / i; i = r + 1) {
      r = n / (n / i);
      R.push_back(i);
    }
    reverse(R.begin(), R.end());

    for (auto i : R) {
      ll m = n / i;
      if (mu_s.find(m) != mu_s.end())
        continue;

      mu_s[m] = 1;
      phi_s[m] = m * (m + 1) / 2LL;
      for (ll j = 2, r = 0; j <= m; j = r + 1) {
        r = m / (m / j);
        if (m / j < lim) {
          mu_s[m] -= (r - j + 1) * mu[m / j];
          phi_s[m] -= (r - j + 1) * phi[m / j];
        } else {
          mu_s[m] -= (r - j + 1) * mu_s[m / j];
          phi_s[m] -= (r - j + 1) * phi_s[m / j];
        }
      }
    }

    cout << phi_s[n] << " " << mu_s[n] << "\n";
  }
}
```

## 練習題

> 求\\( \displaystyle S(n)=\sum_{i=1}^n \sum_{j=1}^n ij\times \gcd(i,j) \\) 的值。（\\( n\le 10^{10} \\)）

（hint：先以數論函數章節所教的技巧進行化簡，變成前綴和的樣子後再做杜教篩。）
