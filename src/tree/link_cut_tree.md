# Link Cut Tree

## 介紹

Link Cut Tree 的名字中雖然有tree，但實際上他是很多樹的集合，也就是forest  
  
Link Cut Tree 是以Splay Tree為基礎實作，因此尚未了解Splay Tree可以先回去參考  
  
    以下文章將用LCT簡稱Link Cut Tree

LCT支援以下操作：

- 在兩個點之間建立一條邊
- 在兩個點之間斷開一條邊
- 查詢兩點之間是否存在一條邊

## LCT實作

### 名詞定義

1. preferred child  
在LCT操作中，最後被走訪的節點會被設定為preferred child，可以把他想成輕重鏈剖分的重鏈  
2. preferred edge  
節點連接preferred child的邊稱為preferred edge  
3. preferred path  
一條全部由preferred edge所構成的path  

### Auxiliary Tree  

**以下用輔助樹來稱呼Auxiliary Tree**  
LCT利用Splay Tree作為輔助樹  

1. 在Splay Tree node左邊的節點深度比自己小，右邊的節點深度比自己大
2. 對於不同的Splay Tree(一群相連的Splay Tree node)，會有path-parent pointer(可以想成輕邊)將他們連接起來，構成LCT  

以下是一個簡單的Splay Tree node：

    ```cpp
    struct splay_tree
    {
        int child[2], parent;
        bool rev;
        splay_tree() : parent(0), rev(0), child({0, 0}) {}
    };
    ```

- ``child[0]``代表左小孩、``child[1]``代表右小孩
- ``parent``代表父親
- ``rev``代表區間反轉的懶惰標記

接下來就是這棵輔助樹的基本操作

- ``splay()``以及``rotate()``  

Splay Tree的基本操作

    ```cpp
    void rotate(int x)
    {
        int y = node[x].parent, z = node[y].parent, d = (node[y].child[1] == x);
        node[x].parent = z;
        if (!isroot(y))
            node[z].child[node[z].child[1] == y] = x;
        node[y].child[d] = node[x].child[d ^ 1];
        node[node[y].child[d]].parent = y;
        node[y].parent = x, node[x].child[d ^ 1] = y;
        up(y);
        up(x);
    }
    void splay(int x)
    {
        push_down(x);
        while (!isroot(x)) /*判斷是否到當前輔助樹的樹根*/
        {
            int y = node[x].parent;
            if (!isroot(y))
            {
                int z = node[y].parent;
                if ((node[z].child[0] == y) ^ (node[y].child[0] == x))
                    rotate(x);
                else
                    rotate(y);
            }
            rotate(x);
        }
    }
    ```

這裡的``splay()``寫法跟一般的Splay Tree不太一樣，LCT中的splay只要到當前這棵輔助樹的樹根即可，因此需要用到``isroot()``來判斷

- ``isroot()``  
判斷當前節點是否為根

        ```cpp
        bool isroot(int x)
        {
            return node[node[x].parent].child[0] != x && node[node[x].parent].child[1] != x;
        }
        ```

如果父節點的左小孩跟右小孩都不是自己，就代表自己是輔助樹的根
換一種說法就是自己與父節點相連的邊是輕邊

- ``push_down()``  
遞迴將祖先的懶惰標記往下推

        ```cpp
        void push_down(int x)
        {
            if (!isroot(x))
                push_down(node[x].parent);
            down(x);
        }
        ```

- ``down()``  
真正在做懶惰標記下推的部分

        ```cpp
        void down(int x)
        {
            if (node[x].rev)
            {
                swap(node[x].child[0], node[x].child[1]);
                if (node[x].child[0])
                    node[node[x].child[0]].rev ^= 1;
                if (node[x].child[1])
                    node[node[x].child[1]].rev ^= 1;
                node[x].rev = 0;
            }
        }
        ```

- ``up()``  
``up()`` 可以將子節點的訊息向上更新，可以自行修改成紀錄其他訊息，例如紀錄子樹大小  
最基本的LCT沒有使用到  

### LCT專屬函式

- ``Access()``  
**LCT中最重要的函式**  
``Access()``會把當前節點到LCT根結點上面所有的邊變成重邊
操作方法：

