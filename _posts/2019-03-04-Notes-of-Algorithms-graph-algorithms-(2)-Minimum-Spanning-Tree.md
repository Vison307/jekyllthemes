# 算法笔记——图论（2） 最小生成树

* 定义

  1. 生成树

     给定一个无向图，如果它的某个子图中任意两个顶点都互相连通，并且这个子图是一棵树，那么这颗树就叫做生成树（spanning tree）.

  2. 最小生成树

     在一个无向图所有的生成树中，边权之和最小的生成树叫做这个图的最小生成树（MST，Minimum Spanning Tree）.

显然，生成树是否存在和图是否联通是等价的。在这里我们假设所有图均是联通的。

* 算法

  常见求解最小生成树的算法有Prim和Kruskal算法。

  1. Prim算法

     Prim算法和Dijkstra算法十分相似。它们都是从某个顶点出发，不断添加边的算法。

     算法步骤：

     首先 假设有一棵只包含一个顶点v的树T。

     然后 我们贪心地选取T和其他顶点相连的最小权值的边，将其加入T中

     最后 不断重复这个操作，直到所有顶点都被访问过。

     和Dijkstra算法相似，我们也能使用堆来维护权值最小的边，这样算法的时间复杂度就是O（E logV）.

  2. Kruskal算法

     Kruskal算法贪心地选取未访问过的边中边权最小的边，将其加入生成树中。若加入某条边时，树中出现了环（即它不再成为一棵树），那么这条边就不应该加入。

     算法步骤：

     第一步： 选择当前边权最小的边

     第二步： 判断讲该边加入T中是否会成环，若会成环则不添加这条边，否则将这条边加入T中

     第三步： 不断重复如上操作，直到所有边均被访问过。

     其中最重要的是判断将边加入T中是否会成环。

     假设我们要把e = （v, u)加入T中。如果加入之前v和u不在同一个连通分量里（即v和u不相连），那么加入e也不会产生环。繁殖，若v和u已经在同一个连通分量里，那么加入e一定会产生环。

     我们可以使用并查集高效地判断两个结点是否属于同一个联通分量：若两个结点的根节点相同，那么这两个结点一定属于同一个联通分量，用代码表述则是：

     ```cpp
     init_union_find(V);//V表示结点的个数
     //如果u和v的根节点相同
     if(find(e.u) == find(e.v)){
         continue;
     }
     else{
         unite(u, v);//将u，v相连
         res += e.cost;//求最小生成树的边权之和
     }
     ```

     Kruskal算法在排序上最费时，时间复杂度为O（E logV）.

