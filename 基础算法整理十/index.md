# 基础算法整理(十)


<!--more-->

转载请注明出处：`https://hts0000.github.io/`

欢迎与我联系：`hts_0000@sina.com`

# 背包问题
有 N 件物品和一个容量为 V 的背包。物品可能多有个属性——费用Ci、价值Wi、数量Si。求解将哪些物品装入背包可使价值总和最大。

- 01背包：每个物品只有一个
- 完全背包：每个物品有无限个
- 多重背包：每个物品有给定数量
- 分组背包：有N组物品，每组物品中只能选一个物品

## 01背包
### 01背包朴素
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
    
    // 存储第 i 件物品的体积和价值
    v, w [N]int
    // dp 数组，第一维表示物品，第二维表示容量
    // f[i][j] 表示把物品 i 放进容量为 j 的背包的最大价值
    f [N][N]int
    // n 是物品的个数，m 是背包容量
    n, m int
)

func main() {
    defer out.Flush()
    
    // 处理输入
    fmt.Fscan(in, &n, &m)
    for i := 1; i <= n; i++ { fmt.Fscan(in, &v[i], &w[i]) }
    
    // 初始化，表示0件物品放进容量为 i 的背包里的最大价值为0
    for i := 0; i <= m; i++ { f[0][i] = 0 }
    
    // i 枚举物品
    for i := 1; i <= n; i++ {
        // j 枚举容量
        for j := 1; j <= m; j++ {
            // 如果背包容量放不下物品 i 了，最大价值等于不放物品 i 的最大价值
            if j < v[i] {
                f[i][j] = f[i-1][j]
            // 能放下物品 i，则比较放与不放哪种价值大
            // f[i-1][j] 表示不放物品 i 的最大价值
            // f[i-1][j-v[i]] + w[i] 表示放物品 i 的最大价值
            } else {
                f[i][j] = max(f[i-1][j], f[i-1][j-v[i]] + w[i])
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

### 01背包优化
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
    
    // 存储第 i 件物品的体积和价值
    v, w [N]int
    // dp 数组
    f [N]int
    // n 是物品的个数，m 是背包容量
    n, m int
)

func main() {
    defer out.Flush()
    
    // 处理输入
    fmt.Fscan(in, &n, &m)
    for i := 1; i <= n; i++ { fmt.Fscan(in, &v[i], &w[i]) }
    
    // 初始化，表示0件物品放进容量为 i 的背包里的最大价值为0
    for i := 0; i <= m; i++ { f[i] = 0 }
    
    // i 枚举物品
    for i := 1; i <= n; i++ {
        // j 枚举容量
        for j := m; j >= v[i]; j-- {
            // f[j] 表示不选第 i 件物品的最大价值，f[j-v[i]] + w[i] 表示选择第 i 件物品的最大价值
            f[j] = max(f[j], f[j-v[i]] + w[i])
        }
    }
    
    fmt.Fprintln(out, f[m])
}

func max(a, b int) int {
    if a > b { return a }
    return b
}
```

### 经典模板题
[2. 01背包问题](https://www.acwing.com/problem/content/2/)

## 完全背包
### 完全背包朴素
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
    // 存储第 i 件物品的体积和价值
    v, w [N]int
    // dp 数组，第一维表示物品，第二维表示容量
    f [N][N]int
    // n 是物品的数量，m 是背包容量
    n, m int
)

func main() {
    defer out.Flush()
    
    fmt.Fscan(in, &n, &m)
    
    for i := 1; i <= n; i++ { fmt.Fscan(in, &v[i], &w[i]) }
    
    // 枚举物品
    for i := 1; i <= n; i++ {
        // 枚举容量
        for j := 1; j <= m; j++ {
            // 枚举物品个数
            for k := 0; k * v[i] <= j; k++ {
                // f[i][j] 表示第 i 个物品取 k 个放进容量为 j 的背包中的最大价值
                f[i][j] = max(f[i][j], f[i-1][j-v[i]*k] + w[i]*k)
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

### 完全背包优化
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
    // 存储第 i 件物品的体积和价值
    v, w [N]int
    // dp 数组
    f [N]int
    // n 是物品的数量，m 是背包容量
    n, m int
)

func main() {
    defer out.Flush()
    
    fmt.Fscan(in, &n, &m)
    
    for i := 1; i <= n; i++ { fmt.Fscan(in, &v[i], &w[i]) }
    
    for i := 1; i <= n; i++ {
        for j := v[i]; j <= m; j++ {
            f[j] = max(f[j], f[j-v[i]] + w[i])
        }
    }
    
    fmt.Fprintln(out, f[m])
}

func max(a, b int) int {
    if a > b { return a }
    return b
}
```

### 经典模板题
[3. 完全背包问题](https://www.acwing.com/problem/content/3/)

## 多重背包
### 多重背包朴素
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
    // 存储第 i 件物品的体积、价值和数量
    v, w, s [N]int
    // dp 数组，
    // f[i][j] 表示数量 s[i] 的物品 i，存入容量为 j 的背包中的最大价值
    f [N][N]int
    // n 是物品的数量，m 是背包容量
    n, m int
)

func main() {
    defer out.Flush()
    
    fmt.Fscan(in, &n, &m)
    
    for i := 1; i <= n; i++ { fmt.Fscan(in, &v[i], &w[i], &s[i]) }
    
    // 枚举物品
    for i := 1; i <= n; i++ {
        // 枚举容量
        for j := 1; j <= m; j++ {
            // 枚举物品数量从 0 ~ s[i]，且总体积不超过 j
            for k := 0; k <= s[i] && k * v[i] <= j; k++ {
                // f[i][j] 表示第 i 个物品取 k 个放进容量为 j 的背包中的最大价值，0 <= k <= s[i]
                f[i][j] = max(f[i][j], f[i-1][j-v[i]*k] + w[i]*k)
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

### 多重背包优化
**多重背包二进制优化**  
把物品 i 的数量 s[i]，按照二进制分成若干个，$2^0 + 2^1 + ... + 2^{k} + s[i] - 2^{k} = s[i]$，将拆分的每一个数看成一个单独的物品。把所有 s[i] 拆分之后，对所有拆分的物品求一次01背包问题。

```go
package main

import (
    "bufio"
    "fmt"
    "os"
)

// 物品个数 N <= 1000
// 物品数量 S <= 2000
// 每个物品都按照二进制拆分成了 logS 个
// 所以总的物品个数是 N * logS 上取整
const N int = 25010

var (
    in = bufio.NewReader(os.Stdin)
    out = bufio.NewWriter(os.Stdout)
    // 存储第 i 件物品的体积、价值和数量
    v, w [N]int
    // dp 数组，
    // f[i][j] 表示数量 s[i] 的物品 i，存入容量为 j 的背包中的最大价值
    f [N]int
    // n 是物品的数量，m 是背包容量
    n, m int
)

func main() {
    defer out.Flush()

    fmt.Fscan(in, &n, &m)

    // 将物品 i 的数量按照二进制拆成若干个
    cnt := 0
    for i := 0; i < n; i++ {
        var a, b, s int
        fmt.Fscan(in, &a, &b, &s)
        k := 1
        // 二进制拆分
        for k <= s {
            cnt++
            v[cnt] = a * k
            w[cnt] = b * k
            s -= k
            k *= 2
        }
        // 2^k 次方是小于 s[i] 的最大二进制数，剩下的数为 s[i] - 2^k
        if s > 0 {
            cnt++
            v[cnt] = a * s
            w[cnt] = b * s
        }
    }
    // 物品个数为拆分了多少个
    n = cnt
    // 对所有拆分的物品做01背包
    for i := 1; i <= n; i++ {
        for j := m; j >= v[i]; j-- {
            f[j] = max(f[j], f[j-v[i]] + w[i])
        }
    }
    fmt.Fprintln(out, f[m])
}

func max(a, b int) int {
    if a > b { return a }
    return b
}
```

### 经典模板题
[4. 多重背包问题 I](https://www.acwing.com/problem/content/4/)  
[5. 多重背包问题 II](https://www.acwing.com/problem/content/5/)

## 分组背包
### 分组背包朴素
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
    // v[i][j] 表示第 i 组里的物品 j 的体积是多少
    v, w [N][N]int
    // s[i] 表示第 i 组里有多少种物品
    s [N]int
    // f[i][j] 表示把每 i 组物品选一个，放进容量为 j 的背包中的最大价值
    f [N][N]int
    // 物品组数和背包容量
    n, m int
)

func main() {
    defer out.Flush()
    
    fmt.Fscan(in, &n, &m)
    
    // 读取输入
    for i := 1; i <= n; i++ {
        // 第 i 组有多少种物品
        fmt.Fscan(in, &s[i])
        // 第 i 组中每个物品的体积和价值
        for j := 1; j <= s[i]; j++ {
            fmt.Fscan(in, &v[i][j], &w[i][j])
        }
    }
    
    // 枚举物品组
    for i := 1; i <= n; i++ {
        // 枚举容量
        for j := 0; j <= m; j++ {
            // 不选第 i 组的最大价值
            f[i][j] = f[i-1][j]
            // 枚举物品组中每一个物品
            for k := 1; k <= s[i] ; k++ {
                // 容量能够放下第 i 组的物品 k
                if v[i][k] <= j {
                    // 第 i 组选择物品 k 放入容量为 j 的背包中的最大价值
                    f[i][j] = max(f[i][j], f[i-1][j-v[i][k]] + w[i][k])
                }
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

### 分组背包优化
**只优化了空间**  
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
    // v[i][j] 表示第 i 组里的物品 j 的体积是多少
    v, w [N][N]int
    // s[i] 表示第 i 组里有多少种物品
    s [N]int
    // f[j] 表示把每一组物品各选一个，放进容量为 j 的背包中的最大价值
    f [N]int
    // 物品组数和背包容量
    n, m int
)

func main() {
    defer out.Flush()
    
    fmt.Fscan(in, &n, &m)
    
    // 读取输入
    for i := 1; i <= n; i++ {
        // 第 i 组有多少种物品
        fmt.Fscan(in, &s[i])
        // 第 i 组中每个物品的体积和价值
        for j := 1; j <= s[i]; j++ {
            fmt.Fscan(in, &v[i][j], &w[i][j])
        }
    }
    
    // 枚举物品组
    for i := 1; i <= n; i++ {
        // 枚举容量
        for j := m; j >= 0; j-- {
            // 枚举物品组中每一个物品
            for k := 1; k <= s[i] ; k++ {
                // 容量能够放下第 i 组的物品 k
                if v[i][k] <= j {
                    // 第 i 组选择物品 k 放入容量为 j 的背包中的最大价值
                    f[j] = max(f[j], f[j-v[i][k]] + w[i][k])
                }
            }
        }
    }
    
    fmt.Fprintln(out, f[m])
}

func max(a, b int) int {
    if a > b { return a }
    return b
}
```

### 经典模板题
[9. 分组背包问题](https://www.acwing.com/problem/content/9/)

