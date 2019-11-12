## splay树详解及板子

参考博客：https://www.cnblogs.com/gzh-red/p/11011557.html

这是一棵特殊的BST树,或者说平衡树基本都是改变树结构样式,但是却不改变最后得出的排列序列.
![avatar](http://img.blog.csdn.net/20170825215240167?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzA5NzQzNjk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

平衡树的精髓,就是改变树的形态结构,但是不改变最后的中序遍历,也就是答案数组.

------------

现在我们的现在目标只有一个：x节点往上面爬,爬到它原本的父亲节点y,然后让y节点下降

* BST性质就是右子树上的点全比父亲节点大,现在x节点比y节点小. 为了不改变答案顺序,可以让y节点成为x节点的右儿子,y节点仍然大于x节点.

* 但x节点的右子树原来是有子树B的.

* x节点的右子树必然是大于x节点的,y节点必然是大于x节点的右子树和x节点的,然后因为x节点和它的右子树都是在y节点的左子树,都比它小

* 既然如此的话,我们为什么不把x节点原来的右子树,放在节点y的左子树上面呢?这样的话,我们就巧妙地避开了冲突,达成了x节点上移,y节点下移.

移动后：
![avatar](http://img.blog.csdn.net/20170825215730979?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzA5NzQzNjk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

这就是一种情况,但是我们不能够局限一种情况,我们要找到通解.

以下为通解
若节点x为y节点的位置z.( z为0则为左节点,1则为右节点 )
* 那么y节点就放到x节点的z^1的位置.(x为y的右子树,那么y就放到x的左子树；x为y的左子树,那么y就放到x的右子树位置)
* 如果说x节点的z^1的位置上,已经有节点,或者一棵子树,那么我们就将原来x节点z^1位置上的子树,放到y节点的位置z上面.(自己可以画图理解一下)
* 移动完毕.



```cpp

#include <bits/stdc++.h>
using namespace std;
const int N = 201000;
struct splay_tree
{
	int fa, cnt, ch[2], val, size;
} t[N];
int root, tot;

void update(int x)
{
	t[x].size = t[t[x].ch[0]].size + t[t[x].ch[1]].size + t[x].cnt;//左子树+右子树+本身多少个,cnt记录重复个数.
}

void rotate(int x)//x是要旋转的节点
{
	int y = t[x].fa;//X的父亲
	int z = t[y].fa;//X的祖父
	int k = t[y].ch[1] == x;//X是Y的哪一个儿子 0是左儿子 1是右儿子
	t[z].ch[t[z].ch[1] == y] = x;//Z的原来的Y的位置变为X
	t[x].fa = z;//X的父亲变成Z
	t[y].ch[k] = t[x].ch[k ^ 1];//X的与X原来在Y的相对的那个儿子变成Y的儿子
	t[t[x].ch[k ^ 1]].fa = y;//更新父节点
	t[x].ch[k ^ 1] = y;//X的 与X原来相对位置的儿子变成 Y
	t[y].fa = x;//更新父节点
	update(y); update(x);//yyb大佬忘记写了,这里是上传修改.
}

void splay(int x, int goal)//将x旋转为goal的儿子，如果goal是0则旋转到根
{
	while (t[x].fa != goal)//一直旋转到x成为goal的儿子
	{
		int y = t[x].fa, z = t[y].fa;//父节点祖父节点
		if (z != goal)//如果Y不是根节点，则分为上面两类来旋转
			(t[z].ch[0] == y) ^ (t[y].ch[0] == x) ? rotate(x) : rotate(y);//判断共线还是不共线
			//这就是之前对于x和y是哪个儿子的讨论
		rotate(x);//无论怎么样最后的一个操作都是旋转x
	}
	if (goal == 0)root = x;//如果goal是0，则将根节点更新为x
}

inline void find(int x)//查找x的位置，并将其旋转到根节点
{
	int u = root;
	if (!u)return;//树空
	while (t[u].ch[x > t[u].val] && x != t[u].val)//当存在儿子并且当前位置的值不等于x
		u = t[u].ch[x > t[u].val];//跳转到儿子，查找x的父节点
	splay(u, 0);//把当前位置旋转到根节点
}

inline void insert(int x)//插入x
{
	int u = root, fa = 0;//当前位置u，u的父节点fa
	while (u && t[u].val != x)//当u存在并且没有移动到当前的值
	{
		fa = u;//向下u的儿子，父节点变为u
		u = t[u].ch[x > t[u].val];//大于当前位置则向右找，否则向左找
	}
	if (u)//存在这个值的位置
		t[u].cnt++;//增加一个数
	else//不存在这个数字，要新建一个节点来存放
	{
		u = ++tot;//新节点的位置
		if (fa)//如果父节点非根
			t[fa].ch[x > t[fa].val] = u;
		t[u].ch[0] = t[u].ch[1] = 0;//不存在儿子
		t[tot].fa = fa;//父节点
		t[tot].val = x;//值
		t[tot].cnt = 1;//数量
		t[tot].size = 1;//大小
	}
	splay(u, 0);//把当前位置移到根，保证结构的平衡
}

inline int Next(int x, int f)//查找x的前驱(0)或者后继(1)
{
	find(x);
	int u = root;//根节点，此时x的父节点（存在的话）就是根节点
	if (t[u].val > x && f)return u;//如果当前节点的值大于x并且要查找的是后继
	if (t[u].val < x && !f)return u;//如果当前节点的值小于x并且要查找的是前驱
	u = t[u].ch[f];//查找后继的话在右儿子上找，前驱在左儿子上找
	while (t[u].ch[f ^ 1])u = t[u].ch[f ^ 1];//要反着跳转，否则会越来越大（越来越小）
	return u;//返回位置
}

inline void Delete(int x)//删除x
{
	int last = Next(x, 0);//查找x的前驱
	int next = Next(x, 1);//查找x的后继
	splay(last, 0); splay(next, last);
	//将前驱旋转到根节点，后继旋转到根节点下面
	//很明显，此时后继是前驱的右儿子，x是后继的左儿子，并且x是叶子节点
	int del = t[next].ch[0];//后继的左儿子
	if (t[del].cnt > 1)//如果超过一个
	{
		t[del].cnt--;//直接减少一个
		splay(del, 0);//旋转
	}
	else
		t[next].ch[0] = 0;//这个节点直接丢掉（不存在了）
}

inline int kth(int x)//查找排名为x的数
{
	int u = root;//当前根节点
	if (t[u].size < x)//如果当前树上没有这么多数
		return 0;//不存在
	while (1)
	{
		int y = t[u].ch[0];//左儿子
		if (x > t[y].size + t[u].cnt)
			//如果排名比左儿子的大小和当前节点的数量要大
		{
			x -= t[y].size + t[u].cnt;//数量减少
			u = t[u].ch[1];//那么当前排名的数一定在右儿子上找
		}
		else//否则的话在当前节点或者左儿子上查找
			if (t[y].size >= x)//左儿子的节点数足够
				u = y;//在左儿子上继续找
			else//否则就是在当前根节点上
				return t[u].val;
	}
}

int main()
{
	int n;
	scanf("%d", &n);
	insert(1e9);
	insert(-1e9);
	while (n--)
	{
		int opt, x;
		scanf("%d%d", &opt, &x);
		if (opt == 1)//插入数值x
			insert(x);
		if (opt == 2)//删除数值x(若有多个相同的数，应只删除一个)
			Delete(x);
		if (opt == 3)//查询数值x的排名(若有多个相同的数，应输出最小的排名)
		{
			find(x);
			printf("%d\n", t[t[root].ch[0]].size);
		}
		if (opt == 4)//查询排名为x的数值
			printf("%d\n", kth(x + 1));
		if (opt == 5)//求数值x的前驱(前驱定义为小于x的最大的数)
			printf("%d\n", t[Next(x, 0)].val);
		if (opt == 6)//求数值x的后继(后继定义为大于x的最小的数)
			printf("%d\n", t[Next(x, 1)].val);
	}
	return 0;
}

```