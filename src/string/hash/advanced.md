# Advanced applications of Rolling Hash
Rolling Hash 也許看似簡單，但是要靈活運用才能發揮他的完整威力，下面給大家一些習題，靈活使用工具才能解出來喔。

## Problems
總共有五題，不會說哪題要先寫出來，後面的題目才知道要怎麼寫。筆者有幫各位把題目由簡單排到難了。

### 字典序最小的 Cyclic Shift
> 目標：在 \\(O(n\ \log n)\\) 的時間內找到長度為 \\(n\\) 的字串的字典序最小循環移位


### 排序所有 Cyclic Shift
> Sorting of all cyclic shifts of a string of length n in lexicographic order in O(n log^2 n) time

### 算有幾個 substring 是回文字串
> Finding the number of sub-palindromes of a string of length n in O(n log n) time

先來一個 Key observation，如果 \\(T[i-j:i+j]\\) 是回文字串，那麼 \\(T[i-(j-1):i+(j-1)]\\) 也是回文字串。

### 交換任意兩個字元一次，問最長的 LCP 有多長
> Largest common prefix of two strings length n with swapping two chars in one of them in O(n log n) time

### 問有幾個後綴滿足特定條件
> The number of suffixes of a string of length n, the infinite extension of which coincides with the infinite extension of the given string for O(n log n) (extension is a duplicate string an infinite number of times)

## 資源

[blog](https://codeforces.com/blog/entry/60445)
[Suffix Array & Rolling Hash](https://www.luogu.com.cn/blog/blackfrog/sa-algorithm)
[Minimum cyclic shift](https://acmp.ru/asp/do/index.asp?main=task&id_course=2&id_section=18&id_topic=43&id_problem=284&locale=en)