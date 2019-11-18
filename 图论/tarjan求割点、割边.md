## tarjan求割点、割边原理


* 由点a搜索点b回溯时，若low[b]>=dfn[a],则a为割点。
节点b的子孙和节点a形成一个块。因为这说明b的子孙不能够通过其他边到达a的祖先，这样去掉a之后，图必然分裂为两个子图。
这样我们处理点a时，首先递归a的子节点b，然后从b回溯至a后，如果发现上述不等式成立，则找到了一个割点a，并且a和b的子树构成一个块。
**注意：如果a是根节点要特判一下。即如果它的子树大于1棵，辣么它也是个割点。**

* 若low[b]>dfn[a],则(a,b)为割边。
但是实际处理时可能有重边，因此需要。
我们记录每条边的标号与每个点的父亲到它的边的标号，
如果边(a,b)是b的父亲边，就不能用dfn[a]更新low[b]。这样如果遍历完b的所有子节点后，发现low[b]仍然=dfn[b]，说明u的父亲边(a,b)为割边。




## 求割点例题
[[P3388 【模板】割点（割顶）](https://www.luogu.org/problem/P3388)]

Time limit：1.00s
Memory limit：125.00MB

### 题意

给出一个n个点，m条边的无向图，求图的割点。
要求输出割点数量，下一行从小到大输出割点编号，用空格隔开

### 代码

Time：20ms
Memory：3.14MB 


```cpp
#include<bits/stdc++.h>
using namespace std;
const int maxn = 2e4 + 10;
const int maxm = 2e5 + 10;
int a[maxn], dfn[maxn], low[maxn];
int head[maxn], ans[maxn], cnt, num;
int rt[maxn], numb;//标记根节点,记录答案数量
struct EDGE
{
	int to, next;//下个点，上条边
}e[maxm];

void init()
{
	cnt = num = numb = 0;
	memset(a, 0, sizeof(a));
	memset(e, 0, sizeof(e));
	memset(head, 0, sizeof(head));
	memset(dfn, 0, sizeof(dfn));
	memset(low, 0, sizeof(low));
	memset(ans, 0, sizeof(ans));
	memset(rt, 0, sizeof(rt));
}

void add(int u,int v)//前向星
{
	e[++cnt].to = v;
	e[cnt].next = head[u];
	head[u] = cnt;
}

void tarjan(int x)
{
	int in = 0;
	dfn[x] = low[x] = ++num;
	for (int i = head[x]; i; i = e[i].next)
	{
		int v = e[i].to;
		if (!dfn[v])//v还没被搜索过
		{
			tarjan(v);
			low[x] = min(low[x], low[v]);
			if (dfn[x] <= low[v] && rt[x] == 0)//x的dfn<=v的low，并且x不是根节点
				ans[x] = 1;
			if (rt[x])
				in++;
		}
		low[x] = min(low[x], dfn[v]);
	}
	if (rt[x] && in >= 2)//入度>=2的根节点也是割点
		ans[x] = 1;
}

int n, m, x, y;
int main()
{
	scanf("%d%d", &n, &m);
	init();
	while (m--)
	{
		scanf("%d%d", &x, &y);
		add(x, y);
		add(y, x);
	}
	for (int i = 1; i <= n; i++)
	{
		if (!dfn[i])
		{
			rt[i] = 1;//标记根节点
			tarjan(i);
		}
	}
	for (int i = 1; i <= n; i++)
	{
		if (ans[i])
			numb++;
	}
	printf("%d\n", numb);
	for (int i = 1; i <= n; i++)
	{
		if (ans[i])
			printf("%d ", i);
	}
	return 0;
}

```

## 求割边模板

```cpp

const int N=110;
struct data
{
    int to,next;
} tu[N*N];
int head[N],low[N],dfn[N];
int ip;
int step;
void init()
{
    ip=0;
    step=1;///遍历的步数
    memset(head,-1,sizeof(head));
    memset(dfn, 0, sizeof(dfn));
}
void add(int u,int v)
{
    tu[ip].to=v,tu[ip].next=head[u],head[u]=ip++;
}
void tarjan(int u,int pre)
{
    dfn[u] = low[u] = step++;
    for(int i = head[u]; i!=-1 ; i=tu[i].next)
    {
        int to = tu[i].to;
        if(!dfn[to])
        {
            tarjan(to,u);
            low[u]=min(low[u],low[to]);
            if(low[to]>dfn[u])
                printf("%d--%d\n",u,to);//表示u到to是一条割边
        }
        else if(to!=pre)
            low[u]=min(low[u],dfn[to]);
    }
}


```

## 求割边（桥）例题

[[poj 3177 Redundant Paths](https://www.cnblogs.com/captain1/p/9758119.html)]

Time Limit: 1000MS
Memory Limit: 65536K


### 题意
要求在原图上加上数量最少的边，使得整张图成为一个边双联通分量。


### 思路
具体的做法是，先在图中求出所有的桥，之后把边双联通分量缩成点，这样的话原图就变成了一棵树。之后，我们就在叶子之间加边即可。如何加最少的边呢？好像第一眼看上去，随便在两个叶子中间加一条边就能减少两个叶子，但事实上不是这样的，如果这两个叶子中间的路径数小于等于1条的话，将新形成的边双联通分量缩点之后有可能出现新的叶子。就像这张图一样，如果连接红色的边，那么新的图会多出一个叶子。

![avatar](https://img2018.cnblogs.com/blog/1296693/201810/1296693-20181009002400265-968605282.png)

这个是因为路径上的边数只有一条，如果我们选择两点路径上边数大于1的两个叶子合并，那么一定会形成一个新的边双联通分量，而且不会有新的叶子。
因为这样的话那个新的边双肯定是至少有两个度的，那他就不会成为叶子，而如果只有一条的话，它的度就是1，那么就形成新的叶子了。
我们要这样去合并：

![avatar](https://img2018.cnblogs.com/blog/1296693/201810/1296693-20181009002757792-443173338.png)

### 结论
需要加的边数=（叶子个数+1) >> 1.

直接求桥，之后缩点，求出每个点最后的度然后计算一下就行。
然后这题因为有重边就很难受……一开始我是直接判断如果是父亲就不管，但是这样不行……因为有重边的就不是桥了，他是需要父亲去更新的，所以后来判断一下，只有树边的那条反向边我们给他特判掉，其他的都能正常更新。


###代码
C++ Time：32MS，Memory：520K
G++ TIme：16MS，Memory：1044K

```cpp
#include<bits/stdc++.h>
#define rep(i,a,n) for(int i = a;i <= n;i++)
using namespace std;
typedef long long ll;
const int M = 100005;
const int N = 10000005;

int read()
{
	int ans = 0, op = 1;
	char ch = getchar();
	while (ch < '0' || ch > '9')
	{
		if (ch == '-') op = -1;
		ch = getchar();
	}
	while (ch >= '0' && ch <= '9')
	{
		ans *= 10;
		ans += ch - '0';
		ch = getchar();
	}
	return ans * op;
}

struct edge
{
	int next, to, from;
}e[M];

int f, r, x, y, ecnt = -1;
int head[M], dfn[M], low[M], scc[M];
int sta[M], top, cnt, rdeg[M], ans, idx;
bool vis[M], pd[M];

void add(int x, int y)
{
	e[++ecnt].to = y;
	e[ecnt].from = x;
	e[ecnt].next = head[x];
	head[x] = ecnt;
}

void tarjan(int x, int g)
{
	bool flag = 0;
	vis[x] = 1, sta[++top] = x;
	dfn[x] = low[x] = ++idx;
	for (int i = head[x]; ~i; i = e[i].next)
	{
		if (e[i].to == g && !flag)
		{
			flag = 1;
			continue;
		}
		if (!dfn[e[i].to]) tarjan(e[i].to, x), low[x] = min(low[x], low[e[i].to]);
		else if (vis[e[i].to])low[x] = min(low[x], dfn[e[i].to]);
	}
	if (low[x] == dfn[x])
	{
		int p;
		cnt++;
		while ((p = sta[top--]))
		{
			scc[p] = cnt, vis[p] = 0;
			if (p == x) break;
		}
	}
}

int main()
{
	memset(head, -1, sizeof(head));
	f = read(), r = read();
	rep(i, 1, r) 
	{
		x = read(), y = read();
		add(x, y), add(y, x);
	}
	rep(i, 1, f) 
		if (!dfn[i]) 
			tarjan(i, i);
	rep(i, 1, f)
	{
		for (int j = head[i]; ~j; j = e[j].next)
		{
			int r1 = scc[e[j].to], r2 = scc[i];
			if (r1 != r2)
			rdeg[r1]++, rdeg[r2]++;
		}
	}
	rep(i, 1, cnt) 
		if (rdeg[i] == 2) 
			ans++;
	printf("%d\n", (ans + 1) >> 1);
	return 0;
}

```