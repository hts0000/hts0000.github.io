# 基础算法整理(八)


<!--more-->

转载请注明出处：`https://hts0000.github.io/`

欢迎与我联系：`hts_0000@sina.com`

# 最短路
最短路问题分为两类：
- 单源最短路：求一个点到其他所有点的最短距离
- 多源汇最短路：源点就是**起点**，汇点就是**终点**，可能有多个询问，每次询问一个点到另一个点的最短距离

单源最短路根据**边权值的正负**，可以使用不同的算法：
> n为图的点的数量，m为图的边的数量
- 边权值为正
	- 朴素版Dijkstra算法，时间复杂度`O(n^2)`，与边数量没有关系，比较适合稠密图
	- 堆优化版Dijkstra算法，时间复杂度`O(mlogn)`，如果是稀疏图，或n数量比较大，应该使用堆优化版
- 边权值存在负数
	- Bellman-Ford算法，时间复杂度`O(nm)`，求经过不超过k条边的最短路，只能用Bellman-Ford算法来做
	- SPFA算法，时间复杂度一般情况下为`O(m)`，最坏情况为`O(nm)`，是对Bellman-Ford算法的优化，SPFA算法一般情况下优于Bellman-Ford算法，但是有的情况只能用Bellman-Ford算法来做

多源汇最短路只有一种算法：Floyd算法，时间复杂度为`O(n^3)`

## 单源最短路
### 朴素Dijkstra算法代码模板
```go
package main

import (
    "fmt"
    "bufio"
    "os"
)

const N int = 510

var (
    in = bufio.NewReader(os.Stdin)
    out = bufio.NewWriter(os.Stdout)
    
    // 用邻接矩阵存储稠密图
    g [N][N]int
    // 存储起点到各个点的最短距离
    d [N]int
    // 存储是否已经确定1->i的最短路
    st [N]bool
    n, m int
)

func dijkstra() int {
    // 初始化d为正无穷，表示所有点都不可达
    for i := 1; i <= n; i++ { d[i] = 0x3f3f3f3f }
    // 1号点为起点，初始化起点到起点距离为0
    d[1] = 0
    for i := 0; i < n; i++ {
        t := -1
        // 寻找还未更新距离的节点中的最小距离的节点
        for j := 1; j <= n; j++ {
            if !st[j] && (t == -1 || d[t] > d[j]) {
                t = j
            }
        }
        // 标记已寻到1->t的最短路
        st[t] = true
        // 从t点出发更新其余点的最短路
        for j := 1; j <= n; j++ {
            d[j] = min(d[j], d[t] + g[t][j])
        }
    }
    // n点不可达
    if d[n] == 0x3f3f3f3f { return -1 }
    return d[n]
}

func main() {
    defer out.Flush()
    
    fmt.Fscan(in, &n, &m)
    // 初始化g为正无穷，表示一开始所有边都没建立连接
    for i := 1; i <= n; i++ {
        for j := 1; j <= n; j++ {
            g[i][j] = 0x3f3f3f3f
        }
    }
    for ; m > 0; m-- {
        var x, y, z int
        fmt.Fscan(in, &x, &y, &z)
        // 为x->y点建立连接，权重为z
        // 因为存在重边，所有需要存储最小值
        // 不用处理自环，因为求1->n的最短路自环没有影响
        g[x][y] = min(g[x][y], z)
    }
    // 返回1->n的最短距离
    fmt.Fprintln(out, dijkstra())
}

func min (a, b int) int {
    if a < b { return a }
    return b
}
```

