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

因為當我們求出有向圖\\( G\\)的所有SCC之後，我們就可以藉由縮點操作，將每個SCC縮成一個點。根據性質4.，縮點操作後產生的圖\\( G'\\)會是一張有向無環圖。
變成有向無環圖之後會發現，原本難解的問題，可藉由拓樸排序或是DP等技巧，將問題化簡為多項式時間可解的問題。

縮點操作code：
```
void Compress() {
    for(int u = 1;u <= n;u++) {
		for(int v : G[u]) {
			if(SCC[v] == SCC[u]) continue;
			DAG[SCC[u]].emplace_back(SCC[v]);
			indegree_SCC[SCC[v]]++;
		}
	}
}
```


## Tarjan's Algorithm for SCC
圖論大師Tarjan發明的求SCC算法，基於DFS Tree跟定義好的low & depth 陣列

### Detail process & Template code

Tarjan Algo code:

```
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
```
### Time complexity and the Correctness
待補

## Kosaraju's Algorithm
藉由兩次DFS求出所有SCC，好處是code比較好寫，但常數較大。

### Detail process & Template code

```
void DFS1(int u){
	visited[u]=true;
	for(auto v:G[u]){
		if(!visited[v])dfs(v);
	}
	st[top++]=u;
}
void DFS2(int u,int k){
	if(SCC[u]) return;
	contract[u] = k; /*u屬於第k個SCC*/
	for(auto v:invG[u]){
		DFS2(v,k);
	}
}
```

### Time complexity and the Correctness
待補

## 2-SAT problem

### Definition 
SAT(Boolean satisfiability problem)中文為滿足性問題，給定一個布林運算式，想辦法assign給每個布林變數一個值(True of False)，使得該運算式為True。

而K-SAT問題為：規定運算式的所有子句，最多只能有K個變數

對於K > 2，K-SAT問題已被證明為NP-Complete。
而對於2-SAT問題則存在多項式時間的算法

### Transfer 2-SAT to Graph Problem
待補
### Krom's Algorithm
待補
## Problems
> [CSES - Reachable Nodes](https://cses.fi/problemset/task/2138)
> 給定一張有向圖，求圖上每一節點能走到的點數（含自身）
<details><summary>Solution</summary>

SCC模板題
對於點\\(v \\)來說，跟它處於同一SCC中的點都是它能夠走到的。
並且，對於它能走到的其他SCC \\(C_u \\)， \\(C_u \\)中的所有點都是他能夠走到的。
因此，只需要先求出所有的SCC，做完縮點操作後拓樸排序算答案，就能算出每點能走到多少點。
這邊要注意因為拓樸排序順序的關係，縮點完後的圖\\(G' \\)的邊要反著存。
    
solution code:
```
#include<bits/stdc++.h>
using namespace std;
const int N = 5e4+5;
 
int n,m,Time = 0,SCCID = 0;
vector<vector<int>> G(N), invDAG(N);
vector<int> low(N,0),depth(N,0),inStack(N),SCC(N,-1);
stack<int> stk;
vector<int> indegree_SCC(N,0);
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
			indegree_SCC[SCC[u]]++;
		}
	}
}
 
void topological_sort() {
	queue<int> Q;
	for(int i = 0;i < SCCID;i++) {
		canReach[i][i] = 1;
		if(indegree_SCC[i] == 0) Q.emplace(i);
	}
	while(!Q.empty()) {
		int u = Q.front(); Q.pop();
		for(int v : invDAG[u]) {
			canReach[v] |= canReach[u];
			indegree_SCC[v]--;
			if(indegree_SCC[v] == 0) Q.emplace(v);
		}
	}
}
 
signed main() {
	cin>>n>>m;
	while(m--) {
		int u,v;cin>>u>>v;
		G[u].emplace_back(v);
	}
	for(int i = 1;i <= n;i++) if(!depth[i]) DFS(i,i);
	Compress();
	topological_sort();
	for(int i=1;i<=n;i++) cout<<canReach[SCC[i]].count()<<' ';
} 
```
</details>
    
> [CSES - Reachability Queries](https://cses.fi/problemset/task/2143)
> 給定一張有向圖及\\(q \\)筆詢問，每次詢問點\\(a \\)能不能走到點\\(b \\)

<details><summary>Solution</summary>

這題也是SCC模板題，作法跟上題基本上一模一樣。
    
</details>    

> [CSES - Flight Routes Check](https://cses.fi/problemset/task/1682)
> 

> [Planets and Kingdoms](https://cses.fi/problemset/task/1683)
> 

- [Giant Pizza](https://cses.fi/problemset/task/1684)
- [The Door Problem](https://codeforces.com/contest/776/problem/D)
-  hdu 3062

## Reference
- 清大競技程式設計二 - 上課講義
- [Strongly Connected Components - GeeksforGeeks](https://www.geeksforgeeks.org/strongly-connected-components/)
- [演算法筆記 - Connected Component ](https://web.ntnu.edu.tw/~algo/ConnectedComponent.html)
- [強連通分量 - OI Wiki](https://oi-wiki.org/graph/scc/)
- [CP Algorithm - strongly connected components](https://cp-algorithms.com/graph/strongly-connected-components.html)
