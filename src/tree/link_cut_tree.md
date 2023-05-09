# Link Cut Tree

## 介紹
Link Cut Tree 的名字中雖然有tree，但實際上他是很多樹的集合，也就是forest  
  
Link Cut Tree 是以Splay Tree為基礎實作，因此尚未了解Splay Tree可以先回去參考  
  
**以下文章將用LCT簡稱Link Cut Tree**  

LCT支援以下操作：
- 在兩個點之間建立一條邊
- 在兩個點之間斷開一條邊 (那條邊必須存在)
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
這裡的splay()寫法跟一般的Splay Tree不太一樣，LCT中的splay只要到當前這棵輔助樹的樹根即可，因此需要用到``isroot()``來判斷
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
3. 