1. 把當前節點splay到目前輔助樹的根
2. 把當前節點的小孩設定為上一次走到的節點
3. 維護節點訊息
4. 對父節點進行``Access()``
重複執行1~4，直到抵達LCT的根結點回傳

        ```cpp
        int access(int x)
        {
            int last = 0;
            for (; x; last = x, x = node[x].parent)
            {
                splay(x);
                node[x].child[1] = last;
                up(x);
            }
            return last;
        }
        ```

操作完成後``x``節點會與根結點存在同一棵Splay Tree中  

- ``make_root()``  
將當前節點變為這棵Splay Tree中的root
操作方法：

1. 先利用``Access()``將當前節點到根節點的邊都變成重邊
2. ``splay()``當前節點，使他變成當前Splay Tree的根  
當完成此操作後，會造成此節點到原本的根的路徑全部反轉，為了要維護**左邊的節點深度比自己小，右邊的節點深度比自己大**這個性質，必須將此區間反轉
3. 變更當前節點的懶惰標記``rev``

        ```cpp
        void make_root(int x)
        {
            access(x);
            splay(x);
            node[x].rev ^= 1;
        }
        ```

- ``link()``
將兩個節點所在的樹合併，兩個節點必須在不同樹
操作方式：

1. 對其中一個節點x變成根結點
2. 將x節點的父節點設為另一個節點

        ```cpp
        void link(int x, int y)
        {
            make_root(x);
            node[x].parent = y;
        }
        ```

- ``cut()``
斷開兩個節點之間的邊，兩個節點之間必須有邊
操作方式：

1. 將其中一個節點x變成根結點
2. ``access()``另一個節點y，讓x和在同一個Splay Tree中
3. ``splay()``節點y，讓他變成Splay Tree的根  
此時x節點會成為y節點的左小孩
4. 斷開x, y節點之間的連結

        ```cpp
        void cut(int x, int y)
        {
            make_root(x);
            access(y);
            splay(y);
            node[y].child[0] = 0;
            node[x].parent = 0;
        }
        ```

另一種``cut()``操作是針對單一節點，斷開該節點與父節點的邊
操作方式：

1. 直接``access()``該節點，讓他與當前的根在同一個Splay Tree中
2. ``splay()``該節點，讓該節點變成新的根結點
此時父節點位於左小孩的位置
3. 斷開該節點與父節點的連結

        ```cpp
        void cut(int x)
        {
            access(x);
            splay(x);
            node[node[x].child[0]].parent = 0;
            node[x].child[0] = 0;
        }
        ```

- ``find_root()``
尋找此節點所在樹的根結點
操作方式：

1. 首先``access()``此節點，讓他和根結點在同一個Splay Tree裡面
2. 根據LCT的性質，只要找到最左邊的節點代表深度最小，也就是根
3. 最後記得``splay()``找到的根

        ```cpp
        int find_root(int x)
        {
            int res = access(x);
            while (node[res].child[0])
                res = node[res].child[0];
            splay(res);
            return res;
        }
        ```

### 各操作時間複雜度

LCT的操作中，都是基於``access()``操作而完成的，因此只要能分析出``access()``複雜度就可以知道各個操作的時間複雜度  

首先``access()``是由``splay()``以及迴圈所構成，``splay()``操作的時間複雜度是\\( O(\log N) \\)，接下來只要分析出``splay()``會被執行幾次，就可以知道時間複雜度了  

對於每個LCT node，都只有一個preferred child，因此\\( size(當前節點) < 1/2 size(父節點) \\)，因此每次迴圈至少會讓子樹大小變成原來的兩倍，可以寫出以下式子：
> \\( 1 \times 2^x >= N \\)
因此可以得出，最多會有\\( O(\log N) \\)次``splay()``，因此目前``access()``的上界變成了\\( O(\log^2 N) \\)  

