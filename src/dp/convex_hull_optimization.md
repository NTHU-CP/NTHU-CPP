# Convex Hull Optimization

## Introduction

中文稱作斜率優化，是其中一種常見的 DP 優化。

先考慮以下例題：

> [CSES - Monster Game I](https://cses.fi/problemset/task/2084/)
>
> 有 \\(n\\) 個關卡，每關有一個怪物，通關時必須從第一關開始依序通關。打敗一關的怪物需消耗 \\(sf\\) 的 cost，\\(s\\) 為怪物血量，\\(f\\) 為能力值，其中 \\(s\\) 隨 \\(i\\) 遞增、\\(f\\) 隨 \\(i\\) 遞減。你會有一個起始的能力值，每到一關，你可以選擇是否打敗該關怪物，並將能力值更新為 \\(f_i\\)。求打敗第 \\(n\\) 關怪物的最小 cost。

考慮 \\(O(n^2)\\) 的做法，令 \\(f(i)\\) 為殺掉第 \\(i\\) 關怪物所需花的最少時間。

轉移式為：\\(F(i)=\underset{i>j}{\min}(F(j)+f_j\times s_i)\\)

但是這個複雜度無法解決此題。

### 斜率單調 & 查詢單調

我們觀察一下可以發現，其實並不是所有的候選人 \\(j\\) 都有可能成為轉移來源。由於 \\(f_j\\) 是遞減的，因此對於\\(j_2>j_1\\)，若有一個 \\(i_1\\) 使得 \\(f_{j_1}s_{i_1}+F(j_1)>f_{j_2}s_{i_1}+F(j_2)\\)，那麼對於任一個 \\(i_2>i_1\\)，\\(f_{j_1}s_{i_2}+F(j_1)>f_{j_2}s_{i_2}+F(j_2)\\)也是成立的（因為\\(s_i\\)遞增），意即在 \\(i_1\\) 之後 \\(j_1\\) 就再也不會被取到了，因為至少有 \\(j_2\\) 做為更好的轉移來源。

我們再觀察一次轉移式：\\(F(i)=\underset{i>j}{\min}(F(j)+f_j\times s_i)\\)。形式其實類似於 \\(y=ax+b\\)，其中 \\(a\\) 與 \\(b\\) 只和 \\(j\\) 有關、\\(x\\) 與 \\(y\\) 只和 \\(i\\) 有關。以剛剛的轉移式來說，對應關係如下：

\\[y=F(i)\\]
\\[x=s_i\\]
\\[a=f_j\\]
\\[b=F(j)\\]

因此對於所有的 \\(j\\)，我們可以將所有的 \\(y=ax+b\\) 在座標平面上畫出來。這時我們的題目便轉變成：給定一個 \\(x\\)，我們要在一個線集中找出最小的 \\(y\\)。

接下來便要思考如何找最小值。如果要代入每一條直線再取極值才能找到答案，並不會優化複雜度，但我們可以來觀察一下這個線集：

// 對範例輸入畫個圖

可以發現這是一個斜率不斷遞減的線集（從算式中也能看出，作為斜率的 \\(f_j\\) 是遞減的）。對於每個 \\(x\\)，我們都能在這個線集中找到對應的最小值，如果我們將它們通通標出來，可以發現會形成一個上凸包：

// 畫圖標記上凸包

對任何 \\(x'\\) 而言，\\(x=x'\\) 與這個上凸包的交點便是所求的最小值。要找到這個交點，我們必須維護這個凸包，並支援兩種操作：

- 查詢：對於 \\(x=s_i\\)，我們要能找出其與凸包的交點。

    由於查詢具單調性，以此題而言，查詢遞增且我們要找最小值，我們取的直線斜率一定會越來越小。因此我們可以只考慮目前最左邊的直線，並確認目前直線與 \\(x=s_i\\) 的交點是否真的是最小值。確認的方法便是比較當前交點以及下一條直線與 \\(x=s_i\\) 的交點，若下一條直線能讓我們得到更好的答案，我們就要把當前最左邊的直線丟掉。反之，最左邊的直線與 \\(x=s_i\\) 的交點就是我們要找的最小值。

    // 補圖說明

- 插入：插入一條新的線
    插入似乎相對直觀，只是畫一條新的線在凸包上。由於斜率單調，插入直線時我們只要將直線插入最右（左）邊就好。但我們考慮下列情況：

    // 畫圖，有直線不在凸包上

    在我們加入新的直線後，有一些直線便不在凸包上了，意即它們不可能成為任何查詢的答案。這時我們要將它們移出凸包，否則會取到錯誤的答案。

綜合上述，我們可以使用單調隊列，開一個 deque 完成這兩項操作。具體而言，每算完一個 \\(F(i)\\)，便將 pair \\((f_i, F(i))\\) 插入隊列尾端，分別代表直線的 \\(a\\) 與 \\(b\\)。插入之前，我們要檢查是否有直線在這次插入後，再也不會被當成答案。檢查的方式是看 deque 中的倒數第二條線與當前要插入的直線的交點，以及 deque 中的倒數第二條線與 deque 中的最後一條線的交點。若前者有更小的 \\(x\\) 座標，則 deque 中的最後一條線永遠不會被取到，要 pop 掉。

// 畫圖

在查詢最小值時，我們不斷比較隊列中第一個與第二個元素，如果將當前的 \\(x\\) 代入兩條直線後，發現代入第二條直線有更好的解，則將第一條直線 pop 掉。

這麼一來複雜度便降到 \\(O(n)\\)。

<details><summary> Solution Code </summary>

```cpp
#include <bits/stdc++.h>
#pragma GCC optimize("O2")
using namespace std;

typedef pair<int, int> pii;
typedef long long ll;
typedef pair<ll, ll> pll;
#define X first
#define Y second
#define io ios_base::sync_with_stdio(0); cin.tie(0);
#define setpr setprecision
#define ef emplace_front
#define eb emplace_back
#define pub push_back
#define em emplace
#define pb pop_back
#define pf pop_front
#define lowb lower_bound
#define uppb upper_bound
#define lowbit(x) ((x) & -(x))
#define mk make_pair
const ll M = 1e9 + 7;
const ll INF = 1e14;
# define N 200005

ll s[N], f[N];
ll dp[N];

ll cal(ll x, pll line){
    return x * line.X + line.Y;
}

bool cmp(pll line1, pll line2, pll line3){
    return (line3.Y - line1.Y) * (line1.X - line2.X) <= (line2.Y - line1.Y) * (line1.X - line3.X); 
}

int main(){
    int n;
    cin >> n >> f[0];
    for(int i = 1; i <= n; i++){
        cin >> s[i];
    }
    for(int i = 1; i <= n; i++){
        cin >> f[i];
    }
    deque <pll> dq;
    dq.eb(f[0], 0);
    for(int i = 1; i <= n; i++){
        while(dq.size() >= 2 && cal(s[i], dq[0]) > cal(s[i], dq[1])) dq.pf();
        dp[i] = cal(s[i], dq[0]);
        pll line(f[i], dp[i]);
        while(dq.size() >= 2 && cmp(dq[dq.size() - 2], dq[dq.size() - 1], line)) dq.pb();
        dq.eb(line);
    }
    cout << dp[n] << "\n";
    return 0;
}
```

</details>

#### 小結

// 需再統整或添加

// 步驟總結

1. 將初始值放入 deque
2. 不斷比較 deque 最前端的兩條線。若第二條線有較好的答案就 pop 掉最前端的線。
3. 不斷比較 deque 最尾端的線與新的直線。若前者在新的線加入後便不在凸包中就 pop 掉，最後插入新的直線。

// 要注意的地方

- 要注意斜率以及查詢的單調性是遞增或遞減，會影響實作（應在頭或尾插入等）
- 注意題目是否會有轉移範圍的限制
- 轉移式改成取 max 也可以做，只是變成建下凸包

#### 另一種思路

// 補上讓 j 為點、i 為線的想法與實作方法

### 斜率單調 & 查詢不單調

上面的題目由於查詢單調，我們才能保證可以只看 deque 前端的線段就找到最小值。那麼，要是查詢不單調該怎麼辦？

查詢不再單調，意味著轉移來源不單調，因此我們查詢的時候不能再取最左（右）邊的線來做比較。但是斜率依然是單調的，因此我們可以改為使用二分搜來尋找答案。我們一樣維護一個斜率有單調性的序列，不過這次我們不再取最前端的直線來計算答案，而是改為使用二分搜。這時我們不再需要 pop 最前段的線，但是在 insert 新的直線到序列最尾端時，我們一樣要將不可能是答案的直線 pop 掉。

至於二分搜要搜什麼呢？注意到我們維護的是一個凸包，凸包上的點的 \\(x\\) 座標是單調的。而每當我們要查詢的時候，我們想找的那條線上的兩個頂點會在當前查詢的 \\(x\\) 座標的左右兩側：

// 一個凸包的圖，標記兩側頂點

因此我們在二分搜時，可以比較當前線段與下一條線段交點的 \\(x\\) 座標，以及當前要查詢的 \\(x\\) 座標，若前者較大則右界改為當前 \\(mid\\)、反之更改左界。

將查詢改為二分搜會讓複雜度多一個 \\(\log\\)，總複雜度是 \\(O(n\log n)\\)。

以下是用此方法解 [CSES - Monster Game I](https://cses.fi/problemset/task/2084/) 的範例 code。

<details><summary> Solution Code </summary>

```cpp
#include <bits/stdc++.h>
#pragma GCC optimize("O2")
using namespace std;

typedef pair<int, int> pii;
typedef long long ll;
typedef pair<ll, ll> pll;
#define X first
#define Y second
#define io ios_base::sync_with_stdio(0); cin.tie(0);
#define setpr setprecision
#define ef emplace_front
#define eb emplace_back
#define pub push_back
#define em emplace
#define pb pop_back
#define pf pop_front
#define lowb lower_bound
#define uppb upper_bound
#define lowbit(x) ((x) & -(x))
#define mk make_pair
const ll M = 1e9 + 7;
const ll INF = 1e14;
# define N 200005

ll s[N], f[N];
ll dp[N];

ll cal(ll x, pll line){
    return x * line.X + line.Y;
}

ll cmp2(pll p1, pll p2, ll x){
    return (p2.Y - p1.Y) > (p1.X - p2.X) * x;
}

pll find_line(vector <pll> v, ll x){
    int l = 0, r = v.size() - 2, mid, ans = max(r + 1, 0);
    while(l <= r){
        mid = (l + r) >> 1;
        if(cmp2(v[mid], v[mid + 1], x)){
            r = mid - 1;
            ans = mid;
        }
        else l = mid + 1;
    }
    return v[ans];
}

bool cmp(pll line1, pll line2, pll line3){
    return (line3.Y - line1.Y) * (line1.X - line2.X) <= (line2.Y - line1.Y) * (line1.X - line3.X); 
}

int main(){
    int n;
    cin >> n >> f[0];
    for(int i = 1; i <= n; i++){
        cin >> s[i];
    }
    for(int i = 1; i <= n; i++){
        cin >> f[i];
    }
    vector <pll> v;
    v.eb(f[0], 0);
    for(int i = 1; i <= n; i++){
        pll fl = find_line(v, s[i]);
        dp[i] = cal(s[i], fl);
        pll line(f[i], dp[i]);
        while(v.size() >= 2 && cmp(v[v.size() - 2], v[v.size() - 1], line)) v.pb();
        v.eb(line);
    }
    cout << dp[n] << "\n";
    return 0;
}
```

</details>

### 斜率不單調

斜率不單調的情況更為複雜，我們不能再使用 deque 或是 set 來維護凸包，以下會介紹幾個解決此問題的方法。

#### CDQ 分治

直接來看一個題目。
> [CSES - Monster Game II](https://cses.fi/problemset/task/2085)
>
> 有 \\(n\\) 個關卡，每關有一個怪物，通關時必須從第一關開始依序通關。打敗一關的怪物需消耗 \\(sf\\) 的 cost，\\(s\\) 為怪物血量，\\(f\\) 為能力值。你會有一個起始的能力值，每到一關，你可以選擇是否打敗該關怪物，並將能力值更新為 \\(f_i\\)。求打敗第 \\(n\\) 關怪物的最小 cost。

題目大致與 [CSES - Monster Game I](https://cses.fi/problemset/task/2084/) 相同，但此題不再保證 \\(f\\) 與 \\(s\\) 的單調性，意即斜率與查詢都不再單調。不過透過 CDQ 分治，我們可以「人為」創造出斜率的單調性。

我們一樣要在遞迴過程中，將序列分左半邊與右半邊處理。前面提到我們想創造出斜率的單調性，因此在遞迴過程中我們要維護一個斜率單調的序列，且裡面沒有不會被取到的直線。先不論直線如何計算，討論維護的部分。維護的方式類似 merge sort 的過程，只是要注意新增直線到序列尾端時，一樣要做 pop 的動作，將不可能會被取到的直線刪除。

由此，我們在做完遞迴完一個區間時，我們可以得到該區間的所有直線，且依照斜率排序。那麼接下來的問題就是該如何計算 dp 值。如果今天我們左半邊的區間已經處理好了（dp 值皆計算完成且直線依斜率排好），那我們便可以用左半區間去更新右半區間的答案。更新完之後，左半邊的資訊便沒有用了，我們就可以往下遞迴右半區間，重複一樣的動作。

更新的時候就跟斜率單調、查詢不單調時的情況一樣。我們已經有左半邊排序好的序列，現在對右半邊的每個查詢（不單調），我們都要在左半邊的序列中找到最佳解並更新。由於這時斜率是單調的，一樣可以使用二分搜。

總結一下重點：

- 先遞迴左半邊，再來用左半邊的資訊更新右半邊的 dp 值，最後遞迴右半邊。
- 遞迴的時候要一邊對斜率做排序。
- 更新右半區間時，二分搜左半區間詢問最佳解，像斜率單調那時一樣。
- 時間複雜度 \\(O(n\log^2 n)\\)。

<details><summary> Solution Code </summary>

```cpp
#include <bits/stdc++.h>
#pragma GCC optimize("O2")
using namespace std;

typedef pair<int, int> pii;
typedef long long ll;
typedef pair<ll, ll> pll;
#define X first
#define Y second
#define io ios_base::sync_with_stdio(0); cin.tie(0);
#define setpr setprecision
#define ef emplace_front
#define eb emplace_back
#define pub push_back
#define em emplace
#define pb pop_back
#define pf pop_front
#define lowb lower_bound
#define uppb upper_bound
#define lowbit(x) ((x) & -(x))
#define mk make_pair
const ll M = 1e9 + 7;
const ll INF = 1e14;
#define N 200005

ll dp[N], s[N], f[N];

ll cal(pll line, ll x){
    return x * line.X + line.Y;
}

ll cmp2(pll p1, pll p2, ll x){
    return (p2.Y - p1.Y) > (p1.X - p2.X) * x;
}

bool cmp(pll line1, pll line2, pll line3){
    return (line3.Y - line1.Y) * (line1.X - line2.X) <= (line2.Y - line1.Y) * (line1.X - line3.X); 
}

pll find_line(vector <pll> v, ll x){
    int l = 0, r = v.size() - 2, mid, ans = max(0, r + 1);
    while(l <= r){
        mid = (l + r) >> 1;
        if(cmp2(v[mid], v[mid + 1], x)){
            r = mid - 1;
            ans = mid;
        }
        else l = mid + 1;
    }
    return v[ans];
}

vector <pll> solve(int l, int r){
    vector <pll> v;
    if(l == r){
        v.eb(f[l], dp[l]);
        return v;
    }
    int mid = (l + r) >> 1;
    vector <pll> v1 = solve(l, mid);
    // calculate dp value for (mid, r]
    for(int i = mid + 1; i <= r; i++){
        pll fl = find_line(v1, s[i]);
        dp[i] = min(dp[i], cal(fl, s[i]));
    }
    vector <pll> v2 = solve(mid + 1, r);
    int idx1 = 0, idx2 = 0;
    while(idx1 < v1.size() || idx2 < v2.size()){
        pll pl;
        if(idx2 >= v2.size() || (idx1 < v1.size() && v1[idx1] >= v2[idx2])) pl = v1[idx1++];
        else pl = v2[idx2++];
        while(v.size() >= 2 && cmp(v[v.size() - 2], v.back(), pl)) v.pb();
        v.eb(pl);
    }
    return v;
}

int main(){
    int n;
    cin >> n >> f[0];
    for(int i = 1; i <= n; i++){
        cin >> s[i];
    }
    for(int i = 1; i <= n; i++){
        cin >> f[i];
        dp[i] = INF;
    }
    solve(0, n);
    cout << dp[n] << "\n";
    return 0;
}
```

</details>

#### 李超線段樹

看題目之前，先來介紹這個資料結構。

假設現在我們想在平面座標上做兩個操作（強制在線）：

1. 加入一條形式為 \\(y=ax+b\\) 的直線
2. 詢問 \\(x=k\\) 與所有直線交點的 \\(y\\) 的最大值

看起來像是區間修改、單點查詢的題目。我們考慮線段樹的作法，令左右界為值域，每個節點存的東西是一條直線。接著我們來思考如何插入與查詢。

**插入**

如果當前的節點沒有存直線，那我們就直接將要插入的直線存進去。

如果當前的節點已經存直線了，那我們要想辦法將他們合併起來。假設當前節點的區間是 \\(\[L, R)\\)、中點為 \\(mid=\frac{L+R}{2}\\)。要加進去的直線為 \\(f\\)、在節點中的直線為 \\(g\\)。不失一般性，我們假設 \\(g\\) 的斜率大於 \\(f\\) （如果不是的話就交換，其餘做法相同）。這兩條直線可能會有以下兩種情況：

1. 在 \\(\[L, R)\\) 中，\\(f\\) 大於 \\(g\\) 的部分較多。
我們可以將當前區間分為兩個區間：\\(f\\) 比較大的區間與 \\(g\\) 較大的區間，以下稱這兩個區間為 \\(F\\) 和 \\(G\\)，如下圖。我們可以發現 \\(G\\) 一定能被 \\(\[L, mid\]\\) 或 \\(\[mid + 1, R)\\) 其中之一完全包含。我們將 \\(G\\) 向下推，像在線段樹上更新懶惰標記一樣，更新對應的子節點。\\(F\\) 則保留下來存在當前節點。

    // 補圖

    判斷方式：\\(f\\) 在 \\(mid\\) 的值大於 \\(g\\) 則成立。

2. 在 \\(\[L, R)\\) 中，\\(f\\) 小於 \\(g\\) 的部分較多。
與上一個情況相似，這次我們將 \\(G\\) 存於當前節點，將 \\(F\\) 向下推。

    判斷方式：\\(f\\) 在 \\(mid\\) 的值小於 \\(g\\) 則成立。

若 \\(f\\) 在 \\(mid\\) 的值等於 \\(g\\)，可以歸入上述任一種情況做操作。

由於我們向下推的時候只會推左或右其中一個子區間，這麼做的複雜度為 \\(O(\log C)\\)，其中 \\(C\\) 是值域大小。

```cpp
void insert(int l, int r, int id, pll line){
    if(l == r){
        if(cal(line, l) < cal(seg[id], l)) seg[id] = line;
        return;
    }
    int mid = (l + r) >> 1;
    if(line.X > seg[id].X) swap(line, seg[id]);
    if(cal(line, mid) <= cal(seg[id], mid)){
        insert(l, mid, id * 2, seg[id]);
        seg[id] = line;
    }
    else{
        insert(mid + 1, r, id * 2 + 1, line);
    }
}
```

**查詢**

類似一般線段樹上的區間修改、單點查詢一樣，我們只看包含 \\(k\\) 的那些區間，並從這些線段中取位於 \\(k\\) 的最大值。

```cpp
ll query(int l, int r, int id, ll x){
    if(x < l || x > r) return INF;
    if(l == r) return cal(seg[id], x);
    int mid = (l + r) >> 1;
    ll val = x <= mid ? query(l, mid, id * 2, x) : query(mid + 1, r, id * 2 + 1, x);
    return min(val, cal(seg[id], x));
}
```

---

現在我們來看題目，一樣是 [CSES - Monster Game II](https://cses.fi/problemset/task/2085)，但這次我們考慮李超線段樹的作法。想法很直觀，我們只要先在李超樹上查詢最小值以得到當前 dp 值，再將新的線插入樹中就好。時間複雜度 \\(O(n\log n)\\)。

<details><summary> Solution Code </summary>

```cpp
#include <bits/stdc++.h>
#pragma GCC optimize("O2")
using namespace std;

typedef pair<int, int> pii;
typedef long long ll;
typedef pair<ll, ll> pll;
#define X first
#define Y second
#define io ios_base::sync_with_stdio(0); cin.tie(0);
#define setpr setprecision
#define ef emplace_front
#define eb emplace_back
#define pub push_back
#define em emplace
#define pb pop_back
#define pf pop_front
#define lowb lower_bound
#define uppb upper_bound
#define lowbit(x) ((x) & -(x))
#define mk make_pair
const ll M = 1e9 + 7;
const ll INF = 1e18;
#define N 1000005

ll s[N], f[N];
ll dp[N];
pll seg[N * 4];

ll cal(pll line, ll x){
    return line.X * x + line.Y;
}

void insert(int l, int r, int id, pll line){
    if(l == r){
        if(cal(line, l) < cal(seg[id], l)) seg[id] = line;
        return;
    }
    int mid = (l + r) >> 1;
    if(line.X > seg[id].X) swap(line, seg[id]);
    if(cal(line, mid) <= cal(seg[id], mid)){
        insert(l, mid, id * 2, seg[id]);
        seg[id] = line;
    }
    else{
        insert(mid + 1, r, id * 2 + 1, line);
    }
}

ll query(int l, int r, int id, ll x){
    if(x < l || x > r) return INF;
    if(l == r) return cal(seg[id], x);
    int mid = (l + r) >> 1;
    ll val = x <= mid ? query(l, mid, id * 2, x) : query(mid + 1, r, id * 2 + 1, x);
    return min(val, cal(seg[id], x));
}

int main(){
    int n, x;
    cin >> n >> x;
    for(int i = 1; i <= n; i++){
        cin >> s[i];
    }
    for(int i = 1; i <= n; i++){
        cin >> f[i];
    }
    for(int i = 1; i <= 4e6; i++){
        seg[i] = pll(0, INF);
    }
    insert(1, 1e6, 1, pll(x, 0));
    for(int i = 1; i <= n; i++){
        dp[i] = query(1, 1e6, 1, s[i]); 
        insert(1, 1e6, 1, pll(f[i], dp[i]));
    }
    cout << dp[n] << "\n";
    return 0;
}
```

</details>

#### 動態凸包

// 希望這一part可以日後再補Q

### 總結

// 幾個重點，需要再整理重寫、添加

- 斜率優化只是能優化特定形式轉移式的轉移複雜度
- 斜率是否單調會影響凸包維護的方式；查詢是否單調會影響尋找轉移來源的方式。

## Exercises

// 題敘與 code 待補

> [CF 1083E - The Fair Nut and Rectangles](https://codeforces.com/contest/1083/problem/E)

> [CF 311B - Cats Transport](https://codeforces.com/contest/311/problem/B)

> [CF 319C - Kalila and Dimna in the Logging Industry](
https://codeforces.com/problemset/problem/319/C)

> [CF 1715E - Long Way Home](https://codeforces.com/problemset/problem/1715/E)

> [CSES - Subarray Squares](https://cses.fi/problemset/task/2086)

> [ICPC WF 2011](https://onlinejudge.org/index.php?option=onlinejudge&Itemid=8&page=show_problem&problem=3547)

> [TIOJ 1921 - 吐鈔機2](https://tioj.ck.tp.edu.tw/problems/1921)

## References

- [Oi wiki - 斜率優化](https://oi-wiki.org/dp/opt/slope/)
- [TIOJ 建中培訓講義](https://tioj.ck.tp.edu.tw/uploads/attachment/5/27/5.pdf)
- [Algorithms for competitive programming - Convex hull trick](https://cp-algorithms.com/geometry/convex_hull_trick.html)
- [[Tutorial] Convex Hull Trick — Geometry being useful](<https://codeforces.com/blog/entry/63823>)
- [PEGWiki - Convex hull trick](https://wcipeg.com/wiki/Convex_hull_trick)
- [USACO Guide - Convex Hull Trick](https://usaco.guide/plat/convex-hull-trick?lang=cpp)
- [【學習筆記】動態規劃—斜率優化DP（超詳細）](https://www.cnblogs.com/Xing-Ling/p/11210179.html)
- [Oi wiki - 李超線段樹](https://oi-wiki.org/ds/li-chao-tree/)
- [A Simple Introduction to Li-Chao Segment Tree](https://robert1003.github.io/2020/02/06/li-chao-segment-tree.html)
