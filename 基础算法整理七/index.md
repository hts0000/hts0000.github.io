# 基础算法整理(七)


<!--more-->

转载请注明出处：`https://hts0000.github.io/`

欢迎与我联系：`hts_0000@sina.com`

# 搜索
## BFS
使用数据结构：`queue`。  
使用空间：因要存储每一层所有节点，因此使用的空间是`O(2^h)`指数级，h是树的高度。

**BFS搜索具有最短路的性质**。BFS搜索两个点的距离，一定是最短的。

BFS是树中的层序遍历。

### 代码模板
走迷宫问题
```go
package main

import (
    "bufio"
    "fmt"
    "os"
)

const N int = 110

var (
    in = bufio.NewReader(os.Stdin)
    out = bufio.NewWriter(os.Stdout)
    
    // 存储迷宫
    maze [N][N]int
    // 存储距离
    dist [N][N]int
    n, m int
)

type pair struct {
    first int
    second int
}

func bfs(start pair) int {
    queue := make([]pair, 1)
    queue[0] = start
    dist[0][0] = 0
    // 上右下左 四个方向
    var dx = [4]int{-1, 0, 1, 0}
    var dy = [4]int{0, 1, 0, -1}
    for len(queue) > 0 {
        t := queue[0]
        queue = queue[1:]
        // 枚举四个方向
        for j := 0; j < 4; j++ {
            x := t.first + dx[j]
            y := t.second + dy[j]
            // 将合法的位置加入队列中
            if x >= 0 && x < n && y >= 0 && y < m && maze[x][y] == 0 && dist[x][y] == -1 {
                queue = append(queue, pair{x, y})
                dist[x][y] = dist[t.first][t.second] + 1
            }
        }
    }
    return dist[n-1][m-1]
}

func main() {
    defer out.Flush()
    fmt.Fscan(in, &n, &m)
    for i := 0; i < n; i++ {
        for j := 0; j < m; j++ {
            fmt.Fscan(in, &maze[i][j])
            // 没走过的点初始化为-1
            dist[i][j] = -1
        }
    }
    fmt.Fprintln(out, bfs(pair{0, 0}))
}
```

