# Heavy Light Decomposition

## Intro

### Intro to Leavy Light Decomposition

輕重鏈剖分，又稱為樹鏈剖分，是一種常見的樹狀結構分解方法，主要用於在樹上執行一些效能較低的操作，概念的話並不困難，比較像是一種優美的樹上問題暴力解法。

### Definition of Terms

在這個環節，我們會用許多東西來紀錄資料。
1. `size`：紀錄每一個節點中，子樹有多少個節點(包含自己)。
2. `Heavy Child`：對於每個節點的子樹中，`size` 最大的子樹，我們稱之為 `Heavy Child`。而連向 `Heavy Child` 的這條邊，我們稱之為 `Heavy Edge`。
3. `Light Child`：對於每個節點的子樹中，非 `Heavy Child` 的部分，我們都稱之為 `Light Child`，且這條連結過去的邊稱之為 `Light Edge`。 
4. `Time Stamp`：時間戳記，記錄節點在深度優先搜尋中的拜訪順序。這邊我們以 `dfn[x]` 來表示第 \\(x\\) 個節點被拜訪的順序。
5. 鏈的起點，我們以 `top[x]` 來表示第 \\(x\\) 個節點的起點是哪一個節點。 

## Implementation

### First DFS

對於一顆無根樹，選擇隨機一個節點，對他做深度優先搜尋(DFS)，在遞迴的過程中，我們可以紀錄每個節點的父節點(father node)、深度(deep)、大小(size)、重小孩(heavy child)。

紀錄這些的目的是之後做查詢比較方便，有些資料也必須用於第二次的深度優先搜尋去建構樹鏈剖分。

### Second DFS

經過第一次深度優先搜尋後，我們已經有每個節點重小孩的資訊了。

第二次的深度優先搜尋，我們要做的是優先遞迴重小孩去建構重鏈(heavy chain)，之後再遞迴其他小孩，也就是輕小孩，去構造輕鏈(light chain)。

在這個部分，我們要紀錄時間戳記、鏈的起點。

紀錄時間戳記通常是因為我們在遞迴時，一條鏈所在資訊具有連續性，所以可以記錄鏈上最大值、最小值、總和等資訊。

紀錄鏈的起點是之後做查詢的時候，快速找到需要的資訊和做跳躍的動作。

### Build Data Structures on Each Heavy Chain

由於我們已知深度優先搜尋遞迴拜訪重鏈編號的連續性，所以可以得到 `[dfn[top[x]], dfn[x]]` 這個區間是我們要的鏈上資訊。

所以我們可以在第二次深度優先搜尋的時候記錄 `dfn[x]=++timestamp`，並對 `dfn[x]` 做資料結構的更新，即可完成紀錄。

### Queries

欲查詢節點 \\(u,v\\) 路徑上的資訊，我們可以將其拆成兩條鏈，分別為 \\(u\sim LCA(u,v)\\) 和 \\(v\sim LCA(u,v)\\)，其中 \\(LCA(u,v)\\) 定義為 \\(u,v\\) 的最低共同祖先。

所以我們可以每次讓 \\(u,v\\) 比較深的那個節點往上搜尋整條鏈上的資訊，接著跳到上一條鏈上，直到做到 \\(LCA(u,v)\\) 即結束。

### Definition of Variables and Code

我們先給出樹鏈剖分會用到的名詞定義：

- `sz[x]`：表示節點 \\(x\\) 的子樹節點數總和(包含自己)。
- `fa[x]`：表示節點 \\(x\\) 的父節點編號。
- `dep[x]`：表示節點 \\(x\\) 的深度。
- `to[x]`：表示重小孩，也就是第二次遞迴優先遞迴的目標。
- `top[x]`：表示節點 \\(x\\) 所在的這條鏈上起點。
- `dfn[x]`：為節點 \\(x\\) 的被拜訪的順序，以 `dfn[x] = ++t` 表示。其中 \\(t\\) 表示時間戳記。

其他名詞定義：
- `vector<int> g[N]`：鄰接串列，用於紀錄樹的資料。
- `int seg[N << 1]`：zkw 線段樹，用來記錄/更新/查詢鏈上資料。
- `int arr[N]`：紀錄節點資訊。

