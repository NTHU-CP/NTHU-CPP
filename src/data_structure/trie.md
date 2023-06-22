# Trie

Trie (字典樹) 是一種無序的樹狀資料結構，用來儲存字串的集合，支援查詢一個字串是否存在於集合中。

最基本 Trie 支援以下兩三種操作：

1. 查詢一個字串是否存在於集合中
2. 加入一個字串進入集合
3. 從集合中刪除一個字串

使用 Trie 可以在線性時間（字串長度）完成這三種操作。

## Basic Trie

Trie 是一個樹狀結構，每一個節點代表的是一個 prefix，其中根節點代表的前綴是空字串。
而每個邊代表的是一個字元，一個代表前綴 `pre` 的節點，會透過代表 `c` 字元的邊，連向代表前綴 `pre + c` 的節點。


有了這個樹狀的 Trie，我們可以在節點上儲存一些資訊，在查詢是否存在的這個例子，我們可以在每個節點上
存一個 `int` 代表目前的字串集合，有幾個字串等於這個節點代表的字串。

圖 1

這個樹是加入了 `bat`, `cat`, `cap` 三個字串得到的 Trie。

### 查詢

查詢一個字串是否出現在集合中，我們可以從跟節點（空字串）開始，照著要查詢的字串，
由左至右，對於每個字元，在當前的節點走向代表那個字元的節點。如果節點沒有連向那個字元的邊
，這代表字典樹沒有這個前綴。如果順利的走完所有的字元，停在一個節點上。我們便可以
根據節點上儲存的數字，判斷集合中是有存在這個字元，或是這個字元只是某個元素的前綴。

圖 2, 3, 4 分別表三種可能的查詢狀況。

### 加入 / 刪除

若要加入與刪除字元，需要先依照查詢的方法，找到代表目標字串的節點，就可以修改節點上的數字。
加入不同於查詢的地方在於，如果在其中一個步驟找不到邊時，應該要創造節點並且將邊連上。

圖 5 為在 Trie 中加入 `com` 的例子。

## 複雜度分析

查詢、加入、刪除一個長度為 L 的字串都需要 $O(L)$ 的時間複雜度，以及 $O(L)$ 的空間複雜度。讀者可能會拿 trie 跟 hash set 比較，的確如果考慮將字串 hash 之後以 hash set 儲存，可以達到與 trie 相同的複雜度，但時務上，trie 可以提供的確定性，以及維護共同前綴的特性，時常能夠幫助解決更多的問題。下一章節 XOR Trie 就會是很好的使用 trie 的例子。

## 實作

```cpp
#include <bits/stdc++.h>

template<typename T, std::size_t _Nm> 
class TrieNode {
private:
  template<typename IT>
  TrieNode* get(IT begin, IT end, bool create=false) {
    if (begin == end) {
      return this;
    }

    auto &next_node = _next_ptr[*begin];
    if (next_node == nullptr && create) {
      next_node = std::make_unique<TrieNode>();
    }

    return next_node == nullptr ? nullptr : next_node->get(std::next(begin), end, create);
  }
public:
  std::array<std::unique_ptr<TrieNode>, _Nm> _next_ptr;
  T _data;

  TrieNode() : _next_ptr(), _data() {}

  template<typename C>
  TrieNode* get(const C &c, bool create=false) {
    return get(std::begin(c), std::end(c), create);
  }
};
```

## XOR Trie

XOR Trie 是一種 Trie 的應用，通常使用時機是求在一個整數集合中，XOR 值最大或最小的兩個元素。讓我們用一個實際的例題來認識 XOR Trie:

### 例題: BZOJ1954 樹上最大 xor sum 路徑

一棵帶有正整數邊權的無根樹，求 (u, v) 使得節點 u 到 v 的路徑之間邊權 xor 總和最大，輸出這個邊權 xor 總和。 ($1 \leq N \leq 10^5, 0 \leq w \le 2^31$)

這題需要一點點的拆開題目包裝，我們定義 T(u, v) 為在樹上 u 到 v 之間的路徑 xor sum。我們可以發現指定任意一個節點作為 root 我們有 T(u, v) = T(u, root) xor T(v, root)。有這樣的觀察，我們可以用一次 dfs 對於所有節點 x 計算出 T(x, root)。接著問題就轉換成：如何在一個整數集合中，找出 XOR 值最大的兩個元素。

解決這個最大 xor pair 問題，我們可以使用 XOR Trie。我們把每個數字以 31 bits 二進位表示，並且保留 leading zeros。我們把這些數字當作字元集只有 0 與 1 長度皆為 31 的字串插入一棵字典樹。這個字典樹代表著我們的整數集合，我們可以在 $O(\log w)$ 的時間內查詢一個數字是否在集合中。

