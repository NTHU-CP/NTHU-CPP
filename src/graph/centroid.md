# Centroid

## introduction

樹的重心(centroid)，和樹的中心(center)有所不同，請注意分辨。

### 什麼是重心？

定義一個函數 \\( f(x) \\) 表示拔除節點 \\( x \\) 後所產生的子樹中，最大的那棵的大小。而一棵樹的所有節點當中，\\( f \\) 函數的最小值就發生在這棵樹的重心。

記住這個 \\( f(x) \\) 下面我們還會繼續使用。

### 性質

1. 一個節點 \\(x \\) 是重心若且唯若 \\( f(x) \le \lfloor\frac{n}{2}\rfloor \\) (\\(n\\) 是樹的大小)。
1. 每棵樹有最多 2 個，最少 1 個重心。
1. 定義函數 \\( dis(x) \\) 表示樹上所有節點到 \\(x\\) 的**距離和**，\\(dis\\) 函數最小值必定發生在重心。

<details><summary> 性質1. 證明 </summary>

為了方便證明，我們用 \\( size_u(v) \\) 來表示以 \\(u\\) 為根節點時，\\(v\\) 的子樹大小。

首先，可以用反證法來證明重心必定滿足 \\( f(x) \le \frac{n}{2} \\)。假設 \\(u\\) 是樹的重心，且有一節點 \\(v\\) 與其相連且 \\(size_u(v) > \frac{n}{2} \\)。那麼 \\(v\\) 當重心肯定會更好，因為 \\(size_v(u) = n - size_u(v) < \frac{n}{2} < size_u(v) = f(u)\\)，也就是說所有拔掉 \\(v\\) 會產生的子樹大小皆小於 \\(f(u)\\)，和重心定義矛盾。

接下來證明一個節點 \\(x\\) 滿足 \\( f(x) \le \frac{n}{2} \\)，必定為重心。假設 \\(u\\) 為重心，則所有與他相鄰的節點 \\(v\\) 都滿足 \\(size_u(v) \le \frac{n}{2} \\)，因為 \\(f(x) < \frac{n}{2}\\)。所以 \\( size_v(u) = n - size_u(v) \le \frac{n}{2} \le f(u)\\)，則 \\(f(v) \ge f(u)\\)。另外對於所有與 \\(u\\) 不相連的節點 \\(x\\)，都必定屬於某個和 \\(u\\) 相連的 \\(v\\) 子樹中(以 \\(u\\) 為根節點時)，所以 \\( size_x(v) > size_v(u) \ge \frac{n}{2} \ge f(u)\\)。

</details>

<details><summary> 性質2. 證明 </summary>

不妨先畫圖觀察一下，如果一棵樹有兩個重心，則他們必定相鄰。而此時樹一定有偶數個節點，將這棵樹從這兩個重心之間斷開，可以拿到兩棵一樣大小的樹。

首先來證明重心必定相鄰，不管有一個或兩個。假設 \\(u, v\\) 為樹的兩個重心(\\(u, v\\) 可以相等)，根據重心的定義 \\(f(u) \le f(v)\\) 且 \\(f(v) \le f(u)\\)，也就是 \\(f(u) = f(v)\\)。

如果 \\(u, v\\) 的路徑上還有其他 \\(k\\) 個節點，則 \\(u\\) 的最大子樹必定會包含這 \\(k\\) 個點，那麼 \\(f(u) \ size_u(v) = k + f(v)\\)，由此可知 \\(k == 0\\) 也就是兩重心相鄰。

而且不可能有三個以上的重心，因為重心必須兩兩相連，所以三個以上的重心會形成環。

</details>

<details><summary> 性質3. 證明 </summary>

對於樹上的任意一個節點 \\(u\\) 如果存在一個相鄰的節點 \\(v\\) 使得 \\(size_u(v) > \frac{N}{2}\\)，則 \\(dis(v) < dis(u)\\)，因為從 \\(u\\) 往 \\(v\\) 移動會使 \\(dis\\) 減少 \\(size_u(v) - (n-size_u(v)) = 2 \times size_u(v) - n > 0\\)。

所以樹上除了重心以外的節點都可以透過這種方式找到 \\(dis\\) 更小的點。

</details>

### 如何求重心?

通常我們要先求出子樹大小才能找重心，所以我們可以任選一個點當根，先求出每個節點子樹大小。接著開始從根開始遞迴檢查，遍歷所有子節點，如果有一個子節點子樹大小 \\( > \frac{n}{2} \\), 則重心一定在這個個子樹裡，所以就往這個子節點遞迴。反之如果所有子節點子樹大小都不超過一半，則現在所在的點即為重心。

另一種實作方式是一樣是任選一個點當根，先算出所有子樹大小，再透過整棵樹大小減去每個節點的子樹大小的方式，算出每個點父親節點所形成的子樹大小來判斷。

