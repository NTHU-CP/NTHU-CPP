# Centroid Decompostion

## Introduction

此技巧也被稱作點分治、重心分治、重心剖分

重心剖分是一個不難的概念，但可以解決一些看起來很可怕的題目，先備知識只需要 Tree, DFS 就能弄懂重心剖分。  

## Centroid 重心

### 什麼是重心？

定義一個函數 \\( f(x) \\) 表示拔除節點 \\( x \\) 後所產生的子樹中，最大的那棵的大小。
而一棵樹的所有節點當中，\\( f \\) 函數的最小值就發生在這棵樹的重心。

### 性質

1. 
1. 每棵樹最多 2 個，最少 1 個重心


<details><summary> 證明 </summary>



</details>

### 如何求重心?

通常我們要先求出子樹大小才能找重心，所以我們可以任選一個點當根，先求出每個節點子樹大小。
接著開始從根開始遞迴檢查，遍歷所有子節點，如果有一個子節點子樹大小 \\( \frac{N}{2} \\), 則重心一定在這個個子樹裡，所以就往這個子節點遞迴。反之如果所有子節點子樹大小都不超過一半，則現在所在的點即為重心。

> [CSES Finding a Centroid](https://cses.fi/problemset/task/2079)
>
> 尋找重心練習題

<details><summary> Sample Code </summary>

```cpp
void get_sz(int u, int p) {
  sz[u] = 1;
  for (int v : g[u]) {
    if (v == p or del[v])
      continue;
    get_sz(v, u);
    sz[u] += sz[v];
  }
}

int get_centroid(int u, int n, int p) {
  for (int v : g[u]) {
    if (v != p and sz[v] > n / 2)
      return get_centroid(v, n, u);
  }
  return u;
}
```

</details>

### 重心應用 (Additional)

這邊分享一個還蠻有趣的題目

> [BOI Village (Maximum)](https://codeforces.com/contest/1387/problem/B2)
>
> 有一座 N 個點的樹形成鎮，每一個點住著一位居民，編號從 1 ~ N。
> 有一天，居民們突然想要搬家，每個居民都要搬到不同的節點，但一個節點只能有一位居民，一個居民的搬家距離為舊家到新家的樹上距離。
> 請你求出所有居民的最大搬家距離總和，且輸出一組解。

<details><summary> Solution </summary>

對於每一條邊 \\( e \\)，假設邊權為 \\( e_w \\)，我們將這條邊斷開後會把樹分成兩棵子樹，大小分別為 \\( SZ_x, SZ_y \\)。
所以答案的上界會是
\\(\begin{equation*} \sum_{e} 2 w_e \min \\{SZ_x, SZ_y\\} \end{equation*}\\)
，因為每一條邊最多被經過 \\( \min \\{SZ_x, SZ_y\\} \\) 次。

而其實這個上界，就是這一題的答案。我們先考慮用任意點當成根節點，則我們只要能讓根節點的子樹裡的節點，在交換後，都不在原本子樹中，這樣一來答案就會達到最大上界(證明可以自行嘗試)。
但是大部分情況下，應該都找不到一組合法的交換方法，因為只要有任何一個子樹的大小大於節點數量的一半，很明顯的，就一定找不出一組解。

所以我們考慮以重心當根，則根節點的所有子樹大小都會 \\( < \frac{n}{2} \\)，有很多種方式可以構造出解答，這邊講一個實作起來最簡單的。

我們將以重心為根的樹先進行一次DFS，然後把節點按照DFS序向右循環平移 \\( \frac{n}{2} \\)，這樣就得到一組解了。

</details>

## Centroid Decompostion

我們來看看重心究竟有什麼用途？

### 什麼是重心樹？

重心樹，其實就只是把重心剖分不斷找重心的過程用一棵樹表示

我們可以用遞迴定義如下

- 重心樹的根節點為原樹的重心
- 將上一步的重心從原樹上刪除後，原樹被分為若干棵子樹。這些子樹的重心，就是重心樹根節點的子節點(在重心樹上)
- 對所有子樹遞迴建立重心樹

注意區分原樹和重心樹!!!

看圖理解一下

<p align="center">
    <img src="images/0.png">
</p>
<p style="text-align: center;"> 原樹 </p>

<p align="center">
    <img src="images/1.png">
</p>
<p style="text-align: center;"> 重心樹 </p>

最後原樹上所有節點都會被刪除，而且每一個原樹上的節點都會對應一個重心樹上的節點

可以參考下面的 gif 來體會一下每次拔掉重心的過程

<p align="center">
    <img src="images/tree.gif">
</p>
<p style="text-align: center;"> 節點刪完之時，重心樹歸來之日XD </p>

### 如何實作重心樹

接著來看看如何實作建出一棵重心樹。

```cpp
vector<int> g[mxN], tree[mxN]; // g存原本的樹 tree存重心樹
bitset<mxN> del;               // 紀錄已被刪除的點
int pa[mxN];                   // 紀錄重心樹上每個節點的父親節點

void get_sz(int u, int p) {
  sz[u] = 1;
  for (int v : g[u]) {
    if (v == p or del[v])
      continue;
    get_sz(v, u);
    sz[u] += sz[v];
  }
}

int get_centroid(int u, int n, int p) {
  for (int v : g[u]) {
    if (v != p and !del[v] and sz[v] > n / 2)
      return get_centroid(v, n, u);
  }
  return u;
}

int build(int u) {
  get_sz(u, -1);
  int centroid = get_centroid(u, sz[u], -1);
  del[centroid] = 1; // 刪除重心

  // 注意 u 不一定等於 centroid
  /*******
  根據題目進行某些計算
  ********/

  for (int v : g[centroid]) {
    if (del[v])
      continue;
    int t = build(v);
    pa[t] = centroid;
    tree[centroid].push_back(t);
  }
  return centroid;
}
```

注意不要走到已經刪除的節點上！

建立出重心樹其實很簡單，而且只需要 \\( O(N \log{N}) \\) 的時間複雜度(下面會證明)。

困難的地方在於重心樹性質的運用。以下是重心樹的一些重要性質以及分析。

### 重心樹性質

1. 重心樹的深度是 \\( O(\log{N}) \\)
1. 重心樹的任一子樹，在原樹中都是連通的(但並不代表在重心樹中相連的兩個點，在原樹中b也會相連)
1. 一條在原樹上的路徑 (a, b)，可以被拆成 a→lca(a,b)→b，lca(a,b) 是 a, b 在**重心樹**上的最低公共祖先
1. \\( \sum SZ_u = O(N \log N) \\), \\( SZ_u \\) 是節點 \\( u \\) 在重心樹上的子樹大小

#### 性質的說明以及證明

大家有興趣的話可以先自己花時間想想看

<details><summary> 性質1. 重心樹深度證明 </summary>

首先發現，重心樹的深度其實也就是我們在建樹過程中的遞迴深度，所以可以順便得出建立重心樹的時間複雜度。

重心樹的深度是 \\( O(\log{N}) \\)，因為在我們每次遞迴，拔掉重心所形成的子樹大小都不會超過原樹的一半，所以最多經過 \\( O(\log{N}) \\) 次，所有子樹大小都會 \\( \le 1 \\)。

因為每一層(重心樹)遞迴都要對所有子樹找重心，時間複雜度 \\( O(N) \\)。

所以總時間複雜度是 \\( O(N \log{N}) \\) 。
想要更完整的證明的話，可以用主定理或是其他證明遞迴算法複雜度的方式。

</details>

<details><summary> 性質2. 說明 </summary>
回想一下重心樹建立的過程，如果一個節點 a 是節點 b 的祖先(在重心樹上)，那 b 必定存在於因為刪除 a 而產生的某個子樹。

所以一個節點在重心樹中的所有子節點，會在原樹中相連。

</details>

<details><summary> 性質3. 說明 </summary>
我們可以用反證法，假設原樹上的路徑 (a, b) 不能被拆解成 a→lca(a,b)→b。
則代表在建立重心樹的過程中，拔除 lca(a, b) 時 a, b 仍然聯通。
而此時在 a, b 所在子樹的重心也會是 a, b 的共同祖先，而且會比 lca(a, b) 更低，這樣就形成矛盾了。

所以也就是說，a, b 會在 lca(a, b) 被拔掉時被拆散在不同子樹中。

</details>

<details><summary> 性質4. 說明 </summary>
仔細一想會發現，這其實就和證明重心剖分複雜度是一樣的。
按照深度將每層的節點取出來，並加他們的子樹大小相加，都是 \\( O(N) \\) 的，而最多只有 \\( O(\log{N}) \\) 層。
所以很有趣的，我們可以在重心樹上做一些有趣的事情，複雜度還會是好的。
像是對所有子樹都做一次 DP 等在普通樹上會很容易 TLE 的操作。

</details>

熟練運用以上性質就可以做出重心剖分大部分的題目了。

## Examples

> [Ciel the Commander](https://codeforces.com/contest/321/problem/C)
>
> 有一個 n 個節點的樹，你要為每個點填入 ‘A’ 到 ‘Z’ 其中一個字母並滿足以下條件：
>
> 任兩個有相同字母的節點 u, v，必須有一個節點 x 在 u 到 v 的路徑上，且 x 上的字母的字典序嚴格小於 u, v 上的字母，
> 請輸出一組滿足條件的解
>
> - \\( 1 \le n \le 10^5 \\)
>

雖然題目沒說，但可以猜測一定有方法能只用26個大寫英文子母填完。
回想重心樹的第三個性質，不難想到我們可以把重心樹的每一層都填上同一個英文字母。
再根據重心樹的第一性質，也就是高度不超過 \\( O(\log{N}) \\)，我們一定可以找到一組解。

> [Xenia and Tree](https://codeforces.com/problemset/problem/342/E)
>
> 有一棵 \\( n \\) 個節點的樹，一 開始所有節點都是藍色，要執行 \\( m \\) 次以下兩種操作：
>
> 1. 將一個藍色節點塗成紅色
>
> 2. 查詢一個節點 \\( x \\) ，回答離 \\( x \\) 最近的紅色節點距離
>
> - \\( 1 \le n,m \le 10^5 \\)
>

這一題是非常經典的重心剖分應用，但是剛學完重心剖分實在很難想到要如何用在這題上面。

我們可以先根據重心樹第三個性質做一點延伸，考慮從 \\( x \\) 點出發的所有路徑 \\( (x, v) \\)。
都可以被拆成 x → lca(x, v) → v，而 lca(x, v) 一定會是 x 的祖先(顯然)，再根據性質一，
x 的祖先只有 \\( O(\log{N}) \\) 種可能。
所以如果我們可以想辦法用 x 在重心樹上的祖先作為中繼站，每次對 x 的操作只需要遍歷 x 的祖先，就能在合理的時間內找到答案。

我們可以用陣列 \\(ans\\) 來紀錄答案，\\( ans[x] \\) 表示在重心樹中 \\(x \\) 子樹裡離 \\( x \\) 最近的**紅色**節點距離。
這時候兩個操作都能非常輕鬆的完成。

將一個節點 \\(x \\) 塗上紅色：
遍歷 \\(x \\) 在重心樹上的祖先\\(y \\)，更新 \\( ans[y] = \min(ans[y], dis(x, y)) \\)。

查詢一個節點 \\(x \\) 到最近紅色節點的距離：
遍歷 \\(x \\) 在重心樹上的祖先 \\(y \\)，紀錄 \\( ans[y]  + dis(x, y) \\) 的最小值即為答案。

上面的 \\( x \\) 在重心樹上的祖先皆包括 \\( x \\) 本身，而 \\( dis(x, y) \\) 代表 \\(x \\) 到 \\( y \\) 在原樹上的距離。
在樹上動態求出任意兩點距離，使用普通的倍增法每次查詢會須要 \\( O(\log{N}) \\) 的時間，所以建議使用歐拉序樹壓平將求LCA問題轉成RMQ問題，就可以使用Sparse Table做到 \\( O(1) \\) 查詢。

但上面這個做法實作量偏大，所以在下方實作補充的地方會講解一個可以在重心剖分過程中，先求出每個點到其重心樹上祖先的實作方式。
  
總而言之，我們可以透過在重心樹上向上更新，
在 \\( O( N \log {N} ) \\) 或 \\(O( N \log^2{N}) \\) 的時間內通過此題。

> [Fixed-Length Path I](https://cses.fi/problemset/task/2080)
>
> 給定一棵樹和一個整數 \\( k \\)，請問樹上長度為 \\( k \\) 的路徑有幾條？
>
> \\(1 \le k \le n \le 2 \times 10 ^ 5\\)

這個題目有很多種作法，甚至有 \\(O(N) \\) 的樹上啟發式合併做法，但這邊只會講解重心剖分的做法。

我們考慮一個樹 DP 的做法，從根節點開始，先計算出所有通過根節點的路徑且長度等於 \\( k \\) 的路徑，接下來把根節點拔掉，接著遞迴下去。
而計算通過根節點且長度為 \\( k \\) 的路徑數量所需要花費的時間為 \\( O(M) \\)， \\( M \\) 是子樹大小。總時間複雜度就會是樹上所有子樹的大小之和。考慮最差的情況，樹是一條鏈，此做法的時間複雜度為 \\( O(N ^ 2) \\)。

上面的做法雖然不能通過此題，但如果我們改變一下 DP 的順序，每次都拔掉樹的重心，其實也就是在重心樹上做 DP。
根據上面的性質4，複雜度就會變成 \\( O(N \log{N}) \\)。

在重心樹上 DP 的方式可能會和在原樹上 DP 稍有不同，可以參考以下部分程式碼。

```cpp
int cnt[200001], ans;
vector<int> data;

void dfs(int u, int p, int len) {
  data.eb(len);
  if (len <= k)
    ans += cnt[k - len];
  for (int v : g[u]) {
    if (v == p or del[v])
      continue;
    dfs(v, u, len + 1);
  }
}

int build(int u) {
  get_sz(u, -1);
  int centroid = get_centroid(u, sz[u], -1);
  del[centroid] = 1;

  int mx = 0;
  cnt[0] = 1;
  for (int v : g[centroid]) {
    if (del[v])
      continue;
    dfs(v, centroid, 1);
    for (int i : data) {
      cnt[i]++;
      mx = max(mx, i);
    }
    data.clear();
  }
  fill(cnt, cnt + mx + 1, 0);

  for (int v : g[centroid]) {
    if (del[v])
      continue;
    build(v);
  }
  return centroid;
}
```

> [Black-White-Tree](https://csacademy.com/contest/archive/task/black-white-tree/)
>
> 一樣給一棵樹，每個節點可能是白色或黑色，要支援以下操作：
>
> 1. 改變一個節點的顏色
>
> 2. 查詢一個點 \\( x \\) 到其他所有相同顏色的點的距離和
>

這一題和上一題非常像，而且這兩題其實都有很好寫的根號作法，可以參考[這裡](../sqrt/sqrt_decomposition.html#操作分塊)。

但是這題如果要使用重心剖分需要注意很多實作上的細節，所以這邊特別拿出來提一下。

在這題中，白色和黑色的資訊都需要紀錄，而且除了紀錄 \\(x \\) 在重心樹子樹裡黑色(或白色)的距離和還有數量。
還要在更新時紀錄 \\(x \\) 對重心樹上的父親節點的貢獻(code 裡的 sub 陣列)，
因為在計算答案時，假如你從 \\( x \\) 開始，\\( x \\) 在重心樹上的父親用 \\( p \\)表示。
會發現 \\( p \\) 裡紀錄的資訊其實會包括 \\( x \\) 子樹的資訊，所以會用重複計算的問題，需要特別小心。

<details><summary> Solution </summary>

- Code

    ```cpp
    #include <bits/stdc++.h>
    #define pb emplace_back
    using namespace std;

    const int mxN = 5e4 + 5;
    int n, m, col[mxN];

    int pa[mxN], sz[mxN], sub[mxN][2], cnt[mxN][2], sum[mxN][2];
    bitset<mxN> del;

    // LCA euler sequence
    int st[mxN * 2][22], pos[mxN], dep[mxN], tp;
    vector<int> g[mxN];

    inline void build_rmq() {
      for (int i = 0; (1 << i) < tp; i++) {
        for (int j = 0; j + (1 << i) - 1 < tp; j++) {
          st[j][i + 1] = min(st[j][i], st[j + (1 << i)][i]);
        }
      }
    }

    inline int query(int l, int r) {
      int k = __lg(r - l + 1);
      return min(st[l][k], st[r - (1 << k) + 1][k]);
    }

    inline int dis(int u, int v) {
      if (pos[u] > pos[v])
        swap(u, v);
      return dep[u] + dep[v] - 2 * query(pos[u], pos[v]);
    }

    void euler_dfs(int u, int p) {
      pos[u] = tp;
      st[tp++][0] = dep[u];
      for (int v : g[u]) {
        if (v == p)
          continue;
        dep[v] = dep[u] + 1;
        euler_dfs(v, u);
        st[tp++][0] = dep[u];
      }
    }

    // 正篇開始，上面都是 O(1) 求距離的程式碼
    void get_sz(int u, int p) {
      sz[u] = 1;
      for (int v : g[u]) {
        if (v == p or del[v])
          continue;
        get_sz(v, u);
        sz[u] += sz[v];
      }
    }

    int get_centroid(int u, int n, int p) {
      for (int v : g[u]) {
        if (v != p and !del[v] and sz[v] * 2 > n)
          return get_centroid(v, n, u);
      }
      return u;
    }

    void dfs(int u, int p, int dis, int tp) {
      sum[tp][0] += dis;
      cnt[tp][0]++;
      for (int v : g[u]) {
        if (del[v] or v == p)
          continue;
        dfs(v, u, dis + 1, tp);
      }
    }

    int build(int u) {
      get_sz(u, -1);
      int centroid = get_centroid(u, sz[u], -1);
      del[centroid] = 1;

      for (int v : g[centroid]) {
        if (del[v])
          continue;
        int p = sum[centroid][0];
        dfs(v, -1, 1, centroid);
        int t = build(v);
        sub[t][0] = sum[centroid][0] - p;
        // trace(sub[t][0], sum[centroid][0]);
        pa[t] = centroid;
      }
      return centroid;
    }

    inline void update(int x) {
      int v = x;
      col[x] ^= 1;
      cnt[x][col[x]]++;
      cnt[x][col[x] ^ 1]--;
      while (pa[v]) {
        int d = dis(pa[v], x), p = v;
        v = pa[v];
        sum[v][col[x]] += d;
        cnt[v][col[x]]++;
        sum[v][(col[x] ^ 1)] -= d;
        cnt[v][(col[x] ^ 1)]--;

        sub[p][col[x]] += d;
        sub[p][(col[x] ^ 1)] -= d;
      }
    }

    inline int query(int x) {
      int v = x, res = sum[x][col[x]];
      while (pa[v]) {
        int d = dis(pa[v], x);
        res += d * (cnt[pa[v]][col[x]] - cnt[v][col[x]]) + sum[pa[v]][col[x]] -
               sub[v][col[x]];
        v = pa[v];
      }
      return res;
    }

    signed main() {
      cin >> n >> m;
      vector<int> t(n + 1);
      for (int i = 1; i <= n; i++) {
        cin >> t[i];
      }
      for (int i = 1; i < n; i++) {
        int a, b;
        cin >> a >> b;
        g[a].pb(b);
        g[b].pb(a);
      }
      euler_dfs(1, -1);
      build_rmq();
      build(1);
      for (int i = 1; i <= n; i++)
        if (t[i])
          update(i);

      while (m--) {
        int o, x;
        cin >> o >> x;
        if (o == 1)
          update(x);
        else
          cout << query(x) << '\n';
      }
    }
    ```

</details>

### 實作補充

這邊要來講解一個不用歐拉序樹壓平就能做到 \\( O(1) \\) 查詢距離的方法，當然這是僅限於查詢一個點到他在重心樹上的祖先，
不是查詢樹上任意兩點。

根據性質四，我們其實可以暴力遍歷每一個點 \\(x \\)，並且更新 \\(x \\) 到其子樹裡所有點的距離

這邊提供一種實作方法

```cpp
void get_dis(int u, int p, int len) {
  dis[u].push_back(len);
  for (auto [v, w] : g[u]) {
    if (v == p or del[v])
      continue;
    get_dis(v, u, len + w);
  }
}

int build(int u) {
  get_sz(u, -1);
  int centroid = get_centroid(u, sz[u], -1);
  del[centroid] = 1; // 刪除重心

  // 記得加入這一行
  get_dis(centroid, -1, 0);

  for (int v : g[centroid]) {
    if (del[v])
      continue;
    int t = build(v);
    pa[t] = centroid;
    tree[centroid].push_back(t);
  }
  return centroid;
}

// 使用方法
// !!!!!! 使用之前要先全部 reverse
void reverse_first() {
  for (int i = 1; i <= n; i++)
    reverse(ALL(dis[u]));
}

void update(int x) {
  // x 到 x 在重心樹上的第 i 個祖先的距離是 dis[x][i]
  for (int u = x, j = 0; u != -1; j++, u = pa[u]) {
    ans = min(ans, ans[u] + dis[x][j]);
  }
}
```

## Exercises

> [Fixed-Length Path II](https://cses.fi/problemset/task/2081)
>
> 給定一棵樹和兩個整數 \\( k_1, k_2 \\)，請問樹上長度介在 \\([k_1, k_2]\\) 的路徑有幾條？
>
> \\(1 \le k_1 \le k_2 \le n \le 2 \times 10 ^ 5\\)

<details><summary> Hint </summary>

這題和上面的 Fixed-Length Path I 一樣，有 \\( O(N) \\) 的做法。

我們將 Fixed-Length Path 紀錄的 DP 資訊改成前綴和，並用資料結構(ex. BIT)來維護即可。

</details>

> [Close Vertices](https://codeforces.com/problemset/problem/293/E)
>
> 給定一棵有邊權的樹和兩個變數 \\( l, w \\)，
> 問你樹上有多少點對 \\( (u, v) \\) 滿足以下條件：
>
> 1. \\( (u, v) \\) 路徑上的邊數量 \\( \le l \\)
>
> 2. \\( (u, v) \\) 路徑上的邊權和 \\( \le w \\)
>

<details><summary> Hint </summary>

這題和上面的 Fixed-Length Path II 長得很像，但是又多了一個條件。

所以其實只要再將前一題的資料結構換成二維資料結構就能通過此題了，但是不建議用這種做法。
假設現在我們選定的根節點為 \\( u \\)，我們會對 \\( u \\) 的子樹(重心樹)，收集兩個資訊 \\((x, y)\\)， \\(x\\) 是到 \\(u \\) 的邊權和而 \\(y\\) 是到 \\(u \\) 的路徑長。
接下來我們要找到所有兩兩組合中滿足限制的，這其實就是很經典的資料結構問題，可以先將一個維度排序後再搭配雙指針解決。

</details>

> [JOISC2020 首都](https://www.luogu.com.cn/problem/P7215)
>
> 給定一棵樹，每個點上有一個 \\(1 \\) ~ \\( K \\) 的編號，
> 希望你能選則一個編號 \\( c \\)，使得編號為 \\( c \\) 的點都相連。
>
> 你能進行合併操作，每次操作可以將一個編號的點全部改為另一個編號，
> 請問最少需要進行幾次操作才能找到滿足條件的編號?
>

## Summary

重心剖分的題目通常會和樹的原型態較無關聯的路徑問題，而且大部分的題目可以轉換成重心樹去思考。
概念本身不難，但各種題目和應用沒寫過類似題真的都不好想到，想真的學好重心剖分，建議去References找更多題目來練習。

## References

- [USACO Guide](https://usaco.guide/plat/centroid?lang=cpp)
- [Illustrated Intro to Centroid Decomposition](https://medium.com/carpanese/an-illustrated-introduction-to-centroid-decomposition-8c1989d53308)
- [Centroid Decomposition CF blog](https://codeforces.com/blog/entry/81661)
- [OI wiki 點分治](https://oi-wiki.org/graph/tree-divide/)