以線段樹實作維護鏈上最大值的樹鏈剖分的範例程式碼。
```cpp
const int N = 2e5 + 5;
int t, n, q, seg[N << 1]; // t := time-stamp
int sz[N], fa[N], dep[N], to[N], top[N], dfn[N], arr[N];
// size, father, depth, to-heavy-child, top, dfs-order, a_i value
vector<int> g[N];
void upd(int x, int v) {
  for (seg[x += n] = v; x > 1; x >>= 1)
    seg[x >> 1] = max(seg[x], seg[x ^ 1]);
}
int qry(int l, int r) { // [l, r]
  int ret = -1e9; // -max
  for (l += n, r += n + 1; l < r; l >>= 1, r >>= 1) {
    if (l & 1) ret = max(ret, seg[l++]);
    if (r & 1) ret = max(ret, seg[--r]);
  }
  return ret;
}
void dfs(int x, int p) {
  sz[x] = 1, fa[x] = p, to[x] = -1, dep[x] = ~p ? dep[p] + 1 : 0;
  for (auto i : g[x])
    if (i != p) {
      dfs(i, x);
      if (to[x] == -1 || sz[i] > sz[to[x]]) to[x] = i;
      sz[x] += sz[i];
    }
}
void dfs2(int x, int f) {
  top[x] = f, dfn[x] = ++t, upd(dfn[x], arr[x]);
  if (to[x] != -1) dfs2(to[x], f);
  for (auto i : g[x])
    if (i != fa[x] && i != to[x]) dfs2(i, i);
}
int qry2(int u, int v) { // query on tree
  int fu = top[u], fv = top[v], ret = -1e9;
  while (fu != fv) {
    if (dep[fu] < dep[fv]) swap(fu, fv), swap(u, v);
    ret = max(ret, qry(dfn[fu], dfn[u])); // interval: [dfn[fu], dfn[u]]
    u = fa[fu], fu = top[u];
  }
  if (dep[u] > dep[v]) swap(u, v);
  // u is the LCA
  ret = max(ret, qry(dfn[u], dfn[v]));
  return ret;
}
```

## Proof of Correctness

定義 \\(s(v)\\) 為節點 \\(v\\) 的子數大小。我們可以知到每一個節點 \\(u\\) 只會有一個重小孩 \\(v\\)，且 \\(s(v)\geq\frac{s(u)}{2}\\)，可以由反證法得證。

假設有兩個重小孩，那麼 \\(s(u)\geq1+2\times\frac{s(v)}{2}>s(u)\\)，產生矛盾。

因為每個節點會往最大子樹的小孩連接重邊，其餘為輕邊。

因此在此演算法的時間複雜度相當於考慮「跳了多少條輕邊」，又因為每次跳過輕邊時，子樹大小會減少至少一半（根據上面的證明），在每次都是至少減半的過程中，可以得知一個節點 \\(u\\) 要跳至根節點，至多只會跳 \\(O(\log N)\\) 條輕邊，也就是說有 \\(O(\log N)\\) 條樹鍊，那顯然任兩個節點的路徑上也只會有 \\(O(\log N)\\) 條樹鍊。


## Excercises

