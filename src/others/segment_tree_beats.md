# Introduction to Segment Tree Beats
## 前言
Segment Tree Beats(簡稱STB)是改良版本的線段樹，可以解決提升線段樹的效率。STB進行區間查詢和區間修改等基本操作時，可在保持時間複雜度不變的情況下，簡化不必要的操作。
## 建造樹節點
STB的每一個節點劃分成
* 總和(sum)
* 最大值(max1)、次大值(max2)、最大值次數(maxc)
* 最小值(min1)、次小值(min2)、最小值次數(minc)
* 懶惰標記(lazy)
```cpp
using ll = long long;
ll A[MAXN];
struct node {
    ll sum; // 總和
    ll max1; // 最大值
    ll max2; // 次大值
    ll maxc; // 最大值次數
    ll min1; // 最小值
    ll min2; // 次小值
    ll minc; // 最小值次數
    ll lazy; // 懶惰標記
} T[MAXN * 4];
```
## 建造STB
對root進行呼叫，不斷對兩個子節點進行呼叫，達成遞迴建樹。當左右邊界相等，表示當前節點為葉節點，總和(sum)、最大值(max1)、最小值(min1)是節點本身的值。<br>
時間複雜度是$O(n)$。
```cpp
// 節點t的區間範圍是[tl, tr]
void build(int t = 1, int tl = 0, int tr = N - 1) {
	T[t].lazy = 0;
	if(tl == tr) {
		T[t].sum = T[t].max1 = T[t].min1 = A[tl];
		T[t].maxc = T[t].minc = 1;
		T[t].max2 = -llINF;
		T[t].min2 = llINF;
		return;
	}
	int tm = (tl + tr) >> 1;
	build(t << 1, tl, tm);
	build(t << 1 | 1, tm + 1, tr);
	merge(t);
}
```
## 區間修改
Update函數主要是針對一段目標範圍，進行區間修改。由Push函數更新子節點的數值。
按照不同功能，劃分成
* update_add / push_add
* update_max / push_max
* update_min / push_min
* push_down
### update_add
對目標區間[l, r]所有數值都增加v。假設目標區間範圍是[l, r]，變化值是v，當前節點的區間範圍是[tl, tr]。
* 如果節點區間完全不在目標區間內，就不需要更新。
* 如果節點區間完全在目標區間內，直接對當前節點進行push_add操作。
* 否則不斷對兩個子節點進行更新操作，最後用兩個子節點的數值更新當前節點的數值。
```cpp
const ll llINF = LLONG_MAX;
// 目標區間範圍是[l, r]，變化值是v，節點t的區間範圍是[tl, tr]
void update_add(int l, int r, ll v, int t = 1, int tl = 0, int tr = N - 1) {
	if(r < tl || tr < l) { return; }
	if(l <= tl && tr <= r) {
		push_add(t, tl, tr, v);
		return;
	}
	push_down(t, tl, tr);
	int tm = (tl + tr) >> 1;
	update_add(l, r, v, t << 1, tl, tm);
	update_add(l, r, v, t << 1 | 1, tm + 1, tr);
	merge(t);
}
```
### push_add
對一段範圍加上一常數時，更新子節點的數值。假設節點t的區間範圍是[tl, tr]，變化值是v。如果變化值為0，就不需要更新。從l到r每個元素都增加v，所以sum會增加(r - l + 1) * v。最大值、最小值、懶惰標記直接增加v。次大值、次小值則需要先判斷存在，再增加v。
```cpp
// 節點t的區間範圍是[tl, tr]，變化值是v
void push_add(int t, int tl, int tr, ll v) {
	if(v == 0) { return; }
	T[t].sum += (tr - tl + 1) * v;
	T[t].max1 += v;
	if(T[t].max2 != -llINF) { T[t].max2 += v; }
	T[t].min1 += v;
	if(T[t].min2 != llINF) { T[t].min2 += v; }
	T[t].lazy += v;
}
```
### update_max
對目標區間[l, r]所有比v大的值都替換為v。假設目標區間範圍是[l, r]，更新值是v，當前節點的區間範圍是[tl, tr]。
* 如果節點區間完全不在目標區間內，或更新值大於等於最大值(max1)，就不需要更新。
* 如果節點區間完全在目標區間內，且次大值(max2)小於更新值，直接對當前節點進行push_max操作。
* 否則不斷對兩個子節點進行更新操作，最後用兩個子節點的數值更新當前節點的數值。
```cpp
// 目標區間範圍是[l, r]，更新值是v，節點t的區間範圍是[tl, tr]
void update_max(int l, int r, ll v, int t = 1, int tl = 0, int tr = N - 1) {
	if(r < tl || tr < l || v >= T[t].max1) { return; }
	if(l <= tl && tr <= r && v > T[t].max2) {
		push_max(t, v, tl == tr);
		return;
	}
	push_down(t, tl, tr);
	int tm = (tl + tr) >> 1;
	update_max(l, r, v, t << 1, tl, tm);
	update_max(l, r, v, t << 1 | 1, tm + 1, tr);
	merge(t);
}
```
### push_max
更新子節點的最大值。假設節點t的更新值是v，l代表是否為葉節點。如果更新值大於等於最大值(max1)，就不需要更新。將最大值更新為v，所以先從sum扣除本來的max1 * maxc，再增加v * maxc。接著處理最小值、次小值。如果當前節點是葉節點，最小值直接設為v。否則更新最小值、次小值。
```cpp
// 節點t的更新值是v，l代表是否為葉節點，l是true表示當前節點是葉節點
void push_max(int t, ll v, bool l) {
	if(v >= T[t].max1) { return; }
	T[t].sum -= T[t].max1 * T[t].maxc;
	T[t].max1 = v;
	T[t].sum += T[t].max1 * T[t].maxc;
	if(l) {
        T[t].min1 = T[t].max1;
	} else {
		if(v <= T[t].min1) {
			T[t].min1 = v;
		} else if(v < T[t].min2) {
			T[t].min2 = v;
		}
	}
}
```
### update_min
對目標區間[l, r]所有比v小的值都替換為v。假設目標區間範圍是[l, r]，更新值是v，當前節點的區間範圍是[tl, tr]。
* 如果節點區間完全不在目標區間內，或更新值小於等於最小值(min1)，就不需要更新。
* 如果節點區間完全在目標區間內，且次小值(min2)大於更新值，直接對當前節點進行push_min操作。
* 否則不斷對兩個子節點進行更新操作，最後用兩個子節點的數值更新當前節點的數值。
```cpp
// 目標區間範圍是[l, r]，更新值是v，節點t的區間範圍是[tl, tr]
void update_min(int l, int r, ll v, int t = 1, int tl = 0, int tr = N - 1) {
	if(r < tl || tr < l || v <= T[t].min1) { return; }
	if(l <= tl && tr <= r && v < T[t].min2) {
		push_min(t, v, tl == tr);
		return;
	}
	push_down(t, tl, tr);
	int tm = (tl + tr) >> 1;
	update_min(l, r, v, t << 1, tl, tm);
	update_min(l, r, v, t << 1 | 1, tm + 1, tr);
	merge(t);
}
```
### push_min
更新子節點的最小值。假設節點t的更新值是v，l代表是否為葉節點。如果更新值小於等於最小值(min1)，就不需要更新。將最小值更新為v，所以先從sum扣除本來的min1 * minc，再增加v * minc。接著處理最大值、次大值。如果當前節點是葉節點，最大值直接設為v。否則更新最小值、次小值。
```cpp
// 節點t的更新值是v，l代表是否為葉節點，l是true表示當前節點是葉節點
void push_min(int t, ll v, bool l) {
	if (v <= T[t].min1) { return; }
	T[t].sum -= T[t].min1 * T[t].minc;
	T[t].min1 = v;
	T[t].sum += T[t].min1 * T[t].minc;
	if(l) {
		T[t].max1 = T[t].min1;
	} else {
		if(v >= T[t].max1) {
			T[t].max1 = v;
		} else if(v > T[t].max2) {
			T[t].max2 = v;
		}
	}
}
```
### push_down
對一段範圍進行區間操作時，將當前節點的懶惰標記(lazy)、最大值(max1)、最小值(min1)向下更新到子節點。假設節點t的區間範圍是[tl, tr]。如果當前節點是葉節點，就不需要向下更新。
```cpp
// 節點t的區間範圍是[tl, tr]
void push_down(int t, int tl, int tr) {
	if(tl == tr) return;
	
	int tm = (tl + tr) >> 1;
	push_add(t << 1, tl, tm, T[t].lazy);
	push_add(t << 1 | 1, tm + 1, tr, T[t].lazy);
	T[t].lazy = 0;

	push_max(t << 1, T[t].max1, tl == tm);
	push_max(t << 1 | 1, T[t].max1, tm + 1 == tr);
	
	push_min(t << 1, T[t].min1, tl == tm);
	push_min(t << 1 | 1, T[t].min1, tm + 1 == tr);
}
```
## Merge函數
整合兩個子節點的數值，更新當前節點的數值。劃分成3個部分分別處理總和、最大值、最小值。
* 總和為左子節點總和加上右子節點總和。
* 如果左右子節點最大值相同，最大值為左子節點最大值(max1)，次大值為左右子節點次大值較大者，最大值次數為左右子節點最大值次數(maxc)相加。如果左子節點最大值較大，最大值為左子節點最大值，次大值為max(左子節點次大值, 右子節點最大值)，最大值次數為左子節點最大值次數。如果右子節點最大值較大，最大值為右子節點最大值，次大值為max(左子節點最大值, 右子節點次大值)，最大值次數為右子節點最大值次數。
* 如果左右子節點最小值相同，最小值為左子節點最小值(min1)，次小值為左右子節點次小值較小者，最小值次數為左右子節點最小值次數(minc)相加。如果左子節點最小值較小，最小值為左子節點最小值，次大值為min(左子節點次小值, 右子節點最小值)，最小值次數為左子節點最小值次數。如果右子節點最小值較小，最小值為右子節點最小值，次小值為min(左子節點最小值, 右子節點次小值)，最小值次數為右子節點最小值次數。
```cpp
// 節點t
void merge(int t) {
	
	T[t].sum = T[t << 1].sum + T[t << 1 | 1].sum;

	if (T[t << 1].max1 == T[t << 1 | 1].max1) {
		T[t].max1 = T[t << 1].max1;
		T[t].max2 = max(T[t << 1].max2, T[t << 1 | 1].max2);
		T[t].maxc = T[t << 1].maxc + T[t << 1 | 1].maxc;
	} else {
		if (T[t << 1].max1 > T[t << 1 | 1].max1) {
			T[t].max1 = T[t << 1].max1;
			T[t].max2 = max(T[t << 1].max2, T[t << 1 | 1].max1);
			T[t].maxc = T[t << 1].maxc;
		} else {
			T[t].max1 = T[t << 1 | 1].max1;
			T[t].max2 = max(T[t << 1].max1, T[t << 1 | 1].max2);
			T[t].maxc = T[t << 1 | 1].maxc;
		}
	}

	if (T[t << 1].min1 == T[t << 1 | 1].min1) {
		T[t].min1 = T[t << 1].min1;
		T[t].min2 = min(T[t << 1].min2, T[t << 1 | 1].min2);
		T[t].minc = T[t << 1].minc + T[t << 1 | 1].minc;
	} else {
		if (T[t << 1].min1 < T[t << 1 | 1].min1) {
			T[t].min1 = T[t << 1].min1;
			T[t].min2 = min(T[t << 1].min2, T[t << 1 | 1].min1);
			T[t].minc = T[t << 1].minc;
		} else {
			T[t].min1 = T[t << 1 | 1].min1;
			T[t].min2 = min(T[t << 1].min1, T[t << 1 | 1].min2);
			T[t].minc = T[t << 1 | 1].minc;
		}
	}
}
```
## 區間查詢
對一段範圍進行區間總和查詢。假設目標區間範圍是[l, r]，節點t的區間範圍是[tl, tr]。
* 如果節點區間完全不在目標區間內，就回傳0。
* 如果節點區間完全在目標區間內，直接回傳當前節點總和(sum)。
* 否則將當前節點的懶惰標記(lazy)、最大值(max1)、最小值(min1)向下更新到子節點，接著不斷對兩個子節點進行區間查詢總和，最後用兩個子節點的總和作為查詢的答案。
```cpp
// 目標區間範圍是[l, r]，節點t的區間範圍是[tl, tr]
ll query_sum(int l, int r, int t = 1, int tl = 0, int tr = N - 1) {
	if (r < tl || tr < l) { return 0; }
	if (l <= tl && tr <= r) { return T[t].sum; }
	push_down(t, tl, tr);
	int tm = (tl + tr) >> 1;
	return query_sum(l, r, t << 1, tl, tm) +
	       query_sum(l, r, t << 1 | 1, tm + 1, tr);
}
```
# Basic Segment Tree Beats Problems
## RMQ
要進行區間最大值查詢，將query_sum改成return T[t].max1即可。
```cpp
// 目標區間範圍是[l, r]，節點t的區間範圍是[tl, tr]
ll query_max(int l, int r, int t = 1, int tl = 0, int tr = N - 1) {
	if (r < tl || tr < l) { return 0; }
	if (l <= tl && tr <= r) { return T[t].max1; }
	push_down(t, tl, tr);
	int tm = (tl + tr) >> 1;
	return query_max(l, r, t << 1, tl, tm) +
	       query_max(l, r, t << 1 | 1, tm + 1, tr);
}
```
## For all i in [l, r], change $A_i$ to $x$
對一段範圍進行區間修改。
* 如果節點區間完全不在目標區間內，就不需要更新。
* 如果節點區間完全在目標區間內，直接將節點的總和(sum)、最大值(max1)設為v。
* 不斷對兩個子節點進行更新操作，最後用兩個子節點的數值更新當前節點的數值。
```cpp
void update_set(int l, int r, ll v = x, int t = 1, int tl = 1, int tr = N) {
	if (r < tl || tr < l) { return; }
    if (tl == tr) {
		T[t].sum = T[t].max = v;
		return;
	}
	int tm = (tl + tr) / 2;
	update_mod(l, r, v, t * 2, tl, tm);
	update_mod(l, r, v, t * 2 + 1, tm + 1, tr);
	merge(t);
}
```
## For all i in [l, r], change $A_i$ to $A_i$ + $x$
跟上一題一樣，差在更新的方式是T[t].sum += v、T[t].max += v。
```cpp
void update_set(int k, ll v = x, int t = 1, int tl = 1, int tr = N) {
    if(r < tl || tr < l) { return; }
	if(tl == tr) {
		T[t].sum += v;
        T[t].max += v;
		return;
	}
	int tm = (tl + tr) / 2;
	if(k <= tm) {
		update_set(k, v, t * 2, tl, tm);
	} else {
		update_set(k, v, t * 2 + 1, tm + 1, tr);
	}
	merge(t);
}
```
## For all i in [l, r], change $A_i$ to max($A_i$ - $x$, 0)
跟上一題一樣，差在更新的方式是T[t].sum = max(T[t].sum - v, 0)、T[t].max = max(T[t].max - v, 0);
<details>
<summary>Solution</summary>
```cpp
void update_set(int k, ll v = x, int t = 1, int tl = 1, int tr = N) {
    if(r < tl || tr < l) { return; }
	if(tl == tr) {
		T[t].sum = max(T[t].sum - v, 0);
        T[t].max = max(T[t].max - v, 0);
		return;
	}
	int tm = (tl + tr) / 2;
	if(k <= tm) {
		update_set(k, v, t * 2, tl, tm);
	} else {
		update_set(k, v, t * 2 + 1, tm + 1, tr);
	}
	merge(t);
}
```
</details>

