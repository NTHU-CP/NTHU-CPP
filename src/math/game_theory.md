# Game Theory

## Introduction

賽局理論考慮玩家在遊戲中的行為，並研究他們的最佳化策略。

而一個有關賽局的題目，我們會想知道在某個狀態，當前玩家能不能採用一個最佳化的策略，使得對手不論如何應對，都無法阻止該玩家獲勝。這種狀態稱為 **winning state**。反之，若玩家在某狀態，不論採取何種行動，都無法阻止對手以某個策略取勝的話，就稱為**losing state**。換句話說，winning state至少存在一種行動，使得遊戲局勢落入losing state。而losing state不論採取什麼行動，都會使局勢落入winning state。

### Example

> [Codeforces Global Round 22 C - Even Number Addicts](https://codeforces.com/contest/1738/problem/C)
>
> 給一個長度為 \\( n \\) 的整數陣列。玩家A，B輪流採取行動，由A開始。玩家每回合可以將一個數字從陣列中移除。當所有數字都被移除後遊戲結束。若A移除的數字和為偶數，那麼A獲勝。反之，B獲勝。題目則問若雙方都採取最佳化策略，那麼誰會獲勝。

可以發現陣列中數字大小其實不重要，重要的是奇數偶數的數量各為多少。那麼我們可以用陣列中剩餘奇數個數，剩餘偶數個數，與目前A移除數字總和的奇偶性這三項資訊來代表一個state。

那麼該如何得知一個state是winning state或是losing state呢?
如果數字都被取完了，也代表當前的state無法轉移至其他狀態，那麼看目前A移除的數字的奇偶性即可判斷誰是勝者。

若還沒取完，就將所有能轉移的state都跑一遍(奇數偶數都試著移除)，並檢查能不能轉移至對手的losing state。如果可以的話，代表當前的state為該玩家的winning state，若不行則為losing state。

在填表時可以發現，一個狀態若轉移至另一個狀態，那麼陣列中的數字數量必定會減少。這也代表轉移的狀態其實可以視為一個有向無環圖，因此填表的過程是不會出現問題的。只要使用Top-down DP並將base case處理好，即可判斷狀態為誰的winning state。

<details><summary> Solution Code </summary>

- Time complexity: \\( O(N^2) \\)

```cpp
#include <bits/stdc++.h> 
using namespace std;
 
int N, odd_count, even_count;
int dp[110][110][2]; // 1 if it's Alice's winning state or Bob's losing state.
                     // 0 if it's Bob's winning state or Alice's losing state.
 
int recur(int x, int y, int p) {
    // x: odd count in the remaining array
    // y: even count in the remaining array
    // p: current parity of the sum of the numbers A removed
    if (dp[x][y][p] != -1) return dp[x][y][p];
    if (x == 0 && y == 0) {
        // base case, all numbers are removed
        if (p % 2 == 0) return dp[x][y][p] = 1;
        else return dp[x][y][p] = 0;
    }
    int ret;
    // determine whose turn it is 
    if ((odd_count + even_count - x - y) % 2 == 0) {
        // Alice's move
        ret = 0; // first assume it's Alice losing state.
        if (x > 0 && recur(x - 1, y, 1 - p)) ret = 1;
        if (y > 0 && recur(x, y - 1, p)) ret = 1;
        // If this state can transition to Bob's losing state, then change it to Alice's winning state.
    }
    else {
        // Bob's move
        ret = 1; // first assume it's Bob's losing state.
        if (x > 0 && recur(x - 1, y, p) == 0) ret = 0;
        if (y > 0 && recur(x, y - 1, p) == 0) ret = 0;
        // If this state can transition to Alice's losing state, then change it to Bob's winning state.
    }
    return dp[x][y][p] = ret; 
}
 
void solve(){ 
    cin >> N;
    odd_count = even_count = 0;
    memset(dp, -1, sizeof(dp));
    for (int i = 0; i < N; i++) {
        int x; cin >> x;
        if (abs(x) % 2 == 0) even_count++;
        else odd_count++;
    }
    if (recur(odd_count, even_count, 0) == 1) cout << "Alice\n";
    else cout << "Bob\n";
}       
 
int main(){
    ios_base::sync_with_stdio(false);
    cin.tie(NULL);
    
    int TestCases; 
    cin >> TestCases;
 
    while (TestCases--) {
        solve();
    }
}    
    
```

</details>

## Impartial Game

不偏賽局(impartial game)是賽局中一種常見的類型。
該賽局包含了以下性質:
- 是一種回合制的雙人遊戲
- 平等 - 雙方在遊戲中的唯一差別只有先後手
- 雙方都能看到完整的遊戲局勢
- 無隨機成分
- 在**有限**的回何數內會結束

對於一個不偏賽局，我們可以將其轉化成Nim Game的形式。因此下面會簡單介紹Nim Game的規則，性質以及一些延伸內容。

## Nim Game

### Rules

有 \\( N \\) 堆石頭，每堆有 \\( a_i \\) 個。玩家A，B輪流取石頭，每次可以選擇一堆，並從該堆取出任意數量的石頭(不能不取)。取得最後一塊石頭的玩家即為勝者。

### Strategy

將每堆的石頭個數進行 \\( bitwise\ XOR \\) 運算，得到的答案若為0，代表當前的局勢為losing state，答案不為0則為winning state。玩家若發現目前局勢為winning state時，保證會有一種取法，使得輪到對手時，所有堆的石頭個數 \\( XOR \\) 後為0，也就是losing state。而按照該策略持續下去，不論對手如何應對，都無法挽回局勢。

### Proof

這邊提出一個數學歸納法的證明。為了方便後續說明，將 \\( XOR \\) 得出的結果命名為 \\( s \\) 。

#### Base Case: 
若沒有任何石頭，則 \\( s\\) 為 \\( 0 \\) ，玩家無法採取行動，為一個losing state。 

#### Inductive Case:
若當前為losing state，亦即 \\( s=0 \\) 。因為遊戲規定每回合都必須取石頭。若從有 \\( x \\) 塊的石頭堆取石頭，設取完剩 \\( y \\) 塊 \\( (x>y) \\) ，那麼取完後的 \\( XOR \\) 答案為 \\( (s \oplus x \oplus y) = (x \oplus y)\neq 0 \\) ，一定會使局勢變為winning state。 

若當前為winning state，即 \\( s \neq 0 \\) 。考慮 \\( s \\) 的most significant bit(假設從右到左為第 \\( k \\) 個bit)，那麼至少存在某一堆石頭的個數(設為 \\( x \\) )，且該 \\( x \\) 從右到左的第 \\( k \\) 個bit也同樣為 \\( 1 \\) (否則 \\( s \\) 的該bit就不會是 \\( 1 \\) )。那麼從該堆石頭中取石頭，取到剩下 \\( (s \oplus x ) \\) 個，那麼取完之後 \\( XOR \\) 的值就會變成 \\( (s \oplus x \oplus (s \oplus x)) = 0 \\)，是一個losing state。而該堆取完後剩 \\( (s \oplus x) \\) 個，該數的第 \\( k \\) 個bit變為 \\( 0 \\) ，由此得知 \\( (s \oplus x) < x \\) ，因此該取法沒有違反規則。

綜上所述以及數學歸納法，且遊戲在有限的回合數內會結束，因此該策略是正確的。

## Nim Game Extension

將原本的Nim Game規則修改成，玩家可以在自己的回合選擇其中一堆石頭並從該堆石頭**移除或增加任意數量**的石頭，那麼玩家的最佳化策略為何呢?
答案是策略會與原本的Nim Game完全一致。若對手選擇新增一些石頭，那麼該玩家可以在下一步將該數量的石頭移除，使得game state回到兩回合前。若遊戲會在有限回合數內結束，那麼該策略依舊是最佳的。

### Example

> [CSES 1099 - Stair Game](https://cses.fi/problemset/task/1099)
>
> 有 \\( N \\) 階的階梯，編號為第 \\( 1 \sim N \\) 階，每階上都放有非負整數顆球。玩家 \\( A\, B \\) 輪流採取行動，由 \\( A \\) 開始。每回合可以選擇一個整數 \\( x \\) \\( (x > 1) \\)，並從第 \\( x \\) 階取任意正整數顆球並移到第 \\( x - 1 \\) 階。到最後，沒有任何方法移動球的一方落敗。題目則問若雙方都採取最佳化策略時，誰會獲勝。
    
經由打表和一些觀察可以發現，若將**偶數階階梯**的球數進行 \\( XOR \\) 運算，若為 \\( 0 \\) 代表該state為losing state，若不為 \\( 0 \\) 則為winning state。
    
該如何驗證這個策略的正確性呢?這邊提供一個與Nim Game有關的想法。將偶數階階梯的球數視為Nim Game每堆的石頭數，且每回合的行動是可以任取一堆並增加該堆石頭數的。而對於一個偶數 \\( K\ (K  \geq  2) \\)，若當時第 \\( K + 1 \\) 階有 \\( x \\) 顆球，則代表第 \\( K \\) 階的石頭堆在當時的回合最多可以新增 \\( x \\) 塊石頭。而因為能新增的石頭數量是**有限**的，因此可以採用同上個章節"Nim Game Extension"的策略。
    
<details><summary> Solution Code </summary>

- Time complexity: \\( O(N) \\)
    
```cpp
#include <bits/stdc++.h>
using ll = long long;
using namespace std;

void solve() {
    int N; cin >> N;
    ll state = 0;
    for (int i = 1; i <= N; i++) {
        ll x; cin >> x;
        if (i % 2 == 0) state ^= x;
    } 
    cout << (state == 0 ? "second" : "first") << '\n';
}

int main() {
    ios_base::sync_with_stdio(false);
    cin.tie(NULL);

    int testcases; 
    cin >> testcases;
    while (testcases--) {
        solve();
    }
}
    
```

</details>
    
## Sprague-Grundy Theorem

在介紹此定理以前，先引入一個符號 \\( MEX \\) 。對於一個集合 \\( s \\) ， \\( MEX \lbrace s \rbrace \\) 的定義為不屬於 \\(s \\) 的數字中最小的非負整數。例如 \\( MEX \lbrace 0, 1, 3, 5 \rbrace = 2 \\) ， \\( MEX \lbrace 1, 2, 3 \rbrace = 0 \\) 。
根據Sprague-Grundy Theorem，一個不偏賽局可以被轉換成Nim Game的形式。對於一個狀態(局勢)，我們定義它的**Grundy Number**(SG value)為 \\( MEX \lbrace g_1, g_2, ..., g_n \rbrace \\) ，這裡的 \\( g_i \\) 代表當前的state可以轉移到某 \\( n \\) 個不同的state，且 \\( g_1, g_2, ... g_n \\) 分別為那些state的Grundy Number。若一個state的Grundy Number為0，代表當前為losing state。若不為0，則為winning state。
另外，若遊戲方式為好幾個子遊戲同時進行。我們可以採用與Nim Game相似的作法，將若干個子遊戲的Grundy Number使用 \\( XOR \\) 來進行合併，得出整個遊戲的狀態。

定理的證明可以利用數學歸納法與 \\( XOR \\) 的性質。但這裡提供一個直觀的想法供讀者參考:
Nim Game每回合的行動可以從數量為 \\( x(x>0) \\) 的石頭堆取出任意數量，在取後的剩餘石頭數有可能為 \\( 0 \sim x-1 \\) 的任意一個。相似地，Grundy Number的性質確保，一個 SG value為x的state，對於任意一個 \\( i\ (x-1\ge i\ge0, \ i \in Z) \\) ，都至少存在一種轉移方法，使玩家在一回合內轉移到SG value為 \\( i \\) 的state。與Nim Game不同之處在於，某些不偏賽局在轉移後，有可能會使SG value變大。那麼這時可以用"Nim Game Extension"的策略，把對手上一回合的行動消除，即可維持在winning state。

### Example
    
> [CSES 1098 - Nim Game II](https://cses.fi/problemset/task/1098)
>
> 有 \\( N \\) 堆石頭。兩名玩家輪流採取行動，每回合可以選擇其中一堆石頭並移除 \\( 1 \sim 3 \\) 個石頭。若玩家無法移除則落敗。題目問若雙方都採取最佳化策略的話，誰將獲勝。

經由觀察可以發現，若一堆石頭有 \\( a_i \\) 個，那麼該堆的Grundy Number為 \\( a_i\ mod\ 4 \\)。而根據上述的定理，可以用 \\( XOR \\) 將各堆的Grundy Number進行合併，得到的數字即可判斷勝敗。

<details><summary> Solution Code </summary>

- Time complexity: \\( O(N) \\)
    
```cpp
#include <bits/stdc++.h>
using ll = long long;
using namespace std;

void solve() {
    int N; cin >> N;
    ll state = 0;
    for (int i = 0; i < N; i++) {
        ll x; cin >> x;
        state ^= (x % 4);
    }
    cout << (state % 4 == 0 ? "second" : "first") << '\n';
}

int main() {
    ios_base::sync_with_stdio(false);
    cin.tie(NULL);

    int testcases; 
    cin >> testcases;
    while (testcases--) {
        solve();
    }
}
    
```

</details>
    

## Exercises
    
> [Atcoder Beginner Contest 297 G - Constrained Nim 2](https://atcoder.jp/contests/abc297/tasks/abc297_g)
>
> 給 \\( N,L,R \\) 三個整數 \\( 1 \leq L \leq R \leq 10^9 \\)。玩家 \\( A, B \\) 玩Nim Game ( \\( N \\) 堆石頭，每堆有 \\( 1 \sim 10^9 \\) 個)，但每回合只能從其中一堆取 \\( L \sim R \\) 個石頭。問若雙方都採取最佳化策略，誰將獲勝。    

<details><summary>Solution</summary>
    
打表找規律後可以發現每堆的SG-value為 \\( (x\ mod\ (L + R)) / L \\) ， \\( x \\) 為該堆石頭個數。而根據上述的定理，可以由 \\( XOR \\) 進行合併，得到最後的SG-value。
    
證明的話可以使用數學歸納法。
分成: 
    
* \\( 0 \leq x < L + R \\)。
在滿足上述不等式的前提下，這個case又可以分成 \\( 0 \leq x < L ,\ \ \  L \leq x < 2L,\ \ \ 2L \leq x < 3L... \\) 。
* \\( L + R \leq x < 2(L + R) \\)。
                        而這個case可以分成 \\( L+R \leq x < L+R+L, \ \ \ L+R+L \leq x < L+R+2L... \\)。
* 接著再討論 \\( x \geq 2(L+R) \\) 的case。

</details>
    
<details><summary> Solution Code </summary>

- Time complexity: \\( O(N) \\)
    
```cpp
#include <bits/stdc++.h>
using namespace std;
using ll = long long;

int main() {
    ios_base::sync_with_stdio(false);
    cin.tie(NULL);

    ll N, L, R; 
    cin >> N >> L >> R;
    ll state = 0;
    for (int i = 0; i < N; i++) {
        ll x; cin >> x;
        state ^= (x % (L + R) / L);
    }
    cout << (state == 0 ? "Second" : "First") << '\n';
}
    
```


    
</details>

// 其他題目待補

## Summary

在解決一個賽局相關的題目時，步驟可以總結為以下幾點
1. 將遊戲局勢轉換成狀態 (或是某個數值 ex: SG value)
2. 理解各個狀態之間如何轉移
3. 判斷獲勝或落敗條件

而在題目範圍太大或是想不到解法時，打表找規律也是一個不錯的辦法。

## Reference
- [The Intuition Behind NIM and Grundy Numbers in Combinatorial Game Theory](https://codeforces.com/blog/entry/66040)
- [Sprague-Grundy theorem. Nim - Algorithms for Competitive Programming](https://cp-algorithms.com/game_theory/sprague-grundy-nim.html)
- [公平组合游戏 - OI wiki](https://oi-wiki.org/math/game-theory/impartial-game/)
- [Sprague-Grundy定理是怎么想出来的](https://zhuanlan.zhihu.com/p/20611132)
