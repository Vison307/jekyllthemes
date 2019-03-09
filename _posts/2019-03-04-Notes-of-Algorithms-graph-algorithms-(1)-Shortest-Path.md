# 算法笔记——图论（1）单源最短路

* 定义

  最短路： 给定两个顶点，以这两个顶点为起点和终点的所有路径中，边的权值和最小的路径。

  单源最短路：固定一个起点，求它到其他所有点的最短路的距离。

若终点也固定，那么这种问题叫做两点之间的最短路问题。因为解决单源最短路问题的复杂度和解决两点之间的最短路问题的时间复杂度是一样的，所以求两点之间的最短路问题通常也当作单源最短路问题来处理。

* 算法

  1. dijkstra（狄克斯特拉）

  设起点为s，终点为t。图中总共有E条边，V个结点。

  设d[V]，其中d[i] = 从起点到编号为i的结点的最短路径。

  初始化d[i] = INF, d[s] = 0;

  设used[V]，其中used[i] = 编号为i的节点有没有使用过

  <br>

  第一步：从所有未使用过的节点中挑出d[i]最小的节点v，并将该节点置为使用过

  第二步：如果所有节点均使用过，说说明算法运行完成，退出循环。否则进入第三步

  第三步：遍历当前节点v的所有字节点，若从起点到子结点的最短路长度大于当前节点的最短路的(d[v])长度加上从当前节点到子结点的最短路长度（d[v.to] > d[v] + cost\[v][v.to] )，则更新该子结点的最短路

  第四步：进入第一步循环

  <br>

  时间复杂度O($V ^ 2$)

  注意到第一步挑出d[i]最小的节点v，这一步我们可以用堆来维护每个节点的d[i]，这样堆中的元素有O(V)个，更新和取出数值的操作共有O(E)次，所以时间复杂度是O(E logV) 

  2. SPFA

     SPFA是Bellman-Ford基于队列优化的算法，最坏时间复杂度O（VE），平均时间复杂度O(kE) （k在2左右）。但是由于2018年NOIP T1出题人卡了SPFA，导致100 -> 60，所以

     ​	众所周知，SPFA，它死了。

     当没有负权边的时候还是用heap优化的dijkstra算法比较好。当有负权边时，由于SPFA能够判断负环，所以是能用SPFA的。

     <br>

     SPFA算法设立一个先进先出的队列用来保存待优化的结点。

     首先将起点s压入队列中。

     第一步：取出队首节点v

     第二步：历当前节点v的所有字节点，若从起点到子结点的最短路长度大于当前节点的最短路的(d[v])长度加上从当前节点到子结点的最短路长度（d[v.to] > d[v] + cost\[v][v.to] )，并且v不在队列中。则更新该子结点的最短路，并且将该子结点放入队尾。并记录更新的次数，若更新次数大于n，那么说明有负环，返回false

     第三步：重复第一二步操作直到队列为空

  #### 例题

  ## 洛谷P3371&&P4779[模板]最小生成树

  * 题目描述

    > 如题，给出一个有向图，请输出从某一点出发到所有点的最短路径长度。

  * 输入输出格式

    * 输入格式：

    > 第一行包含三个整数N、M、S，分别表示点的个数、有向边的个数、出发点的编号。
    >
    > 接下来M行每行包含三个整数Fi、Gi、Wi，分别表示第i条有向边的出发点、目标点和长度。

    * 输出格式：

    > 一行，包含N个用空格分隔的整数，其中第i个整数表示从点S出发到点i的最短路径长度（若S=i则最短路径长度为0，若从点S无法到达点i，则最短路径长度为2147483647）

  * 输入输出样例

    * 输入样例#1：

    > ```
    > 4 6 1
    > 1 2 2
    > 2 3 2
    > 2 4 1
    > 1 3 5
    > 3 4 3
    > 1 4 4
    > ```

    * 输出样例#1：

    > ``` cpp
    > 0 2 4 3
    > ```

