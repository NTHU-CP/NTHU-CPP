# Centroid Decomposition

## Introduction

此技巧也被稱作點分治、重心分治、重心剖分

重心剖分是樹重心的一個很神奇的應用，透過不斷的找重心並刪除，我也們可以在樹上做到分治一樣的事情。

## Centroid Decomposition

在詳細介紹重心剖分之前，我們先來介紹一個新名詞—— **重心樹**，它可以幫助我們更好的了解重心剖分的性質。

### 什麼是重心樹？

重心樹，其實就只是把重心剖分不斷找重心的過程表示成另一棵樹

對於一棵無根樹 \\( T \\)，求重心樹方法如下：

- 找出 \\( T \\) 的重心 \\( c \\)
- 將重心 \\( c \\) 從樹上刪除，\\( T \\) 會被劃分為若干棵樹 \\( T_1, ..., T_k \\) (假設被分為 \\( k \\) 棵)
- 在重心樹上，\\( c \\) 即為 \\( T_1, ..., T_k \\) 的重心節點的父親。
- 對 \\( T_1, ..., T_k \\) 遞迴建立重心樹

對比一下原樹 \\( T \\) 和他的重心樹的差異

<div style="display:flex; flex-direction:row;">
    <div>
        <p align="center">
            <img src="images/0.png" style="height:250px">
        </p>
        <p style="text-align: center;"> 原樹 </p>
    </div>
    <div>
        <p align="center">
            <img src="images/1.png" style="height:250px">
        </p>
        <p style="text-align: center;"> 重心樹 </p>
    </div>
</div>

可以參考下面的 GIF 來體會一下每次拔掉重心的過程

<p align="center">
    <img src="images/tree.gif" style="height:250px">
</p>
<p style="text-align: center;"> 節點刪完之時，重心樹歸來之日XD </p>

### 重心樹實作

接著來看看如何實作建出一棵重心樹。

- `tree` 儲存原樹的鄰接串列

- `centroid_tree` 儲存重心樹的鄰接串列(解題時通常不會用到)

- `del[i]` 紀錄點 \\(i\\) 是否已經被拔除

- `pa[i]` 紀錄點 \\(i\\) 在重心樹上的父親

注意：需要先呼叫 `get_sz()` 才能呼叫 `get_centroid()`

```cpp
struct CentroidTree {
  vector<vector<int>> tree, centroid_tree;
  vector<int> del, pa, sz;

  CentroidTree(int n)
      : tree(n + 1, vector<int>()), centroid_tree(n + 1, vector<int>()),
        del(n + 1), pa(n + 1), sz(n + 1, 1) {}

  void add_edge(int u, int v) {
    tree[u].emplace_back(v);
    tree[v].emplace_back(u);
  }

  void get_sz(int u, int p = -1) {
    sz[u] = 1;
    for (int v : tree[u]) {
      if (v == p || del[v])
        continue;
      get_sz(v, u);
      sz[u] += sz[v];
    }
  }

  int get_centroid(int u, int n, int p = -1) {
    for (int v : tree[u]) {
      if (v != p && !del[v] && sz[v] > n / 2)
        return get_centroid(v, n, u);
    }
    return u;
  }

  // build the centroid tree recursively
  int build(int u = 1) {
    get_sz(u, -1);
    int centroid = get_centroid(u, sz[u], -1);
    del[centroid] = 1; // delete centroid
    // Note : u might not be a centroid

    /*******
    do something
    ********/

    for (int v : tree[centroid]) {
      if (del[v])
        continue;
      int tcd = build(v);
      pa[tcd] = centroid;
      centroid_tree[centroid].emplace_back(tcd);
    }
    return centroid;
  }
};
```

以上的模板只是為了讓大家能更好的暸解重心樹，下面會提供支援更多操作的解題模板。

建立出重心樹其實很簡單，只是不斷的找出重心並刪掉而已，但要小心不要走到已經刪除的點上。

困難的地方在於重心樹性質的運用，以下是重心樹的一些重要性質以及分析。

### Time Complexity

重心剖分、建立重心樹只需要 \\( O(N \log {N}) \\) 的時間 (下面會說明)

### 重心樹性質

1. 重心樹的深度是 \\( O(\log{N}) \\)
    <details><summary> 說明 </summary>

    首先有一個重要的觀察，重心樹的深度其實也就是我們在建樹過程中的遞迴深度，所以可以順便證明建立重心樹的時間複雜度。

    重心樹的深度是 \\( O(\log{N}) \\)，因為在我們每次遞迴，拔掉重心所形成的子樹大小都不會超過原樹的一半，所以最多經過 \\( \log{N} \\) 次，所有子樹大小都會 \\( \le 1 \\)。

    因為對於重心樹上的每一層都需要花 \\(O(N)\\) 的時間找出該層每個子樹的重心，所以總時間複雜度是 \\(O(N \log{N})\\)。

    換一種說法，也就是 \\( \sum\limits_{u = 1}^{N} sz(u) = O(N \log N) \\)。(\\( sz(u) \\) 是節點 \\( u \\) 在重心樹上的子樹大小)
    </details>

1. 重心樹的任一子樹，在原樹中會構成恰好一個連通塊。

    <details><summary> 說明 </summary>

    先提醒一下，在重心樹上**直接**相鄰的兩個點，在原樹中不一定**直接**相鄰。

    回想一下重心樹建立的過程，重心樹上以 \\( u \\) 為根的子樹所對應的原樹中的連通塊，就是刪除 \\( u \\) 點的父親節點時所產生的連通塊其中之一。
    </details>