## [Codeforces 438D The Child and Sequence][1]
[1]: https://codeforces.com/contest/438/problem/D
給定一個序列$a_i$，並對其進行$m$筆操作。操作有3種：
1. 給定l, r，請輸出區間總和a[l]+...+a[r]
2. 給定l, r, 對區間[l, r]的每一個數取模，即$a_i$ = $a_i$ $mod$ $x$
3. 給定k, x，將a[k]的值改為x

我們在這題只需要總和(sum)、最大值(max)。
update_mod處理操作2，update_set處理操作3。
因為操作3只更改a[k]的值，update_set可以進一步優化為判斷k與當前節點區間中點位置tm的大小關係，如果k小於等於tm，只更新左子節點。否則只更新右子節點。
```cpp
#include <bits/stdc++.h>
using namespace std;
using ll = long long;

const int MAXN = 100001;

int N;
ll a[MAXN];

struct node {
    ll sum; // 總和
    ll max; // 最大值
} T[MAXN * 4];

void merge(int t) {
	T[t].sum = T[t << 1].sum + T[t << 1 | 1].sum;
	T[t].max = max(T[t << 1].max, T[t << 1 | 1].max);
}

void build(int t = 1, int tl = 0, int tr = N - 1) {
	if (tl == tr) {
		T[t].sum = T[t].max = a[tl];
		return;
	}
	int tm = (tl + tr) >> 1;
	build(t << 1, tl, tm);
	build(t << 1 | 1, tm + 1, tr);
	merge(t);
}

void update_mod(int l, int r, ll v, int t = 1, int tl = 1, int tr = N) {
	if (r < tl || tr < l || v > T[t].max) { return; }
    if (tl == tr) {
		int val = T[t].max % v;
		T[t].sum = T[t].max = val;
		return;
	}
	int tm = (tl + tr) / 2;
	update_mod(l, r, v, t * 2, tl, tm);
	update_mod(l, r, v, t * 2 + 1, tm + 1, tr);
	merge(t);
}

void update_set(int k, ll v, int t = 1, int tl = 1, int tr = N) {
    if(r < tl || tr < l) { return; }
	if(tl == tr) {
		T[t].sum = T[t].max = v;
		return;
	}
	int tm = (tl + tr) / 2;
	if(k <= tm) {
		update_set(k, v, t * 2, tl, tm);
	} else {
		update_set(k, v, t * 2 + 1, tm + 1, tr);
	}
	merge(t);
}

ll query(int l, int r, int t = 1, int tl = 1, int tr = N) {
	if(r < tl || tr < l) { return 0; }
	if(l <= tl && tr <= r) { return T[t].sum; }
	int tm = (tl + tr) / 2;
	return query(l, r, t * 2, tl, tm) +
           query(l, r, t * 2 + 1, tm + 1, tr);
}

int main() {
    int Q;
	cin >> N >> Q;
	for (int i = 1; i <= N; i++) { cin >> a[i]; }
    build();
	while(Q--) {
		int t;
		cin >> t;
		if (t == 1) {
			int l, r;
			cin >> l >> r;
			cout << query(l, r) << '\n';
		} else if (t == 2) {
			int l, r;
			ll x;
			cin >> l >> r >> x;
			update_mod(l, r, x);
		} else if (t == 3) {
			int k;
			ll x;
			cin >> k >> x;
			update_set(k, x);
		}
	}
}
```
# Advanced Segment Tree Beats Problems
## [洛谷P6242 【模板】線段樹 3][2]
[2]: https://www.luogu.com.cn/problem/P6242
給定一個序列$A_i$、輔助序列$B_i$，並對其進行$m$筆操作。操作有3種：
1. 給定l, r, k，對區間[l, r]的每一個數加上k，即$A_i$ = $A_i$ + $k$
2. 給定l, r, v, 對區間[l, r]的每一個數取最小值，即$A_i$ = min($A_i$, $v$)
3. 給定l, r, 請輸出區間總和a[l]+...+a[r]
4. 給定k, x，對於區間[l, r]的每個i，請輸出$A_i$的最大值
5. 給定l, r，對於區間[l, r]的每個i，請輸出$B_i$的最大值
每次操作後，將$B_i$設為max($B_i$, $A_i$)
* 1 $\leq$ n, m $\leq$ 500000
```cpp
#include <bits/stdc++.h>
using namespace std;
using ll = long long;

const int N = 200021;
int n, m, A[N];

struct SegmentTreeBeats {
    struct node {
        int mx_a, smx, cnt, mx_b;
        ll sum, add_a1, add_a2, add_b1, add_b2;
    } t[N<<2];

    void pushup(int x) {
        t[x].mx_a = max(t[x*2].mx_a, t[x*2+1].mx_a);
        t[x].mx_b = max(t[x*2].mx_b, t[x*2+1].mx_b);
        t[x].sum = t[x*2].sum + t[x*2+1].sum;
        if(t[x*2].mx_a == t[x*2+1].mx_a) t[x].smx = max(t[x*2].smx, t[x*2+1].smx);
        else if(t[x*2].mx_a > t[x*2+1].mx_a) t[x].smx = max(t[x*2].smx, t[x*2+1].mx_a);
        else t[x].smx = max(t[x*2].mx_a, t[x*2+1].smx);
        t[x].cnt = (t[x*2].mx_a >= t[x*2+1].mx_a) * t[x*2].cnt + (t[x*2+1].mx_a >= t[x*2].mx_a) * t[x*2+1].cnt;
    }

    void addtag(int x, int l, int r, ll A, ll b, ll c, ll d) {
        t[x].sum += A*t[x].cnt + c*(r-l+1-t[x].cnt);
        t[x].mx_b = max((ll)t[x].mx_b, t[x].mx_a + b);
        t[x].add_b1 = max(t[x].add_b1, t[x].add_a1 + b);
        t[x].add_b2 = max(t[x].add_b2, t[x].add_a2 + d);
        t[x].mx_a += A, t[x].add_a1 += A, t[x].add_a2 += c;
        if(t[x].smx > -Inf) t[x].smx += c;
    }

    void pushdown(int x, int l, int r) {
        int mid = (l+r)>>1, mx = max(t[x*2].mx_a, t[x*2+1].mx_a);
        ll add_a1 = t[x].add_a1, add_a2 = t[x].add_a2, add_b1 = t[x].add_b1, add_b2 = t[x].add_b2;
        if(t[x*2].mx_a == mx) addtag(x*2, l, mid, add_a1, add_b1, add_a2, add_b2);
        else addtag(x*2, l, mid, add_a2, add_b2, add_a2, add_b2);
        if(t[x*2+1].mx_a == mx) addtag(x*2+1, mid+1, r, add_a1, add_b1, add_a2, add_b2);
        else addtag(x*2+1, mid+1, r, add_a2, add_b2, add_a2, add_b2);
        t[x].add_a1 = t[x].add_b1 = t[x].add_a2 = t[x].add_b2 = 0;
    }

    void build(int x, int l, int r) {
        if(l == r){
            t[x].sum = t[x].mx_a = t[x].mx_b = A[l];
            t[x].smx = -Inf, t[x].cnt = 1;
            return;
        }
        int mid = (l+r)>>1;
        build(x*2, l, mid), build(x*2+1, mid+1, r);
        pushup(x);
    }

    void update_add(int x, int l, int r, int L, int R, int k) {
        if(l >= L && r <= R){ addtag(x, l, r, k, k, k, k); return; }
        int mid = (l+r)>>1;
        pushdown(x, l, r);
        if(mid >= L) update_add(x*2, l, mid, L, R, k);
        if(mid < R) update_add(x*2+1, mid+1, r, L, R, k);
        pushup(x);
    }

    void update_min(int x, int l, int r, int L, int R, int val) {
        if(t[x].mx_a <= val) return;
        if(l >= L && r <= R && t[x].smx < val){ addtag(x, l, r, val-t[x].mx_a, val-t[x].mx_a, 0, 0); return; }
        int mid = (l+r)>>1;
        pushdown(x, l, r);
        if(mid >= L) update_min(x*2, l, mid, L, R, val);
        if(mid < R) update_min(x*2+1, mid+1, r, L, R, val);
        pushup(x);
    }

    ll query(int x, int l, int r, int L, int R, int id) {
        if(l >= L && r <= R) return id == 1 ? t[x].sum : (id == 2 ? t[x].mx_a : t[x].mx_b);
        int mid = (l+r)>>1;
        pushdown(x, l, r);
        ll A = (mid >= L) ? query(x*2, l, mid, L, R, id) : (id == 1 ? 0 : -Inf);
        ll b = (mid < R) ? query(x*2+1, mid+1, r, L, R, id) : (id == 1 ? 0 : -Inf);
        return (id == 1 ? A+b : max(A, b));
    }
} T;

int main() {
    cin>>n>>m;
    for(int i = 1; i <= n; i++) cin>>A[i];
    T.build(1, 1, n);
    int type, l, r, k;
    while(m--) {
        cin>>type>>l>>r;
        if(type == 1) {
            cin>>k, T.update_add(1, 1, n, l, r, k);
        } else if(type == 2) {
            cin>>k, T.update_min(1, 1, n, l, r, k);
        } else {
            cout<< T.query(1, 1, n, l, r, type-2) <<endl;
        }
    }
}

```
## [Codeforces 1290E Cartesian Tree][3]
[3]: https://codeforces.com/problemset/problem/1290/E
給定一個長度為n的序列$a_i$，對於區間[1, n]的每個k，用前k小的值建造笛卡爾樹，求所有子樹大小的總和。
* 1 $\leq$ n $\leq$ 150000

