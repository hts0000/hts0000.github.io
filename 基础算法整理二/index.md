# 基础算法整理(二)


<!--more-->

转载请注明出处：`https://hts0000.github.io/`

欢迎与我联系：`hts_0000@sina.com`

# 高精度
高精度是指，不能直接用一个变量来存储的数据，这种数据有几千位甚至几万位。存储这种数据必须使用数组。高精度运算就是为这种数据做算术运算。
## 1.1 大整数相加
### 代码模板
```go
package main

import "fmt"

func main() {
    a, b := "", ""
    fmt.Scanf("%s", &a)
    fmt.Scanf("%s", &b)
    A := make([]int, 0, len(a))
    B := make([]int, 0, len(b))
    for i := len(a) - 1; i >= 0; i-- { A = append(A, int(a[i] - '0')) }
    for i := len(b) - 1; i >= 0; i-- { B = append(B, int(b[i] - '0')) }
    C := add(A, B)
    for i := len(C) - 1; i >= 0; i-- { fmt.Print(C[i]) }
}

// C = A + B
func add(A, B []int) (C []int) {
    t := 0
    for i := 0; i < len(A) || i < len(B); i++ {
        if i < len(A) { t += A[i] }
        if i < len(B) { t += B[i] }
        C = append(C, t % 10)
        t /= 10
    }
    if t == 1 { C = append(C, 1) }
    return C
}
```
### 经典模板题
[791. 高精度加法](https://www.acwing.com/problem/content/793/)  
[剑指 Offer II 025. 链表中的两数相加](https://leetcode.cn/problems/lMSNwu/)

## 1.2 大整数相减
### 代码模板
```go
package main

import "fmt"

func main() {
    var a, b string
    fmt.Scanf("%s", &a)
    fmt.Scanf("%s", &b)
    A := make([]int, 0, len(a))
    B := make([]int, 0, len(b))
    for i := len(a) - 1; i >= 0; i-- { A = append(A, int(a[i] - '0')) }
    for i := len(b) - 1; i >= 0; i-- { B = append(B, int(b[i] - '0')) }
    if cmp(A, B) {
        C := sub(A, B)
        for i := len(C) - 1; i >= 0; i-- { fmt.Print(C[i]) }
    } else {
        // A < B 的情况，要加上负号
        fmt.Print("-")
        C := sub(B, A)
        for i := len(C) - 1; i >= 0; i-- { fmt.Print(C[i]) }
    }
}

// 判断是否有 A >= B
func cmp(A, B []int) bool {
    if len(A) != len(B) { return len(A) > len(B) }
    for i := len(A) - 1; i >= 0; i-- {
        if A[i] != B[i] { return A[i] > B[i] }
    }
    return true
}

// C = A - B
// 应该保证 A >= B
func sub(A, B []int) (C []int) {
    t := 0
    for i := 0; i < len(A); i++ {
        t = A[i] - t
        if i < len(B) { t -= B[i] }
		// 考虑了需要进位和不需要进位的情况
        C = append(C, (t + 10) % 10)
        if t < 0 {
            t = 1
        } else {
            t = 0
        }
    }
    // 去除前导零
    for len(C) > 1 && C[len(C) - 1] == 0 {
        C = C[:len(C) - 1]
    }
    return C
}
```
### 经典模板题
[792. 高精度减法](https://www.acwing.com/problem/content/794/)

## 1.3 大整数乘法
一个高精度的正整数A，乘上一个低精度的正整数b。
### 代码模板
```go
package main

import "fmt"

func main() {
    var a string
    var b int
    fmt.Scanf("%s", &a)
    fmt.Scanf("%d", &b)
    if b == 0 {
        fmt.Print(0)
    } else {
        A := make([]int, 0, len(a))
        for i := len(a) - 1; i >= 0; i-- { A = append(A, int(a[i] - '0')) }
        C := mul(A, b)
        for i := len(C) - 1; i >= 0; i-- { fmt.Print(C[i]) }
    }
}

// C = A * b
// 0 <= b <= 10000
func mul(A []int, b int) (C []int) {
    for i, t := 0, 0; i < len(A) || t > 0; i++ {
        if i < len(A) { t += A[i] * b }
        C = append(C, t % 10)
        t /= 10
    }
    return C
}
```
### 经典模板题
[793. 高精度乘法](https://www.acwing.com/problem/content/795/)

## 1.4 大整数除法
### 代码模板
```go
package main

import "fmt"

func main() {
    var a string
    var b int
    fmt.Scanf("%s", &a)
    fmt.Scanf("%d", &b)
    A := make([]int, 0, len(a))
    for i := len(a) - 1; i >= 0; i-- { A = append(A, int(a[i] - '0')) }
    C, r := div(A, b)
    for i := len(C) - 1; i >= 0; i-- { fmt.Print(C[i]) }
    fmt.Printf("\n%d", r)
}

// C = A / b, r = A % b
// 1 <= b <= 10000
func div(A []int, b int) (C []int, r int) {
    // 高精度除法需要从最高位开始处理，
    // 与其他算术运算不同
    for i := len(A) - 1; i >= 0 ; i-- {
        r = r * 10 + A[i]
        C = append(C, r / b)
        r %= b
    }
    // 反转C，与其他算术运算输出逻辑保持一致
    reverse(C, 0, len(C) - 1)
    // 去除前导零
    for len(C) > 1 && C[len(C) - 1] == 0 { C = C[:len(C) - 1] }
    return C, r
}

func reverse(a []int, l, r int) {
    for l < r {
        a[l], a[r] = a[r], a[l]
        l++; r--
    }
}
```
### 经典模板题
[794. 高精度除法](https://www.acwing.com/problem/content/796/)  

# 前缀和与差分
## 一维前缀和
### 什么是前缀和？
设有长度为`n`的原数组`a`，则其前缀和`si`为原数组前`i`项的和——`si=a1+a2+...+ai`，当`i=0`时，`s0=0`。

### 如何求前缀和数组？
从前往后遍历原数组，`s[i]=s[i-1]+a[i]`。

### 前缀和的作用
前缀和一般用来在`O(1)`时间复杂度内求解区间和。比如求解原数组`a[3]~a[10]`的区间和，用前缀和求解为`s[10]-s[2]`。

### 代码模板
```go
package main

import (
    "fmt"
    "bufio"
    "os"
)

const N int = 100010

var a[N]int
var b[N]int

func main() {
    in := bufio.NewReader(os.Stdin)
    out := bufio.NewWriter(os.Stdout)
    defer out.Flush()
    var n, m int
    fmt.Fscan(in, &n, &m)
    for i := 1; i <= n; i++ { fmt.Fscan(in, &a[i]) }
    for i := 1; i <= n; i++ { b[i] = b[i - 1] + a[i] }
    for ; m > 0; m-- {
        var l, r int
        fmt.Fscan(in, &l, &r)
        fmt.Fprintln(out, b[r] - b[l-1])
    }
}
```

### 经典模板题
[795. 前缀和](https://www.acwing.com/problem/content/797/)  
[303. 区域和检索 - 数组不可变](https://leetcode.cn/problems/range-sum-query-immutable/)

## 二维前缀和
### 代码模板
```go
package main

import (
    "fmt"
    "bufio"
    "os"
)

func main() {
    in := bufio.NewReader(os.Stdin)
    out := bufio.NewWriter(os.Stdout)
    defer out.Flush()
    const N int = 1010
    var a, s [N][N]int
    var n, m, q int
    fmt.Fscan(in, &n, &m, &q)
    for i := 1; i <= n; i++ {
        for j := 1; j <= m; j++ {
            fmt.Fscan(in, &a[i][j])
        }
    }
    for i := 1; i <= n; i++ {
        for j := 1; j <= m; j++ {
            // 求前缀和
            s[i][j] = s[i - 1][j] + s[i][j - 1] - s[i - 1][j - 1] + a[i][j]
        }
    }
    for ; q > 0; q-- {
        var x1, y1, x2, y2 int
        fmt.Fscan(in, &x1, &y1, &x2, &y2)
        // 求子矩阵的和
        fmt.Fprintln(out, s[x2][y2] - s[x1 - 1][y2] - s[x2][y1 - 1] + s[x1 - 1][y1 - 1])
    }
}
```

### 经典模板题
[796. 子矩阵的和](https://www.acwing.com/problem/content/798/)  
[304. 二维区域和检索 - 矩阵不可变](https://leetcode.cn/problems/range-sum-query-2d-immutable/)

## 一维差分
### 什么是差分？
差分是前缀和的逆运算，对一个差分数组求前缀和，得到原数组。
### 如何求差分数组？
构造这样一个数据，对于原数组`a`而言，差分数组`b`中每一个元素，都等于`b[i] = a[i] - a[i-1]`，其中`b[1] = a[1]`。  
这样构造完之后，我们对其求前缀和可以发现，得到的答案就是原数组。
### 差分的作用
差分数组用于解决区间加值的问题。比如给区间`[l:r]`中每一个数都加上`C`，利用差分数组可以在`O(1)`时间复杂度完成。  
实现过程为，在差分数组`b[l]`处加上`C`，在`b[r+1]`处减去`C`，这样在还原为原数组时，区间`[l:r]`就施加上`C`的影响，在`[r+1:n]`减去`C`的影响。  
公式为：`b[l] += C; b[r+1] -= C`。

图例——为`[4:7]`这个区间加上`3`。
![](https://cdn.jsdelivr.net/gh/hts0000/images/202209162051303.png)

### 代码模板
```go
package main

import (
    "fmt"
    "bufio"
    "os"
)

const N = 100010
var a [N]int
var b [N]int

func main() {
    in := bufio.NewReader(os.Stdin)
    out := bufio.NewWriter(os.Stdout)
    defer out.Flush()
    var n, m int
    fmt.Fscan(in, &n, &m)
    for i := 1; i <= n; i++ {
        fmt.Fscan(in, &a[i])
        insert(i, i, a[i])
    }
    for ; m > 0; m-- {
        var l, r, c int
        fmt.Fscan(in, &l, &r, &c)
        insert(l, r, c)
    }
    for i := 1; i <= n; i++ {
        b[i] += b[i-1]
        fmt.Fprintf(out, "%d ", b[i])
    }
}

func insert(l, r, c int) {
    b[l] += c
    b[r+1] -= c
}
```
### 经典模板题
[797. 差分](https://www.acwing.com/problem/content/799/)

## 二维差分
### 代码模板
```go
package main

import (
    "fmt"
    "bufio"
    "os"
)

const N int = 1010
var a [N][N]int
var b [N][N]int

func main() {
    in := bufio.NewReader(os.Stdin)
    out := bufio.NewWriter(os.Stdout)
    defer out.Flush()
    var n, m, q int
    fmt.Fscan(in, &n, &m, &q)
    // 读取原数组数据，并构建差分数组
    for i := 1; i <= n; i++ {
        for j := 1; j <= m; j++ {
            fmt.Fscan(in, &a[i][j])
            insert(i, j, i, j, a[i][j])
        }
    }
    // q次矩阵增加c值操作
    for ; q > 0; q-- {
        var x1, y1, x2, y2, c int
        fmt.Fscan(in, &x1, &y1, &x2, &y2, &c)
        insert(x1, y1, x2, y2, c)
    }
    // 构建二维前缀和数组
    for i := 1; i <= n; i++ {
        for j := 1; j <= m; j++ {
            b[i][j] += b[i-1][j] + b[i][j-1] - b[i-1][j-1]
        }
    }
    // q次矩阵增加c之后的答案输出
    for i := 1; i <= n; i++ {
        for j := 1; j <= m; j++ {
            fmt.Fprintf(out, "%d ", b[i][j])
        }
        fmt.Fprintln(out)
    }
}

func insert(x1, y1, x2, y2, c int) {
    b[x1][y1] += c
    b[x2+1][y1] -= c
    b[x1][y2+1] -= c
    b[x2+1][y2+1] += c
}
```
### 经典模板题
[798. 差分矩阵](https://www.acwing.com/problem/content/800/)

