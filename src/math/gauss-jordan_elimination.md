# 高斯消去法
## 介紹
高斯消去法（Gauss-Jordan elimination）是一個經常被拿來解線性方程組的經典算法，是線性代數中的基礎知識之一。除了用來解線性方程組以外，還可以用來計算矩陣的行列式值、逆矩陣等。

### 解線性方程式
範例：解二元一次方程式
\begin{cases}
2x + y = -1\\  
3x - y = 11
\end{cases}

可以發現將兩式相加可以消掉 \\(y\\)，得到 \\(5x = 10，x = 2\\)，帶回1式得到 \\(y = -5\\)

但是並不是每次遇到的線性方程組都像這個可以輕松的看出的消掉哪些變數。因此我們需要一個更有系統的解法

### 高斯消去法
#### 原理：
>對於任意的線性方程組，我們可進行以下操作同時不會改變解
>1. 將其中一等式左右同乘非零實數k
>2. 將一個等式加上另外一個等式
>3. 交換等式的順序
>
>上述1, 2點等價於：任意等式可以被"自己和另一個等式的線性組合"取代

接著考慮以下情況：
假設我們要解以下方程組：
\begin{cases}
a_{11} + a_{12} + ...a_{1m} = b_{1} \\
a_{21} + a_{22} + ...a_{2m} = b_{2} \\
...                                 \\
a_{n1} + a_{n2} + ...a_{nm} = b_{n}
\end{cases}


範例：高斯消去法解二元一次方程式
$$
\begin{cases}
{2x + y = -1} \\\ {3x - y = 11}
\end{cases}
$$
將第一式 \\( * \dfrac {-3} 2\\) 並加到第二式得\\(\dfrac {-5} {2} y = \dfrac {25} {2}\\)

平衡係數得\\(y = -5\\)


再將第二式 \\(* -1\\) 並加到第一式得 \\(2x=4,x = 2\\)

## 實作
``` c++
void guass(vector<vector<double>> &v){
    int n = v.size(), m = v[0].size();
    for(int i = 0 ; i < n; i++){
        int now = i;
        for(int j = i+1; j < n; j++){     // find pivot row
            if(abs(v[j][i]) > abs(v[now][i]))now = j; 
        }
        if(abs(v[now][i]) < esp)continue;
        swap(v[i], v[now]);               // swap pivot row to current row

        for(int j = 0; j < n; j++){       // eliminate
            if(j != i){
                double d = v[j][i] / v[i][i];
                for(int k = i; k < m; k++)
                    v[j][k] -= v[i][k] * d;
            }
        }
    }
}
```
先找出pivot row

利用v[i][i]來消除第i列的所有係數

最後得到一個像這樣的矩陣：

$$
  \left[\begin{array}{rrrrrr|r}
   a_{11} & 0 & ... & 0 & ... & 0 & b_{1}  \\\ 0 & a_{22} & ... & 0 & ... & 0 & b_{2}  \\\ &&.&&&&. \\\ &&.&&&&. \\\ &&.&&&&. \\\ 0 & 0 & ... & a_{nn} & ... & a_{nm} & b_{n}
  \end{array}\right]
$$


### 選擇 pivot row
pivot row的選擇並沒有特別的規定。上面用的方法是選擇該column最大值的row當作pivot row。而另外一種方法則是選擇當前的submatrix中最大值所在的row當作pivot row。但是這也會導致一個問題，也就是當其中一個等式被乘上一個很大的係數，如\\(10^6\\)，那這一行基本上一定會在第一步被選擇當pivot row。但這並不是我們想選他當pivot row的原因，因此又有一種比較複雜的方法叫做implict pivoting。implicit pivoting 會先將每一個等式normalize，也就是乘以一個係數使得該等式最大的係數 = 1。接著再用前面提到的方法選擇pivot row。

### 檢查解
線性方程組的解有三種:
1. 無解
2. 剛好1組解
3. 無限組解

#### 1. 無解
考慮以下例子 : 
\begin{cases}
2x + 2y = 3 \\\ x+y = 1
\end{cases}
經過高斯消去法後我們得到
$$
\left[\begin{array}{rr|r}
    2 & 2 & 3 \\\ 0 & 0 & -2
\end{array}\right]
$$
我們可以發現如果其中一個化簡後的等式的LHS全部都是0但是RHS不等於0時，此線性方程組無解。

#### 2. 剛好1組解
考慮以下例子 : 
\begin{cases}
2x + y = 3 \\\ x+y = 1
\end{cases}
經過高斯消去法後我們得到
$$
\left[\begin{array}{rr|r}
    2 & 0 & 4 \\\ 0 & 0.5 & -0.5
\end{array}\right]
$$
如果所有的v[i][i]都不等於0，此線性方程組剛好有1組解。

#### 3. 無限組解
考慮以下例子 : 
\begin{cases}
x + y + z = 6 \\\ x + 3z = 5 \\\ x + 2y - z = 7
\end{cases}
經過高斯消去法後我們得到
$$
\left[\begin{array}{rrr|r}
    1 & 0 & 3 & 5 \\\ 0 & -1 & 2 & -1 \\\ 0 & 0 & 0 & 0 
\end{array}\right]
$$
如果有某些的v[i][i]等於0且該等式RHS也是0，此線性方程組有無限組解。

### 一些小優化

### 降低誤差

## 特殊解 

## 題目
模板提 ： [洛谷P3389](https://www.luogu.com.cn/problem/P3389)