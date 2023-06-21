# Game Theory

## Introduction

賽局理論考慮玩家在遊戲中的行為，並研究他們的最佳化策略。

為了更好理解，我們直接看一個例題。

> 考慮一個遊戲。有 \\( 100 \\) 顆石頭，兩人輪流取走石頭，每次可以取 \\( 1 \sim 7 \\) 顆，取走最後一顆石頭的獲勝。請問誰有必勝的策略。

相信各位讀者可以很快地找出一個策略。那就是先手的玩家必勝，每次取石頭時將剩餘數量為持在 \\( 8 \\) 的倍數。雙方玩家取到最後，先手的玩家必定會取得最後一塊石頭。

簡單地說明一下該策略的正確性。一個數字除以 \\( 8 \\) 的餘數最高為 \\( 7 \\)。因此必定存在一種取法，使得一個數量不能被 \\( 8 \\) 整除的石頭堆，在取完後剩餘數量變為 \\( 8 \\) 的倍數。那麼接下來只要說明為什麼輪到先手的玩家時，剩餘的數量一定不會是 \\( 8 \\) 的倍數，就能完成證明。

一開始石頭數量為 \\( 100 \\)，不是 \\( 8 \\) 的倍數。接下來先手的玩家會取石頭，使其剩下的數字為 \\( 8 \\) 的倍數。輪到對手時，因為一次只能取 \\( 1 \sim 7 \\) 顆，不論如何選擇，取的數量都不會被 \\( 8 \\) 整除，因此對手取完後，剩下的數量一定不被 \\( 8 \\) 整除。而因為石頭是有限的，雙方在進行若干個回合後，先手的玩家取完後剩餘石頭為 \\( 0 \\)，而 \\( 0 \\) 也能被 \\( 8 \\) 整除，與上述的論述相符。

## Game States

接下來會定義何謂 **winning state** 與 **losing state**。利用定義好的狀態，我們可以判斷一個玩家在面對某個局勢時會不會獲勝。

若局勢為 winning state，代表當前的玩家可以採用一個最佳化的策略，使得對手不論採取甚麼行動，都無法阻止該玩家獲勝。反之，若玩家在某狀態，不論採取何種行動，都無法阻止對手以某個策略取勝的話，就稱為 losing state。換句話說，winning state 至少存在一種行動，使得遊戲局勢落入 losing state。而 losing state 不論採取什麼行動，都會使局勢落入 winning state。

對於 Introduction 章節中提到的取石頭問題，我們其實可以將每個局勢的 game state 定義出來，那麼有沒有必勝策略這件事也就顯而易見了。

首先，當剩下的石頭數為 \\( 0 \\) 時，玩家將自動落敗，為一個 losing state。石頭數量為 \\( 1 \sim 7 \\) 時，為 winning state。因為玩家可以找到一個取法，使得行動結束後局勢變為 losing state。數量為 \\( 8 \\) 時，玩家不論在該回合取幾個石頭，都會轉移到 winning state，因此當下是一個 losing state。按照從小到大的順序填表，便能將所有局勢的狀態定義出來。於是我們發現當石頭數為 \\( 8 \\) 的倍數時，狀態為 losing state，反之則為 winning state。

另外，在填表時要注意該狀態會轉移到的其他狀態都是已經定義過的，否則會發生錯誤。因此若不確定狀態轉移的順序或是較難判斷時，可以採用 Top-down DP 的方式填表，較為保險。

### Example

