# Knuth's Optimization

動態規劃解決問題的效率取決於以下因素：\\[時間複雜度 = 狀態總數\times 每個狀態轉移的狀態數\times 每次轉移時間\\]

要優化 DP 算法可以從以上 3 種下手，本文要介紹的 Knuth's Optimization 是透過減少**每個狀態轉移的狀態數**來加速 DP。Knuth's optimization 通常用於解決具有以下形式轉移的 2D/1D 問題：

> 要求的是一個二維函數 \\(dp(i,j)\\)：> \\[ dp(i,j) = w(i,j) + \underset{i\le k\lt j}{min} \\{dp(i,k)+dp(k+1,j)\\}, \forall 1\le i\lt j\le n\\]

其中 \\(w(i,j)\\) 為一可在 \\(O(1)\\) 時間求得的代價函數，我們知道這問題有 \\(O(n^3)\\) 的做法，而我們可以利用 Knuth's Optimization 來加速到 \\(O(n^2)\\)。

- Note : \\(x\\)D/\\(y\\)D 問題是指有 \\(N^x\\) 子問題，每個子問題需 \\(N^y\\) 子問題轉移，因此有時間複雜度為\\(O(N^{x+y})\\) 的 DP 做法。

Knuth's Optimization 的應用可說是非常簡單，以 [Atcoder DP contest-Slime](https://atcoder.jp/contests/dp/tasks/dp_n) 為例，我們可以列出這個 DP 式(在此就不詳細解釋細節)：\\[dp(i,j) = w(i,j) + \underset{i\le k\lt j}{min}\\{dp(i,k)+dp(k+1,j)\\}\\]

可用下面這段 \\(O(n^3)\\) 的程式片段求出答案 \\(dp(1,n)\\)

```cpp=
for(int i=1;i<=n;i++)
{
    for(int k=1;k<=n;k++)
    {
        dp[i][k] = (i == k ? 0 : INF);
    }
}
for(int len=2;len<=n;len++)
{
    for(int i=1;i+len-1<=n;i++)
    {
        int j = i+len-1;
        for(int k=i;k<j;k++)
        {
            dp[i][j] = min(dp[i][j], dp[i][k]+dp[k+1][j]+pre[j]-pre[i-1]);
        }
    }
}
```

若我們知道這題可套用 Knuth's Optimization 要運用……

只需將 **\\(i\le k\lt j\\)** 換成 \\(opt\[i][j-1]\le k\le opt\[i+1][j]\\) 就好！

```cpp=
int i,j,k,len;
for(i=1;i<=n;i++) dp[i][i] = 0, opt[i][i] = i;
for(len=2;len<=n;len++)
{
    for(i=1;i+len-1<=n;i++)
    {
        j = i+len-1;
        dp[i][j] = INF;
        int optl = opt[i][j-1];
        int optr = opt[i+1][j];
        for(k=optl;k<=optr;k++)
        {
            if(dp[i][j] > w(i,j) + dp[i][k] + dp[k+1][j])
            {
                dp[i][j] = w(i,j) + dp[i][k] + dp[k+1][j];
                opt[i][j] = k;  
            }
        }
    }
}
```

\\(opt\[i][j-1]\\) 為 \\(dp\[i][j]\\) 的最佳轉移點，神奇的事發生了，這段程式的複雜度為 \\(O(n^2)\\)！

接下來我們來詳細解釋 Knuth's Optimization 實際上做了什麼。

## 四邊形不等式與轉移點單調

Knuth's Optimization 告訴我們，當 DP 的代價函數 \\(w\\) 滿足**四邊形不等式**與**區間單調性**時，DP 的轉移來源會有單調性，可以藉此加速 DP。

> **四邊形不等式**
>
> 有一函數 \\( f(i,j) \\)，令 \\( a\le b\le c\le d\\)
>
> 若 \\( f(a,c) + f(b,d) \le f(a,d) + f(b,c)\\)，則 \\( f \\) 函數滿足凸四邊形不等式
>
> 若 \\( f(a,c) + f(b,d) \ge f(a,d) + f(b,c)\\)，則 \\( f \\) 函數滿足凹四邊形不等式

