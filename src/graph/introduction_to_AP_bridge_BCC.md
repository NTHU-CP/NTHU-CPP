# AP, Bridge, BCC

## 前言

在此章節中，我們將介紹無向圖上的 AP(Articulation point), Bridge 與 BCC(Bi-connected Component & Bridge Connected Component)，包括他們的定義、相關的演算法和題目。

## Connected & Connected Component
在正式開始我們的主題之前，我們先介紹一些之後會用到的東西。

### Connected(連通)
我們說兩個點 A, B connected，表示 A, B之間存在一條 path。如下圖例子中，A 跟 B connected，但 A 跟 C 不 connected
<img src="image/Connected.JPG" width="500" style="display:block; margin: 0 auto;"/>

### Connected Component(連通分量)
Connected Component 是點的集合，滿足以下條件
- 集合中任兩點都是 connected 的
- 如果兩個點位於不同的 Connected Component，則這兩點必不 connected。  

如下圖的例子，顏色相同的點表示他們屬於相同的 Connected Component，可以看到若是兩個點不同顏色，那他們一定不 connected。而我們不會說紅圈圈起來的那三個點是一個 Connected Component(因為紅圈外還有跟他們連通的點)。

<img src="image/Connected_Component.JPG" width="500" style="display:block; margin: 0 auto;"/>

### DFS tree on undirected graph
我們先介紹無向圖上的 DFS tree。 DFS tree 就如同他的名字，是一棵根據 DFS 的過程長出的 tree。

考慮以下 DFS 的走訪，那些直接走訪的邊被稱為 tree edge，而其他的邊被稱為 back edge。(在有向圖上的 DFS tree 還會有其他種類的邊，這部分留給讀者自行學習)
```cpp
void DFS(int u) {
	visited[u] = true;
	for(int &v : G[u]) {
		if(!visited[v]) {
			DFS(v);//u->v 為 tree edge
		} else {
			//u->v為 back edge
		}
	}
}
```

我們可以看個例子。

<img src="image/DFS Tree.gif" width="400" style="display:block; margin: 0 auto;"/>

如果把上圖畫成 DFS tree 就會長成這樣：

<img src="image/DFS Tree.JPG" width="200" style="display:block; margin: 0 auto;"/>

在無向圖上做 DFS tree 時要注意的是: 無向圖的 DFS 會讓一條邊被看到 2 次。如果你的 code 寫的是看到一條邊就根據它是 tree edge/back edge 做事，那你有可能做了重複的操作而導致 WA。
```cpp
void DFS(int u) {
	visited[u] = true;
	for(int &v : G[u]) {
		if(!visited[v]) {
			DFS(v);
			...
		} else {
			...
			//你可能會在這裡先看到 a->b，之後又看到 b->a
		}
	}
}
```

一個修改方法如下:

```cpp
void DFS(int u, int parent) { //call(u,u) at first
	color[u] = 1;
	for(auto &v : G[u]) {
		if(v == parent) continue;//已經是 tree edge
		if(color[v] == 0) {
			//tree edge
			dfs(v, u);
			...
		} else if(color[v] == 1){
			//第一次看到的back edge
			...
		}
		//如果color[v] == 2，代表 v 的 DFS 已經結束，因此 (u,v) 這條邊已經被 v 看過了
		//如果還沒，那 v 的 DFS 就不應該結束。
	}
	color[u] = 2;
}
```

而如果你在 DFS 的過程中有紀錄一個點在 DFS tree 上的深度，那你也可以根據兩點間的深度關係來判斷一條邊是不是第一次被看到。


## AP & Bridge
 
AP 指的是一張圖 \\(G \\) 移除一個點 \\(v \\) 以及與 \\(v \\) 相連的邊後 Connected Component 的數量變多，則點 \\(v \\) 為 AP。中文常稱之為關節點、割點。例如下圖的 A 點就是 AP  
<img src="image/AP.JPG" width="500" style="display:block; margin: 0 auto;"/>

Bridge 指的是一張圖 \\(G \\) 移除一條邊 \\(e \\) 後 Connected Component 的數量變多，則邊 \\(e \\) 為 Bridge。中文常稱之為橋。例如下圖的紅色邊就是 Bridge   
<img src="image/Bridge.JPG" width="500" style="display:block; margin: 0 auto;"/>

