## 求区间l到r中≤k的数的个数

[[牛客275E 换个角度思考](https://ac.nowcoder.com/acm/contest/275/E)]

时间限制：C/C++ 1秒
空间限制：C/C++ 262144K

#### 题意
第一行输入n和m，表示n个数，m次询问（1 ≤ n ≤ $10^{5}$）
第二行n个整数表示序列a的元素，序列下标从1开始标号，保证1 ≤ ai ≤ $10^{5}$
之后有m行，每行三个整数(l,r,k)，保证1 ≤ l ≤ r ≤ n，且1 ≤ k ≤ $10^{5}$
求l到r之间≤k的数的个数

#### 思路
求树r中小于等于k的数量-树(l-1)中满足小于等于k的数量
因为需要和k进行大小比较，所以不能进行离散化，所以一定要注意数据范围

#### 代码
Time：244ms
Memory：22976kb
```cpp
#include<stdio.h>
#include<cstring>
#include<iostream>
#include<algorithm>
using namespace std;
const int maxn = 2e5 + 10;
int rt[maxn],cnt,a[maxn];//存储每棵线段树的根节点编号,节点个数,存数据
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

//查询l到r区间小于等于k的数的个数
int querynum(int x, int l, int r, int k)
{
	if (l == r)
		return tree[x].sum;
	int mid = (l + r) >> 1;
	if (k <= mid)
		return querynum(tree[x].l, l, mid, k);
	else
		return tree[tree[x].l].sum + querynum(tree[x].r, mid + 1, r, k);
}

int n, m, l, r, k;
int main()
{
	while (~scanf("%d%d", &n, &m))
	{
		init();
		for (int i = 1; i <= n; i++)
			scanf("%d", &a[i]);
		for (int i = 1; i <= n; i++)
		{
			update(rt[i], rt[i - 1], 1, n, a[i]);//更新新的节点，并建树 
		}
		while (m--)
		{
			scanf("%d%d%d", &l, &r, &k);
			printf("%d\n", querynum(rt[r], 1, n, k) - querynum(rt[l - 1], 1, n, k));
		}
	}
	return 0;
}
```