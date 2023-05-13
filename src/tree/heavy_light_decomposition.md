---
tags: school
---

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
對於一顆無根樹，選擇隨機一個節點，對他做深度優先搜尋(DFS)，在遞迴的過程中，我們可以紀錄每個節點的父節點(father node)、深度(deep)、大小(size)、重小孩(heavy child)。紀錄這些的目的是之後做查詢比較方便，有些資料也必須用於第二次的深度優先搜尋去建構樹鏈剖分。

### Second DFS
經過第一次深度優先搜尋後，我們已經有每個節點重小孩的資訊了。第二次的深度優先搜尋，我們要做的是優先遞迴重小孩去建構重鏈(heavy chain)，之後再遞迴其他小孩，也就是輕小孩，去構造輕鏈(light chain)。

在這個部分，我們要紀錄時間戳記、鏈的起點。紀錄時間戳記通常是因為我們在遞迴時，一條鏈所在資訊具有連續性，所以可以記錄鏈上最大值、最小值、總和等資訊；紀錄鏈的起點是之後做查詢的時候，快速找到需要的資訊和做跳躍的動作。

### Build Data Structures on Each Heavy Chain
由於我們已知深度優先搜尋遞迴拜訪重鏈編號的連續性，所以可以得到 `[dfn[top[x]], dfn[x]]` 這個區間是我們要的鏈上資訊。所以我們可以在第二次深度優先搜尋的時候記錄 `dfn[x]=++timestamp`，並對 `dfn[x]` 做資料結構的更新，即可完成紀錄。

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
```cpp=
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
定義 \\(s(v)\\) 為節點 \\(v\\) 的子數大小。我們可以知到每一個節點 \\(u\\) 只會有一個重小孩 \\(v\\)，且 \\(s(v)\geq\frac{s(u)}{2}\\)，可以由反證法得證。假設有兩個重小孩，那麼 \\(s(u)\geq1+2\times\frac{s(v)}{2}>s(u)\\)，產生矛盾。

因為每個節點會往最大子樹的小孩連接重邊，其餘為輕邊。因此在此演算法的時間複雜度相當於考慮「跳了多少條輕邊」，又因為每次跳過輕邊時，子樹大小會減少至少一半（根據上面的證明），在每次都是至少減半的過程中，可以得知一個節點 \\(u\\) 要跳至根結點，至多只會跳 \\(O(\log N)\\) 條輕邊，也就是說有 \\(O(\log N)\\) 條樹鍊，那顯然任兩個節點的路徑上也只會有 \\(O(\log N)\\) 條樹鍊。


## Excercises