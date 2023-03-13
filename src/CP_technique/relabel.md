# 離散化 Relabel

考慮以下問題，  
給你一個長度為\\( n \\)的陣列\\( A \\)，  
求\\( A_i\\)為陣列中第幾大的數。  
\\(（1 \le  n \le 10^6\\), \\(-10^9 \le A_i \le  10^9）\\)  

像這種問題，  
我們不在乎\\( A_i \\)實際上是多大，  
只在乎他的*名次*是多少。  

我們就可以用到**離散化**這個技巧。  
以下是離散化的範例code：  

```
void relabel(vector<int> &arr) {
  set<int> s(arr.begin(), arr.end());
  map<int,int> mp;
  int idx = 0;
  for (auto x : s) mp[x] = idx++;
  for (auto &x : arr) x = mp[x];
}
```
總之就是使用map幫我們把整數編號。  

## Examples

> [CSES - Restaurant Customers](https://cses.fi/problemset/task/1619)
> 有\\( n \\)個顧客進來用餐，每個都有抵達時間與離開時間，求餐廳裡最多同時有多少顧客在裡面？  

<details><summary>Solution</summary>

先對抵達和離開時間做relabel，接著使用sweeping line。  
可參考[Sweeping Line](/Sweeping_line.md)  
    
在這題裡，抵達和離開時間的值域為\\( 10^9 \\)，  
經過離散化我們把值域大小縮小成了\\( 2\times 10^5 \\)。  
    
複雜度為：\\( O(nlogn) + O(n) \\)  
假設不做relabel的話，sweeping line掃過會是\\( O(n) \\)，  
但這邊的\\(n\\)大小是\\( 10^9 \\)，會TLE，所以要先做relabel縮小值域。  
    
</details>


> [UVa 13212 - How Many Inversions](https://onlinejudge.org/index.php?option=onlinejudge&Itemid=8&page=show_problem&problem=5135)
> 給定長度為\\( n\\)的序列\\( A \\)，問其中有幾個inversion（inversion的定義為\\(i < j \\)且\\(a_i > a_j \\)）

<details><summary>Solution</summary>

因為要找的是inversion，所以\\( A\\)的值並不重要，重要的是大小關係。  
我們可以先做relabel，做完relabel後，可發現一個性質，  
當我們從頭開始加入\\( A \\)的元素，我們只需要去注意，  
在當前元素被加入之前，有幾個比它大的元素已經在裡面了（inversion)。  
那我們可以開一個長度等於值域的陣列\\( count, count[i] = i\\)出現了幾次。  
所以當加入元素\\(x\\)時，就在\\(count[x]++ \\)。  
那麼，我們算出x這個index的後綴和，我們可以求出x所造成的inversion。  
把所有的inversion加起來就是答案了。  
    
至於要怎麼維護動態後綴和，可以使用BIT這個輕量的資料結構。  
```
relabel(arr);
bit.assign(n+1,0);
int ans = 0;
for(auto &x:arr) {
    ans += query(x);
	modify(x,1);	
    //注意這邊bit維護的是suffix sum
}
```

    
</details>

## Summary
- 當值域很大，而我們只在乎元素間的關係，元素本身的值並不重要的時候，我們就可以使用離散化來縮小值域。  