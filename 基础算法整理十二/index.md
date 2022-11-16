# 基础算法整理(十二)


<!--more-->

转载请注明出处：`https://hts0000.github.io/`

欢迎与我联系：`hts_0000@sina.com`

# 数位统计DP
数位DP强调分情况讨论

## 计数问题
```go
package main

import (
    "bufio"
    "fmt"
    "os"
)

var (
    in = bufio.NewReader(os.Stdin)
    out = bufio.NewWriter(os.Stdout)
)

func power10(x int) int {
    ans := 1
    for x > 0 {
        ans *= 10
        x--
    }
    return ans
}

// 返回 1~n 中 x 出现次数
func count(n, x int) int {
    ans := 0
    cnt := 0
    
    // 统计 n 有多少位
    for m := n; m > 0; m /= 10 { cnt++ }
    
    // 从右往左枚举每一位上的 x 总数
    for i := 1; i <= cnt; i++ {
        // 我们计算第四位上 x 出现的次数
        // 假设 n = abcdefg，i = 4 指向 d 这一位
        
        // 情况1：高三位为 000~abc-1
        // d 右边可以取到 000~999 共 power10(i - 1) 个数
        r := power10(i - 1)
        // d 左边可以取到 000 ~abc-1 共 abc 种情况
        l := n / (r * 10)
        // abc = n / power10(i) = n / (r * 10)
        // 当 x == 0 时，则为 001~abc-1 种情况
        if x > 0 {
            ans += l * r
        } else {
            ans += (l - 1) * r
        }
        // n / r = abcd, abcd % 10 = d
        d := (n / r) % 10
        // 高三位等于 abc 的情况，只需要考虑 d >= x，因为 d < x 就不符合条件
        // 前四位 abcd 均相同，后三位可取 0~efg 共 efg + 1 种情况
        if d == x {
            ans += n % r + 1
        // 此时后三位可取 000~999 共 power10(i - 1) 种情况
        } else if d > x {
            ans += r
        }
    }
    return ans
}

func main() {
    defer out.Flush()
    
    for {
        var a, b int
        
        fmt.Fscan(in, &a, &b)
        if a == 0 && b == 0 { break }
        
        if a > b { a, b = b, a }
        
        for i := 0; i < 10; i++ {
            fmt.Fprintf(out, "%d ", count(b, i) - count(a - 1, i))
        }
        fmt.Fprintln(out)
    }
}
```

