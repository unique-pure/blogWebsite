---
title: Prim算法详解
date: 2020-7-29 11:34:35
top: true
tags: [Prim,最小生成树,C++]
categories: 算法
mathjax: true
---
# Prim算法

## 1 最小生成树（Minimum Spanning Tree,MST）

在一给定的无向图$G = (V, E)$ 中，$(u, v)$代表连接顶点$u$ 与顶点 $v$ 的边，而 $w(u, v)$ 代表此边的权重，**若存在 $T$ 为 $E$ 的子集且为无循环图，使得 $w(T)$ 最小，则此 $T$ 为 $G$ 的最小生成树**，因为$T$是由图$G$产生的。

![](https://img-blog.csdnimg.cn/2020081109252713.png)


最小生成树其实是最小权重生成树的简称。我们称求取该生成树的问题成为最小生成树问题。==一个连通图可能有多个生成树。当图中的边具有权值时，总会有一个生成树的边的权值之和小于或者等于其它生成树的边的权值之和==。广义上而言，对于非连通无向图来说，它的每一连通分量同样有最小生成树，它们的并被称为**最小生成森林**。以有线电视电缆的架设为例，若只能沿着街道布线，则以街道为边，而路口为顶点，其中必然有一最小生成树能使布线成本最低。

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQud2lraW1lZGlhLm9yZy93aWtpcGVkaWEvY29tbW9ucy90aHVtYi9kL2QyL01pbmltdW1fc3Bhbm5pbmdfdHJlZS5zdmcvMzAwcHgtTWluaW11bV9zcGFubmluZ190cmVlLnN2Zy5wbmc?x-oss-process=image/format,png)

如图，这个是一个平面图，图中黑色线描述的就是最小生成树，它的权值之和小于其他的生成树。

那么，我们如何来求最小生成树呢，由最小生成树的定义我们可以知道构建最小生成树是可以利用贪心算法去实现的，我们接下来介绍的两种算法也都是利用贪心算法去求得$MST$的。**因为贪心算法的策略就是在每一步尽可能多的选择中选择最优的，在当前看是最好的选择**，这种策略虽然一般不能在全局中寻找到最优解，但是对于最小生成树问题来说，它的确可以找到一颗权重最小的树。

## 2 Prim算法

### 2.1 简介

普里姆算法（Prim's algorithm），图论中的一种算法，可在加权连通图里搜索最小生成树。意即由此算法搜索到的边子集所构成的树中，不但包括了连通图里的所有顶点，且其所有边的权值之和亦为最小。该算法于1930年由捷克数学家沃伊捷赫·亚尔尼克发现；并在1957年由美国计算机科学家罗伯特·普里姆独立发现；1959年，艾兹格·迪科斯彻再次发现了该算法。因此，在某些场合，普里姆算法又被称为DJP算法、亚尔尼克算法或普里姆－亚尔尼克算法。（来源于维基百科）

### 2.2 具体步骤

Prim算法是利用贪心算法实现的，我们确定根节点，从这个结点出发。普里姆算法按照以下步骤==逐步扩大树中所含顶点的数目，直到遍及连通图的所有顶点==。下面描述我们==假设$N=(V,E)$是连通网，$TE$是$N$上最小生成树中边的集合==。

​	①  $U=\{u_0\}(u_0∈V) ,TE= \{\}$。

​    ②  在所有$u∈U,v∈(V-U)$的边$(u,v)∈E$找到一条权值最小的边$(u_0,v_0)$并入集合$TE$，同时$v_0$并入集合$U$。

​	③  重复②步骤，知道$U=V$为止。

此时$TE$中必有$n-1$条边，则$T=(V,TE)$即为我们求得的$N$的最小生成树。

