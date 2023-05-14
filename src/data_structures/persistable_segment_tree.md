#  持久化線段樹
>先備知識：線段樹，一點圖論名詞

> 各位在網路搜尋資料的時候可能會出現**主席樹**這種說法，主席樹指的是持久化權值線段樹
## 引入
對於持久化線段樹，最經典的題目莫過於**區間第k小**，題目的內容是這樣的：給定一個長度為 $n$ 的整數序列，每次詢問一個閉區間 $[l,r]$ 和一個整數$k$，求在此區間裡第k小的數字是多少。

>這題雖然可以用**整體二分**來解，但是這裡我們只討論持久化線段樹相關的作法

## 什麼是權值線段樹
權值線段樹也叫值域線段樹，一般用於維護區間中數字出現的次數。其中一個應用就是求整個區間裡的第$k$小的數字是多少。

權值線段樹基本的原理就是：每個葉節點存的是其對應的索引的出現次數，而其他的節點則是存下面節點的和。

#### 如何查詢第 $k$ 小

由於每個節點是存的是區間和，從根節點開始，我們每次只要判斷左子樹的和是否大於等於 $k$，如果是的話表示答案在左子樹，往左子樹找。如果左子樹的和小於 $k$ ，表示答案在右子樹，先把 $k$ 減掉左子樹的和，再往右子樹找答案，直到找到葉節點，所對應的索引值就是答案。

```cpp=
int query_kth(int l, int r, int rt, int k){
    if(l == r) return l;
    int sum = seg[rt << 1], mid = l + r >> 1;
    if(sum >= k) return query_kth(l, mid, rt<<1, k);
    return query_kth(mid+1, r, rt<<1|1, k-sum);
}
```



## 什麼是持久化線段樹
持久化就是保存歷史版本，我們每次在線段樹上修改某個東西就會新建一個版本，這樣如果我們要知道修改之前資料的樣子是什麼，就可以跳回那個版本。

對線段樹上進行持久化，最簡單的想法就是每次修改（單點修改）的時候，直接複製一整顆線段樹，但這樣的話太浪費記憶體空間了。
 
仔細回想一下在線段樹的單點修改，會發現其實每次有改到的點只有根到葉子的那一條鏈，而其他點的值都不變。