在此我們只討論函數滿足凸四邊形不等式的情況，引入最佳轉移來源的想法，定義 \\( opt(i,j) \\) 為使 \\( dp(i,k)+dp(k+1,j) \\) 有最小值(最佳解)的轉移點，也就是 \\( dp(i,j) \\) 的最佳轉移點 \\(k\\)，

> \\[ opt(i,j) = \arg \underset{i\le k\lt j}{min}(dp(i,k) + dp(k+1,j))\\]

若有多個點取最大的即可，當代價函數 \\( w(i,j) \\) 滿足以下條件時

- 令 \\( a\le b\le c\le d \\)
- 區間單調性：\\( w(b,c) \le w(a,d) \\)
- 凸四邊形不等式：\\( w(a,c) + w(b,d) \le w(a,d) + w(b,c)\\)

我們可以證明 \\( opt(i,j-1) \le opt(i,j) \le opt(i+1,j)\\)，即轉移點有單調性。

### Prove 轉移點單調

> 若代價函數 \\( w(i,j) \\) 滿足
>
> - 令 \\( a\le b\le c\le d \\)
> - 區間單調性：\\( w(b,c) \le w(a,d) \\)
> - 凸四邊形不等式：\\( w(a,c) + w(b,d) \le w(a,d) + w(b,c)\\)
>
> 則 \\(opt(i,j-1) \le opt(i,j) \le opt(i+1,j) \\)。

Lemma : 假設以上條件成立，\\( dp \\) 函數也會滿足凸四邊形不等式。

- 定義 \\( dp_t(i,j) = dp(i,t)+dp(t+1,j)+w(i,j) \\)
- 對於 \\(i\lt j\\) 我們可以找到 \\(p ,q\\) 使 \\(i\le p\le q\lt j-1\\)，且令 \\(opt(i,j-1) = q\\)

如果我們能證明 \\(dp_p(i,j-1) \ge dp_q(i,j-1) \implies dp_p(i,j) \ge dp_q(i,j) , \forall i \le p \lt q\\)

則顯然 \\( opt(i,j-1) \le opt(i,j) \\)。

對於 \\( p+1 \le q+1 \le j-1 \le j \\)，根據四邊形不等式我們有：\begin{align}
&dp(p+1, j-1) + dp(q+1, j) ≤ dp(q+1, j-1) + dp(p+1, j) \\\\
\implies& (dp(i, p) + dp(p+1, j-1) + C(i, j-1)) + (dp(i, q) + dp(q+1, j) + C(i, j)) \\\\
\leq& (dp(i, q) + dp(q+1, j-1) + C(i, j-1)) + (dp(i, p) + dp(p+1, j) + C(i, j)) \\\\
\implies& dp_{p}(i, j-1) + dp_{q}(i, j) ≤ dp_{p}(i, j) + dp_{q}(i, j-1) \\\\
\implies& dp_{p}(i, j-1) - dp_{q}(i, j-1) ≤ dp_{p}(i, j) - dp_{q}(i, j) \\\\
\end{align}
又
\begin{align}
&dp_{p}(i, j-1) \geq dp_{q}(i, j-1) \\\\
&\implies 0 \leq dp_{p}(i, j-1) - dp_{q}(i, j-1) \leq dp_{p}(i, j) - dp_{q}(i, j) \\\\
&\implies dp_{p}(i, j) \geq dp_{q}(i, j)
\end{align}
如此一來就證明 \\( opt(i,j-1) \le opt(i,j) \\)，而另一半不等式用相似方法亦可證明。

<details><summary> Prove Lemma </summary>

