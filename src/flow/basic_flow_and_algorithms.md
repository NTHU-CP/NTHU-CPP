
# 最大流基本定理證明與 Ford-Fulkerson and Edmonds-Karp

## 網路流及用語介紹

* 什麼是網路流?
  * 以圖論的角度來說，網路流是一張有向圖，每一條邊都有一個容量代表可流過的上限值，可以將邊想像成水管，流過的值即為水的即時流量。
  * 整張圖會有一個源點以及匯點，源點能有無限制的水量供應，並且所有點滿足水流入量等於水流出量，最終目的則是求出從源點最多能流多少水到匯點。 
  * 許多和分配、運輸量、最小花費的題目都可以被轉換成網路流問題，而看出它們可被轉換、以及轉換的過程往往是網路流問題最困難的部分。
* 網路流用語介紹
  * 源點(Source)：擁有無限水量可供應的點，記為 \\(s\\)。
  * 匯點(Sink)：可接收無限水量的點，記為 \\(t\\)。
  * 弧：網路流圖上的一條帶權邊 \\((u, v) \in E\\) 
  * 淨流：通過一條弧 \\((u, v)\\) 的流量(淨流)，記為 \\(f(u, v)\\)。
  * 容量(Capacity)：能通過一條弧 \\((u, v)\\) 的最大淨流，同時也是邊 \\((u, v)\\) 的權重，記為 \\(c(u, v)\\)。
  * 網路流圖：
    * 洽有一個源點。
    * 洽有一個匯點。
    * 為一個點和邊的集合，可記為 \\(G(V,E)\\)。
    * \\(\forall u, v \in G(V,E), c(u, v) \geq 0\\) 
  * 網路流(Network flow)：一個流量集合 \\(F=\{f(u, v)\}\\)。
  * 可行流：滿足以下條件的網路流稱為可行流
    * 容量限制：\\(0 \leq f(u, v) \leq c(u, v)\\)。
    * 流量守恆：除了源點和匯點，其他點都必須滿足流入量=流出量。
  * 最大流：所有可行流當中流量最大的即為最大流。
  * 剩餘容量(Residual Capacity)：記為 \\(c_{f}(u, v) = c(u, v) - f(u, v)\\) 
  
  * 剩餘網路(Residual Network)：為由所有弧、其反向邊、不存在的邊，以及所有原本的點所組成的網路流圖，以定義來看的話，有以下關係。
  
    * \\(c_f(u, v) = c(u, v) - f(u, v)\ \ \ u, v \in G\\ \\)
    * \\(c_f(v, u) = f(u, v) \qquad \qquad \quad u, v \in G\\ \\)
    * \\(c_f = 0 \qquad \qquad \qquad \ \ otherwise\\)

  * 割(Cut)：
    * 割將網路流圖切分成兩集合 \\(S, T\\)，其中 \\(S \cap T = \emptyset\\) 且 \\(S \cup T = G\\)，其中 \\(s \in S\\)，\\(t \in T\\)。 
    * 割的容量為所有在集合 \\(S\\) 中的點連向集合 \\(T\\) 中的點的弧容量總和。\\(c(S, T) = \sum_{u \in S} \sum_{v \in T} c(u, v)\\) 
  * 最小割(Minimum cut)：所有割裡面容量最小的為最小割。
* 網路流的題目舉例
  * 給定 \\(N\\) 個男生和 \\(M\\) 個女生，以及 \\(K\\) 個關係 \\((u, v)\\)，每個關係代表 \\(u\\) 和 \\(v\\) 互相喜歡，且不會出現同性戀的情形，請求出最多的情侶關係組數。
  * 有一座城市有 \\(N\\) 棟建築物及 \\(M\\) 個道路，每條道路為雙向連接兩棟建築物，且有其長度，請求出最少需要關閉幾條道路，才能使建築物 \\(1\\) 和建築物 \\(N\\) 之間無法往來。

