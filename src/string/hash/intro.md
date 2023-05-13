# Introduction to Rolling Hash

# 前言
Skip
<br>

# Outlines

# How to calculate Rolling Hash
Hash (哈希)，是一種將字串壓縮成一個數字的方法，藉此加速字串匹配。而在競技程式設計中，最常用的方法是 Polynomial Hash (多項式哈希)，在台灣更多被稱作 Rolling Hash (滾動哈希)。

## 如何計算
多項式哈希（Polynomial Hash）是一種常用的哈希算法，它通常用於字符串匹配、文本搜索、比較和壓縮等應用場景。其基本思想是將字符串看作一個多項式，並使用多項式求值的方式來計算字符串的哈希值。

以下是計算 Polynomial hash 的基本步驟：

首先，需要選擇一個適合的模數值 m（通常是一個大質數），用於取模運算。

然後，需要選擇一個固定的基數 a（通常是一個大質數），用於計算多項式中各項的次方。

將字符串中的每個字符轉換為一個對應的數字（通常使用 ASCII 碼值），作為多項式中相應項的係數。

將多項式中各項的次方分別計算出來，可以使用快速冪算法（例如二分冪）來實現。

將多項式中各項的係數和次方相乘，然後將所有項的結果相加，最後對模數值 m 取模，得到最終的哈希值。

## 範例
例如，對於一個字符串 s = "hello"，假設基數 a = 31，模數值 m = 10^9 + 7，則可以按照以下方式計算其 Polynomial hash：

將字符串 s 轉換為對應的數字序列，得到 [104, 101, 108, 108, 111]。

計算多項式中各項的次方，得到 [1, 31, 961, 29791, 923521]。

將各項的係數和次方相乘，得到 [104, 3131, 103788, 3217428, 102509832]。

將所有項的結果相加，得到 105834283。

對模數值 m 取模，得到最終的哈希值 105834283。

## Python Code
```python
def polynomial_hash(s, a, m):
    # 初始化哈希值為 0
    h = 0
    # 計算多項式中各項的次方
    p = 1
    for c in s:
        # 將字符轉換為對應的數字
        x = ord(c)
        # 更新多項式中各項的係數
        h = (h + x * p) % m
        # 更新多項式中各項的次方
        p = (p * a) % m
    # 返回最終的哈希值
    return h
```

## 判斷字串是否相等
很簡單的，比對兩個哈希值 (Hash Value) 是否相等即可，舉例而言，有兩個字串 $S_1$ 與 $S_2$ 要判斷是否相等，那麼，判斷 $polynomial_hash(s_1, a, m) == polynomial_hash(s_2, a, m)$ 即可。

# Hash Collision and Mathematical Analysis
哈希碰撞（Hash Collision）指的是兩個不同的輸入值在經過哈希函數計算後產生了相同的哈希值。由於哈希函數的值域通常比輸入值的集合小得多，因此哈希碰撞是不可避免的現象。

而在 Rolling Hash 中，哈希碰撞就會導致本不該相等的兩個字串，被視為相同的字串，最終使得程式的邏輯出現錯誤。

以 $a=1, m=2$ 為例，字串 `ab` 與 `ba` 有相同的哈希值，進而兩者被視為相等的字串，但實際上這兩者並不相等，最終使得程式結果錯誤。

而要估計 Rolling Hash 的 Collision Rate，可以使用生日悖論（Birthday Paradox）的概念。

生日悖論指的是當有一群人在一起時，假設他們的生日是獨立均勻分布在一年中的每一天，那麼當這個群體的大小達到一定程度時，至少有兩個人的生日是相同的機率遠高於我們通常感覺到的機率。具體來說，當群體的大小為23人時，至少有兩人生日相同的概率已經超過50%。

將生日悖論應用到哈希碰撞的情境中，假設我們有一個哈希函數 $h$，它將 $n$ 個不同的輸入值映射到 $m$ 個哈希值中的某一個。如果我們隨機選擇 $k$ 個輸入值，那麼它們中至少有兩個輸入值的哈希值相同的概率就可以用生日悖論的公式計算：

$P(k,m)=1−\frac{(m−k)!m}{k \cdot m!}$
​
其中 $P(k, m)$ 表示選擇 $k$ 個輸入值中至少有兩個輸入值的哈希值相同的概率，$m$ 表示哈希值的空間大小。

對於 Polynomial Hash，哈希值空間的大小可以看作是模 $p$ 的值，其中 $p$ 是一個大質數。因此，我們可以使用上述公式來估計 Polynomial Hash 的 Collision Rate。

# Picking the correct modulo and base
首先，選定的 $m$ 跟 $a$ 必須要互質，假設 $m$ 跟 $a$ 不互質的話，就會造成 $a^1, a^2, ..., a^m$ 中出現重複的數字，進而使得哈希碰撞的概率上升。

以 $m=8, a=6$ 為例，$a^1=6, a^2=4, a^3=a^4=a^5=a^6=0$，可以觀察到有多個數字重複，因此必須選擇互質的 $m, a$。

其次，不要選擇 $2$ 的冪次作為 $m$，這會讓出題者能夠構造一個 Thue-Morse Sequence 去攻擊你的 Hash，後續章節會介紹更多怎麼攻擊 Rolling Hash。

最後，建議選擇幾組冷門的 $m, a$ 作為你的模板，許多邪惡的出題者會對常用的 $m, a$ 構造容易碰撞的測資，如果你選的 $m, a$ 夠冷門（像是你的生日），就不容易被出題者猜中。下面是幾個常見的 $m, a$
1. $m = 10^N+7$
2. $m = 2^N+1$
3. $m = 2 \cdot 3 \cdot 5 \cdot 7 ...$，然後 $a$ 選再下一個質數
4. $a = 2, 3, 4, 5, 6, 7, 8, 9, 10$

- Compare the string lexicographically (by binary search)
- A simple introduction of how to pick the right modulo and base (General case)
- Basic hash problems
- Get the hash value of a substring
- Alternative solution of KMP, Manacher
- For normal competitive programmers

請搭配一個範例

# 總結

# 資源

待補