# Game Theory

## Introduction

賽局理論考慮玩家在遊戲中的行為，並研究他們的最佳化策略。

而一個有關賽局的題目，我們會想知道在某個狀態，當前玩家能不能採用一個最佳化的策略，使得對手不論如何應對，都無法阻止該玩家獲勝。這種狀態稱為 **winning state**。反之，若玩家在某狀態，不論採取何種行動，都無法阻止對手以某個策略取勝的話，就稱為 **losing state**。換句話說，winning state 至少存在一種行動，使得遊戲局勢落入 losing state。而 losing state 不論採取什麼行動，都會使局勢落入 winning state。

### Examples

> [CSES 1729 - Stick Game](https://cses.fi/problemset/task/1729)
>
> 考慮一個遊戲，規則為雙方輪流從一堆棍子中移除若干個棍子，若輪到某玩家且他沒有任何合法的方法可以移除棍子時，則對方獲勝。已知整數 \\( n \\) ， 整數陣列 \\( p \\) ， \\( p_i \\) 為玩家在每回合可以選擇移除的個數。試問在目前還有 \\( k \\) \\( (1 \leq k \leq n) \\) 個棍子的情況下，且雙方都採取最佳化策略，那麼目前的玩家會獲勝還是落敗。

首先可以發現，若剩下 \\( 0 \\) 個棍子，玩家必定無法轉移到其他狀態，相當於落敗，為一個 losing state。
剩餘的情況則可以把所有行動都試一遍，看能否轉移到 losing state。如果可以，代表現在的狀態是一個 winning state。反之則為 losing state。

而在填表時可以發現，在狀態轉移時，剩餘的棍子數量一定會減少，同時代表狀態的數字也會減少。因此整張 state graph 可以視為一個有向無環圖，填表的過程不會有問題產生。 

<details><summary> Solution Code </summary>

- Time complexity: \\( O(nk) \\)
- Bottom-up DP, fill in states from \\(0 \\) to \\( n \\). 

```cpp
#include <bits/stdc++.h>
using namespace std;

void solve() {
    int n, k; cin >> n >> k;
    vector<int> moves;
    vector<bool> state(n + 1); // 0 for losing state, 1 for winning state
    for (int i = 0; i < k; i++) {
        int x; cin >> x;
        moves.emplace_back(x);
    }
    state[0] = 0; // base case
    for (int i = 1; i <= n; i++) {
        state[i] = 0;
        for (auto m: moves) {
            int new_state = i - m;
            // able to transition to losing state means that current state is winning state
            if (new_state >= 0 && state[new_state] == 0) state[i] = 1;
        }
    }
    for (int i = 1; i <= n; i++) {
        cout << (state[i]? 'W' : 'L');
    }
}

int main() {
    ios_base::sync_with_stdio(false);
    cin.tie(NULL);

    solve(); 
}
    
```

</details>


接著再看另外一題。


> [ZEROJUDGE b902 - 肉墊遊戲](https://zerojudge.tw/ShowProblem?problemid=b902)

> 有兩疊墊子，數量各為 \\( x \\) 塊與 \\( y \\) 塊。玩家 A，B 輪流進行行動。每回合可以選擇一疊非空的墊子，並移除 1 塊墊子，或是在這疊墊子的數量不少於另一疊的前提下，將這疊移除和另一疊數量一樣多的墊子(不能為0)。玩到最後，取走最後一塊墊子的即為勝者。若雙方都採取最佳化策略，那麼誰將獲勝。
- \\( x, y \leq 10^{18} \\)

題目的範圍達到 \\( 10^{18} \\) ，一一判斷每種狀態是否為 winning state 顯然不是一種實際的做法。但我們可以先試一些數字較小的例子，看能不能從中獲得一些啟發。

不妨先討論其中一疊的數量為 \\( 0 \\) 或 \\( 1 \\) 的情況。因為第二種行動取出的墊子數為兩疊中較少的數量。由此得知之後的回合最多只能取一個墊子。那麼判斷兩疊總和的奇偶，即能判斷誰獲勝。對於當前的玩家，是否數量總和為偶數則會落敗，而總和為奇數則會獲勝呢 ? 其實不然。兩疊數量為 {2, 2} 即為一個反例，玩家可以採取第二種行動，從其中一疊取出 2 塊，將數量變為 {2, 0}，使得對手落入 losing state。但是若總合為奇數，兩疊的數量一定是一奇一偶，那麼我們可以選擇從偶數疊中取出一塊，使數量變為 {奇數，奇數}。而不論對手採取甚麼行動，都一定是取出奇數塊的數量，也就會使得場上變為一奇一偶。於是我們可以將數量一奇一偶定義為 winning state，兩疊皆為奇數定義成 losing state。

剩下的只有兩疊皆為偶數的情況了。若當前玩家採取第一種行動，會取出奇數個墊子，使局勢變為一奇一偶，也就是 winning state。但這麼做無疑是一個錯誤的策略，因此雙方玩家只會採取第二種行動，也就是從數量較多的那疊取出較少那疊的數量。於是在雙方的策略都被確定的情況，我們可以開始探討給定兩疊的數量，誰會獲勝。若兩疊的數量為 \\( x, y\\) ，假設 \\( x \geq y \\) ，那麼在之後的 \\( x / y \\) 回合都會從數量為 \\( x \\) 那疊中取出墊子。而數量則會變為 \\( x\ mod\ y \\) 與 \\( y \\) 。接著再從數量為 \\( y \\) 的那疊開始取，以此類推。可以發現過程和求最大公因數時使用的輾轉相除法極為相似，因此若暴力將行動模擬完，複雜度也只會是 \\( log(max\lbrace x, y \rbrace) \\) 。而取到最後，會有其中一疊數量歸零，若另一疊數量還沒歸零，就可以用奇偶來判斷剩餘回合數。綜上所述，在數量為兩偶時，只要求出遊戲過幾個回合後會分出勝負，且該回合數若是奇數，則當前玩家勝利，反之則落敗。


<details><summary> Solution Code </summary>


```cpp
#include <bits/stdc++.h>
using namespace std;
using ll = long long;

void solve() {
    ll x, y; cin >> x >> y; 
    bool win = false;
    if (x % 2 + y % 2 == 1) win = true; // one odd, one even 
    else if (x % 2 + y % 2 == 2) win = false; // two odds
    else {
        ll cnt_round = 0;
        while (x > 0 && y > 0) {
            if (x < y) swap(x, y);
            cnt_round += x / y;
            x = x % y;
        }
        cnt_round += x + y;
        if (cnt_round % 2 == 1) win = true;
        else win = false;
    }
    if (win) cout << ">//<\n"; // first player wins
    else cout << ">\\\\<\n"; // second player wins
}

int main(){
    ios_base::sync_with_stdio(false);
    cin.tie(NULL);

    ll TestCases = 1;
    cin >> TestCases;
 
    while (TestCases--) {
        solve();
    }
}
    
```

</details>

> [The 2021 Zhejiang University City College Freshman Programming Contest I - If I Catch You](https://codeforces.com/gym/103488/problem/I)

> 給一個 \\( n * n \\) 的正方形棋盤方格，A, B 兩名玩家在正方形邊界的 \\( 4n - 4\ \\) 個格子上輪流進行行動。一開始 A 在左下角，B 在右上角。每個回合可以分成三個依序進行的步驟。首先，B 在目前的格子上放置一個陷阱，且該陷阱會永遠留在方格上。陷阱對 B 不造成任何影響，但 A 在每回合結束後不能停留在任何一個陷阱上(可以經過)。接著 B 以順時針方向走 \\( 2 \sim 3 \\) 格，最後 A 以順時針移動 \\( 1 \sim 4 \\) 格。遊戲結束的條件有以下兩種，且在回合中間也可以被觸發。第一種是 A 順時針方向的 \\( 1 \sim 4 \\) 格都有陷阱，則 B 獲勝。第二種則是 A 與 B 某個時間在同一格上，則 A 獲勝。問在雙方都採取最佳策略的情況下，誰將獲勝 ? 若 B 能獲勝，輸出 \\( -1 \\)，若 A 能獲勝，輸出 A 最少需要幾回合才能獲勝，( 同時 B 也盡全力拖延的情況) 。 
- \\( 1 \leq n \leq 10^5 \\)

可以注意到 A 獲勝的條件為**碰到** B，不一定要從後方趕上。那麼換個思路，若 A 每回合都以最少的步數前進，等待 B 自動追上他，是否也是個不錯的策略呢 ? 出乎意料地，這樣的策略是最佳的。以下會提出證明。

\\( n = 1 \\) 與 \\( n = 2 \\) 這兩個 corner case 因為情況較特殊，以下證明會撇除這兩種情況，後續再另外討論。

在遊戲剛開始時，雙方差距的格子數為 \\( 2n - 2 \\) 。若 A 每回合只前進一步，就算 B 每回合以最少的步數，也就是前進兩步，拖延他撞上 A 的時間。每回合雙方的距離仍會減一。在第 \\( 2n - 4 \\) 個回合結束後，雙方的差距格子數會減少至 \\( 2 \\) 。於是在第 \\( 2n - 3 \\) 回合時 B 就會碰到 A ，使遊戲結束。而陷阱的部分因為 A 總共只移動了 \\( 2n - 4 \\) 步，沒有走到 B 的起點，因此不用擔心會踩到陷阱。

這邊簡單說明一下為什麼 A 不會在選擇一個策略後，中途突然改變策略。雖然聽起來有點理所當然，但筆者認為仍需要提一下，使後續的證明更加順暢且完整。若 A 一開始選擇大步數追擊，而後續突然改變為上述守株待兔的策略，那麼兩者的距離差距比一開始來得更遠，因此該策略一定比上述的直接小步數等待還來得差。反之，若 A 一開始不論是因為甚麼原因，像是躲避陷阱，或是誘敵等，選擇小步數追擊，而之後突然改變策略變為大步數追擊。該策略也不會直接從一開始就直接選擇大步數追擊來得佳。原因為就算 A 有可能因為改變步數而成功躲避一些陷阱，但遇到陷阱的代價也只是該回合少移動一步，那麼直接以最大步數進行每一回合，一定不會比改變步數來得差。

接著分析 A 採取追擊的策略。首先 A 在不會踩到陷阱的前提下每回合都會前進 \\( 4 \\) 步。而 B 在看到 A 大步數前進，根據上方的說明，B 於是知道 A 採取的是追擊策略，因此除了第一步以外，B 的每個回合都會前進 \\( 3 \\) 步。因此要討論的只有 B 以 \\( 2 \\) 步開局或是以 \\( 3 \\) 步開局這兩種情況了。若 B 以 \\( 3 \\) 步開局，代表每個回合，A 與 B 的距離只會被拉近 \\( 1 \\)，而且因為每回合是由 B 先動，因此至少需要 \\( 2n - 2 \\) 個回合，A 才能獲勝，比守株待兔的策略來得差。而若 B 以 \\( 2 \\) 步開局，第一個回合結束後，雙方距離差 \\( 2n - 4 \\)。若雙方能在每回合拉近 \\( 1 \\) 格的距離，那麼 A 同樣也能在恰好第 \\( 2n - 3 \\) 回合結束遊戲。但因為 B 在起點以及距離起點 \\( 2 \\) 步的位置都有陷阱，因此 A 在每回合都前進 \\( 4 \\) 步的情況下，一定會被兩個陷阱的其中一個阻礙到，因此無法在\\( 2n - 3 \\) 個回合以內結束遊戲。由此得知，A 在採取守株待兔策略會最快把遊戲結束，需要 \\( 2n - 3 \\) 個回合。另外 \\( n = 1 \\) 因為沒有格子，因此 \\( 0 \\) 個回合便會結束比賽，而在 \\( n = 2 \\) 時，B 在第一回合就會撞上 A，導致 A 獲勝。

<details><summary> Solution Code </summary>


```cpp

#include <bits/stdc++.h>
using namespace std;

void solve() {
    int n; cin >> n;
    if (n == 1) cout << 0 << '\n';
    else if (n == 2) cout << 1 << '\n';
    else cout << 2 * n - 3 << '\n'; 
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

## Impartial Game

不偏賽局(impartial game)是賽局中一種常見的類型。
該賽局包含了以下性質:

- 是一種回合制的雙人遊戲
- 平等 - 雙方在遊戲中的唯一差別只有先後手
- 雙方都能看到完整的遊戲局勢
- 無隨機成分
- 在**有限**的回何數內會結束

對於一個不偏賽局，我們可以將其轉化成 Nim Game 的形式。因此下面會簡單介紹 Nim Game 的規則，性質以及一些延伸內容。

## Nim Game

### Rules

有 \\( N \\) 堆石頭，每堆有 \\( a_i \\) 個。玩家A，B輪流取石頭，每次可以選擇一堆，並從該堆取出任意數量的石頭(不能不取)。取得最後一塊石頭的玩家即為勝者。

### Strategy

將每堆的石頭個數進行 \\( bitwise\ XOR \\) 運算，得到的答案若為 \\( 0 \\) ，代表當前的局勢為 losing state，答案不為 \\( 0 \\) 則為 winning state。玩家若發現目前局勢為 winning state 時，保證會有一種取法，使得輪到對手時，所有堆的石頭個數 \\( XOR \\) 後為 \\( 0 \\) ，也就是 losing state。而按照該策略持續下去，不論對手如何應對，都無法挽回局勢。

### Proof

這邊提出一個數學歸納法的證明。為了方便後續說明，將 \\( XOR \\) 得出的結果命名為 \\( s \\) 。

#### Base Case 

若沒有任何石頭，則 \\( s\\) 為 \\( 0 \\)，玩家無法採取行動，為一個 losing state。

#### Inductive Case

若當前為 losing state，亦即 \\( s=0 \\) 。因為遊戲規定每回合都必須取石頭。若從有 \\( x \\) 塊的石頭堆取石頭，設取完剩 \\( y \\) 塊 \\( (x>y) \\) ，那麼取完後的 \\( XOR \\) 答案為 \\( (s \oplus x \oplus y) = (x \oplus y)\neq 0 \\) ，一定會使局勢變為 winning state。

若當前為 winning state，即 \\( s \neq 0 \\) 。考慮 \\( s \\) 的 most significant bit (假設從右到左為第 \\( k \\) 個bit)，那麼至少存在某一堆石頭的個數(設為 \\( x \\) )，且該 \\( x \\) 從右到左的第 \\( k \\) 個 bit 也同樣為 \\( 1 \\) (否則 \\( s \\) 的該bit就不會是 \\( 1 \\) )。那麼從該堆石頭中取石頭，取到剩下 \\( (s \oplus x ) \\) 個，那麼取完之後 \\( XOR \\) 的值就會變成 \\( (s \oplus x \oplus (s \oplus x)) = 0 \\)，是一個losing state。而該堆取完後剩 \\( (s \oplus x) \\) 個，該數的第 \\( k \\) 個bit變為 \\( 0 \\) ，由此得知 \\( (s \oplus x) < x \\) ，因此該取法沒有違反規則。

綜上所述以及數學歸納法，且遊戲在有限的回合數內會結束，因此該策略是正確的。

## Nim Game Extension

將原本的 Nim Game 規則修改成，玩家可以在自己的回合選擇其中一堆石頭並從該堆石頭**移除或增加任意數量**的石頭，那麼玩家的最佳化策略為何呢?
答案是策略會與原本的 Nim Game 完全一致。若對手選擇新增一些石頭，那麼該玩家可以在下一步將該數量的石頭移除，使得 game state 回到兩回合前。若遊戲會在有限回合數內結束，那麼該策略依舊是最佳的。

### Example

> [CSES 1099 - Stair Game](https://cses.fi/problemset/task/1099)
>
> 有 \\( N \\) 階的階梯，編號為第 \\( 1 \sim N \\) 階，每階上都放有非負整數顆球。玩家 \\( A\, B \\) 輪流採取行動，由 \\( A \\) 開始。每回合可以選擇一個整數 \\( x \\) \\( (x > 1) \\)，並從第 \\( x \\) 階取任意正整數顆球並移到第 \\( x - 1 \\) 階。到最後，沒有任何方法移動球的一方落敗。題目則問若雙方都採取最佳化策略時，誰會獲勝。
    
經由打表和一些觀察可以發現，若將**偶數階階梯**的球數進行 \\( XOR \\) 運算，若為 \\( 0 \\) 代表該 state 為 losing state，若不為 \\( 0 \\) 則為 winning state。
    
該如何驗證這個策略的正確性呢?這邊提供一個與 Nim Game 有關的想法。將偶數階階梯的球數視為 Nim Game 每堆的石頭數，且每回合的行動是可以任取一堆並增加該堆石頭數的。而對於一個偶數 \\( K\ (K  \geq  2) \\)，若當時第 \\( K + 1 \\) 階有 \\( x \\) 顆球，則代表第 \\( K \\) 階的石頭堆在當時的回合最多可以新增 \\( x \\) 塊石頭。而因為能新增的石頭數量是**有限**的，因此可以採用同上個章節"Nim Game Extension"的策略。
    
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
根據 Sprague-Grundy Theorem，一個不偏賽局可以被轉換成 Nim Game 的形式。對於一個狀態(局勢)，我們定義它的**Grundy Number**(SG value)為 \\( MEX \lbrace g_1, g_2, ..., g_n \rbrace \\) ，這裡的 \\( g_i \\) 代表當前的 state 可以轉移到某 \\( n \\) 個不同的 state，且 \\( g_1, g_2, ... g_n \\) 分別為那些 state 的 Grundy Number。若一個 state 的 Grundy Number 為0，代表當前為 losing state。若不為0，則為 winning state。

另外，若遊戲方式為好幾個子遊戲同時進行。我們可以採用與 Nim Game 相似的作法，將若干個子遊戲的 Grundy Number 使用 \\( XOR \\) 來進行合併，得出整個遊戲的狀態。

定理的證明可以利用數學歸納法與 \\( XOR \\) 的性質。但這裡提供一個直觀的想法供讀者參考:
Nim Game 每回合的行動可以從數量為 \\( x(x>0) \\) 的石頭堆取出任意數量，在取後的剩餘石頭數有可能為 \\( 0 \sim x-1 \\) 的任意一個。相似地，Grundy Number 的性質確保，一個 SG value 為 x 的 state，對於任意一個 \\( i\ (x-1\ge i\ge0, \ i \in Z) \\) ，都至少存在一種轉移方法，使玩家在一回合內轉移到SG value為 \\( i \\) 的 state。與 Nim Game 不同之處在於，某些不偏賽局在轉移後，有可能會使 SG value 變大。那麼這時可以用"Nim Game Extension"的策略，把對手上一回合的行動消除，即可維持在 winning state。

### Example
    
> [CSES 1098 - Nim Game II](https://cses.fi/problemset/task/1098)
>
> 有 \\( N \\) 堆石頭。兩名玩家輪流採取行動，每回合可以選擇其中一堆石頭並移除 \\( 1 \sim 3 \\) 個石頭。若玩家無法移除則落敗。題目問若雙方都採取最佳化策略的話，誰將獲勝。

某堆的石頭數若有 \\( x \\) 個，經過一次行動後所剩的石頭數可能為 \\( x - 1 \\) , \\( x - 2 \\), 或 \\( x - 3 \\) 個。可以很容易地推論出 Grundy Number 為四個一循環，且該堆的 Grundy Number 為 \\( x\ mod\ 4 \\)。而根據上述的定理，可以用 \\( XOR \\) 將各堆的 Grundy Number 進行合併，得到的數字即可判斷勝敗。

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

> [Codeforces Global Round 22 C - Even Number Addicts](https://codeforces.com/contest/1738/problem/C)
>
> 給一個長度為 \\( n \\) 的整數陣列。玩家A，B輪流採取行動，由A開始。玩家每回合可以將一個數字從陣列中移除。當所有數字都被移除後遊戲結束。若A移除的數字和為偶數，那麼A獲勝。反之，B獲勝。題目則問若雙方都採取最佳化策略，那麼誰會獲勝。

<details><summary>Solution</summary>

可以發現陣列中數字大小其實不重要，重要的是奇數偶數的數量各為多少。那麼我們可以用陣列中剩餘奇數個數，剩餘偶數個數，與目前A移除數字總和的奇偶性這三項資訊來代表一個 state。

填表的過程可以使用 Top-down DP 並將 base case 處理好，即可判斷狀態是誰的 winning state。

</details>

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


> [Atcoder Beginner Contest 206 F - Interval Game 2](https://atcoder.jp/contests/abc206/tasks/abc206_f)
>
> 給 \\( N \\) 個左閉右開的區間，每個區間的範圍是 \\( [L_i  , R_i ) \\)。玩家 \\( A \\) 與 \\( B \\) 輪流選擇區間，每次可以選擇 \\( N \\) 個區間的其中一個，但不能與先前兩人選擇的區間重疊。若由 \\( A \\) 開始，且雙方都採取最佳化策略，那麼問誰將獲勝。   
- \\( 1 \leq N, L_i, R_i \leq 100 \\)

<details><summary>Solution</summary>

第一眼看到題目可能會覺得狀態不太容易定義。但再想想會發現若把遊戲狀態定義成目前可選擇的範圍是 \\( x \sim y \\)。那麼取完一個區間後，會得到另外兩個可以選的小區間，也就是兩個同類型的子遊戲。這與前面 Grundy Number 章節的概念頗為相似。考慮將狀態定義為 \\( dp[x][y] \\)，代表當現在可選的範圍是 \\( x \sim y \\) 時，狀態的 Grundy Number 會是多少。那麼採用 top-down DP 的方式來填表，就能得到整個遊戲的 SG-value， 也就能得到最後的答案。    

正確性的部分已經沒問題了，那麼時間複雜度呢 ? 狀態總共有 \\( max \lbrace R_i \rbrace ^2  \leq 10^4 \\) 個。每次轉移需要將 \\( N \\) 個可選的區間枚舉一遍，並利用 \\( MEX \\) 函數求出該狀態的 SG-value。總共需要 \\( O(N) \\) 的時間。\\( MEX \\) 函數也只要花 \\( O(N) \\) 是因為每個狀態最多只會轉移到 \\( N \\) 個不同的狀態，由此得知一個狀態的 SG-value 不會大於 \\( N \\)，求值時只要從 \\( 0 \\) 到 \\( N \\) 檢查哪個最小的非負整數不會被轉移到，即為該狀態的 SG-value。綜上所述，整個演算法只需要約 \\( 10^6 \\) 個運算即可完成，能夠在時限內跑完。

</details>

<details><summary> Solution Code </summary>

- Time Complexity: \\( O(max\lbrace R_i \rbrace ^2 * N) \\)

```cpp

#include <bits/stdc++.h>
using namespace std;

int maxn = 110;

void solve() {
    vector<vector<int>> sg(maxn, vector<int>(maxn, -1));
    int N; cin >> N;
    vector<pair<int, int>> a;
    for (int i = 0; i < N; i++) {
        int l, r; cin >> l >> r;
        r--;
        a.emplace_back(l, r);
    }

    auto dfs = [&](auto self, int l, int r) -> int {
        if (sg[l][r] != -1) return sg[l][r];
        vector<bool> vis(maxn);
        for (auto [ql, qr]: a) {
            if (ql >= l && qr <= r) {
                int x = self(self, l, ql - 1) ^ self(self, qr + 1, r);
                vis[x] = 1;
            }
        }
        for (int i = 0; i < maxn; i++) {
            if (!vis[i]) {
                return sg[l][r] = i;
            }
        }
    };

    int val = dfs(dfs, 0, maxn - 1);
    cout << (val == 0 ? "Bob\n" : "Alice\n");
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
    
> [Atcoder Beginner Contest 297 G - Constrained Nim 2](https://atcoder.jp/contests/abc297/tasks/abc297_g)
>
> 給 \\( N,L,R \\) 三個整數。玩家 \\( A, B \\) 玩Nim Game ( \\( N \\) 堆石頭，每堆有 \\( 1 \sim 10^9 \\) 個)，但每回合只能從其中一堆取 \\( L \sim R \\) 個石頭。問若雙方都採取最佳化策略，誰將獲勝。
- \\( N \leq 2*10^5 \\)
- \\( 1 \leq L \leq R \leq 10^9 \\)

<details><summary>Solution</summary>
    
打表找規律後可以發現每堆的 SG-value 為 \\( (x\ mod\ (L + R)) / L \\) ， \\( x \\) 為該堆石頭個數。而根據上述的定理，可以由 \\( XOR \\) 進行合併，得到最後的 SG-value。
    
證明的話可以使用數學歸納法。
分成:
    
- \\( 0 \leq x < L + R \\)。
在滿足上述不等式的前提下，這個case又可以分成 \\( 0 \leq x < L ,\ \ \  L \leq x < 2L,\ \ \ 2L \leq x < 3L... \\)。
- \\( L + R \leq x < 2(L + R) \\)。
                        而這個case可以分成 \\( L+R \leq x < L+R+L, \ \ \ L+R+L \leq x < L+R+2L... \\)。
- 接著再討論 \\( x \geq 2(L+R) \\) 的case。

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

> [Codeforces Round 868 E - Removing Graph](https://codeforces.com/contest/1823/problem/E)
>
> 給一張由數個環(每個點的 degree 皆為2)組成的圖。玩家 \\( A \\) , \\( B \\) 輪流行動。已知 \\( l, r \\) 的值，每回合可以移除 \\( l \sim r \\) 個連通的點(相鄰的邊一同移除)。若輪到某玩家且該玩家沒有合法的移除方式，則玩家落敗。問若雙方都採取最佳化策略，那麼誰將獲勝。
- \\( 1 \leq l < r \leq 2*10^5 \\)

<details><summary>Solution</summary>

圖的每個環都是獨立的。若能求出一個環的 SG value，那麼整個遊戲的 SG value 就是所有環的 SG value \\( XOR \\) 出來的結果。

經由一些觀察後發現，從一個環移除一些點後，會形成一條鏈。一條鏈在移除一些點後，會形成 \\( 1 \sim 2 \\) 條鏈。而一個狀態的 SG value 等同於 \\( MEX \lbrace 轉移到的\ state\ 的\ SG\ value  \rbrace \\) 。方便後續的說明，令 \\( cycle[x] \\) 為 \\( x \\) 個點構成的環的 SG value，\\( chain[x] \\) 為 \\( x \\) 個點構成的鏈的 SG value。那麼綜合以上的資訊可以列出以下方程式 
- \\( cycle[x] = MEX \lbrace chain[x-r], chain[x-r+1], ... chain[x-l] \rbrace \\)
- \\( chain[x] = MEX \lbrace chain[a] \oplus chain[b],\ \forall\ a, b \geq 0,\ x-r \leq a+b \leq x-l \rbrace \\)

根據題目的範圍，若以常規的方式求 \\( cycle[x], chain[x] \\) 是會超時的。但藉由打表觀察或是一些證明可以得出一些性質:
- \\( chain[x] = 0, \ \ 0 \leq x \leq l-1 \\)
- \\( chain[x] = x / l, \ \ 0 \leq x \leq r + l - 1 \\)
- \\( cycle[x] = chain[x], \ \ 0 \leq x \leq r + l - 1 \\)
- \\( cycle[x] = 0, \ \ x \geq l + r \\)

由以上的性質，我們只要求出每個環的節點數，並把他們的 \\( SG \ value \\) 合併，就能得出答案。

證明的部分就請有興趣的讀者自行練習。

</details>

<details><summary> Solution Code </summary>

- Time complexity: \\( O(N) \\)
    
```cpp
#include <bits/stdc++.h>
using namespace std;
 
void solve() {
    int n, l, r; cin >> n >> l >> r;
    vector<vector<int>> adj(n);
    vector<bool> vis(n);
    for (int i = 0; i < n; i++) {
        int u, v; cin >> u >> v;
        u--, v--;
        adj[u].emplace_back(v);
        adj[v].emplace_back(u);
    }  
    int state = 0, cnt = 0;
    auto dfs = [&](auto self, int u) -> void {
        vis[u] = 1;
        cnt++;
        for (auto v: adj[u]) {
            if (!vis[v]) self(self, v);
        }
    };
    for (int i = 0; i < n; i++) {
        if (!vis[i]) {
            cnt = 0;
            dfs(dfs, i);
            if (cnt <= r + l - 1) state ^= cnt / l;
        }
    }
    cout << (state == 0 ? "Bob" : "Alice") << '\n';
}
 
int main() {
    ios_base::sync_with_stdio(false);
    cin.tie(NULL);
 
    solve(); 
}
    
```


    
</details>



## Summary

在解決一個賽局相關的題目時，步驟可以總結為以下幾點
1. 將遊戲局勢轉換成狀態 (或是某個數值 ex: SG value)
2. 理解各個狀態之間如何轉移
3. 判斷獲勝或落敗條件

而在題目範圍太大或是想不到解法時，打表找規律也是一個不錯的辦法。

## References

- [The Intuition Behind NIM and Grundy Numbers in Combinatorial Game Theory](https://codeforces.com/blog/entry/66040)
- [Sprague-Grundy theorem. Nim - Algorithms for Competitive Programming](https://cp-algorithms.com/game_theory/sprague-grundy-nim.html)
- [公平组合游戏 - OI wiki](https://oi-wiki.org/math/game-theory/impartial-game/)
- [Sprague-Grundy定理是怎么想出来的](https://zhuanlan.zhihu.com/p/20611132)
