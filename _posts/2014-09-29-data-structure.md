---
layout: post
title: ACM常用数据结构小结与实现
date: 2014-09-29 17:01:00
category: acm
tags: acm
comments: true
---
应当说这段时间学习了很多的数据结构，也到了一个总结的时候。fotile96的[这篇Blog](https://13331.org/383.html)非常值得推荐，我达不到这个高度，只能给自己和队友做些简单的归纳。

##树状数组

非常简单的数据结构，只需要一个数组，一切操作基于如下的函数。需要注意的是，树状数组的下标必须从1开始（从0开始会死循环）。它可以做到在$$O(\log n)$$时间内完成下述的操作。
{% highlight cpp linenos %}
int bit[MAXN],n;
inline int lowbit(int x) {
    return x&-x;
}
{% endhighlight %}

最常见的是增减单点的值，询问前缀和。
{% highlight cpp linenos %}
void add(int x,int val) {
    for(int i=x; i<=n; i+=lowbit(i))
        bit[i]+=val;
}
int sum(int x) {
    int ret=0;
    for(int i=x; i>0; i-=lowbit(i))
        ret+=bit[i];
    return ret;
}
{% endhighlight %}

我们注意到，只要维护原序列的差分序列，就可以用上述方式实现对前缀的每一位增减同一个值，询问单点值。不过，其实我们不需要手动将原数组转化为差分序列，只需对函数做一个简单的转化。
{% highlight cpp linenos %}
void add(int x,int val) {
    for(int i=x; i>0; i-=lowbit(i))
        bit[i]+=val;
}
int get(int x) {
    int ret=0;
    for(int i=x; i<=n; i+=lowbit(i))
        ret+=bit[i];
    return ret;
}
{% endhighlight %}

树状数组其实是可以实现区间增减，区间求和的。但这个一般还是用线段树来实现。
{% highlight cpp linenos %}
//修改区间：add(r,val);
//          if(l>1) add(l-1,-val);
//查询区间：sum(r)-sum(l-1);
int bit1[MAXN],bit2[MAXN],n;
void add(int x,int val) {
    for(int i=x; i>0; i-=lowbit(i))
        bit1[i]+=val;
    for(int i=x; i<=n; i+=lowbit(i))
        bit2[i]+=x*val;
}
int sum(int x) {
    if(!x)
        return 0;
    int ret1=0,ret2=0;
    for(int i=x; i<=n; i+=lowbit(i))
        ret1+=bit1[i];
    for(int i=x-1; i>0; i-=lowbit(i))
        ret2+=bit2[i];
    return ret1*x+ret2;
}
{% endhighlight %}

树状数组也可以维护最值，但复杂度上升一个$$\log$$，所以不如用线段树来维护。
{% highlight cpp linenos %}
void modify(int x,int val) {
    num[x]=val;
    for(int i=x; i<=n; i+=lowbit(i)) {
        bit[i]=max(bit[i],val);
        for(int j=1; j<lowbit(i); j<<=1)
            bit[i]=max(bit[i],bit[i-j]);
    }
}
int query(int L,int R) {
    int ret=num[R];
    while(true) {
        ret=max(ret,num[R]);
        if(L==R)
            break;
        for(R-=1; R-L>=lowbit(R); R-=lowbit(R))
            ret=max(ret,bit[R]);
    }
    return ret;
}
{% endhighlight %}

##线段树

现在最常见的线段树实现大概有两个版本，一个是NotOnlySuccess的风格，也是我最常使用的风格；它的特点是利用二叉树的数学关系来维护根节点信息。这种写法有一点不足是必须开4倍内存。支持在$$O(1)$$时间内将子树信息合并的序列信息理论上都可以用线段树来维护（如最大子段和等），区间维护依靠lazy标记完成以维持$$O(\log n)$$的单次操作复杂度。
{% highlight cpp linenos %}
#define lson l,m,rt<<1
#define rson m+1,r,rt<<1|1
int sum[MAXN<<2],add[MAXN<<2];
void push_up(int rt) {
    sum[rt]=sum[rt<<1]+sum[rt<<1|1];
}
void push_down(int rt,int len) {
    if(add[rt]) {
        add[rt<<1]+=add[rt];
        add[rt<<1|1]+=add[rt];
        sum[rt<<1]+=add[rt]*(len-(len>>1));
        sum[rt<<1|1]+=add[rt]*(len>>1);
        add[rt]=0;
    }
}
void build(int l,int r,int rt) {
    add[rt]=0;
    if(l==r) {
        scanf("%d",&sum[rt]);
        return;
    }
    int m=l+r>>1;
    build(lson);
    build(rson);
    push_up(rt);
}
void update(int p,int val,int l,int r,int rt) {
    if(l==r) {
        sum[rt]+=val;
        return;
    }
    push_down(rt,r-l+1);
    int m=l+r>>1;
    if(p<=m)
        update(p,val,lson);
    else
        update(p,val,rson);
    push_up(rt);
}
void update(int L,int R,int val,int l,int r,int rt) {
    if(L<=l&&r<=R) {
        add[rt]+=val;
        sum[rt]+=val*(r-l+1);
        return;
    }
    push_down(rt,r-l+1);
    int m=l+r>>1;
    if(L<=m)
        update(L,R,val,lson);
    if(m<R)
        update(L,R,val,rson);
    push_up(rt);
}
int query(int L,int R,int l,int r,int rt) {
    if(L<=l&&r<=R)
        return sum[rt];
    push_down(rt,r-l+1);
    int m=l+r>>1,ret=0;
    if(L<=m)
        ret+=query(L,R,lson);
    if(m<R)
        ret+=query(L,R,rson);
    return ret;
}
{% endhighlight %}

另外一种有些类似，与上面不同的是，它使用下面这个神奇的ID函数组织每个节点在数组中的位置。具体实现不再赘述。
{% highlight cpp linenos %}
inline int ID(int l,int r) {
    return l+r|l!=r;
}
{% endhighlight %}

##可持久化线段树

如果我们想在用数据结构维护信息时，留下所有操作的历史记录，以便回退和查询，我们就需要对数据结构进行可持久化。可持久化的理念非常简单，就是把每次修改节点的操作变成新建新节点的操作，留下了原节点就留下了历史记录。

线段树是天生利于可持久化的数据结构，原因在于形态的一致性。具体地说，对于以固定方式建立的固定大小的线段树，其形态是完全一致的，且不会因修改而产生变动。为了节约空间，采用了自顶向下的函数式风格，保证了每次更新节点个数在$$O(\log n)$$的级别，这个做法由黄嘉泰（fotile96）首创，故又称主席树。更具体的介绍可以看我[以前的Blog](http://wjfwzzc.me/posts/fotile-tree/)，里面有很多例题，这里不再赘述。

##实时开节点的线段树

我们注意到可持久化线段树常常被用来维护集合信息而非序列信息，换言之，经常以权值线段树的形式存在，其大小不由数据量决定，而是由数据的范围决定。这样我们便遇到一个困境，对于可持久化线段树这一在线维护信息的利器，却常常需要先离散化才可以使用，从实际上变成了离线做法，面对强制在线的题目非常尴尬。对此，陈立杰（WJMZBMR）在[13年国家集训队论文](http://www.doc88.com/p-8416874386976.html)中提到实时开节点的权值线段树；陈立杰的解释有一点抽象，我们可以这样理解，可持久化线段树是先建立初始版本的完整的树，在创建新版本时对修改的节点实时开新节点；而我们现在变成一棵初始状态为一棵空树，从一开始就实时开新节点。

即使这样说，还是显得不够具体，难以据此实现出实时开节点的线段树，所以这里以[BZOJ1901](http://www.lydsy.com/JudgeOnline/problem.php?id=1901)为例，给出一个没有离散化，而是采用实时开节点的可持久化线段树的实现，与[这里的代码](http://blog.csdn.net/wjf_wzzc/article/details/24560117#t2)做一个对比；可以发现实现起来并不麻烦。唯一美中不足是线段树部分单次操作复杂度由$$O(\log n)$$升为$$O(\log V)$$，其中n是数据量，V是数据范围上界。
{% highlight cpp linenos %}
#include<cstdio>
#include<cstring>
using namespace std;
#define lson l,m,ls[rt]
#define rson m+1,r,rs[rt]
const int MAXN=60005;
const int MAXM=2500005;
const int INF=0x3f3f3f3f;
int ls[MAXM],rs[MAXM],cnt[MAXM],root[MAXN],tot;
int new_node() {
    ++tot;
    ls[tot]=rs[tot]=cnt[tot]=0;
    return tot;
}
void update(int p,int val,int l,int r,int &rt) {
    if(!rt)
        rt=new_node();
    if(l==r) {
        cnt[rt]+=val;
        return;
    }
    int m=l+r>>1;
    if(p<=m)
        update(p,val,lson);
    else
        update(p,val,rson);
    cnt[rt]=cnt[ls[rt]]+cnt[rs[rt]];
}
int use[MAXN],n;
inline int lowbit(int x) {
    return x&-x;
}
void modify(int x,int p,int val) {
    for(int i=x; i<=n; i+=lowbit(i))
        update(p,val,0,INF,root[i]);
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
int a[MAXN];
int main() {
    int m,l,r,k;
    char op;
    while(~scanf("%d%d",&n,&m)) {
        tot=0;
        memset(root,0,sizeof(root));
        for(int i=1; i<=n; ++i) {
            scanf("%d",&a[i]);
            modify(i,a[i],1);
        }
        while(m--) {
            scanf(" %c%d%d",&op,&l,&r);
            switch(op) {
            case 'Q':
                scanf("%d",&k);
                printf("%d\n",query(l-1,r,0,INF,k));
                break;
            case 'C':
                modify(l,a[l],-1);
                a[l]=r;
                modify(l,a[l],1);
                break;
            }
        }
    }
}
{% endhighlight %}

##SkipList（跳表）

没什么用，这是我对它的评价；它能实现的一切操作用平衡二叉树都可以实现。这里放一个我测过可用，但没花心思修改的版本。
{% highlight cpp linenos %}
const int MAX_LEVEL=18;
struct SkipList {
    struct node {
        int key,val;
        node *next[1];
    };
    int level;
    node *head;
    SkipList() {
        level=0;
        head=NewNode(MAX_LEVEL-1,0,0);
        for(int i=0; i<MAX_LEVEL; ++i)
            head->next[i]=NULL;
    }
    node* NewNode(int level,int key,int val) {

        node *ns=(node *)malloc(sizeof(node)+level*sizeof(node*));
        ns->key=key;
        ns->val=val;
        return ns;
    }
    int randomLevel() {
        int k=1;
        while(rand()&1)
            ++k;
        return k<MAX_LEVEL?k:MAX_LEVEL;
    }
    int find(int key) {
        node *p=head,*q=NULL;
        for(int i=level-1; i>=0; --i)
            while((q=p->next[i])&&q->key<=key) {
                if(q->key==key)
                    return q->val;
                p=q;
            }
        return -INF;
    }
    bool insert(int key,int val) {
        node *update[MAX_LEVEL],*p=head,*q=NULL;
        for(int i=level-1; i>=0; --i) {
            while((q=p->next[i])&&q->key<key)
                p=q;
            update[i]=p;
        }
        if(q&&q->key==key)
            return false;
        int k=randomLevel();
        if(k>level) {
            for(int i=level; i<k; ++i)
                update[i]=head;
            level=k;
        }
        q=NewNode(k,key,val);
        for(int i=0; i<k; ++i) {
            q->next[i]=update[i]->next[i];
            update[i]->next[i]=q;
        }
        return true;
    }
    bool erase(int key) {
        node *update[MAX_LEVEL],*p=head,*q=NULL;
        for(int i=level-1; i>=0; --i) {
            while((q=p->next[i])&&q->key<key)
                p=q;
            update[i]=p;
        }
        if(q&&q->key==key) {
            for(int i=0; i<level; ++i)
                if(update[i]->next[i]==q)
                    update[i]->next[i]=q->next[i];
            free(q);
            for(int i=level-1; i>=0; --i)
                if(head->next[i]==NULL)
                    --level;
            return true;
        }
        return false;
    }
    node* getMin() {
        return head->next[0];
    }
    node* getMax() {
        node *p=head,*q=NULL;
        for(int i=level-1; i>=0; --i)
            while((q=p->next[i])&&q)
                p=q;
        return p==head?NULL:p;
    }
};
{% endhighlight %}

##平衡二叉树

平衡二叉树是维护集合有序性的常用数据结构。我会的平衡二叉树并不多，基本以实用为原则，有Treap、Size Balanced Tree（以下简称SBT）和Splay。它们的很多操作的实现方式完全一致或大同小异，这里为节省篇幅一并放出；注意这些操作适用于不允许重复值的平衡二叉树（set而非multiset），对于允许重复值（拥有cnt域）的实现，只要在一些+1的地方稍作修改（改成cnt[x]）即可。一些使用的例题可以在[这篇Blog](http://blog.csdn.net/wjf_wzzc/article/details/38646837)里找到，不再赘述。
{% highlight cpp linenos %}
bool find(int v) {
    for(int x=root; x; x=ch[x][key[x]<v])
        if(key[x]==v)
            return true;
    return false;
}
int get_kth(int k) {
    int x=root;
    while(size[ch[x][0]]+1!=k)
        if(k<size[ch[x][0]]+1)
            x=ch[x][0];
        else {
            k-=size[ch[x][0]]+1;
            x=ch[x][1];
        }
    return key[x];
}
int get_rank(int v) {
    int ret=0,x=root;
    while(x)
        if(v<key[x])
            x=ch[x][0];
        else {
            ret+=size[ch[x][0]]+1;
            x=ch[x][1];
        }
    return ret;
}
int get_pre(int v) {
    int x=root,y=0;
    while(x)
        if(v<key[x])
            x=ch[x][0];
        else {
            y=x;
            x=ch[x][1];
        }
    return y;
}
int get_next(int v) {
    int x=root,y=0;
    while(x)
        if(v>key[x])
            x=ch[x][1];
        else {
            y=x;
            x=ch[x][0];
        }
    return y;
}
int get_min() {
    if(size[root]==0)
        return -1;
    int x=root;
    while(ch[x][0])
        x=ch[x][0];
    return x;
}
int get_max() {
    if(size[root]==0)
        return -1;
    int x=root;
    while(ch[x][1])
        x=ch[x][1];
    return x;
}
void Treaval(int x) {
    if(x) {
        Treaval(ch[x][0]);
        printf("结点%2d:左儿子 %2d 右儿子 %2d size = %2d ,val = %2d\n",x,ch[x][0],ch[x][1],size[x],key[x]);
        Treaval(ch[x][1]);
    }
}
void debug() {
    printf("root:%d\n",root);
    Treaval(root);
    putchar('\n');
}
{% endhighlight %}

##基于旋转的Treap

Treap大概是赛场上最常见的平衡二叉树，它同时维护序列的有序性和堆的性质，依靠堆值的随机化，将树的高度维护在期望下平衡的程度，从而实现了各种操作期望$$O(\log n)$$的复杂度。它的性价比高在只有两种旋转（而且可以合并地写），比红黑树和AVL短小；又因为复杂度基于期望而非均摊，在各种数据下都有良好的表现。这里只给出允许重复值的实现。
{% highlight cpp linenos %}
struct Treap {
    int tot,root;
    int ch[MAXN][2],key[MAXN],pt[MAXN],cnt[MAXN],size[MAXN];
    void init() {
        tot=root=0;
        pt[0]=INF;
    }
    void push_up(int x) {
        size[x]=size[ch[x][0]]+size[ch[x][1]]+cnt[x];
    }
    void new_node(int &x,int v) {
        x=++tot;
        ch[x][0]=ch[x][1]=0;
        size[x]=cnt[x]=1;
        pt[x]=rand();
        key[x]=v;
    }
    void rotate(int &x,int f) {
        int y=ch[x][f];
        ch[x][f]=ch[y][f^1];
        ch[y][f^1]=x;
        push_up(x);
        push_up(y);
        x=y;
    }
    void insert(int &x,int v) {
        if(!x) {
            new_node(x,v);
            return;
        }
        if(key[x]==v)
            ++cnt[x];
        else {
            int f=key[x]<v;
            insert(ch[x][f],v);
            if(pt[ch[x][f]]<pt[x])
                rotate(x,f);
        }
        push_up(x);
    }
    void erase(int &x,int v) {
        if(!x)
            return;
        if(key[x]==v) {
            if(cnt[x]>1)
                --cnt[x];
            else {
                if(!ch[x][0]&&!ch[x][1])
                    x=0;
                else {
                    rotate(x,pt[ch[x][0]]>pt[ch[x][1]]);
                    erase(x,v);
                }
            }
        } else
            erase(ch[x][key[x]<v],v);
        push_up(x);
    }
    void insert(int v) {
        insert(root,v);
    }
    void erase(int v) {
        erase(root,v);
    }
};
{% endhighlight %}

##Size Balanced Tree

这个由陈启峰发明的数据结构依靠其独特的平摊时间$$O(1)$$的Maintain操作，具有仅次于红黑树的优秀的时间效率，但由于赛场上Treap已经够用，也因为有人指出陈启峰在复杂度的证明上有漏洞（有人声称某些数据可以使SBT退化成人字形，可以想象一下成串的鞭炮的样子），使用的人并不多。我也只是大致学习了一下，用的次数并不多。这里给出不允许重复值的实现。
{% highlight cpp linenos %}
struct SBT {
    int root,tot;
    int ch[MAXN][2],key[MAXN],size[MAXN];
    void init() {
        tot=root=0;
        size[0]=0;
    }
    void rotate(int &x,int f) {
        int y=ch[x][f];
        ch[x][f]=ch[y][f^1];
        ch[y][f^1]=x;
        size[y]=size[x];
        size[x]=size[ch[x][0]]+size[ch[x][1]]+1;
        x=y;
    }
    void maintain(int &x,int f) {
        if(size[ch[ch[x][f]][f]]>size[ch[x][f^1]])
            rotate(x,f);
        else if(size[ch[ch[x][f]][f^1]]>size[ch[x][f^1]]) {
            rotate(ch[x][f],f^1);
            rotate(x,f);
        } else
            return;
        maintain(ch[x][0],0);
        maintain(ch[x][1],1);
        maintain(x,0);
        maintain(x,1);
    }
    void insert(int &x,int v) {
        if(!x) {
            x=++tot;
            ch[x][0]=ch[x][1]=0;
            size[x]=1;
            key[x]=v;
        } else {
            ++size[x];
            insert(ch[x][key[x]<v],v);
            maintain(x,key[x]<v);
        }
    }
    int erase(int &x,int v) {
        if(!x)
            return 0;
        --size[x];
        if(key[x]==v||(key[x]>v&&!ch[x][0])||(key[x]<v&&!ch[x][1])) {
            int ret=key[x];
            if(ch[x][0]&&ch[x][1])
                key[x]=erase(ch[x][0],v+1);
            else
                x=ch[x][0]+ch[x][1];
            return ret;
        }
        return erase(ch[x][key[x]<v],v);
    }
    void insert(int v) {
        insert(root,v);
    }
    void erase(int v) {
        erase(root,v);
    }
};
{% endhighlight %}

##Splay（伸展树）

Splay因其独特性的splay操作（将某一节点旋转到另一节点下）而得名，是一种非常灵活的，在现实生活中应用也非常广泛的二叉查找树；称之为平衡二叉树是不太严谨的，因为它从来不保证高度平衡，而是每次将访问过的节点旋转到根。也因这一操作，Splay变得异常强大，可以实现很多其它平衡树无法实现的操作（如区间翻转），它更像平衡树和线段树的杂糅，既可以维护集合信息，也可以维护序列信息，可以用它来做Treap的题，也可以用它来做线段树的题。更重要的是，Splay可以实现split（将某棵子树从原树中分离）和merge操作（将某棵子树插入另一棵树），这也使得区间插入删除成为可能。它的美中不足是常数稍大，约是Treap的1.5～3倍，线段树的2～5倍。

Splay有单旋和双旋两种实现，其中只有双旋保证了均摊$$O(\log n)$$的单次操作复杂度，但因为很多人认为zigzag太长不好敲（大多是OI选手有此困扰），选择了单旋。其实完全可以稍微损失一点常数，合并成一个rotate函数来完成双旋。此外一个良好的实现通常要在序列一首一尾增加两个哨兵节点，这样可以减少很多边界特判。

有必要进行的扩展性说明是，对于一棵树，如果想要维护子树信息，我们可以用Splay维护这棵树的括号序列（dfs序），这样便可以轻易split出任意子树所属的区间；而用Splay维护dfs序的结构，就是Euler-Tour Tree。同样的，如果想要维护链上信息，可以先树链剖分然后用Splay维护每条重链，根据杨哲在[07年国家集训队作业](http://wenku.baidu.com/view/75906f160b4e767f5acfcedb)的计算，因其势能分析得到的复杂度依然是单次操作均摊$$O(\log n)$$复杂度；而类似的思想做些转化，就变成了后面会提到的Link-Cut Tree（以下简称LCT）。

这里给出了[POJ3580](http://poj.org/problem?id=3580)的Splay实现。注意我这次erase函数写了内存回收，这一做法完全可以照搬到其它平衡树中。
{% highlight cpp linenos %}
#include<cstdio>
#include<algorithm>
using namespace std;
#define keyTree (ch[ch[root][1]][0])
const int MAXN=200005;
const int INF=0x3f3f3f3f;
int num[MAXN];
struct SplayTree {
    int root,tot1,tot2;
    int ch[MAXN][2],pre[MAXN],size[MAXN];
    int gc[MAXN],que[MAXN];
    int key[MAXN],vmin[MAXN],add[MAXN],rev[MAXN];
    void rotate(int x,int f) {
        int y=pre[x];
        ch[y][f^1]=ch[x][f];
        pre[ch[x][f]]=y;
        pre[x]=pre[y];
        if(pre[x])
            ch[pre[y]][ch[pre[y]][1]==y]=x;
        ch[x][f]=y;
        pre[y]=x;
        push_up(y);
    }
    void splay(int x,int goal) {
        push_down(x);
        while(pre[x]!=goal) {
            int y=pre[x],z=pre[y];
            if(z==goal) {
                push_down(y);
                push_down(x);
                rotate(x,ch[y][0]==x);
            } else {
                push_down(z);
                push_down(y);
                push_down(x);
                int f=ch[z][0]==y;
                if(ch[y][f]==x)
                    rotate(x,f^1);
                else
                    rotate(y,f);
                rotate(x,f);
            }
        }
        push_up(x);
        if(goal==0)
            root=x;
    }
    void rotate_to(int k,int goal) {
        int x=root;
        push_down(x);
        while(size[ch[x][0]]!=k) {
            if(k<size[ch[x][0]])
                x=ch[x][0];
            else {
                k-=size[ch[x][0]]+1;
                x=ch[x][1];
            }
            push_down(x);
        }
        splay(x,goal);
    }
    void erase(int x) {
        int fa=pre[x],head=0,tail=0;
        for(que[tail++]=x; head<tail; ++head) {
            gc[tot2++]=que[head];
            if(ch[que[head]][0])
                que[tail++]=ch[que[head]][0];
            if(ch[que[head]][1])
                que[tail++]=ch[que[head]][1];
        }
        ch[fa][ch[fa][1]==x]=0;
        push_up(fa);
    }
    void new_node(int &x,int v,int fa) {
        if(tot2)
            x=gc[--tot2];
        else
            x=++tot1;
        ch[x][0]=ch[x][1]=0;
        pre[x]=fa;
        size[x]=1;
        key[x]=vmin[x]=v;
        add[x]=rev[x]=0;
    }
    void update_add(int x,int d) {
        if(x) {
            key[x]+=d;
            add[x]+=d;
            vmin[x]+=d;
        }
    }
    void update_rev(int x) {
        if(x) {
            swap(ch[x][0],ch[x][1]);
            rev[x]^=1;
        }
    }
    void push_up(int x) {
        size[x]=size[ch[x][0]]+size[ch[x][1]]+1;
        vmin[x]=min(key[x],min(vmin[ch[x][0]],vmin[ch[x][1]]));
    }
    void push_down(int x) {
        if(add[x]) {
            update_add(ch[x][0],add[x]);
            update_add(ch[x][1],add[x]);
            add[x]=0;
        }
        if(rev[x]) {
            update_rev(ch[x][0]);
            update_rev(ch[x][1]);
            rev[x]=0;
        }
    }
    void build(int &x,int l,int r,int f) {
        int m=l+r>>1;
        new_node(x,num[m],f);
        if(l<m)
            build(ch[x][0],l,m-1,x);
        if(r>m)
            build(ch[x][1],m+1,r,x);
        push_up(x);
    }
    void init(int n) {
        root=tot1=tot2=0;
        ch[0][0]=ch[0][1]=pre[0]=size[0]=0;
        add[0]=rev[0]=0;
        key[0]=vmin[0]=INF;
        new_node(root,-1,0);
        new_node(ch[root][1],-1,root);
        size[root]=2;
        for(int i=1; i<=n; ++i)
            scanf("%d",&num[i]);
        build(keyTree,1,n,ch[root][1]);
        push_up(ch[root][1]);
        push_up(root);
    }
    void plus(int l,int r,int v) {
        rotate_to(l-1,0);
        rotate_to(r+1,root);
        update_add(keyTree,v);
    }
    void reverse(int l,int r) {
        rotate_to(l-1,0);
        rotate_to(r+1,root);
        update_rev(keyTree);
    }
    void revolve(int l,int r,int k) {
        k%=r-l+1;
        if(!k)
            return;
        rotate_to(r-k,0);
        rotate_to(r+1,root);
        int tmp=keyTree;
        keyTree=0;
        push_up(ch[root][1]);
        push_up(root);
        rotate_to(l-1,0);
        rotate_to(l,root);
        keyTree=tmp;
        pre[tmp]=ch[root][1];
        push_up(ch[root][1]);
        push_up(root);
    }
    void insert(int k,int v) {
        rotate_to(k,0);
        rotate_to(k+1,root);
        new_node(keyTree,v,ch[root][1]);
        push_up(ch[root][1]);
        push_up(root);
    }
    void del(int k) {
        rotate_to(k-1,0);
        rotate_to(k+1,root);
        erase(keyTree);
        push_up(ch[root][1]);
        push_up(root);
    }
    int query(int l,int r) {
        rotate_to(l-1,0);
        rotate_to(r+1,root);
        return vmin[keyTree];
    }
} splay;
int main() {
    int n,m,x,y,v;
    char op[10];
    while(~scanf("%d",&n)) {
        splay.init(n);
        scanf("%d",&m);
        while(m--) {
            scanf("%s",op);
            switch(op[0]) {
            case 'A':
                scanf("%d%d%d",&x,&y,&v);
                splay.plus(x,y,v);
                break;
            case 'R':
                scanf("%d%d",&x,&y);
                if(op[3]=='E')
                    splay.reverse(x,y);
                else {
                    scanf("%d",&v);
                    splay.revolve(x,y,v);
                }
                break;
            case 'I':
                scanf("%d%d",&x,&v);
                splay.insert(x,v);
                break;
            case 'D':
                scanf("%d",&x);
                splay.del(x);
                break;
            case 'M':
                scanf("%d%d",&x,&y);
                printf("%d\n",splay.query(x,y));
                break;
            }
        }
    }
}
{% endhighlight %}

额外的更新。哨兵节点的存在有利有弊，在对树进行split和merge的时候有时会出现一些调试上的不便，在某些时候显得代码不够优雅。由此，我参照交大板，结合LCT的一些函数写了新的实现。以下是[HDU4453](http://acm.hdu.edu.cn/showproblem.php?pid=4453)的Splay实现，当然用上面的做法和后面提到的不基于旋转的Treap同样可做。
{% highlight cpp linenos %}
#include<cstdio>
#include<algorithm>
using namespace std;
const int MAXN=200005;
int k1,k2,num[MAXN];
struct Splay {
    int root,tot,point;
    int ch[MAXN][2],pre[MAXN],size[MAXN];
    int key[MAXN],add[MAXN],rev[MAXN];
    bool isroot(int x) {
        return !pre[x]||ch[pre[x]][0]!=x&&ch[pre[x]][1]!=x;
    }
    void rotate(int x) {
        int y=pre[x],f=ch[y][1]==x;
        ch[y][f]=ch[x][f^1];
        pre[ch[y][f]]=y;
        if(!isroot(y))
            ch[pre[y]][ch[pre[y]][1]==y]=x;
        pre[x]=pre[y];
        ch[x][f^1]=y;
        pre[y]=x;
        push_up(y);
    }
    void splay(int x) {
        push_down(x);
        while(!isroot(x)) {
            int y=pre[x],z=pre[y];
            if(isroot(y)) {
                push_down(y);
                push_down(x);
                rotate(x);
            } else {
                push_down(z);
                push_down(y);
                push_down(x);
                rotate((ch[z][1]==y)==(ch[y][1]==x)?y:x);
                rotate(x);
            }
        }
        push_up(x);
    }
    void new_node(int &x,int v,int fa) {
        x=++tot;
        ch[x][0]=ch[x][1]=0;
        pre[x]=fa;
        size[x]=1;
        key[x]=v;
        add[x]=rev[x]=0;
    }
    void update_add(int x,int v) {
        if(x) {
            key[x]+=v;
            add[x]+=v;
        }
    }
    void update_rev(int x) {
        if(x) {
            rev[x]^=1;
            swap(ch[x][0],ch[x][1]);
        }
    }
    void push_down(int x) {
        if(add[x]) {
            update_add(ch[x][0],add[x]);
            update_add(ch[x][1],add[x]);
            add[x]=0;
        }
        if(rev[x]) {
            update_rev(ch[x][0]);
            update_rev(ch[x][1]);
            rev[x]=0;
        }
    }
    void push_up(int x) {
        size[x]=size[ch[x][0]]+size[ch[x][1]]+1;
    }
    void build(int &x,int l,int r,int fa) {
        int m=l+r>>1;
        new_node(x,num[m],fa);
        if(l<m)
            build(ch[x][0],l,m-1,x);
        if(r>m)
            build(ch[x][1],m+1,r,x);
        push_up(x);
    }
    void init(int n) {
        root=tot=size[0]=0;
        for(int i=1; i<=n; ++i)
            scanf("%d",&num[i]);
        build(root,1,n,0);
        point=1;
    }
    int find(int rt,int k) {
        int x=rt;
        while(size[ch[x][0]]+1!=k) {
            push_down(x);
            if(k<=size[ch[x][0]])
                x=ch[x][0];
            else {
                k-=size[ch[x][0]]+1;
                x=ch[x][1];
            }
        }
        return x;
    }
    void split(int &x,int &y,int sz) {
        if(!sz) {
            y=x;
            x=0;
            return;
        }
        y=find(x,sz+1);
        splay(y);
        x=ch[y][0];
        ch[y][0]=0;
        push_up(y);
    }
    void split3(int &x,int &y,int &z,int l,int r) {
        split(x,z,r);
        split(x,y,l-1);
    }
    void join(int &x,int &y) {
        if(!x||!y) {
            x|=y;
            return;
        }
        x=find(x,size[x]);
        splay(x);
        ch[x][1]=y;
        pre[y]=x;
        push_up(x);
    }
    void join3(int &x,int y,int z) {
        join(y,z);
        join(x,y);
    }
    void evert() {
        if(point>1) {
            int x;
            split(root,x,point-1);
            swap(root,x);
            join(root,x);
            point=1;
        }
    }
    void plus(int v) {
        evert();
        int x,y;
        split3(root,x,y,point,point+k2-1);
        update_add(x,v);
        join3(root,x,y);
    }
    void reverse() {
        evert();
        int x,y;
        split3(root,x,y,point,point+k1-1);
        update_rev(x);
        join3(root,x,y);
    }
    void insert(int v) {
        evert();
        int x,y;
        split(root,x,point);
        new_node(y,v,0);
        join3(root,y,x);
    }
    void erase() {
        evert();
        int x,y;
        split3(root,x,y,point,point);
        join(root,y);
    }
    void move(int tag) {
        switch(tag) {
        case 1:
            if(--point==0)
                point=size[root];
            break;
        case 2:
            if(++point==size[root]+1)
                point=1;
            break;
        }
    }
    void query() {
        evert();
        int x,y;
        split3(root,x,y,point,point);
        printf("%d\n",key[x]);
        join3(root,x,y);
    }
} splay;
int main() {
    int n,m,v,cas=0;
    char op[10];
    while(~scanf("%d%d%d%d",&n,&m,&k1,&k2)&&(n||m||k1||k2)) {
        splay.init(n);
        printf("Case #%d:\n",++cas);
        while(m--) {
            scanf("%s",op);
            switch(op[0]) {
            case 'a':
                scanf("%d",&v);
                splay.plus(v);
                break;
            case 'r':
                splay.reverse();
                break;
            case 'i':
                scanf("%d",&v);
                splay.insert(v);
                break;
            case 'd':
                splay.erase();
                break;
            case 'm':
                scanf("%d",&v);
                splay.move(v);
                break;
            case 'q':
                splay.query();
                break;
            }
        }
    }
}
{% endhighlight %}

##Link-Cut Tree

动态树是维护森林信息的一类问题的总称。最常见的也是我唯一会的就是LCT；此外还有自适应Top Tree、全局平衡二叉树等。LCT可以维护多棵树（森林）的形态，并在$$O(\log n)$$的时间复杂度内维护链上信息；但LCT处理子树信息将会非常麻烦。它的核心操作是access函数，可以把某个节点到根的路径上所有点按照深度用Splay维护起来，从而结合evert函数（换跟操作）和splay操作可以实现对链的信息维护。LCT几乎可以实现除维护子树信息外以上的所有操作，同时有着优越的理论复杂度，但实际常数较大，很多不改变树形态的题用$$O(\log n)$$的LCT并不比$$O(\log^2n)$$的树链剖分套线段树更优越。以下是[HDU4010](http://acm.hdu.edu.cn/showproblem.php?pid=4010)的LCT实现。
{% highlight cpp linenos %}
#include<cstdio>
#include<cstring>
#include<algorithm>
using namespace std;
const int MAXN=300005;
struct LCT {
    int ch[MAXN][2],pre[MAXN],key[MAXN],rev[MAXN];
    int add[MAXN],vmax[MAXN];
    bool isroot(int x) {
        return !pre[x]||ch[pre[x]][0]!=x&&ch[pre[x]][1]!=x;
    }
    void rotate(int x) {
        int y=pre[x],f=ch[y][1]==x;
        ch[y][f]=ch[x][f^1];
        pre[ch[y][f]]=y;
        if(!isroot(y))
            ch[pre[y]][ch[pre[y]][1]==y]=x;
        pre[x]=pre[y];
        ch[x][f^1]=y;
        pre[y]=x;
        push_up(y);
    }
    void splay(int x) {
        push_down(x);
        while(!isroot(x)) {
            int y=pre[x],z=pre[y];
            if(isroot(y)) {
                push_down(y);
                push_down(x);
                rotate(x);
            } else {
                push_down(z);
                push_down(y);
                push_down(x);
                rotate((ch[z][1]==y)==(ch[y][1]==x)?y:x);
                rotate(x);
            }
        }
        push_up(x);
    }
    int access(int x) {
        int y=0;
        for(; x; x=pre[x]) {
            splay(x);
            ch[x][1]=y;
            push_up(x);
            y=x;
        }
        return y;
    }
    void evert(int x) {
        rev[access(x)]^=1;
        splay(x);
    }
    void push_up(int x) {
        vmax[x]=max(max(vmax[ch[x][0]],vmax[ch[x][1]]),key[x]);
    }
    void push_down(int x) {
        if(add[x]) {
            key[x]+=add[x];
            if(ch[x][0]) {
                add[ch[x][0]]+=add[x];
                vmax[ch[x][0]]+=add[x];
            }
            if(ch[x][1]) {
                add[ch[x][1]]+=add[x];
                vmax[ch[x][1]]+=add[x];
            }
            add[x]=0;
        }
        if(rev[x]) {
            if(ch[x][0])
                rev[ch[x][0]]^=1;
            if(ch[x][1])
                rev[ch[x][1]]^=1;
            swap(ch[x][0],ch[x][1]);
            rev[x]=0;
        }
    }
    int find_root(int x) {
        while(pre[x])
            x=pre[x];
        return x;
    }
    void link(int u,int v) {
        if(find_root(u)==find_root(v)) {
            puts("-1");
            return;
        }
        evert(u);
        pre[u]=v;
    }
    void cut(int u,int v) {
        if(u==v||find_root(u)!=find_root(v)) {
            puts("-1");
            return;
        }
        evert(u);
        access(v);
        splay(v);
        pre[ch[v][0]]=0;
        ch[v][0]=0;
        push_up(v);
    }
    void update(int u,int v,int w) {
        if(find_root(u)!=find_root(v)) {
            puts("-1");
            return;
        }
        evert(u);
        access(v);
        splay(v);
        add[v]+=w;
        vmax[v]+=w;
        push_down(v);
    }
    void query(int u,int v) {
        if(find_root(u)!=find_root(v)) {
            puts("-1");
            return;
        }
        evert(u);
        access(v);
        splay(v);
        printf("%d\n",vmax[v]);
    }
    struct graph {
        int head[MAXN],to[MAXN<<1],next[MAXN<<1];
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
    void dfs(int u,int fa) {
        for(int i=g.head[u]; ~i; i=g.next[i]) {
            int v=g.to[i];
            if(v!=fa) {
                dfs(v,u);
                pre[v]=u;
            }
        }
    }
    void init(int n) {
        int m,x,y;
        g.init();
        for(int i=1; i<n; ++i) {
            scanf("%d%d",&x,&y);
            g.add(x,y);
            g.add(y,x);
        }
        memset(ch,0,sizeof(ch));
        memset(pre,0,sizeof(pre));
        memset(rev,0,sizeof(rev));
        memset(add,0,sizeof(add));
        vmax[0]=0;
        for(int i=1; i<=n; ++i) {
            scanf("%d",&key[i]);
            vmax[i]=key[i];
        }
        dfs(1,0);
    }
} lct;
int main() {
    int n,q,op,x,y,w;
    while(~scanf("%d",&n)) {
        lct.init(n);
        scanf("%d",&q);
        while(q--) {
            scanf("%d",&op);
            switch(op) {
            case 1:
                scanf("%d%d",&x,&y);
                lct.link(x,y);
                break;
            case 2:
                scanf("%d%d",&x,&y);
                lct.cut(x,y);
                break;
            case 3:
                scanf("%d%d%d",&w,&x,&y);
                lct.update(x,y,w);
                break;
            case 4:
                scanf("%d%d",&x,&y);
                lct.query(x,y);
                break;
            }
        }
        putchar('\n');
    }
}
{% endhighlight %}

##不基于旋转的Treap

以上所有平衡树都利用旋转操作来调整树的高度，以保持严格或期望或均摊$$O(\log n)$$的单次操作复杂度，但基于旋转的结构有个很大的弊端是不支持可持久化。由此，范浩强（fanhq666）首先在[这篇Blog](http://fanhq666.blog.163.com/blog/static/819434262011021105212299/)里提出了不基于旋转的Treap。从Splay中我们已经可以意识到，只要设计出$$O(\log n)$$的split和merge操作，便可以在相同时间复杂度内完成平衡树的一切基本操作。算法和理论证明可以看范浩强的Blog和上面提到的陈立杰的[论文](http://www.doc88.com/p-8416874386976.html)。这样这种Treap已经可以完成Splay的一切操作，只是据不多的测试，并不比Splay快。以下是[POJ3580](http://poj.org/problem?id=3580)的Treap实现，可与上述Splay实现做对比。
{% highlight cpp linenos %}
#include<cstdio>
#include<algorithm>
using namespace std;
const int MAXN=200005;
int num[MAXN];
struct Treap {
    int tot,root;
    int ch[MAXN][2],pt[MAXN],size[MAXN];
    int key[MAXN],vmin[MAXN],add[MAXN],rev[MAXN];
    void init() {
        tot=0;
    }
    void new_node(int &x,int v) {
        x=++tot;
        ch[x][0]=ch[x][1]=0;
        size[x]=1;
        pt[x]=rand();
        key[x]=vmin[x]=v;
        add[x]=rev[x]=0;
    }
    void merge(int &p,int x,int y) {
        if(!x||!y) {
            p=x|y;
            return;
        }
        if(pt[x]<pt[y]) {
            push_down(x);
            merge(ch[x][1],ch[x][1],y);
            p=x;
        } else {
            push_down(y);
            merge(ch[y][0],x,ch[y][0]);
            p=y;
        }
        push_up(p);
    }
    void split(int p,int sz,int &x,int &y) {
        if(!sz) {
            x=0;
            y=p;
            return;
        }
        push_down(p);
        if(size[ch[p][0]]>=sz) {
            y=p;
            split(ch[p][0],sz,x,ch[y][0]);
        } else {
            x=p;
            split(ch[p][1],sz-size[ch[p][0]]-1,ch[x][1],y);
        }
        push_up(p);
    }
    void update_add(int x,int v) {
        if(x) {
            key[x]+=v;
            add[x]+=v;
            vmin[x]+=v;
        }
    }
    void update_rev(int x) {
        if(x) {
            swap(ch[x][0],ch[x][1]);
            rev[x]^=1;
        }
    }
    void push_down(int x) {
        if(add[x]) {
            update_add(ch[x][0],add[x]);
            update_add(ch[x][1],add[x]);
            add[x]=0;
        }
        if(rev[x]) {
            update_rev(ch[x][0]);
            update_rev(ch[x][1]);
            rev[x]=0;
        }
    }
    void push_up(int x) {
        size[x]=1;
        vmin[x]=key[x];
        if(ch[x][0]) {
            size[x]+=size[ch[x][0]];
            vmin[x]=min(vmin[x],vmin[ch[x][0]]);
        }
        if(ch[x][1]) {
            size[x]+=size[ch[x][1]];
            vmin[x]=min(vmin[x],vmin[ch[x][1]]);
        }
    }
    int build(int &x,int l,int r) {
        int m=l+r>>1;
        new_node(x,num[m]);
        if(l<m)
            build(ch[x][0],l,m-1);
        if(r>m)
            build(ch[x][1],m+1,r);
        push_up(x);
    }
    void plus(int l,int r,int v) {
        int x,y;
        split(root,l-1,root,x);
        split(x,r-l+1,x,y);
        update_add(x,v);
        merge(x,x,y);
        merge(root,root,x);
    }
    void reverse(int l,int r) {
        int x,y;
        split(root,l-1,root,x);
        split(x,r-l+1,x,y);
        update_rev(x);
        merge(x,x,y);
        merge(root,root,x);
    }
    void revolve(int l,int r,int k) {
        int x,y,p,q;
        k%=r-l+1;
        if(!k)
            return;
        split(root,l-1,root,x);
        split(x,r-l+1,x,y);
        split(x,r-l+1-k,p,q);
        merge(x,q,p);
        merge(x,x,y);
        merge(root,root,x);
    }
    void insert(int k,int v) {
        int x,y;
        new_node(x,v);
        split(root,k,root,y);
        merge(root,root,x);
        merge(root,root,y);
    }
    void erase(int k) {
        int x,y;
        split(root,k-1,root,x);
        split(x,1,x,y);
        merge(root,root,y);
    }
    int query(int l,int r) {
        int x,y,ret;
        split(root,l-1,root,x);
        split(x,r-l+1,x,y);
        ret=vmin[x];
        merge(x,x,y);
        merge(root,root,x);
        return ret;
    }
} treap;
int main() {
    int n,m,x,y,v;
    char op[10];
    while(~scanf("%d",&n)) {
        treap.init();
        for(int i=1; i<=n; ++i)
            scanf("%d",&num[i]);
        treap.build(treap.root,1,n);
        scanf("%d",&m);
        while(m--) {
            scanf("%s",op);
            switch(op[0]) {
            case 'A':
                scanf("%d%d%d",&x,&y,&v);
                treap.plus(x,y,v);
                break;
            case 'R':
                scanf("%d%d",&x,&y);
                if(op[3]=='E')
                    treap.reverse(x,y);
                else {
                    scanf("%d",&v);
                    treap.revolve(x,y,v);
                }
                break;
            case 'I':
                scanf("%d%d",&x,&v);
                treap.insert(x,v);
                break;
            case 'D':
                scanf("%d",&x);
                treap.erase(x);
                break;
            case 'M':
                scanf("%d%d",&x,&y);
                printf("%d\n",treap.query(x,y));
                break;
            }
        }
    }
}
{% endhighlight %}

##可持久化Treap

在拥有了不基于旋转的Treap之后，我们便有了可持久化的Treap。但Treap与其它平衡树不同的地方在于它利用随机权值维护平衡性，考虑一个不断复制子区间的情况，我们发现会出现大量权值相同的节点，从而很容易导致Treap失衡。面对这种情况，范浩强在[这篇Blog](http://fanhq666.blog.163.com/blog/static/81943426201161095456671/)里选择了用相似方式重写不基于旋转的AVL并可持久化的方式来规避。而在陈立杰的[论文](http://www.doc88.com/p-8416874386976.html)中则提到另一种方法，他注意到Treap的随机权值仅仅在merge时利用，于是直接弃掉随机权值，选择在merge时随机概率；在论文中陈立杰表示他无法证明复杂度但找不到可以卡住的数据，在吕凯风（VFleaKing）的群里也讨论过这个问题，当时郭晓旭（ftiasch）说他也无法证明，但实际表现一直良好所以可以直接用。以下是[UVa12538](http://uva.onlinejudge.org/index.php?option=com_onlinejudge&Itemid=8&page=show_problem&problem=3983)的可持久化Treap实现。
{% highlight cpp linenos %}
#include<cstdio>
#include<cstdlib>
#include<cstring>
using namespace std;
const int MAXN=50005;
const int MAXM=5000005;
int root[MAXN],vs,d;
struct Treap {
    int tot;
    int ch[MAXM][2],size[MAXM];
    char key[MAXM];
    bool hey(int x,int y) {
        return (long long)rand()*(size[x]+size[y])<(long long)size[x]*RAND_MAX;
    }
    void init() {
        tot=0;
    }
    void new_node(int &x,char v) {
        x=++tot;
        ch[x][0]=ch[x][1]=0;
        size[x]=1;
        key[x]=v;
    }
    void copy_node(int &x,int y) {
        if(!y) {
            x=0;
            return;
        }
        x=++tot;
        ch[x][0]=ch[y][0];
        ch[x][1]=ch[y][1];
        size[x]=size[y];
        key[x]=key[y];
    }
    void merge(int &p,int x,int y) {
        if(!x||!y) {
            p=0;
            if(x)
                copy_node(p,x);
            if(y)
                copy_node(p,y);
            return;
        }
        if(hey(x,y)) {
            copy_node(p,x);
            merge(ch[p][1],ch[x][1],y);
        } else {
            copy_node(p,y);
            merge(ch[p][0],x,ch[y][0]);
        }
        push_up(p);
    }
    void split(int p,int sz,int &x,int &y) {
        if(!sz) {
            x=0;
            copy_node(y,p);
            return;
        }
        if(size[ch[p][0]]>=sz) {
            copy_node(y,p);
            split(ch[p][0],sz,x,ch[y][0]);
            push_up(y);
        } else {
            copy_node(x,p);
            split(ch[p][1],sz-size[ch[p][0]]-1,ch[x][1],y);
            push_up(x);
        }
    }
    void push_up(int x) {
        size[x]=1;
        if(ch[x][0])
            size[x]+=size[ch[x][0]];
        if(ch[x][1])
            size[x]+=size[ch[x][1]];
    }
    void build(char str[],int &x,int l,int r) {
        int m=l+r>>1;
        new_node(x,str[m]);
        if(l<m)
            build(str,ch[x][0],l,m-1);
        if(r>m)
            build(str,ch[x][1],m+1,r);
        push_up(x);
    }
    void insert(int k,char str[]) {
        int x,y,z;
        build(str,x,0,strlen(str)-1);
        split(root[vs],k,y,z);
        merge(y,y,x);
        merge(root[++vs],y,z);
    }
    void erase(int k,int sz) {
        int x,y,z;
        split(root[vs],k-1,x,y);
        split(y,sz,y,z);
        merge(root[++vs],x,z);
    }
    void output(int x) {
        if(ch[x][0])
            output(ch[x][0]);
        putchar(key[x]);
        d+=key[x]=='c';
        if(ch[x][1])
            output(ch[x][1]);
    }
    void output(int v,int k,int sz) {
        int x,y,z;
        split(root[v],k-1,x,y);
        split(y,sz,y,z);
        output(y);
        putchar('\n');
    }
} treap;
int main() {
    int n,op,p,c,v;
    char s[105];
    treap.init();
    vs=d=0;
    scanf("%d",&n);
    while(n--) {
        scanf("%d",&op);
        switch(op) {
        case 1:
            scanf("%d%s",&p,s);
            treap.insert(p-d,s);
            break;
        case 2:
            scanf("%d%d",&p,&c);
            treap.erase(p-d,c-d);
            break;
        case 3:
            scanf("%d%d%d",&v,&p,&c);
            treap.output(v-d,p-d,c-d);
            break;
        }
    }
}
{% endhighlight %}

##树链剖分

树链剖分是非常神奇的东西，常见的轻重链剖分将一棵树划分成至多$$\log n$$条重链和若干条轻边，满足每个节点属于一条重链，从而将树上路径修改转化为至多$$\log n$$次线性修改，非常利于套用树状数组、线段树等各类数据结构。树链剖分的常数很小，且因着树链剖分的性质，我们发现越是退化的树（极端情况下成为一条链），树链剖分的效果越是好（极端情况下甚至是$$O(1)$$级的，因为只有很少的重链），以至于一些不涉及形态修改的树上路径维护题目，可以用树链剖分套线段树以$$O(\log^2n)$$的单次操作复杂度水过，且实际表现不输于单次操作$$O(\log n)$$但常数很大的LCT。常见轻重链剖分的初始化实现是两次dfs的，但dfs有两个问题，一是递归调用使得时间稍慢，二是有些题目有爆栈风险；所以我抄了bfs实现的很好用的交大板。

需要稍作说明的是，对于点权修改直接维护即可，对于边权修改，常规做法是选定一个根，将边权下垂到深度更大的节点上；换言之，每个点储存的权值是它与它的父节点之间的边权，根节点上没有权值。
{% highlight cpp linenos %}
int top[MAXN];    //top[p]表示编号为p的路径的顶端节点
int len[MAXN];    //len[p]表示路径p的长度
int belong[MAXN]; //belong[v]表示节点v所属的路径编号
int idx[MAXN];    //idx[v]表示节点v在其路径中的编号，按深度由深到浅依次标号
int dep[MAXN];    //dep[v]表示节点v的深度
int fa[MAXN];     //fa[v]表示节点v的父亲节点
int size[MAXN];   //size[v]表示以节点v为根的子树的节点个数
int que[MAXN];
bool vis[MAXN];
int n,cnt;        //n是点数，标号从1到n
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
    cnt=0;
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
            idx[u]=len[++cnt]=1;
            belong[u]=cnt;
            top[cnt]=u;
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
    for(int i=2; i<=cnt; ++i)
        fi[i]=fi[i-1]+len[i-1];
    for(int i=1; i<=n; ++i) {
        cid[i]=fi[belong[i]]+len[belong[i]]-idx[i];
        rank[cid[i]]=i;
    }
}
// 路径修改和查询依下面修改
int query(int x,int y) {
    int ret=0;
    while(belong[x]!=belong[y]) {
        if(dep[top[belong[x]]]<dep[top[belong[y]]])
            swap(x,y);
        ret=max(ret,query(cid[top[belong[x]]],cid[x],1,n,1));
        x=fa[top[belong[x]]];
    }
    if(dep[x]>dep[y])
        swap(x,y);
    ret=max(ret,query(cid[x],cid[y],1,n,1));
    /*边权如下
    if(x!=y)
        ret=max(ret,query(cid[x]+1,cid[y],1,n,1));
    */
    return ret;
}
{% endhighlight %}

但dfs实现的树链剖分就没有实际意义了么？非也。dfs实现有一些bfs实现做不到的神奇用法。比如，我们观察第一次dfs，可以发现这和倍增LCA的dfs部分几乎一致；所以稍作修改就可以无缝衔接LCA。更重要的是，我们观察第二次dfs，这次dfs对节点的新位置进行了标号（对应bfs的getcid函数），我们可以发现无论它以怎样的顺序进行dfs（先dfs重儿子再dfs其它子节点），得到的依旧是这棵树的一个dfs序。换句话说，这里处理出的剖分标号，同时也是dfs序标号。我们知道每棵子树的节点在dfs序中都是连续的一段，这样我们就可以同时维护树上路径信息（剖分部分复杂度$$O(\log n)$$）和子树信息（剖分部分复杂度$$O(1)$$）了。以下是[BZOJ3083](http://www.lydsy.com/JudgeOnline/problem.php?id=3083)的树链剖分套线段树实现，同时展现了dfs实现套用LCA和维护链上以及子树信息的做法，非常巧妙；bfs剖分和LCT无法处理这道题的子树询问（有论文阐述了LCT处理子树信息的做法，蒟蒻表示不会）。
{% highlight cpp linenos %}
#include<cstdio>
#include<cstring>
#include<algorithm>
using namespace std;
const int MAXN=100005;
const int maxd=18;
const int INF=0x7fffffff;
struct graph {
    int head[MAXN],tot;
    int to[MAXN<<1],next[MAXN<<1];
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
int top[MAXN],son[MAXN];
int dep[MAXN],fa[MAXN][maxd],size[MAXN];
int cid[MAXN],rank[MAXN],cnt;
void dfs1(int u) {
    size[u]=1;
    son[u]=-1;
    for(int i=1; i<maxd; ++i)
        fa[u][i]=fa[fa[u][i-1]][i-1];
    for(int i=g.head[u]; ~i; i=g.next[i]) {
        int v=g.to[i];
        if(v!=fa[u][0]) {
            dep[v]=dep[u]+1;
            fa[v][0]=u;
            dfs1(v);
            size[u]+=size[v];
            if(!~son[u]||size[v]>size[son[u]])
                son[u]=v;
        }
    }
}
void dfs2(int u,int tp) {
    top[u]=tp;
    cid[u]=++cnt;
    rank[cid[u]]=u;
    if(~son[u])
        dfs2(son[u],tp);
    for(int i=g.head[u]; ~i; i=g.next[i]) {
        int v=g.to[i];
        if(v!=son[u]&&v!=fa[u][0])
            dfs2(v,v);
    }
}
void split() {
    dfs1(1);
    cnt=0;
    dfs2(1,1);
}
int lca(int u,int v) {
    if(dep[u]<dep[v])
        swap(u,v);
    int k=dep[u]-dep[v];
    for(int i=0; i<maxd; ++i)
        if((1<<i)&k)
            u=fa[u][i];
    if(u==v)
        return u;
    for(int i=maxd-1; i>=0; --i)
        if(fa[u][i]!=fa[v][i]) {
            u=fa[u][i];
            v=fa[v][i];
        }
    return fa[u][0];
}
int n,root;
int a[MAXN];
#define lson l,m,rt<<1
#define rson m+1,r,rt<<1|1
int vmin[MAXN<<2],col[MAXN<<2];
void push_up(int rt) {
    vmin[rt]=min(vmin[rt<<1],vmin[rt<<1|1]);
}
void push_down(int rt) {
    if(col[rt]) {
        col[rt<<1]=col[rt<<1|1]=vmin[rt<<1]=vmin[rt<<1|1]=col[rt];
        col[rt]=0;
    }
}
void build(int l,int r,int rt) {
    col[rt]=0;
    if(l==r) {
        vmin[rt]=a[rank[l]];
        return;
    }
    int m=l+r>>1;
    build(lson);
    build(rson);
    push_up(rt);
}
void update(int L,int R,int val,int l,int r,int rt) {
    if(L<=l&&r<=R) {
        col[rt]=vmin[rt]=val;
        return;
    }
    push_down(rt);
    int m=l+r>>1;
    if(L<=m)
        update(L,R,val,lson);
    if(m<R)
        update(L,R,val,rson);
    push_up(rt);
}
int query(int L,int R,int l,int r,int rt) {
    if(L<=l&&r<=R)
        return vmin[rt];
    push_down(rt);
    int m=l+r>>1;
    int ret=INF;
    if(L<=m)
        ret=min(ret,query(L,R,lson));
    if(m<R)
        ret=min(ret,query(L,R,rson));
    return ret;
}
void modify(int x,int y,int d) {
    while(top[x]!=top[y]) {
        if(dep[top[x]]<dep[top[y]])
            swap(x,y);
        update(cid[top[x]],cid[x],d,1,n,1);
        x=fa[top[x]][0];
    }
    if(dep[x]>dep[y])
        swap(x,y);
    update(cid[x],cid[y],d,1,n,1);
}
int query(int rt) {
    if(rt==root)
        return query(1,n,1,n,1);
    int pre=lca(root,rt);
    if(pre!=rt)
        return query(cid[rt],cid[rt]+size[rt]-1,1,n,1);
    int depth=dep[root]-dep[rt]-1,tmp=root;
    for(int i=maxd-1; i>=0; --i)
        if(depth&(1<<i))
            tmp=fa[tmp][i];
    return min(query(1,cid[tmp]-1,1,n,1),query(cid[tmp]+size[tmp],n,1,n,1));
}
int main() {
    int m,u,v,opt,id;
    while(~scanf("%d%d",&n,&m)) {
        g.init();
        for(int i=1; i<n; ++i) {
            scanf("%d%d",&u,&v);
            g.add(u,v);
            g.add(v,u);
        }
        for(int i=1; i<=n; ++i)
            scanf("%d",&a[i]);
        split();
        build(1,n,1);
        scanf("%d",&root);
        while(m--) {
            scanf("%d",&opt);
            switch(opt) {
            case 1:
                scanf("%d",&root);
                break;
            case 2:
                scanf("%d%d%d",&u,&v,&id);
                modify(u,v,id);
                break;
            case 3:
                scanf("%d",&id);
                printf("%d\n",query(id));
                break;
            }
        }
    }
}
{% endhighlight %}

##KD-Tree

KD-Tree并不是一个在竞赛中常见的数据结构，相反在需要处理多维数据的场合有很多的实际应用；在ACM中主要用来维护多维第K近点对距离一类的信息。它的主要思想是在每一维上依次用一个超平面进行空间划分，将点集比较均匀地分割在各个区域内，结构上则是一棵二叉树，且与线段树的形态和构造方法都有些类似。经过改造的KD-Tree一般可以做到$$O(\log n)$$的单点插入，以及$$O(n^{1-\frac1D})$$的询问操作，其中D是维数；可见维数越大KD-Tree越慢；实质上KD-Tree本身也并不是一个速度很快的数据结构，可以说只为解决特定问题而生。询问距离一个点的前K近点一般需要用一个优先队列进行询问时的维护，写起来其实很简单。以下是[HDU4347](http://acm.hdu.edu.cn/showproblem.php?pid=4347)的代码。
{% highlight cpp linenos %}
#include<cstdio>
#include<algorithm>
#include<queue>
using namespace std;
const int MAXN=50005;
const int INF=~0U>>1;
const int DIM=5;
#define lson l,m-1,dep+1
#define rson m+1,r,dep+1
int cur,K;
struct point {
    int x[DIM];
    bool operator<(const point &oth) const {
        return x[cur]<oth.x[cur];
    }
    void output() {
        for(int i=0; i<K; ++i)
            printf("%d%c",x[i],i<K-1?' ':'\n');
    }
} vec[MAXN],origin[MAXN],pt,ans[10];
inline int sqr(int x) {
    return x*x;
}
int dist(const point &a,const point &b) {
    int ret=0;
    for(int i=0; i<K; ++i)
        ret+=sqr(a.x[i]-b.x[i]);
    return ret;
}
void build(int l,int r,int dep=0) {
    if(l>=r)
        return;
    int m=l+r>>1;
    cur=dep%K;
    nth_element(vec+l,vec+m,vec+r+1);
    build(lson);
    build(rson);
}
priority_queue<pair<int,point> > pq;
void query(const point &x,int k,int l,int r,int dep=0) {
    if(l>r)
        return;
    int m=l+r>>1,cur=dep%K;
    pair<int,point> tmp(dist(x,vec[m]),vec[m]);
    if(pq.size()<k)
        pq.push(tmp);
    else if(pq.top().first>tmp.first) {
        pq.pop();
        pq.push(tmp);
    }
    if(x.x[cur]<vec[m].x[cur]) {
        query(x,k,lson);
        if(pq.top().first>sqr(x.x[cur]-vec[m].x[cur]))
            query(x,k,rson);
    } else {
        query(x,k,rson);
        if(pq.top().first>sqr(x.x[cur]-vec[m].x[cur]))
            query(x,k,lson);
    }
}
int main() {
    int n,t,m;
    while(~scanf("%d%d",&n,&K)) {
        for(int i=1; i<=n; ++i) {
            for(int j=0; j<K; ++j)
                scanf("%d",&origin[i].x[j]);
            vec[i]=origin[i];
        }
        build(1,n);
        scanf("%d",&t);
        while(t--) {
            for(int i=0; i<K; ++i)
                scanf("%d",&pt.x[i]);
            scanf("%d",&m);
            query(pt,m,1,n);
            for(int i=0; i<m; ++i) {
                ans[i]=pq.top().second;
                pq.pop();
            }
            printf("the closest %d points are:\n", m);
            for(int i=m-1; i>=0; --i)
                ans[i].output();
        }
    }
}
{% endhighlight %}

以上代码的预处理复杂度是$$O(n)$$，但并不支持点的插入和删除。由此我找到另外一份风格迥异的模板，较好地解决了这个问题。插入其实和平衡树有些类似，而删除则是通过删除标记完成。
{% highlight cpp linenos %}
#define lson kdt[rt].ls,dep+1
#define rson kdt[rt].rs,dep+1
struct kdnode {
    int ls,rs,x[DIM];
    bool flag; //删点标记
} kdt[MAXN];
inline long long sqr(int x) {
    return (long long)x*x;
}
long long dist(const kdnode &a,const kdnode &b) {
    long long ret=0;
    for(int i=0; i<DIM; ++i)
        ret+=sqr(a.x[i]-b.x[i]);
    return ret;
}
int root,tot;
void init() {
    tot=0;
    root=-1;
}
int add(int pt[]) {
    kdt[tot].flag=false;
    kdt[tot].ls=kdt[tot].rs=-1;
    for(int i=0; i<DIM; ++i)
        kdt[tot].x[i]=pt[i];
    return tot++;
}
void insert(int pt[],int rt,int dep=0) {
    dep%=DIM;
    if(pt[dep]<kdt[rt].x[dep]) {
        if(!~kdt[rt].ls)
            kdt[rt].ls=add(pt);
        else
            insert(pt,lson);
    } else {
        if(!~kdt[rt].rs)
            kdt[rt].rs=add(pt);
        else
            insert(pt,rson);
    }
}
//求最近点距离
long long query(const kdnode &pt,int rt,int dep=0) {
    if(!~rt)
        return INF;
    dep%=DIM;
    long long ret=INF,tmp=sqr(kdt[rt].x[dep]-pt.x[dep]);
    if(!kdt[rt].flag)
        ret=dist(kdt[rt],pt);
    if(pt.x[dep]<=kdt[rt].x[dep]) {
        ret=min(ret,query(pt,lson));
        if(tmp<ret)
            ret=min(ret,query(pt,rson));
    }
    if(pt.x[dep]>=kdt[rt].x[dep]) {
        ret=min(ret,query(pt,rson));
        if(tmp<ret)
            ret=min(ret,query(pt,lson));
    }
    return ret;
}
//查询区间内有多少个点
int query(int pt1[],int pt2[],int rt,int dep=0) {
    if(!~rt)
        return 0;
    dep%=DIM;
    int ret=0,cur;
    for(cur=0; cur<DIM; ++cur)
        if(kdt[rt].x[cur]<pt1[cur]||kdt[rt].x[cur]>pt2[cur])
            break;
    if(cur==DIM)
        ++ret;
    if(pt2[dep]<kdt[rt].x[dep])
        ret+=query(pt1,pt2,lson);
    else if(pt1[dep]>=kdt[rt].x[dep])
        ret+=query(pt1,pt2,rson);
    else {
        ret+=query(pt1,pt2,lson);
        ret+=query(pt1,pt2,rson);
    }
    return ret;
}
{% endhighlight %}

##划分树

划分树也是基于线段树的一种数据结构，通常用于处理无修改询问区间第k小值。它的建树过程与快排类似，基本上相当于将快排的过程用一棵树存储下来。划分树的代码不算长，但并不是很好写；一开始我以为可持久化权值线段树完全可以代替划分树（好写，速度快，只是内存占用稍大），但[HDU3473](http://acm.hdu.edu.cn/showproblem.php?pid=3473)教我做人= =因为这道题需要维护的信息在可持久化权值线段树下有点麻烦，至少不够直观（也可能是我比较弱的缘故）。但除此之外，划分树也没有更多的扩展性应用了。
{% highlight cpp linenos %}
#define lson l,m,dep+1
#define rson m+1,r,dep+1
int part[20][MAXN]; //表示每层每个位置的值
int sod[MAXN]; //已经排序好的数
int tol[20][MAXN]; //tol[p][i] 表示第i层从1到i有数分入左边
void build(int l,int r,int dep) {
    if(l==r)
        return;
    int m=l+r>>1,cnt=m-l+1; //表示等于中间值而且被分入左边的个数
    for(int i=l; i<=r; ++i)
        if(part[dep][i]<sod[m])
            --cnt;
    int lpos=l,rpos=m+1;
    for(int i=l; i<=r; ++i) {
        if(part[dep][i]<sod[m])
            part[dep+1][lpos++]=part[dep][i];
        else if(part[dep][i]==sod[m]&&cnt>0) {
            part[dep+1][lpos++]=part[dep][i];
            --cnt;
        } else
            part[dep+1][rpos++]=part[dep][i];
        tol[dep][i]=tol[dep][l-1]+lpos-l;
    }
    build(lson);
    build(rson);
}
//查询区间第k大的数
int query(int L,int R,int k,int l,int r,int dep) {
    if(L==R)
        return part[dep][L];
    int m=l+r>>1,cnt=tol[dep][R]-tol[dep][L-1];
    if(cnt>=k) {
        int tl=l+tol[dep][L-1]-tol[dep][l-1],tr=tl+cnt-1;
        return query(tl,tr,k,lson);
    } else {
        int tr=R+tol[dep][r]-tol[dep][R],tl=tr-(R-L-cnt);
        return query(tl,tr,k-cnt,rson);
    }
}
{% endhighlight %}

##左偏树

左偏树也不是一个很常用的数据结构，它其实是可并堆的一种实现，可以在$$O(\log n)$$的时间内实现堆的push、pop和两个堆的合并操作，以及$$O(1)$$时间的取堆顶操作。不常见的原因是竞赛中很少遇到需要将堆进行合并的需求，而在掌握了Treap、Splay等平衡树之后，也可以用启发式合并的办法实现$$O(\log^2n)$$甚至$$O(\log n)$$（Fotile说Splay按某种遍历序合并可以达到）的合并复杂度。但左偏树的编程复杂度很小，常数也不错；它常常搭配并查集使用（而且也很好搭配在一起）。以下是[HDU1512](http://acm.hdu.edu.cn/showproblem.php?pid=1512)的左偏树+并查集做法。
{% highlight cpp linenos %}
#include<cstdio>
#include<algorithm>
using namespace std;
const int MAXN=100005;
int val[MAXN],ls[MAXN],rs[MAXN],dep[MAXN],fa[MAXN];
void init(int n) {
    for(int i=1; i<=n; ++i) {
        scanf("%d",&val[i]);
        ls[i]=rs[i]=dep[i]=0;
        fa[i]=i;
    }
}
int find(int x) {
    if(fa[x]!=x)
        fa[x]=find(fa[x]);
    return fa[x];
}
int merge(int x,int y) {
    if(!x||!y)
        return x|y;
    if(val[x]<val[y])
        swap(x,y);
    rs[x]=merge(rs[x],y);
    fa[rs[x]]=x;
    if(dep[ls[x]]<dep[rs[x]])
        swap(ls[x],rs[x]);
    dep[x]=dep[rs[x]]+1;
    return x;
}
int push(int x,int y) {
    return merge(x,y);
}
int pop(int x) {
    int a=ls[x],b=rs[x];
    ls[x]=rs[x]=dep[x]=0;
    fa[x]=x;
    fa[a]=a;
    fa[b]=b;
    return merge(a,b);
}
int main() {
    int n,m,x,y;
    while(~scanf("%d",&n)) {
        init(n);
        scanf("%d",&m);
        while(m--) {
            scanf("%d%d",&x,&y);
            int a=find(x),b=find(y);
            if(a==b)
                puts("-1");
            else {
                val[a]>>=1;
                val[b]>>=1;
                a=push(pop(a),a);
                b=push(pop(b),b);
                printf("%d\n",val[merge(a,b)]);
            }
        }
    }
}
{% endhighlight %}

##笛卡尔树

也是一个不太有名的数据结构，但其实很“常见”。考虑一个键值对的序列，当键与键，值与值之间互不相同时，它们可以唯一地构成这样一棵二叉树：key在中序遍历时呈升序，满足二叉查找树性质；父节点的value大于子节点的value，满足堆的性质。于是我们发现，Treap恰好是将value随机从而保证节点期望深度的笛卡尔树。一个键值对序列的笛卡儿树可以$$O(n)$$时间内构造出来，有一个巧妙的应用是[POJ2559](http://poj.org/problem?id=2559)和[HDU1506](http://acm.hdu.edu.cn/showproblem.php?pid=1506)，虽然一般做法是用单调栈来搞。以下是[POJ2201](http://poj.org/problem?id=2201)（裸笛卡儿树构造题）的实现。
{% highlight cpp linenos %}
#include<cstdio>
#include<algorithm>
using namespace std;
const int MAXN=50005;
int idx[MAXN],n;
struct Cartesian_Tree {
    int root,key[MAXN],val[MAXN],ch[MAXN][2],pre[MAXN];
    void init() {
        for(int i=1; i<=n; ++i) {
            scanf("%d%d",&key[i],&val[i]);
            ch[i][0]=ch[i][1]=pre[i]=0;
        }
    }
    void build() {
        static int st[MAXN];
        int top=-1;
        for(int i=1; i<=n; ++i) {
            int k=top;
            while(k>=0&&val[st[k]]>val[idx[i]])
                --k;
            if(~k) {
                pre[idx[i]]=st[k];
                ch[st[k]][1]=idx[i];
            }
            if(k<top) {
                pre[st[k+1]]=idx[i];
                ch[idx[i]][0]=st[k+1];
            }
            st[++k]=idx[i];
            top=k;
        }
        root=st[0];
    }
} ct;
bool cmp(int x,int y) {
    return ct.key[x]<ct.key[y];
}
int main() {
    while(~scanf("%d",&n)) {
        ct.init();
        for(int i=1; i<=n; ++i)
            idx[i]=i;
        sort(idx+1,idx+n+1,cmp);
        ct.build();
        puts("YES");
        for(int i=1; i<=n; ++i)
            printf("%d %d %d\n",ct.pre[i],ct.ch[i][0],ct.ch[i][1]);
    }
}
{% endhighlight %}

基本上在ACM竞赛中常见的数据结构就这些了，这里的数据结构大都用于处理一些通用数据，而对于一些特定数据的没有涉及，比如处理字符串的Trie树、后缀数组、自动机等。其余的一些数据结构在学习过程中会酌情更新。