---
layout: post
title: 最短路的一些板子
date: 2014-04-03 15:11:00
category: acm
tags: acm
comments: true
---
<p>
	<span style="font-family:Microsoft YaHei; font-size:12px">　　起因是一场训练赛，<a target="_blank" href="https://icpcarchive.ecs.baylor.edu/index.php?option=com_onlinejudge&amp;Itemid=8&amp;page=show_problem&amp;problem=3865">这道题（LA 5854 Long Distance Taxi）</a>我和Thor拿出了正解（之一），但非常令人发指地送出了不计其数的TLE。</span>
</p>
<p>
	<span style="font-family:Microsoft YaHei; font-size:12px">　　题干不说了，我们的做法是前向星建图，然后每个LPG（起点终点也算LPG）跑一遍SPFA，然后所有LPG重新前向星建图，再跑一遍SPFA。复杂度大概是O(k1*n+k2*m^2)，按理说不会超时啊……</span>
</p>
<p>
	<span style="font-family:Microsoft YaHei; font-size:12px">　　然后我们发现sd0061的做法前面和我们一样，后面建邻接矩阵，跑一遍Floyd，理论复杂度是O(k*n+m^3)；令人发指的是竟然比我们的程序快一倍……</span>
</p>
<p>
	<span style="font-family:Microsoft YaHei; font-size:12px">　　于是我和Thor开始了大规模的研究。具体过程按下不表，其中重要的一点是，我们发现两次建图中第一次是稀疏图，第二次是稠密图。最终结论是：</span>
</p>
<p>
	<span style="font-family:Microsoft YaHei; font-size:12px">　　对于稀疏图，优先使用前向星或邻接表+SPFA（为保证速度避免使用vector）；对于稠密图，优先使用邻接矩阵+Dijkstra。</span>
</p>
<p>
	<span style="font-family:Microsoft YaHei; font-size:12px">　　算法上，设某图点数为V，边数为E。邻接矩阵+裸Dijkstra复杂度是O(V^2)，邻接表+Dijkstra+priority_queue的复杂度是O((V+E)logV)，速度上是最稳定的；Barty的板子是Dijkstra+heap，但heap用set去实现，复杂度相同但实测速度上要慢上一些。前向星+SPFA+SLF+LLL的理论复杂度依旧是最快的O(k*E)，但是常数k很不稳定，有退化成O(V*E)的可能；其中两个优化同时使用效果会好一些。</span>
</p>
<p>
	<span style="font-family:Microsoft YaHei; font-size:12px">　　所以如果图是稠密图，那么用SPFA就是作死……但是如果图中有负权边，又必须放弃Dijkstra投奔SPFA的怀抱……如果是带负权边的稠密图，提刀砍向出题人吧……</span>
</p>
<p>
	<span style="font-family:Microsoft YaHei; font-size:12px">　　此外从Bellman-Ford改过来的SPFA还可判负环。</span>
</p>
<p>
	<span style="font-family:Microsoft YaHei; font-size:12px">　　目前Damocles已有的最短路板子速度上已经比较快，基本不会再改了。贴出来供参考。</span>
</p>
<p>
	<span style="font-family:Microsoft YaHei; font-size:12px">　　建图：</span>
</p>
<pre code_snippet_id="272456" snippet_file_name="blog_20140403_1_6765628" name="code" class="cpp">const int MAXN=100005;
const int MAXM=200005;
struct graph
{
    int head[MAXN];
    int to[MAXM];
    int next[MAXM];
    int len[MAXM];
    int tot;
    void init()
    {
        tot=0;
        memset(head,0xff,sizeof(head));
    }
    void add(int x,int y,int z)
    {
        to[tot]=y;
        len[tot]=z;
        next[tot]=head[x];
        head[x]=tot++;
    }
} g;</pre>
<p>
	<span style="font-family:Microsoft YaHei; font-size:12px">　　Dijkstra+priority_queue+邻接表，邻接矩阵的在此基础上随便改改就成：</span>
</p>
<pre code_snippet_id="272456" snippet_file_name="blog_20140403_2_3494789" name="code" class="cpp">int dist[MAXM];
void dijkstra(int src)
{
    memset(dist,0x3f,sizeof(dist));
    dist[src]=0;
    priority_queue&lt;pair&lt;int,int&gt;,vector&lt;pair&lt;int,int&gt; &gt;,greater&lt;pair&lt;int,int&gt; &gt; &gt; pq;
    pq.push(make_pair(dist[src],src));
    while(!pq.empty())
    {
        int x=pq.top().second;
        int d=pq.top().first;
        pq.pop();
        if(d!=dist[x])
            continue;
        for(int i=g.head[x]; ~i; i=g.next[i])
        {
            int y=g.to[i];
            if(d+g.len[i]&lt;dist[y])
            {
                dist[y]=d+g.len[i];
                pq.push(make_pair(dist[y],y));
            }
        }
    }
}</pre>
<p>
	<span style="font-family:Microsoft YaHei; font-size:12px">　　SPFA+SLF+LLL+邻接表，这个就别改邻接矩阵了，太慢：</span>
</p>
<pre code_snippet_id="272456" snippet_file_name="blog_20140403_4_1761092" name="code" class="cpp">int dist[MAXM];
bool inque[MAXM];
void spfa(int src)
{
    memset(dist,0x3f,sizeof(dist));
    memset(inque,false,sizeof(inque));
    dist[src]=0;
    deque&lt;int&gt; q;
    q.push_back(src);
    inque[src]=true;
    long long sum=0;
    while(!q.empty())
    {
        int x=q.front();
        q.pop_front();
        if(!q.empty()&amp;&amp;(long long)dist[x]*q.size()&gt;sum)
        {
            q.push_back(x);
            continue;
        }
        sum-=dist[x];
        inque[x]=false;
        for(int i=g.head[x]; ~i; i=g.next[i])
        {
            int y=g.to[i];
            int d=dist[x]+g.len[i];
            if(d&lt;dist[y])
            {
                if(inque[y])
                    sum+=d-dist[y];
                dist[y]=d;
                if(!inque[y])
                {
                    if(!q.empty()&amp;&amp;dist[q.front()]&gt;dist[y])
                        q.push_front(y);
                    else
                        q.push_back(y);
                    sum+=dist[y];
                    inque[y]=true;
                }
            }
        }
    }
}</pre>
<p>
	<span style="font-family:Microsoft YaHei; font-size:12px">　　基本上比赛能用上的就这两份板子了，根据情况改图或增减优化即可。通常来讲SPFA不加SLF和LLL优化也够用了。</span>
</p>