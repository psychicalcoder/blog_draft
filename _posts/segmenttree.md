---
title: 模板-線段樹
tag: ["coding","data-structure","template"]
categories: Template
---

有個數奧很電的人叫我要學"樹"才會進步，我覺得蠻有道理的
所以今天就來學習寫線段樹吧!

這次的線段樹是參考建中講義的樣板
並且以指標式結構實做

<!-- more -->

指標式有什麼好處呢?
首先就是會有物件導向的清晰感(?
反正是程式碼比較易讀
再來就是實做持久化會比較容易
需要動態配點的時候可以節省一些記憶體

有好處必有壞處嘛
他在一般的使用之下因為要多存幾個指標所以會用到比較多的記憶體
然後很容易寫爆!如果不懂指標的真正意涵那最好先弄懂再來寫

我相信大家對線段樹多少有點概念
這裡就不再綴述

這次我還有實做lazytag懶標記
以便於區間更新

什麼叫做懶標記呢?
如果不實做懶標記
一次要對一個區間修改值時的複雜度就會變成{%katex [false]%}O(nlogn){%endkatex%}
會變得非常的慢
但是如果區間更新的時候不要一次就更新到葉節點
等到要查詢的時候在更新下去那麼複雜度仍然一樣是{%katex [false]%}O(logn){%endkatex%}
我知道很多人沒看懂，看不懂就看程式碼吧(反正這不是奇怪的圖論演算法?

為什麼這樣是{%katex [false]%}O(logn){%endkatex%}
因為你不需要將每個節點都更新到
例如{%katex [false]%}[0,4){%endkatex%}這個區間
如果不實做懶標記你需要更新的節點有{%katex [false]%}[0,1),[1,2),[2,3),[3,4),[0,2),[2,4),[0,4){%endkatex%}這七個節點
但是如果實做懶標記的話
只需要更新{%katex [false]%}[0,4){%endkatex%}這個節點就好了
等到哪天也許你要再次更新{%katex [false]%}[1,3){%endkatex%}這個區間時
再去更新懶標記就行了
那要怎麼更新呢?
我們拿剛剛的{%katex [false]%}[1,3){%endkatex%}做例子
這時{%katex [false]%}[0,4){%endkatex%}上已經有個懶標記了
要遞迴到{%katex [false]%}[1,3){%endkatex%}之前會先遞迴到{%katex [false]%}[0,4){%endkatex%}
這時將懶標記往下壓並清除自己身上的懶標記
懶標記就跑到{%katex [false]%}[0,2),[2,4){%endkatex%}上了
接著遞迴到{%katex [false]%}[0,2){%endkatex%}
重複同樣的動作
懶標記會跑到{%katex [false]%}[0,1),[1,2){%endkatex%}上
就照這樣一步一步的推
每次查詢或更新最多都只要{%katex [false]%}O(logn){%endkatex%}次
而將懶標記往下壓的動作是{%katex [false]%}O(1){%endkatex%}的
所以複雜度不變

線段樹差不多就講到這拉
下面是code
不過是{%katex [false]%}[l,r]{%endkatex%}版本的
因為我左閉右開版的一直寫壞掉
(可以配合區間最大值題目使用)
有bug請email給我，感謝
有不懂得地方就看看code吧
當然也歡迎email給我喔!

###### C/C++

``` C
#include <cstdio>
#include <algorithm>
#include <cstring>
using namespace std;
typedef long long LL;

const LL INF = (LL)1e18;
const int N = 100005;

struct node {
    LL val, tag;
    node *lc, *rc;
    node() {
        tag=0;
        val=0;
        lc = NULL;
        rc = NULL;
    }
    void pull () {
        val = max(lc->val, rc->val);
    }
};

int n, q, qs[N];

node* build(int l, int r) {
    node* a = new node();
    if (l==r) {
        a->val = qs[l];
        return a;
    }
    int mid = (l+r) >> 1;
    a->lc = build(l,mid);
    a->rc = build(mid+1,r);
    a->pull();
    return a;
}

void push(node* a, int l,int r) {
    if (!a->tag) return;
    if (l!=r) {
        int mid= (l+r) >> 1;
        a->lc->tag += a->tag;
        a->rc->tag += a->tag;
        a->lc->val += a->tag;
        a->rc->val += a->tag;
    }
    a->tag = 0;
}

void modify(node* a, int l, int r, int ql, int qr, LL d) {
    if (ql > r || qr < l) return;
    if (ql <= l && r <=qr) {
        a->tag += d;
        a->val += d;
        return;
    }
    push(a, l, r);
    int mid = (l+r) >> 1;
    modify(a->lc, l,mid,ql,qr,d);
    modify(a->rc, mid+1,r,ql,qr,d);
    a->pull();
}

LL query(node* a, int l,int r, int ql, int qr) {
    if (ql > r || qr < l)  return -INF;
    if (ql <= l && r <= qr) return a->val;
    push(a, l, r);
    int mid = (l+r) >> 1;
    return max(query(a->lc, l, mid, ql, qr), query(a->rc, mid+1, r , ql, qr));
}

//demo
int main() {
    memset(qs, 0, sizeof(qs));
    scanf("%d%d", &n, &q);
    for(int i = 1; i <= n; i++){
        scanf("%d", qs+i);
    }
    node *root = build(1, n);
    while(q--) {
        int k, l, r, x;
        scanf("%d%d%d", &k, &l, &r);
        if(k == 1) {
            scanf("%d", &x);
            modify(root, 1, n, l, r, x);
        } else {
            printf("%lld\n", query(root, 1, n, l, r));
        }
    }
    return 0;
}

```

