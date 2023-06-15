# BCC

## 前言

在此章節中，我們將介紹無向圖上的 BCC-Vertex(Bi-connected Component) 和 BCC-Edge(Bridge Connected Component)，包括他們的定義、相關的演算法和題目。

## BCC-Vertex(Bi-connected Component)

BCC-Vertex 指的是沒有 AP 的 Connected Component，在中文常稱之為點雙連通分量。例如下圖中有三個 BCC-Vertex
<img src="image/Biconnected Component.JPG" width="300" style="display:block; margin: 0 auto;"/>

BCC-Vertex 有以下幾個性質:

- 不同的 BCC-Vertex 之間最多共用一個點(否則，這兩個 BCC-Vertex 就會同屬於一個 BCC-Vertex)
- 一個至少有三個點的 BCC-Vertex，任兩點間至少會有兩條沒有共用邊的簡單路徑
- 對於在同一個 BCC-Vertex 中的任意三個相異點 \\(a,b,c\\)，必存在一條簡單路徑依序經過 \\(a,b,c\\) 三點
- 把每個 BCC-Vertex 中除了 AP 以外的點縮成一個點，那麼得到的新圖會是一棵樹或森林。

不同的 BCC-Vertex 之間會共用點，共用的部分恰好會是圖上的 AP。
而要找出圖上所有的 BCC-Vertex，我們可以通過修改找 AP 的演算法來達成。

### 如何修改

我們可以用 stack 紀錄首次遇到的邊。這樣當我們發現 \\(low(v) \geq depth(u) \\) 時，stack 中 \\( (u,v) \\) 及它上面的邊就會位於同一個 BCC-Vertex 中。就像是下圖 \\( (C,D) \\) 這條邊。
<img src="image/BCC Algo explain.PNG" width="300" style="display:block; margin: 0 auto;"/>

一個完整的例子如下
<img src="image/BCC Algo example.gif" width="300" style="display:block; margin: 0 auto;"/>

### Time Complexity

做完一次 DFS 就會有答案。而每條邊只會被 push 跟 pop 一次，因此 Time Complexity 為 \\( O(V+E) \\)

### code

```cpp
using edge = pair<int, int>;
void dfs(int u, int parent, int dep) {
    
    depth[u] = low[u] = dep;

    for(auto &v : G[u]) {
        if(v == parent) continue;
        if(depth[v] < depth[u]) stk.emplace(u,v);
        if(depth[v] == 0) {
            dfs(v, u, dep+1);
            low[u] = min(low[v], low[u]);
            if(low[v] >= depth[u]) {
                edge x;
                bcc.emplace_back({});
                do {
                    x = stk.top(); stk.pop();
                    bcc.back().emplace_back(x);
                } while(x != edge(u,v));
            }
        } else {
            low[u] = min(low[u], depth[v]);
        }
    }
    
    if(u == parent) {
        while(!stk.empty()) {
            edge x;
            bcc.emplace_back({});
            do {
                x = stk.top(); stk.pop();
                bcc.emplace_back(x);
            } while(x.first != 1)
        }
    }
}

```

## Exercise

