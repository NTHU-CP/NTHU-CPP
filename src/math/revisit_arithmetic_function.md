# 再探數論函數

## 引理

1. \\( \sum_{d\mid n}\mu(d)=[n=1] \\)
2. \\( f(n)=\sum_{d\mid n}g(d)\Longleftrightarrow g(n)=\sum_{d\mid n}\mu(\frac n d)f(d) \\)
3. \\( f(n)=\sum_{n\mid d}g(d)\Longleftrightarrow g(n)=\sum_{n\mid d}\mu(\frac d n)f(d) \\)
4. \\( \sum_{d\mid n}\varphi(d)=n \\)，\\( \varphi(n)=\sum_{d\mid n}\mu(\frac n d)\times d \\)

## 例題

1. \\( \sum_{i=1}^N\sum_{j=1}^M [\gcd(i,j)=1] \\)
2. \\( \sum_{i=1}^N\sum_{j=1}^M [\gcd(i,j)\ \mbox{is prime}] \\)
3. \\( \sum_{i=1}^N\sum_{j=1}^M \gcd(i,j) \\)
4. \\( \sum_{i=1}^N\sum_{j=1}^M ij\times \gcd(i,j) \\)

## 套路

可以發現，使用到的技巧不外乎是：
* 看到 \\(\gcd\\) 就枚舉他
* 看到艾佛森括號就把他換掉
* 原本枚舉因數的，就改成枚舉倍數
* 原本枚舉倍數的，就改成枚舉因數
* 把枚舉的倍數提出原本的數
* 看到長得像 Lemma 的就套套看
* 看到兩個 \\(\Sigma\\) 的變數乘在一起，就改成枚舉兩者乘積和其因數

## 習題

1. \\( \sum_{i=1}^N\sum_{j=1}^M [\gcd(i,j)\ \mbox{is a perfect number}] \\)
2. \\( \sum_{i=1}^N\sum_{j=1}^M lcm(i,j) \\)
3. \\( \sum_{i=1}^N\sum_{j=1}^M \sigma_0(ij) \\)
4. \\( \prod_{i=1}^N\prod_{i=1}^M F_{\gcd(i,j)} \\)

其中 \\( \sigma_0 \\) 是因數數量；\\( F \\) 是費氏數列。

每一題都有 \\( Q \\) 筆詢問，\\( N、M、Q\le 1e5 \\)

## 其他

* 給定一棵邊帶權樹，一個路徑的價值是路徑上所有權重的 \\( \gcd \\)，求所有相異路徑的價值總和
    * （點數、權重 \\( \le 10^5 \\) ）
* 給定 \\( a_1,\cdots,a_n\\) （\\( n\le 10^5 \\) ），求 \\( \sum_{i=1}^n \sum_{i=1}^n \gcd(a_i,a_j)\times\gcd(i,j) \\)
