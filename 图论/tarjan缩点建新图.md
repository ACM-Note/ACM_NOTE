## tarjan缩点建新图

[[Popular Cows POJ - 2186](https://vjudge.net/contest/354153#problem/B) ]

### 题意
N 个点(1 <= N <= 10,000) ,M 条有向边(1 <= M <= 50,000) ，下面M行输入A，B表示点A认为B牛逼。注意若A认为B牛逼，B认为C牛逼，那么A也认为C牛逼。求有几个人，其他人都认为他牛逼。

### 思路
* 给的有向图可能不连通/有环，因此需要tarjan缩点，每个强连通分量中的点互相都认为是牛逼的。
* 记录每个强连通分量里点的个数，建立一个新图，这个图无环但可能不连通。
* 若该图中只有一个点的出度为0，说明这一个点就是所求的点，返回它缩点前包含的点个数；若有多个点的度数为0，那么就没有符合条件的答案。

### 代码注意点
1. 由于原图不一定连通，所以tarjan时应该先缩第一个点，然后遍历其余的点，若已经被缩点，就跳过，否则就继续缩点。
2. 在tarjan弹栈的时候，记录每个强连通分量里的点的个数。
3. 建新图的时候最好用全新的其他变量。
4. 缩点后找答案不能用并查集也不能用dfs，并查集只合并集合不看有向图的方向，dfs会爆栈。


### 代码
```cpp
const int maxn = 10010;
const int maxm = 50010;
int n, m, u, v, ans;
stack<int> sta;
int dfn[maxn], low[maxn];
int head[maxn], head2[maxn], vis[maxn], vis2[maxn], cnt, tot, num, sum;
int cnt2, flag[maxn], answ[maxn];//强连通分量数量,每个点所在强连通分量编号,强连通分量所包含的点的个数
int Chu[maxn];//出度
struct EDGE
{
	int u, to, next;//上个点，下个点，上条边
}e[maxm], e2[maxm];
void init()
{
	cnt = cnt2 = tot = num = sum = 0;
	memset(e, 0, sizeof(e));
	memset(head, 0, sizeof(head));
	memset(head2, 0, sizeof(head2));
	memset(vis, 0, sizeof(vis));
	memset(vis2, 0, sizeof(vis2));
	memset(dfn, 0, sizeof(dfn));
	memset(low, 0, sizeof(low));
	memset(flag, 0, sizeof(flag));
	memset(answ, 0, sizeof(answ));
	memset(Chu, 0, sizeof(Chu));
}
void add(int u, int v)//前向星
{
	e[++cnt].to = v;
	e[cnt].u = u;
	e[cnt].next = head[u];
	head[u] = cnt;
}

void add2(int u, int v)//新图前向星
{
	e2[++tot].to = v;
	e2[tot].u = u;
	e2[tot].next = head2[u];
	head2[u] = tot;
}

void tarjan(int x)//板子
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
		int cursum = 0;
		int cur;
		do
		{
			cur = sta.top();
			cursum++;
			sta.pop();
			vis[cur] = 0;
			flag[cur] = cnt2;//记录cur所在强连通分量编号
		} while (cur != x);
		answ[cnt2] = cursum;//记录该强连通分量里有几个点
	}
}

int tmpp,edgenum;
int main()
{
	scanf("%d%d", &n, &m);
	init();
	for(int i=1;i<=m;i++)
	{
		scanf("%d%d", &u, &v);
		add(u, v);
	}
	tarjan(1);//先锁第一个点
	for (int i = 2; i <= n; i++)
	{
		if(flag[i]==0)//i点所在强连通分量还未赋值，即这个点没有被缩过，则缩点
			tarjan(i);
	}
	if (cnt2 == 1)//缩成了一个点，说明强连通，所有点都符合条件
		printf("%d\n", n);
	else
	{
		for (int i = 1; i <= m; i++)//建新图
		{
			int x = e[i].u;
			int y = e[i].to;
			if (flag[x] != flag[y]&&vis2[flag[x]]==0)
				add2(flag[x], flag[y]), Chu[flag[x]]++, edgenum++,vis2[flag[x]]=flag[y];
		}
		int fl = 0, du = 0;
		for (int i = 1; i <= cnt2; i++)//找出度为0的点
		{
			if (Chu[i] == 0)
			{
				fl++;
				du = i;
			}
		}
		if (fl == 1)
			printf("%d\n", answ[du]);
		else
			printf("0\n");
	}
	return 0;
}

```
