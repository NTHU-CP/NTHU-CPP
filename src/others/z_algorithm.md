Z algorithm
===

大綱
---

1. Z algorithm介紹
2. 實作程式碼
3. 時間複雜度分析
4. 相關題目
5. 總結
6. 參考資料

Z algorithm介紹
---

將Z algorithm套用於長度為 $n$ 的字串 $S$ ，可以用 $O(n)$ 的複雜度運算出一個Z array : $Z$。

$Z[i]$ 紀錄在 $index = i(i>0)$ 時， $S[i:n-1]$ 與 $S[0:n-1]$ 連續最長共同前綴的長度。
而Z array具有一些性質可以幫助我們達成一些字串的查找與排序。

note:
設 $Z[0] = 0$，等等求其他性質與兩字串間比較比較方便，如要求 $S[0:n-1]$ 與 $S[0:n-1]$ 連續最長共同前綴的長度，因兩範圍相等，故直接等於字串長度n

程式碼實作
---

為了建構出 Z array ，我們必須初始 $L,R$ 並使用 for 迴圈掃描過 $S[0:n-1]$ 。

### 結構

```c++=
vector<int> buildZ(const string &S) {
    //TODO: L,R
    int n = S.size();
    vector<int> Z(n);
    Z[0] = n; //initialize the Z[0]
    for (int i = 1; i < n; ++i){
        //TODO: 3 Cases
        //...
        //...
    }
    return Z;
}
```

### L and R

我們想使用for迴圈掃描過 $S$ 中的所有元素，並且在 $index = i$ 時產生 $Z[i]$ ，最終以均攤 $O(n)$ 的複雜度完成Z array。

於是我們維護一組 $L,R$ 使得 $0≤L,R≤n-1$ ，該範圍滿足 $Z[L:R] = Z[0: R-L]$ 在每次進入迴圈時更新。
如果每次的 $L_{org}≤L_{new},R_{org}≤R_{new}$ ，因為 $L$ 與 $R$ 最多更新 $n-1$ 次，那麼總共更新 $L, R$ 的複雜度均攤下來為 $O(n)$ 。

初始的 $L=R=0$

```c++=
int L = 0, R = 0;
```

### 三種Cases

#### 1. $i$ 不在 $[L,R]$ 中。

**Case 1**: $R < i$ or $i < L$

>不可用 $L,R$ 性質快速找出 $Z[i]$ 。
更新 $Z[i]$ 為 $0$ ，while迴圈更新 $Z[i]$ 直到兩元素不相等。
更新 $L,R$ 。

```c++=
if (R < i || i < L) { //不在範圍內
    Z[i] = 0;
    while (S[Z[i]] == S[i + Z[i]])
        ++Z[i];
    L = i, R = i + Z[i] - 1;
}
```

#### 2. $i$ 在 $[L,R]$ 中

