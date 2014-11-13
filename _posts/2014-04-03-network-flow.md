---
layout: post
title: 最短路的一些板子
date: 2014-04-03 15:11:00
category: acm
tags: acm
comments: true
---
起因是一场训练赛，[这道题（LA 5854 Long Distance Taxi）](https://icpcarchive.ecs.baylor.edu/index.php?option=com_onlinejudge&Itemid=8&page=show_problem&problem=3865)我和Thor拿出了正解（之一），但非常令人发指地送出了不计其数的TLE

题干不说了，我们的做法是前向星建图，然后每个LPG（起点终点也算LPG）跑一遍SPFA，然后所有LPG重新前向星建图，再跑一遍SPFA。复杂度大概是O(k1*n+k2*m^2)，按理说不会超时啊……

然后我们发现sd0061的做法前面和我们一样，后面建邻接矩阵，跑一遍Floyd，理论复杂度是O(k*n+m^3)；令人发指的是竟然比我们的程序快一倍……

于是我和Thor开始了大规模的研究。具体过程按下不表，其中重要的一点是，我们发现两次建图中第一次是稀疏图，第二次是稠密图。最终结论是：

- 对于稀疏图，优先使用前向星或邻接表+SPFA（为保证速度避免使用vector）；对于稠密图，优先使用邻接矩阵+Dijkstra。

- 算法上，设某图点数为V，边数为E。邻接矩阵+裸Dijkstra复杂度是O(V^2)，邻接表+Dijkstra+priority_queue的复杂度是O((V+E)logV)，速度上是最稳定的；Barty的板子是Dijkstra+heap，但heap用set去实现，复杂度相同但实测速度上要慢上一些。前向星+SPFA+SLF+LLL的理论复杂度依旧是最快的O(k*E)，但是常数k很不稳定，有退化成O(V*E)的可能；其中两个优化同时使用效果会好一些。

- 所以如果图是稠密图，那么用SPFA就是作死……但是如果图中有负权边，又必须放弃Dijkstra投奔SPFA的怀抱……如果是带负权边的稠密图，提刀砍向出题人吧……

- 此外从Bellman-Ford改过来的SPFA还可判负环，但判负环时不可以加优化。

目前Damocles已有的最短路板子速度上已经比较快，基本不会再改了。贴出来供参考。

建图：
{% highlight c++ linenos %}struct graph {
    int tot,head[MAXN];
    int to[MAXM],next[MAXM],len[MAXM];
    void init() {
        tot=0;
        memset(head,0xff,sizeof(head));
    }
    void add(int x,int y,int z) {
        to[tot]=y;
        len[tot]=z;
        next[tot]=head[x];
        head[x]=tot++;
    }
} g;{% endhighlight %}

Dijkstra+priority_queue+邻接表，邻接矩阵的在此基础上随便改改就成：
{% highlight c++ linenos %}int dist[MAXM];
void dijkstra(int src) {
    memset(dist,0x3f,sizeof(dist));
    dist[src]=0;
    priority_queue<pair<int,int>,vector<pair<int,int> >,greater<pair<int,int> > > pq;
    pq.push(make_pair(dist[src],src));
    while(!pq.empty()) {
        int u=pq.top().second,d=pq.top().first;
        pq.pop();
        if(d!=dist[u])
            continue;
        for(int i=g.head[u]; ~i; i=g.next[i]) {
            int v=g.to[i];
            if(d+g.len[i]<dist[v]) {
                dist[v]=d+g.len[i];
                pq.push(make_pair(dist[v],v));
            }
        }
    }
}{% endhighlight %}

SPFA+SLF+LLL+邻接表，这个就别改邻接矩阵了，太慢：
{% highlight c++ linenos %}int dist[MAXM];
bool inque[MAXM];
void spfa(int src) {
    memset(dist,0x3f,sizeof(dist));
    memset(inque,false,sizeof(inque));
    dist[src]=0;
    deque<int> q;
    q.push_back(src);
    inque[src]=true;
    long long sum=0;
    while(!q.empty()) {
        int u=q.front();
        q.pop_front();
        if(!q.empty()&&(long long)dist[u]*q.size()>sum) {
            q.push_back(u);
            continue;
        }
        sum-=dist[u];
        inque[u]=false;
        for(int i=g.head[u]; ~i; i=g.next[i]) {
            int v=g.to[i],d=dist[x]+g.len[i];
            if(d<dist[v]) {
                if(inque[v])
                    sum+=d-dist[v];
                dist[v]=d;
                if(!inque[v]) {
                    if(!q.empty()&&dist[q.front()]>dist[v])
                        q.push_front(v);
                    else
                        q.push_back(v);
                    sum+=dist[v];
                    inque[v]=true;
                }
            }
        }
    }
}{% endhighlight %}

基本上比赛能用上的就这两份板子了，根据情况改图或增减优化即可。通常来讲SPFA不加SLF和LLL优化也够用了。