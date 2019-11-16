## tarjan缩点
[[P2746 [USACO5.3]校园网Network of Schools](https://www.luogu.org/problem/P2746)]

Time limit：1.00s
Memory limit：125.00MB

### 题意
给出一个n代表一共n个点
下面n行每行输入一些数，代表第i个点能够传递单向信息的点，以0结束输入
求：
1. 至少给几个点传递信息，使得所有点都能接收到信息？
2. 至少需要再连几条边，使得给任意一个点传递信息就能让所有点都接收到信息？


### 思路
1. 入度为0的有几个点，就需要传递几个点，因为入度为0的点是树根
2. 入度为0的点和出度为0的点的个数取最大值，因为连接最少的边就需要连成环，为使所有度数不为0，就最大值

(注意：这里说的点，指的是缩点后的点，不缩点计算出的度数不对，如两个环的入度出度没有0，但是需要至少连2条边才能全部联通)

3. 特判缩点后只有一个点的情况，入度出度均为0，但不需要再连边
4. 题目没说没有重边，所以边数要开大点


### 代码

Time：4ms
Memory：1.66MB

```cpp

#include<bits/stdc++.h>
using namespace std;
const int maxn = 110;
stack<int> sta;
int a[maxn], dfn[maxn], low[maxn];
int head[maxn], vis[maxn], cnt, num;
int cnt2,flag[maxn];//强连通分量数量,每个点所在强连通分量编号
int Ru[maxn], Chu[maxn];//入度出度
struct EDGE
{
	int to, next;//下个点，上条边
}e[maxn << 10];

void init()
{
	cnt = cnt2 = num = 0;
	memset(a, 0, sizeof(a));
	memset(e, 0, sizeof(e));
	memset(head, 0, sizeof(head));
	memset(vis, 0, sizeof(vis));
	memset(dfn, 0, sizeof(dfn));
	memset(low, 0, sizeof(low));
	memset(flag, 0, sizeof(flag));
	memset(Ru, 0, sizeof(Ru));
	memset(Chu, 0, sizeof(Chu));
}

void add(int u,int v)//前向星
{
	e[++cnt].to = v;
	e[cnt].next = head[u];
	head[u] = cnt;
}

void tarjan(int x)
{
	dfn[x] = low[x] = ++num;
	sta.push(x);
	vis[x] = 1;
	for (int i = head[x]; i; i = e[i].next)
	{
		int v = e[i].to;
		if (!dfn[v])//v还没被搜索过
		{
			tarjan(v);
			low[x] = min(low[x], low[v]);
		}
		else if (vis[v])//v在栈中
		{
			low[x] = min(low[x], dfn[v]);
			//这里用dfn[v]而不是low[v]的原因是：
			//v可能已经在另一个强连通分量中尚未出栈
			//当前点x不一定能到达low[v]，但一定能到达dfn[v]
		}
	}
	if (dfn[x] == low[x])//找完一个强连通分量，出栈
	{
		cnt2++;//强连通分量个数++
		int cur;
		do
		{
			cur = sta.top();
			sta.pop();
			vis[cur] = 0;
			flag[cur] = cnt2;//记录cur所在强连通分量编号
		} while (cur != x);
	}
}

int n, j;
int main()
{
	scanf("%d", &n);
	init();
	for(int i=1;i<=n;i++)
	{
		while (~scanf("%d", &j) && j)
		{
			add(i, j);
		}
	}
	for (int i = 1; i <= n; i++)
	{
		if (!dfn[i])
			tarjan(i);
	}
	for (int i = 1; i <= n; i++)
	{
		for (int k = head[i]; k; k = e[k].next)
		{
			int v = e[k].to;
			if (flag[i] != flag[v])//前后两点所在不同强连通分量，即缩点后的不同两点
			{
				Chu[flag[i]]++;//i所在强连通分量的编号个数++，即缩点后的点个数++
				Ru[flag[v]]++;//v所在强连通分量的编号个数++，即缩点后的点个数++
			}
		}
	}
	int ans1 = 0, ans2 = 0;
	for (int i = 1; i <= cnt2; i++)
	{
		if (!Ru[i])
			ans1++;//入度为0的个数
		if (!Chu[i])
			ans2++;//出度为0的个数
	}
	if (cnt2 == 1)//缩点后只有一个点，虽然入度出度均为0但不需要连边
	{
		printf("1\n");
		printf("0\n");
	}
	else
	{
		printf("%d\n", ans1);
		printf("%d\n", max(ans1, ans2));
	}
	return 0;
}


```