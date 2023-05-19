# AP, Bridge, BCC

## 前言

在此章節中，我們將介紹無向圖上的AP(Articulation point), Bridge與BCC(Biconnected component)，包括他們的定義、相關的演算法和題目。

## Connected & Connected Component
在正式開始我們的主題之前，我們先來介紹一些之後會用到的東西。

### Connected(連通)
我們說兩個點 A, B connected，表示A, B之間存在一條path。如下圖例子中，A 跟 B connected，但 A 跟 C 不 connected
<img src="image/Connected.JPG" width="500" style="display:block; margin: 0 auto;"/>

### Connected Component(連通分量)
Connected Component 是點的集合，集合中任兩點都是 connected 的，同時每一個 Connected Component 都必須盡量大。如果兩個點位於不同的 Connected Component，則這兩點必不 Connected。  

如下圖的例子，顏色相同的點表示他們屬於相同的 Connected Component，可以看到若是兩個點不同顏色，那他們一定不 connected。而我們不會說紅圈圈起來的那三個點是一個Connected Component。

<img src="image/Connected_Component.JPG" width="500" style="display:block; margin: 0 auto;"/>

## AP & Bridge
 
AP 指的是一張圖 \\(G \\) 移除一個點 \\(V \\) 之後 connected component 的數量變多，則點 \\(V \\) 為 AP。例如下圖的 A 點就是 AP  
<img src="image/AP.JPG" width="500" style="display:block; margin: 0 auto;"/>

Bridge 指的是一張圖 \\(G \\) 移除一條邊 \\(E \\) 之後 connected component 的數量變多，則邊 \\(E \\) 為 Bridge。例如下圖的紅色邊就是 Bridge   
<img src="image/Bridge.JPG" width="500" style="display:block; margin: 0 auto;"/>

那我們要如何快速找到圖上所有的 AP 跟 Bridge 呢？很容易可以想到枚舉每個點或邊，把他拔掉之後看看圖上有沒有多出新的connected component。但這樣做的時間複雜度會是 \\(O((V+E)^2) \\)。不過實際上，我們只要好好觀察圖上的性質就可以在 \\(O(V+E) \\) 的時間做完！以下介紹兩種不同的方法來找出圖上所有的 AP 跟 Bridge

## Using DFS Tree for Bridge

### DFS Tree

### 觀察

我們觀察圖中那些邊絕對不可能是 bridge

<img src="image/DFS Tree Observation.JPG" width="400" style="display:block; margin: 0 auto;"/>

首先，back edge 絕對不會是 bridge。再來**如果\\( (u,v) \\)是back edge，那麼樹上 \\(u \\) 到 \\( v \\) 的路徑都不會是bridge**。例如圖中因為有\\( (F,C) \\) 這條back edge，因此樹上 \\(F \\) 到 \\(C \\) 的路徑都不會是 bridge。

所以，如果我們每遇到一條 back edge \\( (u,v) \\)，就把 Tree 上 \\(u \\) 到 \\( v \\) 的路徑都標記成不是 bridge，那麼最後那些沒被標記到的邊就會是 bridge。問題是，**我們要如何快速標記一條路徑上所有的邊？**

### 快速標記

這個問題我們可以用差分來解決。當我們遇到一條back edge \\( (u,v) \\) 時，就在\\(u \\) 上 +1， 在 \\( v \\) 上 -1，最後我們由下而上計算前綴和，\\(u \\) 到 \\( v \\) 的 path 就全部被標記好了! (補圖)

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
## Tarjan's Algorithm for AP/Bridge
接著我們要介紹 Tarjan 所提出的找 AP/Bridge 的演算法。同樣會用到 DFS Tree。

### 觀察 for AP
我們觀察下圖中 \\( C \\) 點的左子樹。 你會發現當我們移除 \\( C \\) 點後， \\( C \\) 點的左子樹**就沒有路可以走到原本 \\( C \\) 點的祖先**，因此移除\\( C \\) 點後會讓 \\( C \\) 點的左子樹跟\\( C \\) 點的祖先處在不同的 Connected Component。

<img src="image/Tarjan AP Observation.JPG" width="400" style="display:block; margin: 0 auto;"/>

更一般的說，對於一個點 \\(v \\)，如果 \\(v \\) 的某一顆子樹無法在不經過 \\(v \\) 的情況下走到 \\(v \\)的祖先，那麼 \\(v \\) 一定是 AP。