## 最大流最小割定理(Max Flow Min Cut Theorem)

  * 首先，對於一個網路 \\(G\\) 來說，有以下三項條件等價。
    * 1.\\(f\\) 是一個 \\(s-t\\) 最大流。
    * 2.對於流量為 \\(f\\) 的剩餘網路 \\(G_f\\) 中沒有 \\(s\\) 到 \\(t\\) 的增廣路徑。
    * 3.存在一最小割 \\((S, T)\\)，其容量等於 \\(f\\)，亦即\\( |f|=|c(S, T)| \\)。
    
  * 接下來將會證明這三項條件會同時正確。
    * 條件(1) \\(\implies\\) 條件(2)：
      * 以反證法證明，假設 (1) 正確且 (2) 不正確，代表 \\(s,t\\)之間存在增廣路徑，那麼就可以透過其增加更多流量，和條件 (1) 矛盾，故成立。
    
    * 條件(2) \\(\implies\\) 條件(3)：
      * 設集合 \\(S, T\\) 分別為剩餘網路上 \\(s\\) 能到達的點集合，以及 \\(T=V\\) \ \\(S\\)，\\((S, T)\\) 為 \\(s-t\\) 割，因為已無增廣路徑，所以 \\(S\\) 中的點流到 \\(T\\) 中的點的邊全都達到飽和狀態，且 \\(T\\) 中的點流到 \\(S\\) 中的點的邊流量全都為 \\(0\\)，\\(\forall x \in S, \ \forall y \in T \\)，\\(f(x, y) = c(x, y) , \ \ f(y, x) = 0 \\)，故 \\( \sum_{x \in S,\ y \in T} \ f(x,\ y)= \sum_{x \in S, \ y \in T} \ c(x, y)= |c(S, T)| = |f| \\)
    
    * 條件(3) \\(\implies\\) 條件(1)：
      * 對於一個合法流 \\(f'\\) 來說，其流經的所有邊流量都不超過其容量，故 \\(|f'| \leq c(S, T)\\)，又因 \\(c(S, T) = |f|\\)，故 \\(|f|\\) 為 \\(s-t\\) 最大流。

