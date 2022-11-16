# 基础算法整理(十一)


<!--more-->

转载请注明出处：`https://hts0000.github.io/`

欢迎与我联系：`hts_0000@sina.com`

# 线性DP
递推方程有明显的线性关系

## 数字三角形
### 从上到下
```go
package main

import (
    "bufio"
    "fmt"
    "os"
)

const N int = 510
const INF int = 1e9

var (
    in = bufio.NewReader(os.Stdin)
    out = bufio.NewWriter(os.Stdout)
    // 存放数字三角形
    nums [N][N]int
    // dp 数组
    // f[i][j] 表示从起点走到 [i,j] 点的最大路径和
    f [N][N]int
    // n 行的数字三角形
    n int
)

func main() {
    defer out.Flush()
    
    fmt.Fscan(in, &n)
    
    for i := 1; i <= n; i++ {
        for j := 1; j <= i; j++ {
            fmt.Fscan(in, &nums[i][j])
        }
    }
    
    // 初始化为 -INF
    // 因为是从上往下遍历，会用到 [i-1][j]
    // 因此每一行需要多初始化一列，给下一行的使用
    for i := 0; i <= n; i++ {
        for j := 0; j <= i + 1; j++ {
            f[i][j] = -INF
        }
    }
    
    // 初始化起点，表示从起点走到起点的最大路径和
    f[1][1] = nums[1][1]
    for i := 2; i <= n; i++ {
        for j := 1; j <= i; j++ {
            f[i][j] = max(f[i-1][j-1], f[i-1][j]) + nums[i][j]
        }
    }
    
    ans := -INF
    // 遍历最后一行，获取最大路径和
    for i := 1; i <= n; i++ {
        ans = max(ans, f[n][i])
    }
    
    fmt.Fprintln(out, ans)
}

func max(a, b int) int {
    if a > b { return a }
    return b
}
```

### 从下往上
```go
package main

import (
    "bufio"
    "fmt"
    "os"
)

const N int = 510
const INF int = 1e9

var (
    in = bufio.NewReader(os.Stdin)
    out = bufio.NewWriter(os.Stdout)
    // 存放数字三角形
    nums [N][N]int
    // dp 数组
    // f[1][1] 表示从最下面一层走到起点的最大路径和
    f [N][N]int
    // n 行的数字三角形
    n int
)

func main() {
    defer out.Flush()
    
    fmt.Fscan(in, &n)
    
    for i := 1; i <= n; i++ {
        for j := 1; j <= i; j++ {
            fmt.Fscan(in, &nums[i][j])
        }
    }
    
    // 从下往上遍历，只会用到三角形内的值，因此不用初始化
    for i := n; i >= 1; i-- {
        for j := i; j >= 1; j-- {
            f[i][j] = max(f[i+1][j], f[i+1][j+1]) + nums[i][j]
        }
    }
    
    fmt.Fprintln(out, f[1][1])
}

func max(a, b int) int {
    if a > b { return a }
    return b
}
```

