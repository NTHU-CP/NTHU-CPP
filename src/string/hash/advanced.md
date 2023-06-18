# Advanced applications of Rolling Hash

Rolling Hash 也許看似簡單，但是要靈活運用才能發揮他的完整威力，下面給大家一些習題，靈活使用工具才能解出來喔。

## Problems

總共有五題，不會說哪題要先寫出來，後面的題目才知道要怎麼寫。筆者有幫各位把題目由簡單排到難了。

### 字典序最小的 Cyclic Shift

[題目連結](https://acmp.ru/asp/do/index.asp?main=task&id_course=2&id_section=18&id_topic=43&id_problem=284&locale=en)

目標：在 \\(O(n\ \log n)\\) 的時間內找到長度為 \\(n\\) 的字串的字典序最小循環移位

> 字串 \\(s\\) 的循環移位 (Cyclic Shift) 是指將字串 \\(s\\) 按照某個 \\(k, 0 \leq k < n\\)進行移位得到的字串 \\(s_{k+1}s_{k+2} ... s_n s_1 s_2 ... s_k\\)，其中 \\(n\\) 是字串 \\(s\\) 的長度。
>
> 對於給定的字串，找出其字典序最小的循環移位，即在所有可能的循環移位中，找到按字母順序排列的第一個。

<details>
  <summary>解答</summary>
In this problem we need to use compare by great / less in \\(O(log(n))\\) time using binary search by length of equal subsequence. Duplicate string S and calculate polynomial hashes on prefixes. Each cyclic shift will be represented as a number (initial position). Add all the positions to the vector, and then apply a linear algorithm for finding the minimum in the array using the substring comparison operator.

Complexity Estimatation: \\(O(n\ log(n))\\) time and \\(O(n)\\) memory.

範例解法

```C++
#include <stdio.h>
#include <cassert>
#include <algorithm>
#include <vector>
#include <random>
#include <chrono>
#include <string>
 
typedef unsigned long long ull;
 
// Init static variables of PolyHash class:
int PolyHash::base((int)1e9+7);    
std::vector<int> PolyHash::pow1{1};
std::vector<ull> PolyHash::pow2{1};
 
int main() {
    // Input:
    char buf[1+100000];
    scanf("%100000s", buf);
    std::string a(buf);
    a += a; // duplicate
 
    // Len of string:
    const int n = (int)a.size() / 2;
 
    // Max needed power:
    const int mxPow = 2 * n;
 
    // Gen random base:
    PolyHash::base = gen_base(256, PolyHash::mod);
 
    // Create hashing object:
    PolyHash hash(a);
 
    // Put all start positions in vector:
    std::vector<int> pos;
    for (int i = 0; i < n; ++i) {
        pos.push_back(i);
    }
 
    // Linear search of min algorithm:
    auto p = *std::min_element(pos.begin(), pos.end(), [&](const int p1, const int p2) {
        // Binary search by equal subsequences length:
        int low = 0, high = n+1;
        while (high - low > 1) {
            int mid = (low + high) / 2;
            if (hash(p1, mid, mxPow) == hash(p2, mid, mxPow)) {
                low = mid;
            } else {
                high = mid;
            }
        }
        return low < n && a[p1+low] < a[p2+low];
    });
 
    // Output answer:
    printf("%s", a.substr(p, n).c_str());
    return 0;
}
```

</details>

### 排序所有 Cyclic Shift

[題目連結](https://acmp.ru/asp/do/index.asp?main=task&id_course=2&id_section=18&id_topic=43&id_problem=286&locale=en)

目標：在 \\(O(n\ log^2 n)\\) 的時間內，按字典序對長度為 \\(n\\) 的字串的所有循環移位進行排序

> 字串 \\(s\\) 的循環移位 (Cyclic Shift) 是指將字串 \\(s\\) 按照某個 \\(k, 0 \leq k < n\\)進行移位得到的字串 \\(s_{k+1}s_{k+2} ... s_n s_1 s_2 ... s_k\\)，其中 \\(n\\) 是字串 \\(s\\) 的長度。
>
> 對於給定的字串，請構造所有的循環移位，並將每個循環移位排序。接著，輸出每個排序後字串的最後一個字元，以及給定的字串位於排序後列表的第幾名。

是可以建 Suffix Array 去解決這個問題沒錯，但是這樣太複雜了。你能不能粗暴 (但是很簡單) 的用 Rolling Hash 解決它呢？

<details>
  <summary>解答</summary>

We'll duplicate the string \\(S\\) and count the polynomial hash on the prefix. Each cyclic shift will be represented as a number (initial position). Add all the positions to the vector, and then apply a stable merge sort using the substring comparison operator.

Complexity Estimatation: \\(O (n\ log(n)^2)\\) in time and \\(O(n)\\) from memory.

```C++
#include <stdio.h>
#include <cassert>
#include <algorithm>
#include <vector>
#include <random>
#include <chrono>
#include <string>
 
typedef unsigned long long ull;

// Init static variables of PolyHash class:
int PolyHash::base((int)1e9+7);    
std::vector<int> PolyHash::pow1{1};
std::vector<ull> PolyHash::pow2{1};
 
int main() {
    // Input:
    char buf[1+100000];
    scanf("%100000s", buf);
    std::string a(buf);
    a += a; // duplicate
 
    // Len of string:
    const int n = (int)a.size() / 2;
 
    // Max needed power:
    const int mxPow = 2 * n;
 
    // Gen random base:
    PolyHash::base = gen_base(256, PolyHash::mod);
 
    // Create hashing object:
    PolyHash hash(a);
 
    // Put all start positions in vector:
    std::vector<int> pos;
    for (int i = 0; i < n; ++i) {
        pos.push_back(i);
    }
 
    // Linear search of min algorithm:
    auto p = *std::min_element(pos.begin(), pos.end(), [&](const int p1, const int p2) {
        // Binary search by equal subsequences length:
        int low = 0, high = n+1;
        while (high - low > 1) {
            int mid = (low + high) / 2;
            if (hash(p1, mid, mxPow) == hash(p2, mid, mxPow)) {
                low = mid;
            } else {
                high = mid;
            }
        }
        return low < n && a[p1+low] < a[p2+low];
    });
 
    // Output answer:
    printf("%s", a.substr(p, n).c_str());
    return 0;
}
```

</details>

### 算有幾個 substring 是回文字串

[題目連結](https://acmp.ru/asp/do/index.asp?main=task&id_course=2&id_section=18&id_topic=43&id_problem=285&locale=en)
目標：在 \\(O(n\ \log n)\\) 的時間內，算出有幾個 substring 是回文字串，且字串長度為 \\(n\\)

先來一個 Key observation，如果 \\(T[i-j:i+j]\\) 是回文字串，那麼 \\(T[i-(j-1):i+(j-1)]\\) 也是回文字串。

<details>
  <summary>解答</summary>
In this problem we need to use compare by great / less in \\(O(log(n))\\) time using binary search by length of equal subsequence. Duplicate string S and calculate polynomial hashes on prefixes. Each cyclic shift will be represented as a number (initial position). Add all the positions to the vector, and then apply a linear algorithm for finding the minimum in the array using the substring comparison operator.

Complexity Estimatation: \\(O(n\ log(n))\\) time and \\(O(n)\\) memory.

範例解法

```C++
#include <stdio.h>
#include <cassert>
#include <algorithm>
#include <vector>
#include <random>
#include <chrono>
#include <string>
 
typedef unsigned long long ull;
 
// Init static variables of PolyHash class:
int PolyHash::base((int)1e9+7);    
std::vector<int> PolyHash::pow1{1};
std::vector<ull> PolyHash::pow2{1};
 
int main() {
    // Input:
    char buf[1+100000];
    scanf("%100000s", buf);
    std::string a(buf);
    a += a; // duplicate
 
    // Len of string:
    const int n = (int)a.size() / 2;
 
    // Max needed power:
    const int mxPow = 2 * n;
 
    // Gen random base:
    PolyHash::base = gen_base(256, PolyHash::mod);
 
    // Create hashing object:
    PolyHash hash(a);
 
    // Put all start positions in vector:
    std::vector<int> pos;
    for (int i = 0; i < n; ++i) {
        pos.push_back(i);
    }
 
    // Linear search of min algorithm:
    auto p = *std::min_element(pos.begin(), pos.end(), [&](const int p1, const int p2) {
        // Binary search by equal subsequences length:
        int low = 0, high = n+1;
        while (high - low > 1) {
            int mid = (low + high) / 2;
            if (hash(p1, mid, mxPow) == hash(p2, mid, mxPow)) {
                low = mid;
            } else {
                high = mid;
            }
        }
        return low < n && a[p1+low] < a[p2+low];
    });
 
    // Output answer:
    printf("%s", a.substr(p, n).c_str());
    return 0;
}
```

</details>

### 問有幾個後綴滿足特定條件

[題目連結](https://acmp.ru/asp/do/index.asp?main=task&id_course=2&id_section=18&id_topic=42&id_problem=264&locale=en)
目標：對於一個長度為 \\(n\\) 的字串，問有幾個後綴滿足「後綴的 Infinite Extension 跟原本字串的 Infinite Extension 相同」

Infinite Extension: 將原先字串頭尾相接後重複拼貼，也就是說 \\(s[i] = S[i \mod N]\\)，其中 \\(S\\) 是 \\(s\\) 的 Infinite Exntesion 且 \\(|S|=N\\)。

<details>
  <summary>解答</summary>
Key observations:

1. 如果 \\(S\\) 跟 \\(T\\) 的 Infinite Extension 相同，那麼「\\(S\\) 頭尾相接後重複拼貼 \\(|T|\\) 次」必然相等「\\(T\\) 頭尾相接後重複拼貼 \\(|S|\\) 次」。
2. 令\\(S\\) 為 \\(s\\) 頭尾相接後重複拼貼 \\(N\\) 次，\\(m = |S|\\)
$$ Hash(S) = Hash(s) \cdot (1 + p^m + p^{2m} + p^{3m} + ... + p^{(N-1)m}) $$
3. \\( (1 + p^m + p^{2m} + p^{3m} + ... + p^{(N-1)m}) \cdot (p - 1) = p^k - 1 \\)

枚舉每個後綴，並且利用模逆元去計算 \\( (1 + p^m + p^{2m} + p^{3m} + ... + p^{(N-1)m}) \\)，即可判斷該後綴的 Infinite Extension 是否與原字串的 Infinite Extension 相同。

在此之外，在無法取得模逆元的狀況下，作者給出了另外一種方法計算 \\( (1 + p^m + p^{2m} + p^{3m} + ... + p^{(N-1)m}) \\)，詳細可見範例解法。

範例解法

```C++
#include <stdio.h>
#include <cassert>
#include <algorithm>
#include <vector>
#include <random>
#include <chrono>
#include <string>

// Init static variables of PolyHash class:
int PolyHash::base((int)1e9+7);    
std::vector<int> PolyHash::pow1{1};
std::vector<ull> PolyHash::pow2{1};
 
// Functions for calculating this sum: 1 + a + a^2 + ... + a^(k-1) by modulo
// mod and 2^64 in O(log(k))
int sum_int(int a, int k) {
    if (k == 1) {
        return 1;
    } else if (k % 2 == 0) {
        return (1LL + a) * sum_int(1LL * a * a % PolyHash::mod, k / 2) % PolyHash::mod;
    } else {
        return 1 + (a+1LL) * a % PolyHash::mod * sum_int(1LL * a * a % PolyHash::mod, k / 2) % PolyHash::mod;
    }
}
 
ull sum_ull(ull a, int k) {
    if (k == 1) {
        return 1;
    } else if (k % 2 == 0) {
        return (1 + a) * sum_ull(a * a, k / 2);
    } else {
        return 1 + a * sum_ull(a, k - 1);
    }
}
 
int main() {    
    static char buf[1+100000];
    scanf("%100000s", buf);
    std::string a(buf);
 
    std::reverse(a.begin(), a.end()); // reverse
 
    // Gen random point of hashing:
    PolyHash::base = gen_base(256, PolyHash::mod);
 
    // Construct hashes on prefixes of substring a:
    PolyHash hash(a);
 
    // Length of string:
    const int n = (int)a.size();
 
    int answ = 0;    
    for (int len = 1; len <= n; ++len) {
        // Get polynomial hash:
        auto hash1 = hash(0, len); // on prefix a[0...len)
        auto hash2 = hash(0, n);   // on prefix a[0...n)
 
        // Multiply on sum of geometry progression hash modulo mod:
        hash1.first = 1LL * hash1.first * sum_int(PolyHash::pow1[len], n) % PolyHash::mod;
        hash2.first = 1LL * hash2.first * sum_int(PolyHash::pow1[n], len) % PolyHash::mod;
 
        // Multiply on sum of geometry progression hash modulo 2^64:
        hash1.second *= sum_ull(PolyHash::pow2[len], n);
        hash2.second *= sum_ull(PolyHash::pow2[n], len);       
 
        // Compare hashes:
        answ += (hash1 == hash2);
    }
    printf("%d", answ);
 
    return 0;
}
```

</details>

### 交換任意兩個字元一次，問最長的 LCP 有多長

[題目連結](https://www.hackerrank.com/contests/ab-yeh-kar-ke-dikhao/challenges/jitu-and-strings/problem)
目標：給定兩個字串 \\(s, t\\)，長度都是為 \\(n\\)，你能對任一個字串交換任意兩個字元一次，問最長的 [LCP](https://leetcode.com/problems/longest-common-prefix/) (Longest Common Prefix) 有多長。

> 舉例而言，字串分別是 `ABCDEEEE` 與 `ABZDEEEC`。
>
> 此時，我們選擇 `ABZDEEEC` 來交換字元，並將 `ABZDEEEC` 的 `Z` 跟 `C` 交換，成為 `ABCDEEEZ`，最終使得兩個字串的 LCP 等於 `ABCDEEE`，長度為 \\(7\\)。

<details>
  <summary>解答</summary>

  Key observation: 兩個字串中，第一個不同的字元必須要被交換。

  假設兩個字串第一個不同的字元是 \\(i\\)，那只要枚舉 \\(j, i < j < n\\) 並判斷 \\(s[i]\\) 與 \\(s[j]\\) 交換後是否會更好，以及 \\(k, i < kj < n\\) 並判斷 \\(t[i]\\) 與 \\(t[k]\\) 交換後是否會更好即可。

  怎麼判斷是否會更好呢？可以用二分搜來計算兩個字串的 LCP 長度，假設 LCP 的長度是 \\(L\\)，倘若 \\(Hash(s[:L]) \neq Hash(t[:L])\\)，那麼就猜 \\(L\\) 可能更小，反之猜 \\(L\\) 可能更大。

```C++
#include <stdio.h>
#include <cassert>
#include <algorithm>
#include <vector>
#include <random>
#include <chrono>
#include <string>
#include <iostream>
 
typedef unsigned long long ull;

// Init static variables of PolyHash class:
int PolyHash::base((int)1e9+7);    
std::vector<int> PolyHash::pow1{1};
std::vector<ull> PolyHash::pow2{1};
 
int solve(const int n, const std::string& s, const std::string& t) {
    // Gen random base of hashing:
    PolyHash::base = gen_base(256, PolyHash::MOD);
 
    // Construct polynomial hashes on prefixes of strings s and t:
    PolyHash hash_s(s), hash_t(t);
 
    // Find first not-equal symbol:
    int pos1 = 0;
    while (pos1 < n && s[pos1] == t[pos1]) ++pos1;
 
    // Try to swap pos1 with everything symbol after and use binary search for finding LCP:
    int answ = pos1;
    for (int pos2 = pos1+1; pos2 < n; ++pos2) {
        // Binary search:
        int low = 0, high = n+1;
        while (high-low > 1) {
            const int mid = (low + high) / 2;
            const auto hs = hash_s.prefix_after_swap(mid, pos1, pos2, s[pos1], s[pos2]);
            if (hs == hash_t(0, mid)) {
                low = mid;
            } else {
                high = mid;
            }
        }
        // Update answer:
        answ = std::max(answ, low);
    }
    return answ;
}
 
int main() {
    // Input:
    int n;
    scanf("%d", &n);
    char buf[1+200000];
    scanf("%200000s", buf);
    std::string s(buf);
    scanf("%200000s", buf);
    std::string t(buf);
    // Solve and output:
    printf("%d", solve(n,s,t));
    return 0;
}
```

</details>

## Codebase

```C++
// Generate random base in (before, after) open interval:
int gen_base(const int before, const int after) {
    auto seed = std::chrono::high_resolution_clock::now().time_since_epoch().count();
    std::mt19937 mt_rand(seed);
    int base = std::uniform_int_distribution<int>(before+1, after)(mt_rand);
    return base % 2 == 0 ? base-1 : base;
}
 
struct PolyHash {
    // -------- Static variables --------
    static const int mod = (int)1e9+123; // prime mod of polynomial hashing
    static std::vector<int> pow1;        // powers of base modulo mod
    static std::vector<ull> pow2;        // powers of base modulo 2^64
    static int base;                     // base (point of hashing)
 
    // --------- Static functons --------
    static inline int diff(int a, int b) { 
        // Diff between `a` and `b` modulo mod (0 <= a < mod, 0 <= b < mod)
        return (a -= b) < 0 ? a + mod : a;
    }
 
    // -------------- Variables of class -------------
    std::vector<int> pref1; // Hash on prefix modulo mod
    std::vector<ull> pref2; // Hash on prefix modulo 2^64
 
    // Constructor from string:
    PolyHash(const std::string& s)
        : pref1(s.size()+1u, 0)
        , pref2(s.size()+1u, 0)
    {
        assert(base < mod);
        const int n = s.size(); // Firstly calculated needed power of base:
        while ((int)pow1.size() <= n) {
            pow1.push_back(1LL * pow1.back() * base % mod);
            pow2.push_back(pow2.back() * base);
        }
        for (int i = 0; i < n; ++i) { // Fill arrays with polynomial hashes on prefix
            assert(base > s[i]);
            pref1[i+1] = (pref1[i] + 1LL * s[i] * pow1[i]) % mod;
            pref2[i+1] = pref2[i] + s[i] * pow2[i];
        }
    }
 
    // Polynomial hash of subsequence [pos, pos+len)
    // If mxPow != 0, value automatically multiply on base in needed power. Finally base ^ mxPow
    inline std::pair<int, ull> operator()(const int pos, const int len, const int mxPow = 0) const {
        int hash1 = pref1[pos+len] - pref1[pos];
        ull hash2 = pref2[pos+len] - pref2[pos];
        if (hash1 < 0) hash1 += mod;
        if (mxPow != 0) {
            hash1 = 1LL * hash1 * pow1[mxPow-(pos+len-1)] % mod;
            hash2 *= pow2[mxPow-(pos+len-1)];
        }
        return std::make_pair(hash1, hash2);
    }
};
```

## 資源

[blog](https://codeforces.com/blog/entry/60445)

[Suffix Array & Rolling Hash](https://www.luogu.com.cn/blog/blackfrog/sa-algorithm)

[Minimum cyclic shift](https://acmp.ru/asp/do/index.asp?main=task&id_course=2&id_section=18&id_topic=43&id_problem=284&locale=en)

[Modulo inverse](https://medium.com/algorithm-solving/modular-multiplicative-inverse-333ab622d886)

[Binary Search](https://oi-wiki.org/basic/binary/)