* 例题

  ## 洛谷P3366 【模板】 最小生成树

  * 题目描述

    > 如题，给出一个无向图，求出最小生成树，如果该图不连通，则输出orz

  * 输入输出格式

    * 输入格式

    > 第一行包含两个整数N、M，表示该图共有N个结点和M条无向边。（N<=5000，M<=200000）
    >
    > 接下来M行每行包含三个整数Xi、Yi、Zi，表示有一条长度为Zi的无向边连接结点Xi、Yi

    * 输入格式

    > 输出包含一个数，即最小生成树的各边的长度之和；如果该图不连通则输出orz

  * 输入输出样例

    * 输入样例#1：

    > ```
    > 4 5
    > 1 2 2
    > 1 3 2
    > 1 4 3
    > 2 3 4
    > 3 4 3
    > ```

    * 输出样例#1：

    > ```
    > 7
    > ```

  * 说明

    * 时空限制：

      100ms, 128M

    * 数据规模：

      对于20%的数据：N<=5，M<=20

      对于40%的数据：N<=50，M<=2500

      对于70%的数据：N<=500，M<=10000

      对于100%的数据：N<=5000，M<=200000

    * 样例解释：

      [![kjU2dg.md.png](https://s2.ax1x.com/2019/03/05/kjU2dg.md.png)](https://imgchr.com/i/kjU2dg)

      所以最小生成树总边权为2+2+3=7.

* Answer：

  * Prim:

    ```cpp
    //P3366 239ms
    #include<bits/stdc++.h>
    using namespace std;
    const int N = 5005;
    const int M = 200005;
    int read(){
        char c = getchar();
        int ans = 0, sign = 1;
        while(c < '0' || c > '9'){
            if(c == '-')    sign = -1;
            c = getchar();
        }
        while(c >= '0' && c <= '9'){
            ans = 10*ans + c - '0';
            c = getchar();
        }
        return sign * ans;
    }
    //链式前向星
    int cnt = 0;
    int tail[N];
    struct edge{
        int next;
        int to;
        int cost;
    }e[M*2];
    void add_edge(int f, int t, int w){
        e[cnt].to = t;
        e[cnt].cost = w;
        e[cnt].next = tail[f];
        tail[f] = cnt;
        ++cnt;
    }
    //最小生成树
    int mincost[N];
    int used[N];
    int res = 0;
    int main(void){
        memset(tail, -1, sizeof(tail));
        memset(mincost, 0x4f, sizeof(mincost));
        memset(used, false, sizeof(used));
    
        int n = read(), m = read();
    
        for(int i = 0; i < m; i++){
            int x = read(), y = read(), z = read();
            add_edge(x, y, z);
            add_edge(y, x, z);
        }
    
        mincost[1] = 0;
        int count = 0;
        while(true){
            int v = -1;
            for(int i = 1; i <= n; i++){
                if(!used[i] && (v == -1 || mincost[i] < mincost[v])) v = i;
            }
    
            if(v == -1) break;
            used[v] = true;
            count++;
            res += mincost[v];
            for(int i = tail[v]; ~i; i = e[i].next){
                int temp = e[i].to;
                mincost[temp] = min(mincost[temp], e[i].cost);
            }
        }
        if(count != n)  printf("orz\n");
        else printf("%d\n", res);
        return 0;
    }
    ```

  * Prim+heap:

    ```cpp
    //P3366 95ms
    #include<bits/stdc++.h>
    using namespace std;
    const int N = 5005;
    const int M = 200005;
    //读入优化
    int read(){
        char c = getchar();
        int ans = 0, sign = 1;
        while(c < '0' || c > '9'){
            if(c == '-')    sign = -1;
            c = getchar();
        }
        while(c >= '0' && c <= '9'){
            ans = 10*ans + c - '0';
            c = getchar();
        }
        return sign * ans;
    }
    //链式前向星
    int cnt = 0;
    int tail[N];
    struct edge{
        int next;
        int to;
        int cost;
    }e[M*2];
    void add_edge(int f, int t, int w){
        e[cnt].to = t;
        e[cnt].cost = w;
        e[cnt].next = tail[f];
        tail[f] = cnt;
        ++cnt;
    }
    //最小生成树
    int used[N];
    int mincost[N];
    int res = 0;
    //heap优化
    struct node{
        int w;
        int now;
        inline bool operator <(const node& n)const{
            return this->w > n.w;
        }
    };
    priority_queue<node> que;
    
    int main(void){
        memset(tail, -1, sizeof(tail));
        memset(mincost, 0x4f, sizeof(mincost));
        memset(used, false, sizeof(used));
        int n = read(), m = read();
    
        for(int i = 0; i < m; i++){
            int x = read(), y = read(), z = read();
            add_edge(x, y, z);
            add_edge(y, x, z);
        }
    
        mincost[1] = 0;
        int count = 0;
        que.push((node){0, 1});
        while(!que.empty()){
            node n = que.top(); que.pop();
            int v = n.now;
            if(n.w != mincost[v])    continue ;
            used[v] = true;
            res += mincost[v];
            count++;
            for(int i = tail[v]; ~i; i = e[i].next){
                int temp = e[i].to;
                if(!used[temp] && mincost[temp] > e[i].cost){
                    mincost[temp] = e[i].cost;
                    que.push((node){mincost[temp], temp});
                }
            }
        }
        if(count != n)  printf("orz\n");
        else printf("%d\n", res);
        return 0;
    }
    ```

  * Kruskal

    ```cpp
    //P3366 109ms
    #include<bits/stdc++.h>
    using namespace std;
    const int E = 200005;
    const int V = 5005;
    int read(){
        char c = getchar();
        int ans = 0, sign = 1;
        while(c < '0' || c > '9'){
            if(c == '-')    sign = -1;
            c = getchar();
        }
        while(c >= '0' && c <= '9'){
            ans = 10 * ans + c - '0';
            c = getchar();
        }
        return sign * ans;
    }
    struct edge{
        int u;
        int v;
        int cost;
    } es[E];
    bool cmp(const edge& e1, const edge& e2){
        return e1.cost < e2.cost;
    }
    
    int fa[V];
    int find(int x){
        if(fa[x] == x)  return x;
        else    return fa[x] = fa[find(fa[x])];
    }
    
    int main(void){
        int n = read(), m = read();
        for(int i = 0; i < m; i++){
            int x = read(), y = read(), z = read();
            es[i].u = x;
            es[i].v = y;
            es[i].cost = z;
        }
    
        for(int i = 0; i < n; i++){
            fa[i] = i;
        }
    
        sort(es, es+m, cmp);
        int res = 0;
        int cnt = 0;
        for(int i = 0; i < m; i++){
            int eu = find(es[i].u), ev = find(es[i].v);
            int cost = es[i].cost;
            //如果不成环
            if(eu != ev){
                fa[eu] = ev;
                res += cost;
                cnt++;
            }
        }
        if(cnt != n-1){
            printf("orz\n");
        }
        else{
            printf("%d\n", res);
        }
    
        return 0;
    }
    ```

    由代码的运行时间可见，加入堆优化的Prim和Kruskal算法最快，在运行时间方面相差在10%左右，可以认为是误差所致。而裸的Prim算法则较慢，运行时间在堆优化Prim和Kruskal算法的两倍左右。

