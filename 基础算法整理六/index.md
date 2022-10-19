# 基础算法整理(六)


<!--more-->

转载请注明出处：`https://hts0000.github.io/`

欢迎与我联系：`hts_0000@sina.com`

# Hash表/哈希表
Hash表通过Hash函数映射的方式，将一个稀疏的集合存储到一个紧凑的集合中。比如一个集合的数据范围是1~10^9次方，但里面的数只有10^5个，我们就可以通过一个Hash函数(通常是取模)的方式将集合中的数映射到一个10^5的集合中，减少存储空间的浪费。

但是将一个大集合映射到小集合，必然造成信息的损失，表现到Hash表中就是**Hash冲突**。通过Hash后，大集合中的某些数必定被映射到了同一个点上。因此我们无法确认该点是否存在某些值，因为有多个值同时被映射过来了。

解决Hash冲突的经典解决方式(都需要额外判断)：
- 拉链法
- 开放寻址法

**拉链法**
每一个点存储的是一个链式结构，冲突时将新值链接到前面或后面。确认是否存在该值时，可以依次判断链上的每一个节点。

**开放寻址法**
如果该点存在冲突，则往下或往上寻找空的位置，将新值写入。确认是否存在该值时，先找到映射的位置，然后向上或向下依次判断每一个值。

### 代码模板
**拉链法**
```go
package main

import (
    "fmt"
    "bufio"
    "os"
)

// 拉链法

const N int = 100003
var h, e, ne [N]int
var idx int

func insert(x int) {
    // x 为负数时 % N 结果为负数，加上 N 变为正数，再 % N
    k := (x % N + N) % N
    e[idx] = x
    ne[idx] = h[k]
    h[k] = idx
    idx++
}

func query(x int) bool {
    k := (x % N + N) % N
    for i := h[k]; i != -1; i = ne[i] {
        if e[i] == x {
            return true
        }
    }
    return false
}

func main() {
    in := bufio.NewReader(os.Stdin)
    out := bufio.NewWriter(os.Stdout)
    defer out.Flush()
    
    // 将h所有槽位初始化为-1，表示空节点
    for i := 0; i < N; i++ { h[i] = -1 }
    var n int
    fmt.Fscan(in, &n)
    for ; n > 0; n-- {
        var op string
        var x int
        fmt.Fscan(in, &op, &x)
        if op == "I" {
            insert(x)
        } else {
            if query(x) {
                fmt.Fprintln(out, "Yes")
            } else {
                fmt.Fprintln(out, "No")
            }
        }
    }
}
```

**开放寻址法**
```go
package main

import (
    "fmt"
    "bufio"
    "os"
)

// 开放寻址法通常开数据范围2~3倍的空间
const N int = 200003
// 用一个不存在于数据范围内的数来表示该点没有值
const null int = 0x3f3f3f3f
var h [N]int

// 在 h 中寻找 x 应该插入的位置
func find(x int) int {
    k := (x % N + N) % N
    // 在h中找一个x能插入的位置，如果x已经插入过了，返回插入的位置
    for h[k] != null && h[k] != x {
        k++
        // 如果到最后了，尝试从头开始查找插入的问题
        if k == N { k = 0 }
    }
    return k
}

func main() {
    in := bufio.NewReader(os.Stdin)
    out := bufio.NewWriter(os.Stdout)
    defer out.Flush()

    // 初始化将所有点标记为空
    for i := 0; i < N; i++ { h[i] = null }
    var n int
    fmt.Fscan(in, &n)
    for ; n > 0; n-- {
        var op string
        var x int
        fmt.Fscan(in, &op, &x)
        k := find(x)
        if op == "I" {
            h[k] = x
        } else {
            if h[k] != null {
                fmt.Fprintln(out, "Yes")
            } else {
                fmt.Fprintln(out, "No")
            }
        }
    }
}
```

### 经典模板题
[840. 模拟散列表](https://www.acwing.com/problem/content/842/)

# 字符串Hash
字符串hash是一种快速判断字符串是否相等的方法。Golang中strings.Index等库均用到了字符串Hash——RK算法。详细可以看[Golang中的字符串匹配——RK算法](http://hts0000.top/golang%E4%B8%AD%E7%9A%84%E5%AD%97%E7%AC%A6%E4%B8%B2%E5%8C%B9%E9%85%8Drk%E7%AE%97%E6%B3%95/)。

### 代码模板
```golang
package main

import (
    "fmt"
    "bufio"
    "os"
)

const N int = 100010
// P表示P进制，题目提到字符串中只包含大小写英文字母和数字，因此128可以完全存下
// ASCII码总共128，131是大于128的最小素数，用素数是因为可以有效的降低冲突的概率
// 这里还有个细节是用到了uint64
// 当字符串足够长时，产生的hash值可能会非常大，int可能会存不下，但是uint64也不一定存的下
// 经验告诉我们，当P取131或13331时，hash值mod一个2^64，在99.99%的情况下不会发生冲突
// 因此用uint64还有个好处，当uint64发生溢出时，就相当于mod了2^64，可以减少许多次计算
// 因此这个字符串hash方法，是假定不会发生冲突的
const P uint64 = 131
// h存储的是s所有前缀的hash值，预处理好之后，求某一段的hash就只需要O(1)的时间
// p存储的是P^i次方数，方便O(1)时间计算hash
var h, p [N]uint64

func get(l, r int) uint64 {
    return h[r] - h[l-1] * p[r-l+1]
}

func main() {
    in := bufio.NewReader(os.Stdin)
    out := bufio.NewWriter(os.Stdout)
    defer out.Flush()
    
    var n, m int
    fmt.Fscan(in, &n, &m)
    var s string
    fmt.Fscan(in, &s)
    s = " " + s
    
    // 预处理前缀hash和P进制每一位数
    p[0] = 1
    for i := 1; i <= n; i++ {
        p[i] = p[i-1] * P
        h[i] = h[i-1] * P + uint64(s[i])
    }
    for ; m > 0; m-- {
        var l1, r1, l2, r2 int
        fmt.Fscan(in, &l1, &r1, &l2, &r2)
        // 因为hash冲突极大概率不存在，因此判断hash值相同就认为字符串相同
        if get(l1, r1) == get(l2, r2) {
            fmt.Fprintln(out, "Yes")
        } else {
            fmt.Fprintln(out, "No")
        }
    }
}
```

### 经典模板题
[841. 字符串哈希](https://www.acwing.com/problem/content/843/)

# 求大于x的最小素数
### 代码模板
```go
// 快速求比128数大的最小素数
for i := 128; ; i++ {
    flag := true
    for j := 2; j * j <= i; j++ {
        if i % j == 0 {
            flag = false
            break
        }
    }
    if flag {
        fmt.Fprintln(out, i)
        break
    }
}
```