$l_i$, $r_i$分別為$a_i$左右最後一個比其小的數的位置，節點i的子樹大小為$r_i$ - $l_i$ + 1。分開維護$l_i$和$r_i$，以$r_i$為例，當k增加1，k+1插入序列時，$pos_{k+1}$左側的$r_i$被更新成min($r_i$, $pos_{k+1}$)，右側的$r_i$被更新成$r_i$+1。可以使用STB維護區間總和。<br>
時間複雜度是$O(nlog^2n)$。
```cpp
#include <bits/stdc++.h>
using namespace std;
using ll = long long;

const int N = 160000;

int n;
int a[N], pos[N];

struct STB {
    struct node {
        ll sum;
        ll max1, max2, cnt, num, tag = -1;
        ll lazy;
    } T[N<<2];

    void merge(int t) {
        T[t].max1 = max(T[t*2].max1, T[t*2+1].max1);
        T[t].num = T[t*2].num + T[t*2+1].num;
        T[t].sum = T[t*2].sum + T[t*2+1].sum;
        if(T[t*2].max1 == T[t*2+1].max1) {
            T[t].max2 = max(T[t*2].max2, T[t*2+1].max2);
        }
        else if(T[t*2].max1 > T[t*2+1].max1) {
            T[t].max2 = max(T[t*2].max2, T[t*2+1].max1);
        }
        else {
            T[t].max2 = max(T[t*2].max1, T[t*2+1].max2);
        }
        T[t].cnt = (T[t*2].max1 >= T[t*2+1].max1) * T[t*2].cnt + (T[t*2+1].max1 >= T[t*2].max1) * T[t*2+1].cnt;
    }

    void push_add(int t, int k) {
        T[t].sum += (ll)k * T[t].num;
        if(T[t].num) {
            T[t].max1 += k;
            T[t].lazy += k;
            if(T[t].max2 > 0) T[t].max2 += k;
            if(~T[t].tag) T[t].tag += k;
        }
    }

    void push_min(int t, int k) {
        if(T[t].max1 <= k) { return; }
        T[t].sum -= (ll)T[t].cnt * (T[t].max1 - k);
        T[t].max1 = k;
        T[t].tag = k;
    }

    void push_down(int t) {
        if(T[t].lazy) {
            push_add(t*2, T[t].lazy);
            push_add(t*2+1, T[t].lazy);
        }
        if(~T[t].tag) {
            push_min(t*2, T[t].tag);
            push_min(t*2+1, T[t].tag);
        }
        T[t].tag = -1, T[t].lazy = 0;
    }

    void insert(int t, int tl, int tr, int pos, int val) {
        if(tl == tr) {
            T[t].sum = T[t].max1 = val;
            T[t].cnt = T[t].num = 1;
            return;
        }
        int tm = (tl+tr)>>1;
        push_down(t);
        if(tm >= pos) insert(t*2, tl, tm, pos, val);
        else insert(t*2+1, tm+1, tr, pos, val);
        merge(t);
    }

    void update(int l, int r, int k, int id, int t = 1, int tl = 1, int tr = n) {
        if(l > r) return;
        if(id && T[t].max1 <= k) return;
        if(l <= tl && tr <= r){
            if(id && T[t].max2 < k) {
                push_min(t, k);
                return;
            }
            if(!id) {
                push_add(t, k);
                return;
            }
        }
        int tm = (tl+tr)>>1;
        push_down(t);
        if(tm >= l) update(l, r, k, id, t*2, tl, tm);
        if(tm < r) update(l, r, k, id, t*2+1, tm+1, tr);
        merge(t);
    }
} LB, RB;

struct Fenwick {
    int t[N];
    void update(int pos, int k) {
        while(pos <= n) t[pos] += k, pos += (pos&-pos);
    }
    int get(int pos) {
        int ret = 0;
        while(pos) ret += t[pos], pos -= (pos&-pos);
        return ret;
    }
} T;

int main() {
    cin>>n;
    for(int i = 1; i <= N; i++) cin>>a[i], pos[a[i]] = i;

    for(int i = 1; i <= N; i++) {
        RB.update(pos[i]+1, n, 1, 0);
        RB.update(1, pos[i]-1, T.get(pos[i]), 1);
        LB.update(1, n-pos[i]+1, -1, 0);
        LB.update(1, n-pos[i]+1, n-T.get(pos[i])-1, 1);
        RB.insert(1, 1, n, pos[i], i);
        LB.insert(1, 1, n, n-pos[i]+1, n);
        cout<< RB.T[1].sum - ((ll)i*(n+1) - LB.T[1].sum) + i <<"\n";
        T.update(pos[i], 1);
    }
}
```
## [Codeforces 1572F Stations][4]
[4]: https://codeforces.com/problemset/problem/1572/F
給定n座城市，每座城市有2個屬性$h_i$, $w_i$，第i座城市可以向第j座城市廣播，如果對於所有的k在區間[i, j]中，滿足max{$h_k$} < $h_i$。一開始$h_i$ = 0且$w_i$ = i。
其發生q次事件，事件有2種：
1. 給定$c_i$, $g_i$, 將城市$c_i$的h變成全局嚴格最大值，並修改其w為$g_i$
2. 給定$l_i$, $r_i$, 請輸出區間[$l_i$, $r_i$]的每座城市可以覆蓋到的城市數量$b_i$之總和
* 1 $\leq$ n $\leq$ 200000

