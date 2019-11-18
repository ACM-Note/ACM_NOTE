## LCA

### 例题
[[P3379 【模板】最近公共祖先（LCA）](https://www.luogu.org/problem/P3379)]

### 题意

如题，给定一棵有根多叉树，请求出指定两个点直接最近的公共祖先。
**输入**
第一行包含三个正整数N、M、S，分别表示树的结点个数、询问的个数和树根结点的序号。
接下来N-1行每行包含两个正整数x、y，表示x结点和y结点之间有一条直接连接的边（数据保证可以构成树）。
接下来M行每行包含两个正整数a、b，表示询问a结点和b结点的最近公共祖先。
**输出格式**
输出包含M行，每行包含一个正整数，依次为每一个询问的结果。

Time limit：1.00s
Memory limit：500.00MB

### 题解
直接套用模板就可以了，其实倍增这道题并不会崩掉，反而，倍增来做这道题，其实更利于初学者理解（比如我）。
注意的是，因为有些题数据可能会太大，要注意数组会不会超内存。同时因为是一个树，所以所有边是双向的，就需要把struct开两倍大小。

### 代码

```cpp

#include<cstdio>
#include<iostream>
#include<cstring>
using namespace std;
const int maxn=500000+2;
int n,m,s;
int k=0;
int head[maxn],d[maxn],p[maxn][21];
//head数组就是链接表标配了吧？
//d存的是深度（deep）
//p[i][j]存的[i]向上走2的j次方那么长的路径
struct node{
    int v,next;
}e[maxn*2];//存树

void add(int u,int v)
{
    e[k].v=v;
    e[k].next=head[u];
    head[u]=k++;
}

void dfs(int u,int fa)
{
    d[u]=d[fa]+1;
    p[u][0]=fa;
    for(int i=1;(1<<i)<=d[u];i++)
        p[u][i]=p[p[u][i-1]][i-1];
    for(int i=head[u];i!=-1;i=e[i].next)
    {
        int v=e[i].v;
        if(v!=fa)
            dfs(v,u);
    }
}

//首先进行的预处理，将所有点的deep和p的初始值dfs出来
int lca(int a,int b)
{
    if(d[a]>d[b])
        swap(a,b);//保证a是在b结点上方，即a的深度小于b的深度
    for(int i=20;i>=0;i--)
        if(d[a]<=d[b]-(1<<i))
            b=p[b][i];//先把b移到和a同一个深度
    if(a==b)//特判，若b上来和就和a一样了,那就可以直接返回答案了
        return a;
    for(int i=20;i>=0;i--)
    {
        if(p[a][i]==p[b][i])
            continue;
        else
            a=p[a][i],b=p[b][i];//A和B一起上移
    }
    return p[a][0];//找出最后a值的数字
}

int main()
{
    memset(head,-1,sizeof(head));
    int a,b;
    scanf("%d%d%d",&n,&m,&s);
    for(int i=1;i<n;i++)
    {
        scanf("%d%d",&a,&b);
        add(a,b);//无向图，要加两次
        add(b,a);
    }
    dfs(s,0);
    for(int i=1;i<=m;i++)
    {
        scanf("%d%d",&a,&b);
        printf("%d\n",lca(a,b));
    }
    return 0;
}

```