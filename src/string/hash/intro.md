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

將各項的係數和次方相乘，得到 [104, 3131, 1056488, 31381032, 101738233]。

將所有項的結果相加，得到 103831988。

對模數值 m 取模，得到最終的哈希值 388026728。

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

- Check if strings are identical
- Mathematical analysis of the collision rate
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