那我們要如何快速找到圖上所有的 AP 跟 Bridge 呢？很容易可以想到枚舉每個點或邊，把他拔掉之後看看圖上有沒有多出新的 Connected Component。但這樣做的時間複雜度會是 \\(O((V+E)^2) \\)。不過實際上，我們只要好好觀察圖上的性質就可以在 \\(O(V+E) \\) 的時間做完！以下介紹兩種不同的方法來找出圖上所有的 AP 跟 Bridge

## Using DFS tree for Bridge/AP

### 觀察 for bridge

我們觀察圖中那些邊絕對不可能是 bridge

<img src="image/DFS Tree Observation.JPG" width="400" style="display:block; margin: 0 auto;"/>

- back edge 絕對不會是 bridge。
- **如果\\( (u,v) \\)是 back edge，那麼樹上 \\(u \\) 到 \\( v \\) 的路徑都不會是 bridge**。例如圖中因為有 \\( (F,C) \\) 這條 back edge，因此樹上 \\(F \\) 到 \\(C \\) 的路徑都不會是 bridge。

所以，如果我們每遇到一條 back edge \\( (u,v) \\)，就把樹上 \\(u \\) 到 \\( v \\) 的路徑都標記成不是 bridge，那麼最後那些沒被標記到的邊就會是 bridge。問題是，**我們要如何快速標記一條路徑上所有的邊？**

### 快速標記

這個問題我們可以用前綴和的想法來解決。當我們遇到一條 back edge \\( (u,v) \\) 時，就在 \\(u \\) 上 +1， 在 \\( v \\) 上 -1。 **這代表說有一條 back edge 從 \\(u \\) 開始，在 \\( v \\) 結束**。

<img src="image/prefix 1.JPG" width="400" style="display:block; margin: 0 auto;"/>

這樣當我們由下而上計算前綴和時，\\(u \\) 到 \\( v \\) 的 path 就全部被標記好了!

<img src="image/prefix 2.JPG" width="400" style="display:block; margin: 0 auto;"/>

### 前綴和 -> 子樹標記總和
而實際上，對於一條 tree edge \\( (u,v) \\)，我們只要判斷以 \\( v \\) 為根的子樹標記總和是否為 0，若為 0 則 \\( (u,v) \\) 就會是 bridge。

為甚麼？從下圖可以發現，如果一條 back edge 的開始跟結束都在以 \\( v \\) 為根的子樹內，那麼這條 back edge 對子樹總和的貢獻為 0，否則為 1。而當子樹總和不為 0 的時候，\\( (u,v) \\) 顯然不會是 bridge。
<img src="image/subtree sum.JPG" width="400" style="display:block; margin: 0 auto;"/>

### Time Complexity
我們做完一次 DFS 之後就能得到答案，因此 Time Complexity 為 \\( O(V+E) \\)

### Code
```cpp
void dfs(int u, int parent) {
	color[u] = 1;
	for(auto &v : G[u]) {
		if(v == parent) continue;
		if(color[v] == 0) {
			dfs(v, u);
			if(sum[v] == 0) 
				bridge.emplace_back(u, v);
			sum[u]+=sum[v];
		} else if(color[v] == 1){
			sum[u]+=1;
			sum[v]-=1;
		}
	}
	color[u] = 2;
}
```

### 如何找 AP

我們首先回想一下 Tarjan 對 AP 的觀察：
- 對於 root，如果他有至少兩棵子樹的時候，他就一定是 AP，否則就不是。
- 對於一個點 \\(v \\)，如果 \\(v \\) 的某一棵子樹無法在不經過 \\(v \\) 的情況下走到 \\(v \\) 的祖先，那麼 \\(v \\) 一定是 AP。
<img src="image/General AP.JPG" width="400" style="display:block; margin: 0 auto;"/>

第一點很好解決。

對於第二點，我們要檢查的其實就是**對於一棵子樹，有沒有 back edge 跨過 \\( v \\)**，如果沒有那 \\( v \\) 就會是 AP。