有興趣的讀者可以參考這篇 [Efficient Dynamic Programming Using Quadrangle Inequalities](https://dl.acm.org/doi/pdf/10.1145/800141.804691)

</details>

## 轉移點單調應用

有了轉移點單調的性質能做甚麼呢？若我們將 \\( i-j = 0\\) 斜排的轉移點範圍列出如下：

\\[
\begin{array}{ccc}
opt_{i-2,j-3} \le opt_{i-2,j-2} \le opt_{i-1,j-2}\\\\
opt_{i-1,j-2} \le opt_{i-1,j-1} \le opt_{i,j-1}\\\\
opt_{i,j-1} \le opt_{i,j} \le opt_{i+1,j}\\\\
opt_{i+1,j} \le opt_{i+1,j+1} \le opt_{i+2,j+1}\\\\
opt_{i+2,j+1} \le opt_{i+2,j+2} \le opt_{i+3,j+2}\\\\
\end{array}
\\]

會發現前式轉移點的右界剛好是後式轉移點的左界！這樣一想，計算每個斜排時，最多只有 \\( O(N) \\) 種轉移來源，又約有 \\( 2N \\) 排，所以如果我們以索引值 \\(j-i\\) 由小到大遞增的順序計算 DP 表格，算出 DP 表格的複雜度 \\(O(N)\times O(2N) = O(N^2)\\)，比直接算的 \\(O(N^3)\\) 快多了！

但這樣做的正確性呢？若我們以索引值 \\(j-i\\) 由小到大遞增的順序計算，索引值差為 \\( j-i-1 \\) 的 \\( dp(i,j-1) \\) 與 \\( dp(i+1,j) \\) 會在算索引值差 \\( j-i \\) 的 \\( dp(i,j) \\) 前算出來，因此要算 \\( dp(i,j) \\) 時，\\( opt(i,j-1)、opt(i+1,j) \\) 都是已知，我們就不必再跑過所有轉移來源 \\([i,j]\\)，只需要去檢查 \\( [opt(i,j-1), opt(i+1,j)] \\) 就能求得 \\(dp(i,j)\\) 與 \\(opt(i,j)\\)。

下圖中黑色格子為非法值，黃色已知，綠色為正在計算，灰色未計算，計算順序如箭頭所示。

<img src="image\Knuth_optimization\dp_order.png" width="300" style="display:block; margin: 0 auto;"/>

\\[ 計算每塊綠色格子只需左邊與下方黃色格子的 opt\\]

以下為程式碼片段

```cpp=
int w(int i, int j); // cost function
void solve(int n)
{
    int i,j,k,len;
    for(i=1;i<=n;i++) dp[i][i] = 0, opt[i][i] = i;
    for(len=2;len<=n;len++)
    {
        for(i=1;i+len-1<=n;i++)
        {
            j = i+len-1;
            dp[i][j] = INF;
            int optl = opt[i][j-1];
            int optr = opt[i+1][j];
            for(k=optl;k<=optr;k++)
            {
                if(dp[i][j] > dp[i][k] + dp[k+1][j] + w(i,j))
                {
                    dp[i][j] = w(i,j) + dp[i][k] + dp[k+1][j];
                    opt[i][j] = k;
                }
            }
        }
    }
}
```

## Problems

> [Atcoder - Slimes 加強版](https://atcoder.jp/contests/dp/tasks/dp_n)
>
> 給定一個包含 \\(n\\) 個數字的序列 \\(A\\)，你會對這個序列進行 \\(n-1\\) 次操作：
>
> - 選兩個相鄰數字 \\(A_i,A_{i+1}\\) 花費 \\(A_i + A_{i+1}\\) 合併成 \\(A_i + A_{i+1}\\)，此時序列長度會減少 \\(1\\)。
>
> 如果以最佳方式行動，最小的總花費為何？
>
> - \\(n\le 3000\\)
> - \\(1 \le A_i \le 10^9\\)

<details><summary> Solution </summary>

定義 \\(cost(l,r) = \sum_{i=l}^r A_i\\)，可以列出以下 DP 式

\\[dp(i,j) = cost(i,j) + \underset{i\le k\lt j}{min} \\{dp(i,k)+dp(k+1,j)\\} \\]
原題 \\(n\le 400\\) 可在 \\(O(n^3)\\) 時間解決，但現在 \\(n\le 3000\\) 不夠快！

所以我們嘗試優化 DP，觀察到代價函數顯然滿足

1. 區間單調性：\\( w(b,c) \le w(a,d) \\)，小區間的總和必小於大區間。
2. 四邊形不等式：\\( w(a,c) + w(b,d) \le w(a,d) + w(b,c)\\)，左式必等於右式。
<img src="image\Knuth_optimization\equal_explain.jpg" width="300" style="display:block; margin: 0 auto;"/>

因此可套用 Knuth's optimization 在 \\(O(n^2)\\) 時間內解決。
</details>

<details><summary> Code </summary>

```cpp=
#include<bits/stdc++.h>
using namespace std;

using ll = long long;
const int maxn = 405;
const ll INF = 1e18;

ll pre[maxn];
ll dp[maxn][maxn];
int opt[maxn][maxn];

ll w(int i, int j)
{
    return pre[j] - pre[i-1];
}

void solve(int n)
{
    int i,j,k,len;
    for(i=1;i<=n;i++) dp[i][i] = 0, opt[i][i] = i;
    for(len=2;len<=n;len++)
    {
        for(i=1;i+len-1<=n;i++)
        {
            j = i+len-1;
            dp[i][j] = INF;
            int optl = opt[i][j-1];
            int optr = opt[i+1][j];
            for(k=optl;k<=optr;k++)
            {
                if(dp[i][j] > dp[i][k] + dp[k+1][j] + w(i,j))
                {
                    dp[i][j] = w(i,j) + dp[i][k] + dp[k+1][j];
                    opt[i][j] = k;
                }
            }
        }
    }
}

int main()
{
    int n,m,i;
    cin >> n;
    for(i=1;i<=n;i++)
    {
        cin >> m;
        pre[i] = pre[i-1] + m;
    }
    solve(n);
    cout << dp[1][n] << endl;
    return 0;
}
```

</details>

> [TIOJ - 郵局設置問題 EXTREME](https://tioj.ck.tp.edu.tw/problems/1449)
>
> 在數線上有 \\(N\\) 棟房子，現在想將其中 \\(M\\) 棟改建成郵局，使得每棟房子到最近郵局距離總和最小，問距離總和最小為何？
>
> - \\( 1 \le N,M \le 1000 \\)

<details><summary> Solution </summary>

將房子座標由小到大排序，\\( x(i) \\) 為第 \\(i\\) 棟房子的位置，\\(x(i) \le x(i+1)\\)。

首先我們考慮只有一間郵局的情況，令 \\( w(l,r) \\) 為 \\([l,r]\\) 之間只有一間郵局的最小距離和，我們知道將最中間設成郵局會有最小值，因此 \\(w(l,r)\\) 可透過以下 DP 計算：

\\[ w(i,j) = w(i,j-1) + x(j) - x( \lfloor \frac{i+j}{2}\rfloor ) \\]

可以花 \\(O(N^2) \\) 預處理好 \\( w(i,j) \\)。

接下來考慮有多個郵局的情況，令 \\( dp(i,j) \\) 為用 \\(j\\) 個郵局覆蓋 \\( [1,i] \\) 區間的最小和，可以得出以下轉移式：

\\[ dp(i,j) = \underset{i-1 \le k \lt j}{min} \\{ dp(i-1,k) + w(k+1,j) \\} \\]

也就是說 \\( [0,k] \\) 有 \\( i-1 \\) 間房子，\\( [k+1,j] \\) 有 \\( 1 \\) 間房子。這個 dp 式是 \\( O(N^3) \\) 的，我們嘗試證明它滿足 Knuth's Optimization 的條件。

### Prove

我們要證 \\( dp(i,j) \\) 滿足套用 Knuth's Optimization 的條件，只需證明代價函數 \\( w(i,j) \\) 滿足

> - 令 \\( a\le b\le c\le d \\)
> - 區間單調性：\\( w(b,c) \le w(a,d) \\)
> - 凸四邊形不等式：\\( w(a,c) + w(b,d) \le w(a,d) + w(b,c)\\)

1. 區間單調性 \\( w(b,c) \le w(a,d) \\) 很直觀，只用一間郵局處理大範圍的花費不可能少於小範圍的花費。

2. 四邊形不等式 \\( w(a,c) + w(b,d) \le w(a,d) + w(b,c)\\) 我們用反證法。

假設 \\( w \\) 不滿足四邊形不等式，則對於 \\( i \le i+1 \le j \le j+1 \\) 套用四邊形不等式，以下皆不會成立：

\begin{align}
&w(i,j) + w(i+1,j+1) ≤ w(i+1,j) + w(i,j+1) \\\\
\implies& w(i,j) + [w(i+1,j) + x(j+1) - x((i+1+j+1)/2)] \\\\
\leq& w(i+1,j) + [w(i,j) + x(j+1) - x((i+j+1)/2)] \\\\
\implies& -x((i+j)/2+1) \le -x((i+j+1)/2) \\\\
\implies& x((i+j)/2+1) \ge x((i+j+1)/2) \\\\
\end{align}

若函數不滿足四邊形不等式，\\( x(\lfloor \frac{i+j}{2}\rfloor +1) \ge x(\lfloor \frac{i+j+1}{2}\rfloor ) \\) 也不會成立，而這與 \\( x(i+1) \ge x(i) \\) 矛盾，因此 \\(w\\) 滿足四邊形不等式。

所以本題可以套用 Knuth's Optimization，會有轉移點單調 \\(opt(i,j-1) \le opt(i,j) \le opt(i+1,j) \\)

</details>

<details><summary> Code </summary>

```cpp=
#include <bits/stdc++.h>
using namespace std;

using ll = long long;
const int maxn = 1005;
const int INF = 2147483647;

int dp[maxn][maxn], opt[maxn][maxn], cost[maxn][maxn];
int a[maxn];

int w(int i, int j)
{
    return cost[i][j];
}

void solve(int n, int m)
{
    int i,j,k;
    for(i=1;i<=n;i++) dp[i][1] = cost[1][i];
    for(j=2;j<=m;j++)
    {
        opt[n+1][j] = n-1;
        for(i=n;i>=j;i--)
        {
            int optl = opt[i][j-1];
            int optr = opt[i+1][j];
            dp[i][j] = INF;
            for(k=optl;k<=optr;k++)
            {
                if(dp[i][j] > dp[k][j-1] + w(k+1,i))
                {
                    dp[i][j] = dp[k][j-1] + w(k+1,i);
                    opt[i][j] = k;
                }
            }
        }
    }
}

int main()
{
    int n,m,i,j; cin >> n >> m;
    a[1] = 0;
    for(i=2;i<=n;i++) cin >> a[i];
    sort(a+1,a+1+n);
    for(i=1;i<=n;i++)
    {
        for(j=i+1;j<=n;j++)
        {
            cost[i][j] = cost[i][j-1] + a[j] - a[(i+j)/2];
        }
    }
    solve(n,m);
    cout << dp[n][m] << endl;
    return 0;
}
```

</details>

## Exercises

> [CSES - Knuth Division](https://cses.fi/problemset/task/2088)
>
> 給定一個包含 \\(n\\) 個數字的陣列 \\(A\\)，你的任務是將其分成 \\(n\\) 個子陣列，每個子陣列只包含一個元素。在每次移動中，你可以選擇任意一個子陣列並將其分成兩個子陣列。成本是所選子陣列中的數值總和。
>
> 如果以最佳方式行動，最小的總成本是多少？
>
> - \\(n\le 5000\\)

<details><summary> Hint </summary>

> 代價函數 \\( w(l,r) = \sum_{i=l}^r A[i]\\)
>
> 顯然 \\( w(a,c)+w(b,d) = w(a,d) + w(b,c),\\) \\(\forall a \le b \le c \le d\\)

觀察可發現代價函數滿足四邊形不等式，因此可直接套用 Knuth's Optimization 解決。

</details>

> [Optimal Binary Search Tree](https://onlinejudge.org/index.php?option=com_onlinejudge&Itemid=8&page=show_problem&problem=1245)
>
> 有一包含 \\(n\\) 個相異元素的集合 \\(S = (e_1, e_2, \dots , e_n)\\)，保證 \\(e_1\lt e_2\lt \dots \lt e_n\\)，把 \\(S\\) 的元素建一顆二元搜尋樹，希望查詢頻率愈高的元素離樹根愈近。
>
> 訪問樹中元素 \\(e_i\\) 的成本 \\(cost(e_i)\\) 為該元素到樹根的路徑邊數。給定各元素的查詢次數 \\(f(e_1), f(e_2), \dots , f(e_n)\\)，定義一顆二元搜尋樹的總成本是：> \\[f(e_1)*cost(e_1) + f(e_2)*cost(e_2) + \dots + f(e_n)*cost(e_n)\\]
>
> 問最佳搜尋樹的最小成本為何？
>
> - \\(1\le n\le 250\\)
> - \\(0\le f(e_i)\le 100\\)
> - 原題 \\(O(n^3)\\) 就可通過，這邊要求時間複雜度 \\(O(n^2)\\) 內的解。

<details><summary> Hint </summary>

觀察到一顆最佳搜尋樹的左右子樹也必定會是最佳搜尋樹(如子樹不為空)，我們可以用這個最佳子結構的性質設計 DP 求解。

若定義 \\(dp(i,j)\\) 代表建一顆 BST 包含區間 \\([i,j]\\) 元素的最小成本，將其以 \\(k\\) 為根結點，左右子樹為 \\([i, k-1], [k+1,j]\\) 兩顆 BST 即可遞迴求解，轉移式：\\[dp(i,j) = \underset{i\le k\le j}{min}\\{dp(i,k-1) + dp(k+1,j) + w(i,j) - f(k)\\}\\]
\\(w(i,j)\\) 是區間和，這樣就是一個經典的區間 DP，\\(O(n^3)\\)。

而我們知道代價函數 \\( w(i,j) \\) (區間和)會滿足 Knuth's Optimization 的條件，因此可優化到 \\(O(n^2)\\)。
</details>

> [CF321E - Ciel and Gondolas](https://codeforces.com/contest/321/problem/E)
>
> 有 \\(n\\) 個人要分成編號連續的 \\(m\\) 組(不可為空)，如果 \\(i, j\\) 在同一組會產生 \\(A_{i,j}\\) 的不和諧度，問最小不和諧度可以是多少？
>
> - \\(1\le n\le 4000\\)
> - \\(1\le m\le min(n,800)\\)

<details><summary> Hint </summary>

首先想想動態規劃，應該很容易列出這個轉移式：\\[dp\[i]\[j] = \underset{1\le k\lt i}{min} \\{dp\[k][j-1] + cost(k+1, i)\\}\\]
\\(dp\[i][j]\\) 為將前 \\(i\\) 個人分成 \\(j\\) 組的最小值，\\(cost(l,r)\\) 為將編號 \\([l,r]\\) 的人放在同一組會產生的不和諧度，可用前綴和預處理好，\\(dp\[n][m]\\) 即為答案。

但這個 DP 複雜度為 \\(O(n^2m)\\)，太慢了！

所以想嘗試優化這個 DP，仔細觀察代價函數發現滿足 Knuth's Optimization 條件可套用！

> 若令 \\( a\le b\le c\le d \\)
>
> 1. 區間單調性：\\( w(b,c) \le w(a,d) \\)，大區間的不和諧度必大於等於小區間。
> 2. 凸四邊形不等式：\\( w(a,c) + w(b,d) \le w(a,d) + w(b,c)\\)，等號必成立(參考下圖)
>
> <img src="image\Knuth_optimization\equal_explain.jpg" width="300" style="display:block; margin: 0 auto;"/>
</details>

## References

- [Efficient Dynamic Programming Using Quadrangle Inequalities](https://dl.acm.org/doi/pdf/10.1145/800141.804691)
- [樹與 DP 進階](https://tioj.ck.tp.edu.tw/uploads/attachment/11/54/7.pdf)
- [建中資培講義](http://pisces.ck.tp.edu.tw/~peng/index.php?action=showfile&file=fcaa846ddcba22c1c63777723152ba9492a9f2218)
- [Knuth's Optimization in Dynamic Programming](https://www.geeksforgeeks.org/knuths-optimization-in-dynamic-programming/)
- [Knuth's Optimization - CP Algorithms](https://cp-algorithms.com/dynamic_programming/knuth-optimization.html)
- [【学习笔记】动态规划—各种 DP 优化](https://www.cnblogs.com/Xing-Ling/p/11317315.html)
- [chino's blog- 優化之四邊形不等式優化](http://www.chino.taipei/code-2016-0402Algorithm-DP優化之四邊形不等式優化/)
- [Kevin Zhang's 四邊形優化 slide](https://slides.com/kevin_zhang/deck-8c1c8e)
- IOIC 講義
- IONC 講義
