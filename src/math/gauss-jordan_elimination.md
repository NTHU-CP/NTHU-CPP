# 高斯消去法
## 介紹
高斯消去法（Gauss-Jordan elimination）是一個經常被拿來解線性方程組的經典算法，是線性代數中的基礎知識之一。除了用來解線性方程組以外，還可以用來計算矩陣的行列式值、逆矩陣等。

### 解線性方程式
範例：解二元一次方程式
$$
\begin{cases}
2x + y = -1 \\\ 3x - y = 11
\end{cases}
$$

可以發現將兩式相加可以消掉 \\(y\\)，得到 \\(5x = 10，x = 2\\)，帶回1式得到 \\(y = -5\\)

但是並不是每次遇到的線性方程組都像這個只有兩個變數而且可以輕松的看出要消掉哪些變數，因此我們需要一個更有系統的解法。

### 高斯消去法
#### 原理：
>對於任意的線性方程組，我們可進行以下操作同時不會改變解
>1. 將其中一等式左右同乘非零實數k
>2. 將一個等式加上另外一個等式
>3. 交換等式的順序
>
>上述1, 2點等價於：任意等式可以被"自己和另一個等式的線性組合"取代

接著考慮以下情況：
假設我們要解以下方程組：
$$
\begin{cases}
a_{11} + a_{12} + ...a_{1m} = b_{1} \\\ a_{21} + a_{22} + ...a_{2m} = b_{2} \\\ ...  \\\ a_{n1} + a_{n2} + ...a_{nm} = b_{n}
\end{cases}
$$
**第一步**：將係數化為增廣矩陣。

$$
  \left[\begin{array}{rrrr|r}
   a_{11} & a_{12} & ... & a_{1m} & b_{1} \\\ a_{21} & a_{22} & ... & a_{2m} & b_{2}  \\\ &.&&&. \\\ &.&&&. \\\ &.&&&. \\\ a_{n1} & a_{n2} & ... & a_{nm} & b_{n}
  \end{array}\right]
$$
**第二步**：利用高斯消去法將矩陣化為row echelon form。   
1. 從第 i = 1 行開始，利用\\(a_{11}\\)將\\(a_{21} ~ a_{n1}\\)化為0：  
    a. 將第1行 \\( \times \dfrac {a_{11}} {a_{j1}} \\) 並加到第 j = 2 行  
    b. 對第j = 2~n 行重複這個過程
2. 對第 i = 2~n 行重複這個過程，每次利用\\(a_{ii} \\) 將所有 \\( a_{ji}，j \neq i\\) 化為0。

寫成sudo code就是
```
for i = 1; i < n; i++:
    for j = 0 ; j <= n; j++ : 
        if(j != i)continue;
        row[j] += row[i] * a[j][i] / a[i][i];
```

**範例：高斯消去法解二元一次方程式**
$$
\begin{cases}
{2x + y = -1} \\\ {3x - y = 11}
\end{cases}
$$
1. 化簡成增廣矩陣
$$
    \left[\begin{array}{rr|r}
        2 & 1 & -1 \\\ 3 & -1 & 11
    \end{array}\right]
$$
2. 高斯消去法
$$
    \left[\begin{array}{rr|r}
        2 & 1 & -1 \\\ 0 & -\dfrac 5 2 & \dfrac {25} 2
    \end{array}\right]
$$
$$
    \left[\begin{array}{rr|r}
        2 & 0 & 4 \\\ 0 & -\dfrac 5 2 & \dfrac {25} 2
    \end{array}\right]