而這個東西要計算的，其實就是 \\(v \\) 的子樹標記總和減去這顆子樹中在 \\( v \\) 結束的 back edge 數量。我們可以看下圖的例子，左邊的子樹總和為 1，而在 \\(v \\) 結束的 back edge 也為 1， 因此 1-1 = 0， \\( v \\) 為 AP。而在右邊則因為沒有 back edge 在 \\(v \\) 結束，因此 1-0 = 1，\\( v \\) 不是 
AP 
<img src="image/AP subtree sum.JPG" width="400" style="display:block; margin: 0 auto;"/>

### Time Complexity
我們做完一次 DFS 之後就能得到答案，因此 Time Complexity 為 \\( O(V+E) \\)

### Code
```cpp
void dfs(int now, int pa) {
	color[now] = 1;
	bool isAP = false;
	int child = 0;

	for(auto &e : G[now]) {
		if(e == pa) continue;
		if(color[e] == 0) {
			child++;
			int backEdgeNum = backEdgeEnd[now];
			dfs(e, now);
			if(sum[e] - (backEdgeEnd[now] - backEdgeNum) == 0) {
				isAP = true;;
			}
			sum[now]+=sum[e];
		} else if(c[e] == 1){
			sum[now]+=1;
			backEdgeEnd[e]+=1; 
		}
	}
	sum[now] -= backEdgeEnd[now];
	color[now] = 2;
	
	if(now == pa && child == 1) isAP = false;
	if(isAP) AP.emplace_back(now);
}

```


## Tarjan's Algorithm for AP/Bridge
接著我們要介紹 Tarjan 所提出的找 AP/Bridge 的演算法。同樣會用到 DFS tree。

### 觀察 for AP
我們觀察下圖中 \\( C \\) 點的左子樹。 你會發現當我們移除 \\( C \\) 點後， \\( C \\) 點的左子樹**就沒有路可以走到原本 \\( C \\) 點的祖先**，因此移除 \\( C \\) 點後會讓 \\( C \\) 點的左子樹跟 \\( C \\) 點的祖先處在不同的 Connected Component。

<img src="image/Tarjan AP Observation.JPG" width="400" style="display:block; margin: 0 auto;"/>

更一般的說，對於一個點 \\(v \\)，如果 \\(v \\) 的某一棵子樹無法在不經過 \\(v \\) 的情況下走到 \\(v \\) 的祖先，那麼 \\(v \\) 一定是 AP。

<img src="image/General AP.JPG" width="400" style="display:block; margin: 0 auto;"/>

但 root 是沒有祖先的，因此 root 我們要拉出來特別判斷。很明顯，當 root 有至少兩棵子樹的時候，root 一定會是 AP，否則就不是。
<img src="image/Root AP Observation.JPG" width="400" style="display:block; margin: 0 auto;"/>

### 觀察 for Bridge
我們觀察下圖中 \\( (C,D) \\) 這條邊，當我們把這條邊拔掉後，以 \\(D \\) 為根的子樹**就沒有路可以走到 \\( C \\) 點**，因此移除 \\( (C,D) \\) 後會讓 \\( C \\) 跟 \\( D \\) 處在不同的 Connected Component。

<img src="image/Tarjan Bridge Observation.JPG" width="400" style="display:block; margin: 0 auto;"/>

更一般的說，令 \\(u \\) 是 \\( v \\) 的 parent，則對於一條邊 \\( (u,v) \\)，如果 \\( v \\) 的子樹都無法在不經過 \\( (u,v) \\) 的情況下走到 \\( u \\)，那麼 \\( (u,v) \\) 一定是 bridge。

<img src="image/General Bridge.JPG" width="400" style="display:block; margin: 0 auto;"/>

### 演算法
現在我們來看看 Tarjan 是怎麼把這個東西做出來的。

Tarjan 首先定義了兩個函數 \\(depth \\) 跟 \\(low \\)。

\\(depth(v) \\) 表示 \\( v \\) 這個點在 DFS tree 上的深度。
<img src="image/Depth Example.JPG" width="400" style="display:block; margin: 0 auto;"/>

\\( low(v) \\) 表示 \\(v \\) 子樹中所有的點和這些點的鄰點，以及 \\( v \\) 本身的最淺深度。