> [CSES 1729 - Stick Game](https://cses.fi/problemset/task/1729)
>
> 考慮一個遊戲，規則為雙方輪流從一堆棍子中移除若干個棍子，若輪到某玩家且他沒有任何合法的方法可以移除棍子時，則對方獲勝。已知整數 \\( n \\)，長度為 \\( k \\) 的整數陣列 \\( p \\)。\\( p_i \\) 為玩家在每回合可以選擇移除的個數。試問對於所有的 \\( x\ (1 \leq x \leq n) \\)，在目前還有 \\( x \\) 個棍子的情況下，且雙方都採取最佳化策略，那麼目前的玩家會獲勝還是落敗。

- \\( 1 \leq n \leq 10^6 \\)
- \\( 1 \leq k \leq 100 \\)
- \\( p_i \leq n \\)

這題與前面章節的取石頭題目類似。若剩下 \\( 0 \\) 個棍子，玩家必定無法轉移到其他狀態，相當於落敗，為一個 losing state。

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

## Exercises

接下來會提供一些有關賽局的題目與解法。這些題目比較沒有固定的解題模式，也不需要特定的定理。但解這些題目往往需要一些觀察力與創新的思維。相信讀者在練習完這些題目後，可以更有效地找到類似問題的切入點。

> [Codeforces Global Round 22 C - Even Number Addicts](https://codeforces.com/contest/1738/problem/C)
>
> 給一個長度為 \\( n \\) 的整數陣列。玩家 A，B 輪流採取行動，由 A 開始。玩家每回合可以將一個數字從陣列中移除。當所有數字都被移除後遊戲結束。若 A 移除的數字和為偶數，那麼 A 獲勝。反之，B 獲勝。題目則問若雙方都採取最佳化策略，那麼誰會獲勝。

<details><summary>Solution</summary>

可以發現陣列中數字大小其實不重要，重要的是奇數偶數的數量各為多少。那麼我們可以用陣列中剩餘奇數個數，剩餘偶數個數，與目前 A 移除數字總和的奇偶性這三項資訊來代表一個 state。

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

> [ZEROJUDGE b902 - 肉墊遊戲](https://zerojudge.tw/ShowProblem?problemid=b902)
>
> 有兩疊墊子，數量各為 \\( x \\) 塊與 \\( y \\) 塊。玩家 A，B 輪流進行行動。每回合可以選擇一疊非空的墊子，並移除 1 塊墊子，或是在這疊墊子的數量不少於另一疊的前提下，將這疊移除和另一疊數量一樣多的墊子(不能為 0)。玩到最後，取走最後一塊墊子的即為勝者。若雙方都採取最佳化策略，那麼誰將獲勝。

- \\( x, y \leq 10^{18} \\)

<details><summary>Solution</summary>

題目的範圍達到 \\( 10^{18} \\)，一一判斷每種狀態是否為 winning state 顯然不是一種實際的做法。但我們可以先試一些數字較小的例子，看能不能從中獲得一些啟發。

不妨先討論其中一疊的數量為 \\( 0 \\) 或 \\( 1 \\) 的情況。因為第二種行動取出的墊子數為兩疊中較少的數量。由此得知之後的回合最多只能取一個墊子。那麼判斷兩疊總和的奇偶，即能判斷誰獲勝。對於當前的玩家，是否數量總和為偶數則會落敗，而總和為奇數則會獲勝呢 ? 其實不然。兩疊數量為 {2, 2} 即為一個反例，玩家可以採取第二種行動，從其中一疊取出 2 塊，將數量變為 {2, 0}，使得對手落入 losing state。但是若總合為奇數，兩疊的數量一定是一奇一偶，那麼我們可以選擇從偶數疊中取出一塊，使數量變為 {奇數，奇數}。而不論對手採取甚麼行動，都一定是取出奇數塊的數量，也就會使得場上變為一奇一偶。於是我們可以將數量一奇一偶定義為 winning state，兩疊皆為奇數定義成 losing state。

剩下的只有兩疊皆為偶數的情況了。若當前玩家採取第一種行動，會取出奇數個墊子，使局勢變為一奇一偶，也就是 winning state。但這麼做無疑是一個錯誤的策略，因此雙方玩家只會採取第二種行動，也就是從數量較多的那疊取出較少那疊的數量。於是在雙方的策略都被確定的情況，我們可以開始探討給定兩疊的數量，誰會獲勝。若兩疊的數量為 \\( x, y\\)，假設 \\( x \geq y \\)，那麼在之後的 \\( x / y \\) 回合都會從數量為 \\( x \\) 那疊中取出墊子。而數量則會變為 \\( x\ mod\ y \\) 與 \\( y \\)。接著再從數量為 \\( y \\) 的那疊開始取，以此類推。可以發現過程和求最大公因數時使用的輾轉相除法極為相似，因此若暴力將行動模擬完，複雜度也只會是 \\( log(max\lbrace x, y \rbrace) \\)。而取到最後，會有其中一疊數量歸零，若另一疊數量還沒歸零，就可以用奇偶來判斷剩餘回合數。綜上所述，在數量為兩偶時，只要求出遊戲過幾個回合後會分出勝負，且該回合數若是奇數，則當前玩家勝利，反之則落敗。

</details>

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
>
> 給一個 \\( n * n \\) 的正方形棋盤方格，A, B 兩名玩家在正方形邊界的 \\( 4n - 4\ \\) 個格子上輪流進行行動。一開始 A 在左下角，B 在右上角。每個回合可以分成三個依序進行的步驟。首先，B 在目前的格子上放置一個陷阱，且該陷阱會永遠留在方格上。陷阱對 B 不造成任何影響，但 A 在每回合結束後不能停留在任何一個陷阱上(可以經過)。接著 B 以順時針方向走 \\( 2 \sim 3 \\) 格，最後 A 以順時針移動 \\( 1 \sim 4 \\) 格。遊戲結束的條件有以下兩種，且在回合中間也可以被觸發。第一種是 A 順時針方向的 \\( 1 \sim 4 \\) 格都有陷阱，則 B 獲勝。第二種則是 A 與 B 某個時間在同一格上，則 A 獲勝。問在雙方都採取最佳策略的情況下，誰將獲勝 ? 若 B 能獲勝，輸出 \\( -1 \\)，若 A 能獲勝，輸出 A 最少需要幾回合才能獲勝，( 同時 B 也盡全力拖延的情況)。

- \\( 1 \leq n \leq 10^5 \\)

<details><summary>Solution</summary>

可以注意到 A 獲勝的條件為**碰到** B，不一定要從後方趕上。那麼換個思路，若 A 每回合都以最少的步數前進，等待 B 自動追上他，是否也是個不錯的策略呢 ? 出乎意料地，這樣的策略是最佳的。以下會提出證明。

\\( n = 1 \\) 與 \\( n = 2 \\) 這兩個 corner case 因為情況較特殊，以下證明會撇除這兩種情況，後續再另外討論。

在遊戲剛開始時，雙方差距的格子數為 \\( 2n - 2 \\)。若 A 每回合只前進一步，就算 B 每回合以最少的步數，也就是前進兩步，拖延他撞上 A 的時間。每回合雙方的距離仍會減一。在第 \\( 2n - 4 \\) 個回合結束後，雙方的差距格子數會減少至 \\( 2 \\)。於是在第 \\( 2n - 3 \\) 回合時 B 就會碰到 A，使遊戲結束。而陷阱的部分因為 A 總共只移動了 \\( 2n - 4 \\) 步，沒有走到 B 的起點，因此不用擔心會踩到陷阱。

這邊簡單說明一下為什麼 A 不會在選擇一個策略後，中途突然改變策略。雖然聽起來有點理所當然，但筆者認為仍需要提一下，使後續的證明更加順暢且完整。若 A 一開始選擇大步數追擊，而後續突然改變為上述守株待兔的策略，那麼兩者的距離差距比一開始來得更遠，因此該策略一定比上述的直接小步數等待還來得差。反之，若 A 一開始不論是因為甚麼原因，像是躲避陷阱，或是誘敵等，選擇小步數追擊，而之後突然改變策略變為大步數追擊。該策略也不會直接從一開始就直接選擇大步數追擊來得佳。原因為就算 A 有可能因為改變步數而成功躲避一些陷阱，但遇到陷阱的代價也只是該回合少移動一步，那麼直接以最大步數進行每一回合，一定不會比改變步數來得差。

接著分析 A 採取追擊的策略。首先 A 在不會踩到陷阱的前提下每回合都會前進 \\( 4 \\) 步。而 B 在看到 A 大步數前進，根據上方的說明，B 於是知道 A 採取的是追擊策略，因此除了第一步以外，B 的每個回合都會前進 \\( 3 \\) 步。因此要討論的只有 B 以 \\( 2 \\) 步開局或是以 \\( 3 \\) 步開局這兩種情況了。若 B 以 \\( 3 \\) 步開局，代表每個回合，A 與 B 的距離只會被拉近 \\( 1 \\)，而且因為每回合是由 B 先動，因此至少需要 \\( 2n - 2 \\) 個回合，A 才能獲勝，比守株待兔的策略來得差。而若 B 以 \\( 2 \\) 步開局，第一個回合結束後，雙方距離差 \\( 2n - 4 \\)。若雙方能在每回合拉近 \\( 1 \\) 格的距離，那麼 A 同樣也能在恰好第 \\( 2n - 3 \\) 回合結束遊戲。但因為 B 在起點以及距離起點 \\( 2 \\) 步的位置都有陷阱，因此 A 在每回合都前進 \\( 4 \\) 步的情況下，一定會被兩個陷阱的其中一個阻礙到，因此無法在\\( 2n - 3 \\) 個回合以內結束遊戲。由此得知，A 在採取守株待兔策略會最快把遊戲結束，需要 \\( 2n - 3 \\) 個回合。另外 \\( n = 1 \\) 因為沒有格子，因此 \\( 0 \\) 個回合便會結束比賽，而在 \\( n = 2 \\) 時，B 在第一回合就會撞上 A，導致 A 獲勝。

</details>

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