## Ford-Fulkerson 算法

  * 首先先介紹一個最大流演算法的核心概念之一，增廣路徑(Augmenting Path)
    * 一個路徑 \\(\{u_{1}, u_{2}, ..., u_{n}\}\\)，其中 \\(u_{1}=s\\)，\\(u_{n}=t\\)，\\(c_{f}(u_{i}, u_{i + 1}) > 0\\)。
    * 簡單來說，就是從源點走到匯點的路徑中，若該路徑上所有弧的剩餘容量都大於 0，則該路徑稱作增廣路徑。
    * 以功能來說，增廣路徑的存在，就表示從源點開始還有可流通至匯點的容量可使用。
    * 當圖上達到最大流時，不存在增廣路徑。
    
  * 先介紹一個直覺找最大流的作法
    * 不斷使用 DFS 尋找增廣路徑
      * 若找到增廣路徑，且增廣路徑上所有弧的剩餘容量中的最小值為 \\(c_f\\)，則將增廣路徑上所有弧的剩餘容量減去 \\(c_f\\)，且將所有弧的反向邊的剩餘容量加上 \\(c_f\\)。
      * 將總流量增加 \\(c_f\\)，並持續尋找增廣路徑，當找不到增廣路徑時，演算法結束，此時總流量即為這個圖上的最大流。
    * 以下圖為例。
    
        <details>
        <summary> 點擊展開圖片 </summary>
    
        ![](images/FF_image_1.jpg)

        </details>
    
    * 第一次找到的增廣路徑為：{ \\(S \to A\\), \\(A \to B\\), \\(B \to t\\) }，並更新路徑上所有弧。
    
        <details>
        <summary> 點擊展開圖片 </summary>

        ![](images/FF_image_2.jpg)

        </details>
    
    * 第二次找到的增廣路徑為：{ \\(S \to A\\), \\(A \to t\\) }，並更新路徑上所有弧。
    
        <details>
        <summary> 點擊展開圖片 </summary>

        ![](images/FF_image_3.jpg)

        </details>
    
    * 此時，圖上已無增廣路徑，總流量為 \\(2\\)，圖上即為最大流……嗎？事實上，此圖的最大流為 \\(3\\)，也就是說，我們找到的增廣路徑組合出錯了，以下是正確的增廣路徑組合之一。
    
    * 第一次找到的增廣路徑為：{ \\(S \to A\\), \\(A \to t\\) }，並更新路徑上所有弧。
    
        <details>
        <summary> 點擊展開圖片 </summary>

        ![](images/FF_image_4.jpg)

        </details>

    * 第二次找到的增廣路徑為：{ \\(S \to B\\), \\(B \to t\\) }，並更新路徑上所有弧。
    
        <details>
        <summary> 點擊展開圖片 </summary>

        ![](images/FF_image_5.jpg)

        </details>
        
    * 如果將上面兩個最後完成的圖拿出來對比，可以發現若在圖一最後的圖上，以路徑 { \\(S \to B\\), \\(B \to A\\), \\(A \to t\\) } 再更新一次流(其中弧 \\(A \to B\\) 增加的流量變成負的)，可以發現就能得到圖二的最終結果。
    
        <details>
        <summary> 最大流為2的圖一 </summary>

        ![](images/FF_image_6.jpg)

        </details>
        
        <details>
        <summary> 最大流為3的圖二 </summary>

        ![](images/FF_image_7.jpg)

        </details>
    
    * 為甚麼會發生這種情況呢？其實是因為找到增廣路徑時，對每一條弧來說，最佳解不一定是將流量調整增加路徑上最小的剩餘容量 \\(c_f\\)，所以需要一個除錯的機制，而這個機制正是為所有弧建立「反向邊」，讓當前流量並非最佳的弧有逆推回正確流量的可能。
      * 反向邊：對於一條弧 \\(u \to v\\) 來說，設其剩餘容量為\\(c_f\\)，其反向邊為 \\(v \to u\\) 且剩餘容量為 \\(c - c_f\\)。
    
  * 演算法過程
    * 為所有弧建立反向邊，若弧的容量為 \\(c\\)，則反向邊的容量為 \\(0\\)。
    * 進行前文所提到的「不斷使用 DFS 尋找增廣路徑」中的動作。
    
  * 時間複雜度及正確性分析
    * 尋找一條增廣路徑複雜度為 \\(O(V+E)\\)，最差的情況之下，每次增廣路徑增加的流量為 \\(1\\)，總共要找 \\(F\\) 次增廣路徑( \\(F\\) 為圖上最大的最大流)，演算法總複雜度為 \\(O((V+E)F)\\)，邊數通常較多，可簡化為 \\(O(EF)\\)，請看以下例子。
      <details>
        
        * 以下為理論上存在的最壞情況，圖片中邊的數值為剩餘容量。
    
        ![](images/FF_image_time_1.jpg)
        
        * 第一次找到的增廣路徑為 {\\(s \to A, \ A \to B, \ B \to t\\)}
    
        ![](images/FF_image_time_2.jpg)
    
        * 第二次找到的增廣路徑為 {\\(s \to B, \ B \to A, \ A \to t\\)}
    
        ![](images/FF_image_time_3.jpg)
        
        * 不斷重複同樣動作，直到出現最大流，此時共進行了 1998 次的尋找增廣路徑。
    
        ![](images/FF_image_time_4.jpg)

      </details> 
    * 根據最大流最小割定理中的條件(1)、(2)，只要在 \\(s, t\\) 之間還存在增廣路徑，則當前網路尚未達到最大流，反之，當 \\(s, t\\) 之間不存在增廣路徑，則當前流量為最大流，故在存在增廣路徑時不斷增加新流量，直至找不到增廣路徑後，當前網路即達到最大流。
    
  <details>
      <summary> 程式碼範例（模板） </summary>

    class Ford_Fulkerson {

    public :
        #define INF 100000000000000009
        #define ll long long
        struct _edge{
            ll to, rev;
            ll cap;
        };

        ll N, M, s, t;
        vector<_edge> v[10010];
        bool vis[10010];

        void setVal(ll N, ll M, ll s, ll t){
            this->N = N;
            this->M = M;
            this->s = s;
            this->t = t;
        }

        void build(ll a, ll b, ll cap){
            v[a].push_back({b, (int)v[b].size(), cap});
            v[b].push_back({a, (int)v[a].size() - 1, 0});
        }

        ll dfs(ll now, ll flow){
            if(now == t){
                return flow;
            }
            vis[now] = true;
            for(_edge &e : v[now]){
                if(!vis[e.to] && e.cap > 0){
                    ll f = dfs(e.to, min(flow, e.cap));
                    if(f){
                        e.cap -= f;
                        v[e.to][e.rev].cap += f;
                        return f;
                    }
                }
            }
            return 0;
        }

        ll flow(){
            ll i, j;
            ll ans = 0;
            while(true){
                for(i = 1; i <= N; i ++) vis[i] = false;
                vis[s] = false;
                vis[t] = false;
                ll f = dfs(s, INF);
                if(!f) break;
                ans += f;
            }
            return ans;
        }
      };
  </details>
  