$$
3. 計算答案  
$$
\begin{cases}
{2x = 4 \rightarrow x = 2} \\\ {\dfrac 5 2 y = \dfrac{25}{2} \rightarrow y = 5}
\end{cases}
$$
## 實作
``` c++
void guass(vector<vector<double>> &v, vector<int> &ans){
    int n = v.size(), m = v[0].size();
    for(int i = 0 ; i < n; i++){
        int now = i;
        for(int j = i+1; j < n; j++){     // find pivot row
            if(abs(v[j][i]) > abs(v[now][i]))now = j; 
        }
        if(abs(v[now][i]) < esp)continue;
        swap(v[i], v[now]);               // swap pivot row to current row

        for(int j = 0; j < n; j++){       // eliminate
            if(j != i){
                double d = v[j][i] / v[i][i];
                for(int k = i; k < m; k++)
                    v[j][k] -= v[i][k] * d;
            }
        }
    }
    m--;
    ans.assign(m, 0);
    for(int i = 0; i < n; i++)
        ans[i] = v[i][m] / v[i][i];
}
```
1. 先找出pivot row

2. 利用v[i][i]來消除第i列的所有係數

3. 最後得到一個像這樣的矩陣：

$$
  \left[\begin{array}{rrrrrr|r}
   a_{11} & 0 & ... & 0 & ... & 0 & b_{1}  \\\ 0 & a_{22} & ... & 0 & ... & 0 & b_{2}  \\\ &&.&&&&. \\\ &&.&&&&. \\\ &&.&&&&. \\\ 0 & 0 & ... & a_{nn} & ... & a_{nm} & b_{n}
  \end{array}\right]
$$


### 選擇 pivot row
pivot row的選擇並沒有特別的規定。以下介紹3種選擇pivot row的方法：
1. partial pivoting：   
    選擇第i個column中絕對值最大的那個element所在的row當作pivot row。這是最常被使用的方法，上方的實作也是使用這種方法。
2. complete pivoting：   
    選擇剩下的submatrix中絕對值最大的那個element所在的row當作pivot row。
3. implict pivoting：   
    一開始先將所有的等式normalize，也就是乘以一個係數使得該等式最大的係數 = 1。接著再套partial pivoting。

