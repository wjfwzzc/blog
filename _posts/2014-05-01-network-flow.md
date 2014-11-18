---
layout: post
title: 最大流的一些板子
date: 2014-05-01 02:06:00
category: acm
tags: acm
comments: true
---
昂神这两天放出了[Andrew Stankevich's Contest #1](http://acm.hust.edu.cn/vjudge/contest/view.action?cid=45179)的题目（ZOJ2313～2320或SGU193～200），以为自己做不完但还真的AK了，觉得题目质量相当不错。A题[Chinese Girls' Amusement](http://acm.zju.edu.cn/onlinejudge/showProblem.do?problemCode=2313)大数板子题，用C++和Java各写一发；C题[New Year Bonus Grant](http://acm.zju.edu.cn/onlinejudge/showProblem.do?problemCode=2315)用贪心和树形dp各写一发，这也是自己第一次写树形dp；D题[Matrix Multiplication](http://acm.zju.edu.cn/onlinejudge/showProblem.do?problemCode=2316)签到；E题[Nice Patterns Strike Back](http://acm.zju.edu.cn/onlinejudge/showProblem.do?problemCode=2317)状压dp+矩阵快速幂+大数板子；F题[Get Out!](http://acm.zju.edu.cn/onlinejudge/showProblem.do?problemCode=2318)计算几何巧妙转化成图论spfa判负环，非常棒的一道题；G题[Beautiful People](http://acm.zju.edu.cn/onlinejudge/showProblem.do?problemCode=2319)排序+LIS+回溯输出；H题Gauss消元求异或方程组自由变元+大数板子；不过这些不是这篇文章的重点。

B题[Reactor Cooling](http://acm.zju.edu.cn/onlinejudge/showProblem.do?problemCode=2314)是一道裸的无源汇带上下界网络流，这也是自己第一次写带上下界的网络流。带上下界的网络流主要是通过重新构图，重构图的最大流即是原图的可行流。这里强烈推荐周源大大的论文[《一种简易的方法求解流量有上下界的网络中网络流问题》](http://wenku.baidu.com/view/0f3b691c59eef8c75fbfb35c.html)，清晰简洁通俗易懂。

简单地说，假设每条边$$u\rightarrow v$$的流量下界是l，流量上界是r，则重构图时，添加容量为$$r-l$$的边$$u\rightarrow v$$；再建立附加源点S'，附加汇点T'，统计每个点k的总流入量下界in[k]和总流出量下界out[k]，如果$$in>out$$则添加流量为$$in-out$$的边$$S'\rightarrow k$$，如果$$in<out$$则添加流量为$$out-in$$的边$$k\rightarrow T'$$。这样，如果$$S'\rightarrow T'$$的最大流满流（S'的每条边都满流），则原图有可行流；此时新图每条边的流量加下界l就是原图对应边的流量。

具体证明可以看上面给出的论文，很好理解。

然后就是测验各种板子的时间。我不知道Thor使用的板子是什么（在Damocles图论一般归Thor处理），我的板子大都是在kuangbin大大的基础上修改来的（咳咳，貌似没怎么改的样子），以ISAP为主，全部加了GAP优化；这道题数据量挺小，各个板子之间看不出多少速度差距。过几天再找几道题测一下。

更新：已测[poj1273](http://poj.org/problem?id=1273)与[hdu1532](http://acm.hdu.edu.cn/showproblem.php?pid=1532)（题目一致数据不同），[LA3031](https://icpcarchive.ecs.baylor.edu/index.php?option=com_onlinejudge&Itemid=8&category=192&page=show_problem&problem=1032)与[poj1966](http://poj.org/problem?id=1966)（题目一致数据不同，最小点割集），以及[UVa820](http://uva.onlinejudge.org/index.php?option=com_onlinejudge&Itemid=8&category=56&page=show_problem&problem=761)，数据量都不大，ISAP和Dinic相差不大，互有胜负，看来还是要根据图的特点而定。在无法准确判断图的特点的情况下，优先考虑使用ISAP+邻接表，酌情加优化。

更新：已测号称数据量巨大的[poj3469](http://poj.org/problem?id=3469)，Dinic跑出5000+ms，加入当前弧优化后跑出3500+ms，而无论是否加当前弧优化的ISAP都在2300+ms左右。

关于各个板子的速度，这里有几篇不错的blog：

- [USACO 4.2.1 Ditch 网络最大流问题算法小结](http://dantvt.is-programmer.com/posts/7974.html)
- [Dinic PK Isap](http://www.cnblogs.com/zhsl/archive/2012/12/03/2800092.html)

先是SAP+邻接矩阵（点的编号默认从0开始，如果从1开始也很好修改），cap存储容量，flow存储流量：
{% highlight cpp linenos %}
int maze[MAXN][MAXN];
int gap[MAXN],dis[MAXN],pre[MAXN],cur[MAXN],src,sink,nodenum;
int flow[MAXN][MAXN];
int sap() {
    memset(cur,0,sizeof(cur));
    memset(dis,0,sizeof(dis));
    memset(gap,0,sizeof(gap));
    memset(flow,0,sizeof(flow));
    int u=src,maxflow=0,aug=-1;
    pre[src]=src;
    gap[0]=nodenum;
    while(dis[src]<nodenum) {
loop:
        for(int v=cur[u]; v<nodenum; ++v)
            if(maze[u][v]-flow[u][v]&&dis[u]==dis[v]+1) {
                if(!~aug||aug>maze[u][v]-flow[u][v])
                    aug=maze[u][v]-flow[u][v];
                pre[v]=u;
                u=cur[u]=v;
                if(v==sink) {
                    maxflow+=aug;
                    for(u=pre[u]; v!=src; v=u,u=pre[u]) {
                        flow[u][v]+=aug;
                        flow[v][u]-=aug;
                    }
                    aug=-1;
                }
                goto loop;
            }
        int mindis=nodenum-1;
        for(int v=0; v<nodenum; ++v)
            if(maze[u][v]-flow[u][v]&&mindis>dis[v]) {
                cur[u]=v;
                mindis=dis[v];
            }
        if(--gap[dis[u]]==0)
            break;
        dis[u]=mindis+1;
        ++gap[dis[u]];
        u=pre[u];
    }
    return maxflow;
}
{% endhighlight %}

接着是ISAP+邻接表：
{% highlight cpp linenos %}
struct Edge {
    int to,next,cap,flow;
} edge[MAXM];
int tot,head[MAXN];
void init() {
    tot=0;
    memset(head,0xff,sizeof(head));
}
void addedge(int u,int v,int w,int rw=0) {
    edge[tot].to=v;
    edge[tot].cap=w;
    edge[tot].flow=0;
    edge[tot].next=head[u];
    head[u]=tot++;
    edge[tot].to=u;
    edge[tot].cap=rw;
    edge[tot].flow=0;
    edge[tot].next=head[v];
    head[v]=tot++;
}
int gap[MAXN],dep[MAXN],pre[MAXN],cur[MAXN],src,sink,nodenum;
int sap() {
    memset(gap,0,sizeof(gap));
    memset(dep,0,sizeof(dep));
    memcpy(cur,head,sizeof(head));
    int u=src,ret=0;
    pre[u]=-1;
    gap[0]=nodenum;
    while(dep[src]<nodenum) {
        if(u==sink) {
            int mindis=INF;
            for(int i=pre[u]; ~i; i=pre[edge[i^1].to])
                mindis=min(mindis,edge[i].cap-edge[i].flow);
            for(int i=pre[u]; ~i; i=pre[edge[i^1].to]) {
                edge[i].flow+=mindis;
                edge[i^1].flow-=mindis;
            }
            u=src;
            ret+=mindis;
            continue;
        }
        bool flag=false;
        int v;
        for(int i=cur[u]; ~i; i=edge[i].next) {
            v=edge[i].to;
            if(edge[i].cap-edge[i].flow&&dep[v]+1==dep[u]) {
                flag=true;
                cur[u]=pre[v]=i;
                break;
            }
        }
        if(flag) {
            u=v;
            continue;
        }
        int mindis=nodenum;
        for(int i=head[u]; ~i; i=edge[i].next) {
            v=edge[i].to;
            if(edge[i].cap-edge[i].flow&&dep[v]<mindis) {
                mindis=dep[v];
                cur[u]=i;
            }
        }
        if(--gap[dep[u]]==0)
            return ret;
        dep[u]=mindis+1;
        ++gap[dep[u]];
        if(u!=src)
            u=edge[pre[u]^1].to;
    }
    return ret;
}
{% endhighlight %}

然后是ISAP+bfs+stack+邻接表，其中bfs是进行了初始标号，不过据说加速意义并不大：
{% highlight cpp linenos %}
struct Edge {
    int to,next,cap,flow;
} edge[MAXM];
int tot,head[MAXN];
void init() {
    tot=0;
    memset(head,0xff,sizeof(head));
}
void addedge(int u,int v,int w,int rw=0) {
    edge[tot].to=v;
    edge[tot].cap=w;
    edge[tot].flow=0;
    edge[tot].next=head[u];
    head[u]=tot++;
    edge[tot].to=u;
    edge[tot].cap=rw;
    edge[tot].flow=0;
    edge[tot].next=head[v];
    head[v]=tot++;
}
int gap[MAXN],dep[MAXN],cur[MAXN],src,sink,nodenum;
void bfs() {
    memset(gap,0,sizeof(gap));
    memset(dep,0xff,sizeof(dep));
    queue<int> q;
    gap[0]=1;
    dep[sink]=0;
    q.push(sink);
    while(!q.empty()) {
        int u=q.front();
        q.pop();
        for(int i=head[u]; ~i; i=edge[i].next) {
            int v=edge[i].to;
            if(~dep[v])
                continue;
            q.push(v);
            dep[v]=dep[u]+1;
            ++gap[dep[v]];
        }
    }
}
int S[MAXN];
int sap() {
    bfs();
    memcpy(cur,head,sizeof(head));
    int top=0,u=src,ret=0;
    while(dep[src]<nodenum) {
        if(u==sink) {
            int mindis=INF,inser;
            for(int i=0; i<top; ++i)
                if(mindis>edge[S[i]].cap-edge[S[i]].flow) {
                    mindis=edge[S[i]].cap-edge[S[i]].flow;
                    inser=i;
                }
            for(int i=0; i<top; ++i) {
                edge[S[i]].flow+=mindis;
                edge[S[i]^1].flow-=mindis;
            }
            ret+=mindis;
            top=inser;
            u=edge[S[top]^1].to;
            continue;
        }
        bool flag=false;
        int v;
        for(int i=cur[u]; ~i; i=edge[i].next) {
            v=edge[i].to;
            if(edge[i].cap-edge[i].flow&&dep[v]+1==dep[u]) {
                flag=true;
                cur[u]=i;
                break;
            }
        }
        if(flag) {
            S[top++]=cur[u];
            u=v;
            continue;
        }
        int mindis=nodenum;
        for(int i=head[u]; ~i; i=edge[i].next) {
            v=edge[i].to;
            if(edge[i].cap-edge[i].flow&&dep[v]<mindis) {
                mindis=dep[v];
                cur[u]=i;
            }
        }
        if(--gap[dep[u]]==0)
            return ret;
        dep[u]=mindis+1;
        ++gap[dep[u]];
        if(u!=src)
            u=edge[S[--top]^1].to;
    }
    return ret;
}
{% endhighlight %}

一个修改自交大的Dinic板子，加了当前弧优化。
{% highlight cpp linenos %}
struct Edge {
    int to,next,cap,flow;
} edge[MAXM];
int tot,head[MAXN];
void init() {
    tot=0;
    memset(head,0xff,sizeof(head));
}
void addedge(int u,int v,int w,int rw=0) {
    edge[tot].to=v;
    edge[tot].cap=w;
    edge[tot].flow=0;
    edge[tot].next=head[u];
    head[u]=tot++;
    edge[tot].to=u;
    edge[tot].cap=rw;
    edge[tot].flow=0;
    edge[tot].next=head[v];
    head[v]=tot++;
}
int dis[MAXN],cur[MAXN],src,sink;
bool bfs() {
    memset(dis,0,sizeof(dis));
    queue<int> q;
    dis[src]=1;
    q.push(src);
    while(!q.empty()) {
        int u=q.front();
        q.pop();
        for(int i=head[u]; ~i; i=edge[i].next) {
            int v=edge[i].to;
            if(edge[i].cap>edge[i].flow&&!dis[v]) {
                q.push(v);
                dis[v]=dis[u]+1;
            }
        }
    }
    return dis[sink];
}
int dfs(int u,int delta=INF) {
    if(u==sink||!delta)
        return delta;
    int ret=0;
    for(int &i=cur[u]; delta&&~i; i=edge[i].next) {
        int v=edge[i].to;
        if(edge[i].cap>edge[i].flow&&dis[v]==dis[u]+1) {
            int aug=dfs(v,min(edge[i].cap-edge[i].flow,delta));
            if(!aug)
                continue;
            edge[i].flow+=aug;
            edge[i^1].flow-=aug;
            delta-=aug;
            ret+=aug;
        }
    }
    return ret;
}
int dinic() {
    int ret=0;
    while(bfs()) {
        memcpy(cur,head,sizeof(cur));
        ret+=dfs(src);
    }
    return ret;
}
{% endhighlight %}

此外以上都是基于增广路的（ISAP除外，它算是一个杂糅）。

我一直在寻找HLPP（最高标号预流推进）的板子，这是目前为止我所知道的时间复杂度最低的网络流算法，但也因为其较高的编程复杂度导致很少有人用。很偶然地在看Andrew Stankevich's Contest #3的题解时，我发现其中一道阻塞流的题watashi使用了HLPP，就借鉴了过来。很奇怪watashi的代码一贯靠谱，但这次直接拷贝他的代码会编译错误，所以我做了些修改，并在[UVa820](http://uva.onlinejudge.org/index.php?option=com_onlinejudge&Itemid=8&category=56&page=show_problem&problem=761)和[LA3031](https://icpcarchive.ecs.baylor.edu/index.php?option=com_onlinejudge&Itemid=8&category=192&page=show_problem&problem=1032)上测试通过，因数据量小速度与前几个差不多；点的编号默认从0开始。

HLPP很适合处理分层图的情况，在很多ISAP和Dinic会超时，或者需要加贪心预流的分层图题目中，HLPP可以直接过。

经过更多题目的测试，这个HLPP板子的速度比较让我失望，也许是板子中的大量使用的vector严重制约了算法的实际速度，比较闲的时候再来改写吧；目前来讲ISAP完全够用了。
{% highlight cpp linenos %}
struct FlowNetwork {
    int n,m,src,sink;
    vector<int> e[MAXN];
    struct Edge {
        int v,c;
    } edge[MAXM<<1];
    void init(int _n,int _src,int _sink) {
        n=_n;
        m=0;
        src=_src;
        sink=_sink;
        for(int i=0; i<n; ++i)
            e[i].clear();
    }
    void addEdge(int a,int b,int c,int rc=0) {
        edge[m].v=b;
        edge[m].c=c;
        e[a].push_back(m++);
        edge[m].v=a;
        edge[m].c=rc;
        e[b].push_back(m++);
    }
    int c[MAXN<<1],d[MAXN],w[MAXN],done[MAXN];
    void bfs() {
        queue<int> q;
        fill(c,c+(n<<1),0);
        c[n+1]=n-1;
        fill(d,d+n,n+1);
        d[src]=n;
        d[sink]=0;
        q.push(sink);
        while(!q.empty()) {
            int u=q.front();
            q.pop();
            --c[n+1];
            ++c[d[u]];
            for(int i=0; i<e[u].size(); ++i) {
                Edge &cra=edge[e[u][i]^1];
                int v=edge[e[u][i]].v;
                if(d[v]==n+1&&cra.c>0) {
                    d[v]=d[u]+1;
                    q.push(v);
                }
            }
        }
    }
    int hlpp() {
        vector<queue<int> > q(n<<1);
        vector<bool> mark(n,false);
        int todo=-1;
        bfs();
        mark[src]=mark[sink]=true;
        fill(w,w+n,0);
        for(int i=0; i<e[src].size(); ++i) {
            Edge &arc=edge[e[src][i]],&cra=edge[e[src][i]^1];
            int v=arc.v;
            w[v]+=arc.c;
            cra.c+=arc.c;
            arc.c=0;
            if(!mark[v]) {
                mark[v]=true;
                q[d[v]].push(v);
                todo=max(todo,d[v]);
            }
        }
        fill(done,done+n,0);
        while(todo>=0) {
            if(q[todo].empty()) {
                --todo;
                continue;
            }
            int u=q[todo].front();
            mark[u]=false;
            q[todo].pop();
            while(done[u]<(int)e[u].size()) {
                Edge &arc=edge[e[u][done[u]]];
                int v=arc.v;
                if(d[u]==d[v]+1&&arc.c>0) {
                    Edge &cra=edge[e[u][done[u]]^1];
                    int f=min(w[u],arc.c);
                    w[u]-=f;
                    w[v]+=f;
                    arc.c-=f;
                    cra.c+=f;
                    if(!mark[v]) {
                        mark[v]=true;
                        q[d[v]].push(v);
                    }
                    if(w[u]==0)
                        break;
                }
                ++done[u];
            }
            if(w[u]>0) {
                int du=d[u];
                --c[d[u]];
                d[u]=n<<1;
                for(int i=0; i<e[u].size(); ++i) {
                    Edge &arc=edge[e[u][i]];
                    int v=arc.v;
                    if(d[u]>d[v]+1&&arc.c>0) {
                        d[u]=d[v]+1;
                        done[u]=i;
                    }
                }
                ++c[d[u]];
                if(c[du]==0)
                    for(int i=0; i<n; ++i)
                        if(d[i]>du&&d[i]<n+1) {
                            --c[d[i]];
                            ++c[n+1];
                            d[i]=n+1;
                        }
                mark[u]=true;
                q[d[u]].push(u);
                todo=d[u];
            }
        }
        return w[sink];
    }
};
{% endhighlight %}

吐个槽：事实上我发现，网上各种最大流的资料相当地凌乱啊……且不说不能保证一定正确，而且在一些基本的定义和分类上也是各种乱套……