## Edmond-Karp 算法

  * 基於 Ford-Fulkerson 算法的優化
    * 在前文中 Ford-Fulkerson 算法的時間複雜度分析中，可以發現若使用 DFS 來尋找增廣路徑，理論上會存在重複擴充新流量，但每次新增的流量都極小的情況，這也是為甚麼它的複雜度是 \\(O(EF)\\) 的原因，我們會發現若將找增廣路徑的方法改成 BFS，則會在更早的時候就將所有增廣路徑找完，以下為前文 Ford-Fulkerson 算法中所舉的例子。
    
      <details>
      <summary> 以BFS尋找增廣路徑的過程 </summary>

        ![](images/EK_image_1.jpg)
    
        ![](images/EK_image_2.jpg)
    
        ![](images/EK_image_3.jpg)
    
        ![](images/EK_image_4.jpg)
    
        ![](images/EK_image_5.jpg)
    
        ![](images/EK_image_6.jpg)
    
        ![](images/EK_image_7.jpg)

      </details>
    
    * 除了以上例子之外，還有一種情況在使用 DFS 時會尋找較慢，請看以下例子。
      <details>
      <summary> 另一個例子 </summary>
      
        ![](images/EK_image_8.jpg)
        
        * 在此圖中，若用 DFS 的話，會先找到上面那一條路徑，但因為最後最大流只需要底下那條較短的路徑貢獻就好，若是使用 BFS 則會在一開始就找到底下的路徑。
    
      </details>

  * 演算法過程
    * 為所有弧建立反向邊
    * 不斷使用 BFS 尋找增廣路徑
      * 若找到增廣路徑，則和 Ford-Fulkerson 算法一樣，將路徑上所有弧減去路徑上的最小剩餘容量 \\(f\\)。
      * 將總流量增加 \\(f\\)，並持續尋找增廣路徑，當找不到增廣路徑時，演算法結束，此時總流量即為這個圖上的最大流。
    * 需要注意的是，在 Ford-Fulkerson 算法中，尋找增廣路徑使用的是 DFS，所以可以邊尋找增廣路徑邊做更改剩餘容量的動作，而這裡使用的是 BFS，所以找增廣路徑的時候，必須同時為每個點紀錄它的上一個點，如此一來，要更改剩餘容量時，即可從匯點利用該紀錄回朔出增廣路徑。
  * 時間複雜度及正確性分析
    * 尋找一條增廣路徑的複雜度為 \\(O(E)\\)，最多會尋找 \\(O(VE)\\) 次增廣路徑，總複雜度為 \\(O(VE^{2})\\)，而這樣的複雜度和 Ford-Fulkerson 相比，即可忽略掉值域的影響，算是優化了不少。
    * 和 Ford-Fulkerson 的正確性一樣，根據最大流最小割定理，找不到增廣路徑時，則當前網路上形成最大流。
  <details>
      <summary> 程式碼範例（模板） </summary>
  
    class Edmond_Karp {

    public :
        #define INF 100000000000000009
        #define ll long long
        struct _edge{
            ll to, rev;
            ll cap;
        };

        ll N, M, s, t;
        pair<ll, ll> pre[10010];
        vector<_edge> v[10010];
        ll MinCap[10010];

        void setVal(ll N, ll M, ll s, ll t){
            this->N = N;
            this->M = M;
            this->s = s;
            this->t = t;
        }

        void build(ll a, ll b, ll cap){
            v[a].push_back({b, (int)v[b].size(), cap});
            v[b].push_back({a, (int)v[a].size() - 1, 0});
        }

        bool bfs(){
            ll i, j;
            for(i = 0; i <= N; i ++) MinCap[i] = 0;
            MinCap[s] = INF;
            queue<ll> q;
            q.push(s);

            while(!q.empty()){
                ll now = q.front();
                q.pop();
                for(i = 0; i < (ll)v[now].size(); i ++){
                    ll to = v[now][i].to;
                    ll cap = v[now][i].cap;
                    if(!MinCap[to] && cap > 0){
                        pre[to] = make_pair(now, i);
                        MinCap[to] = min(MinCap[now], cap);
                        q.push(to);
                    }
                }
                if(MinCap[t]) break;
            }
            return MinCap[t];
        }

        ll getflow(){
            ll now = t;

            while(now != s){
                ll last = pre[now].first;
                ll index = pre[now].second;
                v[last][index].cap -= MinCap[t];
                v[now][v[last][index].rev].cap += MinCap[t];
                now = last;
            }
            return MinCap[t];
        }

        ll flow(){
            ll i, j;
            ll ans = 0;
            while(bfs()){
                ans += getflow();
            }
            return ans;
        }
    };
  
  </details>