### 经典模板题
[849. Dijkstra求最短路 I](https://www.acwing.com/problem/content/description/851/)

### 用堆来优化朴素版Dijkstra算法
朴素版的Dijkstra算法中，**寻找没确认最短路的点中的距离最小的点**这一步，时间复杂度是O(n^2)的。

```go
for i := 0; i < n; i++ {
    t := -1
    // 寻找还未更新距离的节点中的最小距离的节点
	// 这一步总共执行n * n次
    for j := 1; j <= n; j++ {
        if !st[j] && (t == -1 || d[t] > d[j]) {
            t = j
        }
    }
	...
}
```

寻找一个集合中的最小值，我们可以用堆来优化。因此我们可以**用堆来存储所有点到起点的最短距离`d`**，这样每次寻找最小值只需要`O(1)`的时间复杂度，需要寻找`n`次，因此这一步的时间复杂度降到了`O(n)`。

```go
// d是一个小根堆，或叫优先队列
// 寻找最小距离点操作改为
t := heap.Pop(d)
```

除此之外，我们还需要用最小距离节点——t更新到其他点的距离。朴素算法中，这一步时间复杂度是`O(m)`的。因为我们每次只会更新节点t能到达的节点，访问内存操作最多只有m次。

```go
for i := 0; i < n; i++ {
    ...
    // 从t点出发更新其余点的最短路
    // 每次循环，只有t能到达的点有可能被更新
    // n次循环加起来，有效操作次数<=m次
    for j := 1; j <= n; j++ {
        d[j] = min(d[j], d[t] + g[t][j])
    }
}
```

堆优化后，因为需要更新堆元素，需要调整堆，因此这一步时间复杂度为`O(mlogn)`。

```go
for i := 0; i < n; i++ {
    ...
    // 从t点出发更新其余点的最短路
    // 每次循环，只有t能到达的点有可能被更新
    // n次循环加起来，有效操作次数<=m次
    for j := 1; j <= n; j++ {
        d[j] = min(d[j], d[t] + g[t][j])
        // 这里调整堆是O(logn)的
        heap.Push(d, j)
    }
}
```

最后，朴素算法就被优化成了`O(mlogn)`

### 堆优化版Dijkstra算法代码模板
```go
package main

import (
    "fmt"
    "bufio"
    "os"
    "container/heap"
)

const N int = 1000010

var (
    in = bufio.NewReader(os.Stdin)
    out = bufio.NewWriter(os.Stdout)
    
    // 邻接表存储图
    h, e, ne [N]int
    idx int
    // 存储起点到i点的最短距离
    d [N]int 
    // 存储i->j的边权
    w [N]int
    // 记录点i是否已经寻到最短路
    st [N]bool
    
    n, m int
)

// 存储no节点到起点的距离
type Pair struct { dis, no int }
type PriorityQueue []*Pair
func (pq PriorityQueue) Len() int { return len(pq) }
func (pq PriorityQueue) Swap(i, j int) { pq[i], pq[j] = pq[j], pq[i] }
func (pq PriorityQueue) Less(i, j int) bool { return pq[i].dis < pq[j].dis }
func (pq *PriorityQueue) Push(x interface{}) { *pq = append(*pq, x.(*Pair)) }
func (pq *PriorityQueue) Pop() interface{} {
    old := *pq
    n := len(old)
    x := old[n-1]
    old[n-1] = nil  // avoid memory leak
    *pq = old[0:n-1]
    return x
}

func add(a, b, c int) {
    e[idx] = b
    w[idx] = c
    ne[idx] = h[a]
    h[a] = idx
    idx++
}

func dijkstra() int {
    // 初始化小根堆，并存入起点到起点的最短路
    pq := &PriorityQueue{ { 0, 1 } }
    d[1] = 0
    for pq.Len() > 0 {
        t := heap.Pop(pq).(*Pair)
        
        // t.no是目前最短距离节点编号，t.dis是最短距离节点到起点的距离
        
        // 因为存在重边，再遇到相同节点，就跳过
        if st[t.no] { continue }
        st[t.no] = true
        
        // 更新最短距离节点能到达的所有节点
        for i := h[t.no]; i != -1; i = ne[i] {
            j := e[i]
            if d[j] > d[t.no] + w[i] {
                d[j] = d[t.no] + w[i]
                heap.Push(pq, &Pair{ d[j], j })
            }
        }
    }
    
    if d[n] == 0x3f3f3f3f { return -1 }
    return d[n]
}

func main() {
    defer out.Flush()
    
    fmt.Fscan(in, &n, &m)
    for i := 0; i < N; i++ {
        h[i] = -1
        d[i] = 0x3f3f3f3f
    }
    for ; m > 0; m-- {
        var a, b, c int
        fmt.Fscan(in, &a, &b, &c)
        add(a, b, c)
    }
    
    fmt.Fprintln(out, dijkstra())
}
```

### 经典模板题
[850. Dijkstra求最短路 II](https://www.acwing.com/problem/content/description/852/)

### Bellman-Ford算法
两重循环，第一重循环节点次数n，第二重循环所有边a->b及权重w，并更新`dist[b] = min(dist[b], dist[a] + w)`。这个更新的过程叫做**松弛操作**。

Bellman-Ford算法证明了，所有循环结束之后，所有的边都满足`dist[b] <= dist[a] + w`，这个等式也叫作三角不等式

负权回路是回路的权值加起来小于0

如果存在负权回路，那么Bellman-Ford求出的最短路不一定存在(最短路为负无穷时不存在)

Bellman-Ford算法可以求出是否存在负权回路

### Bellman-Ford算法代码模板
```go
package main

import (
    "fmt"
    "bufio"
    "os"
)

const (
    N int = 510
    M int = 10010
)

// 存储所有边，a -> b 权值为 w 的边
type Edge struct { a, b, w int }

var (
    in = bufio.NewReader(os.Stdin)
    out = bufio.NewWriter(os.Stdout)
    
    // Bellman-Ford算法只要能遍历到所有边就可以求最短路
    edges [M]Edge
    
    // dist 存储 起点 到 点i 的最短距离
    dist [N]int
    
    // backup 存储上一次 dist 的值，避免本次更新时用到被覆盖的值
    // 跟 背包问题 中从后往前遍历，避免使用被覆盖值一个道理
    backup [N]int
    
    // 标识是否找到最短路
    flag bool = true;
    
    n, m, k int
)

func bellmanFord() int {
    // 初始化 dist 所有点到起点距离为无穷
    for i := 1; i <= n; i++ { dist[i] = 0x3f3f3f3f }
    // 起点到起点距离为0
    dist[1] = 0
    for i := 0; i < k; i++ {
        // 保存 dist 状态
        copy(backup[:], dist[:])
        for j := 0; j < m; j++ {
            a := edges[j].a
            b := edges[j].b
            w := edges[j].w
            // 使用 backup 的状态，而不是 dist
            dist[b] = min(dist[b], backup[a] + w)
        }
    }
    // 因为存在负权，距离可能被更新为0x3f3f3f3f - 某个负权w，所以不能简单判断 dist[n] == 0x3f3f3f3f
    // 但是负权最多为 10000，k为500，最多为0x3f3f3f3f - 500 * 10000
    if dist[n] > 0x3f3f3f3f / 2 { flag = false; return -1 }
    return dist[n]
}

func main() {
    defer out.Flush()
    
    fmt.Fscan(in, &n, &m, &k)
    for i := 0; i < m; i++ {
        var a, b, w int
        fmt.Fscan(in, &a, &b, &w)
        edges[i] = Edge{a, b, w}
    }
    
    t := bellmanFord()
    
    if ! flag && t == -1 {
        fmt.Fprintln(out, "impossible")
    } else {
        fmt.Fprintln(out, t)
    }
}

func min(a, b int) int {
    if a < b { return a }
    return b
}
```

### 经典模板题
[853. 有边数限制的最短路](https://www.acwing.com/problem/content/855/)

### SPFA算法
SPFA算法求最短路问题的限制最少，只要图中没有负环的存在，就可以用SPFA算法来求解最短路问题，而99%的最短路问题，都没有负环。

SPFA算法是对Bellman-Ford算法的优化。

SPFA算法的第一重循环是所有被更新过距离的点，用一个队列来存放。一开始队列中只有起点，用起点更新起点的所有邻边，能更新的点加入队列中，用这些变小了的点去更新其他点，才有会使其他点的距离缩小，才有意义。

### SPFA算法代码模板
```go
package main

import (
    "fmt"
    "bufio"
    "os"
)

const N int = 100010

var (
    in = bufio.NewReader(os.Stdin)
    out = bufio.NewWriter(os.Stdout)
    
    // 稀疏图，用邻接表来存
    h, e, ne, w [N]int
    idx int
    // 数组模拟队列，或者直接用slice也行
    que [N]int
    qh, qt int = 0, -1
    // 存储 点1 到 点i 的最短距离
    dist [N]int
    // 标识 点i 是否在队列中，在队列中的点是距离缩小了的点，
    // 用这些点去更新其他点才有意义
    st [N]bool
    flag bool = true
    
    n, m int
)

// 邻接表建图
func add(a, b, c int) {
    e[idx] = b
    ne[idx] = h[a]
    w[idx] = c
    h[a] = idx
    idx++
}

func spfa() int {
    // 初始化所有点为无穷
    for i := 1; i <= n; i++ { dist[i] = 0x3f3f3f3f }
    // 起点 到 起点 的距离为0
    dist[1] = 0
    qt++
    que[qt] = 1;
    st[1] = true
    for qh <= qt {
        t := que[qh]
        qh++
        // 改点出队，标记为不在队列中
        st[t] = false
        // 遍历t的所有边
        for i := h[t]; i != -1; i = ne[i] {
            j := e[i]
            // 用t更新它的所有邻边，如果能缩小距离，将点j也加入队列
            if dist[j] > dist[t] + w[i] {
                dist[j] = dist[t] + w[i]
                // 只有不在队列中的点才加入队列
                if ! st[j] {
                    qt++
                    que[qt] = j
                    st[j] = true
                }
            }
        }
    }
    if dist[n] == 0x3f3f3f3f {
        flag = false
        return -1
    }
    return dist[n]
}

func main() {
    defer out.Flush()
    
    fmt.Fscan(in, &n, &m)
    for i := 1; i <= n; i++ { h[i] = -1 }
    for ; m > 0; m-- {
        var a, b, c int
        fmt.Fscan(in, &a, &b, &c)
        add(a, b, c)
    }
    t := spfa()
    
    if ! flag && t == - 1 { fmt.Fprintln(out, "impossible") } else { fmt.Fprintln(out, t) }
}
```

### 经典模板题
[851. spfa求最短路](https://www.acwing.com/problem/content/853/)

### SPFA算法判断是否存在负环
`dist[i]`的含义**是`起点`到`i点`的最短路距离**，我们新加一个`cnt[i]`，表示**`起点`到`i点`最短路经过的边数**。

`dist[i]`更新的条件是，与i连接的其他点j，起点->j的距离+j->i的距离，小于当前i记录的最短路距离。也就是当前i的状态能被它连接的其他点更新(DP思想)。dist[i] = min(dist[i], dist[j] + w)。

`cnt[i]`跟着`dist[i]`一起更新，更新为**起点->j的边数 + 1**，因为起点->j->i是新的最短路，所以当前i点最短路的边数为，能更新i点最短路的点j所经过的边数加1，可以理解为DP中当前状态由上一个状态推导出来。

判断是否存在负环的条件是，`cnt[i] >= n`，其中n为图中点的个数。如果最短路径等于n，意味这经过了n+1个点，但图中只有n个点，根据抽屉原理，则图中某一个点被经过了两次，则图中有环。因为SPFA是最短路算法，因此正环实际上不会循环(走正环一定没有不走正环短)，只有负环会一直更新(走负环会一直得到更小的值，产生负无穷)。所以当`cnt[i] >= n`时，就可以停止循环，返回找到负环了，如果没有负环，就是正常的找最短路的过程。

> 抽屉原理：桌上有十个苹果，要把这十个苹果放到九个抽屉里，无论怎样放，我们会发现至少会有一个抽屉里面放不少于两个苹果。这一现象就是我们所说的“抽屉原理”。抽屉原理的一般含义为：“如果每个抽屉代表一个集合，每一个苹果就可以代表一个元素，假如有n+1个元素放到n个集合中去，其中必定有一个集合里至少有两个元素。” 抽屉原理有时也被称为鸽巢原理。它是组合数学中一个重要的原理。

### SPFA算法判断是否存在环代码模板
```go
package main

import (
    "fmt"
    "bufio"
    "os"
)

const (
    N int = 2010
    M int = 10010
)

var (
    in = bufio.NewReader(os.Stdin)
    out = bufio.NewWriter(os.Stdout)
    
    // 邻接表存储图
    h [N]int
    e, ne, w [M]int
    idx int
    
    // 存储起点到点i的最短距离
    dist [N]int
    // 存储起点到点i最短路经过的边数
    cnt [N]int
    // 记录点i是否在队列中
    st [N]bool
    
    n, m int
)

// 邻接表建图
func add(a, b, c int) {
    e[idx] = b
    ne[idx] = h[a]
    w[idx] = c
    h[a] = idx
    idx++
}

// SPFA算法判断是否存在负环
func spfa() bool {
    que := make([]int, 0, n)
    for i := 1; i <= n; i++ {
        // 初始化所有距离为正无穷
        dist[i] = 0x3f3f3f3f
        // 这里与spfa求最短路不一样，要把所有点入队
        // 因为负环可能从起点到不了
        que = append(que, i)
        st[i] = true
    }
    // 起点到起点的距离为0
    dist[1] = 0
    for len(que) > 0 {
        t := que[0]
        que = que[1:]
        
        st[t] = false
        
        for i := h[t]; i != -1; i = ne[i] {
            j := e[i]
            // 点j可以被点t更新距离
            if dist[j] > dist[t] + w[i] {
                // 更新dist[j]的同时更新cnt[j]
                dist[j] = dist[t] + w[i]
                cnt[j] = cnt[t] + 1
                // 如果经过了n条边，意味这经过了n+1个点，存在环
                // 最短路中正环不会循环，只有负环会循环，因此可以判断存在负环
                if cnt[j] >= n { return true }
                // j不在队列中才加入队列
                if ! st[j] {
                    que = append(que, j)
                    st[j] = true
                }
            }
        }
    }
    return false
}

func main() {
    defer out.Flush()
    
    fmt.Fscan(in, &n, &m)
    for i := 1; i <= n; i++ { h[i] = -1 }
    for ; m > 0; m-- {
        var a, b, c int
        fmt.Fscan(in, &a, &b, &c)
        add(a, b, c)
    }
    if spfa() {
        fmt.Fprintln(out, "Yes")
    } else {
        fmt.Fprintln(out, "No")
    }
}
```

### 经典模板题
[852. spfa判断负环](https://www.acwing.com/problem/content/description/854/)

### Floyd算法求最短路
Floyd算法是基于DP的，时间复杂度O(n^3)。用邻接矩阵来存储图——`d[i][j]`表示从i点到j点的最短路是多少。

### Floyd算法求最短路代码模板
```go
package main

import (
    "fmt"
    "bufio"
    "os"
)

const (
    N int = 210
    INF int = 1e9
)

var (
    in = bufio.NewReader(os.Stdin)
    out = bufio.NewWriter(os.Stdout)
    
    // 用邻接矩阵存储图
    d [N][N]int
    // n个点，m条边，q个询问
    n, m, q int
)

func floyd() {
    for k := 1; k <= n; k++ {
        for i := 1; i <= n; i++ {
            for j := 1; j <= n; j++ {
                // DP思想
                d[i][j] = min(d[i][j], d[i][k] + d[k][j])
            }
        }
    }
}

func main() {
    defer out.Flush()
    
    fmt.Fscan(in, &n, &m, &q)
    
    // 初始化邻接矩阵，图存在重边和自环
    // floyd要求图中没有负环，自环自己到自己初始化为0
    for i := 1; i <= n; i++ {
        for j := 1; j <= n; j++ {
            if i == j {
                d[i][j] = 0
            } else {
                d[i][j] = INF
            }
        }
    }
    
    // 读入m条边
    for ; m > 0; m-- {
        var a, b, w int
        fmt.Fscan(in, &a, &b, &w)
        // 重边用取min的方式解决
        d[a][b] = min(d[a][b], w)
    }
    
    // Floyd算法会将d[N][N]变成存储点i到点j的最短路距离
    floyd()
    
    // q个询问
    for ; q > 0; q-- {
        var a, b int
        fmt.Fscan(in, &a, &b)
        // 因为存在负权值，最大值可能会被更新
        if d[a][b] > INF / 2 {
            fmt.Fprintln(out, "impossible")
        } else {
            fmt.Fprintln(out, d[a][b])
        }
    }
}

func min(a, b int) int {
    if a < b { return a }
    return b
}
```

### 经典模板题
[854. Floyd求最短路](https://www.acwing.com/problem/content/856/)