### 2.3 算法示例图
![](https://img-blog.csdnimg.cn/20210322125136390.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h6ZjA3MDE=,size_16,color_FFFFFF,t_70#pic_center)


### 2.4 算法实现

我们如果对Dijkstra算法很熟悉的话，Prim算法也很好实现了，它们都是利用了一样的思路，但却不相同。==我们用利用$lowcost$数组来表示到集合中最近的距离，用$closest$数组来表示最小生成树的边。怎么来表示呢？我们用顶点来形容边，也就是说我们要求的就是$closet$数组。其中$closest[i]$表示的值就是与$i$顶点相邻边的顶点序号==。具体看代码（附带打印最小生成树代码）。

```cpp
#include<bits/stdc++.h>	//POJ不支持

#define rep(i,a,n) for (int i=a;i<=n;i++)//i为循环变量，a为初始值，n为界限值，递增
#define per(i,a,n) for (int i=a;i>=n;i--)//i为循环变量， a为初始值，n为界限值，递减。
#define pb push_back
#define IOS ios::sync_with_stdio(false);cin.tie(0); cout.tie(0)
#define fi first
#define se second
#define mp make_pair

using namespace std;

const int inf = 0x3f3f3f3f;//无穷大
const int maxn = 1e3;//最大值。
typedef long long ll;
typedef long double ld;
typedef pair<ll, ll>  pll;
typedef pair<int, int> pii;
//*******************************分割线，以上为自定义代码模板***************************************//

int n,m;//图的大小和边数。
int graph[maxn][maxn];//图
int lowcost[maxn],closest[maxn];//lowcost[i]表示i到距离集合最近的距离，closest[i]表示i与之相连边的顶点序号。
int sum;//计算最小生成树的权值总和。
void Prim(int s){
	//初始化操作，获取基本信息。
	rep(i,1,n){
		if(i==s)
			lowcost[i]=0;
		else
			lowcost[i]=graph[s][i];
		closest[i]=s;
	}
	int minn,pos;//距离集合最近的边，pos代表该点的终边下标。
	sum=0;
	rep(i,1,n){
		minn=inf;
		rep(j,1,n){
			//找出距离点集合最近的边。
			if(lowcost[j]!=0&&lowcost[j]<minn){
				minn=lowcost[j];
				pos=j;
			}
		}
		if(minn==inf)break;//说明没有找到。
		sum+=minn;//计算最小生成树的权值之和。
		lowcost[pos]=0;//加入点集合。
		rep(j,1,n){
			//由于点集合中加入了新的点，我们要去更新。
			if(lowcost[j]!=0&&graph[pos][j]<lowcost[j]){
				lowcost[j]=graph[pos][j];
				closest[j]=pos;//改变与顶点j相连的顶点序号。
			}
		}
	}
	cout<<sum<<endl;//closest数组就是我们要的最小生成树。它代表的就是边。
}
void print(int s){
	//打印最小生成树。
	int temp;
	rep(i,1,n){
		//等于s自然不算，故除去这个为n-1条边。
		if(i!=s){
			temp=closest[i];
			cout<<temp<<"->"<<i<<"边权值为："<<graph[temp][i]<<endl;
		}
	}
}
int main(){
	//freopen("in.txt", "r", stdin);//提交的时候要注释掉
	IOS;
	while(cin>>n>>m){
		memset(graph,inf,sizeof(graph));//初始化。
		int u,v,w;//临时变量。
		rep(i,1,m){
			cin>>u>>v>>w;
			//视情况而论，我这里以无向图为例。
			graph[u][v]=graph[v][u]=w;
		}
		//任取根结点，我这里默认取1.
		Prim(1);
		print(1);//打印最小生成树。
	}
	return 0;
}



```
### 2.5 算法分析
对于此算法，我们图中有$n$个顶点，则第一个进行初始化的循环语句的频度为$n$,第二个循环语句的频度为$n$，==但其中第二个循环中有两个内循环：第一个是在$lowcost$中求最小值，其频度为$n$，第二个是重新选择具有最小权值的边，频度为$n$==，由此我们可知Prim算法的时间复杂度为$O(n^2)$，与图中的边数无关，故十分适合于稠密图。
### 2.6 测试
我们用示例来测试：

```cpp
7 11
1 2 7
1 4 5
2 4 9
2 3 8
2 5 7
4 5 15
4 6 6
6 7 11
5 6 8
5 7 9
3 5 5
```
测试结果如图：

![](https://img-blog.csdnimg.cn/20200811095845614.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2h6ZjA3MDE=,size_16,color_FFFFFF,t_70)