## 題目實作

* 有 \\(N\\) 台電腦和 \\(M\\) 個連結，每個連結 \\((a, b, c)\\) 代表 \\(a\\) 電腦單向以速度 \\(c\\) 傳輸資料到 \\(b\\) 電腦，電腦 \\(1\\) 是伺服器，電腦 \\(N\\) 是使用者，請求出使用者最大能從伺服器下載資料的總速度。[CSES 1694](https://cses.fi/problemset/task/1694)
  
  <details>
      <summary> 解題想法 </summary>

    * 將每台電腦視為點，連結和速度分別視為弧和其容量，建完圖後將源點設為 \\(1\\)，匯點設為 \\(N\\)，最大流即為答案。

  </details>
  
  <details>
      <summary> 參考程式碼（使用Edmond-Karp模板） </summary>

    ```
    int main(){
      int n, m, a, b, c, i;
      while(cin >> n >> m){
          Edmond_Karp ek;
          ek.setVal(n, m, 1, n);
          for(i = 1; i <= m; i ++){
              cin >> a >> b >> c;
              ek.build(a, b, c);
          }
          cout << ek.flow() << '\n';
      }
    }
    ```

  </details>
* 有一座城市有 \\(N\\) 棟建築物及 \\(M\\) 個道路，每條道路為雙向連接兩棟建築物，且有其長度，請求出最少需要關閉幾條道路，才能使建築物 \\(1\\) 和建築物 \\(N\\) 之間無法往來。
  
  <details>
      <summary> 解題想法 </summary>

    * 此問題等同於求出圖上的最小割，因有最大流等於最小割的性質，將每條邊的容量設為 \\(1\\)，最後求出的最大流即為答案。

  </details>
  
  <details>
      <summary> 參考程式碼 </summary>

    ```
    int main(){
      int n, m, a, b, i;
      while(cin >> n >> m){
          Edmond_Karp ek;
          ek.setVal(n, m, 1, n);
          for(i = 1; i <= m; i ++){
              cin >> a >> b;
              ek.build(a, b, 1);
          }
          cout << ek.flow() << '\n';
      }
    }
    ```

  </details>

## References

[網路流 - 維基百科](https://zh.wikipedia.org/zh-tw/%E7%BD%91%E7%BB%9C%E6%B5%81)

[网络流的基本概念 - 算法竞赛教程](https://www.dotcpp.com/course/1070)

[最大流最小割定理 - 演算法的分析與證明](https://tmt514.github.io/algorithm-analysis/max-flow/max-flow-min-cut-theorem.html)

[Ford-Fulkerson 最大流求解方法](https://www.desgard.com/algo/docs/part4/ch03/2-ford-fulkerson/)

[Edmond-Karp 最大流算法详解](https://www.desgard.com/algo/docs/part4/ch03/4-edmond-karp/)
