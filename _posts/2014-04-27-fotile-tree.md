---
layout: post
title: 主主主主主席树Orz
date: 2014-04-27 01:17:00
category: acm
tags: acm
comments: true
---
断断续续一个月，才稍稍搞明白一点主席树，真是累煞我也。不过从Damocles的分工来看，高级数据结构是我不得不啃的难关，所以只能慢慢来……

因为还有一些经典的题没做，日后做完了也会更新到这篇blog上来……

首先，学习主席树，我们是躲不开这几篇日志的：

- [Seter：1901: Zju2112 Dynamic Rankings](http://seter.is-programmer.com/posts/31907.html)
- [MetalSeed：主席树介绍](http://blog.csdn.net/metalseed/article/details/8045038)
- [始之星火：主席树UVA12003 + ZOJ2112](http://shizhixinghuo.diandian.com/post/2013-08-03/40052038468)

此外推荐范浩强的论文《wc2012谈谈各种数据结构》和陈立杰的论文《可持久化数据结构研究》，网上很多，自行Google去吧……

主席树，主体是可持久化线段树，或者说函数式线段树。线段树的修改不多说了，可持久化就是不在原节点上修改，而是针对每次修改建立一棵新的线段树，这样就做到了保存历史信息。换言之，主席树就是很多棵线段树。只是这样做无论时间还是空间复杂度都过高，我们注意到每一次修改都只是修改了树的一个分支，于是我们尽可能重复使用相同的部分，用建立新分支来代替建立新树，这样每次修改的时间复杂度就控制到了$$O(logn)$$，空间消耗依旧比较高，不过这是主席树固有的缺点吧。

主席树适用于区间第K大问题，以及一些需要查询区间历史信息的题目。还是结合经典例题吧，主席树真是可以变着花地使……

##[POJ2104](http://poj.org/problem?id=2104)、[POJ2761](http://poj.org/problem?id=2761)

我的主席树入门题，经典的无修改区间第K小。这两道题几乎一样，同样的代码无改动AC……

做法相当多啦，而且貌似被很多人拿来学习各种数据结构，各种平衡树+离线、线段树+二分答案$$O(mlog^3n)$$、归并树$$O(mlog^3n)$$、划分树$$O(mlogn)$$、主席树$$O(mlogn)$$……虽然这些我都不会不过不妨碍我学主席树……

离散化，建立权值线段树；这里权值线段树是指，每个节点维护权值是1到x的数的个数，换言之，每个节点维护了权值是x的数的个数的前缀和。然后由于线段树维护的信息显然支持区间减法，于是查询时只要拿出两棵线段树相减就行了。
{% highlight cpp linenos %}
#include<cstdio>
#include<algorithm>
using namespace std;
#define lson l,m,ls[rt]
#define rson m+1,r,rs[rt]
const int MAXN=100005;
int ls[MAXN<<5],rs[MAXN<<5],cnt[MAXN<<5],root[MAXN],tot;
void build(int l,int r,int &rt) {
    rt=tot++;
    cnt[rt]=0;
    if(l==r)
        return;
    int m=l+r>>1;
    build(lson);
    build(rson);
}
void update(int last,int p,int l,int r,int &rt) {
    rt=tot++;
    ls[rt]=ls[last];
    rs[rt]=rs[last];
    cnt[rt]=cnt[last]+1;
    if(l==r)
        return;
    int m=l+r>>1;
    if(p<=m)
        update(ls[last],p,lson);
    else
        update(rs[last],p,rson);
}
int query(int ss,int tt,int l,int r,int k) {
    if(l==r)
        return l;
    int m=l+r>>1,sum=cnt[ls[tt]]-cnt[ls[ss]];
    if(k<=sum)
        return query(ls[ss],ls[tt],l,m,k);
    else
        return query(rs[ss],rs[tt],m+1,r,k-sum);
}
int num[MAXN],hash[MAXN];
int main() {
    int n,m,l,r,k;
    while(~scanf("%d%d",&n,&m)) {
        for(int i=1; i<=n; ++i) {
            scanf("%d",&num[i]);
            hash[i]=num[i];
        }
        sort(hash+1,hash+n+1);
        int size=unique(hash+1,hash+n+1)-hash-1;
        for(int i=1; i<=n; ++i)
            num[i]=lower_bound(hash+1,hash+size+1,num[i])-hash;
        tot=0;
        build(1,size,root[0]);
        for(int i=1; i<=n; ++i)
            update(root[i-1],num[i],1,size,root[i]);
        while(m--) {
            scanf("%d%d%d",&l,&r,&k);
            printf("%d\n",hash[query(root[l-1],root[r],1,size,k)]);
        }
    }
}
{% endhighlight %}

##[HDU4417](http://acm.hdu.edu.cn/showproblem.php?pid=4417)

区间第K小一般是查询某个区间里第K小的数是多少，这道题恰好反过来，查询在某个区间有多少个数不大于h，依旧无修改操作。基本只是在query部分稍作改动，稍微注意一下输入和离散化部分，对比上一道题还是很好理解的。
{% highlight cpp linenos %}
#include<cstdio>
#include<algorithm>
using namespace std;
#define lson l,m,ls[rt]
#define rson m+1,r,rs[rt]
const int MAXN=100005;
int ls[MAXN<<5],rs[MAXN<<5],cnt[MAXN<<5],root[MAXN],tot;
void build(int l,int r,int &rt) {
    rt=tot++;
    cnt[rt]=0;
    if(l==r)
        return;
    int m=l+r>>1;
    build(lson);
    build(rson);
}
void update(int last,int p,int l,int r,int &rt) {
    rt=tot++;
    ls[rt]=ls[last];
    rs[rt]=rs[last];
    cnt[rt]=cnt[last]+1;
    if(l==r)
        return;
    int m=l+r>>1;
    if(p<=m)
        update(ls[last],p,lson);
    else
        update(rs[last],p,rson);
}
int query(int ss,int tt,int l,int r,int x) {
    if(l==r)
        return x<l?0:cnt[tt]-cnt[ss];
    int m=l+r>>1;
    if(x<=m)
        return query(ls[ss],ls[tt],l,m,x);
    else
        return cnt[ls[tt]]-cnt[ls[ss]]+query(rs[ss],rs[tt],m+1,r,x);
}
int num[MAXN],hash[MAXN];
int main() {
    int t,n,m,l,r,h;
    scanf("%d",&t);
    for(int cas=1; cas<=t; ++cas) {
        scanf("%d%d",&n,&m);
        for(int i=1; i<=n; ++i) {
            scanf("%d",&num[i]);
            hash[i]=num[i];
        }
        sort(hash+1,hash+n+1);
        int size=unique(hash+1,hash+n+1)-hash-1;
        for(int i=1; i<=n; ++i)
            num[i]=lower_bound(hash+1,hash+size+1,num[i])-hash;
        tot=0;
        build(1,size,root[0]);
        for(int i=1; i<=n; ++i)
            update(root[i-1],num[i],1,size,root[i]);
        printf("Case %d:\n",cas);
        while(m--) {
            scanf("%d%d%d",&l,&r,&h);
            h=upper_bound(hash+1,hash+size+1,h)-hash-1;
            printf("%d\n",query(root[l],root[r+1],1,size,h));
        }
    }
}
{% endhighlight %}

此外[SPOJ3266](http://www.spoj.com/problems/KQUERY/)是求某个区间有多少个数大于k，与这道题恰好相反，改一两处细节就可以了。因为这道题的时限太过严苛，我加了IO优化依然只能过掉前几个点，但做法应该是正确的，理论复杂度也没问题；有些人是用树状数组离线做的，我就不再尝试了。

##[BZOJ1901](http://www.lydsy.com/JudgeOnline/problem.php?id=1901)

可修改区间第K小。绝对是一道坎，这个地方卡住我很长时间。因为以前没有写过树套树，加上过掉POJ那两题时比较运气，对主席树还没太理解，所以一开始不知所措。

无修改时，主席树的每个结点维护了权值前缀和，这里因为要进行修改，所以把这个工作分拆开，维护前缀和交给树状数组。所以最后是树状数组套主席树。

联系到之前POJ2104的代码，建树时每一次update都是在前一个root的基础上建立的，所以这次直接在对应root上建立，这样建好的主席树就不会维护到前缀和，然后用树状数组把root都维护起来就可以了；只是这样做很占内存，过不掉ZOJ2112。这样每次修改和查询都是树状数组$$O(logn)$$，主席树$$O(logn)$$，总复杂度$$O(mlog^2n)$$。query操作写了两种，对照着看对理解有一定帮助；先是非递归的。
{% highlight cpp linenos %}
#include<cstdio>
#include<algorithm>
using namespace std;
#define lson l,m,ls[rt]
#define rson m+1,r,rs[rt]
const int MAXN=60005;
const int MAXM=2500005;
int n,m,size;
int ls[MAXM],rs[MAXM],cnt[MAXM],root[MAXN],tot;
void build(int l,int r,int &rt) {
    rt=tot++;
    cnt[rt]=0;
    if(l==r)
        return;
    int m=l+r>>1;
    build(lson);
    build(rson);
}
void update(int last,int p,int val,int l,int r,int &rt) {
    rt=tot++;
    ls[rt]=ls[last];
    rs[rt]=rs[last];
    cnt[rt]=cnt[last]+val;
    if(l==r)
        return;
    int m=l+r>>1;
    if(p<=m)
        update(ls[last],p,val,lson);
    else
        update(rs[last],p,val,rson);
}
int use[MAXN];
inline int lowbit(int x) {
    return x&-x;
}
void modify(int x,int p,int val) {
    for(int i=x; i<=n; i+=lowbit(i))
        update(root[i],p,val,1,size,root[i]);
}
int sum(int x) {
    int ret=0;
    for(int i=x; i>0; i-=lowbit(i))
        ret+=cnt[ls[use[i]]];
    return ret;
}
int query(int ss,int tt,int l,int r,int k) {
    for(int i=ss; i>0; i-=lowbit(i))
        use[i]=root[i];
    for(int i=tt; i>0; i-=lowbit(i))
        use[i]=root[i];
    while(l<r) {
        int m=l+r>>1,tmp=sum(tt)-sum(ss);
        if(k<=tmp) {
            r=m;
            for(int i=ss; i>0; i-=lowbit(i))
                use[i]=ls[use[i]];
            for(int i=tt; i>0; i-=lowbit(i))
                use[i]=ls[use[i]];
        } else {
            l=m+1;
            k-=tmp;
            for(int i=ss; i>0; i-=lowbit(i))
                use[i]=rs[use[i]];
            for(int i=tt; i>0; i-=lowbit(i))
                use[i]=rs[use[i]];
        }
    }
    return l;
}
int num[MAXN],hash[MAXN],l[MAXN],r[MAXN],k[MAXN];
char op[MAXN];
int main() {
    while(~scanf("%d%d",&n,&m)) {
        size=0;
        for(int i=1; i<=n; ++i) {
            scanf("%d",&num[i]);
            hash[++size]=num[i];
        }
        for(int i=0; i<m; ++i) {
            scanf(" %c%d%d",&op[i],&l[i],&r[i]);
            switch(op[i]) {
            case 'Q':
                scanf("%d",&k[i]);
                break;
            case 'C':
                hash[++size]=r[i];
                break;
            }
        }
        sort(hash+1,hash+size+1);
        size=unique(hash+1,hash+size+1)-hash-1;
        for(int i=1; i<=n; ++i)
            num[i]=lower_bound(hash+1,hash+size+1,num[i])-hash;
        tot=0;
        build(1,size,root[0]);
        for(int i=1; i<=n; ++i)
            root[i]=root[0];
        for(int i=1; i<=n; ++i)
            modify(i,num[i],1);
        for(int i=0; i<m; ++i)
            switch(op[i]) {
            case 'Q':
                printf("%d\n",hash[query(l[i]-1,r[i],1,size,k[i])]);
                break;
            case 'C':
                modify(l[i],num[l[i]],-1);
                num[l[i]]=lower_bound(hash+1,hash+size+1,r[i])-hash;
                modify(l[i],num[l[i]],1);
                break;
            }
    }
}
{% endhighlight %}

然后是query操作递归的版本，注意query前对树状数组的初始化。
{% highlight cpp linenos %}
#include<cstdio>
#include<algorithm>
using namespace std;
#define lson l,m,ls[rt]
#define rson m+1,r,rs[rt]
const int MAXN=60005;
const int MAXM=2500005;
int n,m,size;
int ls[MAXM],rs[MAXM],cnt[MAXM],root[MAXN],tot;
void build(int l,int r,int &rt) {
    rt=tot++;
    cnt[rt]=0;
    if(l==r)
        return;
    int m=l+r>>1;
    build(lson);
    build(rson);
}
void update(int last,int p,int val,int l,int r,int &rt) {
    rt=tot++;
    ls[rt]=ls[last];
    rs[rt]=rs[last];
    cnt[rt]=cnt[last]+val;
    if(l==r)
        return;
    int m=l+r>>1;
    if(p<=m)
        update(ls[last],p,val,lson);
    else
        update(rs[last],p,val,rson);
}
int use[MAXN];
inline int lowbit(int x) {
    return x&-x;
}
void modify(int x,int p,int val) {
    for(int i=x; i<=n; i+=lowbit(i))
        update(root[i],p,val,1,size,root[i]);
}
int sum(int x) {
    int ret=0;
    for(int i=x; i>0; i-=lowbit(i))
        ret+=cnt[ls[use[i]]];
    return ret;
}
int query(int ss,int tt,int l,int r,int k) {
    if(l==r)
        return l;
    int m=l+r>>1,tmp=sum(tt)-sum(ss);
    if(k<=tmp) {
        for(int i=ss; i>0; i-=lowbit(i))
            use[i]=ls[use[i]];
        for(int i=tt; i>0; i-=lowbit(i))
            use[i]=ls[use[i]];
        return query(ss,tt,l,m,k);
    }
    else {
        for(int i=ss; i>0; i-=lowbit(i))
            use[i]=rs[use[i]];
        for(int i=tt; i>0; i-=lowbit(i))
            use[i]=rs[use[i]];
        return query(ss,tt,m+1,r,k-tmp);
    }
}
int num[MAXN],hash[MAXN],l[MAXN],r[MAXN],k[MAXN];
char op[MAXN];
int main() {
    while(~scanf("%d%d",&n,&m)) {
        size=0;
        for(int i=1; i<=n; ++i) {
            scanf("%d",&num[i]);
            hash[++size]=num[i];
        }
        for(int i=0; i<m; ++i) {
            scanf(" %c%d%d",&op[i],&l[i],&r[i]);
            switch(op[i]) {
            case 'Q':
                scanf("%d",&k[i]);
                break;
            case 'C':
                hash[++size]=r[i];
                break;
            }
        }
        sort(hash+1,hash+size+1);
        size=unique(hash+1,hash+size+1)-hash-1;
        for(int i=1; i<=n; ++i)
            num[i]=lower_bound(hash+1,hash+size+1,num[i])-hash;
        tot=0;
        build(1,size,root[0]);
        for(int i=1; i<=n; ++i)
            root[i]=root[0];
        for(int i=1; i<=n; ++i)
            modify(i,num[i],1);
        for(int i=0; i<m; ++i)
            switch(op[i]) {
            case 'Q':
                for(int j=l[i]-1; j>0; j-=lowbit(j))
                    use[j]=root[j];
                for(int j=r[i]; j>0; j-=lowbit(j))
                    use[j]=root[j];
                printf("%d\n",hash[query(l[i]-1,r[i],1,size,k[i])]);
                break;
            case 'C':
                modify(l[i],num[l[i]],-1);
                num[l[i]]=lower_bound(hash+1,hash+size+1,r[i])-hash;
                modify(l[i],num[l[i]],1);
                break;
            }
    }
}
{% endhighlight %}

##[ZOJ2112](http://acm.zju.edu.cn/onlinejudge/showProblem.do?problemCode=2112)

和BZOJ1901题意几乎一致，但是数据量大了许多，内存限制反而缩小了不少……在这样令人发指的严苛要求下，上面的代码交上去会毫不犹豫地RE，把数组加大又会轻易MLE。所以必须寻找缩减内存占用的方法。

查了些资料，后来意识到可以把整个问题分成两部分。首先按照无修改的方式建主席树，然后把修改的情况用树状数组套主席树维护起来；即修改过程相当于建一棵空树，在其上进行修改，因为区间减法的存在，查询的时候别忘记带上初始的树上的值就好了。这样做，查询部分与之前区别不大，但建树部分没搞错的话，时间和空间上都少了一个$$O(logn)$$。

这个地方很多日志尝试说清也没能做到，以至于我在这个地方耽搁了很久；我感觉我也不能说得很好。还是直接上代码吧，对比一下建树部分和query部分的区别，应该会好理解很多。

顺便这道题的代码改下输入方式就可过掉BZOJ1901，而且可以根据数据量把空间改小一点。这两道题还可以用线段树套平衡树$$O(mlog^2n)$$来搞，也留着以后学Treap和Splay时用；甚至还有块状链表+二分O(m*sqrt(n)*log(sqrt(n))这样神奇的存在，不过这个就不是很想学了……
{% highlight cpp linenos %}
#include<cstdio>
#include<algorithm>
using namespace std;
#define lson l,m,ls[rt]
#define rson m+1,r,rs[rt]
const int MAXN=60005;
const int MAXM=2500005;
int t,n,m,size;
int ls[MAXM],rs[MAXM],cnt[MAXM],root[MAXN],tot;
void build(int l,int r,int &rt) {
    rt=tot++;
    cnt[rt]=0;
    if(l==r)
        return;
    int m=l+r>>1;
    build(lson);
    build(rson);
}
void update(int last,int p,int val,int l,int r,int &rt) {
    rt=tot++;
    ls[rt]=ls[last];
    rs[rt]=rs[last];
    cnt[rt]=cnt[last]+val;
    if(l==r)
        return;
    int m=l+r>>1;
    if(p<=m)
        update(ls[last],p,val,lson);
    else
        update(rs[last],p,val,rson);
}
int tree[MAXN],use[MAXN];
inline int lowbit(int x) {
    return x&-x;
}
void modify(int x,int p,int val) {
    for(int i=x; i<=n; i+=lowbit(i))
        update(tree[i],p,val,1,size,tree[i]);
}
int sum(int x) {
    int ret=0;
    for(int i=x; i>0; i-=lowbit(i))
        ret+=cnt[ls[use[i]]];
    return ret;
}
int query(int ss,int tt,int l,int r,int k) {
    for(int i=ss; i>0; i-=lowbit(i))
        use[i]=tree[i];
    for(int i=tt; i>0; i-=lowbit(i))
        use[i]=tree[i];
    int sr=root[ss],tr=root[tt];
    while(l<r) {
        int m=l+r>>1,tmp=sum(tt)-sum(ss)+cnt[ls[tr]]-cnt[ls[sr]];
        if(k<=tmp) {
            r=m;
            for(int i=ss; i>0; i-=lowbit(i))
                use[i]=ls[use[i]];
            for(int i=tt; i>0; i-=lowbit(i))
                use[i]=ls[use[i]];
            sr=ls[sr];
            tr=ls[tr];
        } else {
            l=m+1;
            k-=tmp;
            for(int i=ss; i>0; i-=lowbit(i))
                use[i]=rs[use[i]];
            for(int i=tt; i>0; i-=lowbit(i))
                use[i]=rs[use[i]];
            sr=rs[sr];
            tr=rs[tr];
        }
    }
    return l;
}
int num[MAXN],hash[MAXN],l[MAXN],r[MAXN],k[MAXN];
char op[MAXN];
int main() {
    scanf("%d",&t);
    while(t--) {
        scanf("%d%d",&n,&m);
        size=0;
        for(int i=1; i<=n; ++i) {
            scanf("%d",&num[i]);
            hash[++size]=num[i];
        }
        for(int i=0; i<m; ++i) {
            scanf(" %c%d%d",&op[i],&l[i],&r[i]);
            switch(op[i]) {
            case 'Q':
                scanf("%d",&k[i]);
                break;
            case 'C':
                hash[++size]=r[i];
                break;
            }
        }
        sort(hash+1,hash+size+1);
        size=unique(hash+1,hash+size+1)-hash-1;
        for(int i=1; i<=n; ++i)
            num[i]=lower_bound(hash+1,hash+size+1,num[i])-hash;
        tot=0;
        build(1,size,root[0]);
        for(int i=1; i<=n; ++i)
            update(root[i-1],num[i],1,1,size,root[i]);
        for(int i=1; i<=n; ++i)
            tree[i]=root[0];
        for(int i=0; i<m; ++i)
            switch(op[i]) {
            case 'Q':
                printf("%d\n",hash[query(l[i]-1,r[i],1,size,k[i])]);
                break;
            case 'C':
                modify(l[i],num[l[i]],-1);
                num[l[i]]=lower_bound(hash+1,hash+size+1,r[i])-hash;
                modify(l[i],num[l[i]],1);
                break;
            }
    }
}
{% endhighlight %}

##[SPOJ3267](http://www.spoj.com/problems/DQUERY/)

查询某个区间[L,R]里有多少个不同的数，无修改。用主席树维护pre[num[x]]，表示x位置的数（即num[x]）左边最近且相等的位置；这样就变成了询问区间[L,R]内有多少个数小于L。
{% highlight cpp linenos %}
#include<cstdio>
#include<cstring>
#include<algorithm>
using namespace std;
#define lson l,m,ls[rt]
#define rson m+1,r,rs[rt]
const int MAXN=30005;
const int MAXM=1000005;
int ls[MAXN<<5],rs[MAXN<<5],cnt[MAXN<<5],root[MAXN],tot;
void build(int l,int r,int &rt) {
    rt=tot++;
    cnt[rt]=0;
    if(l==r)
        return;
    int m=l+r>>1;
    build(lson);
    build(rson);
}
void update(int last,int p,int val,int l,int r,int &rt) {
    rt=tot++;
    ls[rt]=ls[last];
    rs[rt]=rs[last];
    cnt[rt]=cnt[last]+val;
    if(l==r)
        return;
    int m=l+r>>1;
    if(p<=m)
        update(ls[last],p,val,lson);
    else
        update(rs[last],p,val,rson);
}
int query(int ss,int tt,int l,int r,int x) {
    if(l==r)
        return x<l?0:cnt[tt]-cnt[ss];
    int m=l+r>>1;
    if(x<=m)
        return query(ls[ss],ls[tt],l,m,x);
    else
        return cnt[ls[tt]]-cnt[ls[ss]]+query(rs[ss],rs[tt],m+1,r,x);
}
int pre[MAXM];
int main() {
    int n,q,l,r,num;
    while(~scanf("%d",&n)) {
        memset(pre,0,sizeof(pre));
        tot=0;
        build(0,n,root[0]);
        for(int i=1; i<=n; ++i) {
            scanf("%d",&num);
            update(root[i-1],pre[num],1,0,n,root[i]);
            pre[num]=i;
        }
        scanf("%d",&q);
        while(q--) {
            scanf("%d%d",&l,&r);
            printf("%d\n",query(root[l-1],root[r],0,n,l-1));
        }
    }
}
{% endhighlight %}
##[UVa12345](http://uva.onlinejudge.org/index.php?option=com_onlinejudge&Itemid=8&page=show_problem&problem=3767)

上一题的升级版，增加了单点修改的操作，故而实际上是zoj2112和spoj3267的综合；用pre[i]表示第i位左边与其相同的最近的坐标，可以比较容易地用set来更新，再用树状数组套主席树维护即可。写这道题时历尽艰辛，主要是因为之前几道题虽然写了出来，但是理解很不到位；现在回头看一开始的代码，写得跟屎一样。我会把一些题用更好的姿势重写，然后再更新上来。
{% highlight cpp linenos %}
#include<cstdio>
#include<cstring>
#include<algorithm>
#include<set>
using namespace std;
#define lson l,m,ls[rt]
#define rson m+1,r,rs[rt]
const int MAXN=50005;
const int MAXM=20000005;
const int MAXK=1000005;
int ls[MAXM],rs[MAXM],cnt[MAXM],root[MAXN],tot;
void build(int l,int r,int &rt) {
    rt=tot++;
    cnt[rt]=0;
    if(l==r)
        return;
    int m=l+r>>1;
    build(lson);
    build(rson);
}
void update(int last,int p,int val,int l,int r,int &rt) {
    rt=tot++;
    ls[rt]=ls[last];
    rs[rt]=rs[last];
    cnt[rt]=cnt[last]+val;
    if(l==r)
        return;
    int m=l+r>>1;
    if(p<=m)
        update(ls[last],p,val,lson);
    else
        update(rs[last],p,val,rson);
}
int query(int x,int l,int r,int rt) {
    if(l==r)
        return x<l?0:cnt[rt];
    int m=l+r>>1;
    if(x<=m)
        return query(x,lson);
    else
        return cnt[ls[rt]]+query(x,rson);
}
int n,m,num[MAXN],pre[MAXN],l,r,tree[MAXN];
char op;
set<int> bst[MAXK];
set<int>::iterator it;
inline int lowbit(int x) {
    return x&-x;
}
int query(int l,int r) {
    int ret=query(l-1,0,n,root[r])-query(l-1,0,n,root[l-1]);
    for(int i=r; i>0; i-=lowbit(i))
        ret+=query(l-1,0,n,tree[i]);
    for(int i=l-1; i>0; i-=lowbit(i))
        ret-=query(l-1,0,n,tree[i]);
    return ret;
}
void link(int x,int newpre) {
    for(int i=x; i<=n; i+=lowbit(i)) {
        update(tree[i],pre[x],-1,0,n,tree[i]);
        update(tree[i],newpre,1,0,n,tree[i]);
    }
    pre[x]=newpre;
}
void modify(int x,int val) {
    if(num[x]==val)
        return;
    bst[num[x]].erase(x);
    it=bst[num[x]].lower_bound(x);
    if(it!=bst[num[x]].end())
        link(*it,pre[x]);
    it=bst[val].lower_bound(x);
    if(it==bst[val].begin())
        link(x,0);
    else
        link(x,*(--bst[val].lower_bound(x)));
    if(it!=bst[val].end())
        link(*it,x);
    bst[val].insert(x);
    num[x]=val;
}
int main() {
    while(~scanf("%d%d",&n,&m)) {
        memset(pre,0,sizeof(pre));
        for(int i=0; i<MAXK; ++i)
            bst[i].clear();
        tot=0;
        build(0,n,root[0]);
        for(int i=1; i<=n; ++i) {
            scanf("%d",&num[i]);
            it=bst[num[i]].lower_bound(i);
            if(it!=bst[num[i]].begin())
                pre[i]=*(--it);
            bst[num[i]].insert(i);
            update(root[i-1],pre[i],1,0,n,root[i]);
        }
        for(int i=1; i<=n; ++i)
            tree[i]=root[0];
        while(m--) {
            scanf(" %c%d%d",&op,&l,&r);
            switch(op) {
            case 'Q':
                printf("%d\n",query(l+1,r));
                break;
            case 'M':
                modify(l+1,r);
                break;
            }
        }
    }
}
{% endhighlight %}

##[SPOJ11470](http://www.spoj.com/problems/TTM/)

四种操作，区间修改，当前区间查询，历史区间查询，回到历史时间。朴素的办法就是只建一棵线段树，记录下历史事件，在进行历史操作的时候倒着做回去……据说不会超时，估计是数据不够强= =不打算去试了。也可以可持久化树状数组来搞，以后再说。这道题里主席树就是单纯的可持久化线段树，可以做到$$O(logn)$$修改+$$O(logn)$$查询+$$O(1)$$历史回溯；要注意的是因为lazy操作的存在，push_down的时候子节点可能与当前节点不是一个历史版本，所以还需要再加一个标记来记录，如果是同一个历史版本则直接修改，否则新建节点再修改；这个地方卡了自己一段时间。

不得不说，主席树虽然很神，但是当真吃内存……目前的做法是搞不定HDU4348的，各种MLE……
{% highlight cpp linenos %}
#include<cstdio>
using namespace std;
#define lson l,m,ls[rt]
#define rson m+1,r,rs[rt]
const int MAXN=100005;
const int MAXM=8000005;
int ls[MAXM],rs[MAXM],col[MAXM],root[MAXN],tot;
bool mark[MAXM];
long long sum[MAXM];
void push_up(int rt) {
    sum[rt]=sum[ls[rt]]+sum[rs[rt]];
}
void build(int l,int r,int &rt) {
    rt=tot++;
    col[rt]=0;
    mark[rt]=false;
    if(l==r) {
        scanf("%lld",&sum[rt]);
        return;
    }
    int m=l+r>>1;
    build(lson);
    build(rson);
    push_up(rt);
}
void push_down(int rt,int l,int r) {
    if(mark[rt]) {
        col[tot]=col[ls[rt]];
        sum[tot]=sum[ls[rt]];
        ls[tot]=ls[ls[rt]];
        rs[tot]=rs[ls[rt]];
        ls[rt]=tot++;
        col[tot]=col[rs[rt]];
        sum[tot]=sum[rs[rt]];
        ls[tot]=ls[rs[rt]];
        rs[tot]=rs[rs[rt]];
        rs[rt]=tot++;
        mark[ls[rt]]=mark[rs[rt]]=true;
        mark[rt]=false;
    }
    if(col[rt]) {
        int m=l+r>>1;
        col[ls[rt]]+=col[rt];
        col[rs[rt]]+=col[rt];
        sum[ls[rt]]+=(long long)col[rt]*(m-l+1);
        sum[rs[rt]]+=(long long)col[rt]*(r-m);
        col[rt]=0;
    }
}
void update(int last,int L,int R,int val,int l,int r,int &rt) {
    rt=tot++;
    if(L<=l&&r<=R) {
        col[rt]=col[last]+val;
        sum[rt]=sum[last]+(long long)val*(r-l+1);
        if(L!=R) {
            ls[rt]=ls[last];
            rs[rt]=rs[last];
            mark[rt]=true;
        }
        return;
    }
    sum[rt]=col[rt]=0;
    mark[rt]=false;
    push_down(last,l,r);
    int m=l+r>>1;
    if(L<=m)
        update(ls[last],L,R,val,lson);
    else
        ls[rt]=ls[last];
    if(m<R)
        update(rs[last],L,R,val,rson);
    else
        rs[rt]=rs[last];
    push_up(rt);
}
long long query(int L,int R,int l,int r,int rt) {
    if(L<=l&&r<=R)
        return sum[rt];
    push_down(rt,l,r);
    int m=l+r>>1;
    long long ret=0;
    if(L<=m)
        ret+=query(L,R,lson);
    if(m<R)
        ret+=query(L,R,rson);
    return ret;
}
int main() {
    int n,m,l,r,d,h;
    char op;
    while(~scanf("%d%d",&n,&m)) {
        tot=h=0;
        build(1,n,root[h]);
        while(m--) {
            scanf(" %c",&op);
            switch(op) {
            case 'C':
                scanf("%d%d%d",&l,&r,&d);
                update(root[h],l,r,d,1,n,root[h+1]);
                ++h;
                break;
            case 'Q':
                scanf("%d%d",&l,&r);
                printf("%lld\n",query(l,r,1,n,root[h]));
                break;
            case 'H':
                scanf("%d%d%d",&l,&r,&d);
                printf("%lld\n",query(l,r,1,n,root[d]));
                break;
            case 'B':
                scanf("%d",&h);
                break;
            }
        }
    }
}
{% endhighlight %}

后来在想，新标记占用的空间也很大，那我干脆不加新标记，每次push_down都新建节点，看看如何？

结果发现完全可行，甚至比上面的内存消耗要少上那么一点，白白兜了个圈子真是无语……不过在HDU4348上这依然是远远不够的骚年……
{% highlight cpp linenos %}
#include<cstdio>
using namespace std;
#define lson l,m,ls[rt]
#define rson m+1,r,rs[rt]
const int MAXN=100005;
const int MAXM=8000005;
int ls[MAXM],rs[MAXM],col[MAXM],root[MAXN],tot;
long long sum[MAXM];
void push_up(int rt) {
    sum[rt]=sum[ls[rt]]+sum[rs[rt]];
}
void build(int l,int r,int &rt) {
    rt=tot++;
    col[rt]=0;
    if(l==r) {
        scanf("%lld",&sum[rt]);
        return;
    }
    int m=l+r>>1;
    build(lson);
    build(rson);
    push_up(rt);
}
void push_down(int rt,int l,int r) {
    if(col[rt]) {
        col[tot]=col[ls[rt]];
        sum[tot]=sum[ls[rt]];
        ls[tot]=ls[ls[rt]];
        rs[tot]=rs[ls[rt]];
        ls[rt]=tot++;
        col[tot]=col[rs[rt]];
        sum[tot]=sum[rs[rt]];
        ls[tot]=ls[rs[rt]];
        rs[tot]=rs[rs[rt]];
        rs[rt]=tot++;
        int m=l+r>>1;
        col[ls[rt]]+=col[rt];
        col[rs[rt]]+=col[rt];
        sum[ls[rt]]+=(long long)col[rt]*(m-l+1);
        sum[rs[rt]]+=(long long)col[rt]*(r-m);
        col[rt]=0;
    }
}
void update(int last,int L,int R,int val,int l,int r,int &rt) {
    rt=tot++;
    if(L<=l&&r<=R) {
        col[rt]=col[last]+val;
        sum[rt]=sum[last]+(long long)val*(r-l+1);
        if(L!=R) {
            ls[rt]=ls[last];
            rs[rt]=rs[last];
        }
        return;
    }
    sum[rt]=col[rt]=0;
    push_down(last,l,r);
    int m=l+r>>1;
    if(L<=m)
        update(ls[last],L,R,val,lson);
    else
        ls[rt]=ls[last];
    if(m<R)
        update(rs[last],L,R,val,rson);
    else
        rs[rt]=rs[last];
    push_up(rt);
}
long long query(int L,int R,int l,int r,int rt) {
    if(L<=l&&r<=R)
        return sum[rt];
    push_down(rt,l,r);
    int m=l+r>>1;
    long long ret=0;
    if(L<=m)
        ret+=query(L,R,lson);
    if(m<R)
        ret+=query(L,R,rson);
    return ret;
}
int main() {
    int n,m,l,r,d,h;
    char op;
    while(~scanf("%d%d",&n,&m)) {
        tot=h=0;
        build(1,n,root[h]);
        while(m--) {
            scanf(" %c",&op);
            switch(op) {
            case 'C':
                scanf("%d%d%d",&l,&r,&d);
                update(root[h],l,r,d,1,n,root[h+1]);
                ++h;
                break;
            case 'Q':
                scanf("%d%d",&l,&r);
                printf("%lld\n",query(l,r,1,n,root[h]));
                break;
            case 'H':
                scanf("%d%d%d",&l,&r,&d);
                printf("%lld\n",query(l,r,1,n,root[d]));
                break;
            case 'B':
                scanf("%d",&h);
                break;
            }
        }
    }
}
{% endhighlight %}

##[HDU4348](http://acm.hdu.edu.cn/showproblem.php?pid=4348)

首先YM一下张昆玮大神的讲稿《统计的力量-线段树》，虽然我搞不懂也不打算学ZKW线段树（非递归太神，但在线段树上我觉得意义不大），但如果没有这篇讲稿的基础，我也完全没有办法应对这道题变态的空间限制。

在上面SPOJ11470的两种实现中我们已经知道，每次push_down操作都要新建节点是内存消耗的主要原因，那能不能减少这部分的消耗？讲稿中介绍了线段树成段更新时节约内存的另一种办法，那就是改变lazy标记的意义，使其根本不进行push_down……具体的自行Google讲稿吧，大概写在第49~50页，实在有些神。

另外在SPOJ11470上也测了一下，空间是上面两种实现的三分之一，耗时是上面的一半略多的样子，两个方面都完爆lazy标记的传统更新方式。再次向张昆玮大神Orz……

update和query分别写了两种风格，可以互换。前一种是我习惯的，和上面的类似，但似乎后一种更好理解一些。
{% highlight cpp linenos %}
#include<cstdio>
#include<algorithm>
using namespace std;
#define lson l,m,ls[rt]
#define rson m+1,r,rs[rt]
const int MAXN=100005;
const int MAXM=2500005;
int ls[MAXM],rs[MAXM],col[MAXM],root[MAXN],tot;
long long sum[MAXM];
void push_up(int rt) {
    sum[rt]=sum[ls[rt]]+sum[rs[rt]];
}
void build(int l,int r,int &rt) {
    rt=tot++;
    col[rt]=0;
    if(l==r) {
        scanf("%I64d",&sum[rt]);
        return;
    }
    int m=l+r>>1;
    build(lson);
    build(rson);
    push_up(rt);
}
void update(int last,int L,int R,int val,int l,int r,int &rt) {
    rt=tot++;
    col[rt]=col[last];
    sum[rt]=sum[last]+(long long)val*(min(R,r)-max(L,l)+1);
    if(L<=l&&r<=R) {
        col[rt]+=val;
        if(L!=R) {
            ls[rt]=ls[last];
            rs[rt]=rs[last];
        }
        return;
    }
    int m=l+r>>1;
    if(L<=m)
        update(ls[last],L,R,val,lson);
    else
        ls[rt]=ls[last];
    if(m<R)
        update(rs[last],L,R,val,rson);
    else
        rs[rt]=rs[last];
}
long long query(int L,int R,int l,int r,int rt) {
    if(L<=l&&r<=R)
        return sum[rt];
    long long ret=(long long)col[rt]*(min(R,r)-max(L,l)+1);
    int m=l+r>>1;
    if(L<=m)
        ret+=query(L,R,lson);
    if(m<R)
        ret+=query(L,R,rson);
    return ret;
}
int main() {
    int n,m,l,r,d,h;
    char op;
    while(~scanf("%d%d",&n,&m)) {
        tot=h=0;
        build(1,n,root[h]);
        while(m--) {
            scanf(" %c",&op);
            switch(op) {
            case 'C':
                scanf("%d%d%d",&l,&r,&d);
                update(root[h],l,r,d,1,n,root[h+1]);
                ++h;
                break;
            case 'Q':
                scanf("%d%d",&l,&r);
                printf("%I64d\n",query(l,r,1,n,root[h]));
                break;
            case 'H':
                scanf("%d%d%d",&l,&r,&d);
                printf("%I64d\n",query(l,r,1,n,root[d]));
                break;
            case 'B':
                scanf("%d",&h);
                break;
            }
        }
    }
}
{% endhighlight %}

{% highlight cpp linenos %}
#include<cstdio>
using namespace std;
#define lson l,m,ls[rt]
#define rson m+1,r,rs[rt]
const int MAXN=100005;
const int MAXM=2500005;
int ls[MAXM],rs[MAXM],col[MAXM],root[MAXN],tot;
long long sum[MAXM];
void push_up(int rt) {
    sum[rt]=sum[ls[rt]]+sum[rs[rt]];
}
void build(int l,int r,int &rt) {
    rt=tot++;
    col[rt]=0;
    if(l==r) {
        scanf("%I64d",&sum[rt]);
        return;
    }
    int m=l+r>>1;
    build(lson);
    build(rson);
    push_up(rt);
}
void update(int last,int L,int R,int val,int l,int r,int &rt) {
    rt=tot++;
    col[rt]=col[last];
    sum[rt]=sum[last]+(long long)val*(R-L+1);
    if(L==l&&r==R) {
        col[rt]+=val;
        if(L!=R) {
            ls[rt]=ls[last];
            rs[rt]=rs[last];
        }
        return;
    }
    int m=l+r>>1;
    if(R<=m) {
        update(ls[last],L,R,val,lson);
        rs[rt]=rs[last];
    } else if(m<L) {
        ls[rt]=ls[last];
        update(rs[last],L,R,val,rson);
    } else {
        update(ls[last],L,m,val,lson);
        update(rs[last],m+1,R,val,rson);
    }
}
long long query(int L,int R,int l,int r,int rt) {
    if(L==l&&r==R)
        return sum[rt];
    long long ret=(long long)col[rt]*(R-L+1);
    int m=l+r>>1;
    if(R<=m)
        ret+=query(L,R,lson);
    else if(m<L)
        ret+=query(L,R,rson);
    else
        ret+=query(L,m,lson)+query(m+1,R,rson);
    return ret;
}
int main() {
    int n,m,l,r,d,h;
    char op;
    while(~scanf("%d%d",&n,&m)) {
        tot=h=0;
        build(1,n,root[h]);
        while(m--) {
            scanf(" %c",&op);
            switch(op) {
            case 'C':
                scanf("%d%d%d",&l,&r,&d);
                update(root[h],l,r,d,1,n,root[h+1]);
                ++h;
                break;
            case 'Q':
                scanf("%d%d",&l,&r);
                printf("%I64d\n",query(l,r,1,n,root[h]));
                break;
            case 'H':
                scanf("%d%d%d",&l,&r,&d);
                printf("%I64d\n",query(l,r,1,n,root[d]));
                break;
            case 'B':
                scanf("%d",&h);
                break;
            }
        }
    }
}
{% endhighlight %}

然后我果断跳坑，用可持久化树状数组写了一发。首先要会用树状数组处理区间修改和区间求和，然后对每一处节点都开一个vector记录历史版本，有点像邻接表的形式，只不过表头用树状数组维护起来罢了。现在的问题是复杂度为$$O(logn)$$修改+$$O(logn)$$查询+$$O(n)$$历史回溯，测试得出时间上是主席树的二倍多，空间上是主席树的不到四分之一，也许是数据不够强吧。代码长度上差不多，写的难度上可持久化树状数组会好一些，但除非空间限制卡得太死，我可能还是倾向于使用更全能的主席树。不过我相信依然有优化的空间，比如vector数组可以写成类似于前向星的形式，或者考虑在某些地方二分，也许可以把历史回溯的复杂度降到$$O(logn)$$。
{% highlight cpp linenos %}
#include<cstdio>
#include<algorithm>
#include<vector>
using namespace std;
const int MAXN=100005;
inline int lowbit(int x) {
    return x&-x;
}
struct Node {
    int d,h;
    long long nd;
    Node(int _d=0,long long _nd=0,int _h=0):d(_d),nd(_nd),h(_h) {}
};
vector<Node> bit[MAXN];
long long sum[MAXN];
int n,tl;
void init() {
    tl=sum[0]=0;
    for(int i=1; i<=n; ++i) {
        scanf("%I64d",&sum[i]);
        sum[i]+=sum[i-1];
        bit[i].clear();
        bit[i].push_back(Node());
    }
}
void add(int x,int val) {
    for(int i=x; i>0; i-=lowbit(i)) {
        int size=bit[i].size();
        bit[i].push_back(bit[i][size-1]);
        ++size;
        bit[i][size-1].d+=val;
        bit[i][size-1].h=tl;
    }
    for(int i=x; i<=n; i+=lowbit(i)) {
        int size=bit[i].size();
        if(i!=x) {
            bit[i].push_back(bit[i][size-1]);
            ++size;
        }
        bit[i][size-1].nd+=x*(long long)val;
        bit[i][size-1].h=tl;
    }
}
long long sumbit(int x,int h) {
    if(!x)
        return 0;
    long long sumd=0,sumnd=0;
    for(int i=x; i<=n; i+=lowbit(i)) {
        int size=bit[i].size();
        while(bit[i][size-1].h>h)
            --size;
        sumd+=bit[i][size-1].d;
    }
    for(int i=x-1; i>0; i-=lowbit(i)) {
        int size=bit[i].size();
        while(bit[i][size-1].h>h)
            --size;
        sumnd+=bit[i][size-1].nd;
    }
    return sumd*x+sumnd;
}
void back() {
    for(int i=1; i<=n; ++i)
        for(int size=bit[i].size(); bit[i][size-1].h>tl; --size)
            bit[i].pop_back();
}
void modify(int l,int r,int val) {
    ++tl;
    add(r,val);
    if(l>1)
        add(l-1,-val);
}
long long query(int l,int r,int h) {
    return sum[r]-sum[l-1]+sumbit(r,h)-sumbit(l-1,h);
}
int main() {
    int m,l,r,d;
    char op;
    while(~scanf("%d%d",&n,&m)) {
        init();
        while(m--) {
            scanf(" %c",&op);
            switch(op) {
            case 'C':
                scanf("%d%d%d",&l,&r,&d);
                modify(l,r,d);
                break;
            case 'Q':
                scanf("%d%d",&l,&r);
                printf("%I64d\n",query(l,r,tl));
                break;
            case 'H':
                scanf("%d%d%d",&l,&r,&d);
                printf("%I64d\n",query(l,r,d));
                break;
            case 'B':
                scanf("%d",&tl);
                back();
                break;
            }
        }
    }
}
{% endhighlight %}

隔段时间再来看这道题的可持久化树状数组做法，我意识到兴许可以做到$$O(log^2n)$$修改+$$O(log^2n)$$查询+$$O(1)$$历史回溯，就是不进行显式的回溯操作，而是在修改和查询时二分出对应的时间点；再考虑一下lazy标记的思想，最终做到$$O(logn)$$修改+$$O(logn)$$当前查询+$$O(log^2n)$$历史查询+$$O(1)$$历史回溯似乎也是可以的。这里就不再尝试了。

##[SPOJ10628](http://www.spoj.com/problems/COT/)

填坑填坑……这道题是树上无修改第K小，注意到u、v两点间路径对应的子树就是u+v-lca(u,v)-fa(lca(u,v))，然后就跟区间第K小没什么区别了……其实在写过一些通过数据结构维护树上信息的题之后，这道题写起来就变得很轻松了。
{% highlight cpp linenos %}
#include<cstdio>
#include<cstring>
#include<algorithm>
using namespace std;
#define lson l,m,ls[rt]
#define rson m+1,r,rs[rt]
const int MAXN=100005;
const int MAXM=MAXN<<1;
int ls[MAXN<<5],rs[MAXN<<5],cnt[MAXN<<5],root[MAXN],tot,size;
void build(int l,int r,int &rt) {
    rt=tot++;
    cnt[rt]=0;
    if(l==r)
        return;
    int m=l+r>>1;
    build(lson);
    build(rson);
}
void update(int last,int p,int l,int r,int &rt) {
    rt=tot++;
    ls[rt]=ls[last];
    rs[rt]=rs[last];
    cnt[rt]=cnt[last]+1;
    if(l==r)
        return;
    int m=l+r>>1;
    if(p<=m)
        update(ls[last],p,lson);
    else
        update(rs[last],p,rson);
}
int query(int ss,int tt,int fa,int ffa,int l,int r,int k) {
    if(l==r)
        return l;
    int m=l+r>>1,sum=cnt[ls[ss]]+cnt[ls[tt]]-cnt[ls[fa]]-cnt[ls[ffa]];
    if(k<=sum)
        return query(ls[ss],ls[tt],ls[fa],ls[ffa],l,m,k);
    else
        return query(rs[ss],rs[tt],rs[fa],rs[ffa],m+1,r,k-sum);
}
int num[MAXN],hash[MAXN];
struct graph {
    int head[MAXN];
    int to[MAXM],next[MAXM];
    int tot;
    void init() {
        tot=0;
        memset(head,0xff,sizeof(head));
    }
    void add(int u,int v) {
        to[tot]=v;
        next[tot]=head[u];
        head[u]=tot++;
    }
} g;
const int maxd=18;
int d[MAXN],f[MAXN][maxd];
void dfs(int u,int fa) {
    f[u][0]=fa;
    d[u]=d[fa]+1;
    update(root[fa],num[u],1,size,root[u]);
    for(int i=1; i<maxd; ++i)
        f[u][i]=f[f[u][i-1]][i-1];
    for(int i=g.head[u]; ~i; i=g.next[i]) {
        int v=g.to[i];
        if(v!=fa)
            dfs(v,u);
    }
}
int lca(int u,int v) {
    if(d[u]<d[v])
        swap(u,v);
    int k=d[u]-d[v];
    for(int i=0; i<maxd; ++i)
        if((1<<i)&k)
            u=f[u][i];
    if(u==v)
        return u;
    for(int i=maxd-1; i>=0; --i)
        if(f[u][i]!=f[v][i]) {
            u=f[u][i];
            v=f[v][i];
        }
    return f[u][0];
}
int main() {
    int n,m,u,v,k;
    while(~scanf("%d%d",&n,&m)) {
        for(int i=1; i<=n; ++i) {
            scanf("%d",&num[i]);
            hash[i]=num[i];
        }
        g.init();
        for(int i=1; i<n; ++i) {
            scanf("%d%d",&u,&v);
            g.add(u,v);
            g.add(v,u);
        }
        sort(hash+1,hash+n+1);
        size=unique(hash+1,hash+n+1)-hash-1;
        for(int i=1; i<=n; ++i)
            num[i]=lower_bound(hash+1,hash+size+1,num[i])-hash;
        tot=0;
        build(1,size,root[0]);
        dfs(1,0);
        while(m--) {
            scanf("%d%d%d",&u,&v,&k);
            int fa=lca(u,v);
            printf("%d\n",hash[query(root[u],root[v],root[fa],root[f[fa][0]],1,size,k)]);
        }
    }
}
{% endhighlight %}

##[SPOJ9066](http://www.spoj.com/problems/XXXXXXXX/)

也许还有人记得[UVa12345](http://uva.onlinejudge.org/index.php?option=com_onlinejudge&Itemid=8&page=show_problem&problem=3767)？好吧其实就在上面……这道题提出了更高的要求：要求可修改的同时，询问一个区间内不同的数的和。我们考虑一下之前那道题的做法，每当修改一个位置的时候，就在主席树原位置-1，在新位置+1，这样统计的和就是个数了。那么，如果我加减的不是1，而是这个数的值呢？统计出的和是不是就变成了题目要求的东西？这个思想也可以推广到平衡树中，也就是说，size域（区间大小）的维护，在本质上与区间和的维护没什么两样，都是某个节点的权值而已。
{% highlight cpp linenos %}
#include<cstdio>
#include<cstring>
#include<algorithm>
#include<set>
#include<map>
using namespace std;
#define lson l,m,ls[rt]
#define rson m+1,r,rs[rt]
const int MAXN=50005;
const int MAXH=150005;
const int MAXM=10000005;
int ls[MAXM],rs[MAXM],root[MAXN],tot;
long long cnt[MAXM];
void build(int l,int r,int &rt) {
    rt=tot++;
    cnt[rt]=0;
    if(l==r)
        return;
    int m=l+r>>1;
    build(lson);
    build(rson);
}
void update(int last,int p,int val,int l,int r,int &rt) {
    rt=tot++;
    ls[rt]=ls[last];
    rs[rt]=rs[last];
    cnt[rt]=cnt[last]+val;
    if(l==r)
        return;
    int m=l+r>>1;
    if(p<=m)
        update(ls[last],p,val,lson);
    else
        update(rs[last],p,val,rson);
}
long long query(int x,int l,int r,int rt) {
    if(l==r)
        return x<l?0:cnt[rt];
    int m=l+r>>1;
    if(x<=m)
        return query(x,lson);
    else
        return cnt[ls[rt]]+query(x,rson);
}
int n,m,num[MAXN],pre[MAXN],x,y,bit[MAXN],ht;
char op;
map<int,int> idx;
set<int> pos[MAXH];
set<int>::iterator it,jt;
int getidx(int x) {
    if(idx.count(x))
        return idx[x];
    idx[x]=++ht;
    return ht;
}
inline int lowbit(int x) {
    return x&-x;
}
void update(int x,int newpre,int prev,int val) {
    for(int i=x; i<=n; i+=lowbit(i)) {
        update(bit[i],pre[x],-prev,0,n,bit[i]);
        update(bit[i],newpre,val,0,n,bit[i]);
    }
    pre[x]=newpre;
}
void update(int x,int val) {
    if(num[x]==val)
        return;
    int pk=getidx(num[x]),vk=getidx(val);
    pos[pk].erase(x);
    it=pos[pk].lower_bound(x);
    if(it!=pos[pk].end())
        update(*it,pre[x],num[x],num[x]);
    it=jt=pos[vk].lower_bound(x);
    update(x,it==pos[vk].begin()?0:*(--jt),num[x],val);
    if(it!=pos[vk].end())
        update(*it,x,val,val);
    pos[vk].insert(x);
    num[x]=val;
}
long long query(int l,int r) {
    long long ret=query(l-1,0,n,root[r])-query(l-1,0,n,root[l-1]);
    for(int i=r; i>0; i-=lowbit(i))
        ret+=query(l-1,0,n,bit[i]);
    for(int i=l-1; i>0; i-=lowbit(i))
        ret-=query(l-1,0,n,bit[i]);
    return ret;
}
int main() {
    while(~scanf("%d",&n)) {
        memset(pre,0,sizeof(pre));
        idx.clear();
        for(int i=0; i<MAXH; ++i)
            pos[i].clear();
        tot=ht=0;
        build(0,n,root[0]);
        for(int i=1; i<=n; ++i) {
            scanf("%d",&num[i]);
            int k=getidx(num[i]);
            it=pos[k].end();
            if(it!=pos[k].begin())
                pre[i]=*(--it);
            pos[k].insert(i);
            update(root[i-1],pre[i],num[i],0,n,root[i]);
        }
        for(int i=1; i<=n; ++i)
            bit[i]=root[0];
        scanf("%d",&m);
        while(m--) {
            scanf(" %c%d%d",&op,&x,&y);
            switch(op) {
            case 'U':
                update(x,y);
                break;
            case 'Q':
                printf("%lld\n",query(x,y));
                break;
            }
        }
    }
}
{% endhighlight %}

##[SPOJ11985](http://www.spoj.com/problems/GOT/)

询问树上任意一条链，是否存在权值为某个值的点。这道题大多数人用的都是我看不懂的离线算法= =我在这里给出复杂度为$$O(mlog^2n)$$的在线做法，其实很好理解，就是用树链剖分套主席树；对每条划分出的路径，看对应的主席树的对应权值个数是否大于0即可。
{% highlight cpp linenos %}
#include<cstdio>
#include<cstring>
#include<algorithm>
using namespace std;
const int MAXN=100005;
const int MAXM=MAXN<<1;
const int MAXF=2000005;
struct graph {
    int head[MAXN],tot;
    int to[MAXM],next[MAXM];
    void init() {
        tot=0;
        memset(head,0xff,sizeof(head));
    }
    void add(int u,int v) {
        to[tot]=v;
        next[tot]=head[u];
        head[u]=tot++;
    }
} g;
int top[MAXN],len[MAXN];
int belong[MAXN],idx[MAXN];
int dep[MAXN],fa[MAXN],size[MAXN];
int que[MAXN];
bool vis[MAXN];
int n,stot;
void split() {
    memset(dep,0xff,sizeof(dep));
    int l=0,r=0;
    que[++r]=1;
    dep[1]=0;
    fa[1]=-1;
    while(l<r) {
        int u=que[++l];
        vis[u]=false;
        for(int i=g.head[u]; ~i; i=g.next[i]) {
            int v=g.to[i];
            if(!~dep[v]) {
                que[++r]=v;
                dep[v]=dep[u]+1;
                fa[v]=u;
            }
        }
    }
    stot=0;
    for(int i=n; i>0; --i) {
        int u=que[i],p=-1;
        size[u]=1;
        for(int j=g.head[u]; ~j; j=g.next[j]) {
            int v=g.to[j];
            if(vis[v]) {
                size[u]+=size[v];
                if(!~p||size[v]>size[p])
                    p=v;
            }
        }
        if(!~p) {
            idx[u]=len[++stot]=1;
            belong[u]=stot;
            top[stot]=u;
        } else {
            belong[u]=belong[p];
            idx[u]=++len[belong[u]];
            top[belong[u]]=u;
        }
        vis[u]=true;
    }
}
int fi[MAXN],cid[MAXN],rank[MAXN];
void getcid() {
    fi[1]=1;
    for(int i=2; i<=stot; ++i)
        fi[i]=fi[i-1]+len[i-1];
    for(int i=1; i<=n; ++i) {
        cid[i]=fi[belong[i]]+len[belong[i]]-idx[i];
        rank[cid[i]]=i;
    }
}
#define lson l,m,ls[rt]
#define rson m+1,r,rs[rt]
int ls[MAXF],rs[MAXF],cnt[MAXF],root[MAXN],ftot;
void build(int l,int r,int &rt) {
    rt=ftot++;
    cnt[rt]=0;
    if(l==r)
        return;
    int m=l+r>>1;
    build(lson);
    build(rson);
}
void update(int last,int p,int l,int r,int &rt) {
    rt=ftot++;
    ls[rt]=ls[last];
    rs[rt]=rs[last];
    cnt[rt]=cnt[last]+1;
    if(l==r)
        return;
    int m=l+r>>1;
    if(p<=m)
        update(ls[last],p,lson);
    else
        update(rs[last],p,rson);
}
int query(int ss,int tt,int v,int l,int r) {
    if(l==r)
        return cnt[tt]-cnt[ss];
    int m=l+r>>1;
    if(v<=m)
        return query(ls[ss],ls[tt],v,l,m);
    else
        return query(rs[ss],rs[tt],v,m+1,r);
}
bool query(int x,int y,int v) {
    int sum=0;
    while(belong[x]!=belong[y]) {
        if(dep[top[belong[x]]]<dep[top[belong[y]]])
            swap(x,y);
        sum+=query(root[cid[top[belong[x]]]-1],root[cid[x]],v,0,n);
        x=fa[top[belong[x]]];
    }
    if(dep[x]>dep[y])
        swap(x,y);
    sum+=query(root[cid[x]-1],root[cid[y]],v,0,n);
    return sum>0;
}
int num[MAXN];
int main() {
    int m,a,b,c;
    while(~scanf("%d%d",&n,&m)) {
        for(int i=1; i<=n; ++i)
            scanf("%d",&num[i]);
        g.init();
        for(int i=1; i<n; ++i) {
            scanf("%d%d",&a,&b);
            g.add(a,b);
            g.add(b,a);
        }
        split();
        getcid();
        ftot=0;
        build(0,n,root[0]);
        for(int i=1; i<=n; ++i)
            update(root[i-1],num[rank[i]],0,n,root[i]);
        while(m--) {
            scanf("%d%d%d",&a,&b,&c);
            puts(query(a,b,c)?"Find":"NotFind");
        }
        putchar('\n');
    }
}
{% endhighlight %}

基本就这些了吧……