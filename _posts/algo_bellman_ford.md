---
title: 演算法週記 單源最短路徑之Bellman-Ford演算法
tag: ["algorithm","SSSP"]
categories: Algorithm-Weeekly-Journal
---
### 最短路徑
{% blockquote %}
什麼是最短路徑呢?
首先最短路徑要有起點({% katex [false]%}S{% endkatex %})跟終點({% katex [false]%}T{% endkatex %})
從{% katex [false]%}S{% endkatex %}走到{% katex [false]%}T{% endkatex %}經過的邊權總和最短的一條路徑就稱為"最短路徑"(Shortest Path)
而這個邊權和就是"最短距離"

最短路徑有好多種演算法
今天我們來看看Bellman-Ford演算法
{% endblockquote %}

<!-- more -->

### Bellman-Ford Algorithm
{% blockquote %}
這是個相當常用的演算法
他除了可以找出最短路徑之外
還能找出負環</br>
將從起點{% katex [false] %}S{% endkatex %}到終點{% katex [false] %}Y{% endkatex %}的最短路徑設為{% katex [false] %}d[y]{% endkatex %}
那麼怎麼找出{% katex [false] %}d[y]{% endkatex %}呢?
首先呢，{% katex [false] %}S{% endkatex %}到{% katex [false] %}Y{% endkatex %}的路徑一定是從某個頂點{% katex [false] %}X{% endkatex %}由邊{% katex [false] %}e(x,y){% endkatex %}延伸過去的
那麼假設{% katex [false] %}S{% endkatex %}到{% katex [false] %}X{% endkatex %}的最短路徑都已經確定了
則下列的式子一定會成立
{% katex [false] %}d[y] = min\{d[x]+cost(x,y)|e(x,y) \in E\}{% endkatex %}
其中{% katex [false] %}cost(x,y){% endkatex %}代表邊{% katex [false] %}e(x,y){% endkatex %}的權重</br>
那麼要怎麼確保所有的{% katex [false] %}X{% endkatex %}的最短路都已經被確定了呢?</br>
其實不需要這麼做
因為最短距離只會變小不會變大
所以只要在能讓最短距離變小的時候更新
到最後就會是真的最短距離了喔!</br>
以下就是Bellman-Ford Algorithm的核心
1.將{% katex [false]%}d[]{% endkatex %}初始化成INF
2.將{% katex [false]%}d[s]{% endkatex %}設為{% katex [false]%}0{% endkatex %}
3.掃過所有的邊{% katex [false]%}e(x,y){% endkatex %}
4.對於每個邊{% katex [false]%}e(x,y){% endkatex %}的{% katex [false]%}x,y{% endkatex %}如果{% katex [false]%}d[x]{% endkatex %}不是初始值且{% katex [false]%}d[y] > d[x]+cost(x,y){% endkatex %}成立
5.則更新{% katex [false]%}d[y] = d[x]+cost(x,y){% endkatex %}
6.如果掃過一次所有邊有任何更新則跳回步驟3否則繼續步驟7
7.結束演算法</br>
如果看不是很懂的話可以參考下面的程式碼</br>
接著我們來分析一下他的複雜度吧!</br>
考慮一下沒有負環的情況(路徑中有負環的話，最短路徑不存在，迴圈也跑不完)
在沒有負環的情況下
最短路徑不會重複通過同個頂點
那麼今天有條很長的最短路徑
對於路徑上的每個點來說
這條路徑一定是最短路</br>
考慮某個已經被確定的頂點{% katex [false]%}A{% endkatex %}的連外頂點{% katex [false]%}B{% endkatex %}
{% katex [false]%}B{% endkatex %}的最短路徑上的前一個點是{% katex [false] %}A{% endkatex %}的話
則最多在{% katex [false]%}A{% endkatex %}被確定後的下個迴圈
一定也會被確定(也可能在同個迴圈就被確定)
也就是說迴圈最多會跑{% katex [false]%}V{% endkatex %}次({% katex [false]%}V-1{% endkatex %}個待確認的頂點+一次沒有用的迴圈)
所以說時間複雜度是{% katex [false]%}O(VE){% endkatex %}鄰接矩陣的話則是{% katex [false]%}O(V^3){% endkatex %}</br>
利用最多會跑{% katex [false]%}V{% endkatex %}次迴圈的這個特性
我們可以統計迴圈的次數
如果超過{% katex [false]%}V-1{% endkatex %}次還在更新的話
就能確定這張圖有負環
{% endblockquote %}

以下程式碼由C++撰寫
#### edge struct版
``` C++
#include <iostream>
using namespace std;

const int INF = 1000000000; //1e9

//邊from->to, 費用cost
struct edge {
    int from, to, cost;
};

int V, E; //頂點數、邊數
int d[105]; //所有點到S的距離
edge es[10005]; //邊集合

void bellman_ford(int s) {
    for(int i = 0; i < V; i++) d[i] = INF; //初始化距離
    d[s] = 0; //源點距離=0
    while(true) {
        bool update = false;
        for(int i = 0; i < E; i++) {
            edge &e = es[i];
            if(d[e.from] != INF && d[e.to] > d[e.from]+e.cost) {
                d[e.to] = d[e.from] + e.cost;
                update = true;
            }
//            if(d[e.to] != INF && d[e.from] > d[e.to]+e.cost) {
//                d[e.from] = d[e.to] + e.cost;
//                update = true;
//            } //無向圖
        }
        if(!update) break; //沒更新就退出
    }
}

//demo
int main() {
    cin >> V >> E;
    for(int i = 0; i < E; i++) {
        cin >> es[i].from >> es[i].to >> es[i].cost;
    }
    bellman_ford(0);
    for(int i = 0; i < V; i++) {
        cout << "d[" << i << "] = " << d[i] << endl;
    }
    return 0;
}

```

#### 鄰接矩陣版
``` C++
#include <iostream>
using namespace std;

const int INF = 1000000000; //1e9

int V, E; //頂點數、邊數
int G[105][105]; //有邊則G[i][j] != INF
int d[105]; //所有點到S的距離

void bellman_ford(int s) {
    for(int i = 0; i < V; i++) d[i] = INF; //初始化距離
    d[s] = 0; //源點距離=0
    while(true) {
        bool update = false;
        for(int i = 0; i < V; i++) {
            for(int j = 0; j < V; j++) {
                if(G[i][j] != INF && d[i] != INF && d[j] > d[i]+G[i][j]) {
                    d[j] = d[i]+G[i][j];
                    update = true;
                }
            }
        }
        if(!update) break; //沒更新就退出
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
    bellman_ford(0);
    for(int i = 0; i < V; i++) {
        cout << "d[" << i << "] = " << d[i] << endl;
    }
    return 0;
}
```

本來想多寫幾種SSSP的演算法
可是光一個Bellman-Ford就寫了一下午OAO
真的很好奇我是不是真的能一週一篇=ㄦ=