然而使用一般的查詢是無法解決這個最大值問題，我們需要使用類似線段樹上二分搜的技巧處理。要最大化 T(u, v)，首先我們可以枚舉 u，對於每個 T(u, root) 我們要找到一個 v 使得 T(u, root) xor T(v, root) 最大。對筆者來說，這個技巧與其說是二分搜，我更喜歡用 greedy 的想法去理解。我們要在 trie 中代表的集合，找出一個元素 T(v, root) 最大化 T(u, root) xor T(v, root)。我們可以從 root 開始，從最高位開始 greedy，優先選擇與 T(u, root) 最高位相反的 edge，如果相反的 edge 是存在的，選擇它一定更優。如果相反的 edge 不存在，只能選擇往相同的 edge 走去。接著對於第 2~31 bit 也使用同樣的策略，看與 T(u, root) 該 bit 相反的 edge 是否存在，盡量走相反的 edge。最後走完 31 條邊，得出來的 node 代表的就會是我們想要最大化 T(u, root) xor T(v, root) 的 v。

### 參考程式碼

```cpp
#include <bits/stdc++.h>

template<typename T, std::size_t _Nm> 
class TrieNode {
private:
  template<typename IT>
  TrieNode* get(IT begin, IT end, bool create=false) {
    if (begin == end) {
      return this;
    }

    auto &next_node = _next_ptr[*begin - '0'];
    if (next_node == nullptr && create) {
      next_node = std::make_unique<TrieNode>();
    }

    return next_node == nullptr ? nullptr : next_node->get(std::next(begin), end, create);
  }
public:
  std::array<std::unique_ptr<TrieNode>, _Nm> _next_ptr;
  T _data;

  TrieNode() : _next_ptr(), _data() {}

  template<typename C>
  TrieNode* get(const C &c, bool create=false) {
    return get(std::begin(c), std::end(c), create);
  }
};

using Edges = std::vector<std::vector<std::pair<int, int>>>;

void DFSXorPathSum(int node, const Edges &edges, std::vector<int> &xor_sum) {
  for (auto [child, weight]: edges[node]) {
    if (xor_sum[child] == -1) {
      xor_sum[child] = xor_sum[node] ^ weight;
      DFSXorPathSum(child, edges, xor_sum);
    }
  }
}

int main() {
  std::ios_base::sync_with_stdio(0);
  std::cin.tie(0);
  int n;
  std::cin >> n;
  Edges edges(n);
  for (int i = 0; i < n - 1; ++i) {
    int u, v, w;
    std::cin >> u >> v >> w;
    u--, v--;
    edges[u].emplace_back(v, w);
    edges[v].emplace_back(u, w);
  }

  std::vector<int> xor_sum(n, -1);
  xor_sum[0] = 0;
  DFSXorPathSum(0, edges, xor_sum);

  TrieNode<int, 2> root;
  for (int i = 0; i < n; ++i) {
    const std::string value = std::bitset<31>(xor_sum[i]).to_string();
    root.get(value, true);
  }

  int t_u_v_max = 0;
  for (int i = 0; i < n; ++i) {
    const std::string t_u_root = std::bitset<31>(xor_sum[i]).to_string();
    auto *itr = &root;
    int t_v_root = 0;
    for (int b = 30; b >= 0; --b) {
      int flip = '1' - t_u_root[30 - b];
      int edge = itr->_next_ptr[flip] ? flip : (1 - flip);
      t_v_root += edge << b;
      itr = itr->_next_ptr[edge].get();
    }
    t_u_v_max = std::max(t_u_v_max, t_v_root ^ xor_sum[i]);
  }

  std::cout << t_u_v_max << std::endl;
}
```

## 其他例題

- 最大 xor sum 子陣列
- 給定正整數 x 和一個整數集合 A，最大化 $x ^ a, a \in A$
- 給定正整數 x 和一個整數陣列 A，最大化 $x ^ a, a \in A[L:R]$

## 結語

本文章從最基本的操作開始，介紹了 trie 的用途與實作。接著將 Trie 延伸到 xor sum 的應用上，維護一個集合的精神不變，只是從正常的字串延伸到二進位的 01 字串。因為表示的是數字，因此可以使用 greedy 的方式找集合中的最大值。筆者認為 XOR Trie 可以類比到動態開點線段樹，因此有興趣的讀者可以挑戰持久化 XOR Trie，套用持久化資料結構的技巧，可以延伸解決一些區間問題。

