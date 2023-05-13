# 最大流基本定理證明與Ford-Fulkerson and Edmonds-Karp

## 網路流及用語介紹

* 什麼是網路流?

    * 以圖論的角度來說，網路流是一張有向圖，每一條邊都有一個容量代表可流過的上限值，可以將邊想像成水管，流過的值即為水的即時流量。
   
   * 整張圖會有一個源點以及匯點，源點能有無限制的水量供應，並且所有點滿足水流入量等於水流出量，最終目的則是求出從源點最多能流多少水到匯點。 
   
   * 許多和分配、運輸量、最小花費的題目都可以被轉換成網路流問題，而看出它們可被轉換、以及轉換的過程往往是網路流問題最困難的部分。

* 網路流用語介紹
  * 源點(Source)：擁有無限水量可供應的點，記為 \\(s\\) 。
  * 匯點(Sink)：可接收無限水量的點，記為 \\(t\\) 。
  * 弧：網路流圖上的一條帶權邊 \\((u, v) \in E\\) 
  * 淨流：通過一條弧 \\((u, v)\\) 的流量(淨流)，記為 \\(f(u, v)\\) 。
  * 容量(Capacity)：能通過一條弧 \\((u, v)\\) 的最大淨流，同時也是邊 \\((u, v)\\) 的權重，記為 \\(c(u, v)\\) 。
  * 網路流圖：
    * 洽有一個源點。
    * 洽有一個匯點。
    * 為一個點和邊的集合，可記為 \\(G(V,E)\\) 。
    * \\(\forall u, v \in G(V,E), c(u, v) \geq 0\\) 
  * 網路流(Network flow)：一個流量集合 \\(F=\{f(u, v)\}\\) 。
  * 可行流：滿足以下條件的網路流稱為可行流
    * 容量限制： \\(0 \leq f(u, v) \leq c(u, v)\\) 。
    * 流量守恆：除了源點和匯點，其他點都必須滿足流入量=流出量。
  * 最大流：所有可行流當中流量最大的即為最大流。
  * 剩餘容量(Residual Capacity)：記為 \\(c_{f}(u, v) = c(u, v) - f(u, v)\\) 
  * 剩餘網路(Residual Network)：由所有為其剩餘容量的邊，以及所有原本的點所組成的網路流圖。
  * 割(Cut)：
    * 割將網路流圖切分成兩集合 \\(S, T\\) ，其中 \\(S \cap T = \emptyset\\) 且 \\(S \cup T = G\\) ，其中 \\(s \in S\\) ， \\(t \in T\\) 。 
    * 割的容量為所有在集合 \\(S\\) 中的點連向集合 \\(T\\) 中的點的弧容量總和。 \\(c(S, T) = \sum_{u \in S} \sum_{v \in T} c(u, v)\\) 
  * 最小割(Minimum cut)：所有割裡面容量最小的為最小割。

* 網路流的題目舉例
  * 給定 \\(N\\) 個男生和 \\(M\\) 個女生，以及 \\(K\\) 個關係 \\((u, v)\\) ，每個關係代表 \\(u\\) 和 \\(v\\) 互相喜歡，且不會出現同性戀的情形，請求出最多的情侶關係組數。
  * 有一座城市有 \\(N\\) 棟建築物及 \\(M\\) 個道路，每條道路為雙向連接兩棟建築物，且有其長度，請求出最少需要關閉幾條道路，才能使建築物 \\(1\\) 和建築物 \\(N\\) 之間無法往來。


## 最大流最小割定理(Max Flow Min Cut Theorem)



## Ford-Fulkerson 算法

  * 首先先介紹一個最大流演算法的核心概念之一，增廣路徑(Augmenting Path)
    * 一個路徑 \\(\{u_{1}, u_{2}, ..., u_{n}\}\\) ，其中 \\(u_{1}=s\\) ， \\(u_{n}=t\\) ， \\(c_{f}(u_{i}, u_{i + 1}) > 0\\) 。
    * 簡單來說，就是從源點走到匯點的路徑中，若該路徑上所有弧的剩餘容量都大於0，則該路徑稱作增廣路徑。
    * 以功能來說，增廣路徑的存在，就表示從源點開始還有可流通至匯點的容量可使用。
  * 先介紹一個直覺找最大流的作法
  
  
  * 演算法過程
    * 為所有弧建立反向邊。
    * 不斷使用DFS尋找增廣路徑
      * 若找到增廣路徑，且增廣路徑上所有弧的剩餘容量中的最小值為 \\(f\\) ，則將增廣路徑上所有弧的剩餘容量減去 \\(f\\) ，且將所有弧的反向邊的剩餘容量加上 \\(f\\) 。
      * 將總流量增加 \\(f\\) ，並持續尋找增廣路徑，當找不到增廣路徑時，演算法結束，此時總流量即為這個圖上的最大流。
  
  * 時間複雜度及正確性分析
    * 尋找一條增廣路徑複雜度為 \\(O(V+E)\\) ，最差的情況之下，每次增廣路徑增加的流量為 \\(1\\) ，總共要找 \\(F\\) 次增廣路徑( \\(F\\) 為圖上最大的最大流)，演算法總複雜度為 \\(O((V+E)F)\\) ，邊數通常較多，可簡化為 \\(O(EF)\\) 。
    *
  * 程式碼範例（模板）
  ```
template<class T>
class Ford_Fulkerson {
    public :
        #define INF 100000000000000009
        struct _edge{
            int to, rev;
            T cap;
        };

        int N, M, s, t;
        vector<_edge> v[10010];
        bool vis[10010];

        void setVal(int N, int M, int s, int t){
            this->N = N;
            this->M = M;
            this->s = s;
            this->t = t;
        }

        void build(int a, int b, T cap){
            v[a].push_back({b, (int)v[b].size(), cap});
            v[b].push_back({a, (int)v[a].size() - 1, 0});
        }

        T dfs(int now, T flow){
            if(now == t){
                return flow;
            }
            vis[now] = true;
            for(_edge &e : v[now]){
                if(!vis[e.to] && e.cap > 0){
                    T f = dfs(e.to, min(flow, e.cap));
                    if(f){
                        e.cap -= f;
                        v[e.to][e.rev].cap += f;
                        return f;
                    }
                }
            }
            return 0;
        }

        T flow(){
            int i, j;
            T ans = 0;
            while(true){
                for(i = 1; i <= N; i ++) vis[i] = false;
                vis[s] = false;
                vis[t] = false;
                T f = dfs(s, INF);
                if(!f) break;
                ans += f;
            }
            return ans;
        }
};

  ```
