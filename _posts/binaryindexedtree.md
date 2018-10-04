---
title: 模板-Binary Indexed Tree
tag: ["coding","data-structure","template"]
categories: Template
---
今天要來講BinaryIndexedTree
BinaryIndexedTree 俗稱 BIT
台灣好像沒有翻譯??
大陸人稱之為"樹狀數組"
我覺得這個說法蠻不錯的
BIT長的不像棵樹但是擁有樹的精神(X
他的起源比線段樹還要早
這還蠻奇怪的，BIT應該比線段樹還要難想到才對
<!-- more -->
廢話不多說
到底什麼是BIT?
先來說說BIT的功能好了
BIT可以在{% katex [false] %}O(logn){% endkatex %}的時間查詢前綴合以及修改單點
(修改區間以後有空再說
有了前綴合就等於有區間合了!
而且BIT的運作比起線段樹更為快速
佔用的記憶體也比較小
實做的難度更是低
唯獨要理解他的運作原理有些難度

在瞭解BIT的運作原理前
要先知道
``` C++
x & -x;
```
代表x以二進位來看最低位的1
這是因為負整數是以補數用法表示
想瞭解更多請自己Google囉

接下來呢
正式進入BIT了(什麼~現在才進入??

先來看看這張圖
當初在演算法筆記上看到類似的圖時才比較瞭解BIT
(這是我用Excel做的XDD)
{% img /images/bit.png %}

要用數學方式來解說原理的話也不會很難

任何一個正整數都能用很多個{% katex [false] %}2^i | i \in \mathbb{N} \cup \{0\}{% endkatex %}就像二進位一樣
那麼BIT就是將一段陣列分割成很多個{% katex [false] %}2^i{% endkatex %}來分別求解
例如說11可以分割成{% katex [false] %}2^3 + 2^2 + 2^0{% endkatex %}
可以轉換成二進位1101
刪去最低位的1，切割成 1101 ~ 1100+1，剩下1100
刪去最低位的1，切割成 1100 ~ 1000+1，剩下1000
刪去最低位的1，切割成 1000 ~ 0000+1，剩下0000
再搭配上面那張圖是不是就比較好理解了呢??

如果有不懂得地方可以Email給我喔!!(在About可以看到
###### C/C++

``` C++
#include <bits/stdc++.h>
using namespace std;
typedef long long LL; 

const int N = 1000000;

int n;
int BIT[N];

int low_bit(int x) {
    return x & -x;
}

void add(int idx, int val) {
    while(idx <= n){
        BIT[idx] += val;
        idx += low_bit(idx);
    }
}

int sum(int idx) {
    int ans = 0;
    while(idx > 0) {
        ans += BIT[idx];
        idx -= low_bit(idx);
    }
    return ans;
}

//Demo
int main(){
    memset(BIT,0,sizeof(BIT));
    scanf("%d", &n);
    int a, i, v;
    while(scanf("%d", &a) == 1 && a){
        if(a == 1) {
            scanf("%d%d", &i, &v);
            add(i,v);
            printf("ADD(%d,%d) successfully\n", i, v);
        } else if(a == 2) {
            scanf("%d", &i);
            printf("SUM(%d) = %d\n\n", i, sum(i));
        } else {
            puts("Please Input 0(exit),1(add),2(sum)\n");
        }
    }
    return 0;
}
```

