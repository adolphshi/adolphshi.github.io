---
title: P9565 Not Another Path Query Problem题解
date: 2024-04-23 19:33:47
tags:
  - OI
  - 寄巧
cover: https://s1.imagehub.cc/images/2024/05/28/42d4927fc8e00609ea420210f691fe72.png
categories: 题目相关
---

**一道非常好的二进制拆分与并查集练手题**

[题目传送门](https://www.luogu.com.cn/problem/P9565)

## 题目摘要

给出一个 $n$ 个点, $m$ 条边的无向图，每一条边有一个权值 $w$ ,共有 $q$ 此询问，每一次查询两个点是否有一条权值按位与起来大于等于 $V$ 的路径。$V$ 的值固定。

## 分析

不难发现，每一次按位与运算过后，值总是不增的。
对于两个数 $x,y$ 想要让他们按位与后的值大于 $V$ ，就要保证存在一个位数 $i$ 使得 $x \text{AND} y$ 的前 $i$ 位大于 $V$  的前 $i$ 位，也就是如果 $V$ 的这一位为一，那么 $x,y$ 的这一位都需要是一。于是我们从高位到低位去判断它这一位与 $V$ 的这一位的关系。

最后再特判一下 $V$ 与 $x$ 是否相等，相等也要计算，因为这是符合题意的。

因为 $V\leq 2^{60}$ 所以我们选择开 $61$ 个并查集，第 $i$ 个并查集代表一个并查集内的元素的前 $i$ 位的按位与值大于 $V$ 的前 $i$ 位。最后一个并查集代表与 $V$ 相同的元素。如果两个节点中间的边的权值满足上述条件，就将两个节点放入同一个并查集中。

最后查询就是在每一种并查集中查找是否在同一个并查集中。在就输出 `Yes` 。

## code

```cpp
#include <bits/stdc++.h>
#define _F(x,y,z) for(int x=y;x<=z;x++)
#define F_(x,z,y) for(int x=z;x>=y;x--)
#define TF(x,y,z) for(int x=head[y],z;x;x=nex[x])

using namespace std;

typedef long long ll;
typedef double dou;
typedef const int ci;

ci maxn=1e5+10;
ll n,m,q,v,fa[maxn][65],c[65],w;
int find(int x,int y)
{
  if(x==fa[x][y])
    return x;
  return fa[x][y]=find(fa[x][y],y);
}
int main()
{
  scanf("%lld%lld%lld%lld",&n,&m,&q,&v);
  _F(i,1,n)
    _F(j,0,61)
      fa[i][j]=i;
  F_(j,60,0)
    if(v&(1ll<<j))
      c[j]=1;
  _F(i,1,m)
  {
    int x,y;
    scanf("%d%d%lld",&x,&y,&w);
    F_(j,60,0)
    {
      bool fl=w&(1ll<<j);
      if((!c[j])&fl) fa[find(x,j)][j]=find(y,j);
      else if(c[j]&(!fl)) break;
    }
    if((w&v)>=v) fa[find(x,61)][61]=find(y,61);
  }
  _F(i,1,q)
  {
    int x,y,fl=0;
    scanf("%d%d",&x,&y);
    F_(j,61,0)
    {
      if(find(x,j)==find(y,j))
      {
        puts("Yes");
        fl=1;break;
      }
    }
    if(!fl)
      puts("No");
  }
  return 0;
}
```


 