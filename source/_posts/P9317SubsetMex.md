---
title: P9317 [EGOI2022] SubsetMex / 子集 mex 
date: 2024-11-16 16:38:07
toc: true
tags: 算法
categories: 算法
---

## 题意

选取集合$S$的一个子集$T$,把$T$从集合中删除,把$mex(T)$加入集合中,问使得$mex(S)==n$的最少操作次数

## 思路

把$x$加入集合的条件是$[1,x-1]$在集合中，至少出现一次  

如果把$x$加入集合则$x$的出现次数增加$1$,$[1,x-1]$的出现次数减少$1$   

$x$增加的出现次数即为操作次数

## AC代码

```c++
#include <bits/stdc++.h>
#define int long long

using namespace std;

int n;
const int N = 55;
int a[N];

signed main()
{
    int t, tmp;
    cin >> t;
    while(t--)
    {
        memset(a, 0, sizeof a);
        int ans = 1;
        cin >> n;
        for(int i = 0; i < n; i++)
        {
            cin >> tmp;
            a[i] = tmp-1; // 显然的,要把n加入集合，[1,n-1]的出现次数会减1
        }
        for(int i=n-1;i>=0;i--)
        {
            if(a[i]<0) // 如果出现次数不足,需要增加出现次数
            {
                ans -=a[i];
                for(int j=0;j<i;j++)
                {
                    a[j] += a[i]; // 要增加i的出现次数需要减少[0,i-1]的出现次数
                }
            }
        }
        cout << ans << "\n";
    }
    return 0;
}

```