## Edmond-Karp 算法
  * 關於Ford-Fulkerson算法的問題
  
  * 演算法過程
    * 為所有弧建立反向邊
    * 不斷使用BFS尋找增廣路徑
      * 若找到增廣路徑，則和Ford-Fulkerson算法一樣，將路徑上所有弧減去路徑上的最小剩餘容量 \\(f\\) 。
      * 將總流量增加 \\(f\\) ，並持續尋找增廣路徑，當找不到增廣路徑時，演算法結束，此時總流量即為這個圖上的最大流。
    * 需要注意的是，在Ford-Fulkerson算法中，尋找增廣路徑使用的是DFS，所以可以邊尋找增廣路徑邊做更改剩餘容量的動作，而這裡使用的是BFS，所以找增廣路徑的時候，必須同時為每個點紀錄它的上一個點，如此一來，要更改剩餘容量時，即可從匯點利用該紀錄回朔出增廣路徑。
  * 時間複雜度及正確性分析
    * 尋找一條增廣路徑的複雜度為 \\(O(E)\\) ，最多會尋找 \\(O(VE)\\) 次增廣路徑，總複雜度為 \\(O(VE^{2})\\) 。
    *
  * 程式碼範例（模板）
  ```
template<class T>
class Edmond_Karp {

    public :
        #define INF 100000000000000009
        struct _edge{
            int to, rev;
            T cap;
        };

        int N, M, s, t;
        pair<int, int> pre[10010];
        vector<_edge> v[10010];
        T MinCap[10010];

        void setVal(int N, int M, int s, int t){
            this->N = N;
            this->M = M;
            this->s = s;
            this->t = t;
        }

        void build(int a, int b, T cap){
            v[a].push_back({b, (int)v[b].size(), cap});
            v[b].push_back({a, (int)v[a].size() - 1, 0});
        }

        bool bfs(){
            int i, j;
            for(i = 0; i <= N; i ++) MinCap[i] = 0;
            MinCap[s] = INF;
            queue<int> q;
            q.push(s);

            while(!q.empty()){
                int now = q.front();
                q.pop();
                for(i = 0; i < (int)v[now].size(); i ++){
                    int to = v[now][i].to;
                    T cap = v[now][i].cap;
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

        T getflow(){
            int now = t;

            while(now != s){
                int last = pre[now].first;
                int index = pre[now].second;
                v[last][index].cap -= MinCap[t];
                v[now][v[last][index].rev].cap += MinCap[t];
                now = last;
            }
            return MinCap[t];
        }

        T flow(){
            int i, j;
            T ans = 0;
            while(bfs()){
                ans += getflow();
            }
            return ans;
        }
};

  ```

## 題目實作

* 有 \\(N\\) 台電腦和 \\(M\\) 個連結， 每個連結 \\((a, b, c)\\) 代表 \\(a\\) 電腦單向以速度 \\(c\\) 傳輸資料到 \\(b\\) 電腦，電腦 \\(1\\) 是伺服器，電腦 \\(N\\) 是使用者，請求出使用者最大能從伺服器下載資料的總速度。https://cses.fi/problemset/task/1694
  * 將每台電腦視為點，連結和速度分別視為弧和其容量，建完圖後將源點設為 \\(1\\) ，匯點設為 \\(N\\) ，最大流即為答案。
  * 參考程式碼（使用Edmond-Karp模板）
  ```
  int main(){
    int n, m, a, b, c, i;
    while(cin >> n >> m){
        Edmond_Karp<long long> ek;
        ek.setVal(n, m, 1, n);
        for(i = 1; i <= m; i ++){
            cin >> a >> b >> c;
            ek.build(a, b, c);
        }
        cout << ek.flow() << '\n';
    }
  }
  ```
  
* 有一座城市有 \\(N\\) 棟建築物及 \\(M\\) 個道路，每條道路為雙向連接兩棟建築物，且有其長度，請求出最少需要關閉幾條道路，才能使建築物 \\(1\\) 和建築物 \\(N\\) 之間無法往來。
  * 此問題等同於求出圖上的最小割，因有最大流等於最小割的性質，將每條邊的容量設為 \\(1\\) ，最後求出的最大流即為答案。
  * 參考程式碼
  ```
  int main(){
    int n, m, a, b, i;
    while(cin >> n >> m){
        Edmond_Karp<long long> ek;
        ek.setVal(n, m, 1, n);
        for(i = 1; i <= m; i ++){
            cin >> a >> b;
            ek.build(a, b, 1);
        }
        cout << ek.flow() << '\n';
    }
  }
  ```
  
  