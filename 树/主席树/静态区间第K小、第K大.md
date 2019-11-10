## 区间第k小、第k大
### 题一：K-th Number
[[POJ 2104](http://poj.org/problem?id=2104)]
Time Limit: 20000MS
Memory Limit: 65536K
### 题二：【模板】可持久化线段树 1（主席树）
[[P3834](https://www.luogu.org/problem/P3834)]
Time Limit:1.00s ~ 1.20s
Memory Limit:125.00MB ~ 250.00MB

#### 题意
输入n和m，代表n个数，m次询问
输入n个数
下面m次询问输入l r k，查询第k小的数

#### 思路
主席树模板

#### 代码
Time：1219MS
Memory：22848K

```cpp
#include<stdio.h>
#include<cstring>
#include<iostream>
#include<algorithm>
using namespace std;
const int maxn = 2e5 + 10;
int rt[maxn];//存储每棵线段树的根节点编号
int cnt, p, q;//节点个数，离散化后的序列长度 
int a[maxn], b[maxn];//存数据 
struct TREE
{
	int l, r, sum;
}tree[maxn << 5];

void init()
{
	cnt = 0;
	tree[cnt].l = tree[cnt].r = tree[cnt].sum = 0;
	rt[cnt] = 0;
	memset(a, 0, sizeof(a));
	memset(b, 0, sizeof(b));
}

//更新新的树 
void update(int& x, int y, int l, int r, int pos)
{
	x = ++cnt;//记录节点坐标 
	tree[x] = tree[y];//复制节点 
	tree[x].sum++;//区间内p的数量+1 
	if (l == r)
		return;
	int mid = (l + r) >> 1;
	if (pos <= mid)//更新左子树 
		update(tree[x].l, tree[y].l, l, mid, pos);
	else//更新右子树 
		update(tree[x].r, tree[y].r, mid + 1, r, pos);
}

//查询区间第k小
int querymin(int x, int y, int l, int r, int k)
{
	if (l == r)
		return l;
	int mid = (l + r) >> 1;
	int sum = tree[tree[y].l].sum - tree[tree[x].l].sum;//1~y区间的左sum减去1~x-1区间的左sum，得到的是x到y的线段树左子树的sum
	if (k <= sum)//进入左子树
		return querymin(tree[x].l, tree[y].l, l, mid, k);
	else//进入右子树 
		return querymin(tree[x].r, tree[y].r, mid + 1, r, k - sum);

}

//查询区间第k大
int querymax(int x, int y, int l, int r, int k)
{
	if (l == r)
		return l;
	int mid = (l + r) >> 1;
	int sum = tree[tree[y].r].sum - tree[tree[x].r].sum;//1~y区间的右sum减去1~x-1区间的右sum，得到的是x到y的线段树右子树的sum
	if (k <= sum)//进入右子树
		return querymax(tree[x].r, tree[y].r, mid + 1, r, k);
	else//进入左子树 
		return querymax(tree[x].r, tree[y].r, l, mid, k - sum);

}

int n, m, l, r, k;
int main()
{
	while (~scanf("%d%d", &n, &m))
	{
		init();
		for (int i = 1; i <= n; i++)
		{
			scanf("%d", &a[i]);
			b[i] = a[i];
		}
		sort(b + 1, b + n + 1);
		q = unique(b + 1, b + n + 1) - b - 1;
		for (int i = 1; i <= n; i++)
		{
			int p = lower_bound(b + 1, b + q + 1, a[i]) - b;//a[i]在b中的位置p 
			update(rt[i], rt[i - 1], 1, q, p);//更新新的节点，并建树 
		}
		while (m--)
		{
			scanf("%d%d%d", &l, &r, &k);
			printf("%d\n", b[querymin(rt[l - 1], rt[r], 1, q, k)]);//查询l到r区间的第k小个数
			printf("%d\n", b[querymax(rt[l - 1], rt[r], 1, q, k)]);//查询l到r区间的第k大个数
		}
	}
	return 0;
}
```