# Centroid Decompostion

## Introduction
此技巧也被稱作點分治、樹分治、重心分治、重心剖分

### Centroid 重心

#### 定義
當以重心為根節點時，每個子樹的大小都不超過 \\( \frac{N}{2} \\)

#### 性質
每棵樹最多 2 個，最少 1 個重心

#### 如何求重心?

通常我們要先求出子樹大小才能找重心，所以我們可以任選一個點當根，先求出每個節點子樹大小。
接著開始從根開始遞迴檢查，遍歷所有子節點，如果有一個子節點子樹大小 \\( > frac{N}{2} \\), 則重心一定在這個個子樹裡，所以就往這個子節點遞迴。反之如果所有子節點子樹大小都不超過一半，則現在所在的點即為重心。

> [CSES Finding a Centroid](https://cses.fi/problemset/task/2079)
>
> 尋找重心練習題

<details><summary> Sample Code </summary>

```cpp
void get_sz(int u, int p) {
	sz[u] = 1;
	for (int v : g[u]) {
		if (v == p or del[v]) continue;
		get_sz(v, u);
		sz[u] += sz[v];	
	}
}
 
int get_centroid(int u, int n, int p) {
	for (int v : g[u]) {
		if (v != p and sz[v] > n / 2) 
			return get_centroid(v, n, u);
	}
	return u;
}
```

</details>

#### 重心應用

這邊分享一個還蠻有趣的題目

> [BOI Village (Maximum)](https://codeforces.com/contest/1387/problem/B2)
>
> 有一座 N 個點的樹形成鎮，每一個點住著一位居民，編號從 1 ~ N，有一天，居民們突然想要搬家，每個居民都要搬到不同的節點，但一個節點只能有一位居民，一個居民的搬家距離為舊家到新家的樹上距離，請你求出所有居民的最大搬家距離總和，且輸出一組解。
> 


<details><summary> Solution </summary>
TODO
</details>

## 重心樹

我們來看看重心究竟有什麼用途？ 

### 什麼是重心樹？

重心樹，其實就只是把重心剖分不斷找重心的過程用一棵樹表示

我們可以用遞迴定義如下

- 重心樹的根節點為整棵樹的重心
- 將重心從樹上刪除後，分為若干棵子樹，對這些子樹遞迴下去建立重心樹，並把這些子樹的重心樹根節點和被刪除的重心節點在重心樹上連邊

看圖理解一下

<p align="center">
    <img src="images/tree.gif">
</p>
<p style="text-align: center;"> 原樹 </p>

<p align="center">
    <img src="images/1.png">
</p>
<p style="text-align: center;"> 重心樹 </p>

TODO 不確定這樣寫img能不能用

### 如何實作重心樹

接著來看看如何實作建出一棵重心樹。

```cpp
vector<int> g[mxN], tree[mxN];// g存原本的樹 tree存重心樹
bitset<mxN> del; // 紀錄已被刪除的點
int pa[mxN]; // 紀錄重心樹上每個節點的父親節點
 
void get_sz(int u, int p) {
	sz[u] = 1;
	for (int v : g[u]) {
		if (v == p or del[v]) continue;
		get_sz(v, u);
		sz[u] += sz[v];	
	}
}
 
int get_centroid(int u, int n, int p) {
	for (int v : g[u]) {
		if (v != p and !del[v] and sz[v] > n / 2) 
			return get_centroid(v, n, u);
	}
	return u;
}
 
int build(int u) {
	get_sz(u, -1);	
	int centroid = get_centroid(u, sz[u], -1);
	del[centroid] = 1; // 刪除重心
	
    // 注意 u != centroid
	for (int v : g[centroid]) {
        // 根據題目進行某些操作
	}
	
	for (int v : g[centroid]) {
		if (del[v]) continue;
		int t = build(v);
		pa[t] = centroid;
		tree[centroid].pb(t);
	}
	return centroid;
}
```

注意不要走到已經刪除的節點上！

建立出重心樹其實很簡單，而且只需要 \\( O(N \log{N}) \\) 的時間複雜度(下面會證明)。

困難的地方在於重心樹性質的運用。以下是重心樹的一些重要性質以及分析。

### 重心樹性質

1. 重心樹的深度是 \\( O(\log{N}) \\)
1. 重心樹的任一子樹，在原樹中都是連通的(但並不代表在重心樹中 a 和 b 相連，原樹中 a, b也會相連)
1. 一條在原樹上 a 到 b 的路徑，可以被拆成 a→lca(a,b)→b，lca(a,b) 是 a, b 在**重心樹**上的最低公共祖先


#### 性質的說明以及證明

大家有興趣的話可以先自己花時間想想看

<details><summary> 重心樹深度證明 </summary>

首先發現，重心樹的深度其實也就是我們在建樹過程中的遞迴深度，所以可以順便得出建立重心樹的時間複雜度。

重心樹的深度是 \\( O(\log{N}) \\)，因為在我們每次遞迴，拔掉重心所形成的子樹大小都不會超過原樹的一半，所以最多經過 \\( O(\log{N}) \\) 次，所有子樹大小都會 \\( \le 1 \\)。

因為每一層(重心樹)遞迴都要對所有子樹找重心，時間複雜度 \\( O(N) \\)。

所以總時間複雜度是 \\( O(N \log{N}) \\) 。
想要更完整的證明的話，可以用主定理或是其他證明遞迴算法複雜度的方式。

</details>

TODO 其他性質的說明




> [例題 Ciel the Commander](https://codeforces.com/contest/321/problem/C)
> 有一個 n 個節點的樹，你要為每個點填入 ‘A’ 到 ‘Z’ 其中一個字母並滿足以下條件：
> 
> 任兩個有相同字母的節點 u, v，必須有一個節點 x 在 u 到 v 的路徑上，且 x 上的字母的字典序嚴格小於 u, v 上的字母，
> 請輸出一組滿足條件的解
> 
> - \\( 1 \le n \le 10^5 \\)
> 

> [例題 Xenia and Tree](https://codeforces.com/problemset/problem/342/E)
> 有一棵 \\( n \\) 個節點的樹，一 開始所有節點都是藍色，要執行 \\( m \\) 次以下兩種操作：
> 1. 將一個藍色節點塗成紅色
> 2. 查詢一個節點 \\( x \\) ，回答離 \\( x \\) 最近的紅色節點距離
> -  \\( 1 \le n,m \le 10^5 \\)
> 

> [例題 Black-White-Tree](https://csacademy.com/contest/archive/task/black-white-tree/)
> 一樣給一棵樹，每個節點可能是白色或黑色，要支援以下操作：
> 1. 改變一個節點的顏色
> 2. 查詢一個點 \\( x \\) 到其他所有相同顏色的點的距離和
> 

雖然這兩題看起來很像，但實作上有很多需要注意的地方，所以附上程式碼

<details><summary> Solution </summary>

TODO 說一下需要注意的地方

- Code
    
    ```cpp
    #define ll loli
    #include<bits/stdc++.h>
    #define pb push_back
    #define eb emplace_back
    #define push emplace
    #define lb(x, v) lower_bound(all(x), v)
    #define ub(x, v) upper_bound(all(x), v)
    #define sz(x) (int)(x.size())
    #define re(x) reverse(all(x))
    #define uni(x) x.resize(unique(all(x)) - x.begin())
    #define all(x) x.begin(), x.end()
    #define mem(x, v) memset(x, v, sizeof x); 
    #define int long long
    #define pii pair<int, int>
    #define inf 1e9
    #define INF 1e18
    #define mod 1000000007
    #define F first
    #define S second
    #define IO ios_base::sync_with_stdio(0); cin.tie(0);
    #define chmax(a, b) (a = (b > a ? b : a))
    #define chmin(a, b) (a = (b < a ? b : a))
    using namespace std;
    
    void trace_() {cerr << "\n";}
    template<typename T1, typename... T2> void trace_(T1 t1, T2... t2) {cerr << ' ' << t1; trace_(t2...); }
    #define trace(...) cerr << "[" << #__VA_ARGS__ << "] :", trace_(__VA_ARGS__);
    
    const int mxN = 5e4 + 5;
    int n, m, col[mxN];
    
    int pa[mxN], sz[mxN], sub[mxN][2], cnt[mxN][2], sum[mxN][2];
    bitset<mxN> del;
    
    // LCA euler sequence
    int st[mxN * 2][22], pos[mxN], dep[mxN], tp;
    vector<int> g[mxN];
    
    inline void build_rmq() {
    	for(int i = 0; (1<<i) < tp; i++) {
    		for(int j = 0; j + (1<<i) - 1 < tp; j++) {
    			st[j][i + 1] = min(st[j][i], st[j + (1<<i)][i]);
    		}
    	}
    }
    
    inline int query(int l, int r) {
    	int k = __lg(r - l + 1);
    	return min(st[l][k], st[r - (1<<k) + 1][k]);
    }
    
    inline int dis(int u, int v) {
    	if (pos[u] > pos[v]) swap(u, v);
    	return dep[u] + dep[v] - 2 * query(pos[u], pos[v]);	
    }
    
    void euler_dfs(int u, int p) {
    	pos[u] = tp;
    	st[tp++][0] = dep[u];	
    	for (int v : g[u]) {
    		if (v == p) continue;
    		dep[v] = dep[u] + 1;
    		euler_dfs(v, u);
    		st[tp++][0] = dep[u];
    	}
    }
    
    void get_sz(int u, int p) {
    	sz[u] = 1;
    	for (int v : g[u]) {
    		if (v == p or del[v]) continue;
    		get_sz(v, u);
    		sz[u] += sz[v];	
    	}
    }
    
    int get_centroid(int u, int n, int p) {
    	for (int v : g[u]) {
    		if (v != p and !del[v] and sz[v] * 2 > n) 
    			return get_centroid(v, n, u);
    	}
    	return u;
    }
    
    void dfs(int u, int p, int dis, int tp) {
    	sum[tp][0] += dis;	
    	cnt[tp][0]++;
    	for (int v : g[u]) {
    		if (del[v] or v == p) continue;
    		dfs(v, u, dis + 1, tp);
    	}
    }
    
    int build(int u) {
    	get_sz(u, -1);	
    	int centroid = get_centroid(u, sz[u], -1);
    	del[centroid] = 1;
    
    	for (int v : g[centroid]) {
    		if (del[v]) continue;
    		int p = sum[centroid][0];
    		dfs(v, -1, 1, centroid);
    		int t = build(v);
    		sub[t][0] = sum[centroid][0] - p;
    		//trace(sub[t][0], sum[centroid][0]);
    		pa[t] = centroid;
    	}
    	return centroid;
    }
    
    inline void update(int x) {
    	int v = x;
    	col[x] ^= 1;
    	cnt[x][col[x]]++;
    	cnt[x][col[x]^1]--;
    	while (pa[v]) {
    		int d = dis(pa[v], x), p = v;
    		v = pa[v];
    		sum[v][col[x]] += d; 
    		cnt[v][col[x]]++;
    		sum[v][(col[x]^1)] -= d; 
    		cnt[v][(col[x]^1)]--;
    
    		sub[p][col[x]] += d;
    		sub[p][(col[x]^1)] -= d;
    	}
    }
    
    inline int query(int x) {
    	int v = x, res = sum[x][col[x]];
    	while (pa[v]) {
    		int d = dis(pa[v], x);	
    		res += d * (cnt[pa[v]][col[x]] - cnt[v][col[x]]) + sum[pa[v]][col[x]] - sub[v][col[x]]; 
    		v = pa[v];
    	}
    	return res;
    }
    
    signed main() {
    	IO;	
    	cin >> n >> m;
    	vector<int> t(n + 1);
    	for (int i = 1; i <= n; i++) {
    		cin >> t[i];
    	}
    	for (int i = 1; i < n; i++) {
    		int a, b; cin >> a >> b;
    		g[a].pb(b);
    		g[b].pb(a);
    	}
    	euler_dfs(1, -1);
    	build_rmq();
    	build(1);
    	for (int i = 1; i <= n; i++) if (t[i]) update(i);		
    	
    	while (m--) {
    		int o, x;
    		cin >> o >> x;
    		if (o == 1) update(x);
    		else cout << query(x) << '\n';
    	}
    }
    ```
code
</details>
    

> [例題 Close Vertices](https://codeforces.com/problemset/problem/293/E)
>
> 給定一棵有邊權的樹和兩個變數 \\( l, w \\)
問你樹上有多少點對 \\( (u, v) \\) 滿足以下條件：
> 1. \\( (u, v) \\) 路徑上的邊數量 \\( \le l \\)
> 2. \\( (u, v) \\) 路徑上的邊權和 \\( \le w \\)
> 

> [例題 ****[JOISC2020] 首都****](https://www.luogu.com.cn/problem/P7215)
>
> 給定一棵樹，每個點上有一個 1 ~ K 的編號
希望你能選則一個編號 $c$，使得編號為 $c$ 的點都相連
你能進行合併操作，將一個編號的點全部改為另一個編號
問你最少需要進行幾次操作才能找到滿足條件的編號
>

TODO 給每一題加上solution
