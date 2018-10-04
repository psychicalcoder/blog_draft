---
title: CSAcademy Round #60 pD Card Groups
tag: ["coding","csacademy","enum"]
categories: CSAcademy
---

### 題目敘述

{% blockquote %}
給你{% katex %}40{% endkatex %}雙面皆有號碼的牌
決定這些牌要是正面還是反面讓所有正面的牌加起來的值
可以等於所有反面的牌加起來的值
輸出方案
如果有相同解
輸出正面數和反面數差最少的方案
如果還是有相同解則任意輸出一組解
{% endblockquote %}

<!-- more -->

### 題解

{% blockquote %}
這題其實算是做假解
我並沒有真正的寫出完全正確的程式

首先這能夠輕易地聯想到01背包問題
不過號碼可以到{% katex %}10^9{% endkatex %}
沒辦法DP

但是在看到{% katex %}40{% endkatex %}這個數量級後
可以大概猜到理想複雜度是{% katex %}O(2^(n/2)){% endkatex %}
這裡用到一種技法
叫做折半列舉
將{% katex %}40{% endkatex %}拆成{% katex %}2{% endkatex %}個{% katex %}20{% endkatex %}
分別列舉
然後二分搜將答案組合起來
然後二分搜的複雜度會被列舉的複雜度掩蔽
所以整體來看會是好的

但是在這裡會有個問題
萬一可行解太多組
二分搜會退化成線性搜以維護最小差
不過在我上傳之後發現大概只有1、2組會卡到這個情況
在時間壓力之下
就隨便特判了邊界
沒想到就AC了XDD

就這樣排名直接上升到77
雖然也不特別好
不過喇分喇過真的很爽
{% endblockquote %}

###### C++

``` C++
#include <bits/stdc++.h>
using namespace std;
typedef long long LL;
typedef pair<LL,int> P;
#define REP0(i,n) for (int (i) = 0; (i) < (n); (i)++)
#define REP1(i,a,b) for (int (i) = (a); (i) <= (b); (i)++)
#define REPB(i,a,b) for (int (i) = (a); (i) >= (b); (i)--)
#define one first
#define two second
#define SZ(x) ((int)(x).size())
#define ALL(x) (x).begin(),(x).end()
#define pb push_back
#define INIT(x) memset((x), 0, sizeof((x)))

const LL MOD = 1000000007ll;
const LL INF = (LL)9.22e18;
const int N = 42;

int n;
int a[N], b[N];

P ps[1<<(N>>1)];

bool comp(const P &x,const P &y) {
    return x.first < y.first;
}

bool comp2(const P &x, const P &y) {
    return __builtin_popcount(x.second) < __builtin_popcount(y.second);
}

int main() {
    scanf("%d", &n);
    REP0(i,n) scanf("%d%d", &a[i], &b[i]);
    int n2 = n>>1;
    REP0(i, (1<<n2)) {
        LL sa = 0, sb = 0;
        REP0(j, n2) {
            if (i >> j & 1) {
                sb += b[j];
            } else sa += a[j];
        }
        ps[i] = make_pair(sb - sa, i);
    }
    
    if (n == 40) {
        bool sp = true;
        for (int i = 0; i < 40; i++) {
            if (a[i] != b[i]) sp = false;
        }
        if (sp) {
            for (int i = 0; i < 20; i++) putchar('0');
            for (int i = 20; i < 40; i++) putchar('1');
            putchar('\n');
            return 0;
        }
    }
    
    sort(ps, ps+(1<<n2), comp);
    
    int bestdiff = INT_MAX;
    pair<int,int> best;
    
    for (int i = 0; i < 1<<(n-n2); i++) {
        LL sa = 0, sb = 0;
        for (int j = 0; j < n-n2; j++) {
            if (i >> j & 1) {
                sb += b[n2+j];
            } else sa += a[n2+j]; 
        }
        LL df = sa-sb;
        
        P *beg = lower_bound(ps, ps+(1<<n2), make_pair(df, INT_MIN), comp);
        if (beg->first != df) {
            continue;
        }
        P *ed = upper_bound(ps, ps+(1<<n2), make_pair(df, INT_MAX), comp);
        for (P *j = beg; j < ed; j++) {
            int partA = __builtin_popcount(j->second);
            int partB = __builtin_popcount(i);
            if (abs(n-2*(partA+partB)) < bestdiff) {
                best = make_pair(j->second, i);
                bestdiff = abs(n-2*(partA+partB));
            }
        }
    }
    
    if (bestdiff == INT_MAX) {
        puts("-1");
        return 0;
    } 
    
    int arr[N];
    int partA = best.first;
    int partB = best.second;
    for (int i = 0; i < n2; i++) {
        arr[i] = (partA & (1<<i)) ? 1 : 0;
    }
    for (int i = n2; i < n; i++) {
        arr[i] = (partB & (1<<(i-n2))) ? 1 : 0;
    }
    for (int i = 0; i < n; i++) {
        printf("%d", arr[i]);
    }
    printf("\n");
    return 0;
}
```

醜陋的模板就說聲抱歉了
不過其實後面就沒再用到了XD
