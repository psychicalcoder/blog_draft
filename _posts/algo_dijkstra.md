---
title: 演算法週記 單源最短路徑之Dijkstra's Algorithm
tag: ["algorithm","SSSP"]
categories: Algorithm-Weeekly-Journal
---
### Dijkstra's Algorithm
{% blockquote %}
接續上次Bellman-Ford沒講完的SSSP算法
今天來說說Dijkstra's Algorithm</br>
這個演算法實作簡單
是目前已知時間複雜度最小的SSSP算法
但是只適用於邊權全都大於等於{% katex [false] %}0{% endkatex %}的圖</br>
那麼我們來思考一下Bellman-Ford Algorithm的缺點吧
可以注意到Bellman-Ford每次都要掃過全部的邊
可是有時候掃過這些邊一點用處都沒有
就算有被更新到，也不一定會是真正的最短距離
是不是做了很多多餘的事呢?</br>
要怎麼樣才能避免做到無謂的事呢?</br>
可以很輕易的想到
只對已經確定最短路徑的頂點做延伸
延伸完就不再用這個頂點
這兩個優化</br>
如何確定一個頂點是否已經被確定了呢?
這時我們引入最短路徑樹的概念
一棵最短路徑樹上的點
必是一個已經被確定的點</br>
最初只有起點在最短路徑樹上
接著找出一個不在樹上
離起點最近的一個點加入最短路徑樹
不停的重複這個動作
直到所有的點都被加入最短路徑樹</br>
詳細的實作法如下
1.將所有的{% katex [false] %}d[i]{% endkatex %}設為{% katex [false] %}INF{% endkatex %}
2.將{% katex [false] %}d[s]{% endkatex %}設為{% katex [false] %}0{% endkatex %}
3.找到所有未訪問的點中{% katex [false] %}d{% endkatex %}最小的點{% katex [false] %}x{% endkatex %}
4.將{% katex [false] %}x{% endkatex %}設為已訪問
5.更新x所有聯外點{% katex [false] %}y{% endkatex %}的{% katex [false] %}d{% endkatex %}
6.回到3.直到找不到為止
{% endblockquote %}

<!-- more -->

以下程式碼由C++撰寫
``` C++
#include <iostream>
using namespace std;

const int INF = 1000000000; //1e9

int V, E; //頂點數、邊數
int G[105][105]; //有邊則G[i][j] != INF
int d[105]; //所有點到S的距離
bool vis[105]; //是否被加入最短路徑樹

void dijkstra(int s) {
    for(int i = 0; i < V; i++) d[i] = INF;
    memset(vis, false, sizeof(vis));
    d[s] = 0;
    while(true) {
        int x = -1;
        for(int i = 0; i < V; i++) {
            if(!vis[i] && (x == -1 || d[x] > d[i])) {
                x = i;
            } //找到最近的點
        }
        if(x == -1) break; //找不到就退出
        vis[x] = true;
        for(int i = 0; i < V; i++) {
            if(d[i] > d[x]+G[x][i]) {
                d[i] = d[x]+G[x][i];
            }
        } //更新x的聯外點
    }
}

//demo
int main() {
    cin >> V >> E;
    for(int i = 0; i < V; i++) for(int j = 0; j < V; j++) G[i][j] = INF;
    int u,v,c;
    for(int i = 0; i < E; i++) {
        cin >> u >> v >> c;
        //G[v][u] = G[u][v] = c; //無向圖
    }
    dijkstra(0);
    for(int i = 0; i < V; i++) {
        cout << "d[" << i << "] = " << d[i] << endl;
    }
    return 0;
}
```

### 分析
{% blockquote %}
先來算算複雜度吧
每次都會把一個點加入最短路徑樹
做多加入{% katex [false] %}V-1{% endkatex %}個點加起點
更新(鬆弛-{% katex [false] %}relaxation{% endkatex %})其他點的時候
會遍歷所有的點(因為是鄰接矩陣)
所以時間複雜度顯然是{% katex [false] %}O(V^2){% endkatex %}</br>
接著我們來確定一下這樣的做法是正確的
考慮到找目前距離起點最近的點{% katex [false] %}x{% endkatex %}
如果這時的{% katex [false] %}d[x]{% endkatex %}不是最短距離的話
就意味著還會有另外的一條最短路徑會連上{% katex[false] %}x{% endkatex %}
產生一個比目前的{% katex [false] %}d[x]{% endkatex %}還小的{% katex [false] %}d[x]{% endkatex %}
但是邊權不會為負
所以那條"將來"最短路徑上所有的點{% katex [false] %}i{% endkatex %}
他的{% katex [false] %}d[i]{% endkatex %}必定{% katex [false] %}\leq d[x]{% endkatex %}
所以{% katex [false] %}d[x]{% endkatex %}就不可能在這個迴圈被找到為最小值
{% katex [false] %}x{% endkatex %}也就不會被加入最短路徑樹
因此我們可以確定這樣的情況不會發生
這也就是為什麼我們要限制邊權一定要大於{% katex [false] %}0{% endkatex %}
{% endblockquote %}