<img src="image/General AP.JPG" width="400" style="display:block; margin: 0 auto;"/>

但 root 是沒有祖先的呀，因此 root 我們要拉出來特別判斷。很明顯，當 root 有至少兩顆子樹的時候，root 一定會是 AP，否則就不是。
<img src="image/Root AP Observation.JPG" width="400" style="display:block; margin: 0 auto;"/>

### 觀察 for Bridge
我們觀察下圖中 \\( (C,D) \\) 這條邊，當我們把這條邊拔掉後，以 \\(D \\) 為根的子樹**就沒有路可以走到 \\( C \\) 點**，因此移除 \\( (C,D) \\) 後會讓\\( C \\) 跟 \\( D \\) 處在不同的 Connected Component。

<img src="image/Tarjan Bridge Observation.JPG" width="400" style="display:block; margin: 0 auto;"/>

更一般的說，令\\(u \\) 是 \\( v \\) 的 parent，則對於一條邊 \\( (u,v) \\)，如果 \\( v \\) 的子樹都無法在不經過 \\( (u,v) \\) 的情況下走到 \\( u \\)，那麼 \\( (u,v) \\) 一定是 bridge。

<img src="image/General Bridge.JPG" width="400" style="display:block; margin: 0 auto;"/>

### 演算法
現在我們來看看 Tarjan 是怎麼把這個東西做出來的。

Tarjan 首先定義了兩個函數 \\(depth \\) 跟 \\(low \\)。

\\(depth(v) \\) 表示 \\( v \\) 這個點在 DFS Tree 上的深度。
<img src="image/Depth Example.JPG" width="400" style="display:block; margin: 0 auto;"/>

\\( low(v) \\) 表示 \\(v \\) 子樹中所有的點和這些點的鄰點，以及 \\( v \\) 本身的最淺深度。

例如下圖的 \\( C \\) 點，本身的深度是 \\(3\\)，子樹中所有的點深度都 \\(>3\\)，而子樹中最淺的鄰點為 \\( B \\)，深度為 \\(2\\)，因此 \\(low(C) = 2\\) 
<img src="image/Low Example.JPG" width="400" style="display:block; margin: 0 auto;"/>

我們回想一下剛才的圖，發現對於一個點 \\(u \\)，如果他的某個子節點 \\(v \\) 滿足 \\(low(v) >= depth(u) \\) 那麼 \\(u\\)就會是 AP。

<img src="image/General AP Algo.JPG" width="400" style="display:block; margin: 0 auto;"/>

而對於一條邊 \\((u,v)\\)，如果滿足 \\(low(v) > depth(u)\\)，那麼 \\((u,v) \\) 就會是 Bridge

<img src="image/General Bridge Algo.JPG" width="400" style="display:block; margin: 0 auto;"/>

那麼，我們現在的問題只剩下如何求出 \\(low(v) \\)。

可以發現，對於一個點 \\(v \\)，如果我們知道他所有子節點的 \\(low \\) 的答案，那麼我們就可以簡單推出　\\( low(v) \\) 的答案，如下圖所示。而要達成這件事情，我們只需在 DFS 的時候用 post order 的順序計算 \\(low \\) 的答案就可以了！

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

## BCC(Biconnected Component)
BCC 指的是沒有 AP 的 Connected Component。例如下圖我們能找到三個 BCC
<img src="image/Biconnected Component.JPG" width="300" style="display:block; margin: 0 auto;"/>

## Problems

> [CSES - Necessary Cities](https://cses.fi/problemset/task/2077)
> 
> 給定一張 \\( N \\) 個點 \\( M \\) 條邊的無向圖，要你找出圖上所有的 AP

AP 模板題

> [CSES - Necessary Roads](https://cses.fi/problemset/task/2076)
> 
> 給定一張 \\( N \\) 個點 \\( M \\) 條邊的無向圖，要你找出圖上所有的 Bridge


Bridge 模板題

> [Codeforces - Break Up](https://codeforces.com/problemset/problem/700/C)
>
> 給一張帶權無向圖與兩點 \\(S \\) , \\(T \\)，要你選至多兩條邊刪除後使\\(S \\) , \\(T \\)不連通。要求選的邊權重和最小。


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
- [codeforce blog - DFS Tree](https://codeforces.com/blog/entry/68138)