* dijkstra

  ````cpp
  //dijkstra
  //P3371 2086ms P4799 5TLE
  #include<bits/stdc++.h>
  using namespace std;
  const int INF = 2147483647;
  const int N = 1e4+5;
  const int M = 5e5+5;
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
      int next;
      int to;
      int cost = INF;
  }e[M];
  int tail[N];
  int cnt = 0;
  void add_edge(int f, int t, int w){
      e[cnt].to = t;
      e[cnt].cost = w;
      e[cnt].next = tail[f];
      tail[f] = cnt;
      ++cnt;
  }
  bool used[N];
  long long d[N];
  int main(void){
  	memset(used, false, sizeof(used));
  	memset(tail, -1, sizeof(tail));
  	fill(d, d+N, INF);
  	int n = read(), m = read(), s = read();//点的个数，有向边个数，出发点标号
  
  	for(int i = 0; i < m; i++){
  		int f = read(), g = read(), w = read();
  		add_edge(f, g, w);
  	}
  
  	d[s] = 0;
  	while(true){
  		int v = -1;
  		//对于每一个没有操作过的点
  		for(int i = 1; i <= n; i++){
  			if(!used[i] && (v == -1 || d[i] < d[v]))	v = i;//取距离最小的点
  		}
  		if(v == -1)	break;
  		used[v] = true;
  
  		for(int i = tail[v]; ~i; i = e[i].next){
  			d[e[i].to] = min(d[e[i].to], d[v] + e[i].cost);
  		}
  	}
  	for(int i = 1; i <= n; i++){
  			printf("%lld ", d[i]);
  	}
  	return 0;
  }
  ````

  * dijkstra+heap

  ````cpp
  //dijkstra+heap
  //P3371 207ms P4799 177ms
  #include<bits/stdc++.h>
  using namespace std;
  const int N = 1e5+5;//点的个数
  const int M = 5e5+5;//边的个数
  const int INF = 0x7fffffff;
  //读入优化
  int read(){
      char c = getchar();
      int ans = 0, sign = 1;
      while(c < '0' || c > '9'){
          if(c == '-')    sign = -1;
          c = getchar();
      }
      while(c >= '0' && c <= '9'){
          ans = ans * 10 + c - '0';
          c = getchar();
      }
      return ans * sign;
  }
  int d[N];
  //链式前向星
  int cnt = 0;
  int tail[N];
  struct edge{
      int to;
      int cost;
      int next;
  }e[M];
  void add_edge(int f, int t, int w){
      e[cnt].to = t;
      e[cnt].cost = w;
      e[cnt].next = tail[f];
      tail[f] = cnt;
      cnt++;
  }
  //heep优化
  struct node{
      int w;//权值
      int now;//当前顶点的编号
      inline bool operator < (const node& n)const{
          return this->w > n.w;
      }
  };
  priority_queue<node> que;
  int main(void){
      memset(tail, -1, sizeof(tail));//链式前向星初始化
      fill(d, d+N, INF);
  
      int n = read(), m = read(), s = read();
      for(int i = 0; i < m; i++){
          int f = read(), g = read(), w = read();
          add_edge(f, g, w);
      }
      d[s] = 0;
      que.push((node){0, s});
  
      while(!que.empty()){
          node n = que.top(); que.pop();
          int v = n.now;//当前点
          if(d[v] < n.w)  continue;
          for(int i = tail[v]; ~i; i = e[i].next){
              int t = e[i].to;
              if(d[t] > d[v] + e[i].cost){
                  d[t] = d[v] + e[i].cost;
                  que.push((node){d[t], t});
              }
          }
      }
      for(int i = 1; i <= n; i++){
          printf("%d ", d[i]);
      }
      return 0;
  ````

  * SPFA

  ````cpp
  //SPFA
  //P3371 289ms P4779 3TLE
  #include<bits/stdc++.h>
  using namespace std;
  const int N = 1e5+5;//点的个数
  const int M = 5e5+5;//边的个数
  const int INF = 0x7fffffff;
  //读入优化
  int read(){
      char c = getchar();
      int ans = 0, sign = 1;
      while(c < '0' || c > '9'){
          if(c == '-')    sign = -1;
          c = getchar();
      }
      while(c >= '0' && c <= '9'){
          ans = ans * 10 + c - '0';
          c = getchar();
      }
      return ans * sign;
  }
  int d[N];
  bool vis[N];
  //int in[N];//判断负环
  //链式前向星
  int cnt = 0;
  int tail[N];
  struct edge{
      int next;
      int to;
      int cost;
  }e[M];
  void add_edge(int f, int t, int w){
      e[cnt].to = t;
      e[cnt].cost = w;
      e[cnt].next = tail[f];
      tail[f] = cnt;
      cnt++;
  }
  //SPFA
  queue<int> que;
  int main(void){
      memset(tail, -1, sizeof(tail));//链式前向星初始化
      memset(vis, false, sizeof(vis));
  //    memset(in, 0, sizeof(in));
      fill(d, d+N, INF);
  
      int n = read(), m = read(), s = read();
      for(int i = 0; i < m; i++){
          int f = read(), g = read(), w = read();
          add_edge(f, g, w);
      }
  
      d[s] = 0;
      vis[s] = true;
      que.push(s);
      while(!que.empty()){
          int cur = que.front(); que.pop();
          vis[cur] = false;
          for(int i = tail[cur]; ~i; i = e[i].next){
              int t = e[i].to;
              if(d[t] > d[cur] + e[i].cost){
                  d[t] = d[cur] + e[i].cost;
                  if(!vis[t]){
                      que.push(t);
                      vis[t] = true;
                     // if(++in[t] > n)   return false; //有负环
                  }
              }
          }
      }
      for(int i = 1; i <= n; i++){
          printf("%d ", d[i]);
      }
      return 0;
  }
  ````

  从运行结果来看，dijkstra+heap优化速度最快，其次是SPFA，最后是不带优化的dijkstra。