### 優化
{% blockquote %}
在完全圖的情況下
剛剛的複雜度會等於{% katex [false] %}O(E){% endkatex %}
但是如果是個稀疏圖
豈不是浪費了一堆時間在找最小值跟鬆弛</br>
想要快速的找到最小值
可以馬上想到heap</br>
快速且有效的鬆弛
則是可以使用鄰接串列來實現</br>
在heap中插入存有{% katex [false] %}(d[i],i){% endkatex %}的數對
可以快速的找出最小值及其對應的點</br>
遇到已經被加入最短路徑樹的點則直接跳過
馬上來看Code吧
{% endblockquote %}

``` C++
#include <iostream>
#include <vector>
#include <queue>
using namespace std;
typedef pair<int,int> P;

const int INF = 1000000000; //1e9


struct edge {
    int to, cost;
};

int V, E; //頂點數、邊數
vector<edge> G[105];
int d[105];
bool vis[105];
priority_queue<P, vector<P>, greater<P> > Q;

void dijkstra(int s) {
    //while(!Q.empty()) Q.pop(); //在不只做一次的情況下需要先清空heap
    for(int i = 0; i < V; i++) d[i] = INF;
    d[s] = 0;
    Q.push(make_pair(0, s));
    for(int i = 0; i < V; i++) {
        int x = Q.top().second; Q.pop();
        if(vis[x]) continue;
        vis[x] = true;
        for(edge &e : G[x]) {
            if(d[e.to] > d[x]+e.cost) {
                d[e.to] = d[x]+e.cost;
                Q.push(make_pair(d[e.to], e.to));
            }
        }
    }
}

//demo
int main() {
    int from, to, cost;
    cin >> V >> E;
    for(int i = 0; i < E; i++) {
        cin >> from >> to >> cost;
        G[from].push_back((edge){to, cost});
        //G[to].push_back((edge){from, cost}); //無向圖
    }
    dijkstra(0);
    for(int i = 0; i < V; i++) {
        cout << "d[" << i << "] = " << d[i] << endl;
    }
    return 0;
}
```

{% blockquote %}
來算算優化完的複雜度吧
考慮最糟糕的情況
就是每條邊都拿去鬆弛成功
也就是heap要插入{% katex [false] %}E{% endkatex %}次取出{% katex [false] %}V{% endkatex %}次
一般的heap插入取出都是{% katex [false] %}O(logn){% endkatex %}
所以時間複雜度{% katex [false] %}O((V+E)logE){% endkatex %}
{% endblockquote %}

### 二次優化
{% blockquote %}
那個{% katex [false] %}logE{% endkatex %}我們還是不太滿意
大家都知道除了peek這個操作之外
heap所有的操作都可以用
binary search tree做出來
我們不彷用BST來取代heap
利用BST的刪除操作
每次鬆弛的時候
都把舊的點從BST中刪除
再插新的進去
這樣可以維持BST的大小在{% katex [false] %}O(V){% endkatex %}
插入刪除均維{% katex [false] %}O(ElogV){% endkatex %}
取出一樣是{% katex [false] %}O(VlogV){% endkatex %}
這樣時間複雜度就進步成{% katex[false] %}O((V+E)logV){% endkatex %}了
雖然你可能被BST的常數淹沒
{% endblockquote %}

### 三次優化
{% blockquote %}
每次都要刪除再重新插入
太慢了!!!
Michael L. Fredman與Robert E. Tarjan這兩位神人在1984年發明了Fibonacci heap
這詭異又神奇的資料結構我就不多說了
但是不能不說他神奇的複雜度
兩個Fibonacci heap合併只要{% katex [false] %}O(1){% endkatex %}
也就是說插入也只要{% katex [false] %}O(1){% endkatex %}(建立只有一個元素的heap並合併)
只要是不涉及刪除的操作都是{% katex [false] %}O(1){% endkatex %}
刪除也只要{% katex [false] %}O(logn){% endkatex %}
此外還新增了decrease-key的功能</br>
這個decrease-key可以{% katex [false] %}O(1){% endkatex %}減少某個節點的值</br>
把這些操作套到Dijkstra's Algorithm之後
插入{% katex [false] %}O(V){% endkatex %}
修改值{% katex [false] %}O(E){% endkatex %} (因為最短路不會變長)
取出{% katex [false] %}O(VlogV){% endkatex %}
時間複雜度合計{% katex [false] %}O(E+VlogV){% endkatex %}</br>
雖然說Fibonacci heap那神一般的複雜度都是均攤出來的
而且常數可能會把你嚇死
但是這仍然是非常了不起的優化
也許哪天我會出一篇Fibonacci heap實作吧!
{% endblockquote %}

又是一下午的時間消失了QAQ