例如下圖的 \\( C \\) 點，本身的深度是 \\(3\\)，子樹中所有的點深度都 \\(>3\\)，而子樹中最淺的鄰點為 \\( B \\)  ( \\( L \\) 的鄰點)，深度為 \\(2\\)，因此 \\(low(C) = 2\\) 
<img src="image/Low Example.JPG" width="400" style="display:block; margin: 0 auto;"/>

我們回想一下剛才的圖，發現對於一個點 \\(u \\)，如果他的某個子節點 \\(v \\) 滿足 \\(low(v) >= depth(u) \\)，那麼 \\(u\\)就會是 AP。

<img src="image/General AP Algo.JPG" width="400" style="display:block; margin: 0 auto;"/>

而對於一條邊 \\((u,v)\\)，如果滿足 \\(low(v) > depth(u)\\)，那麼 \\((u,v) \\) 就會是 Bridge

<img src="image/General Bridge Algo.JPG" width="400" style="display:block; margin: 0 auto;"/>

我們現在的問題只剩下如何求出 \\(low(v) \\)。

可以發現，對於一個點 \\(v \\)，如果我們知道他所有子節點的 \\(low \\) 的答案，那麼我們就可以簡單推出 \\(low(v) \\) 的答案，如下圖所示。而要達成這件事情，我們只需在 DFS 的時候用 post order 的順序計算 \\(low \\) 的答案就可以了！

<img src="image/Compute Low.JPG" width="400" style="display:block; margin: 0 auto;"/>

### Time Complexity
同樣也是做完一次 DFS 之後就能得到答案，因此 Time Complexity 為 \\( O(V+E) \\)

## code
``` cpp
void dfs(int u, int parent, int dep) {
	
	depth[u] = low[u] = dep;
	int child = 0;
	bool isAP = false; 
	
	for(auto &v : G[u]) {
		if(v == parent) continue;
		if(depth[v] == 0) {
			child++;
			dfs(v, u, dep+1);
			low[u] = min(low[v], low[u]);
			if(low[v] >= depth[u]) isAP = true;
			if(low[v] > depth[u]) Bridge.emplace_back(u,v);
		} else {
			low[u] = min(low[u], depth[v]);
		}
	}

	if(u == parent && child < 2) isAP = false;
	if(isAP) AP.emplace_back(u);	
	
}
```

## BCC(Bi-connected Component)

BCC 指的是沒有 AP 的 Connected Component，在中文常稱之為點雙連通分量。例如下圖中有三個 BCC
<img src="image/Biconnected Component.JPG" width="300" style="display:block; margin: 0 auto;"/>

不同的 BCC 之間會共用點，共用的部分恰好會是圖上的 AP。
而要找出圖上所有的 BCC，我們可以通過修改找 AP 的演算法來達成。

### 如何修改
我們可以用 stack 紀錄首次遇到的邊。這樣當我們發現 \\(low(v) >= depth(u) \\) 時，stack 中 \\( (u,v) \\) 及它上面的邊就會位於同一個 BCC 中。就像是下圖 \\( (C,D) \\) 這條邊。
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
				bcc.emplace_back({});
				while(stk.top() != edge(u,v)) { 
					bcc.back().emplace_back(stk.top());
					stk.pop();
				}
				bcc.back().emplace_back(stk.top());
				stk.pop();
			}
		} else {
			low[u] = min(low[u], depth[v]);
		}
	}
	
	if(u == parent) {
		while(!stk.empty()) {
			bcc.emplace_back({});
			while(stk.top().first != 1) {
				bcc.back().emplace_back(stk.top());
				stk.pop();
			}
			bcc.back().emplace_back(stk.top());
			stk.pop();
		}
	}
}