> [CSES - Company Queries II](https://cses.fi/problemset/task/1688/)
> 
> 給一棵 $N$ 個節點的有根樹，$Q$ 次詢問求節點 $a,b$ 的最低共同祖先(LCA)。
>
> 其中 \\(N,Q\leq2\times10^5\\)。

<details><summary> Solution </summary>

LCA 的做法有很多種，倍增法、樹壓平...等，這邊介紹的是輕重鏈剖分的作法。
這邊 dfs 建完需要的資訊後，查詢的部分就一直讓較深的那個節點往上跳，直到找到共同祖先，時間複雜度是 \\(O(N+Q\log N)\\)。

```cpp
#include <bits/stdc++.h>
using namespace std;
const int N = 2e5 + 5;
int t, n, q;
int sz[N], fa[N], dep[N], to[N], top[N];
vector<int> g[N];
void dfs(int x, int p) {
  sz[x] = 1, fa[x] = p, to[x] = -1, dep[x] = ~p ? dep[p] + 1 : 0;
  for (auto i : g[x])
    if (i != p) {
      dfs(i, x);
      if (to[x] == -1 || sz[i] > sz[to[x]]) to[x] = i;
      sz[x] += sz[i];
    }
}
void dfs2(int x, int f) {
  top[x] = f;
  if (to[x] != -1) dfs2(to[x], f);
  for (auto i : g[x])
    if (i != fa[x] && i != to[x]) dfs2(i, i);
}
int qry(int u, int v) { // query on tree
  int fu = top[u], fv = top[v];
  while (fu != fv) {
    if (dep[fu] < dep[fv]) swap(fu, fv), swap(u, v);
    u = fa[fu], fu = top[u];
  }
  if (dep[u] > dep[v]) swap(u, v);
  return u;
}
int main() {
  ios::sync_with_stdio(false), cin.tie(nullptr);
  cin >> n >> q;
  for (int i = 2, x; i <= n; i++)
    cin >> x, g[x].push_back(i);
  dfs(1, -1), dfs2(1, 1);
  while (q--) {
    int a, b; cin >> a >> b;
    cout << qry(a, b) << '\n';
  }
}
```

</details>

> [CSES - Path Queries II](https://cses.fi/problemset/task/2134/)
> 
> 給一棵 $N$ 個節點的無根樹，$Q$ 次詢問求節點 $a,b$ 路徑上的最大值，帶修改。
>
> 其中 \\(N,Q\leq2\times10^5\\)。

<details><summary> Solution </summary>

在每一條重鏈上建立線段樹，因為 dfs 的時間戳記是連續的，所以可以很輕易做到區間查詢、區間更新的操作。可以發現我們每次要查詢的部分是區間是 \\([dfn[fu],dfn[u]]\\)，其中 \\(u\\) 是較深的那個節點，

```cpp
#include <bits/stdc++.h>
using namespace std;
const int N = 2e5 + 5;
int t, n, q, seg[N << 1];
int sz[N], fa[N], dep[N], to[N], top[N], dfn[N], arr[N];
vector<int> g[N];
void upd(int x, int v) {
  for (seg[x += n] = v; x > 1; x >>= 1)
    seg[x >> 1] = max(seg[x], seg[x ^ 1]);
}
int qry(int l, int r) {
  int ret = -1e9;
  for (l += n, r += n + 1; l < r; l >>= 1, r >>= 1) {
    if (l & 1) ret = max(ret, seg[l++]);
    if (r & 1) ret = max(ret, seg[--r]);
  }
  return ret;
}
void dfs(int x, int p) {
  sz[x] = 1, fa[x] = p, to[x] = -1, dep[x] = ~p ? dep[p] + 1 : 0;
  for (auto i : g[x]) {
    if (i == p) continue;
    dfs(i, x);
    if (to[x] == -1 || sz[i] > sz[to[x]]) to[x] = i;
    sz[x] += sz[i];
  }
}
void dfs2(int x, int f) {
  top[x] = f, dfn[x] = ++t, upd(dfn[x], arr[x]);
  if (to[x] != -1) dfs2(to[x], f);
  for (auto i : g[x]) {
    if (i == fa[x] || i == to[x]) continue;
    dfs2(i, i);
  }
}
int qry2(int u, int v) { // query on tree
  int fu = top[u], fv = top[v], ret = -1e9;
  while (fu != fv) {
    if (dep[fu] < dep[fv]) swap(fu, fv), swap(u, v);
    ret = max(ret, qry(dfn[fu], dfn[u]));
    u = fa[fu], fu = top[u];
  }
  if (dep[u] > dep[v]) swap(u, v);
  ret = max(ret, qry(dfn[u], dfn[v]));
  return ret;
}
int main() {
  ios::sync_with_stdio(false), cin.tie(nullptr);
  cin >> n >> q;
  for (int i = 1; i <= n; i++) cin >> arr[i];
  for (int i = 1, a, b; i < n; i++)
    cin >> a >> b, g[a].push_back(b), g[b].push_back(a);
  dfs(1, -1), dfs2(1, 1);
  while (q--) {
    int cmd; cin >> cmd;
    if (cmd == 1) {
      int x, v; cin >> x >> v, arr[x] = v, upd(dfn[x], v);
    }
    else {
      int a, b; cin >> a >> b;
      cout << qry2(a, b) << '\n';
    }
  }
}
```

</details>

> [SPOJ - QTREE - Query on a tree](https://www.spoj.com/problems/QTREE/)
> 
> 給一棵 \\(N-1\\) 條邊資訊的無根樹，每次詢問求節點 \\(a,b\\) 路徑上的最大值，帶修改。
>
> 其中 \\(N\leq10^4\\)。

<details><summary> Solution </summary>

一樣是樹鏈剖分，這次的題目是給定邊的資訊，那我們其實只要把每條邊的值記錄在較深的那個節點即可。要注意的是查詢時，LCA 不能計算進答案。


```cpp
#include <bits/stdc++.h>
using namespace std;
const int N = 1e4 + 5;
struct edge {
  int nxt, cost, id;
};
int t, n, seg[N << 1];
int sz[N], fa[N], dep[N], to[N], top[N], dfn[N], ptn[N], arr[N];
vector<edge> g[N];
void upd(int x, int v) {
  for (seg[x += n] = v; x > 1; x >>= 1)
    seg[x >> 1] = max(seg[x], seg[x ^ 1]);
}
int qry(int l, int r) {
  int ret = 0;
  for (l += n, r += n + 1; l < r; l >>= 1, r >>= 1) {
    if (l & 1) ret = max(ret, seg[l++]);
    if (r & 1) ret = max(ret, seg[--r]);
  }
  return ret;
}
void dfs(int x) {
  sz[x] = 1, to[x] = -1;
  for (int i = 0; i < (int)g[x].size(); i++) {
    auto [nxt, cost, id] = g[x][i];
    if (nxt == fa[x]) continue;
    fa[nxt] = x, dep[nxt] = dep[x] + 1, arr[nxt] = cost;
    dfs(nxt);
    if (to[x] == -1 || sz[nxt] > sz[g[x][to[x]].nxt]) to[x] = i;
    sz[x] += sz[nxt];
  }
}
void dfs2(int x, int f, int idx) {
  top[x] = f, ptn[idx] = dfn[x] = ++t, upd(dfn[x], arr[x]);
  if (to[x] != -1) dfs2(g[x][to[x]].nxt, f, g[x][to[x]].id);
  for (auto [nxt, cost, id] : g[x]) {
    if (nxt == fa[x] || nxt == g[x][to[x]].nxt) continue;
    dfs2(nxt, nxt, id);
  }
}
int qry2(int u, int v) { // query on tree
  int fu = top[u], fv = top[v], ret = 0;
  while (fu != fv) {
    if (dep[fu] < dep[fv]) swap(fu, fv), swap(u, v);
    ret = max(ret, qry(dfn[fu], dfn[u]));
    u = fa[fu], fu = top[u];
  }
  if (dep[u] > dep[v]) swap(u, v);
  if (u != v) ret = max(ret, qry(dfn[u] + 1, dfn[v]));
  return ret;
}
int main() {
  ios::sync_with_stdio(false), cin.tie(nullptr);
  int ts; cin >> ts;
  while (ts--) {
    cin >> n;
    for (int i = 1; i <= n; i++) g[i].clear();
    memset(seg, 0, sizeof(seg)), t = 0;
    for (int i = 1, a, b, c; i < n; i++) {
      cin >> a >> b >> c;
      g[a].push_back({b, c, i}), g[b].push_back({a, c, i});
    }
    dfs(1);
    dfs2(1, 1, 0);
    for (string cmd; cin >> cmd, cmd[0] != 'D';) {
      if (cmd[0] == 'Q') { // QUERY
        int a, b; cin >> a >> b;
        cout << qry2(a, b) << '\n';
      }
      else { // CHANGE
        int i, ti; cin >> i >> ti;
        upd(ptn[i], ti);
      }
    }
  }
}
```

</details>

> [CF 2022 I2CP Mock Contest - A. Palindrome Path](
https://codeforces.com/group/dnlUA4rsoS/contest/380617/problem/A)
> 
> 給一棵 \\(N\\) 個節點的無根樹，\\(Q\\) 次詢問求節點 \\(a,b\\) 路徑上的資訊是否可以重組而形成迴文，帶修改。
>
> 其中 \\(N,Q\leq2\times10^5\\)。

<details><summary> Solution </summary>

組成迴文的條件是字母數都必須是偶數或有一個字母可以是奇數。

我的做法是對字母 a-z 做 hash。把問題轉換成路徑上 xor sum，如果 xor sum 等於零或是出現在字母 hash 出的東西裡面，那答案就是 YES，反之 NO。

```cpp
#include <bits/stdc++.h>
using namespace std;
const int N = 2e5 + 5;
int t, n, q, seg[N << 1];
int sz[N], fa[N], dep[N], to[N], top[N], dfn[N], arr[N], ch[26];
vector<int> g[N];
mt19937 rng;
void upd(int x, int v) {
  for (seg[x += n] = v; x > 1; x >>= 1)
    seg[x >> 1] = seg[x] ^ seg[x ^ 1];
}
int qry(int l, int r) {
  int ret = 0;
  for (l += n, r += n + 1; l < r; l >>= 1, r >>= 1) {
    if (l & 1) ret ^= seg[l++];
    if (r & 1) ret ^= seg[--r];
  }
  return ret;
}
void dfs(int x, int p) {
  sz[x] = 1, fa[x] = p, to[x] = -1, dep[x] = ~p ? dep[p] + 1 : 0;
  for (auto i : g[x]) {
    if (i == p) continue;
    dfs(i, x);
    if (to[x] == -1 || sz[i] > sz[to[x]]) to[x] = i;
    sz[x] += sz[i];
  }
}
void dfs2(int x, int f) {
  top[x] = f, dfn[x] = ++t, upd(dfn[x], arr[x]);
  if (to[x] != -1) dfs2(to[x], f);
  for (auto i : g[x]) {
    if (i == fa[x] || i == to[x]) continue;
    dfs2(i, i);
  }
}
int qry2(int u, int v) { // query on tree
  int fu = top[u], fv = top[v], ret = 0;
  while (fu != fv) {
    if (dep[fu] < dep[fv]) swap(fu, fv), swap(u, v);
    ret ^= qry(dfn[fu], dfn[u]);
    u = fa[fu], fu = top[u];
  }
  if (dep[u] > dep[v]) swap(u, v);
  ret ^= qry(dfn[u], dfn[v]);
  return ret;
}
int main() {
  ios::sync_with_stdio(false), cin.tie(nullptr);
  for (auto& i : ch) i = rng();
  cin >> n >> q;
  for (int i = 1; i <= n; i++) {
    char c; cin >> c, arr[i] = ch[c - 'a'];
  }
  for (int i = 1, a, b; i < n; i++)
    cin >> a >> b, g[a].push_back(b), g[b].push_back(a);
  dfs(1, -1), dfs2(1, 1);
  while (q--) {
    string cmd; cin >> cmd;
    if (cmd[0] == 'C') { // CHANGE
      int x; char c; cin >> x >> c, arr[x] = ch[c - 'a'], upd(dfn[x], arr[x]);
    }
    else { // QUERY
      int a, b; cin >> a >> b;
      int res = qry2(a, b);
      bool flag = 0;
      for (auto i : ch)
        if (res == i) flag = 1;
      cout << (!res || flag ? "YES" : "NO") << '\n';
    }
  }
}
```

</details>

## Other Problems

> [LibreOJ 2236. 松鼠的新家](https://loj.ac/p/2236)
> 
> 給一棵 \\(N\\) 個節點的樹，每次對於一個路徑上加值，最後問各個節點上的值。
> 其中 \\(N\leq3\times10^5\\)。

> [Zerojudge b322. 樹形鎖頭](https://zerojudge.tw/ShowProblem?problemid=b322)
> 
> 給定一顆 \\(N\\) 個點的樹， \\(M\\) 次操作 \\((u,v,k)\\)，將路徑 \\(u,v\\) 之間經過的節點權重加上 \\(k\\)，試問最後各個節點的值為多少。

> [CF 601 - D. Tree Queries](https://codeforces.com/contest/1254/problem/D)
> 
> 給定一顆 \\(N\\) 個點的樹，\\(Q\\) 次詢問，兩種操作如下：
> \\(1,v,d\\) : 選一個節點 \\(v\\)、數字 \\(d\\)，然後隨機取一個節點 \\(r\\)，列出所有節點 \\(u\\) 使得 \\(v,r\\) 的路徑經過 \\(u\\)，並將這些 \\(u\\) 上的數字增加 \\(d$\\)。
> \\(2,v\\) : 查詢節點 \\(v\\) 的期望值。
> 其中 \\(N,Q\leq1.5\times10^5\\)。

> [CF gym 101808 - L. V--o$\_$o--V](https://codeforces.com/gym/101808)
> 
> 給一段有關於樹上 LCA 問題的 code，要用更有效率的方式做出來。
> 其中 \\(N\leq2\times10^5\\)。

## Reference
- IOICamp 2021 Handout
- [cp-algorithms Heavy-light decomposition](https://cp-algorithms.com/graph/hld.html)
- [樹鏈剖分 - OI Wiki](https://oi-wiki.org/graph/hld/)