### 经典模板题
[898. 数字三角形](https://www.acwing.com/problem/content/900/)  
[剑指 Offer II 100. 三角形中最小路径之和](https://leetcode.cn/problems/IlPe0q/)

## 最长上升子序列
### 最长上升子序列朴素
```go
package main

import (
    "bufio"
    "fmt"
    "os"
)

const N int = 1010

var (
    in = bufio.NewReader(os.Stdin)
    out = bufio.NewWriter(os.Stdout)
    // 存储输入
    a [N]int
    // dp 数组，f[i] 表示以 i 结尾的最长子序列长度
    f [N]int
    
    n int
)

func main() {
    defer out.Flush()
    
    // 处理输入
    fmt.Fscan(in, &n)
    for i := 1; i <= n; i++ { fmt.Fscan(in, &a[i]) }
    
    // 初始化，每个以 i 结尾的子序列长度至少为1
    for i := 1; i <= n; i++ { f[i] = 1 }
    
    // 枚举每个数字结尾
    for i := 1; i <= n; i++ {
        // 遍历 i 前面所有数字
        for j := 1; j <= i; j++ {
            // 题目要求严格小于
            if a[j] < a[i] {
                // f[j] 表示以 j 结尾的最长子序列长度，加1表示加上 i 的长度
                f[i] = max(f[i], f[j] + 1)
            }
        }
    }
    
    // 遍历一遍 dp 数组获取最大值
    ans := 0
    for i := 1; i <= n; i++ { ans = max(ans, f[i]) }
    
    fmt.Fprintln(out, ans)
}

func max(a, b int) int {
    if a > b { return a }
    return b
}
```

### 最长上升子序列优化
```go
package main

import (
    "bufio"
    "fmt"
    "os"
)

const N int = 100010

var (
    in = bufio.NewReader(os.Stdin)
    out = bufio.NewWriter(os.Stdout)
    a [N]int
    f [N]int
    
    n int
)

func main() {
    defer out.Flush()
    
    fmt.Fscan(in, &n)
    
    for i := 0; i < n; i++ { fmt.Fscan(in, &a[i]) }
    
    length := 0
    for i := 0; i < n; i++ {
        l, r := 0, length
        for l < r {
            mid := (l + r + 1) >> 1
            if f[mid] < a[i] {
                l = mid
            } else {
                r = mid - 1
            }
        }
        length = max(length, r + 1)
        f[r + 1] = a[i]
    }
    
    fmt.Fprintln(out, length)
}

func max(a, b int) int {
    if a > b { return a }
    return b
}
```

### 经典模板题
[895. 最长上升子序列](https://www.acwing.com/problem/content/897/)  
[896. 最长上升子序列 II](https://www.acwing.com/problem/content/898/)

## 最长公共子序列
### 最长公共子序列
```go
package main

import (
    "bufio"
    "fmt"
    "os"
)

const N int = 1010

var (
    in = bufio.NewReader(os.Stdin)
    out = bufio.NewWriter(os.Stdout)
    
    a, b string
    // f[i][j] 表示在 a 字符串前 i 个字符中出现，且在 b 字符串前 j 个字符中出现的最长公共子序列长度
    f [N][N]int
    
    n, m int
)

func main() {
    defer out.Flush()
    
    fmt.Fscan(in, &n, &m)
    
    fmt.Fscan(in, &a, &b)
    // 让字符串从下标1开始
    a = " " + a
    b = " " + b
    
    for i := 1; i <= n; i ++ {
        for j := 1; j <= m; j ++ {
            f[i][j] = max(f[i-1][j], f[i][j-1])
            if a[i] == b[j] {
                f[i][j] = max(f[i][j], f[i-1][j-1] + 1)
            }
        }
    }
    
    fmt.Fprintln(out, f[n][m])
}

func max(a, b int) int {
    if a > b { return a }
    return b
}
```

### 经典模板题
[897. 最长公共子序列](https://www.acwing.com/problem/content/899/)  
[剑指 Offer II 095. 最长公共子序列](https://leetcode.cn/problems/qJnOS7/)

## 编辑距离
### 最短编辑距离
```go
package main

import (
    "bufio"
    "fmt"
    "os"
)

const N int = 1010

var (
    in = bufio.NewReader(os.Stdin)
    out = bufio.NewWriter(os.Stdout)
    
    a, b string
    // f[i][j] 表示将 a 的前 i 个字符变成 b 的前 j 个字符最少需要多少步
    f [N][N]int
    // 字符串长度
    n, m int
)

func main() {
    defer out.Flush()
    
    fmt.Fscan(in, &n, &a)
    fmt.Fscan(in, &m, &b)
    a = " " + a
    b = " " + b
    
    // 表示将 a 的前0个字符变成 b 的前 i 个字符最少需要多少步
    for i := 1; i <= m; i++ { f[0][i] = i }
    // 表示将 a 的前 i 个字符变成 b 的前 0 个字符最少需要多少步
    for i := 1; i <= n; i++ { f[i][0] = i }
    
    for i := 1; i <= n; i++ {
        for j := 1; j <= m; j++ {
            // f[i][j-1]+1 表示 a 的前 i 个字符与 b 的前 j-1 个字符相同需要的最小步骤，
            // +1 表示为 a 字符串增加一个字符，步骤数 +1
            // f[i-1][j]+1 表示 a 的前 i-1 个字符与 b 的前 j 个字符相同需要的最小步骤，
            // +1 表示为 a 字符串删除一个字符，步骤数 +1
            f[i][j] = min(f[i][j-1]+1, f[i-1][j]+1)
            
            // 当前字符不相同，需要修改当前字符
            if a[i] != b[j] {
                // f[i-1][j-1] 表示 a 的前 i-1 个字符与 b 的前 j-1 个字符相同需要的最小步骤，
                // +1 表示将 a 的 i 字符修改为 b 的 j 字符，步骤数 +1
                f[i][j] = min(f[i][j], f[i-1][j-1]+1)
            // 如果当前字符相同，那么当前步骤数就是使得 a 的前 i-1 个字符与 b 的前 j-1 个字符相同需要的最小步骤
            } else {
                f[i][j] = min(f[i][j], f[i-1][j-1])
            }
        }
    }
    
    fmt.Fprintln(out, f[n][m])
}

func min(a, b int) int {
    if a < b { return a }
    return b
}
```

### 经典模板题
[902. 最短编辑距离](https://www.acwing.com/problem/content/904/)  
[72. 编辑距离](https://leetcode.cn/problems/edit-distance/)  
[899. 编辑距离](https://www.acwing.com/problem/content/901/)

# 区间DP
## 石子合并
### 石子合并
```go
package main

import (
    "bufio"
    "fmt"
    "os"
)

const N int = 310
const INF int = 1e9

var (
    in = bufio.NewReader(os.Stdin)
    out = bufio.NewWriter(os.Stdout)
    
    // 存储输入和前缀和
    s [N]int
    // f[i][j] 表示合并 [i~j] 堆石子所需最小代价
    f [N][N]int
    
    n int
)

func main() {
    defer out.Flush()
    
    fmt.Fscan(in, &n)
    // 读取输入
    for i := 1; i <= n; i++ { fmt.Fscan(in, &s[i]) }
    // 处理前缀和
    for i := 1; i <= n; i++ { s[i] += s[i-1] }
    // f[i][i] 单独一堆的石子，合并单独一堆石子代价为0
    for i := 1; i <= n; i++ { f[i][i] = 0 }
    
    // length 枚举区间范围，从 2 开始是因为上面已经初始化了区间范围为 1 的情况
    for length := 2; length <= n; length++ {
        // 枚举区间左端点
        for i := 1; i + length - 1 <= n; i++ {
            // l, r 表示每一个 [l,r] 小区间，最终枚举 [1,n] 整个区间
            // [1,2], [2,3], [3,4], ...
            // [1,3], [2,4], ...
            // [1,4], ...
            l, r := i, i + length - 1
            
            // 表示合并 [l,r] 这个区间代价一开始为无穷
            f[l][r] = INF
            
            // 以 k 为中点，划分为左右两块区间，
            // f[i][j] 的值就为合并左区间的最小代价 f[l][k] + 合并右区间的最小代价 f[k+1][r] + 合并左右区间的代价 s[r] - s[l-1]
            // s[r] - s[l-1] 用前缀和快速求 [l,r] 这段区间的值
            // 为什么要加上 [l,r] 这段区间的值？因为最后还需要合并左右两堆石子
            for k := l; k < r; k++ {
                f[l][r] = min(f[l][r], f[l][k] + f[k+1][r] + s[r] - s[l-1])
            }
        } 
    }
    
    fmt.Fprintln(out, f[1][n])
}

func min(a, b int) int {
    if a < b { return a }
    return b
}
```

# 计数类DP
## 整数划分
### 整数划分
```go
package main

import (
    "bufio"
    "fmt"
    "os"
)

const N int = 1010
const MOD int= 1e9 + 7

var (
    in = bufio.NewReader(os.Stdin)
    out = bufio.NewWriter(os.Stdout)
    
    f [N]int
    
    n int
)

func main() {
    defer out.Flush()
    
    fmt.Fscan(in, &n)
    
    f[0] = 1
    
    // 将整数划分看成是完全背包问题
    // f[j] 表示 1~n 当中选，体积恰好是 j 的方案数
    for i := 1; i <= n; i++ {
        for j := i; j <= n; j++ {
            f[j] = (f[j] + f[j-i]) % MOD
        }
    }
    
    fmt.Fprintln(out, f[n])
}
```


