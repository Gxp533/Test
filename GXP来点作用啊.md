

# 图

## 一  最短路径

### Dijkstra

求解单源最短路径问题

#### 1 邻接矩阵

初始化时将dis数组初始化为inf，dis[s]初始化为0，在执行第一次循环后会将dis数组赋值为各个点与s点的距离

```cc
void dijkstra(int s)
{
    memset(dis,0x3f,sizeof(dis));
    memset(vis,0,sizeof(vis));
    dis[1]=0;
    for(int t=0;t<n;++t)
    {
        int u=-1;
        for(int i=1;i<=n;++i)
        {
            if(!vis[i]&&(u==-1||dis[u]>=dis[i]))
                u=i;
        }
        if(u==-1)
            break;
        vis[u]=1;
        for(int i=1;i<=n;++i)
            if(dis[i]>dis[u]+g[u][i])
                dis[i]=dis[u]+g[u][i];
    }
}
```

#### 2 堆优化

一般用于优化单向路之间的最短路径问题

1. 数组g[i]存放着与第i条边相连的边;
2. 每次松弛将在优先队列中加入一条边e2，而原来e1的数据不会被删除，只是被e2所顶替了（由于优先队列的性质），达到更新的目的；

```cc
struct node{
    int to,w;
    bool operator<(const edge& p) const
    {
		return w>p.w;//优先队列默认大根堆
    }
};

vector<node> g[N]；//存放邻接表
    
void dijikstra(int s)
{
	memset(dis,0x3f,sizeof(dis));
	priority_queue<node> que;
	que.push({1,0});
	dis[s]=0;
	while(!que.empty())
	{
		node cur=que.top();
		que.pop();
		int u=cur.to;
		if(vis[u])
		    continue;
		vis[u]=1;
		for(int i=0;i<g[u].size();++i)
		{
			int v=g[u][i].to;
			if(dis[v]>dis[u]+g[u][i].w)
			{
				dis[v]=dis[u]+g[u][i].w;
				que.push({v,dis[v]});
			}
		}
	}
}
```



### Floyd

求解多源点的最短路径问题

1. 初始化邻接矩阵即可，不相邻设置为inf；
2. i点和j点之间能否经过k点进行路径的优化；

```c++
void floyd()
{
	for(int k=0;k<n;k++)
        for(int i=0;i<n;i++)
            for(int j=0;j<n;j++)
                if(map[i][j]>map[i][k]+map[k][j])
                    map[i][j]=map[i][k]+map[k][j];
}
```



### Bellman-Ford

单源点最短路径（包含**负权边**）

1. 和dijkstra算法大同小异，区别在dijkstra是通过松弛点来求解而Bellman-Ford是通过n-1次松弛边的操作来求解；

2. 一个无负环的图的最短路径至多包含n-1边，对边进行n-1次松弛就一定求解出最短路径；

   ```cc
   struct node{
       int x,y,w;
   }e[M];
   
   void Bellamn-Ford(int s)
   {
       memset(dis,0x3f,sizeof(dis));
       dis[1]=0;
       for(int i=0;i<k;++i)
       {
           memcpy(backup,dis,sizeof(dis));
           for(int j=1;j<=m;++j)
               dis[e[j].y]=min(backup[e[j].x]+e[j].w,dis[e[j].y]);
       }
       return dis[n];
   }
   ```
   
   

### Spfa

Bellman-Ford的优化算法，BF算法中盲目松弛n-1次来求解最短路径。而spfa算法则优化了这一过程。只遍历能松弛的边的邻边。

一般适配于邻接表。

#### 1  链式前向星模拟的邻接表

```cc
int h[N],e[N],w[N],ne[N],idx;

void add(int a,int b,int c)
{
    e[idx]=b;
    w[idx]=c;
    ne[idx]=h[a];
    h[a]=idx++;
}

void spfa()
{
    memset(dis,0x3f,sizeof(dis));
    memset(vis,0,sizeof(vis));
    dis[1]=0;
    queue<int> que;
    que.push(1);
    vis[1]=1;
    while(!que.empty())
    {
        int cur=que.front();
        que.pop();
        vis[cur]=0;
        for(int i=h[cur];i!=-1;i=ne[i])
        {
            int j=e[i];
            if(dis[j]>dis[cur]+w[i])
            {
                dis[j]=dis[cur]+w[i];
                if(!vis[j])
                {
                    vis[j]=1;
                    que.push(j);
                }
            }
        }
    }
}
```



#### 2 vector模拟的邻接表 

和dijk算法中的邻接表一样

```cc
struct edge{
	int to,w;
    edge(int x,int y)
    {
	   to=x;
        w=y;
    }
}
int dis[N];
bool vis[N];
vector<edge> g[N];

void spfa(int s)
{
    memset(dis,inf,sizeof(dis));
    memset(vis,0,sizeof(vis));
  	queue<edge> que;
    que.push(edge(s,0));
    while(!que.empty())
    {
        edge cur=que.front();
        que.pop();
        int u=cur.to;
        vis[u]=0;
        for(int i=0;i<g[u].size();i++)
        {
            int v=g[u][i].to;
            if(dis[v]>dis[u]+g[u][i].w)
            {
                dis[v]=dis[u]+g[u][i].w;
                if(!vis[v])
                {
                    que.push(g[u][i]);//区别于dijk，根本在于spfa是通过松弛邻边来进行求解
                    vis[v]=1;
                }
            }
		}

    }
}

```