> [The 2020 ICPC Asia Taipei-Hsinchu Site Programming Contest pI - Critical Structures](https://codeforces.com/gym/102835/problem/I)
>
>請有興趣的讀者自行去連結查看題目敘述，以免暴雷未來打算模擬這屆台北站當作團練的人。

<details><summary> Solution </summary>

題目要找 AP, Bridge, BCC-Vertex 的數量，以及擁有最多邊的 BCC-Vertex 的邊數。

其實就是模板題

</details>

>[Codeforce - Simple Cycles Edges](https://codeforces.com/problemset/problem/962/F)
>
>給定一張 \\( N \\) 個點 \\( M \\) 條邊的無向圖，問那些邊剛好位於一個簡單環上。

<details><summary> Solution </summary>

想想 BCC-Vertex 的性質，會發現

</details>

## BCC-Edge(Bridge Connected Component)

BCC-Edge 指的是沒有 Bridge 的 Connected Component，在中文常稱之為邊雙連通分量、橋連通分量。例如下圖我們能找到兩個 BCC-Edge
<img src="image/Bridge Connected Component.JPG" width="300" style="display:block; margin: 0 auto;"/>

BCC-Edge 有以下幾個性質

- 同一個 BCC-Edge 的任兩點間至少存在兩條沒有共用邊的簡單路徑
- 如果把每個 BCC-Edge 縮成一個點，那麼新得到的圖會是一棵樹或者森林。

不同的 BCC-Edge 沒有交集。而要找出圖上所有的 BCC-Edge，我們可以通過修改找 Bridge 的算法來達成

### 如何修改

跟找 BCC-Vertex 的想法很像。我們用 stack 紀錄走過的點，當我們發現 \\(low(u) == depth(u) \\) 時，我們就發現了橋的下端點。而 stack 中 \\(u \\) 和他上面的點就會位於同一個 BCC。就像下圖 \\( D \\) 這個點，他是 \\( (C,D) \\) 這條 bridge 的下端點。
<img src="image/BCC Algo explain 2.PNG" width="300" style="display:block; margin: 0 auto;"/>

一個完整的例子如下
<img src="image/BCC Algo example 2.gif" width="300" style="display:block; margin: 0 auto;"/>

### Time Complexity

做完一次 DFS 就會有答案。而每個點只會被 push 跟 pop 一次，因此 Time Complexity 為 \\( O(V+E) \\)

### code

```cpp

void dfs(int u, int parent, int dep) {
    
    depth[u] = low[u] = dep;
    stk.emplace(u);

    for(auto &v : G[u]) {
        if(v == parent) continue;
        if(depth[v] == 0) {
            dfs(v, u, dep+1);
            low[u] = min(low[v], low[u]);
        } else {
            low[u] = min(low[u], depth[v]);
        }
    }
    
    if(low[u] == depth[u]) {
        bcc.emplace_back({});
        int x;
        do {
            x = stk.top(); stk.pop();
            bcc.back().emplace_back(x);
        } while(x != u);
    }
}

```

## Exercise

>[2015 ACM Amman Collegiate Programming Contest pH - Bridges](https://codeforces.com/gym/100712)
>
> 請有興趣的讀者自行去連結查看題目敘述，以免暴雷未來打算模擬這場比賽當作團練的人。

<details><summary> Solution </summary>

我們首先把所有的 BCC-Edge 縮點，根據 BCC-Edge 的性質我們會得到一棵樹，樹上的邊都是 bridge。

我們思考如果在樹上 \\( u,v \\) 兩點間加上一條邊後，有哪些邊就不再是 bridge 了。可以發現樹上從 \\( u \\) 到 \\( v \\) path 上的所有邊都不再是 bridge。而我們想要讓最多的 bridge 消失，因此問題就轉換為了求樹的直徑。

</details>

## Problems

>[TIOJ - 圓桌武士](https://tioj.ck.tp.edu.tw/problems/1684)
>
>有 \\( N \\) 個武士和 \\( M \\) 個武士之間倆倆的仇恨關係，如果能選出奇數個武士(至少三個)坐在圓桌邊，且相鄰的兩人不互相仇恨，那他們就可以開會。問有多問有多少位武士永遠不可能開會。

<details><summary> Hint </summary>

反過來思考，我們改成找可以和別人開會的武士有誰。我們可以觀察原圖的補圖 (即武士之間的不仇恨關係)，想想看這張補圖上符合怎樣性質的點至少可以參與到一場會議中。

</details>

>[Codeforces - Tourist Reform](https://codeforces.com/contest/732/problem/F)
>
>給定一張 \\( N \\) 個點 \\( M \\) 條邊的無向圖，定義 \\(r_i \\) 為幫每條邊定向後，\\(i \\) 點可以走到的點數。要你給出一種定向方法使得 \\(min_i({r_i})\\) 最大

<details><summary> Hint </summary>

找出所有的 BCC-Edge 後，想想看最大化的 \\(min_i({r_i})\\) 至少會等於什麼?

</details>

> [POJ - Road Construction](http://poj.org/problem?id=3352)
>
> 給一張無向圖，問最少要加幾條邊才能使得圖上沒有橋。

<details><summary> Hint </summary>

跟 BCC-Edge 的練習題有點像，想想看 leaf 之間要怎樣加邊。

</details>

## Reference

- [oi-wiki - BCC](https://oi-wiki.org/graph/bcc/)
- [Hackerearth - BCC](https://www.hackerearth.com/practice/algorithms/graphs/biconnected-components/tutorial/)
- [sylveon slides - BCC](https://slides.com/sylveon/graph-7#/4)
- [建中培訓講義 - BCC](https://tioj.ck.tp.edu.tw/uploads/attachment/5/33/8.pdf)
