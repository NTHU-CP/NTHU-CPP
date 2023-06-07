# SCC and 2-SAT

## Introduction to Strongly Connected Component(強連通分量)
### Definition

在一張有向圖\\(G \\)中，假使兩個節點\\(u \\)跟\\(v \\)之前同時存在路徑\\(path(u,v) \\)和\\(path(v,u) \\)，則我們說\\(u \\)跟\\(v \\)屬於同一個強連通分量（簡稱SCC)中。換句話說，如果能從\\(u \\)走到\\(v \\)，也能從\\(v \\)走到\\(u \\)（相互連通），則\\(u \\)跟\\(v \\)處於同一個SCC中。

#### Property
1. 若\\(u \\)跟\\(v \\)屬於強連通分量\\(C \\)，那麼在\\(path(u,v) \\)和\\(path(v,u) \\)上的所有點，都屬於強連通分量\\(C \\)
<details><summary>Proof</summary>

設有一點\\(w \\)處於\\(path(u,v) \\)上，則可從點\\(w \\)延路徑到達點\\(v \\)，同時，因為存在\\(path(v,u) \\)，可從點\\(v \\)走到點\\(u \\)再延著\\(path(u,v) \\)到達點\\(w \\)，故點\\(w,v \\)為相互連通（點\\(u,w \\)亦然）。因此，點\\(u,w,v \\)屬於同一強連通分量。
    
</details>

2. 若\\(u \\)跟\\(v \\)互相連通，且\\(v \\)跟\\(w \\)也互相連通，則\\(u \\)跟\\(w \\)也相互連通，且\\( u, v, w \\)屬於同一強連通分量中
<details><summary>Proof</summary>
    
節點\\(v \\)跟節點\\(u,w \\)都互相連通，則可透過\\(path(u,v)+path(v, w)\\)從點\\(u \\)走到點\\(w \\)，反之亦然。因此\\(u \\)跟\\(w \\)相互連通，故\\(u,v,w \\)屬於同一強連通分量中。

</details>

3. 有向圖\\(G \\)的所有強連通分量兩兩不會有交集
<details><summary>Proof</summary>

假設存在一點\\(w \\)為SCC \\(C_1 \\)與\\(C_2 \\)的交點，則根據性質2. \\(C_1 \\)的所有點應與\\(C_2 \\)的所有點相互連通(經過\\(w \\))，\\(C_1 \\)與\\(C_2 \\)應屬於同一強連通分量，故交點\\(w \\)必不存在。
    
</details>

4. 倘若將每個SCC縮成一個點，產生的新圖為有向無環

<details><summary>Proof</summary>

假設縮點後產生的新圖\\(G' \\)存在環，則環上的點必為互相連通，應屬於同一個SCC中，經縮點操作後應合併為一個點，故圖\\G' \\)上不存在環。
    
</details>

根據以上性質，我們可以得到強連通分量(SCC)的正式定義：

對於一個有向圖\\(G \\)的強連通分量\\(C \\)：
\\(C \\)是\\(G \\)的一個極大子圖，such that \\(\forall u,v \in C, \exists path(u,v),path(v,u) \\)

為什麼找出強連通分量那麼重要呢？