> [CSES Finding a Centroid](https://cses.fi/problemset/task/2079)
>
> 給一棵 \\(n\\) 個節點的樹，輸出樹的重心編號
>
> - \\( 1 \le n \le 2 \times 10 ^ 5 \\)

<details><summary> Sample Code </summary>

```cpp
#include <bits/stdc++.h>
using namespace std;

const int mxN = 2e5 + 5;
int sz[mxN];
vector<int> g[mxN];

void get_sz(int u, int p) {
  sz[u] = 1;
  for (int v : g[u]) {
    if (v == p)
      continue;
    get_sz(v, u);
    sz[u] += sz[v];
  }
}

int get_centroid(int u, int n, int p) {
  for (int v : g[u]) {
    if (v != p && sz[v] > n / 2)
      return get_centroid(v, n, u);
  }
  return u;
}

int main() {
  int n;
  cin >> n;
  for (int i = 0; i < n - 1; i++) {
    int u, v;
    cin >> u >> v;
    g[u].emplace_back(v);
    g[v].emplace_back(u);
  }
  get_sz(1, -1);
  cout << get_centroid(1, n, -1);
}
```

</details>

### 重心應用 (Additional)

這邊分享一個還蠻有趣的題目

> [BOI Village (Maximum)](https://codeforces.com/contest/1387/problem/B2)
>
> 有一座 \\(n\\) 個點的樹形成鎮，每一個點住著一位居民，編號從 \\( 1 \sim n \\)。
>
> 有一天，居民們突然想要搬家，每個居民都要搬到不同的節點，但一個節點只能住一位居民，一個居民的搬家距離為舊家到新家的路徑上的邊數。
>
> 請你給出一種搬家方案使所有居民的搬家距離總和最大。
>
> - \\(1 \le n \le 10^5 \\)

對於每一條邊 \\( (u, v) \\)，將這條邊斷開後會把樹分成兩棵大小分別為 \\( SZ_u, SZ_v \\)的樹，所以這條邊至多對答案貢獻 \\( 2 \times \min(SZ_u, SZ_v) \\) (乘以 \\(2\\) 是因為兩個方向都最多被經過 \\( \min(SZ_u, SZ_v) \\) 次)。所以考慮每條邊的貢獻，答案最大會是
\\(\begin{equation*} \sum\limits_{(u, v) \in E} \min \\{SZ_u, SZ_v\\} \end{equation*}\\)。

而其實這個上界，就是這一題的答案。我們先考慮用任意點當成根節點，則我們只要能讓根節點的子樹裡的節點，在交換後，都不在原本子樹中，這樣一來答案就會達到最大上界(證明可以自行嘗試)。但是大部分情況下，應該都找不到一組合法的交換方法，因為只要有任何一個子樹的大小大於節點數量的一半，很明顯的，就一定找不出一組解。

所以我們考慮以重心當根，則根節點的所有子樹大小都會 \\( < \frac{n}{2} \\)，有很多種方式可以構造出解答，這邊講一個實作起來最簡單的。

我們將以重心為根的樹先進行一次 DFS，並將依照每個點的拜訪順序紀錄成一個陣列 \\( a \\)，將 \\(a\\) 向右循環平移 \\( \lfloor\frac{n}{2}\rfloor \\)得到新的陣列 \\(b\\)，接著 \\( \forall{i \in 1 \sim n} \\)，將 \\(a_i\\) 搬到 \\(b_i\\)，這樣就得到一組解了。

## Exercises

> [CF 1406C Link Cut Centroids](https://codeforces.com/contest/1406/problem/C)
>
> 給你一棵 \\(n\\) 節點的樹，你能刪除樹上的一條邊並且再加入一條邊，使得圖還是一棵樹。
>
> 請問你有沒有辦法讓操作完的樹重心唯一? 請輸出一組可行的操作。
>
> - \\(1 \le n \le 10^5\\)

<details><summary> Hint </summary>

回想上面的性質 2，兩個重心會發生在什麼情況？

</details>

> [CF 685B Kay and Snowflake](https://codeforces.com/problemset/problem/685/B)
>
> 給你一棵 \\(n\\) 節點的有根樹，求每個子樹的重心。
>
> - \\(1 \le n \le 3 \times 10^5\\)

<details><summary> Hint </summary>

在上面找重心的程式碼中，我們每次只會往子樹大小 \\( > \frac{n}{2} \\) 的節點走。

那我們試著將這個做法反過來，從葉子節點開始，假設現在所在節點為 \\(u\\)，\\(u\\) 的父節點為 \\(p\\)。如果 \\( 2 \times size(u) \ge size(p)\\) 的話就繼續往上爬，並把經過的點的重心都設為向上爬的起始點。

</details>

## References

- [OI wiki 樹的重心](https://oi-wiki.org/graph/tree-centroid/)
- [算法學習筆記：樹的重心 By Pecco](https://zhuanlan.zhihu.com/p/357938161)
