---
title: 插头DP学习笔记
date: 2024-03-26 19:28:12
tags: 
	- OI
	- 算法
	- DP
cover: https://s1.imagehub.cc/images/2024/05/28/b22351df7ca20de20fb902074a0e7b13.png
categories: 算法学习
---

@[toc]

# 插头DP

## 定义

插头DP并不是字面~~也不是封面~~意义上的“在插头上DP”，而是一种基于连通性状态压缩的动态规划问题，它通常出现在棋盘问题中，可以解决棋盘中回路、铺地板、连通性等问题。
而在学习插头DP之前，还需要了解一下轮廓线DP。

## 轮廓线DP

在面对棋盘上的DP问题时，如果数据范围不大，可以使用状态DP枚举每一行，进行逐行解决。

而另一种解决方案就是进行在轮廓线上状压，然后进行逐格转移。

至于什么是轮廓线，如图：

![OIwiki的图](https://oi-wiki.org/dp/images/contour_line.svg)

可得，轮廓线其实就是已完成转移的方格，与未完成转移的方格之间的分界线。

而在轮廓线上进行状压则是将当前轮廓线的状态压成数字进行储存，再对下一格进行转移,时间复杂度不变。

## 插头DP

那我们在进行轮廓线DP时，状压什么样的轮廓线呢？答案就是状压该轮廓上的插头。

而插头又是什么呢？OIwiki的表述如下：

>一个格子某个方向的插头存在，表示这个格子在这个方向与相邻格子相连。

然而并不太好理解，通俗点来说，就是一个箭头，表示你只能向这一个方向走，而在走到的目标格，必须也有一个箭头与之相对（玩过接水管游戏的可以类比一下，很像），如图：

![OIwiki的图](https://oi-wiki.org/dp/images/plug.svg)

而我们所状压的，就是插头的状态(例如：类别等需要根据题目设定)。

另：因为插头的状态多变，所以我们通常用四进制进行状压。

### 插头的状压与转移

为了方便状压，我们选择将轮廓线上的插头进行编号一般编号如下：

（**编号为i的插头在四进制（或二进制）编码中排第 $n-1$ 位**）

![编号](https://s1.imagehub.cc/images/2024/05/28/2998d13d8b5ac519ae07d93ff5d49251.gif)

转移后

![转移后的编号](https://s1.imagehub.cc/images/2024/05/28/f6c58882895b6232a7a93f4971af9bb6.gif)

这里面要非常注意一下当中右插头（见下） 的标号变化，理解了它你就能理解插头DP转移过程中的状态变化。


此外，为了节省空间，我们采用滚动数组的方式进行DP，因此只进行保存上一个格所推出的结果。

------

那么现在编号已经完毕，接下来就是转移了，上文已经说过了，转移是逐格转移，那么转移时插头的变化如下：

![逐格转移](https://s1.imagehub.cc/images/2024/05/28/179bffee4d48997853b17d8075e5ca54.gif)

可以发现，转移时，我们需要关注当前格上方向下的**下插头**和当前格左边向右的**右插头**。（因为这两个插头我们必须回应，所以会影响我们的转移）  

我们还需要根据上面下插头与右插头的状态来设计转移。

换行时要添加新的空插头：

![换行](https://s1.imagehub.cc/images/2024/05/28/f7600d017f5fd6150b68a0fffbdc6d05.gif)

------



### 手写哈希

在有一部分题目中，用状态代表的四进制会非常稀疏，直接枚举时间复杂度会非常大，因此我们通常会在插头DP的过程中手写哈希，当然如果题目的时间充裕，我们也可以使用`unordered_map`来解决这个问题。**(在部分题目中，使用`unordered_map`会超时，例如[Lg P 3272](https://www.luogu.com.cn/problem/P3272))**

代码会在code部分统一讲解。

## code

插头DP的码量**巨大**，因此我会进行分步骤讲解，需要注意的细节将会在代码中标出。

在了解插头DP之前，我们需要先解释一下代码中各个数组的用处：

+ `mp[][]` 为棋盘，用于存储障碍物
+ `bit[]` 为 $2^{2\times i}$ ,方便取状压后的数
+ `tot[2]` 为计数数组，用于记录状态个数（使用滚动数组）
+ `head[] nex[]` 哈希表挂链
+ `st[2][]` 为状态数组，用于存储轮廓线的状态。(使用滚动数组)
+ `dp[2][]`  为dp数组，用于存储结果。（使用滚动数组）

{% spoiler  手写哈希%}
``` cpp
void insert(int s,int v)//s为状态 v为值
{
	int tmp=s%p;//p为hash模数
	for (int i = head[tmp];i; i=nex[i])
	{
		if(st[pos][i]==s)//如果已经存在
		{
			dp[pos][i]+=v;//累加答案
			return ;
		}
	}
	tot[pos]++;//否则新建一个节点
	st[pos][tot[pos]]=s;
	dp[pos][tot[pos]]=v;
	nex[tot[pos]]=head[tmp];
	head[tmp]=tot[pos];
}
```
{% endspoiler %}

{% spoiler  DP框架%}
``` cpp
void DP()
{
	tot[pos]=1,dp[pos][1]=1,st[pos][1]=0;//初始化
	//n，m为棋盘大小，lst为pos^1，_F(i,x,y)为用i从x到y遍历
	_F(i,1,n)
	{
		_F(j,1,tot[pos]) st[pos][j]<<=2;//创建空插头
		_F(j,1,m)
		{
			pos^=1;
			tot[pos]=0,memset(head,0,sizeof head);//每转移一格清空一次哈希
			_F(k,1,tot[lst])
			{
				ll s=st[lst][k],v=dp[lst][k];//s为当前轮廓线状态
				ll pl=(s>>bit[j-1])%4,pd=(s>>bit[j])%4;//取到当前格的下插头与右插头
				//后面根据题意进行转移
			}
		}
	}
}
```
{% endspoiler %}

好了，代码就这么简单。但是基本50%的代码都在转移过程中，而转移又视情况而定。（其实很简单，只要明确插头的状态即可）
每一道题的代码具体问题具体分析。

## 例题

### P5056 【模板】插头 DP ——插头DP维护回路问题

给出 $n\times m$ 的方格，有些格子不能铺线，其它格子必须铺，形成一个闭合回路。问有多少种铺法?

首先我们先定义插头的状态，我们发现对于一条回路，对于其中从左到右的四个插头 $a b c d$ 这四个插头肯定匹配，且不会互相交叉（即 $a c$ 相连，$b d$ 相连 ），如图：
![1](https://s1.imagehub.cc/images/2024/06/11/e437618a2a45e00ef1a3c8322f979870.md.png)
![2](https://s1.imagehub.cc/images/2024/06/11/ea49953da63266fdc53a2f96f44fd965.md.png)
![3](https://s1.imagehub.cc/images/2024/06/11/4d19d8066df9cb2950f04c671fba3e5d.md.png)
图一、图二肯定没有问题，而图三很显然交叉了是无法形成回路的。

综合这个性质，我们会发现，这里面的状态与括号匹配十分类似，所以我们设0代表无插头，1代表左括号，2代表右括号那么上面那两幅图就可以表示为 $((#)#)#$ 即 $1 1 0 2 0 2 0$ 和 $()#(#)#$ 即 $1 2 0 1 0 2 0$ 。（其中 $#$ 代表无插头）

恭喜你，你就学会了插头DP中较为常用来表示状态的括号表示法，这很重要。

那么接下来我们就根据下插头和右插头的状态来设计转移吧！

下面一共分为八种情况，我们一个一个分析(其中蓝色箭头代表1号，红色箭头代表2号，紫色表示任意，其中当前格有插头的意思为有指向这个格的插头)：
1. 当前格为障碍物，当且仅当没有下插头与右插头时进行转移![1](https://s1.imagehub.cc/images/2024/06/11/a71927ea5a51651f3a5c4df7dfe4a941.png)
2. 当前格没有插头，此时因为这个点必须被经过，所以创建两个新的插头![2.png](https://s1.imagehub.cc/images/2024/06/11/eea30c7a37945e98031ee816ad5033b2.png)
3. 当前格有一个右插头，此时可以创建一个新的，状态与右插头相同的下插头或者是右插头，当且仅当所指向的格不是障碍物才进行转移。![3.png](https://s1.imagehub.cc/images/2024/06/11/ce9e7ed2f3a1daed2b7b1a1abc930fe5.png)
4. 当前格有一个下插头，与只有一个右插头类似地进行转移。![4.png](https://s1.imagehub.cc/images/2024/06/11/c75a7a6f4ed6d8fc559f9d9cd36914b9.png)
5. 当前格有下插头和右插头，且状态均为1。此时我们会发现我们在将这两个插头合并（删除）时，需要改变与之匹配的靠左的状态为2的插头的状态![5.png](https://s1.imagehub.cc/images/2024/06/11/124bb70887dde8362f9b3ccf06750928.png)![55.png](https://s1.imagehub.cc/images/2024/06/11/89db1f451edde399cd86b81b2bbd48b1.png)
6. 当前格有下插头和右插头，且状态均为2。此时我们在将两个插头合并时需要改变与之匹配的靠右的状态为1的插头的状态。![6.png](https://s1.imagehub.cc/images/2024/06/11/884b93781f3db105728053278d4c8bcf.png)![66.png](https://s1.imagehub.cc/images/2024/06/11/238bb0a934cf091fe7e22f09155d9d50.png)
7. 当前格有一个状态为2的右插头，且有一个状态为1 的下插头，此时我们将这两个插头直接合并（删除）即可。![7.png](https://s1.imagehub.cc/images/2024/06/11/3e7bc0c1d0f2869f8cc38fd837256012.png)![77.png](https://s1.imagehub.cc/images/2024/06/11/8e384491a84cea0f333c6d1dc3308d77.png)
8. 当前个有一个状态为1的右插头和一个状态为2的下插头，若此时为最后一个可以进行转移的格子，那么进行统计答案，若不是，则不统计答案。![8.png](https://s1.imagehub.cc/images/2024/06/11/5f65d94d5553ddc92c40439f3068aa23.png)![88.png](https://s1.imagehub.cc/images/2024/06/11/0104942735228c989fd00b3eea11458a.png)

在进行完对转移的分类讨论之后，我们就可以写出如下的代码：

{% spoiler  转移部分代码%}
``` cpp
if(!mp[i][j])//case 1
{
	if((!pl)&&(!pd)) 
		insert(s,v);
}
else if((!pl)&&(!pd))//case 2
{
	if(mp[i][j+1]&&mp[i+1][j])//如果不为障碍物，在这类题中，一般将可行走的点置成1（这样就不需要判出界）
		insert(s+(1<<bit[j-1])+2*(1<<bit[j]),v);//转移过程中注意编号的变化
}
else if(pl&&(!pd))//case 3
{
	if(mp[i][j+1])
		insert(s-pl*(1<<bit[j-1])+pl*(1<<bit[j]),v);
	if(mp[i+1][j])
		insert(s,v);//这里不进行改变的原因是因为当前个的下插头在下一格中进行转移时为那一格的右插头（看动图）
}
else if((!pl)&&pd)//case 4
{
	if(mp[i+1][j])
		insert(s-pd*(1<<bit[j])+pd*(1<<bit[j-1]),v);
	if(mp[i][j+1])
		insert(s,v);//同上
}
else if(pl==1&&pd==1)//case 5
{
	int cnt=1;
	_F(l,j+1,m)//找第一个匹配的2号插头，用栈进行括号匹配
	{
		if((s>>bit[l])%4==1) cnt++;
		else if((s>>bit[l])%4==2) cnt--;
		if(!cnt)
		{
			insert(s-(1<<bit[j])-(1<<bit[l])-(1<<bit[j-1]),v);//删除一对插头，并且修改2号为1
			break;
		}
	}
}
else if(pl==2&&pd==2)//case 6
{
	int cnt=1;
	F_(l,j-2,0)//同上，注意顺序
	{
		if((s>>bit[l])%4==1) cnt--;
		else if((s>>bit[l])%4==2 )cnt++;
		if(!cnt)
		{
			insert(s-2*(1<<bit[j])-2*(1<<bit[j-1])+(1<<bit[l]),v);//这里也注意一下
			break;
		}
	}
}
else if(pl==2&&pd==1)//case 7
{
	insert(s-2*(1<<bit[j-1])-(1<<bit[j]),v);//直接删除一对括号
}
else if(i==ex&&j==ey&&pl==1&&pd==2)//case 8 ex、ey 为我们找好的最后一个可以进行转移的点
{
	ans+=v;//统计答案
}
}
```
{% endspoiler %}
### P3190 [HNOI2007] 神奇游乐园

给出一个的棋盘，每一个点都有权值，求一条回路（不能交叉），使得路上所经过的点的权值和最大。

思路很简单，求不需要经过每一个点的回路只需将最后一种可能性中，对于最后一个点的特判，改为状态中只含有这两个插头的判断即可。（具体改动见代码）

至于如何选择权值最大，只需将dp过程中的累加转为取最大，在每一次插入过程中加上当前点的权值即可。

另外，还要在注意的一点是，在本题中对于向右的插头需要判断是否出界.

{% spoiler  代码%}
``` cpp
#include <bits/stdc++.h>
#define _F(x,y,z) for(int x=y;x<=z;x++)
#define F_(x,z,y) for(int x=z;x>=y;x--)
#define TF(x,y,z) for(int x=head[y],z;x;x=nex[x])
#define lst pos^1

using namespace std;

typedef long long ll;
typedef double dou;
typedef const int ci;

ll n,m,a[110][20],dp[2][20010],st[2][20010],tot[2];
ll pos,ans,bit[20],mp[20010];
void ins(ll s,ll v)
{
	if(mp[s])
	{
		dp[pos][mp[s]]=max(dp[pos][mp[s]],v);//改动1
		return ;
	}
	mp[s]=++tot[pos];
	dp[pos][mp[s]]=v;
	st[pos][mp[s]]=s;
}
void DP()
{
	dp[pos][1]=0,st[pos][1]=0,tot[pos]=1;ans=INT_MIN;
	_F(i,1,n)
	{
		_F(j,1,tot[pos]) st[pos][j]<<=2;
		_F(j,1,m)
		{
			pos^=1;
			tot[pos]=0;
			memset(mp,0,sizeof mp);
			_F(k,1,tot[lst])
			{
				int s=st[lst][k],v=dp[lst][k];
				int pl=(s>>bit[j-1])%4,pd=(s>>bit[j])%4;
				if(!pl&&!pd)
				{
					if(j!=m)
						ins(s+(1<<bit[j-1])+2*(1<<bit[j]),v+a[i][j]);//以下每一种insert操作中都+a[i][j]
					ins(s,v);
				}
				else if(pl&&!pd)
				{
					if(j!=m)
						ins(s-pl*(1<<bit[j-1])+pl*(1<<bit[j]),v+a[i][j]);
					ins(s,v+a[i][j]);
				}
				else if(!pl&&pd)
				{
					if(j!=m)
						ins(s,v+a[i][j]);
					ins(s-pd*(1<<bit[j])+pd*(1<<bit[j-1]),v+a[i][j]);
				}
				else if(pl==1&&pd==1)
				{
					int cnt=1;
					_F(l,j+1,m)
					{
						if((s>>bit[l])%4==1) cnt++;
						if((s>>bit[l])%4==2) cnt--;
						if(!cnt)
						{
							ins(s-(1<<bit[j])-(1<<bit[j-1])-(1<<bit[l]),v+a[i][j]);
							break;
						}
					}
				}
				else if(pl==2&&pd==2)
				{
					int cnt=1;
					F_(l,j-2,0)
					{
						if((s>>bit[l])%4==1) cnt--;
						if((s>>bit[l])%4==2) cnt++;
						if(!cnt)
						{
							ins(s+(1<<bit[l])-2*(1<<bit[j-1])-2*(1<<bit[j]),v+a[i][j]);
							break;
						}
					}
				}
				else if(pl==2&&pd==1)
				{
					ins(s-2*(1<<bit[j-1])-(1<<bit[j]),v+a[i][j]);
				}
				else if(pl==1&&pd==2)
				{
					if(s==(1<<bit[j-1])+2*(1<<bit[j])&&ans<v+a[i][j])
						ans=v+a[i][j];//满足条件随时统计答案
				}
			}
		}
	}
}
int main()
{
	scanf("%lld%lld",&n,&m);
	_F(i,1,n)
	{
		_F(j,1,m)
		{
			scanf("%lld",&a[i][j]);
		}
	}
	_F(i,1,16)
		bit[i]=(i<<1);
	DP();
	printf("%lld",ans);
	return 0;
}
```
{% endspoiler %}

### P3272 [SCOI2011] 地板——插头DP解决铺地板问题

给你一个有障碍的棋盘，让你用'L' 形的瓷砖去铺满这个棋盘，其中每一个瓷砖必须有且仅有一个拐角，问方案数。

~~这题是紫~~
看似与模板题不是很相似，但实际上也是只有分类讨论部分是不一样的。

先设状态：我们规定，1号插头为当前瓷砖没有拐弯的插头，2号插头为当前瓷砖已经拐弯的插头。
这样我们就有以下几种分类讨论：(老规矩，1号蓝色，2号红色)
1. 当前格是障碍，在没有插头时转移![00.png](https://s1.imagehub.cc/images/2024/06/18/382b63d0d2fe166507ae4ec87e28cd45.png)
2. 当前格没有插头，有三种选择，向右或下创建一个1号插头（也就是L起点）；还有同时向右和下创建两个2号插头（也就是L拐角）![1.png](https://s1.imagehub.cc/images/2024/06/18/4776dfdaca6ec9f6d7ac5ebbc619926b.png)![11.png](https://s1.imagehub.cc/images/2024/06/18/160ee07cecfb66326fc40167ced53f69.png)![111.png](https://s1.imagehub.cc/images/2024/06/18/56cd31fbee60caa36ba56220edd8eb89.png)
3. 当前格有一个1号下插头，创建一个向下的1号插头，或创建一个向右的2号插头。![2.png](https://s1.imagehub.cc/images/2024/06/18/b5293afea1105183833df66f142f540a.png)
4. 有1号右插头与上一个类似![3.png](https://s1.imagehub.cc/images/2024/06/18/db8dd95e91c68b4e91f4805e435f8088.png)
5. 当前格有一个2号下插头，这时可以选择创建一个2号下插头，或者在这里结束（删除）![4.png](https://s1.imagehub.cc/images/2024/06/18/68f35deccc0dfc2175806fcb28418908.png)![44.png](https://s1.imagehub.cc/images/2024/06/18/f4ef3254e4dd82202ae64dc2a7502042.png)
6. 当前格有2号右插头与上一个类似![5.png](https://s1.imagehub.cc/images/2024/06/18/d7c7222672184a5c9503e84441d0e346.png)![55.png](https://s1.imagehub.cc/images/2024/06/18/4f795b07ae4c13da817f10e820109bb0.png)
7. 当前格两个1号插头，将他们合并![6.png](https://s1.imagehub.cc/images/2024/06/18/a8365ccd10cf2e0bc26d647808328d4d.png)

注意，在第5、6、7种情况时，如果到达了最后一个可以进行转移的格子，需要统计答案。

另外，在本题中，你可能还需要调转矩阵的方向以保证时间复杂度。

{% spoiler  代码%}
``` cpp
#include <bits/stdc++.h>
#define _F(x,y,z) for(int x=y;x<=z;x++)
#define F_(x,z,y) for(int x=z;x>=y;x--)
#define TF(x,y,z) for(int x=head[y],z;x;x=nex[x])
#define lst pos^1

using namespace std;

typedef long long ll;
typedef double dou;
typedef const int ci;

ci N=12,p=20110520,has=499979;
ll n,m,mp[110][110],ex,ey,ans;
ll pos,st[2][500010],bit[20],dp[2][500010];
ll tot[2],head[500010],nex[500010];
void init()
{
	_F(i,1,19) bit[i]=i<<1;
}
void ins(ll s,ll v)
{
	int tmp=s%has;
	TF(i,tmp,y)
	{
		y=y;
		if(st[pos][i]==s)
		{
			dp[pos][i]=(dp[pos][i]+v)%p;
			return ;
		}
	}
	nex[++tot[pos]]=head[tmp];
	head[tmp]=tot[pos];
	st[pos][tot[pos]]=s;
	dp[pos][tot[pos]]=v;
}
void DP()
{
	init();
	st[pos][1]=0,dp[pos][1]=1,tot[pos]=1;
	_F(i,1,n)
	{
		_F(j,1,tot[pos]) st[pos][j]<<=2;
		_F(j,1,m)
		{
			pos^=1;
			memset(head,0,sizeof head);
			tot[pos]=0;
			_F(k,1,tot[lst])
			{
				ll s=st[lst][k],v=dp[lst][k];
				ll pl=(s>>bit[j-1])%4,pd=(s>>bit[j])%4;
				if(!mp[i][j])
				{
					if(!pl&&!pd)//注意转移条件！！！！
						ins(s,v);
				}
				else if(!pl&&!pd)
				{
					if(mp[i+1][j]&&mp[i][j+1])
						ins(s+2*(1<<bit[j-1])+2*(1<<bit[j]),v);
					if(mp[i+1][j])
						ins(s+(1<<bit[j-1]),v);
					if(mp[i][j+1])
						ins(s+(1<<bit[j]),v);
				}
				else if(!pl&&pd==1)
				{
					if(mp[i+1][j])
						ins(s+(1<<bit[j-1])-(1<<bit[j]),v);
					if(mp[i][j+1])
						ins(s+2*(1<<bit[j])-(1<<bit[j]),v);
				}
				else if(pl==1&&!pd)
				{
					if(mp[i+1][j])
						ins(s+2*(1<<bit[j-1])-(1<<bit[j-1]),v);
					if(mp[i][j+1])
						ins(s-(1<<bit[j-1])+(1<<bit[j]),v);
				}
				else if(!pl&&pd==2)
				{
					if(ex==i&&ey==j) ans=(ans+v)%p;//统计答案
					if(mp[i+1][j]) 
						ins(s-2*(1<<bit[j])+2*(1<<bit[j-1]),v);
					ins(s-2*(1<<bit[j]),v);
				}
				else if(pl==2&&!pd)
				{
					if(ex==i&&ey==j) ans=(ans+v)%p;//统计答案
					if(mp[i][j+1])
						ins(s-2*(1<<bit[j-1])+2*(1<<bit[j]),v);
					ins(s-2*(1<<bit[j-1]),v);
				}
				else if(pl==1&&pd==1)
				{
					if(ex==i&&ey==j) ans=(ans+v)%p;//统计答案
					ins(s-(1<<bit[j-1])-(1<<bit[j]),v);
				}
			}
		}
	}
}
int main()
{
	scanf("%lld%lld",&n,&m);
	_F(i,1,n)
	{
		char s[110];
		scanf("%s",s+1);
		_F(j,1,m)
		{
			if(s[j]=='_')
			{
				mp[i][j]=1;
				ex=i,ey=j;//找到最后一个可以进行DP的格子
			}
		}
	}
	if(n<m)//调转矩阵方向
	{
		swap(n,m),swap(ex,ey);
		_F(i,1,n)
		{
			_F(j,1,i)
			{
				swap(mp[i][j],mp[j][i]);
			}
		}
	}
	DP();
	printf("%lld",ans);
	return 0;
}

```
{% endspoiler %}

### [Code+#3] 白金元首与莫斯科

给你一个带障碍的棋盘，询问你对于每一个空地 $(i,j)$ 改成障碍后放置 $1\times 2$ 骨牌的方案数（不需要铺满）

这道题的状态和转移都很简单，甚至可以用二进制，有插头的地方就要响应即可，不过我们需要考虑如何降时间复杂度，因为我们发现，一个一个将点修改后再进行统计答案时间复杂度会爆炸。

我们发现，在前面的转移中，我们使用了滚动数组来优化空间，而当我们不进行滚动数组时，当前这个点上保存了转移到当前点的状态和方案数。于是我们可以考虑将DP再倒着进行一遍这样的话前半段的方案数和后半段的方案数之积就是答案（在合并时需要保证状态一致）

本体也不需要使用hash，但读者还需细细品味如何进行倒着DP

{% spoiler  代码%}
``` cpp
#include <bits/stdc++.h>
#define _F(x,y,z) for(int x=y;x<=z;x++)
#define F_(x,z,y) for(int x=z;x>=y;x--)
#define TF(x,y,z) for(int x=head[y],z;x;x=nex[x])

using namespace std;

typedef long long ll;
typedef double dou;
typedef const int ci;

ci p=1e9+7;
int n,m,f[18][18][270000],g[18][18][270000],ans[20][20],mp[20][20];

int main()
{
	scanf("%d%d",&n,&m);
	_F(i,1,n)
		_F(j,1,m)
			scanf("%d",&mp[i][j]),mp[i][j]^=1;
	int mx=(1<<(m+1))-1;
	f[0][m][0]=1;//正着进行DP
	_F(i,1,n)
	{
		_F(j,0,mx) f[i][0][j<<1]=(f[i][0][j<<1]+f[i-1][m][j]);
		_F(j,1,m)
		{
			_F(k,0,mx)
			{
				int v=f[i][j-1][k];
				if(!v) continue;
				int pl=(k>>(j-1))&1,pd=(k>>j)&1;
				if(!mp[i][j])//为障碍
				{
					if(!pl&&!pd)
						f[i][j][k]=(f[i][j][k]+v)%p;
				}
				else if(!pl&&!pd)//无插头
				{
					f[i][j][k]=(f[i][j][k]+v)%p;
					if(mp[i+1][j]) f[i][j][k^(1<<(j-1))]=(f[i][j][k^(1<<(j-1))]+v)%p;
					if(mp[i][j+1]) f[i][j][k^(1<<j)]=(f[i][j][k^(1<<j)]+v)%p;
				}
				else if(!pl&&pd)//下插头
				{
					f[i][j][k^(1<<j)]=(f[i][j][k^(1<<j)]+v)%p;
				}
				else if(pl&&!pd)//右插头
				{
					f[i][j][k^(1<<(j-1))]=(f[i][j][k^(1<<(j-1))]+v)%p;
				}
			}
		}
	}
	g[n+1][1][0]=1;//反着DP
	F_(i,n,1)
	{
		_F(j,0,mx) g[i][m+1][j>>1]=(g[i][m+1][j>>1]+g[i+1][1][j]);
		F_(j,m,1)
		{
			_F(k,0,mx)
			{
				int v=g[i][j+1][k];
				if(!v) continue;
				int pl=(k>>(j-1))&1,pd=(k>>j)&1;//状态表示不变
				if(!mp[i][j])
				{
					if(!pl&&!pd)
						g[i][j][k]=(g[i][j][k]+v)%p;
				}
				else if(!pl&&!pd)
				{
					g[i][j][k]=(g[i][j][k]+v)%p;
					if(mp[i][j-1]) g[i][j][k^(1<<(j-1))]=(g[i][j][k^(1<<(j-1))]+v)%p;//还是要注意这里与上面的区别
					if(mp[i-1][j]) g[i][j][k^(1<<j)]=(g[i][j][k^(1<<j)]+v)%p;
				}
				else if(!pl&&pd)
				{
					g[i][j][k^(1<<j)]=(g[i][j][k^(1<<j)]+v)%p;
				}
				else if(pl&&!pd)
				{
					g[i][j][k^(1<<(j-1))]=(g[i][j][k^(1<<(j-1))]+v)%p;
				}
			}
		}
	}
	_F(i,1,n)
	{
		_F(j,1,m)
		{
			if(!mp[i][j]) continue;
			int st=mx^(1<<(j-1))^(1<<j);//当前格不能拥有插头
			for(int k=st;k;k=(k-1)&st)//枚举子集，自行理解
				ans[i][j]=(ans[i][j]+1ll*f[i][j-1][k]*g[i][j+1][k]%p)%p;
			ans[i][j]=(ans[i][j]+1ll*f[i][j-1][0]*g[i][j+1][0]%p)%p;
		}
	}
	_F(i,1,n)
	{
		_F(j,1,m)
		{
			printf("%d ",ans[i][j]);
		}
		puts("");
	}
	return 0;
}
```
{% endspoiler %}