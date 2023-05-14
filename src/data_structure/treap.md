# 樹堆 (Treap)

## 概念

\\(Treap = Binary\ Search\ Tree\ +\ Heap\\)
### Binary Search Tree

對於任何一顆二元搜尋樹，都會滿足以下性質
* 若任意節點的左子樹不空，則左子樹上所有節點的值均小於它的根節點的值
* 若任意節點的右子樹不空，則右子樹上所有節點的值均大於它的根節點的值
* 任意節點的左、右子樹也分別為二元搜尋樹

### Heap

對於任何一顆Heap，都會滿足以下性質
* 任何一個父節點他的子節點的值都會小於等於(或大於等於)父節點的值

而我們可以利用Heap的性質搭配隨機產生的pri值去讓整個Treap的高度達到\\(\log(n)\\)的期望高度

### 請相信treap

![](https://i.imgur.com/c4apmmM.png)

圖片來源: https://www.luogu.com.cn/blog/85514/fhq-treap-xue-xi-bi-ji

## 無旋轉Treap (Split-Merge Treap)

### 概念

### 節點

```cpp!
struct node{
    node *l = NULL,*r = NULL;
    T key;
    int pri = rand();
    node(T x):key(x){}
    ~node(){
        for(auto &i:{l,r})
            delete i;
    }
};
```

### 合併

`merge(node *a,node *b)`
依照Heap的性質，去合併兩個節點，這裡是以最大堆為示範

```cpp!
node *merge(node *a,node *b){
    if(!a or !b)return a?:b;
    if(a->pri>b->pri){
        a->r = merge(a->r,b);
        return a;
    }
    else{
        b->l = merge(a,b->l);
        return b;
    }
}
```

### 分裂

`split_by_key(node *t,int k,node *&a,node *&b)`
依照key(k)的大小，去將一個Treap t分裂成兩個節點a,b，滿足$a.key\le k$，$b.key>k$。

```cpp!
void split_by_key(node *t,T k,node *&a,node *&b){
    if(!t){a = b = NULL;return;};
    if(k<=t->key){
        b = t;
        split_by_key(t->l,k,a,b->l);
    }
    else{
        a = t;
        split_by_key(t->r,k,a->r,b);
    }
}
```

### 插入

插入一個key(k)，並且還要維持BST的性質，可以先用split分裂出兩個Treap a,b，滿足$a.key\le k$，$b.key\gt k$，然後再新增一個節點，節點的key為k，將該節點和a合併，再和b節點合併，這樣就可以維持$a.key\le k\lt b.key$了。

```cpp!
void insert(node *&t,T k){
    node *a,*b;
    split_by_key(t,k,a,b);
    t = merge(merge(a,new node(k)),b);
}
```

### 刪除

刪除跟插入類似，依照key(k)去分裂出三個Treap a,b,c，滿足\\(a.key < b.key = k < c.key \\)，然後刪掉b就好了。

```cpp!
void erase(node *&t,T k){
    node *a,*b,*c;
    split_by_key(t,k-1,a,b);
    split_by_key(b,k,b,c);
    t = merge(a,c);
    delete b;
}
```

### 序列第k小

前面的`split_by_key`可以依照key(k)的大小分裂出兩個Treap a,b，其中$a.key\le k$，而order_of_key就是要詢問Treap a的大小是多少，所以我們可以在節點的部分多維護一個size

為了避免讀取到空指標的size成員，我們可以寫一個函式來判斷該指標是否為空，如果是的話就回傳0，否則回傳該指標的size

而`find_by_order`就是找第k小的值，對於一個Treap t，如果t的左子樹大小+1等於k的話，那t的key就是第k小的值，如果左子樹大小>=k的話，就遞迴往左去找第k小，否則就往右去找。

整合的程式碼如下

```cpp!=
template<class T>struct Treap{
	struct node{
		node *l = NULL,*r = NULL;
		T key;
		int pri = rand(),sz = 1;
		node(T x):key(x){}
		~node(){
			for(auto &i:{l,r})
				delete i;
		}
		void pull(){
			sz = 1;
			for(auto i:{l,r})
				if(i)sz+=i->sz;
		}
	};
	node *root = NULL;
	node *merge(node *a,node *b){
		if(!a or !b)return a?:b;
		if(a->pri>b->pri){
			a->r = merge(a->r,b);
			a->pull();
			return a;
		}
		else{
			b->l = merge(a,b->l);
			b->pull();
			return b;
		}
	}
	void split_by_key(node *t,T k,node *&a,node *&b){
		if(!t){a = b = NULL;return;};
		if(t->key<=k){
			a = t;
			split_by_key(t->r,k,a->r,b);
			a->pull();
		}
		else{
			b = t;
			split_by_key(t->l,k,a,b->l);
			b->pull();
		}
	}
	int size(node *t){
		return t?t->sz:0;
	}
	int size(){
		return size(root);
	}
	void insert(T k){
		node *a,*b;
		split_by_key(root,k,a,b);
		root = merge(merge(a,new node(k)),b);
	}
	void erase(T k){
		node *a,*b,*c;
		split_by_key(root,k-1,a,b);
		split_by_key(b,k,b,c);
		root = merge(a,c);
		delete b;
	}
	T find_by_order(node *t,int x){
		if(size(t->l)+1==x)return t->key;
		if(size(t->l)>=x)return find_by_order(t->l,x);
		return find_by_order(t->r,x-size(t->l)-1);
	}
	T find_by_order(int x){
		return find_by_order(root,x);
	}
	int order_of_key(node *t,T k){
		if(t->key==k)return size(t->l)+1;
		if(t->key<k)return size(t->l)+1+order_of_key(t->r,k);
		return size(t->l,k);
	}
	int order_of_key(T k){
		return order_of_key(root,k);
	}
};
```

~~然後你會發現這些操作內建pbds的tree都有支援，完全不必手刻~~

### Range Queries

上面我們多維護了一個size就可以知道一顆Treap的大小，就算是split或是merge都可以透過pull這個函式去好好地維護，那假如我們今天想維護的是常見的區間和、區間最大值、區間最小值呢？一樣可以透過類似的方式去維護，這裡以最小值來做示範。

```cpp!
struct node{
    node *l = NULL,*r = NULL;
    T key,mn;
    int pri = rand(),sz = 1,lz = 0;
    node(T x):key(x),mn(x){}
    ~node(){
        for(auto &i:{l,r})
            delete i;
    }
    void pull(){
        sz = 1,mn = key;
        for(auto i:{l,r})
            if(i)sz+=i->sz,mn = min(mn,i->mn);
    }
};
```

因為我們現在是要對序列做操作，所以不再用序列的值去滿足二元搜尋樹性質，而是用序列的$index$。
而一個值的$index$就是他前面有幾個數字，這裡我們可以透過左子樹的size去得知，那一顆Treap該如何按照$index$插入一個值呢？這裡要介紹一個操作叫`split_by_size`，也就是按照大小去將一顆Treap分裂成兩顆。

這裡為了方便就直接把函式取為`split`，因為當Treap應用在序列操作的時候就已經不需要`split_by_key`這個東西了。

```cpp!
void split(node *t,int k,node *&a,node *&b){
    if(!t){a = b = NULL;return;}
    if(size(t->l)+1<=k){
        a = t;
        split(t->r,k-size(t->l)-1,a->r,b);
        a->pull();
    }
    else{
        b = t;
        split(t->l,k,a,b->l);
        b->pull();
    }
}
```

而區間詢問就是把包含那個區間的Treap給分裂出來，然後詢問那顆Treap的最小值是多少就好了，因為在split時pull就已經幫我們維護了Treap的最小值了，詢問完後再把Treap給merge回去就好。

```cpp!
T query_min(int l,int r){
    node *a,*b,*c;
    split(root,l,a,b);
    split(b,r-l+1,b,c);
    T ans = b->mn;
    root = merge(a,merge(b,c));
    return ans;
}
```

單點修改就是直接把那個位置的Treap給分裂出來，然後修改那個點的key之後再merge回去。

```cpp!
void update(int x,T k){
    node *a,*b,*c;
    split(root,x,a,b);
    split(b,1,b,c);
    b->key = k;
    root = merge(a,merge(b,c));
}
```

### Lazy Tag

如果要區間修改的話，那我們就在節點多紀錄一個懶人標記，懶人標記代表的是：目前這個Treap的值還沒被真的修改，但如果要進行詢問時，就把懶人標記往下推，順便修改他的值。

為了達到往下推這個動作，我們在多寫一個`push`函式。

這裡用區間加值做示範

```cpp!
struct node{
    node *l = NULL,*r = NULL;
    T key,sum,add = 0;
    int pri = rand(),sz = 1;
    node(T x):key(x),sum(x){}
    ~node(){
        for(auto &i:{l,r})
            delete i;
    }
    void push(){
        if(!add)return;
        sum+=add*sz,key+=add;
        for(auto &i:{l,r})
            if(i)i->add+=add;
        add = 0;
    }
    void pull(){
        sz = 1,sum = key;
        for(auto i:{l,r})
            if(i)sz+=i->sz,sum+=i->sum;
    }
};
```

有了push之後，就要記得在split跟merge之前都先把懶標給push下去
因為我們幾乎每個操作都是仰賴split跟merge這兩個函式，所以只要在這兩個函式進行push跟pull就可以好好地維護了。

```cpp!
node *merge(node *a,node *b){
    if(!a or !b)return a?:b;
    if(a->pri>b->pri){
        a->push();
        a->r = merge(a->r,b);
        a->pull();
        return a;
    }
    else{
        b->push();
        b->l = merge(a,b->l);
        b->pull();
        return b;
    }
}
void split(node *t,int k,node *&a,node *&b){
    if(!t){a = b = NULL;return;}
    t->push();
    if(size(t->l)+1<=k){
        a = t;
        split(t->r,k-size(t->l)-1,a->r,b);
        a->pull();
    }
    else{
        b = t;
        split(t->l,k,a,b->l);
        b->pull();
    }
}
```

而區間加值就是把那個區間的Treap給分裂出來，然後加上值後打上懶標merge回去就好了

```cpp!
void update(int l,int r,T k){
    node *a,*b,*c;
    split(root,l,a,b);
    split(b,r-l+1,b,c);
    b->add+=k;
    root = merge(a,merge(b,c));
}
```

### 區間翻轉

上述這些操作線段樹都可以做到，那Treap有哪些操作是線段樹做不到的呢？有！就是經典的區間翻轉。
由於一顆Treap可以透過swap交換兩顆子樹，然後一樣用打懶標的方式，讓子樹的子樹也進行交換，這樣就可以達到區間翻轉了。
要注意翻轉的懶標比較特別，rev = 1/0，代表轉 or 不轉。
push跟區間翻轉的程式碼如下

#### push
```cpp!
void push(){
    if(!rev)return;
    swap(l,r);
    for(auto &i:{l,r})
        if(i)i->rev^=1;
    rev = 0;
}
```

#### reverse

```cpp!
void reverse(int l,int r){
    node *a,*b,*c;
    split(root,l,a,b);
    split(b,r-l+1,b,c);
    b->rev^=1;
    root = merge(a,merge(b,c));
}
```

### O(N)建樹

#### 笛卡爾樹介紹

https://vjudge.net/problem/HDU-1506

#### Build

#### 例題

[Codeforces 167E Wizards and Roads](https://codeforces.com/contest/167/problem/D)

### 深度期望複雜度證明

## 例題

* [Luogu Splay和Treap習題](https://www.luogu.com.cn/training/14422#problems)
* [POJ 3580](https://vjudge.net/problem/POJ-3580)
* [UVa 12538](https://vjudge.net/problem/UVA-12538)
* [HDU 1754](https://vjudge.net/problem/HDU-1754)
* [HDU 2795](https://vjudge.net/problem/HDU-2795)
* [HDU 1394](https://vjudge.net/problem/HDU-1394)
* [HDU 1890](https://vjudge.net/problem/HDU-1890)
* [Codeforces 702F](https://codeforces.com/problemset/problem/702/F)
* [Codeforces 863D](https://codeforces.com/contest/863/problem/D)
* [CSES 2072](https://cses.fi/problemset/task/2072)
* [CSES 2073](https://cses.fi/problemset/task/2073)
* [CSES 2074](https://cses.fi/problemset/task/2074)
* [CSES 2075](https://cses.fi/problemset/task/2075)
* https://tioj.ck.tp.edu.tw/problems/1169
* https://zerojudge.tw/ShowProblem?problemid=b491
* https://www.luogu.com.cn/problem/P4309
* https://www.luogu.com.cn/problem/P2596
* https://codeforces.com/gym/102787
* https://zerojudge.tw/ShowProblem?problemid=h925
* https://tioj.ck.tp.edu.tw/problems/1975

## 參考資料

* https://oi-wiki.org/ds/treap/
* https://oi-wiki.org/ds/cartesian-tree/
* https://oi-wiki.org/ds/persistent-balanced/
* https://cp-algorithms.com/data_structures/treap.html
* https://usaco.guide/adv/treaps?lang=cpp
* https://www.luogu.com.cn/blog/85514/fhq-treap-xue-xi-bi-ji