![螢幕快照 2023-06-22 23-05-29](https://github.com/williamlin0208/NTHU-CPP/assets/122626646/467f1b1e-2086-4f6d-afd4-127ab4e7f737)

依照定義， $S[L:R] = S[0:R-L]$。
故 $S[0 : 0+Z[i-L]-1] = S[i : i+Z[i-L]-1]$ 。

$R-i+1$ 表示 $S[i:R]$ 共有幾個字元。

因為我們在處理完 index = 1 之後，必定 $L \ge 1$ 。
因為 $\forall$ $0 \le j < i$ ， $Z[j]$ 都在已被計算出來，我們可以利用 $Z[i-L]$ 快速計算 $Z[i]$ 的結果，於是我們分別處理 $Z[i-L] \lt R-i+1$(不可擴張) 和 $Z[i-L] \ge R-i+1$(具擴張潛力)兩種情況。

**Case 2**: $L \le i \le R$ and $Z[i-L] \lt R-i+1$

![螢幕快照 2023-06-22 23-13-11](https://github.com/williamlin0208/NTHU-CPP/assets/122626646/de19d0de-4a04-4e56-8efa-30658bd8bc2d)

>因為 $Z[i-L]$ 表示 $S[i-L:n-1]$ 與 $S[0:n-1]$ 連續最長共同前綴的長度。
>故如果 $Z[i-L] \lt R-i+1$ ， 則必定 $S[0+Z[i-L]] \not= S[i+Z[i-L]]$ 。
>因此， $Z[i]=Z[i-L]$ 。
>我們只需要將 $Z[i]$ 更新為 $Z[i - L]$ 。

```c++=
if (L <= L && i <= R && (Z[i-L] < R-i+1)) { //在範圍內且滿足(i+Z[i-L]-1 < R)
    Z[i] = Z[i - L];
}
```

**Case 3**: $L \le i \le R$ and $Z[i-L] \ge R-i+1$

![螢幕快照 2023-06-22 23-14-00](https://github.com/williamlin0208/NTHU-CPP/assets/122626646/dda3a9bd-7a25-485a-b4bc-58ce90dcd50b)

>$Z[i-L] \ge R-i+1$ 代表 $S[i:R]$ 完全等於 $S[i-L:R-L]$ ，因為我們只保證 $S[L:R]$ 和 $S[0:R-L]$ 內所有元素皆相等，我們無法確定之後的元素是否相等，故需要往後找直到兩元素不相等。
>$Z[i]$ 至少等於 $R-i+1$ ， $Z[i]$ 更新為 $R-i+1$。
>使用while迴圈更新 $Z[i]$ 直到兩元素不相等。
>更新 $L,R$ 。

```c++=
if (L <= L && i <= R && (Z[i-L] >= R-i+1)) { //在範圍內且滿足(Z[i-L] >= R-i+1)
    Z[i] = R-i+1;
    while (S[Z[i]] == S[i + Z[i]])
        ++Z[i];
    L = i, R = i + Z[i] - 1;
}
```

### 未化簡程式碼

```c++=
vector<int> buildZ(const string &S) {
    int L = 0, R = 0; //initialize L,R
    int n = S.size();
    vector<int> Z(n);
    for (int i = 1; i < n; ++i) {
        if (R < i || i < L) { //Case 1
            Z[i] = 0;
            while (S[Z[i]] == S[i + Z[i]])
                ++Z[i];
            L = i, R = i + Z[i] - 1;
        }
        if (L <= L && i <= R && (Z[i-L] < R-i+1)) { //Case 2
            Z[i] = Z[i - L];
        }
        if (L <= L && i <= R && (Z[i-L] >= R-i+1)) { //Case 3
            Z[i] = R-i+1;
            while (S[Z[i]] == S[i + Z[i]])
                ++Z[i];
            L = i, R = i + Z[i] - 1;
        }
    }
    return Z;
}
```

將條件簡化一下...

```c++=
vector<int> buildZ(const string &S) {
    int L = 0, R = 0; //initialize L,R
    int n = S.size();
    vector<int> Z(n);
    for (int i = 1; i < n; ++i) {
        if (R < i || i < L) { //Case 1
            Z[i] = 0;
            while (S[Z[i]] == S[i + Z[i]])
                ++Z[i];
            L = i, R = i + Z[i] - 1;
        }else if (Z[i-L] < R-i+1) { //Case 2
            Z[i] = Z[i - L];
        }else{ //Case 3
            Z[i] = R-i+1;
            while (S[Z[i]] == S[i + Z[i]])
                ++Z[i];
            L = i, R = i + Z[i] - 1;
        }
    }
    return Z;
}
```

觀察一下繼續化簡...

Case1 與 Case 3重複很多，只差了一個 $Z[i]$ 的初始值，能不能省略一下?

如果不是Case2 -> Case1 or Case3

if Case 1: $R < i$ and $R-i+1 \le 0$
if Case 3: $R \ge i$ and $R-i+1 > 0$

因為 $R-i+1 \le 0$ 在 Case 1， $0 < R-i+1$ 在 Case 3。
$max(0,R-i+1)$ 可以同時滿足這兩個Case時 $Z[i]$ 的初始值。

於是我們可以將Case 1 與 Case 3合併。

### 化簡後程式碼

```c++=
vector<int> buildZ(const string &S) {
    int L = 0, R = 0; //initialize L,R
    int n = S.size();
    vector<int> Z(n);
    for (int i = 1; i < n; ++i) {
        if (Z[i-L] < R-i+1) { //Case 2
            Z[i] = Z[i - L];
        }else{ //Case1 & Case 3
            Z[i] = max(0,R-i+1);
            while (S[Z[i]] == S[i + Z[i]])
                ++Z[i];
            L = i, R = i + Z[i] - 1;
        }
    }
    return Z;
}
```

三種Cases:
Case 1: $R < i$ or $i < L$ ( $i$ 於 $L,R$ 範圍之外)
Case 2: $L \le i \le R$ and $Z[i-L] \lt R-i+1$ ( $i$ 於 $L,R$ 範圍之內，不可擴展)
Case 3: $L \le i \le R$ and $Z[i-L] \ge R-i+1$ ( $i$ 於 $L,R$ 範圍之內，具擴展潛力)

### 觀察與疑問

為什麼 for 迴圈從 index = 1 開始而不可以從 index = 0 ?

假設我們從 index = 0 開始，那麼在第一次進入迴圈便會進入 Case 1 並將 $L,R$ 設為 $0,n-1$ 。
如果 $L = 0$ 、 $R = n-1$ ，會產生什麼問題?

在非 Case 1 的情況中，我們必須判斷是否可擴張，進入 Case 2 或 Case 3。
而為了判斷是否可擴張，我們必須比較 $S[L:R]$ 和 $S[0:R-L]$。

如果 $L = 0$ ，
$L = 0$ 、 $R = R-L$ ， $S[L:R]$ 和 $S[0:R-L]$ 的區間相同時，在判斷 $Z[i-L] < R-i+1$ 時， $i-L = i$ ， $Z[i]$ 原先尚未被初始化，並且用來比較的index等於要被計算出來的index，故無法利用 $Z[i-L]$ 計算出 $Z[i]$ 。

時間複雜度分析
---

首先，我們有一個 for 迴圈在外層，時間複雜度 $O(n)$ 。

我們同時維護一組 $L,R$ 使得 $0≤L,R≤n-1$ ，在每次進入迴圈時更新。
如果每次的 $L_{org}≤L_{new},R_{org}≤R_{new}$ ，因為 $L$ 與 $R$ 最多更新 $n-1$ 次，那麼總共更新 $L, R$ 的複雜度均攤下來為 $O(n)$ 。

另外，在每一次的迴圈中，維護 L,R 同時也求出 $Z[i]$ 的值，故建造 Z array 時間複雜度為 $O(n)$ 。

$T(n) = O(n) + O(n) + O(n) = O(n)$

相關題目
---

### CSES - Finding Periods

找到自己的前綴子序列

模版題，先計算出Z array，使用for迴圈掃過一遍。 $O(n+n) = O(n)$

在Z array 中，當index = $i$ ， $0 \le Z[i] \le n-i$ 。
如果 $Z[i] = n-i$ ，那麼代表 $S[i:n-1]$ 可以被 $S[0:i-1]$ 重複並以 $S[0:i-1]+S[0:i-1]...+S[0:i-1]$ 的形式組合起來(**最後一次可以是 $S[0:i-1]$ 的前綴**)。
故 $S[0:i-1]+S[i:n-1]$ 亦可以被 $S[0:i-1]$ 重複組合起來。

如果此種情況發生，我們稱 $Z[0:i-1]$ 為 $S$ 的前綴子序列。

另外，因為S自己亦是S的子序列，要記得將自己加入自己的週期集合中。

note:
因為 $Z[i]$ 代表 $Z[i:n-1]$ 與 $Z[0:n-1]$ 的最長共同前綴長度。

正常來說應該 $Z[0] = n$，但這樣會出現一個問題，當index = $0$ ， $Z[i] = n-i$，符合我們判斷period的標準，但如果 $i=0$ 會導致 $S[0:i-1]$ 成為一空字串，無法成為任一有效字串的period。

於是前面我們將 $Z[0]$ 設為 $0$ ，避免此情況發生，方便進行判斷。

```c++=
void find_ans(string &s,vector<int> &z){
    int sz = s.size();
    for(int i=0;i<sz;i++){
        if(z[i] == sz-i) cout<<i<<" ";
    }
    cout<<sz;
}
```

### Codeforces 126B - Password

找出一最長 t 同時出現在字串 s 最前面、最後面、中間(非前且非後)

創建完 Z array 後，使用for迴圈掃過一遍。 $O(n+n) = O(n)$

觀察:
當 index = $i$ , $(0 \lt i)$ , max_n 代表 $1 \le k \lt i$，最大的 $Z[k]$ 。
如果 $Z[i] = n-i$ ，那前綴等於後綴(前面、後面)，滿足 $S[i:n-1] = S[0:n-i-1]$ 。
接著使用max_n判斷此後綴 $Z[i:n-1]$ 是否於 index = $k$ , $0 \lt k \lt i$ 時出現，如果max_n $\ge Z[i]$ ，代表 $S[i:n-1]$ 必定出現於先前的某一次的 $S[k:n-1]$ 中(中間)。

因後綴長度隨著index增加而遞減，在保證在index = $i$ 時找到的答案必定比當index = $k$, ($k \gt i$)找到的答案更長的情況下，找到符合答案即可直接回傳。

回傳值是找到的字串第一個字母所在的index。

```c++=
int find_ans(string &s,vector<int> &z){
    int sz = s.size();
    int max_n = 0;
    for(int i=1;i<sz;i++){
        if(z[i] == sz-i && max_n >= z[i]) return i;
        max_n = max(max_n,z[i]);
    }
    return -1; 
}
```

總結
---

Z algorithm 以 $O(n)$ 的時間複雜度算出所有後綴子字串與本字串的共同最長前綴。
相比於暴力法 $O(n^2)$ 的時間複雜度，節省了許多時間。

並且利用Z array的性質，我們通常可以再利用一次for迴圈掃過Z array中的所有元素，找出具最大或最小某性質的子前後綴字串。

參考資料
---

[](https://theriseofdavid.github.io/2020/09/18/Codeforces/Codeforces%20126B/)
[](https://kenjichao.gitbooks.io/algorithm/content/z_algorithm.html)
[](https://wangwilly.github.io/willywangkaa/2018/03/19/Algorithm-Z-%E6%BC%94%E7%AE%97%E6%B3%95/)