因為當我們求出有向圖\\( G\\)的所有SCC之後，我們就可以藉由縮點操作，將每個SCC縮成一個點。根據性質4.，縮點操作後產生的圖\\( G' \\)會是一張有向無環圖。
變成有向無環圖之後會發現，原本難解的問題，可藉由拓樸排序或是DP等技巧，將問題化簡為多項式時間可解的問題。

#### 縮點操作：

非常暴力且樸素的作法，建造一個縮點後的圖\\( G' \\)，暴力看過原圖\\( G \\)中的所有邊，圖\\( G' \\)中只存跨越不同SCC的邊。
```
void Compress() {
    for(int u = 1;u <= n;u++) {
		for(int v : G[u]) {
			if(SCC[v] == SCC[u]) continue;
			DAG[SCC[u]].emplace_back(SCC[v]);
			//indegree_SCC[SCC[v]]++; 做拓樸排序用
		}
	}
}
```
時間複雜度: \\( O(V+E) \\)


## Tarjan's Algorithm for SCC
圖論大師Tarjan發明的求SCC算法，基於DFS Tree跟定義好的\\( L(x) \\) 跟 \\( D(x) \\) 函數 

定義：
\\( D(u) \\)為DFS造訪順序的時間戳記
\\( L(u) \\)為所有點\\( u \\)能夠走到的點中\\( D(x) \\)的最小值
 
### Detail process & Template code

Tarjan Algo code:

```
void DFS(int u, int fa) {
	D[u] = L[u] = ++Time;
	stk.emplace(u); //走過的點丟進stack裡
	inStack[u] = 1;
	for(int v : G[u]) {
		if(!D[v]) { // Tree edge
			DFS(v,u); //直接往下走
			L[u] = min(L[u], L[v]); //更新
		}
		else if (inStack[v]) { //Back edge
			L[u] = min(L[u], D[v]); //更新
		}
	}
	if(D[u] == L[u]) { //表示u的子樹存在一SCC
		int x;
		do {
			x = stk.top();
			stk.pop();
			SCC[x] = SCCID;
			inStack[x] = 0; //這個點已經隸屬於某個SCC了，從stack中pop出
		} while(x != u); //當前所有在stack中(未縮點)，都屬於此SCC中。
		++SCCID;
	}
	return ;
}
```
### Time complexity and the Correctness
只需要做一次DFS，因此時間複雜度為: \\( O(V+E) \\)，正確性可以由以下性質得出。

性質：
1. 不存在從一個SCC到另一個SCC的Back edge
<details><summary>Proof</summary>

假設SCC \\( C_u \\)中一點\\( u \\)存在一到SCC \\( C_v \\)中點\\( v \\)的Back edge，代表\\( u \\)可以透過\\( u \\)到\\( v \\)的Tree edge走到\\( v \\)，而\\( v \\)又可以透過Back edge走到\\( u \\)，兩者應屬於同一SCC中，故\\( C_u \\)與\\( C_v \\)應為同一SCC。
    
</details>

2. 可以存在SCC之間的Cross edge，但是Cross edge不會拿來更新答案

<details><summary>Proof</summary>

Cross edge會可能更新答案的情況只有:走到之前已經DFS走過的點，否則這條邊應該要屬於Tree edge。而在實做上，在DFS完前面的點之後，我們已經將SCC順便求了出來，所以前面已經形成SCC的點，不會在當前的stack中，因此不會拿來更新答案。
    
</details>

## Kosaraju's Algorithm
藉由兩次DFS求出所有SCC，好處是code比較好寫，但常數較大。

### Detail process & Template code

先對原圖跑一次DFS，用後序走訪的方式把點丟進stack裡。再把stack裡的點一個個拿出來在反圖上DFS。stack是用來維護每個點的DFS完成時間，在stack中維護完成時間從大到小。因此，此演算法的核心為： 先做一次DFS並紀錄每個點的完成時間，接著每次取完成時間最晚的點出來在反圖上做DFS，能走到的所有點就處於同一個SCC。

```
void DFS1(int u) {
  	if(visited[u]) return;
	visited[u] = true;
	for(int v : G[u]) DFS1(v);
	stk.emplace(u);
}
 
void DFS2(int u, int k) {
	if(SCC[u]) return ;
	SCC[u] = k;
	for(int v : invG[u]) DFS2(v, k);
}
```

### Time complexity and the Correctness

Kosaraju Algo分成3個部份: 
1. 原圖DFS紀錄完成時間（\\( O(V+E) \\)）
2. 建立反圖\\( G^T \\)（\\( O(V+E) \\)）
3. 反圖上DFS （\\( O(V+E) \\)）
故總時間複雜度也是\\(O(V+E))

而正確性跟以下性質有關：

性質：
1. \\( G^T\\)跟\\( G \\)的SCC集合相同

<details><summary>Proof</summary>

若在圖\\( G \\)中存在路徑\\( u \to v \\)，則在反圖\\( G^T \\)中存在路徑\\( v \to u \\)。同樣的，若在圖\\( G \\)中存在路徑\\( v \to u \\)，則在反圖\\( G^T \\)中存在路徑\\( u \to v \\)。因此，對於點\\(u \\)跟點\\(v \\)，無論在\\(G \\)還是\\(G^T \\)中，都屬於同一個SCC。
    
</details>

2. 設點\\( u \\)為SCC \\( C_u \\)中完成時間最晚的，點\\( v \\)為SCC \\( C_v \\)中完成時間最晚的，且能從\\( C_u \\)走到\\( C_v \\)，則點\\( u \\)的完成時間一定比\\( v \\)還晚。

<details><summary>Proof</summary>

可參考[Kosaraju's Proof](https://www.personal.kent.edu/~rmuhamma/Algorithms/MyAlgorithms/GraphAlgor/strongComponent.htm)
    
</details>

3. 在所有SCC中，若\\( C_u \\)的完成時間最晚，則不存在其他\\( C_v \\) s.t \\( C_v \\)能走到\\( C_u\\)。

<details><summary>Proof</summary>

從性質2可知
    
</details>

4. 每次挑完成時間最晚的點出來在反圖上DFS可以得到一組SCC。

重複4直到所有點都打上SCC編號，可得到圖\\(G \\)上的所有SCC。


## Problems
> [Planets and Kingdoms](https://cses.fi/problemset/task/1683)
> 給定一張有向圖,找出圖上各點各自屬於哪個SCC

<details><summary>Solution</summary>
    
模板題，只需要跑一次Tarjan/Kosaraju把所有點打上SCC編號就好。
    
```
#include<bits/stdc++.h>
using namespace std;
 
vector<vector<int>> G, invG;
vector<bool> visited;
stack<int> stk;
vector<int> SCC;
int SCCID = 0;
 
void DFS1(int u) {
  	if(visited[u]) return;
	visited[u] = true;
	for(int v : G[u]) DFS1(v);
	stk.emplace(u);
}
 
void DFS2(int u, int k) {
	if(SCC[u]) return ;
	SCC[u] = k;
	for(int v : invG[u]) DFS2(v, k);
}
 
signed main() {
	int n,m;cin>>n>>m;
	G.resize(n+1), invG.resize(n+1);
	visited.assign(n+1, false);
	SCC.assign(n+1, 0);
	while(m--) {
		int u,v;cin>>u>>v;
		G[u].emplace_back(v);
		invG[v].emplace_back(u);
	}
	for(int i=1;i<=n;i++) DFS1(i);
	while(!stk.empty()) {
		int u = stk.top(); stk.pop();
		if(!SCC[u]) DFS2(u, ++SCCID);
	}
  	cout<<SCCID<<'\n';
	for(int i=1;i<=n;i++) cout<<SCC[i]<<' ';
}
```
    
</details>
    
> [CSES - Reachability Queries](https://cses.fi/problemset/task/2143)
> 給定一張有向圖及\\(q \\)筆詢問，每次詢問點\\(a \\)能不能走到點\\(b \\)

<details><summary>Solution</summary>

SCC模板題
對於點\\(v \\)來說，跟它處於同一SCC中的點都是它能夠走到的。
並且，對於它能走到的其他SCC \\(C_u \\)， \\(C_u \\)中的所有點都是他能夠走到的。
因此，只需要先求出所有的SCC，做完縮點操作後拓樸排序算答案，就能算出每點能走到多少點。
這邊對每個SCC維護一個bitset，紀錄能夠走到哪些點。
這邊要注意因為拓樸排序算答案的順序關係，縮點完後的圖\\(G' \\)的邊要反著存，且會發現照著Tarjan Algo打上的SCC編號就會是一組拓樸排序，不需要再DFS一次。
    
solution code:
```
#include<bits/stdc++.h>
using namespace std;
const int N = 5e4+5;
 
int n,m,q,Time = 0,SCCID = 0;
vector<vector<int>> G(N), invDAG(N);
vector<int> low(N,0),depth(N,0),inStack(N),SCC(N,-1);
stack<int> stk;
bitset<N> canReach[N];
void DFS(int u, int fa) {
	depth[u] = low[u] = ++Time;
	stk.emplace(u);
	inStack[u] = 1;
	for(int v : G[u]) {
		if(!depth[v]) {
			DFS(v,u);
			low[u] = min(low[u], low[v]);
		}
		else if (inStack[v]) {
			low[u] = min(low[u], depth[v]);
		}
	}
	if(depth[u] == low[u]) {
		int x;
		do {
			x = stk.top();
			stk.pop();
			SCC[x] = SCCID;
			inStack[x] = 0;
		} while(x != u);
		++SCCID;
	}
	return ;
}
 
void Compress() {
    for(int u = 1;u <= n;u++) {
		for(int v : G[u]) {
			if(SCC[v] == SCC[u]) continue;
			invDAG[SCC[v]].emplace_back(SCC[u]);
		}
	}
}
 
signed main() {
	cin>>n>>m>>q;
	while(m--) {
		int u,v;cin>>u>>v;
		G[u].emplace_back(v);
	}
	for(int i = 1;i <= n;i++) if(!depth[i]) DFS(i,i);
        Compress();
	for(int i=0;i<SCCID;i++) {
		canReach[i][i] = 1;
		for(auto x : invDAG[i]) canReach[x] |= canReach[i];
	}
	while(q--) {
		int a,b;cin>>a>>b;
		cout<<(canReach[SCC[a]][SCC[b]] ? "YES\n" : "NO\n");
	}
}
```
</details>   

> [CSES - Coin Collector](https://cses.fi/problemset/task/1686)
> 有\\(n \\)個房間, \\(m \\)個單向通道，每個房間有一些金幣。問最多能收集多少金幣？(起點跟終點可以自由選擇)    

<details><summary>Solution</summary>

經典的縮點+DP
    
首先，因為金幣數一定為正整數，所以對於某個SCC來說，走遍該SCC內的點一定是最佳走法。 因此，可以先做縮點，求出每個SCC內的金幣總數。
    
縮點後，可得到一張代表所有SCC的圖\\( DAG \\)，定義\\( DP[i] \\)為：以\\( C_i \\)內的點為終點所能收集的最多金幣數。
不難看出DP轉移式為: \\( DP[v] = max\{DP[u] + Coins[v]\}, \forall u \to v \\) 
因此，就可以在縮點後的圖\\( DAG \\)上做拓樸排序的同時順便DP。

    
```
#include<bits/stdc++.h>
using namespace std;
#define int long long
const int N = 1e5+5;
 
int n,m,q,Time = 0,SCCID = 0;
vector<vector<int>> G(N), invDAG(N);
vector<int> low(N,0),depth(N,0),inStack(N),SCC(N,-1);
vector<int> coin(N,0), SCC_value(N,0);
stack<int> stk;
void DFS(int u, int fa) {
	depth[u] = low[u] = ++Time;
	stk.emplace(u);
	inStack[u] = 1;
	for(int v : G[u]) {
		if(!depth[v]) {
			DFS(v,u);
			low[u] = min(low[u], low[v]);
		}
		else if (inStack[v]) {
			low[u] = min(low[u], depth[v]);
		}
	}
	if(depth[u] == low[u]) {
		int x;
		do {
			x = stk.top();
			stk.pop();
			SCC[x] = SCCID;
			inStack[x] = 0;
		} while(x != u);
		++SCCID;
	}
	return ;
}
 
void Compress() {
    for(int u = 1;u <= n;u++) {
		SCC_value[SCC[u]] += coin[u];	
		for(int v : G[u]) {
			if(SCC[v] == SCC[u]) continue;
			invDAG[SCC[v]].emplace_back(SCC[u]);
		}
	}
}
 
signed main() {
	cin>>n>>m;
	for(int i=1;i<=n;i++) cin>>coin[i];
	while(m--) {
		int u,v;cin>>u>>v;
		G[u].emplace_back(v);
	}
	for(int i=1;i<=n;i++) if(!depth[i]) DFS(i,i);
	Compress();
	vector<int> DP(SCCID,0);
	for(int u=SCCID-1;u>=0;u--) {
		DP[u] = SCC_value[u];
		for(int v : invDAG[u]) DP[u] = max(DP[u], DP[v] + SCC_value[u]);
	}
  	int ans = 0;
	for(int i=0;i<SCCID;i++) ans = max(ans, DP[i]);
  	cout<<ans;
}
```
    
</details>
    

    
## Reference
- 清大競技程式設計二 - 上課講義
- [Strongly Connected Components - GeeksforGeeks](https://www.geeksforgeeks.org/strongly-connected-components/)
- [Tarjan’s Algorithm to find Strongly Connected Components - GeeksforGeeks](https://www.geeksforgeeks.org/tarjan-algorithm-find-strongly-connected-components/)
- [演算法筆記 - Connected Component ](https://web.ntnu.edu.tw/~algo/ConnectedComponent.html)
- [強連通分量 - OI Wiki](https://oi-wiki.org/graph/scc/)
- [CP Algorithm - strongly connected components](https://cp-algorithms.com/graph/strongly-connected-components.html)
- [Strongly Connected Component](https://www.personal.kent.edu/~rmuhamma/Algorithms/MyAlgorithms/GraphAlgor/strongComponent.htm)