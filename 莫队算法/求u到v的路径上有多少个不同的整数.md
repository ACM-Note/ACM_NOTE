## 求u到v的路径上有多少个不同的整数
[[SP10707 COT2 - Count on a tree II](https://www.luogu.org/problem/SP10707)]

#### 题面
给定一个n个节点的树，每个节点表示一个整数，问u到v的路径上有多少个不同的整数。
第一行有两个整数n和m（n＝40000，m＝100000）。
第二行有n个整数。第i个整数表示第i个节点表示的整数。
在接下来的n-1行中，每行包含两个整数u v，描述一条边（u，v）。
在接下来的m行中，每一行包含两个整数u v，询问u到v的路径上有多少个不同的整数。

#### 代码

```cpp
#include<iostream>
#include<cstdio>
#include<cstdlib>
#include<cstring>
#include<cmath>
#include<algorithm>
#include<set>
#include<map>
#include<vector>
#include<queue>
using namespace std;
#define ll long long
#define RG register
#define MAX 111111
inline int read()
{
    RG int x=0,t=1;RG char ch=getchar();
    while((ch<'0'||ch>'9')&&ch!='-')ch=getchar();
    if(ch=='-')t=-1,ch=getchar();
    while(ch<='9'&&ch>='0')x=x*10+ch-48,ch=getchar();
    return x*t;
}
struct Line{int v,next;}e[MAX];
int h[MAX],cnt=1;
inline void Add(int u,int v){e[cnt]=(Line){v,h[u]};h[u]=cnt++;}
int S[MAX],a[MAX],top,G[MAX],gr;
int f[16][MAX],dep[MAX],dfn[MAX],tim;
int n,m,blk,ans,Ans[MAX],sum[MAX];
bool vis[MAX];
int dfs(int u,int ff)
{
    f[0][u]=ff;dep[u]=dep[ff]+1;dfn[u]=++tim;
    for(int i=1;i<=15;++i)f[i][u]=f[i-1][f[i-1][u]];
    int ret=0;
    for(int i=h[u];i;i=e[i].next)
    {
        int v=e[i].v;if(v==ff)continue;
        ret+=dfs(v,u);
        if(ret>=blk){++gr;while(ret--)G[S[top--]]=gr;ret=0;}
    }
    S[++top]=u;
    return ret+1;
}
int LCA(int u,int v)
{
    if(dep[u]<dep[v])swap(u,v);
    for(int i=15;~i;--i)if(dep[f[i][u]]>=dep[v])u=f[i][u];
    if(u==v)return u;
    for(int i=15;~i;--i)if(f[i][u]!=f[i][v])u=f[i][u],v=f[i][v];
    return f[0][u];
}
struct query{int id,u,v,lb,rb;}q[MAX];
bool operator<(query a,query b){if(a.lb!=b.lb)return a.lb<b.lb;return a.rb<b.rb;}
void Change(int x)
{
    if(vis[x])--sum[a[x]],ans-=!sum[a[x]];
    else ans+=!sum[a[x]],++sum[a[x]];
    vis[x]^=1;
}
void ModifyLink(int u,int v)
{
    while(u!=v)
    {
        if(dep[u]<dep[v])swap(u,v);
        Change(u);u=f[0][u];
    }
}
int main()
{
    n=read();m=read();blk=sqrt(n);
    for(int i=1;i<=n;++i)S[i]=a[i]=read();
    sort(&S[1],&S[n+1]);top=unique(&S[1],&S[n+1])-S-1;
    for(int i=1;i<=n;++i)a[i]=lower_bound(&S[1],&S[top+1],a[i])-S;
    for(int i=1,u,v;i<n;++i)u=read(),v=read(),Add(u,v),Add(v,u);
    top=0;dfs(1,0);
    for(int i=1;i<=m;++i)
    {
        int u=read(),v=read();if(dfn[u]>dfn[v])swap(u,v);
        q[i]=(query){i,u,v,G[u],G[v]};
    }
    sort(&q[1],&q[m+1]);
    int lca=LCA(q[1].u,q[1].v);
    ModifyLink(q[1].u,q[1].v);
    Change(lca);Ans[q[1].id]=ans;Change(lca);
    for(int i=2;i<=m;++i)
    {
        ModifyLink(q[i-1].u,q[i].u);
        ModifyLink(q[i-1].v,q[i].v);
        lca=LCA(q[i].u,q[i].v);
        Change(lca);Ans[q[i].id]=ans;Change(lca);
    }
    for(int i=1;i<=m;++i)printf("%d\n",Ans[i]);
    return 0;
}

```