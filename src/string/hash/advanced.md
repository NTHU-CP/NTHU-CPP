# Advanced applications of Rolling Hash
Rolling Hash 也許看似簡單，但是要靈活運用才能發揮他的完整威力，下面給大家一些習題，靈活使用工具才能解出來喔。

## Problems
總共有五題，不會說哪題要先寫出來，後面的題目才知道要怎麼寫。筆者有幫各位把題目由簡單排到難了。

### 字典序最小的 Cyclic Shift
> 目標：在 \\(O(n\ \log n)\\) 的時間內找到長度為 \\(n\\) 的字串的字典序最小循環移位

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
> 目標：在 \\(O(n\ log^2 n)\\) 的時間內，按字典序對長度為 \\(n\\) 的字串的所有循環移位進行排序

> 字串 \\(s\\) 的循環移位 (Cyclic Shift) 是指將字串 \\(s\\) 按照某個 \\(k, 0 \leq k < n\\)進行移位得到的字串 \\(s_{k+1}s_{k+2} ... s_n s_1 s_2 ... s_k\\)，其中 \\(n\\) 是字串 \\(s\\) 的長度。
>
> 對於給定的字串，請構造所有的循環移位，並將每個循環移位排序。接著，輸出每個排序後字串的最後一個字元，以及給定的字串位於排序後列表的第幾名。

> 是可以建 Suffix Array 去解決這個問題沒錯，但是這樣太複雜了。你能不能粗暴 (但是很簡單) 的用 Rolling Hash 解決它呢？

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
> Finding the number of sub-palindromes of a string of length n in O(n log n) time

先來一個 Key observation，如果 \\(T[i-j:i+j]\\) 是回文字串，那麼 \\(T[i-(j-1):i+(j-1)]\\) 也是回文字串。

### 交換任意兩個字元一次，問最長的 LCP 有多長
> Largest common prefix of two strings length n with swapping two chars in one of them in O(n log n) time

### 問有幾個後綴滿足特定條件
> The number of suffixes of a string of length n, the infinite extension of which coincides with the infinite extension of the given string for O(n log n) (extension is a duplicate string an infinite number of times)

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