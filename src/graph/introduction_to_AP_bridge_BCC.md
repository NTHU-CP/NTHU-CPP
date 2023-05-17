# AP, Bridge, BCC

## 前言

在此章節中，我們將介紹無向圖上的AP(Articulation point), Bridge與BCC(Biconnected component)，包括他們的定義、相關的演算法和題目。

## Connected & Connected Component
在正式開始我們的主題之前，我們先來介紹一些之後會很常看到的東西。

### Connected(連通)
我們說兩個點 A, B connected，表示A, B之間存在一條path。如下圖例子中，A 跟 B connected，但 A 跟 C 不 connected
<img src="image/Connected.JPG" width="500" style="display:block; margin: 0 auto;"/>

### Connected Component(連通分量)
Connected Component 是點的集合，集合中任兩點都是 connected 的，同時每一個 Connected Component 都必須盡量大。如果兩個點位於不同的 Connected Component，則這兩點必不 Connected。  
如下圖的例子，顏色相同的點表示他們屬於相同的 Connected Component，可以看到若是兩個點不同顏色，那他們一定不 connected。而我們不會說紅圈圈起來的那三個點是一個Connected Component。
<img src="image/Connected_Component.JPG" width="500" style="display:block; margin: 0 auto;"/>

## AP & Bridge

### definition
若一張圖 G 移除一個點 V 之後有超過一個connected component，則點 V 為 AP  
若一張圖 G 移除一條邊 E 之後有超過一個connected component，則邊 E 為 Bridge  
(補圖)
### Tarjan's Algorithm for AP/Bridge
很簡單可以想到O(V^2)的做法

### Using DFS Tree 

## BCC

### definition

## Problems

> [CSES - Necessary Cities](https://cses.fi/problemset/task/2077)
> 
> 給定一張 \\( N \\) 個點 \\( M \\) 條邊的無向圖，要你找出圖上所有的 AP

AP 模板題

> [CSES - Necessary Roads](https://cses.fi/problemset/task/2076)
> 
> 給定一張 \\( N \\) 個點 \\( M \\) 條邊的無向圖，要你找出圖上所有的 Bridge

Bridge 模板題
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
- [oi-wiki - bcc](https://oi-wiki.org/graph/bcc/)
- [Hackerearth - bcc](https://www.hackerearth.com/practice/algorithms/graphs/biconnected-components/tutorial/)
- [codeforce blog - DFS Tree](https://codeforces.com/blog/entry/68138)