1. 對於任意兩點 \\( a, b \\)，在原樹中從 \\(a\\) 走到 \\(b\\) 一定會經過 \\( LAC'(a, b) \\)。(\\( LCA'(a, b) \\) 表示在**重心樹**上的最低公共祖先)

    <details><summary> 說明 </summary>

    根據性質二，當 \\(a, b \\) 還在重心樹上的同一個子樹上時他們在原樹上連通，反之，\\(a, b\\) 最早開始不連通的時候就是當 \\( LCA'(a, b) \\) 被刪除時。

    既然刪除 \\( LCA'(a, b) \\) 會使得 \\(a, b\\) 在原樹上不連通，那表示 \\( LCA'(a, b) \\) 一定會出現在原樹中從 \\(a\\) 走到 \\(b\\) 的路徑上。

    接下來的題目中，這個性質會一直出現。舉例來說，我們想枚舉從一個固定點 \\(x\\) 出發到其他所有點的路徑，而這些路徑一定可以被拆分成 \\( x \rightarrow LCA'(x, y) \rightarrow y\\) ( \\( y \\) 是我們要前往的點)。

    根據性質一，\\(x\\) 的祖先的數量是 \\(O(\log{N})\\)，所以 \\( LCA'(x, y) \\) 的可能數量也是。因此我們只需要枚舉 \\(x\\) 的所有祖先，並在上面紀錄我們需要的資訊，就能達到用 \\(O(\log{N})\\) 個點枚舉完 \\(O(N)\\) 條路徑的效果。
    </details>

大部分的題目都是由以上幾個性質衍生出來的。

## Examples

> [CF 321C - Ciel the Commander](https://codeforces.com/contest/321/problem/C)
>
> 有一個 \\(n\\) 個節點的樹，你要為每個點填入 'A' 到 'Z' 其中一個字母並滿足以下條件：
>
> 任兩個有相同字母的節點 \\(u, v\\)，必須有一個節點 \\(x\\) 在 \\(u\\) 到 \\(v\\) 的路徑上，且 \\(x\\) 上的字母的字典序嚴格小於 \\(u, v\\) 上的字母，請輸出一組滿足條件的解。
>
> - \\( 1 \le n \le 10^5 \\)
>

利用性質 3，我們可以把重心樹的每一層都填上同一個英文字母，並且按照深度由字母序小到大填入。再根據性質 1，重心樹的高度不超過 \\( \log{N} \\)，\\( \log{10^5} \approx 17 < 26 \\)，我們一定可以找到一組解。

> [CSES Fixed-Length Path I](https://cses.fi/problemset/task/2080)
>
> 給定一棵無根樹和一個整數 \\( k \\)，請問樹上長度為 \\( k \\) 的路徑有幾條？
>
> - \\(1 \le k \le n \le 2 \times 10 ^ 5\\)

這個題目有很多種作法，甚至有 \\(O(n) \\) 的樹上啟發式合併做法，但這邊只會講解重心剖分的做法。

重心分治很適合處理這中需要找出樹上 **所有** 路徑中，符合題目給定條件之路徑的題目。就像我們會用分治來做處理一個序列的各種區間問題類似，這也是為什麼重心剖分又被稱作重心分治、點分治。

在做重心剖分的過程中，我們可以把路徑分為兩類：

- 經過重心的路徑
- 不經過重心的路徑

不經過重心的路徑一定存在拔除重心後所產生之連通塊中，所以只需要遞迴處理即可。因此我們專心處理通過重心且符合條件的路徑就好，這樣就簡單多了。

以這一題來說，我們現在要處理的問題是必定經過重心且長度恰好為 \\(k\\) 的路徑數量，因為題目給定的樹是無根樹，那不妨先將重心設為根節點。

接下來我們能將經過根節點的路徑也分為兩類：

- 一端點為根節點的路徑
- 兩端點分別在根節點的兩個不同兒子節點的子樹內

注意：不要誤算到兩端點都來自同一個根節的點子節點子樹的路徑。

第一種非常好處理，從根節點開始做一次 DFS 就能完成。而第二種，我們可以對根節點的每個兒子節點紀錄，從這個點為端點往下到所有子孫節點的路徑長度，接著用合併的方式，每次合併一個子節點的子樹(這樣就不會有合併到同一個子節點子樹的點的問題)，將兩條從不同子節點出發的路徑合併成一條經過根節點的路徑來計算答案。

這個做法的複雜度就是重心剖分原本的複雜度 \\(O(n\log{n})\\)，因為我們只是多從重心開始做一次 DFS 而已，複雜度和找重心的 DFS 是完全一樣的。

<details><summary> Sample Code </summary>

在重心剖分的同時，每次都以 `centroid` 為根計算通過這個點且長度為 \\(k\\) 的路徑數量。以下幾點需要注意：

- `cnt` 要用類似 Undo 的方式去清空，不然時間複雜度可能會退化
- 在用 `dfs_ans()` 計算答案的時候一樣不能走到已經被刪除的點上

```cpp
#include <bits/stdc++.h>
using namespace std;

struct CentroidTree {
  vector<vector<pair<int, int>>> tree;
  vector<int> del, pa, sz, cnt;

  CentroidTree(int n)
      : tree(n + 1, vector<pair<int, int>>()), del(n + 1), pa(n + 1),
        sz(n + 1, 1), cnt(n + 1, 0) {}

  void add_edge(int u, int v, int w = 1) {
    tree[u].emplace_back(v, w);
    tree[v].emplace_back(u, w);
   }

  void get_sz(int u, int p) {
    sz[u] = 1;
    for (auto [v, w] : tree[u]) {
      if (v == p || del[v])
        continue;
      get_sz(v, u);
      sz[u] += sz[v];
    }
  }

  int get_centroid(int u, int n, int p) {
    for (auto [v, w] : tree[u]) {
      if (v != p && !del[v] && sz[v] > n / 2)
        return get_centroid(v, n, u);
    }
    return u;
  }

  void dfs_ans(vector<int> &path, int k, int u, int p = -1, int len = 1) {
    if (len > k)
      return;
    path.emplace_back(len);
    for (auto [v, w] : tree[u]) {
      if (v == p || del[v])
        continue;
      dfs_ans(path, k, v, u, len + w);
    }
  }

  // build the centroid tree recursively
  int build(int u, int k, long long &ans) {
    get_sz(u, -1);
    int centroid = get_centroid(u, sz[u], -1);
    del[centroid] = 1;

    // count path go through centroid
    cnt[0] = 1;
    int mx_depth = 0; // for clear up the array cnt
    for (auto [v, w] : tree[centroid]) {
      if (del[v])
        continue;
      vector<int> path_len;
      dfs_ans(path_len, k, v, -1, 1);
      for (int len : path_len) {
        ans += cnt[k - len];
        mx_depth = max(mx_depth, len);
      }
      for (int len : path_len) {
        cnt[len]++;
      }
    }
    // clear the array cnt
    for (int i = 0; i <= mx_depth; i++)
      cnt[i] = 0;

    for (auto [v, w] : tree[centroid]) {
      if (del[v])
        continue;
      build(v, k, ans);
    }
    return centroid;
  }
};

int main() {
  ios_base::sync_with_stdio(0);
  cin.tie(0);

  int n, k;
  cin >> n >> k;
  CentroidTree CD(n);
  for (int i = 0; i < n - 1; i++) {
    int u, v;
    cin >> u >> v;
    CD.add_edge(u, v);
  }

  long long ans = 0;
  CD.build(1, k, ans);
  cout << ans;
}
```

</details>

> [CF 342E - Xenia and Tree](https://codeforces.com/problemset/problem/342/E)
>
> 有一棵 \\( n \\) 個節點的樹，一開始所有節點都是藍色，要執行 \\( m \\) 次以下兩種操作：
>
> 1. 將一個藍色節點塗成紅色
>
> 2. 查詢一個節點 \\( x \\)，回答離 \\( x \\) 最近的紅色節點距離
>
> - \\( 1 \le n,m \le 10^5 \\)
>

這一題是非常經典的重心剖分應用，但是剛開始可能比較難看出來這題和重心剖分的關聯。

在性質 3 的說明裡有提過，對於一個點 \\( x \\)，我們只需要對 \\(x\\) 所有重心樹上的祖先紀錄需要的資訊。以這一題來說，我們需要對每個點 \\(u\\) 紀錄 \\( ans[u] \\) 代表在重心樹以 \\(u\\) 為根的子樹中離點 \\(u\\) 最近的紅色點距離。

接下來當一個節點 \\(x \\) 被染成紅色時，只需要對所有 \\( x \\) 在重心樹上的祖先，更新點 \\(x\\) 對他們的影響。具體來說，假設 \\(u\\) 為 \\(x\\) 在重心樹上的祖先，就用 \\( dis(x, u)\\) 去更新 \\(ans[u]\\)。

查詢一個節點 \\(x \\) 時，只需找出 \\(x\\) 到 \\( x \\) 在重心樹上的祖先，再到離他們最近的紅點的距離的最小值。具體來說，假設 \\(u\\) 為 \\(x\\) 在重心樹上的祖先，只需要把所有的 \\(dis(x, u) + ans[u]\\) 取最小值就會是答案。

這樣不管是查詢還是更改節點顏色的操作，我們都可以把 \\(x\\) **在重心樹上的祖先** 當作中繼站，只對這個集合做更改和查詢。因此每次操作只需要處理 \\(O(\log{n})\\) 個點。

我們在進行更新和查詢時，需要知道樹上任兩點之間的距離，這可以透過一些 \\(LCA\\) 演算法來完成，沒有學過的人可以參考[這裡](lca.html)。(注意：如果使用 \\( O(\log{n})\\) 查詢兩點距離的 \\(LCA\\) 演算法會使複雜度變差)

但由於我們只需要知道每個點到他的重心樹上的祖先的距離，所以有另一個可以在建立重心樹的過程中順便預處理距離，\\(O(1)\\) 查詢的方法，我會在下方 Sample Code 補充說明。

如使用 \\(O(1)\\) 查詢距離的方法，這題就能在 \\(O((n + m) \log{n}) \\) 時間內完成。

有一點需要稍微注意，如果將這題的查詢改成 **最遠** 的紅色點距離。我們所需要紀錄的資訊就不會這麼簡單了，因為在查詢時可能會遇到這種情況：

假設我們在查詢點 \\(x\\)， \\(u\\) 為 \\(x\\) 在重心樹上的祖先，且離 \\(u\\) 最遠的紅色節點為 \\(v\\)，所以我們會用 \\(x \rightarrow u \rightarrow v\\) 的距離來更新答案，但如果 \\( LCA'(x, v) \neq u \\)，那麼 \\(u\\) 很可能就不在 \\(x\\) 到 \\(v\\) 的路徑上，所以得出來的答案會比正確答案來得 **大**。

讀者可以思考一下為什麼這種錯誤不會在求 **最近** 距離的時候發生，而這種情況要怎麼處理會在下一題例題中被提到。

<details><summary> Sample Code </summary>

- `ans[i]` 紀錄離點 \\(i\\) 最近的紅色點的距離
- `dis[i]` 紀律點 \\(i\\) 到他所有祖先的距離(包括到自己)
- `get_dis()` 預處理每個節點到所有祖先節點的距離的函數
- `upd(), qry()` 在重心樹上網上爬，並做更新和查詢

在做重心剖分的過程中，我們每次都從重心開始多做一次 DFS，紀錄重心到其他也在同一個聯通塊中的點的距離，其實也就是對於這個重心，紀錄他到所有重心樹上的子孫節點的距離。由於我們做重心剖分是在重心樹上由上至下，而查詢的時候我們是由下而上往上爬，所以必須要反著去遍歷 `dis[]` 陣列

```cpp
#include<bits/stdc++.h>
#define inf 100000000
using namespace std;

struct CentroidTree {
  vector<vector<pair<int, int>>> tree;
  
  vector<vector<int>> centroid_tree, dis;
  vector<int> del, pa, sz;
  vector<int> ans;

  CentroidTree(int n)
      : tree(n + 1, vector<pair<int, int>>()),
        centroid_tree(n + 1, vector<int>()), dis(n + 1, vector<int>()),
        del(n + 1), pa(n + 1), sz(n + 1, 1), ans(n + 1, inf) {}

  void add_edge(int u, int v, int w) {
    tree[u].emplace_back(v, w);
    tree[v].emplace_back(u, w);
  }

  void get_sz(int u, int p) {
    sz[u] = 1;
    for (auto [v, w] : tree[u]) {
      if (v == p || del[v])
        continue;
      get_sz(v, u);
      sz[u] += sz[v];
    }
  }

  int get_centroid(int u, int n, int p) {
    for (auto [v, w] : tree[u]) {
      if (v != p && !del[v] && sz[v] > n / 2)
        return get_centroid(v, n, u);
    }
    return u;
  }

  void get_dis(int u, int p, int len) {
    dis[u].emplace_back(len);
    for (auto [v, w] : tree[u]) {
      if (v == p || del[v])
        continue;
      get_dis(v, u, len + w);
    }
  }

  // build the centroid tree recursively
  int build(int u = 1) {
    get_sz(u, -1);
    int centroid = get_centroid(u, sz[u], -1);
    del[centroid] = 1; // delete centroid
    // Note : u might not be a centroid
    get_dis(centroid, -1, 0);

    /*******
    do something
    ********/

    for (auto [v, w] : tree[centroid]) {
      if (del[v])
        continue;
      int tcd = build(v);
      pa[tcd] = centroid;
      centroid_tree[centroid].emplace_back(tcd);
    }
    return centroid;
  }

  void upd(int x) {
    int u = x;
    for (int i = dis[x].size() - 1; i >= 0; i--) {
      int dist = dis[x][i]; // dist 為 x, u 之間的距離
      /*******
      do something
      ********/
      ans[u] = min(ans[u], dist); 
      u = pa[u];
    }
  }

  int qry(int x) {
    int u = x, res = inf;
    for (int i = dis[x].size() - 1; i >= 0; i--) {
      int dist = dis[x][i]; // dist 為 x, u 之間的距離
      /*******
      do something
      ********/
      res = min(res, dist + ans[u]);
      u = pa[u];
    }
    return res;
  }
};

int main() {
  ios_base::sync_with_stdio(0); cin.tie(0);
  int n, m;
  cin >> n >> m;
  CentroidTree CD(n);
  for (int i = 0; i < n - 1; i++) {
    int u, v;
    cin >> u >> v;
    CD.add_edge(u, v, 1);
  }

  CD.build(1);
  CD.upd(1);

  while (m--) {
    int o, x;
    cin >> o >> x;
    if (o == 1) {
      CD.upd(x);
    } else {
      cout << CD.qry(x) << '\n';
    }
  }
}
```

</details>

> [CSacademy Black-White-Tree](https://csacademy.com/contest/archive/task/black-white-tree/)
>
> 給定一棵 \\(n\\) 個節點的樹，每個節點不是白色就是黑色，要 \\(m\\) 次支援以下操作：
>
> 1. 改變一個節點的顏色(黑變白、白變黑)
>
> 2. 查詢一個點 \\( x \\) 到其他所有相同顏色的點的距離和
>
> - \\(1 \le n, m \le 5 \times 10 ^ 4\\)

這一題和上一題 **例題 Xenia and the Tree** 類似，而且這兩題其實都有很好寫的根號作法，可以參考[這裡](../sqrt/sqrt_decomposition.html#操作分塊)。

但是如果要使用重心剖分的解法，需要處裡上一題最後所提到的答案多算的問題。這邊就和大家講一下要如何解決。

首先這一題一樣是需要在重心樹上往上爬並做更改，所以先考慮一下要紀錄什麼資訊。

對於每個點 \\(u\\) 我們需要對兩種顏色分別紀錄 \\(sum_u\\) 和 \\(cnt_u\\)，代表到點 \\(u\\) 到它重心樹上的所有那個顏色子孫節點的 **距離總和** 和 **數量**。

如此一來，我們就能用 \\( dis(x, v) \times cnt_v + sum_v \\) 來算出點 \\(x\\) 經過他在重心樹上的祖先 \\(v\\) 所能到達的所有黑/白點的距離和。但其實這個作法是錯的，算出來的答案都會比真實答案來得大，舉一個最好懂的例子來說，假設點 \\(x\\) 是黑色，那它一定會對他所有重心樹上的祖先紀錄黑色的 \\(sum, cnt\\) 有所貢獻，所以這樣我們就會計算到 \\(x\\) 透過自己的祖先再走回到 \\(x\\) 自己這樣的一條路徑，但這樣很顯然是有問題的。

更一般來說，當我們在計算一個點 \\(x\\) 經過它在重心樹上的祖先 \\(v\\) 的某個顏色路徑數量時，我們不能計算到 \\(LCA'(x, y) \neq v\\) 的情況(假設 \\(y\\) 路徑的另一個端點)。

所以，在計算點 \\(x\\) 經過他的祖先 \\(v\\) 的路徑時，\\(x\\) 一定在 \\(v\\) 的某一個 **子節點** 的子樹內(假設這個子節點為 \\(s\\))，我們就不能再從 \\(s\\) 的子樹內選擇這條路徑的另一端點，所以 \\( sum_v, cnt_v\\) 必須扣掉從 \\(s\\) 節點來的貢獻。這聽起來很麻煩，但其實我們一樣只需要對於每個點 \\(u\\) 紀錄 \\(subsum_u, subcnt_u\\) 代表以 \\(u\\) 為根的子樹對 \\(u\\) 的父親節點的 \\(sum, cnt\\) 有多少貢獻，查詢的時候再把它扣掉就好(上面皆在敘述重心樹非原樹)。時間複雜度仍是 \\(O(N \log{N})\\)。

所以之後遇到這種類型的重心剖分題目，就可以嘗試用類似的方法去扣掉多算。

另外，這題也有用輕重鏈剖分來做的方法，但時間複雜度為 \\(O((n + m) \log^{2}{n})\\)。可以參考[這篇](https://omeletwithoutegg.github.io/2019/12/04/TIOJ-1171/)，雖然題目不太一樣，但概念是差不多的。

<details><summary> Sample Code </summary>

實作上，在 `upd()` 和 `qry()` 往上爬的過程中，多用一個變數 `lst` 紀錄是從哪裡爬過來。

- `sum, cnt` 用法如上所述，`sum[0], cnt[0]` 紀錄白色點、`sum[1], cnt[1]` 紀錄黑色點
- `subcnt` 這可以被 `cnt` 直接取代，所以下面程式碼中完全沒用到
- `subsum` 一樣 `subsum[0]` 紀錄白色點、`sumsum[1]` 紀錄黑色點

```cpp
#include <bits/stdc++.h>
using namespace std;

const int mxN = 5e4 + 5;
int col[mxN];

struct CentroidTree {
  vector<vector<pair<int, int>>> tree;

  vector<vector<int>> centroid_tree, dis;
  vector<int> del, pa, sz;

  // for this problem
  vector<int> sum[2], cnt[2], subsum[2], subcnt[2];

  CentroidTree(int n)
      : tree(n + 1, vector<pair<int, int>>()),
        centroid_tree(n + 1, vector<int>()), dis(n + 1, vector<int>()),
        del(n + 1), pa(n + 1), sz(n + 1, 1) {
    sum[0].resize(n + 1);
    sum[1].resize(n + 1);
    cnt[0].resize(n + 1);
    cnt[1].resize(n + 1);
    subsum[0].resize(n + 1);
    subsum[1].resize(n + 1);
  }

  void add_edge(int u, int v, int w = 1) {
    tree[u].emplace_back(v, w);
    tree[v].emplace_back(u, w);
  }

  void get_sz(int u, int p) {
    sz[u] = 1;
    for (auto [v, w] : tree[u]) {
      if (v == p || del[v])
        continue;
      get_sz(v, u);
      sz[u] += sz[v];
    }
  }

  int get_centroid(int u, int n, int p) {
    for (auto [v, w] : tree[u]) {
      if (v != p && !del[v] && sz[v] > n / 2)
        return get_centroid(v, n, u);
    }
    return u;
  }

  void get_dis(int u, int p, int len) {
    dis[u].emplace_back(len);
    for (auto [v, w] : tree[u]) {
      if (v == p || del[v])
        continue;
      get_dis(v, u, len + w);
    }
  }

  // build the centroid tree recursively
  int build(int u) {
    get_sz(u, -1);
    int centroid = get_centroid(u, sz[u], -1);
    del[centroid] = 1; // delete centroid
    get_dis(centroid, -1, 0);

    for (auto [v, w] : tree[centroid]) {
      if (del[v])
        continue;
      int tcd = build(v);
      pa[tcd] = centroid;
      centroid_tree[centroid].emplace_back(tcd);
    }
    return centroid;
  }

  void upd(int x, int col, int toggle) { // call build() before upd
    int u = x, lst = 0;
    for (int i = dis[x].size() - 1; i >= 0; i--) {
      int dist = dis[x][i]; // dist is the distance between x, u
      if (lst) {
        subsum[col][lst] += dist;
        if (toggle)
          subsum[col ^ 1][lst] -= dist;
      }
      sum[col][u] += dist;
      cnt[col][u]++;
      if (toggle) {
        sum[col ^ 1][u] -= dist;
        cnt[col ^ 1][u]--;
      }
      lst = u;
      u = pa[u];
    }
  }

  int qry(int x, int col) { // call build() before qry
    int u = x, res = 0, lst = 0;
    for (int i = dis[x].size() - 1; i >= 0; i--) {
      int dist = dis[x][i]; // dist is the distance between x, u
      if (lst) {
        res += sum[col][u] - subsum[col][lst] +
               (cnt[col][u] - cnt[col][lst]) * dist;
      } else {
        res += sum[col][u] + cnt[col][u] * dist;
      }
      lst = u;
      u = pa[u];
    }
    return res;
  }
};

int main() {
  ios_base::sync_with_stdio(0);
  cin.tie(0);
  int n, m;
  cin >> n >> m;
  CentroidTree CD(n);

  for (int i = 1; i <= n; i++) {
    cin >> col[i];
  }

  for (int i = 0; i < n - 1; i++) {
    int u, v;
    cin >> u >> v;
    CD.add_edge(u, v);
  }

  CD.build(1);

  for (int i = 1; i <= n; i++) {
    CD.upd(i, col[i], 0); // toggle = 0
  }

  while (m--) {
    int o, x;
    cin >> o >> x;
    if (o == 1) {
      col[x] ^= 1;
      CD.upd(x, col[x], 1); // toggle = 1
    } else {
      cout << CD.qry(x, col[x]) << '\n';
    }
  }
}
```

</details>

### Code

這邊附上筆者解題用的模板。多加入了預處理每個點到重心樹祖先的距離，和查詢、修改的雛形。

提醒一下，因為重心剖分的運用比較多元，所以筆者提供的模板只是幫忙先實作出重心剖分一些比較繁瑣的部分，但要真正運用在題目上，可能還需要自己多加入一些函數、多紀錄一些資訊等等。建議大家都要自己實作看看，才能更好地理解重心剖分(只想著套模板是不可行的)。

<details><summary> Template </summary>

```cpp
struct CentroidTree {
  vector<vector<pair<int, int>>> tree;

  vector<vector<int>> centroid_tree, dis;
  vector<int> del, pa, sz;

  CentroidTree(int n)
      : tree(n + 1, vector<pair<int, int>>()),
        centroid_tree(n + 1, vector<int>()), dis(n + 1, vector<int>()),
        del(n + 1), pa(n + 1), sz(n + 1, 1) {}

  void add_edge(int u, int v, int w = 1) {
    tree[u].emplace_back(v, w);
    tree[v].emplace_back(u, w);
  }

  void get_sz(int u, int p) {
    sz[u] = 1;
    for (auto [v, w] : tree[u]) {
      if (v == p || del[v])
        continue;
      get_sz(v, u);
      sz[u] += sz[v];
    }
  }

  int get_centroid(int u, int n, int p) {
    for (auto [v, w] : tree[u]) {
      if (v != p && !del[v] && sz[v] > n / 2)
        return get_centroid(v, n, u);
    }
    return u;
  }

  void get_dis(int u, int p, int len) {
    dis[u].emplace_back(len);
    for (auto [v, w] : tree[u]) {
      if (v == p || del[v])
        continue;
      get_dis(v, u, len + w);
    }
  }

  // build the centroid tree recursively
  int build(int u = 1) {
    get_sz(u, -1);
    int centroid = get_centroid(u, sz[u], -1);
    del[centroid] = 1;
    get_dis(centroid, -1, 0);

    // so something

    for (auto [v, w] : tree[centroid]) {
      if (del[v])
        continue;
      int tcd = build(v);
      pa[tcd] = centroid;
      centroid_tree[centroid].emplace_back(tcd);
    }
    return centroid;
  }

  void upd(int x) {
    int u = x, lst = 0;
    for (int i = dis[x].size() - 1; i >= 0; i--) {
      int dist = dis[x][i];
      // do something
      lst = u;
      u = pa[u];
    }
  }

  int qry(int x, int col) {
    int u = x, res = 0, lst = 0;
    for (int i = dis[x].size() - 1; i >= 0; i--) {
      int dist = dis[x][i];
      // do something
      lst = u;
      u = pa[u];
    }
    return res;
  }
}
```

</details>

## Exercises

> [CSES Fixed-Length Path II](https://cses.fi/problemset/task/2081)
>
> 給定一棵樹和兩個整數 \\( k_1, k_2 \\)，請問樹上長度介在 \\([k_1, k_2]\\) 的路徑有幾條？
>
> - \\(1 \le k_1 \le k_2 \le n \le 2 \times 10 ^ 5\\)

<details><summary> Solution </summary>

這題和上面的 Fixed-Length Path I 一樣，有 \\( O(N) \\) 的樹上啟發式做法，想學的人可以看[這篇](https://usaco.guide/problems/cses-2081-fixed-length-paths-ii/solution)。

我們將 Fixed-Length Path 紀錄的 DP 資訊改成前綴和，並用資料結構(ex. BIT)來維護即可，但是目前這一題的重心剖分 + BIT 解法可能會在最後幾筆測資 TLE，想要通過的話可以用上面的 \\( O(N) \\) 作法。

</details>

> [CF 293E - Close Vertices](https://codeforces.com/problemset/problem/293/E)
>
> 給定一棵有邊權的樹和兩個變數 \\( l, w \\)，> 問你樹上有多少點對 \\( (u, v) \\) 滿足以下條件：
>
> 1. \\( (u, v) \\) 路徑上的邊數量 \\( \le l \\)
>
> 2. \\( (u, v) \\) 路徑上的邊權和 \\( \le w \\)
>
> - \\(1 \le l \le n \le 10 ^ 5, 0 \le w \le 10 ^ 9\\)

<details><summary> Solution </summary>

這題和上面的 Fixed-Length Path II 長得很像，但是又多了一個條件。

所以其實只要再將前一題的資料結構換成二維資料結構就能通過此題了，但是不建議用這種做法。假設現在我們現在要找出通過點 \\(u\\) 符合條件的路徑，就將 \\(u\\) 定為根，然後對其他連通的節點收集兩個資訊，一個是 \\(u \\) 到這個點的邊權和，二是到 \\(u \\) 的路徑長(邊的數量)。

接下來我們將這些 pair 進行配對，找到符合條件的，這其實就是很經典的資料結構問題(二維偏序)，可以先將一個維度排序後再搭配雙指針解決。

全部一起算後可能會多算到從 \\(u\\) 的同一個子節點的子樹(\\(LCA\\) 不是 \\(u\\) 的情況)，所以要再把來自同一個 \\(u\\) 的子節點子樹的點拿出來做一樣的計算並扣掉。

某種意義上這也算是三維偏序問題，複雜度 \\(O(n \log^2{N}) \\)

</details>

> [JOISC2020 首都](https://www.luogu.com.cn/problem/P7215)
>
> 給定一棵樹，每個點上有一個 \\(1 \\) ~ \\( k \\) 的編號，> 希望你能選則一個編號 \\( c \\)，使得編號為 \\( c \\) 的點都相連。
>
> 你能進行合併操作，每次操作可以將一個編號的點全部改為另一個編號，> 請問最少需要進行幾次操作才能找到滿足條件的編號?
>
> - \\(1 \le k \le n \le 2 \times 10 ^ 5\\)

<details><summary> Solution </summary>

先將題目簡化，選定一個根節點，我們只需要算最少要多少次合併才能將和根節點編號一樣的點全部連通。

可以用以下作法，把所有和根節點編號一樣的點先取出來放入一個 queue 中，然後每次取出一個點開始往上爬直到根節點或者遇到已經走過的點，途中經過的點編號就一定要選起來，並且把那個編號的點丟到 queue 中。

因為每個點只需要被經過一次，所以時間複雜度 \\(O(N)\\), \\(N\\) 是樹的大小。

接著套用重心剖分，每次以重心為根(必選此節點)計算答案，如果和重心節點的編號一樣的點沒有全部都在這個連通塊中，那就跳過這次計算。接著就不斷的拔重心，遞迴下去，最後取做過的所有計算的最小值就是答案。

為什麼這樣一定能計算到最優解呢？稍微說明一下：

在以重心 \\(c\\) 為根節點做計算時，假設重心上的編號為 \\(x\\)，然後考慮選一個編號 \\(y \neq x\\) 會不會比選 \\(x\\) 還好。目前只考慮 \\(x, y\\) 編號的節點都在這個連通塊中的情況。可以分成以下兩種情況：

1. 編號 \\(y\\) 的點，出現在兩個以上的 \\(c\\) 的子節點的子樹中
1. 編號 \\(y\\) 的點，只出現在一個 \\(c\\) 的子節點的子樹中

情況一，選了編號 \\(y\\) 必定也會選到編號 \\(x\\) 因為根節點在他們的必經之路上，不如直接選編號 \\(x\\)。

情況二，我們在往下遞迴的時候會考慮到。

所以以上作法能找到最小值，時間複雜度 \\(O(n \log{n})\\)。

</details>

> [ZJOI 捉迷藏](https://www.luogu.com.cn/problem/P2056)
>
> 給定一棵 \\(n\\) 個節點的樹，一開始所點都是黑色，要 \\(m\\) 次支援以下操作：
>
> 1. 改變一個節點的顏色(黑變白、白變黑)
>
> 2. 查詢最遠的兩個黑點的距離
>
> - \\(1 \le n \le 10 ^ 5, \ 1 \le m \le 5 \times 10 ^ 5\\)

<details><summary> Solution </summary>

首先，樹直徑可以用一個非常經典的樹 DP 求出，我們需要對於每個點計算從這個點出發往下走的 **最長** 和 **次長** 距離(兩條路徑不能有邊重疊)。我們可以根據這個做法下去延伸。

套用在重心樹上，我們也要對每個點 \\(u\\) 找出，到 \\(u\\) 在重心樹上子樹中黑色點的 **最長** 和 **次長** 距離。並且最長和次長不能來自 \\(u\\) 的同一個子節點子樹。所以我們一樣要紀錄貢獻是從哪個子節點傳上來的。因為需要支援新增、刪除和找最大值和次大值，可能會需要使用一些資料結構輔助。

修改的時候，就暴力的爬重心樹。查詢的部分，我們要隨時紀錄每個點的 **最長** 和 **次長** 相加，並將這個值放入一個集合 \\(S\\) 中，所以查詢的結果就是 \\(S\\) 裡面最大的元素。

這題稍微有一點卡常，如果用太多 `multiset` 來實作 \\(O((n + m) \log^2{n}) \\) 的做法，應該只能拿到 90 分。

這邊提供一個用兩個 `priority_queue` 來代替平衡樹的實作方式。

悄悄話：其實這一題有比較好寫的 \\(O(n \log{n})\\) 的線段樹作法。

<details><summary> Sample Code </summary>

當要在 pq 中刪除一個元素時，我們將要刪除的值 push 到另一個代表刪除的 pq 中。之後在要取用 pq 的最大值時，先看看是否和代表刪除的 pq 的最大值一樣，樣的話就一起 pop 掉，這樣就達到懶惰刪除的效果了。

```cpp
#include<bits/stdc++.h>
using namespace std;

struct CentroidTree {
  vector<vector<pair<int, int>>> tree;

  vector<vector<int>> centroid_tree, dis;
  vector<int> del, pa, sz;

  // for this problem
  int blk_cnt = 0;
  vector<priority_queue<int>> pa_con, pa_del, mx, mx_del;
  multiset<int> diameter;

  CentroidTree(int n)
      : tree(n + 1, vector<pair<int, int>>()),
        centroid_tree(n + 1, vector<int>()), dis(n + 1, vector<int>()),
        del(n + 1), pa(n + 1), sz(n + 1, 1) {
    mx.resize(n + 1);
    mx_del.resize(n + 1);
    pa_con.resize(n + 1);
    pa_del.resize(n + 1);
  }

  void add_edge(int u, int v, int w = 1) {
    tree[u].emplace_back(v, w);
    tree[v].emplace_back(u, w);
  }

  void get_sz(int u, int p) {
    sz[u] = 1;
    for (auto [v, w] : tree[u]) {
      if (v == p || del[v])
        continue;
      get_sz(v, u);
      sz[u] += sz[v];
    }
  }

  int get_centroid(int u, int n, int p) {
    for (auto [v, w] : tree[u]) {
      if (v != p && !del[v] && sz[v] > n / 2)
        return get_centroid(v, n, u);
    }
    return u;
  }

  void get_dis(int u, int p, int len) {
    dis[u].emplace_back(len);
    for (auto [v, w] : tree[u]) {
      if (v == p || del[v])
        continue;
      get_dis(v, u, len + w);
    }
  }

  // build the centroid tree recursively
  int build(int u = 1) {
    get_sz(u, -1);
    int centroid = get_centroid(u, sz[u], -1);
    del[centroid] = 1; // delete centroid
    get_dis(centroid, -1, 0);

    for (auto [v, w] : tree[centroid]) {
      if (del[v])
        continue;
      int tcd = build(v);
      pa[tcd] = centroid;
      centroid_tree[centroid].emplace_back(tcd);
    }
    return centroid;
  }

  void lazy_delete(priority_queue<int> &pq, priority_queue<int> &del) {
    while (!pq.empty() && !del.empty() && pq.top() == del.top()) {
      pq.pop();
      del.pop();
    }
  }

  int get_diameter(int x) {
    lazy_delete(mx[x], mx_del[x]);
    if (mx[x].size() == 0) return -1;
    
    int res = 0, tmp = 0;
    tmp = mx[x].top(); mx[x].pop();
    lazy_delete(mx[x], mx_del[x]);

    if (mx[x].size() == 0) { 
      mx[x].push(tmp);
      return -1;
    }

    res = tmp + mx[x].top();
    mx[x].push(tmp);
    return res;
  }

  void add_diameter(int x) {
    int diam = get_diameter(x);
    if (diam == -1)
      return;
    diameter.insert(diam);
  }

  void del_diameter(int x) {
    int diam = get_diameter(x);
    if (diam == -1)
      return;
    diameter.erase(diameter.find(diam));
  }


  void add(int x, int col) {
    del_diameter(x);
    if (col) {
      blk_cnt++;
      mx[x].push(0);
    } else {
      blk_cnt--;
      mx_del[x].push(0);
    }
    add_diameter(x);
    
    int u = pa[x], lst = x;
    for (int i = dis[x].size() - 2; i >= 0; i--) {
      int dist = dis[x][i];
      del_diameter(u);
      if (col) {
        lazy_delete(pa_con[lst], pa_del[lst]);
        if (!pa_con[lst].empty())
          mx_del[u].push(pa_con[lst].top());
        pa_con[lst].push(dist);
        mx[u].push(pa_con[lst].top());
      } else {
        mx_del[u].push(pa_con[lst].top());
        pa_del[lst].push(dist);
        lazy_delete(pa_con[lst], pa_del[lst]);
        if (!pa_con[lst].empty())
          mx[u].push(pa_con[lst].top());
      }
      add_diameter(u);
  
      lst = u;
      u = pa[u];
    }
  }
};

int main() {  
  ios_base::sync_with_stdio(0);
  cin.tie(0);
  int n, m;
  cin >> n;
  vector<int> col(n+1);
  CentroidTree CD(n);
  for (int i = 0; i < n - 1; i++) {
    int u, v;
    cin >> u >> v;
    CD.add_edge(u, v);
  }

  CD.build();

  for (int i = 1; i <= n; i++) {
    col[i] = 1;
    CD.add(i, col[i]);
  }
  
  cin >> m;
  while (m--) {
    char c;
    int x;
    cin >> c;
    if (c == 'C') {
      cin >> x;
      col[x] ^= 1;
      CD.add(x, col[x]);
    } else {
      if (CD.blk_cnt == 0) cout << -1 << '\n';
      else if (CD.blk_cnt == 1) cout << 0 << '\n';
      else cout << *CD.diameter.rbegin() << '\n';
    }
  }
}
```

</details>
</details>

> [CEOI Dynamic Diameter](https://codeforces.com/contest/1192/problem/B)
>
> 給定一棵 \\(n\\) 個點有邊權的樹，要支援 \\(q\\) 操作：
>
> 每次操作要對一條邊的邊權進行修改，修改完後輸出目前的樹直徑。
>
> - \\(2 \le n, q \le 10 ^ 5\\)
> - \\(1 \le w \le 2 \times 10 ^ {13}\\)
>

<details><summary> Solution </summary>

建議寫過上一題 **捉迷藏** 且熟悉樹壓平的人再來用重心剖分寫這一題。

應該算是經典題，而且解法非常多，每一種都很巧妙，這邊只講重心剖分的做法，想要看其他做法的話請參考[這篇](https://www.cnblogs.com/TinyWong/p/11260601.html)。

首先，我們先想想看，要如何在會更改邊權的情況下，還能知道一個點到根的距離(假設 \\(1\\) 為根節點)。

這是一個經典的樹壓平題目，更改一條邊只會對一個子樹造成影響，壓平之後就是一個區間，所以配合能區間修改的資料結構，像是懶標線段樹就能完成。

接著，回到原題，要維護樹直徑，套用上一題的想法，就是對每一個點 \\(u\\) 維護 \\(u\\) 到重心樹上的子孫節點的距離 **最長** 和 **次長**。但這樣好像沒辦法應付修改，因為每次修改會影響到不止一個點。

試試看一個大膽的想法，結合一開始的線段樹作法和重心剖分，在重心剖分的過程中，以重心為根開始 DFS，把目前和重心連通的那個樹做樹壓平，並存入線段樹中。因為每一個節點只會被存入他在重心樹上祖先節點的線段樹中，所以線段樹的長度總和是 \\(O(n \log{n})\\) (重心樹深度是 \\(log{n})\\)

注意：上述的樹壓平是在原樹上進行的，在重心樹上做樹壓平沒有意義

這樣我們就可以知道每個點到他任意一個重心樹上子節點的距離，自然也能維護最長和次長。那麼至於修改，每一條最多只會影響到 \\( \log{n} \\) 棵線段樹中的區間，所以就找出所有被這條邊影響的線段樹進行區間修改，並更新樹直徑，這樣就在 \\(O(n \log^2{n})\\) 的時間內完成此題。

</details>

## Summary

重心剖分的應用大概可以分成以下三類：

- 路徑問題 (e.g. Fixed-Length Path)
- 點集問題 (e.g. Xenia and Tree)
- 其他 (e.g. Ciel the Commander)

路徑問題，和分治一樣，我們只需要想好如何計算通過重心的路徑，剩下的就交給遞迴。

點集問題，題目會要你維護一個點集合，通常需要在重心樹上往上爬做修改和查詢，要清楚知道需要記錄什麼資訊。

以上兩種類型，都需要小心處理不能多算兩個來自同一個子樹的點的路徑。

其他問題，重心剖分的應用還有非常多，需要運用到重心、重心樹的各種性質。如果題目在重心上會出現特別的性質，或者是題目沒有給定根，但有了根會比較好做，我們就可以將重心節點當根來試試看，然後再套用重心剖分。

最後，對很多題目來說，重心剖分都不是唯一的解決辦法，可以先多想想看其他樹上演算法(e.g. 樹壓平、輕重鏈剖分)能不能使用，再挑選最好實作的來寫。

## References

- [USACO Guide - Centroid Decomposition](https://usaco.guide/plat/centroid?lang=cpp)
- [A Visual Introduction to Centroid Decomposition](https://medium.com/carpanese/an-illustrated-introduction-to-centroid-decomposition-8c1989d53308)
- [Hybrid Tutorial: Centroid Decomposition](https://codeforces.com/blog/entry/81661)
- [OI wiki 點分治](https://oi-wiki.org/graph/tree-divide/)
- [sshwy's note 點分治總結](https://notes.sshwy.name/Data-Structure/Centroid-Decomposition/)
- [蛋餅的競程隨筆](https://omeletwithoutegg.github.io/2019/12/04/TIOJ-1171/)
- [liouzhou_101's blog](https://www.cnblogs.com/TinyWong/p/11260601.html)