### 负环判断

spfa和Bellman-Ford都能解决负环的判断

#### Bellman-Ford判断

和求解最短路径一样，不过在求解完最短路径后，即进行了n-1次松弛后再遍历每一条边，如果还能进行松弛则说明存在负环

```c++
struct edge{
    int start,end,w;
}e[N];
bool Bellamn-Ford(int s)
{
    for(int i=0;i<n;i++)
        dis[i]=inf;
    dis[0]=0;
    for(int i=0;i<n-1;i++)
		for(int j=0;j<edge_size;j++)
            if(dis[e[j].end]>dis[e[j].start]+e[j].w)
                dis[e[j].end]=dis[e[j].start]+e[j].w;
    for(int i=0;i<n-1;i++)
        for(int j=0;j<edge.size();j++)
            if(dis[e[j].end]>dis[e[j].start]+e[j].w);
    			return true;//存在
    return false;//不存在
}
```

#### Spfa判断

原理同上，若循环次数超过n-1次说明有负环。也可以用单个点的入队次数进行，如果一个点的入队次数大于等于n则说明存在负环，引入cnt数组来记录每给点的入队次数

```cc
bool spfa()
{
	queue<int> que;
		
	for(int i=1;i<=n;++i)
	{
		que.push(i);
		vis[i]=1;
	}
	
	while(!que.empty())
	{
		int cur=que.front();
		que.pop();
		vis[cur]=0;
		for(int i=h[cur];i!=-1;i=ne[i])
		{
			int j=e[i];
			if(dis[j]>dis[cur]+w[i])
			{
				dis[j]=dis[cur]+w[i];
				if(++cnt[j]>=n)
					return true;
				if(!vis[j])
				{
					vis[j]=1;
					que.push(j);
				}
			}
		}
	}
	return false;
}
```



## 二  最小生成树

### Prim

和dijk最短路径一样，区别就在于dis数组存放的是边的权值不需要累积，每次只需将最小的边取出就行。

```cc
int prime()
{
    memset(dis,0x3f,sizeof(dis));
    int res=0;
    dis[1]=0;
    for(int t=0;t<n;++t)
    {
        int u=-1;
        for(int i=1;i<=n;++i)
            if(!vis[i]&&(u==-1||dis[u]>dis[i]))
                u=i;
        if(dis[u]==inf)
            return inf;
        res+=dis[u];
        vis[u]=1;
        for(int i=1;i<=n;++i)
            dis[i]=min(dis[i],g[u][i]);
    }
    return res;
}
```



### Kruskal

主要思想是每次在剩余的边中挑选出一条权值最小且不会构成回路的边加入到树中，执行n-1次（终止条件）就能得到最小生成树。

1.用优先队列来保存边的信息，使每次弹出的边都是权值最小的边；2.使用并查集来验证新加入的边是否会构成回路，

具体做法是：每次向森林中加边时都将边的起点和终点merge，这样保证了已使用的且相连的边都有一个共同父辈。算

法执行完后每个点都有共同父辈,即连通。

#### 1  优先队列

```cc
struct edge{
    int start,end,w;
    edge(int x,int y,iny z)
    {
        start=x;
        end=y;
        w=z;
    }
    bool operator<(const edge& p) const
    {
        return w>p.w;
    }
}
priority_queue<edge> que;
int pre[N];
void init()
{
    for(int i=0;i<n;i++)
        pre[i]=i;
}
int find(int x)
{
    if(x==pre[x])
        return x;
    return pre[x]=find(pre[x]);
}
void unio(int x,int y)
{
    x=find(x);
    y=find(y);
    if(x!=y)
    {
        pre[x]=y;
    }
}
int kurskal()
{
    int res=0,cnt=0,sz=que.size();
    for(int i=0;i<sz;i++)//i<que.size()不行，队列的大小是在变化的
    {
        edge cur=que.top();
        que.pop();
        int s=find(cur.start);
        int e=find(cur.end);
        if(s!=e)
        {
            res+=cur.w;
            pre[s]=e;//相当于unio，避免了重复操作
            cnt++;
        }
        if(cnt==n-1)
            break;
    }
    return res;
}
```

#### 2 排序

利用了堆来写的kruskal适用于稠密图，而对于边数较少的图来说的话，适用排序往往会有更好的效率。与优先队列大同小异。

```cc
int pre[N];
struct node{
	int x,y,w;
	bool operator<(const node& p) const
	{
		return w<p.w;
	}
}e[M];

int find(int x)
{
	if(x==pre[x])
		return x;
	return pre[x]=find(pre[x]);
}

bool merge(int x,int y)
{
	x=find(x);
	y=find(y);
	if(x==y)
		return false;
	pre[x]=y;
	return true;
}

void krus()
{
    sort(e+1,e+m+1);
	for(int i=1;i<=m;++i)
	{
		if(merge(e[i].x,e[i].y))
		{
			res+=e[i].w;
			cnt++;
		}
		if(cnt==n-1)
			break;
	}
    if(cnt==n-1)
    	return res;
	return -1;
}
```