### 经典模板题
[338. 计数问题](https://www.acwing.com/problem/content/340/)  
[233. 数字 1 的个数](https://leetcode.cn/problems/number-of-digit-one/)

# 状态压缩DP
状态压缩就是把一个集合的状态，压缩成一个数，用这个数的二进制表示集合的每一种情况。再使用位运算、逻辑运算等操作方便的计算状态。

## 蒙德里安的梦想
```go
package main

import (
    "bufio"
    "fmt"
    "os"
)

const (
    N = 12 // 以便计算最后一列 d[m][0]
    M = 1<<N
)

var reader = bufio.NewReader(os.Stdin)
var writer = bufio.NewWriter(os.Stdout)

var st [M]bool // 字典表，存储所有位置的合法情况

func initStatus(n int){
    for i:=0;i<1<<n;i++{
        cnt:=0
        st[i] = true
        for j:=0;j<n;j++{
            // 遇到1，被占， 且统计的空格数为奇数，无法填充竖着的长方形
            if i>>j & 1 > 0 {
                if cnt&1>0{
                    st[i] = false
                    break
                }
                cnt=0
                continue
            }
            cnt++
        }

        // 判断剩下的统计数是否奇数
        if st[i] && cnt&1>0{
            st[i] = false
        }
    }
}

// - **数位统计DP**: 所谓的状态压缩DP，就是用二进制数保存状态。为什么不直接用数组记录呢？因为用一个二进制数记录方便作位运算。前面做过的八皇后，八数码，也用到了状态压缩
//  - 计数问题: 蒙德里安的梦想, 求把N*M的棋盘分割成若干个1*2的的长方形，有多少种方案。
//      - 核心: 找到所有横放 1 X 2 小方格的方案数，因为所有横放确定了，那么竖放方案是唯一的。
//      - 状态定义:
//          - 集合: 用`f[i][j]` 记录第i列第j个状态的所有合法方案。j二进制中1表示对应行的上一列有横放格子并捅出到本列中。
//          - 属性: 方案数
//      - 状态计算: `f[i][j] += f[i - 1][k]`
//          - i 列和 i - 1 列同一行不同时捅出来 ； 本列捅出来的状态j和上列捅出来的状态k求或，得到上列是否存在连续奇数空行状态，奇数空行不转移。
//          - `f[m][0]` 表示没有m列没有涌出。
var f [N][M]int
func solution(n,m int) int{
    initStatus(n)

    f[0][0] = 1

    for i:=1;i<=m;i++{
        for j:=0;j<1<<n;j++{
            f[i][j] = 0  // 重置统计
            for k:=0;k<1<<n;k++{
                if j&k==0 && st[j|k] {
                    f[i][j] +=f[i-1][k]
                }
            }
        }

    }

    return f[m][0] // 最后一列
}

func main() {
    var n,m int
    for {
        fmt.Fscan(reader, &n, &m)
        if n | m == 0 {
            break
        }
        fmt.Fprintln(writer, solution(n,m))
    }
    writer.Flush()
}
```

## 91. 最短Hamilton路径
```go
package main

import "fmt"

/*
举例说明：如果是4个点
0->1->2->3
0->2->1->3
...总方案是3！，剩下的4种方法略...
假设第一种走法的方案，距离是10，第二种走法是20，
则第二种走法后面的点再怎么走的方案也就不被采纳，并且后面点的方法是可以直接套在第一种方案后的。
所以我们就可以只关注两点：
1、那些点是走过的
2、现在走在了哪个点上
状态表示：
f[status][j] 第一维表示走过的所有点，第二维度表示现在落在哪个点上
状态转移：
f[status][j]=f[status_k-j][k]+weight[k][j]  k表示当前走到的j点的前一步k点，并且k点的所有状态集合status_k中不包含j点
status的状态使用二进制状态压缩，1，0分别表示走过和没走过
*/

const (
	N = 20
	M = 1 << 20
)

var (
	n      int
	f      [M][N]int
	weight [N][N]int
)

func main() {
	fmt.Scan(&n)
	// 输入
	for i := 0; i < n; i++ {
		for j := 0; j < n; j++ {
			fmt.Scan(&weight[i][j])
		}
	}
	// 初始化f数组
	for i := 0; i < M; i++ {
		for j := 0; j < N; j++ {
			f[i][j] = 0x3f3f3f3f
		}
	}

	f[1][0] = 0 // 在起点的状态，还没走，所以距离是0

	for i := 0; i < 1<<n; i++ {
		for j := 0; j < n; j++ {
			if i>>j&1 > 0 { // 查看集合i中的j点有没有走过，只有走过的点的集合才是可以状态转移的合法状态
				for k := 0; k < n; k++ { //枚举所有的整数
					if i-(1<<j)>>k&1 > 0 { //当前状态i集合还不包括j点，所以集合中要减去，减去后的集合也得满足合法的状态转移条件
						// 所以k点是已经走过的，二进制1表示。
						f[i][j] = min(f[i][j], f[i-(1<<j)][k]+weight[k][j])
					}
				}
			}
		}
	}
	fmt.Println(f[(1<<n)-1][n-1])
}

func min(a, b int) int {
	if a < b {
		return a
	}
	return b
}
```

### 经典模板题
[291. 蒙德里安的梦想](https://www.acwing.com/problem/content/293/)  
[最短Hamilton路径](https://www.acwing.com/problem/content/93/)

# 树形DP
## 没有上司的舞会
```go
package main

import (
    "bufio"
    "fmt"
    "os"
)

const N int = 6010

var (
    in = bufio.NewReader(os.Stdin)
    out = bufio.NewWriter(os.Stdout)
    
    h, e, ne [N]int
    idx int
    
    happy [N]int
    
    f [N][2]int
    
    hasFather [N]bool
    
    n int
)

func add(a, b int) {
    e[idx] = b
    ne[idx] = h[a]
    h[a] = idx
    idx++
}

func dfs(u int) {
    f[u][1] = happy[u]
    
    for i := h[u]; i != -1; i = ne[i] {
        j := e[i]
        dfs(j)
        
        f[u][1] += f[j][0]
        f[u][0] += max(f[j][0], f[j][1])
    }
}

func main() {
    defer out.Flush()
    
    fmt.Fscan(in, &n)
    
    for i := 1; i <= n; i++ { fmt.Fscan(in, &happy[i]) }
    
    for i := 0; i < N; i++ { h[i] = -1 }
    
    for i := 0; i < n - 1; i++ {
        var a, b int
        fmt.Fscan(in, &a, &b)
        add(b, a)
        hasFather[a] = true
    }
    
    root := 1
    for hasFather[root] { root++ }
    
    dfs(root)
    
    fmt.Fprintln(out, max(f[root][0], f[root][1]))
}

func max(a, b int) int {
    if a > b { return a }
    return b
}
```

# 记忆化搜索
## 滑雪问题
### 滑雪
```go
package main

import (
    "bufio"
    "fmt"
    "os"
)

const N int = 310

var (
    in = bufio.NewReader(os.Stdin)
    out = bufio.NewWriter(os.Stdout)
    
    g [N][N]int
    f [N][N]int
    
    dx = [4]int{-1, 0, 1, 0}
    dy = [4]int{0, 1, 0, -1}
    
    n, m int
)

func dp(x, y int) int {
    if f[x][y] != -1 { return f[x][y] }
    
    f[x][y] = 1
    
    for i := 0; i < 4; i++ {
        a := x + dx[i]
        b := y + dy[i]
        if a >= 1 && a <= n && b >= 1 && b <= m && g[x][y] > g[a][b] {
            f[x][y] = max(f[x][y], dp(a, b) + 1)
        }
    }
    
    return f[x][y]
}

func main() {
    defer out.Flush()
    
    fmt.Fscan(in, &n, &m)
    
    for i := 1; i <= n; i++ {
        for j := 1; j <= m; j++ {
            fmt.Fscan(in, &g[i][j])
        }
    }
    
    for i := 0; i < N; i++ {
        for j := 0; j < N; j++ {
            f[i][j] = -1
        }
    }
    
    ans := 0
    for i := 1; i <= n; i++ {
        for j := 1; j <= m; j++ {
            ans = max(ans, dp(i, j))
        }
    }
    
    fmt.Fprintln(out, ans)
}

func max(a, b int) int {
    if a > b { return a }
    return b
}
```

