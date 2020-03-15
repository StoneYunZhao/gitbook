# Graph

## 概念

**图**（Graph）是一种非线性表结构。图中的元素叫做**顶点**（vertex），图中的顶点可以与任意其它顶点建立连接，这种连接叫做**边**（edge），与顶点相连接的边数叫做**度**（degree）。

边有方向的图叫做**有向图**，比如微博的用户关注与粉丝的关系，有向图中的度分为**出度**（Out-degree）和**入度**（In-degree）。入度表示指向这个顶点的边数，即粉丝数；出度表示这个顶点指向多少个其它顶点，即关注数。边没有方向的叫做**无向图**，比如微信好友关系。

边有权重的图叫做**带权图**（weighted graph），比如 qq 好友的亲密度。

所有的顶点都是连通的图叫做**连通图**。

## 存储

### 邻接矩阵

邻接矩阵（Adjacency Matrix）底层依赖于一个二维数组。如下图：

![](../../.gitbook/assets/image%20%28108%29.png)

**缺点**：浪费存储空间。若是无向图，就白白浪费一半。若是稀疏图，则浪费非常严重，比如微信几亿用户。

**优点**：获取顶点关系时非常高效；方便计算，很多问题可以转换为矩阵计算。

### 邻接表

邻接表（Adjacency List）底层依赖于一维数组 + 列表。每个顶点对应一个列表。如下图：

![](../../.gitbook/assets/image%20%28225%29.png)

这就是时间换空间的思想。尽管节约了很多空间，但是使用起来比较耗时。

```java
// 无向图的实现
public final class Graph {
    private final int v;
    private final List<Integer>[] adj;

    public Graph(int v) {
        this.v = v;
        adj = new List[v];
        for (int i = 0; i < v; i++) {
            adj[i] = new LinkedList<>();
        }
    }

    public void addEdge(int s, int t) {
        adj[s].add(t);
        adj[t].add(s);
    }
}
```

**优化**：可以将列表换成其它数据结构，比如红黑树、跳表、散列表、有序动态数组（可用二分查找）等。

{% hint style="info" %}
对于有向图来说，列表存储的是顶点指向的顶点，所以很容易查找用户关注了哪些用户。但是若要查找用户的粉丝列表，就很困难。所以需要一个**逆邻接表**，顶点存储的是指向它的顶点。
{% endhint %}

## BFS

广度优先搜索（Breath-First-Search）即先查找最近的，然后是次近的。

```java
/**
* bfs 搜索到的路径就是最短路径
*/
public void bfs(int s, int t) {
    int[] prev = new int[v];
    Arrays.fill(prev, -1);
    
    if (s == t) {
        print(prev, t);
    }
    
    boolean[] visited = new boolean[v];
    Arrays.fill(visited, false);
    visited[s] = true;
    
    Queue<Integer> queue = new LinkedList<>();
    queue.add(s);
    
    while (queue.size() > 0) {
        Integer p = queue.poll();
        for (int i : adj[p]) {
            if (!visited[i]) {
                prev[i] = p;
                if (i == t) {
                    print(prev, t);
                    return;
                }
                visited[i] = true;
                queue.add(i);
            }
        }
    }
}
```

时间复杂度 O\(E\)，空间复杂度 O\(V\)，E 表示边数，V 表示顶点数。

## DFS

深度优先搜索（Depth-First-Search）采用的是回溯思想，非常适合递归实现。

```java
/**
* dfs 搜索到的路径不一定是最短路径
*/
public void dfs(int s, int t) {
    final AtomicReference<Boolean> found = new AtomicReference<>(false);
    
    boolean[] visited = new boolean[v];
    Arrays.fill(visited, false);
    
    int[] prev = new int[v];
    Arrays.fill(prev, -1);
    
    recurDfs(found, visited, prev, s, t);
    print(prev, t);
}

private void recurDfs(AtomicReference<Boolean> found, boolean[] visited, int[] prev, int s, int t) {
    if (visited[s] || found.get()) {
        return;
    }
    if (s == t) {
        found.set(true);
        return;
    }
    visited[s] = true;
    for (Integer i : adj[s]) {
        prev[i] = s;
        recurDfs(found, visited, prev, i, t);
    }
}
```

时间复杂度 O\(E\)，空间复杂度 O\(V\)，E 表示边数，V 表示顶点数。

{% hint style="info" %}
DFS 和 BFS 是最简单的两种搜索方法，简单粗暴，没有什么优化，也叫做暴力搜索算法。**适合于状态空间不大的图的搜索。**
{% endhint %}

