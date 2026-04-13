### 问题描述

某景区一共有 N 个景点，编号 1 到 N。景点之间共有 N−1 条双向的摆渡车线路相连，形成一棵树状结构。在景点之间往返只能通过这些摆渡车进行，需要花费一定的时间。

小明是这个景区的资深导游，他每天都要按固定顺序带客人游览其中 K 个景点：A1​,A2​,…,AK​。今天由于时间原因，小明决定跳过其中一个景点，只带游客按顺序游览其中 K−1 个景点。具体来说，如果小明选择跳过 Ai​，那么他会按顺序带游客游览 A1​,A2​,…,Ai−1​,Ai+1​,…,AK​，（1≤i≤K)。

请你对任意一个 Ai​，计算如果跳过这个景点，小明需要花费多少时间在景点之间的摆渡车上？

### 输入格式

第一行包含 2 个整数 N 和 K。

以下 N−1 行，每行包含 3 个整数u,v 和 t，代表景点 u 和 v 之间有摆渡车线路，花费 t 个单位时间。

最后一行包含 K 个整数A1​,A2​,…,AK​，代表原定游览线路。

### 输出格式

输出 K 个整数，其中第 i 个代表跳过 Ai​ 之后，花费在摆渡车上的时间。

### 样例输入

```
6 4
1 2 1
1 3 1
3 4 2
3 5 2
4 6 3
2 6 5 1
```

### 样例输出

```
10 7 13 14
```

### 样例说明

原路线是 2→6→5→1

当跳过 22 时，路线是 6→5→1，其中 6→5 花费时间 3+2+2=7，5→1花费时间 2+1=32+1=3，总时间花费 10。

当跳过 66 时，路线是 2→5→1，其中 2→5 花费时间 1+1+2=4，5→1 花费时间 2+1=32+1=3，总时间花费 7。

当跳过 5 时，路线是 2→6→1，其中 2→6 花费时间 1+1+2+3=7，6→1 花费时间 3+2+1=6，总时间花费 13。

当跳过 1 时，路线时 2→6→5，其中 2→6花费时间 1+1+2+3=7，6→5花费时间 3+2+2=7，总时间花费 14。

### 评测用例规模与约定

对于 20的数据，2≤K≤N≤102。

对于 40 的数据，2≤K≤N≤104。

对于 100 的数据，2≤K≤N≤105，1≤u,v,Ai≤N，1≤t≤105。保证 Ai​ 两两不同。

# 题解
```cpp
//本题主要考察LCA
 //若要求u->v的路径长度,等于求u->root+v->root - root->2*LCA(u,v) 
 //即分别从u和v走到根结点,再减去2倍的根结点到(u,v)的最近公共祖先的路径长度 
 //同理,若原本的遍历顺序是a->b->c,总路径长度为sum,若要跳过b,则新的路径长度为 
 //sum-dist(a,b)-dist(b,c)+dist(a,c),可画图验证 
 #include <bits/stdc++.h> 
 using namespace std; 
 typedef long long ll; 
 const int N=1e5+100; 
 int n,k; 
 int deep[N];
 //深度 
 int dp[N][21];
 //dp[i][j]表示从i结点开始跳2^j步可到达的结点 
 vector<int>edge[N];//边 
 vector<int>weight[N];//权值 
 ll path[N];//原始的游览路线 
 ll dist[N];//dist[i]存储i到根结点的距离 
 void dfs(int u,int father)//LCA的前置算法(模板) 
 { 
 deep[u]=deep[father]+1;
  dp[u][0]=father; 
  for(int i=1;i<=20;i++) 
  { 
  dp[u][i]=dp[dp[u][i-1]][i-1]; 
  } 
  for(int i=0;i<edge[u].size();i++)
   { 
   int v=edge[u][i],w=weight[u][i];
if(v==father)continue; 
dist[v]=dist[u]+w; 
dfs(v,u); 
}
 } 
 
 
 int LCA(int x,int y)//求x和y的最近公共祖先(模板)
  { 
  if(deep[x]<deep[y])swap(x,y);
   for(int i=20;i>=0;i--) 
   { 
   if(deep[dp[x][i]]>=deep[y]) 
   {
    x=dp[x][i]; 
    }
     } 
     if(x==y)return x; 
     for(int i=20;i>=0;i--) 
     {
      if(dp[x][i]!=dp[y][i]) 
      {
       x=dp[x][i]; 
       y=dp[y][i];
        }
	} 
         return dp[x][0];
    } 
        
  ll get_dist(int x,int y)//求x和y的距离
     { 
     if(x==0||y==0)return 0;
      return dist[x]+dist[y]-2*dist[LCA(x,y)];
       } 
       
       int main() 
       { 
       scanf("%d%d",&n,&k);
        for(int i=1;i<n;i++)//插入n-1条无向边 
        { 
        int u,v,w; 
        scanf("%d%d%d",&u,&v,&w); 
        edge[u].push_back(v);
         edge[v].push_back(u);
          weight[u].push_back(w); 
          weight[v].push_back(w); 
          } 
          dfs(1,0);//跑一遍dfs为LCA做准备
           ll sum=0;//sum存储原始游览路线的总路径长度 
           for(int i=1;i<=k;i++) 
           { 
           scanf("%d",&path[i]);
            sum+=get_dist(path[i],path[i-1]);//依次累加 
            } 
            for(int i=1;i<=k;i++)//除去第i个景点 
            { 
            ll dist1=get_dist(path[i],path[i-1]);
            ll dist2=get_dist(path[i],path[i+1]); 
            ll dist3=get_dist(path[i-1],path[i+1]); 
            printf("%lld ",sum-dist1-dist2+dist3);//套公式计算即可
             }
              printf("\n");
               return 0;
                }
```