### 经典模板题
[844. 走迷宫](https://www.acwing.com/problem/content/846/)  
[845. 八数码](https://www.acwing.com/problem/content/847/)

## DFS
使用数据结构：`stack`。  
使用空间：只需要存储当前路径上的所有节点，因此使用的空间是`O(h)`，h是树的高度。

DFS的一些概念：
- DFS搜索不具有最短路性质。
- DFS是树的中序遍历(左中右)。
- DFS最重要的是画出递归树。

DFS两个重要概念——回溯和剪枝。  
回溯的难点在于递归过程中的去重问题。常见去重有：
- 同层去重
- 子树去重
- 路径去重

深度优先搜索可以求出某个节点为根的子树上节点的数量。

### 代码模板
N皇后问题
```go
package main

import (
    "fmt"
    "bufio"
    "os"
)

var in = bufio.NewReader(os.Stdin)
var out = bufio.NewWriter(os.Stdout)

const N int = 10
// 棋盘
var chessbord [N][N]byte
var (
    // 记录列是否有皇后
    col [N]bool
    // 记录45度对角是否有皇后，有n行n列，因此需要开2*N的空间
    udg [2*N]bool
    // 记录135度对角是否有皇后，有n行n列，因此需要开2*N的空间
    dg  [2*N]bool
)
var n int

func dfs(row int) {
    // 找到了一种解决方案，输出棋盘
    if row == n {
        for i := 0; i < n; i++ {
            for j := 0; j < n; j++ {
                fmt.Fprintf(out, "%c", chessbord[i][j])
            }
            fmt.Fprintln(out)
        }
        fmt.Fprintln(out)
        return
    }
    // 枚举每一列，能放下皇后的进入下一行
    for c := 0; c < n; c++ {
        // 如果这一列、135度对角和45度对角上都没有皇后，就在c列放下皇后，并进入下一行
        // row - c + n 和 row + c 如何理解？每条对角线可以看成是一个 y = x + b 或 y = -x + b 的函数
        // 唯一的 x和y 可以确认唯一的 b，我们可以用这个 b 来代表y行x列的对角线
        // 135度对角线y = x + b，45度对角线y = -x + b
        // 变形后135度对角线 b = y - x，45度对角线b = y + x
        // 因为 y - x 可能为负，我们可以加上一个n，将整体结果映射到0~n这个区间
        if !col[c] && !udg[row - c + n] && !dg[row + c] {
            // 放下皇后
            chessbord[row][c] = 'Q'
            // 标记 c 列有皇后；标记 row 行 -c 列有皇后；标记 row 行 c 列有皇后
            col[c] = true; udg[row - c + n] = true; dg[row + c] = true
            // 进入下一行
            dfs(row + 1)
            // 回溯状态，为下一列枚举提供可能
            chessbord[row][c] = '.'
            col[c] = false; udg[row - c + n] = false; dg[row + c] = false
        }
    }
    
}

func main() {
    defer out.Flush()
    
    fmt.Fscan(in, &n)
    
    // 初始化棋盘
    for i := 0; i < n; i++ {
        for j := 0; j < n; j++ {
            chessbord[i][j] = '.'
        }
    }
    
    dfs(0)
}
```

### 经典模板题
[842. 排列数字](https://www.acwing.com/problem/content/844/)  
[843. n-皇后问题](https://www.acwing.com/problem/content/845/)  
[51. N 皇后](https://leetcode.cn/problems/n-queens/)

# 图论
## 树与图的存储
树是一种特殊的图，因此我们只讨论图即可。

**图的形式**  
图分为有向图和无向图，无向图是一种特殊的有向图，其两个顶点有两条相互连接的边，因此在讨论图的遍历时，只讨论有向图即可。

**图的存储形式**
图的存储形式有：
- 邻接矩阵
- 邻接表

邻接矩阵是一个N*N的二维数组。x行y列存储值为1，表示x顶点与y顶点有一条边。邻接矩阵使用比较少，因为他需要`O(n^2)`的存储空间，比较适合存储稠密图。  
![](https://cdn.jsdelivr.net/gh/hts0000/images/202210211554433.png)

邻接表是一个元素是链表的数组，下标x存储的是一条链表，链表所有节点是x顶点能到达的所有其他顶点。  
![](https://cdn.jsdelivr.net/gh/hts0000/images/202210211544438.png)

重边：指的是点i到点j之间存在不止一条边

自环：指的是点i有一条边指向自己

## 图的遍历
深度有限搜索可以方便的获取子树的节点数量。

### 深度优先搜索遍历代码模板
```go
const N int = 100010

var (
    // 邻接表的存储方式
    h [N]int
    e, ne [2*N]int
    idx int
    // 记录哪些点被访问过了
    st [N]bool
)

// 为 a，b 点建立连接
func add(a, b int) {
    e[idx] = b
    ne[idx] = h[a]
    h[a] = idx
    idx++
}

// 深度有限搜索遍历图
func dfs(u int) {
    st[u] = true
    for i := h[u]; i != -1; i = ne[i] {
        j := e[i]
        if !st[j] { dfs(j) }
    }
}
```

### 经典模板题
[846. 树的重心](https://www.acwing.com/problem/content/848/)

### 广度优先搜索代码模板
```go
const N int = 100010

var (
    // 存储邻接表
    h, e, ne [N]int
    idx int
    // 存储点是否被访问过
    st [N]bool
)

func add(a, b int) {
    e[idx] = b
    ne[idx] = h[a]
    h[a] = idx
    idx++
}

func bfs() {
    queue := make([]int, 1, N)
    queue[0] = 1
    st[1] = true
    for len(queue) > 0 {
        t := queue[0]
        queue = queue[1:]
        // 当前点的邻接表
        for i := h[t]; i != -1; i = ne[i] {
            j := e[i]
            if !st[j] {
                st[j] = true
                queue = append(queue, j)
            }
        }
    }
}
```

### 经典模板题
[847. 图中点的层次](https://www.acwing.com/problem/content/849/)

## 拓扑排序
拓扑排序是指，将有向图中的所有顶点排序，使得所有的有向边均从排在前面的元素指向排在后面的元素。  
拓扑排序是针对有向图而言，无向图没有拓扑序列。  
拓扑排序的一个典型应用是——有前导课程的课程表，所有后面的课程指向前面的先导课程。

一个有环图不存在拓扑排序，因为必定有一个前面的节点指向后面的节点。反之，一个有向无环图必定存在拓扑排序，因此有向无环图又称为拓扑图。

**图的入度与出度：**  
- 入度：有多少条边指向该节点
- 出度：该节点有多少条边出去

**求拓扑排序的步骤**  
1. 寻找入度为0的点入队，入度为0意味着没有其他节点指向它。
2. 取出队头元素，将队头元素所有的出边删掉
3. 将出边指向节点中，入度为0的点入队

拓扑图必定存在一个入度为0的点。如果存在环，那么将不会有元素入队，循环结束。

### 代码模板
```go
package main

import (
    "fmt"
    "bufio"
    "os"
)

const N int = 1e5 + 10

var (
    in = bufio.NewReader(os.Stdin)
    out = bufio.NewWriter(os.Stdout)
    
    h, e, ne [N]int
    idx int
    
    // d 存储所有节点的入度
    queue, d [N]int
    hh, tt int = 0, -1
    
    n, m int
)

func add(a, b int) {
    e[idx] = b
    ne[idx] = h[a]
    h[a] = idx
    idx++
}

func topsort() bool {
    // 先将所有入度为0的点入队
    for i := 1; i <= n; i++ {
        if d[i] == 0 {
            tt++
            queue[tt] = i
        }
    }
    // 取出队头元素，将其所有出边删除
    for hh <= tt {
        t := queue[hh]
        hh++
        for i := h[t]; i != -1; i = ne[i] {
            j := e[i]
            // 将入度为0的点入队
            d[j]--
            if d[j] == 0 {
                tt++
                queue[tt] = j
            }
        }
    }
    // 如果是拓扑图，那么所有点都会入队
    return tt == n - 1
}

func main() {
    defer out.Flush()
    
    fmt.Fscan(in, &n, &m)
    for i := 1; i <= n; i++ { h[i] = -1 }
    var a, b int
    for i := 1; i <= m; i++ {
        fmt.Fscan(in, &a, &b)
        add(a, b)
        d[b]++
    }
    if topsort() {
        for i := 0; i < hh; i++ {
            fmt.Fprintf(out, "%d ", queue[i])
            fmt.Fprintln(out)
        }
    } else {
        fmt.Fprintln(out, "-1")
    }
}
```

### 经典模板题
[848. 有向图的拓扑序列](https://www.acwing.com/problem/content/description/850/)