最常被使用的方法是partial pivoting，因為它的誤差很小。另外一種方法則是complete pivoting，但其實有一個問題：當其中一個等式被乘上一個很大的係數，如\\(10^6\\)，那這一行基本上一定會被選擇當做pivot row。但這並不是我們想選他當pivot row的原因，因此才又有一種比較複雜的方法叫做implict pivoting來解決這個問題。implicit pivoting 會先將每一個等式normalize，也就是乘以一個係數使得該等式最大的係數 = 1。接著再套用partial pivoting 或是 complete pivoting。這一篇[codeforces blog](https://codeforces.com/blog/entry/65787)提到有關pivoting帶來的誤差和一些其他的探討，供讀者參考。

### 檢查解
***\*以下僅考慮n個方程式n個未知數的情況\****   
線性方程組的解有三種:
1. 無解
2. 剛好1組解
3. 無限組解

#### 1. 無解
考慮以下例子 : 
\begin{cases}
2x + 2y = 3 \\\ x+y = 1
\end{cases}
經過高斯消去法後我們得到
$$
\left[\begin{array}{rr|r}
    2 & 2 & 3 \\\ 0 & 0 & -2
\end{array}\right]
$$
我們可以發現如果其中一個化簡後的等式的LHS全部都是0但是RHS不等於0時，此線性方程組無解。

#### 2. 剛好1組解
考慮以下例子 : 
\begin{cases}
2x + y = 3 \\\ x+y = 1
\end{cases}
經過高斯消去法後我們得到
$$
\left[\begin{array}{rr|r}
    2 & 0 & 4 \\\ 0 & 0.5 & -0.5
\end{array}\right]
$$
如果所有的v[i][i]都不等於0，此線性方程組剛好有1組解。

#### 3. 無限組解
考慮以下例子 : 
\begin{cases}
x + y + z = 6 \\\ x + 3z = 5 \\\ x + 2y - z = 7
\end{cases}
經過高斯消去法後我們得到
$$
\left[\begin{array}{rrr|r}
    1 & 0 & 3 & 5 \\\ 0 & -1 & 2 & -1 \\\ 0 & 0 & 0 & 0 
\end{array}\right]
$$
如果有某些的v[i][i]等於0且該等式RHS也是0，此線性方程組有無限組解。

### 時間複雜度
假設矩陣的大小是\\(n \times m\\)。對於每一行，我們都會把它跟其他每一行做運算，每一次運算花費\\(O(m)\\)。假設n < m，那代表這是一個underdetermined system，所以我們至多要以每個等式為pivot做次消去，總共\\(n\\)個等式，每次花費\\(O(nm)\\)，因此總共花費\\(O(n^2m)\\)。如果\\( n > m \\)，那代表這個系統最多有m個變數，我們至多利用這\\(m\\)個變數對系統做高斯消去，每次花費\\(O(nm)\\)，因此總共花費\\(O(nm^2)\\)。因此總時間複雜度是\\(O(nm \times min(n, m))\\)。

## Modular Cases
當題目不希望我們輸出小數點的答案時，通常會要求我們輸出一個答案mod質數p。  
這時我們可以把模板中的用到除法的地方換成反元素。   
Ex : \\(\frac a b \rightarrow a \times b^{-1}\\)    
在這裡 \\(b^{-1}\\) 代表b mod p 的反元素。根據費馬小定理 \\(b^{-1} = b^{p-2}\\)，可以用快速冪求得。

### p = 2
當p = 2 時，我們發現所有係數都是0和1，因此我們可以用bitset或是bitwise operation來優化。      
有些關於xor的題目我們也可以用bitset和高斯消去法來優化我們的算法。  

#### 時間複雜度
根據硬體的不同，可能會有不同倍率的優化，但通常都是\\(O( \frac {nm\times min(n, m)} {32})\\)。

## 題目
> [洛谷 P3389 - 【模板】高斯消元法](https://www.luogu.com.cn/problem/P3389)
>
> 給你一個n個式子n個變數的線性方程組，如果有唯一解就輸出解，反之輸出No Solution。
<details><summary> Solution </summary>

模板題
```c++
#include <bits/stdc++.h>
using namespace std;
#define endl '\n'
#define ll long long
#define fast_io ios_base::sync_with_stdio(false);cin.tie(0)
double esp = 1e-9;
int guass(vector<vector<double>> &v, vector<double> &ans){
    int n = v.size(), m = v[0].size();
    for(int i = 0 ; i < n; i++){
        int now = i;
        for(int j = i+1; j < n; j++){     // find pivot row
            if(abs(v[j][i]) > abs(v[now][i]))now = j; 
        }
        if(abs(v[now][i]) < esp)continue;
        swap(v[i], v[now]);               // swap pivot row to current row

        for(int j = 0; j < n; j++){       // eliminate
            if(j != i){
                double d = v[j][i] / v[i][i];
                for(int k = i; k < m; k++)
                    v[j][k] -= v[i][k] * d;
            }
        }
    }
    m--;
    ans.assign(m, 0);
    int cc = 1;
    for(int i = 0; i < n; i++){
        if(abs(v[i][i]) > esp)ans[i] = v[i][m] / v[i][i];
        else cc = 2;
    }
    for(int i = 0; i < n; i++){
        for(int j = 0; j < m; j++){
            if(abs(v[i][j]) > esp)break;
            if(j == m-1 && abs(v[i][m]) > esp)return 0;
        }
    }
    return cc;
}
int main(){
    fast_io;
    int n;
    cin >> n;
    vector<vector<double>> v(n, vector<double>(n+1));
    vector<double> ans;
    for(int i = 0 ; i < n; i++)
        for(int j = 0 ; j < n+1; j++)
            cin >> v[i][j];
    int cc = guass(v, ans);
    if(cc == 1)for(auto &i : ans)cout << fixed << setprecision(2) << i << endl;
    else cout << "No Solution";
    return 0;
}
```
</details>

> [洛谷 P4035 - 【JSOI2008】球形空間產生器](https://www.luogu.com.cn/problem/P4035)
>
> 給一個n維空間中的球體表面上的n+1個點的座標，問球心座標。   
> n <= 10   
>
<details><summary> Solution </summary>

很顯然我們可以列出 \\(\sqrt{(x_1-x)^2 + (y_1-y)^2 + .... } = \sqrt{(x_i-x)^2 + (y_i-y)^2 + ....}\\)   
其中(x, y, .....)是n維球心座標，而2 <= i <= n。   
如此一來我們就有n個等式和n個未知數，最後就套模板。 

```c++
#include <bits/stdc++.h>
using namespace std;
#define endl '\n'
#define ll long long
#define fast_io ios_base::sync_with_stdio(false);cin.tie(0)
int main(){
    fast_io;
    int n;
    cin >> n;
    vector<vector<double>> in(n+1, vector<double>(n)), v(n, vector<double>(n+1, 0));
    vector<double> ans;
    for(int i = 0 ; i < n+1; i++)
        for(int j = 0 ; j < n; j++)
            cin >> in[i][j];
    for(int i = 0 ; i < n; i++){
        for(int j = 0 ; j < n; j++){
            v[i][j] = -2*in[0][j] + 2*in[i+1][j];
            v[i][n] += pow(in[i+1][j], 2) - pow(in[0][j], 2);
        }
    }
    guass(v, ans);
    for(auto &i : ans)cout << fixed << setprecision(3) << i << " ";
    return 0;
}

```
</details>


> [洛谷 P2973 - 【USACO10HOL】Driving Out the Piggies G](https://www.luogu.com.cn/problem/P2973)
>
> 給一張無向圖，在1號點有一顆炸彈。每回合炸彈有p/q的機率會爆炸，沒有的話則會公平的隨機移動到一個相鄰的點。問炸彈在每個點的爆炸機率。    
> n <= 300
<details><summary> Solution </summary>

對於1號點來說，炸彈一該使爆炸的機率是p/q。   
假設炸彈在i號點爆炸的機率是prob(i)，與i相鄰的點有\\(k_i\\)個。我們可以得到炸彈在1號點爆炸的機率為   
\\( prob(1) = \frac p q + \frac 1 {k_1} \sum_{j} prob(adj[1][j])\times(1-\frac p q ) \\)  

而炸彈在其它點爆炸的機率為    
\\( prob(i) = \frac 1 {k_i} \sum_{j} prob(adj[i][j])\times(1-\frac p q ) \\)

因此我們就有n個變數(prob(i))和n個等式，然後又可以套模板了。

```c++
#include <bits/stdc++.h>
using namespace std;
#define endl '\n'
#define ll long long
#define fast_io ios_base::sync_with_stdio(false);cin.tie(0)
vector<vector<double>> table;
vector<vector<int>> v;
double P;
bool visited[305];
void dfs(int now){
    visited[now] = 1;
    for(auto &i : v[now]){
        table[i-1][now-1] = (1 - P)/v[now].size();
        if(!visited[i]){
            dfs(i);
        }
    }
}
int main(){
    fast_io;    
    int n, m, p, q, a, b;
    cin >> n >> m >> p >> q;
    P = p;
    P /= q;
    v.assign(n+1, vector<int>(0));
    table.assign(n, vector<double>(n+1));
    for(int i = 0 ; i < m; i++){
        cin >> a >> b;
        v[a].emplace_back(b);
        v[b].emplace_back(a);
    }
    table[0][n] = -P;
    for(int i = 0 ; i < n; i++)table[i][i] = -1;
    dfs(1);
    vector<double> ans;
    guass(table, ans);
    for(int i = 0 ; i < n; i++)
        cout << fixed << setprecision(9) << ans[i] << " ";
    return 0;
}

```
</details>

> [LightOJ - Graph Coloring](https://lightoj.com/problem/graph-coloring)
>
> 給一張無向圖，我們想要幫每個點塗色，總共有 k 種顏色。對於每個點 v 必須滿足 \\(color(v) = color(u_1) + color(u_2) + ... color (u_m)  mod  k\\)，其中 \\( u_1, u_2, ... , u_m \\) 是跟 \\(v\\) 相鄰的點。試問有幾種塗色方法。
<details><summary> Solution </summary>

稍微調整一下等式我們會得到  \\(-color(v) + color(u_1) + color(u_2) + ... color (u_m) = 0  \  mod \  k\\)  
總共有n個點，因此我們可以得到n個等式。我們可以把這n個等式存成一個\\(n \times n\\)的矩陣。  
題目是問有幾種塗色方法，因此我們要求的是有幾組解。我們可以發現這個線性方程組每個free variable都可以帶入0, 1, 2, ...k 而得到一種塗色方法。  
因此有 r 個 free variable 就有 \\(k^{r}\\) 種塗色方法。  
一個線性方程組的 free variable 的數量 = n - matrix rank，因此我們其實只要求 \\(k^{n - rank}\\)，然後又可以套模板了。  

</details>

> [SPOJ - XMAX - XOR Maximization](https://www.spoj.com/problems/XMAX/)
> 我們在集合S上定義\\(X(S)=a_{1} \oplus a_{2} \oplus ... \oplus a_{n}， a_{i} \in S\\)。給你一個有n個數字的集合S，試問X(S)的最大值。   
> \\(n <= 10^5\\)
<details><summary> Solution </summary>
將每個數字用2進位表示，我們可以得到一個\\(n \times 64\\)的矩陣。這時我們直接對這個矩陣做高斯消去法，但是是用XOR的方式，我們就可以知道每個bit是否可以是1。如果這個bit可以是1那代表答案中的這個bit也可以是1，因此我們就可以得到一個盡可能大的答案了。
</details>

> [CF 167E - Wizards and Bets](https://codeforces.com/contest/167/problem/E)
> 
> 給一張DAG，定義in deg = 0的點叫source, out deg = 0的點叫sink，並保證source的數量等於sink的數量。有兩個巫師在玩遊戲。每回合他們會選擇一種把所有source連到不同的sink且每條路徑不相交的配對方法。對於任意一條某個source到某個sink的路徑，如果source的編號大於sink的編號，那他們形成一對*inversion*。對於每一個回合，如果inversion 的數量是偶數，則first wizard會獲得一枚金幣，反之則失去一枚。因為這兩個巫師很喜歡玩遊戲，因此每回合他們會選擇不同的配對方法，並且玩到不能再玩為止。問最後first wizard的金幣總和mod p。   
> n <= 600, m <= \\(10^5\\), 保證 p 是質數
<details><summary> Solution </summary>

首先先解決每條路徑不相交的問題。假設我們選到了一種會相交的路徑，那就表示對於每經過這個交點的(source, sink)至少有兩個。這時我們選擇任意兩個，就發現\\((source_i, sink_i)\\)，\\((source_j, sink_j)\\)和\\((source_i, sink_j)\\)，\\((source_j, sink_i)\\)的所帶來的得分會互相抵銷。   
因此結論就是我們不需要管選到的路徑有沒有相交。

接下來我們要算所有配對的*inversion*數量。假設我們建立一個表格cnt，其中cnt[i][j]代表source[i]走到sink[j]的路徑有幾種。因為題目保證source的數量 = sink的數量，因此不妨假設cnt的大小是n\\(\times\\)n。對於每一個種配對，我們可以發現其實就是在cnt中選擇n個點\\(cnt\[x_i\]\[y_i\]\\)，而且每一列每一行恰好有一個點。這時我們發現這種配對所帶來的回合數有
\\( \prod_i cnt\[x_i\]\[y_i\] \\)。   
得分的話則需要再判斷\\((x_i, y_i)\\)有多少個inversion。如果是奇數sign = -1，偶數sign = 1。
最後則是把所有配對的回合數\\(\times\\)sign加起來，也就是first wizard的得分。   
因此得分其實就是cnt的determinant。   

建立cnt可以用bsf和dfs，然後最後又可以套模板了。

</details>


## 結語

## Reference

http://e-maxx.ru/algo/linear_systems_gauss  
https://cp-algorithms.com/linear_algebra/linear-system-gauss.html  
https://oi-wiki.org/math/linear-algebra/gauss/   
https://codeforces.com/blog/entry/65787  
https://github.com/jaehyunp/stanfordacm/blob/master/code/ReducedRowEchelonForm.cc  
https://github.com/mochow13/competitive-programming-library  