經過一些均攤分析後(不會證明qwq)，最終時間複雜度可以更新為\\( O(\log N) \\)，有興趣可以去參考[維基百科](https://en.wikipedia.org/wiki/Link/cut_tree)

有了``access()``的時間複雜度後，發現剩下的操作都是基於``access()``以及``splay()``，因此各個LCT操作的時間複雜度皆是\\( O(\log N) \\)

|操作|時間複雜度|
|-----|--------|
|``access()``|\\( O(\log N) \\)|
|``make_root()``|\\( O(\log N) \\)|
|``link()``|\\( O(\log N) \\)|
|``cut()``|\\( O(\log N) \\)|
|``find_root()``|\\( O(\log N) \\)|

### 最終模板

<details><summary> Template Code </summary>

    ```cpp
    using namespace std;
    struct splay_tree
    {
        int child[2], parent;
        bool rev;
        splay_tree() : parent(0), rev(0), child({0, 0}) {}
    };
    struct LCT
    {
        std::vector<splay_tree> node;
        LCT(int _size) { node.resize(_size + 1); }
        bool isroot(int x)
        { /*判斷當前節點是否為根*/
            return node[node[x].parent].child[0] != x && node[node[x].parent].child[1] != x;
        }
        void down(int x)
        { /*懶惰標記下推*/
            if (node[x].rev)
            {
                swap(node[x].child[0], node[x].child[1]);
                if (node[x].child[0])
                    node[node[x].child[0]].rev ^= 1;
                if (node[x].child[1])
                    node[node[x].child[1]].rev ^= 1;
                node[x].rev = 0;
            }
        }
        void push_down(int x)
        { /*將所有祖先的懶惰標記下推*/
            if (!isroot(x))
                push_down(node[x].parent);
            down(x);
        }
        void up(int x) { /*可以把子節點的內容向上更新*/ }
        void rotate(int x)
        { /*樹平衡*/
            int y = node[x].parent, z = node[y].parent, d = (node[y].child[1] == x);
            node[x].parent = z;
            if (!isroot(y))
                node[z].child[node[z].child[1] == y] = x;
            node[y].child[d] = node[x].child[d ^ 1];
            node[node[y].child[d]].parent = y;
            node[y].parent = x, node[x].child[d ^ 1] = y;
            up(y);
            up(x);
        }
        void splay(int x)
        { /*對x節點splay*/
            push_down(x);
            while (!isroot(x))
            {
                int y = node[x].parent;
                if (!isroot(y))
                {
                    int z = node[y].parent;
                    if ((node[z].child[0] == y) ^ (node[y].child[0] == x))
                        rotate(x);
                    else
                        rotate(y);
                }
                rotate(x);
            }
        }
        int access(int x)
        { /*把x節點點到根節點的路徑上的邊都變成重邊*/
            int last = 0;
            for (; x; last = x, x = node[x].parent)
            {
                splay(x);
                node[x].child[1] = last;
                up(x);
            }
            return last; /*回傳access後的根*/
        }
        void make_root(int x)
        { /*讓x節點變成根節點*/
            access(x);
            splay(x);
            node[x].rev ^= 1;
        }
        void link(int x, int y)
        { /*把x, y所在的兩棵樹連接起來*/
            make_root(x);
            node[x].parent = y;
        }
        void cut(int x, int y)
        { /*斷開x, y所連接的邊，必須保證這條邊必須存在*/
            make_root(x);
            access(y);
            splay(y);
            node[y].child[0] = 0;
            node[x].parent = 0;
        }
        void cut(int x)
        { /*斷掉x與父節點的邊*/
            access(x);
            splay(x);
            node[node[x].child[0]].parent = 0;
            node[x].child[0] = 0;
        }
        int find_root(int x)
        { /*尋找x節點所在樹的根結點*/
            int res = access(x);
            while (node[res].child[0])
                res = node[res].child[0];
            splay(res);
            return res;
        }
    };
    ```

</details>

## LCT用途

> 有一個國家有很多城市，城市之間一開始沒有道路，但隨著時間會有道路新建成，也會有道路被拆掉  
問題是在任意時間點，要輸出A, B城市是否相連？

解題思路：  
一個國家有很多城市可以想像成一個森林，每個城市都是獨立的一棵樹，因此符合LCT是一個森林的性質。  
道路的新建可以用LCT的``link()``來達成  
道路的拆除可以用LCT的``cut()``來達成
所以只要能判斷A, B城市是否相連就可以回答問題了  
要判斷兩個城市是否相連其實就等同於兩個節點是否在同一棵樹上，所以我們只要看A, B節點的根是否一樣就可以達成  
  
時間複雜度分析：  
對於每一個操作都可以在\\( O(\log N) \\)的時間完成，因此總複雜度就是\\( O(Q \log N) \\)
  
參考題目：[DYNACON1](https://www.spoj.com/problems/DYNACON1/)
<details><summary> Solution Code </summary>

在``cut()``之前要先判斷有沒有邊存在，但在這一題他保證邊一定存在，因此可以大膽的給他``cut()``下去  

    ```cpp
    #include <bits/extc++.h>
    using namespace std;
    struct splay_tree
    {
        int child[2], parent;
        bool rev;
        splay_tree() : parent(0), rev(0), child() {}
    };
    struct LCT
    {
        std::vector<splay_tree> node;
        LCT(int _size) { node.resize(_size + 1); }
        bool isroot(int x)
        { /*判斷當前節點是否為根*/
            return node[node[x].parent].child[0] != x && node[node[x].parent].child[1] != x;
        }
        void down(int x)
        { /*懶惰標記下推*/
            if (node[x].rev)
            {
                swap(node[x].child[0], node[x].child[1]);
                if (node[x].child[0])
                    node[node[x].child[0]].rev ^= 1;
                if (node[x].child[1])
                    node[node[x].child[1]].rev ^= 1;
                node[x].rev = 0;
            }
        }
        void push_down(int x)
        { /*將所有祖先的懶惰標記下推*/
            if (!isroot(x))
                push_down(node[x].parent);
            down(x);
        }
        void up(int x) { /*可以把子節點的內容向上更新*/ }
        void rotate(int x)
        { /*樹平衡*/
            int y = node[x].parent, z = node[y].parent, d = (node[y].child[1] == x);
            node[x].parent = z;
            if (!isroot(y))
                node[z].child[node[z].child[1] == y] = x;
            node[y].child[d] = node[x].child[d ^ 1];
            node[node[y].child[d]].parent = y;
            node[y].parent = x, node[x].child[d ^ 1] = y;
            up(y);
            up(x);
        }
        void splay(int x)
        { /*對x節點splay*/
            push_down(x);
            while (!isroot(x))
            {
                int y = node[x].parent;
                if (!isroot(y))
                {
                    int z = node[y].parent;
                    if ((node[z].child[0] == y) ^ (node[y].child[0] == x))
                        rotate(x);
                    else
                        rotate(y);
                }
                rotate(x);
            }
        }
        int access(int x)
        { /*把x節點點到根節點的路徑上的邊都變成重邊*/
            int last = 0;
            for (; x; last = x, x = node[x].parent)
            {
                splay(x);
                node[x].child[1] = last;
                up(x);
            }
            return last; /*回傳access後的根*/
        }
        void make_root(int x)
        { /*讓x節點變成根節點*/
            access(x);
            splay(x);
            node[x].rev ^= 1;
        }
        void link(int x, int y)
        { /*把x, y所在的兩棵樹連接起來*/
            make_root(x);
            node[x].parent = y;
        }
        void cut(int x, int y)
        { /*斷開x, y所連接的邊，必須保證這條邊必須存在*/
            make_root(x);
            access(y);
            splay(y);
            node[y].child[0] = 0;
            node[x].parent = 0;
        }
        void cut(int x)
        { /*斷掉x與父節點的邊*/
            access(x);
            splay(x);
            node[node[x].child[0]].parent = 0;
            node[x].child[0] = 0;
        }
        int find_root(int x)
        { /*尋找x節點所在樹的根結點*/
            int res = access(x);
            while (node[res].child[0])
                res = node[res].child[0];
            splay(res);
            return res;
        }
    };
    int main()
    {
        ios::sync_with_stdio(false);
        cin.tie(0);

        int n, q;
        cin >> n >> q;
        LCT lct(n);
        while (q--)
        {
            string str;
            int u, v;
            cin >> str >> u >> v;
            if (str[0] == 'a')
                lct.link(u, v);
            else if (str[0] == 'r')
                lct.cut(u, v);
            else if (lct.find_root(u) == lct.find_root(v))
                cout << "YES\n";
            else
                cout << "NO\n";
        }

        return 0;
    }
    ```

</details>

> [SPOJ QTREE](https://www.spoj.com/problems/QTREE/)  
> 給你一棵\\( N \\)個節點的樹，每個邊有邊權\\( w \\)，然後會有\\( Q \\)筆操作。操作有兩種：  
>
>    1. 改動其中一條邊的邊權
>    2. 詢問節點\\( u \\)到節點\\( v \\)路徑上權重最大的邊
>
> - \\( N, Q \leq 10^4 \\)  
  
解題思路：
這題第一眼看到就會想用輕重鏈剖分做，但這樣太無趣了，來挑戰看看用LCT做  
在LCT中我們沒辦法維護邊權，因此我們可以利用**拆邊**的技巧，也就是將將一條邊變成一個點加上兩個邊，而那個點的權重變成原本邊的權重，可以參考下圖：  
<img src="image/LCT/cut_edge.png" width="700" style="display:block; margin: 0 auto;"/>

解決邊權的問題後，但在建樹之前要先維護好節點的資訊  
對於每個節點，需要維護：  

1. 點權 (程式碼中利用``val``表示)
2. Splay Tree點權的最大值 (程式碼中利用``mx``表示)  

因為需要維護最大值，因此需要改動``up()``函式，當前節點的最大值就是小孩的最大值跟自己的點權取max

    ```cpp
    void up(int x)
    { /*可以把子節點的內容向上更新*/
        node[x].mx = max(max(node[node[x].child[0]].mx, node[node[x].child[1]].mx), node[x].val);
    }
    ```

對於題目的操作可以用下面方式達成：

1. 改動其中一條邊的邊權  
直接更改那條邊拆點後的點權即可  
2. 詢問路徑上最大的邊  
與``cut()``操作相似，先讓``u``節點變成根，再把``v``節點旋轉到根，``v``節點的值就是答案  

時間複雜度分析：  
建樹時需要建立\\( 2N - 2 \\)條邊，建邊時間複雜度為\\( O(\log N) \\)，因此建樹時時間複雜度是\\( O(N\log N) \\)  
對於每一個操作都可以在\\( O(\log N) \\)的時間完成，因此\\( Q \\)筆操作時間複雜度就是\\( O(Q \log N) \\)  
總時間複雜度為\\( O(N\log N + Q \log N) \\)

<details><summary> Solution Code </summary>

    ```cpp
    #include <bits/extc++.h>
    using namespace std;
    struct splay_tree
    {
        int child[2], parent;
        int val, mx;
        bool rev;
        splay_tree() : parent(0), rev(0), child(), val(0), mx(0) {}
    };
    struct LCT
    {
        std::vector<splay_tree> node;
        LCT(int _size) { node.resize(_size + 1); }
        bool isroot(int x)
        { /*判斷當前節點是否為根*/
            return node[node[x].parent].child[0] != x && node[node[x].parent].child[1] != x;
        }
        void down(int x)
        { /*懶惰標記下推*/
            if (node[x].rev)
            {
                swap(node[x].child[0], node[x].child[1]);
                if (node[x].child[0])
                    node[node[x].child[0]].rev ^= 1;
                if (node[x].child[1])
                    node[node[x].child[1]].rev ^= 1;
                node[x].rev = 0;
            }
        }
        void push_down(int x)
        { /*將所有祖先的懶惰標記下推*/
            if (!isroot(x))
                push_down(node[x].parent);
            down(x);
        }
        void up(int x)
        { /*可以把子節點的內容向上更新*/
            node[x].mx = max(max(node[node[x].child[0]].mx, node[node[x].child[1]].mx), node[x].val);
        }
        void rotate(int x)
        { /*樹平衡*/
            int y = node[x].parent, z = node[y].parent, d = (node[y].child[1] == x);
            node[x].parent = z;
            if (!isroot(y))
                node[z].child[node[z].child[1] == y] = x;
            node[y].child[d] = node[x].child[d ^ 1];
            node[node[y].child[d]].parent = y;
            node[y].parent = x, node[x].child[d ^ 1] = y;
            up(y);
            up(x);
        }
        void splay(int x)
        { /*對x節點splay*/
            push_down(x);
            while (!isroot(x))
            {
                int y = node[x].parent;
                if (!isroot(y))
                {
                    int z = node[y].parent;
                    if ((node[z].child[0] == y) ^ (node[y].child[0] == x))
                        rotate(x);
                    else
                        rotate(y);
                }
                rotate(x);
            }
        }
        int access(int x)
        { /*把x節點點到根節點的路徑上的邊都變成重邊*/
            int last = 0;
            for (; x; last = x, x = node[x].parent)
            {
                splay(x);
                node[x].child[1] = last;
                up(x);
            }
            return last; /*回傳access後的根*/
        }
        void make_root(int x)
        { /*讓x節點變成根節點*/
            access(x);
            splay(x);
            node[x].rev ^= 1;
        }
        void link(int x, int y)
        { /*把x, y所在的兩棵樹連接起來*/
            make_root(x);
            node[x].parent = y;
        }
        void cut(int x, int y)
        { /*斷開x, y所連接的邊，必須保證這條邊必須存在*/
            make_root(x);
            access(y);
            splay(y);
            node[y].child[0] = 0;
            node[x].parent = 0;
        }
        void cut(int x)
        { /*斷掉x與父節點的邊*/
            access(x);
            splay(x);
            node[node[x].child[0]].parent = 0;
            node[x].child[0] = 0;
        }
        int find_root(int x)
        { /*尋找x節點所在樹的根結點*/
            int res = access(x);
            while (node[res].child[0])
                res = node[res].child[0];
            splay(res);
            return res;
        }
    };

    void solve()
    {
        int n, u, v, w;
        cin >> n;
        LCT lct(2 * n);
        for (int i = 1; i < n; i++) // n + i 代表 ith 邊
        {
            cin >> u >> v >> w;
            lct.node[n + i].val = w;
            lct.link(u, n + i);
            lct.link(n + i, v);
        }
        string op;
        while (true)
        {
            cin >> op;
            if (op == "DONE")
                break;
            else if (op == "CHANGE")
            {
                cin >> u >> w;
                u += n;
                lct.access(u);
                lct.splay(u);
                lct.node[u].val = w;
            }
            else
            {
                cin >> u >> v;
                lct.make_root(u);
                lct.access(v);
                lct.splay(v);
                cout << lct.node[v].mx << '\n';
            }
        }
    }

    int main()
    {
        ios::sync_with_stdio(false);
        cin.tie(0);

        int t;
        cin >> t;
        while (t--)
        {
            solve();
        }

        return 0;
    }
    ```

</details>

總結：  

- LCT可以判斷連通性
- LCT可以做路徑維護，因此可以維護路徑資訊
- LCT可以動態切邊，因此當看到**cut**關鍵字的時候，或許可以想想看LCT可不可以做，因為大部分輕重鏈剖分可以做的題目，LCT也都可以做

## 相關題目

[DYNALCA - Dynamic LCA](https://www.spoj.com/problems/DYNALCA/)
<details><summary> Solution Code </summary>

詢問lca時，先``access()``其中一個節點，再``access()``另一個節點，即可找到lca  

    ```cpp
    #include <bits/extc++.h>
    using namespace std;
    struct splay_tree
    {
        int child[2], parent;
        bool rev;
        splay_tree() : parent(0), rev(0), child({0, 0}) {}
    };
    struct LCT
    {
        std::vector<splay_tree> node;
        LCT(int _size) { node.resize(_size + 1); }
        bool isroot(int x)
        { /*判斷當前節點是否為根*/
            return node[node[x].parent].child[0] != x && node[node[x].parent].child[1] != x;
        }
        void down(int x)
        { /*懶惰標記下推*/
            if (node[x].rev)
            {
                swap(node[x].child[0], node[x].child[1]);
                if (node[x].child[0])
                    node[node[x].child[0]].rev ^= 1;
                if (node[x].child[1])
                    node[node[x].child[1]].rev ^= 1;
                node[x].rev = 0;
            }
        }
        void push_down(int x)
        { /*將所有祖先的懶惰標記下推*/
            if (!isroot(x))
                push_down(node[x].parent);
            down(x);
        }
        void up(int x) { /*可以把子節點的內容向上更新*/ }
        void rotate(int x)
        { /*樹平衡*/
            int y = node[x].parent, z = node[y].parent, d = (node[y].child[1] == x);
            node[x].parent = z;
            if (!isroot(y))
                node[z].child[node[z].child[1] == y] = x;
            node[y].child[d] = node[x].child[d ^ 1];
            node[node[y].child[d]].parent = y;
            node[y].parent = x, node[x].child[d ^ 1] = y;
            up(y);
            up(x);
        }
        void splay(int x)
        { /*對x節點splay*/
            push_down(x);
            while (!isroot(x))
            {
                int y = node[x].parent;
                if (!isroot(y))
                {
                    int z = node[y].parent;
                    if ((node[z].child[0] == y) ^ (node[y].child[0] == x))
                        rotate(x);
                    else
                        rotate(y);
                }
                rotate(x);
            }
        }
        int access(int x)
        { /*把x節點點到根節點的路徑上的邊都變成重邊*/
            int last = 0;
            for (; x; last = x, x = node[x].parent)
            {
                splay(x);
                node[x].child[1] = last;
                up(x);
            }
            return last; /*回傳access後的根*/
        }
        void make_root(int x)
        { /*讓x節點變成根節點*/
            access(x);
            splay(x);
            node[x].rev ^= 1;
        }
        void link(int x, int y)
        { /*把x, y所在的兩棵樹連接起來*/
            make_root(x);
            node[x].parent = y;
        }
        void cut(int x, int y)
        { /*斷開x, y所連接的邊，必須保證這條邊必須存在*/
            make_root(x);
            access(y);
            splay(y);
            node[y].child[0] = 0;
            node[x].parent = 0;
        }
        void cut(int x)
        { /*斷掉x與父節點的邊*/
            access(x);
            splay(x);
            node[node[x].child[0]].parent = 0;
            node[x].child[0] = 0;
        }
        int find_root(int x)
        { /*尋找x節點所在樹的根結點*/
            int res = access(x);
            while (node[res].child[0])
                res = node[res].child[0];
            splay(res);
            return res;
        }
    };
    
    int main()
    {
        ios::sync_with_stdio(false);
        cin.tie(0);
    
        int n, m, u, v;
        cin >> n >> m;
        LCT lct(n);
        while (m--)
        {
            string query;
            cin >> query;
            if (query == "link")
            {
                cin >> u >> v;
                lct.link(u, v);
            }
            else if (query == "cut")
            {
                cin >> u;
                lct.cut(u);
            }
            else if (query == "lca")
            {
                cin >> u >> v;
                lct.access(u);
                cout << lct.access(v) << '\n';
            }
        }
    
        return 0;
    }
    ```

</details>

SPOJ-QTREE系列題目：  
[QTREE](https://www.spoj.com/problems/QTREE/)  
[QTREE2](https://www.spoj.com/problems/QTREE2/)  
[QTREE3](https://www.spoj.com/problems/QTREE3/)  
[QTREE4](https://www.spoj.com/problems/QTREE4/)  
[QTREE5](https://www.spoj.com/problems/QTREE5/)  
[QTREE6](https://www.spoj.com/problems/QTREE6/)  
[QTREE7](https://www.spoj.com/problems/QTREE7/)  
QTREE系列題目可以參考這篇文章：[【Qtree】Query on a tree系列LCT解法](https://blog.csdn.net/thy_asdf/article/details/50768620)  

[CF 117E - Tree or not Tree](https://codeforces.com/problemset/problem/117/E)  
[CF 342E - Xenia and Tree](https://codeforces.com/problemset/problem/342/E)  
[CF 1344E - Train Tracks](https://codeforces.com/problemset/problem/1344/E)  

## 參考資料

[Link/cut tree - Wikipedia](https://en.wikipedia.org/wiki/Link/cut_tree)  
[Link Cut Tree - OI Wiki](https://oi-wiki.org/ds/lct/)  
[日月卦長的模板庫: [ link-cut tree ] 動態樹教學+模板](<http://sunmoon-template.blogspot.com/2015/11/link-cut-tree.html>)  
[Splay tree tutorial - Codeforces](https://codeforces.com/blog/entry/79524)  
[Link-cut tree tutorial - Codeforces](https://codeforces.com/blog/entry/80383)  
[【Qtree】Query on a tree系列LCT解法](https://blog.csdn.net/thy_asdf/article/details/50768620)  
