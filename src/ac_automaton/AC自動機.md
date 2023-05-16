# AC自動機(AC Automaton)

## Introduction

AC自動機是結合Trie的結構(Data Structure)及KMP演算法(Algorithm)而形成的自動機，經常用於解決多模式匹配的問題。
該一算法誕生於1975年的貝爾實驗室，由Alfred Aho和Margaret Corasick發表。了解AC自動機之前需先理解字典樹Trie如何使用，
該算法將KMP算法的Fail Function應用於Trie中作為失敗邊

## 自動機理論
Trie的節點(Vertex)可以視為確定有限狀態自動機(Finite deterministic automaton)的狀態，藉由輸入的字母達到狀態轉移
(待補充)
## Trie的實現
在一個Trie中節點的含意是某個模式串的前綴，或者我們也可以將其稱之為一種狀態(State)，一個節點即代表一個狀態。而邊則是代表狀態之間的轉移，
對於長度為n的字串，可能出現的字元數為k的字串，一個簡單具有插入及搜尋操作的Trie可以如以下方式呈現，時間複雜度顯然是線性的 \\(O(n)\\)，
但其空間複雜度可以至 \\(O(nk)\\)，占用過多的空間是Trie在實作上必須特別注意的一個點。
```c++=
const int K = 26;
struct Vertex {
    int next[K];
    bool leaf = false;
    Vertex() {
        fill(begin(next), end(next), -1);
    }
};

vector<Vertex> trie(1);

void insert(string const &s) {
    int v = 0;
    for (char ch : s) {
        int c = ch - 'a';
        if (trie[v].next[c] == -1) {
            trie[v].next[c] = trie.size();
            trie.emplace_back();
        }
        v = trie[v].next[c];
    }
    trie[v].leaf = true;
}

bool search(string const& s) {
    int v = 0;
    for(char ch : s) {
        int c = ch - 'a';
        v = trie[v].next[c];
        if(v == trie.size())
            return false;
    }
    return trie[v].leaf;
}

```
![Trie](image/Trie.svg)

## 失配指針

## AC自動機的構建

## 優化與加強

## 例題


## Reference
- [[Tutorial] Aho-Corasick algorithm](https://cp-algorithms.com/string/aho_corasick.html)
- [[Tutorial] https://oi-wiki.org/string/ac-automaton/](https://oi-wiki.org/string/ac-automaton/)
- [[Slides] Stanford's CS166 - Aho-Corasick Automata](http://web.stanford.edu/class/archive/cs/cs166/cs166.1166/lectures/02/Slides02.pdf)