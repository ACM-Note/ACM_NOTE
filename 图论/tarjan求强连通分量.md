## tarjan求强连通分量

### 模板

```cpp

#include<bits/stdc++.h>
using namespace std;
const int maxn = 1e5 + 10;
const int maxn = 1e5 + 10;
stack<int> sta;
int a[maxn], dfn[maxn], low[maxn];
int head[maxn], vis[maxn], cnt, num;
struct EDGE
{
	int to, next;//下个点，上条边
}e[maxm];

void init()
{
	cnt = num = 0;
	memset(a, 0, sizeof(a));
	memset(head, 0, sizeof(head));
	memset(vis, 0, sizeof(vis));
	memset(dfn, 0, sizeof(dfn));
	memset(low, 0, sizeof(low));
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
		int cur;
		do
		{
			cur = sta.top();
			sta.pop();
			vis[cur] = 0;
			printf("%d ", cur);
		} while (cur != x);
		printf("\n");
	}
}

int n, m, x, y;
int main()
{
	scanf("%d%d", &n, &m);
	init();
	for (int i = 1; i <= n; i++)
	{
		scanf("%d", &a[i]);
	}
	while (m--)
	{
		scanf("%d%d", &x, &y);
		add(x, y);
	}
	for (int i = 1; i <= n; i++)
	{
		if (!dfn[i])
			tarjan(i);
		printf("low[%d]=%d\n", i, low[i]);
	}
	return 0;
}


```