每個城市覆蓋的區域是一個從i開始的區間，而維護區間右端點是一個單點修改，區間取min問題，另外用一棵線段樹維護區間覆蓋，因為STB裡可直接對值進行修改，所以修改時可以順帶在另一棵線段樹上進行區間修改，即打tag時維護即可。詢問時直接在線段樹查詢區間和。
時間複雜度是$O(nlog^2n)$。
```cpp
#include <bits/stdc++.h>
using namespace std;
using ll = long long;

const int N = 200021;

int n, q;

struct SegmentTree{
    struct node {
        ll sum;
        ll lazy;
    } t[N<<2];

    void push_down(int x, int l, int r) {
        int mid = (l+r)>>1;
        t[x*2].sum += (mid-l+1) * t[x].lazy, t[x*2].lazy += t[x].lazy;
        t[x*2+1].sum += (r-mid) * t[x].lazy, t[x*2+1].lazy += t[x].lazy;
        t[x].lazy = 0;
    }

    void update(int x, int l, int r, int L, int R, ll k) {
        if(L > R) return;
        if(l >= L && r <= R){ t[x].sum += (r-l+1) * k, t[x].lazy += k; return; }
        int mid = (l+r)>>1;
        push_down(x, l, r);
        if(mid >= L) update(x*2, l, mid, L, R, k);
        if(mid < R) update(x*2+1, mid+1, r, L, R, k);
        t[x].sum = t[x*2].sum + t[x*2+1].sum;
    }

    ll get(int x, int l, int r, int L, int R) {
        if(l >= L && r <= R) return t[x].sum;
        int mid = (l+r)>>1; ll ret = 0;
        push_down(x, l, r);
        if(mid >= L) ret += get(x*2, l, mid, L, R);
        if(mid < R) ret += get(x*2+1, mid+1, r, L, R);
        return ret;
    }
} BIT;

struct Segment_Tree_Beats {
    struct node {
        int mx, smx, cnt, tag = -1;
    } t[N<<2];

    void push_up(int x) {
        t[x].mx = max(t[x*2].mx, t[x*2+1].mx);
        if(t[x*2].mx == t[x*2+1].mx) t[x].smx = max(t[x*2].smx, t[x*2+1].smx);
        else if(t[x*2].mx > t[x*2+1].mx) t[x].smx = max(t[x*2].smx, t[x*2+1].mx);
        else t[x].smx = max(t[x*2].mx, t[x*2+1].smx);
        t[x].cnt = (t[x*2].mx >= t[x*2+1].mx) * t[x*2].cnt + (t[x*2+1].mx >= t[x*2].mx) * t[x*2+1].cnt;
    }

    void addtag(int x, int k, bool frt) {
        if(t[x].mx <= k) return;
        if(frt) BIT.update(1, 1, n, k+1, t[x].mx, -t[x].cnt);
        t[x].mx = t[x].tag = k;
    }

    void push_down(int x){
        if(~t[x].tag)
            addtag(x*2, t[x].tag, 0), addtag(x*2+1, t[x].tag, 0);
        t[x].tag = -1;
    }

    void insert(int x, int l, int r, int pos, int val){
        if(l == r){
            BIT.update(1, 1, n, l, t[x].mx, -t[x].cnt), BIT.update(1, 1, n, l, val, 1);
            t[x].mx = val, t[x].cnt = 1; return;
        }
        int mid = (l+r)>>1;
        push_down(x);
        if(mid >= pos) insert(x*2, l, mid, pos, val);
        else insert(x*2+1, mid+1, r, pos, val);
        push_up(x);
    }

    void update(int x, int l, int r, int L, int R, int k) {
        if(t[x].mx <= k || L > R) return;
        if(l >= L && r <= R && t[x].smx < k){ addtag(x, k, 1); return; }
        int mid = (l+r)>>1;
        push_down(x);
        if(mid >= L) update(x*2, l, mid, L, R, k);
        if(mid < R) update(x*2+1, mid+1, r, L, R, k);
        push_up(x);
    }
} T;

int main() {
    cin>>n>>q;
    for(int i = 1; i <= n; i++) T.insert(1, 1, n, i, i);
    int type, c, g, l, r;
    while(q--) {
        cin>>type;
        if(type == 1) {
            cin>>c>>g;
            T.insert(1, 1, n, c, g), T.update(1, 1, n, 1, c-1, c-1);
        } else cin>>l>>r, cout<< BIT.get(1, 1, n, l, r) <<"\n";
    }
}
```
# Reference
* https://codeforces.com/blog/entry/57319
* https://usaco.guide/adv/segtree-beats?lang=cpp
* https://www.cnblogs.com/Neal-lee/p/15695984.html
* https://mzhang2021.github.io/cp-blog/historic-segtree