```
## BCC(Bridge Connected Component)

BCC 指的是沒有 Bridge 的 Connected Component，在中文常稱之為邊雙連通分量、橋連通分量。例如下圖我們能找到兩個 BCC
<img src="image/Bridge Connected Component.JPG" width="300" style="display:block; margin: 0 auto;"/>

不同的BCC沒有交集。而要找出圖上所有的 BCC，我們可以通過修改找 Bridge 的算法來達成

### 如何修改
跟找 Bi-connected Component 的想法很像。我們用 stack 紀錄走過的點，當我們發現 \\(low(u) == depth(u) \\) 時，我們就發現了橋的下端點。而 stack 中 \\(u \\) 和他上面的點就會位於同一個 BCC。就像下圖 \\( D \\) 這個點。
<img src="image/BCC Algo explain 2.PNG" width="300" style="display:block; margin: 0 auto;"/>

一個完整的例子如下
<img src="image/BCC Algo example 2.gif" width="300" style="display:block; margin: 0 auto;"/>

### Time Complexity
做完一次 DFS 就會有答案。而每個點只會被 push 跟 pop 一次，因此 Time Complexity 為 \\( O(V+E) \\)

### code
```cpp
wait
```

## Problems

> [CSES - Necessary Cities](https://cses.fi/problemset/task/2077)
> 
> 給定一張 \\( N \\) 個點 \\( M \\) 條邊的無向圖，要你找出圖上所有的 AP

AP 模板題

> [CSES - Necessary Roads](https://cses.fi/problemset/task/2076)
> 
> 給定一張 \\( N \\) 個點 \\( M \\) 條邊的無向圖，要你找出圖上所有的 Bridge


Bridge 模板題

> [The 2020 ICPC Asia Taipei-Hsinchu Site Programming Contest pI - Critical Structures](https://codeforces.com/gym/102835/problem/I)
>
>請有興趣的讀者自行去連結查看題目敘述，以免暴雷未來打算模擬這屆台北站當作團練的人。

<details><summary> Solution </summary>

題目要找 AP, Bridge, Biconnected Component 的數量，以及擁有最多邊的 Biconnected Component 的邊數。

其實就是模板題

</details>

>[Codeforces - Break Up](https://codeforces.com/problemset/problem/700/C)
>
> 給一張帶權無向圖與兩點 \\(S \\) , \\(T \\)，要你選至多兩條邊刪除後使\\(S \\) , \\(T \\)不連通。要求選的邊權重和最小。

>[Uva - Mining Your Own Business](https://onlinejudge.org/index.php?option=com_onlinejudge&Itemid=8&category=246&page=show_problem&problem=3549)
>
>在一張無向圖上選擇盡可能少的點塗黑，使得刪除任一個點後，每個連通分量裡至少有一個黑點。

>[Codeforces - Tourist Reform](https://codeforces.com/contest/732/problem/F)
>
>給定一張 \\( N \\) 個點 \\( M \\) 條邊的無向圖，定義 \\(r_i \\) 為幫每條邊定向後，\\(i \\) 點可以走到的點數。要你給出一種定向方法使得 \\(min_i({r_i})\\) 最大

>[Codeforce - Simple Cycles Edges](https://codeforces.com/problemset/problem/962/F)
>
>


## Reference
- [CP Algorithm - Connected components](https://cp-algorithms.com/graph/search-for-connected-components.html)
- [演算法筆記 - Connected components](https://web.ntnu.edu.tw/~algo/ConnectedComponent.html)
- [CP Algorithm - cutpoints](https://cp-algorithms.com/graph/cutpoints.html)
- [CP Algorithm - bridge](https://cp-algorithms.com/graph/bridge-searching.html)
- [oi-wiki - cut & bridge](https://oi-wiki.org/graph/cut/)
- [演算法筆記 - AP & bridge](https://web.ntnu.edu.tw/~algo/ConnectedGraph.html#3)
- [Hackerearth - AP & bridge](https://www.hackerearth.com/practice/algorithms/graphs/articulation-points-and-bridges/tutorial/)
- [演算法海牛 - bridge](https://www.facebook.com/algo.seacow/posts/pfbid0PMMPJEWmh3XgFtstTh8pptxjnJKK5jwpeVCQWEmfWVyRKT66LqccAv5DiSZ22zDhl)
- [codeforce blog - AP & bridge](https://codeforces.com/blog/entry/71146)
- [oi-wiki - BCC](https://oi-wiki.org/graph/bcc/)
- [Hackerearth - BCC](https://www.hackerearth.com/practice/algorithms/graphs/biconnected-components/tutorial/)
- [codeforce blog - DFS tree](https://codeforces.com/blog/entry/68138)