（修改第13個點的值）
![](https://i.imgur.com/dvOTjzK.png)


所以我们只要新建一條鏈出來，並把沒有修改的部分連到上一個版本的樹就好了。


![](https://i.imgur.com/bosOj5z.png)

## 回到題目

現在回到題目上，如果我們把區間這個條件修改一下，每次詢問 $[1, r]$ 的第 $k$ 小，我們就只要找到第$r$ 個版本，利用權值線段樹求出來就可以了


那對於區間 $[l,r]$，我們可以利用前綴和的特性，用第 $r$ 個版本的線段樹減掉第 $l-1$ 個版本就可以求出 $[l,r]$ 這個區間的權值線段樹的樣子，這樣答案就出來了。

有一點要注意的是，持久化線段樹需要使用動態開點來實作，如果使用僞指標的話就要特別注意一下記憶體的大小，如果有 $n$ 次修改，那每次修改需要增加 $\lceil log_2{n} \rceil + 1$ 個點，線段樹總空間會到 $2n -1 + n(\lceil log_2{n} \rceil + 1)$。 依照題目大小帶入數字 $n \leq 2\times10^5$， $2 \times 2\times10^5 -1 + 2\times10^5 \times 19$ 約等於 $21 \times 2\times10^5$，因此空間需要開到 $21$ 倍，而因爲題目一般情況下不會特別限制空間大小，所以爲了方便都會寫成 `n<<5`，即開到32倍。

附上 [洛谷 P3834](https://www.luogu.com.cn/problem/P3834) 的code

```cpp=
#include<bits/stdc++.h>
using namespace std;

const int maxn=200001;
int n, m, ls[maxn<<5], rs[maxn<<5], sum[maxn<<5], a[maxn], b[maxn], root[maxn], cnt;
void build(int l, int r, int &o){
    sum[o = ++cnt] = 0;
    int mid=l+r>>1;
    if(l == r) return;
    build(l, mid, ls[o]);
    build(mid+1, r, rs[o]);
}
void add(int l, int r, int &o, int lst, int x){
    o=++cnt;
    ls[o] = ls[lst];
    rs[o] = rs[lst];
    sum[o] = sum[lst]+1;
    int mid = l + r >> 1;
    if(l == r) return;
    if(x <= mid) 
        add(l, mid, ls[o], ls[lst], x);
    else 
        add(mid+1, r, rs[o], rs[lst], x);
}
int query(int L, int R, int l, int r, int k){
    if(l == r) return l;
    int x = sum[ls[R]] - sum[ls[L]], mid= l + r >>1;
    if(x >= k) 
        return query(ls[L], ls[R], l, mid, k);
    else 
        return query(rs[L], rs[R], mid+1, r, k-x);
}
int main(){
    ios::sync_with_stdio(0),cin.tie(0),cout.tie(0);
    cin >> n >> m;
    for(int i = 1; i <= n; ++i) 
        cin>>a[i], b[i]=a[i];
    sort(b + 1, b + 1 + n);
    int len = unique(b + 1, b + 1 + n) - b - 1;
    build(1, len, root[0]);
    for(int i = 1;i <= n; ++i)
        add(1, len, root[i], root[i-1], lower_bound(b+1,b+1+len,a[i]) - b);
    for(int l, r, k; m--;){
        cin >> l >> r >> k;
        cout << b[query(root[l-1],root[r],1,len,k)] << "\n";
    }
    return 0;
}
```

## 其他練習題

### [TIOJ 1827 Yet another simple task \^____\^](https://tioj.ck.tp.edu.tw/problems/1827)

#### 題目敘述


給定一個長度為 $N$ 的整數序列 $B$。有 $M$ 筆詢問，對於每組詢問 $(P, K)$，求最小的答案 $S$ 滿足至少有 $K$ 個元素滿足以下條件

- $|P - i| \leq S$
- $B_i \leq S$

$1 \leq N, M \leq 10^5$
$1 \leq K, B_i \leq N$
$0 \leq P \lt N$

#### 想法
我們可以二分搜 $S$ ，至於二分搜的判斷就是要求出區間 $[P - S, P +S]$ 中小於等於 $S$ 的元素有多少個，這個問題我們可以用持久化線段樹解決。

時間複雜度：$Nlog{N} + Mlog^2{N}$

#### AC code
```cpp 
#include<bits/stdc++.h>
//#include<bits/extc++.h>
//using namespace __gnu_pbds;
using namespace std;
#define F first
#define S second
typedef long long ll;
#define REP(i,n) for(register int i=0;i<(n);++i)
#define REP1(i,a,b) for(register int i=(a);i<=(b);++i)
#define em emplace_back
#define mkp make_pair
#define pb push_back
#define ALL(x) (x).begin(),(x).end()
#define pii pair<int,int>
#define pll pair<ll,ll>
#define Rosario ios::sync_with_stdio(0),cin.tie(0),cout.tie(0);
//#define lb(x) (x&-x)

const int z=1e5+5;
int n,m,rt[z],cnt,lc[z*70],rc[z*70],sum[z*70];
void build(int l,int r,int &o){
	if(!o) o=++cnt;
	sum[o]=0;
	if(l==r) return;
	int mid=l+r>>1;
	build(l,mid,lc[o]),build(mid+1,r,rc[o]);
}
void mdf(int prv,int &o,int l,int r,int x){
	o=++cnt;
	lc[o]=lc[prv], rc[o]=rc[prv], sum[o]=sum[prv]+1;
	if(l==r) return;
	int mid=l+r>>1;
	if(x<=mid) mdf(lc[prv],lc[o],l,mid,x);
	else mdf(rc[prv],rc[o],mid+1,r,x);
}
int qry(int l,int r,int o,int ql,int qr){
	if(ql<=l&&r<=qr) return sum[o];
	int mid=l+r>>1,s=0;
	if(ql<=mid) s=qry(l,mid,lc[o],ql,qr);
	if(qr>mid) s+=qry(mid+1,r,rc[o],ql,qr);
	return s;
}
int get_(int l,int r,int k){
	int res=qry(1,n,rt[r],1,k)-qry(1,n,rt[l-1],1,k);
	return res;
}
signed main(){Rosario
	cin>>n>>m;
	build(1,n,rt[0]);
	for(int x,i=1;i<=n;++i) cin>>x,mdf(rt[i-1],rt[i],1,n,x);
	for(int mid,l,r,p,k,o;m;--m){
		cin>>p>>k; ++p;
		l=1,r=n;
		while(l<r){
			mid=l+r>>1;
			if(get_(max(1,p-mid),min(n,p+mid),mid)<k) l=mid+1;
			else r=mid;
		}
		cout<<l<<"\n";
	}
	return 0;
}
```

### [洛谷 P2617 Dynamic Rankings](https://www.luogu.com.cn/problem/P2617)

#### 題目敘述

裸裸的動態區間第 $k$ 小，支援**單點修改**以及**詢問區間第 $k$ 小**兩種操作。

#### 想法
這題其實也可以用整體二分來解，但本章節不討論此解法。

仔細思考發現靜態的區間第 $k$ 小利用的是前綴和，然後透過差分來求出答案，那有什麽東西時候可以支援單點修改又支援區間求和的呢？ 就是我們可愛的**樹狀數組**，所以只要樹狀數組套權值線段樹就好啦。

#### AC code
```cpp=
#include<bits/stdc++.h>
using namespace std;
#define lb(x) x&-x

const int maxn=100001;
int n, m, ls[maxn*400], rs[maxn*400], sum[maxn*400], a[maxn], b[maxn << 1], root[maxn], cnt, len;
vector<int> ql, qr;
struct query{
    int op, x, y, z;
}q[maxn];
void mdf(int l, int r, int x, int &o, int val){
    if(!o) o = ++cnt;
    sum[o] += val;
    if(l == r) return;
    int mid = l + r >> 1;
    if(x <= mid) mdf(l, mid, x, ls[o], val);
    else mdf(mid+1, r, x, rs[o], val);
}
void mdf_all(int x, int val){
    int idx = lower_bound(b+1, b+1+len, a[x]) - b;
    for(;x <= n; x += lb(x)) mdf(1, len, idx, root[x], val);
}
int qry(int l, int r, int k){
    if(l == r) return l;
    int mid = l + r >> 1, S = 0;
    for(auto &i:ql) S -= sum[ls[i]];
    for(auto &i:qr) S += sum[ls[i]]; 
    if(k <= S){
        for(int i=0,j=ql.size();i<j;++i) ql[i] = ls[ql[i]];
        for(int i=0,j=qr.size();i<j;++i) qr[i] = ls[qr[i]];
        return qry(l, mid, k);
    }else{
        for(int i=0,j=ql.size();i<j;++i) ql[i] = rs[ql[i]];
        for(int i=0,j=qr.size();i<j;++i) qr[i] = rs[qr[i]];
        return qry(mid+1, r, k - S);
    }
}
int qry_all(int l, int r, int k){
    ql.clear(), qr.clear();
    for(; r; r -= lb(r)) qr.emplace_back(root[r]);
    for(--l; l; l -= lb(l)) ql.emplace_back(root[l]);
    return qry(1, len, k);
}
int main(){
    ios::sync_with_stdio(0),cin.tie(0),cout.tie(0);
    cin >> n >> m;
    for(int i = 1; i <= n; ++i) 
        cin>>a[i], b[++len]=a[i];
    for(int i = 1; i <= m; ++i){
        char c; cin>>c;
        q[i].op = c=='Q';
        if(c == 'Q') cin >> q[i].x >> q[i].y >> q[i].z;
        else cin >> q[i].x >> q[i].y, b[++len]=q[i].y;
    }
    sort(b + 1, b + 1 + len);
    len = unique(b + 1, b + 1 + len) - b - 1;
    for(int i = 1; i <= n; ++i) mdf_all(i, 1);
    for(int i = 1; i <= m; ++i){
        if(q[i].op) cout << b[qry_all(q[i].x, q[i].y, q[i].z)] << "\n";
        else{
            mdf_all(q[i].x, -1);
            a[q[i].x] = q[i].y;
            mdf_all(q[i].x, 1);
        }
    }
    return 0;
}



```

### [洛谷 P2633 Count on a tree](https://www.luogu.com.cn/problem/P2633)

#### 題目敘述
給定一棵 $n$ 個節點的樹，每個點有一個權值。有 $m$ 個詢問，每次給你 $u,v,k$，你需要回答 $u \text{ xor last}$ 和 $v$ 這兩個節點間第 $k$ 小的點權。  

其中 $\text{last}$ 是上一個詢問的答案，定義其初始為 $0$。

#### 想法
轉換一下題目會發現這是區間第 $k$ 小的樹上版本：每次詢問樹上某兩點的路徑上第 $k$ 小的值。

考慮持久化線段樹有前綴和的性質，我們可以利用樹上差分來解這題，我們定義 $p[u]$ 代表從根節點到點 $u$ 的持久化線段樹版本，那在點 $u$ 到點 $v$ 路徑之間的 "持久化線段樹" 就是

$p[u] + p[v] - p[LCA(u,v)] - p[fa[LCA(u, v)]]$

只要用上面的公式算出相對應的持久化線段樹，接下來的詢問就是套模板了。

#### AC code
```cpp=
//#pragma GCC optimize("Ofast,unroll-loops,no-stack-protector,fast-math")
#include<bits/stdc++.h>
//#include<bits/extc++.h>
//using namespace __gnu_pbds;
using namespace std;
typedef long long ll;
#define F first
#define S second
#define REP(i,n) for(int i=0;i<(n);++i)
#define REP1(i,a,b) for(int i=(a);i<=(b);++i)
#define em emplace_back
#define ALL(x) (x).begin(),(x).end()
#define mkp make_pair
#define pii pair<int,int>
#define pll pair<ll,ll>
#define Rosario ios::sync_with_stdio(0),cin.tie(0),cout.tie(0);
//#define lb(x) (x&-x)

const int z=1e5+5,Z=z*120;
struct edge{int to,nxt;}e[z<<1];
int tot=0,cnt=0,h[z],n,m,dep[z],lca[20][z],a[z],b[z],rs[Z],ls[Z],val[Z],len,p[z];
void add(int u,int v){
    e[++cnt]=edge{v,h[u]};
    h[u]=cnt;
}
void build_lca(){REP1(j,1,17) REP1(i,1,n) lca[j][i]=lca[j-1][lca[j-1][i]];}
int LCA(int a,int b){
    if(dep[a]<dep[b]) swap(a,b);
    for(int i=dep[a]-dep[b],k=0;i;i>>=1,++k) if(i&1) a=lca[k][a];
    if(a==b) return a;
    for(int i=17;~i;--i) if(lca[i][a]!=lca[i][b]) a=lca[i][a],b=lca[i][b];
    return lca[0][a];
}
void build_tree(int l,int r,int &rt){
    rt=++tot; val[rt]=0;
    if(l==r) return;
    int mid=l+r>>1;
    build_tree(l,mid,ls[rt]),build_tree(mid+1,r,rs[rt]);
}
void inse(int l,int r,int x,int &rt,int pre){
    rt=++tot; rs[rt]=rs[pre],ls[rt]=ls[pre],val[rt]=val[pre]+1;
    if(l==r) return;
    int mid=l+r>>1;
    if(x<=mid) inse(l,mid,x,ls[rt],ls[pre]);
    else inse(mid+1,r,x,rs[rt],rs[pre]);
}
void dfs(int x,int pa){
    inse(1,len,lower_bound(b+1,b+1+len,a[x])-b,p[x],p[pa]);
    for(int i=h[x];i;i=e[i].nxt){
        int v=e[i].to;
        if(dep[v]) continue;
        dep[v]=dep[x]+1; lca[0][v]=x;
        dfs(v,x);
    }
}
int query(int l,int r,int A,int B,int lc,int fa,int k){
    if(l==r) return l;
    int x=val[ls[A]]+val[ls[B]]-val[ls[lc]]-val[ls[fa]],mid=l+r>>1;
    if(k<=x) return query(l,mid,ls[A],ls[B],ls[lc],ls[fa],k);
    return query(mid+1,r,rs[A],rs[B],rs[lc],rs[fa],k-x);
}
int main(){Rosario
    cin>>n>>m;
    REP1(i,1,n)cin>>a[i],b[i]=a[i];
    sort(b+1,b+1+n); len=unique(b+1,b+1+n)-b-1;
    int x,y,k,last=0;
    REP(i,n-1) cin>>x>>y,add(x,y),add(y,x);
    dep[1]=1;  build_tree(1,len,p[0]); dfs(1,0); build_lca();
    while(m--){
        cin>>x>>y>>k;
        x^=last;
        int lc=LCA(x,y),fa=lca[0][lc];
        last=b[query(1,len,p[x],p[y],p[lc],p[fa],k)];
        cout<<last<<"\n"; 
    }
    return 0;
}
```

### [Codefroces 1171F](https://codeforces.com/problemset/problem/1771/F)
待补
#### 題目敘述

#### 想法


### [洛谷 P3168 [CQOI2015] 任务查询系统](https://www.luogu.com.cn/problem/P3168)

待补
#### 題目敘述

#### 想法

```cpp=
//#pragma GCC optimize("Ofast,unroll-loops,no-stack-protector,fast-math")
#include<bits/stdc++.h>
//#include<bits/extc++.h>
//using namespace __gnu_pbds;
using namespace std;
#define F first
#define S second
typedef long long ll;
#define REP(i,n) for(register int i=0;i<(n);++i)
#define REP1(i,a,b) for(register int i=(a);i<=(b);++i)
#define em emplace_back
#define mkp make_pair
#define pb push_back
#define ALL(x) (x).begin(),(x).end()
#define pii pair<int,int>
#define pll pair<ll,ll>
#define Rosario ios::sync_with_stdio(0),cin.tie(0),cout.tie(0);
//#define lb(x) (x&-x)

#define int ll
const int z=1e5+5;
int sz[z<<6],sum[z<<6],rt[z],lc[z<<6],rc[z<<6],n,m,cnt,b[z];
void mdf(int l,int r,int x,int v,int &o,int pre){
    o=++cnt;
    lc[o]=lc[pre],rc[o]=rc[pre],sz[o]=sz[pre],sum[o]=sum[pre];
    sum[o]+=v*b[x]; sz[o]+=v;
    if(l==r) return;
    int mid=l+r>>1;
    if(x<=mid) mdf(l,mid,x,v,lc[o],lc[pre]);
    else mdf(mid+1,r,x,v,rc[o],rc[pre]);
}
int qry(int o,int k,int l,int r){
    if(l==r) return sum[o]/sz[o]*k;
    int mid=l+r>>1;
    if(sz[lc[o]]>=k) return qry(lc[o],k,l,mid);
    else return sum[lc[o]]+qry(rc[o],k-sz[lc[o]],mid+1,r);
}
vector<pii> tme[z];
signed main(){Rosario
    cin>>n>>m;
    for(int l,r,x,i=1;i<=n;++i){
        cin>>l>>r>>x; b[i]=x;
        tme[l].em(x,1);
        tme[r+1].em(x,-1);
    }
    sort(b+1,b+1+n); int len=unique(b+1,b+1+n)-b-1;
    REP1(i,1,m){
        rt[i]=rt[i-1];
        for(auto &j:tme[i]){
            int pos=lower_bound(b+1,b+1+len,j.F)-b;
            mdf(1,len,pos,j.S,rt[i],rt[i]);
        }
    }
    int pre=1;
    for(int a,b,c,x,i=0;i<m;++i){
        cin>>x>>a>>b>>c;
        pre=1+(a*pre+b)%c;
        if(pre>=sz[rt[x]]) pre=sum[rt[x]];
        else pre=qry(rt[x],pre,1,len);
        cout<<pre<<"\n";
    }
    return 0;
}
```




## References
https://blog.csdn.net/yanweiqi1754989931/article/details/117380913

https://oi-wiki.org/ds/persistent-seg/

https://tioj.ck.tp.edu.tw/problems/1827

https://www.luogu.com.cn/problem/P2617

https://www.luogu.com.cn/problem/P2633

https://www.luogu.com.cn/problem/P3168

https://codeforces.com/problemset/problem/1771/F
