# 基础算法整理(四)


<!--more-->

转载请注明出处：`https://hts0000.github.io/`

欢迎与我联系：`hts_0000@sina.com`

# 链表与邻接表
## 单链表
链表通常使用结构体加指针的方式来构建：
```go
type Node struct {
	Val int
	Next *Node
}
```
但是这种方式创建链表很慢，笔试算法中链表长度可能达到10^5甚至10^6级别，只是初始化链表就超时了。所以笔试算法通常使用数组来模拟链表，用数组模拟链表也被称为静态链表。  
用数组模拟链表：
```go
// 存储节点i的值
var e = [N]int{0, 1, 2, 3, 4, 5}
// 存储节点i的next，这里存储的实际是数组下标，-1表示nil
var ne = [N]int{3, 1, 2, 5, 4, -1}
// [0]->[3]->[1]->[2]->[5]->[4]->nil
```

单链表最大的用途是用来写邻接表，存储树和图。  
双链表通常用于优化某些问题。

### 代码模板
```go
package main

import (
    "fmt"
    "bufio"
    "os"
)

const N = 100010
// 指向头节点，指示链表长度
var head, length int
var e, ne [N]int

func main() {
    in := bufio.NewReader(os.Stdin)
    out := bufio.NewWriter(os.Stdout)
    defer out.Flush()
    
    var m int
    fmt.Fscan(in, &m)
    
    initLinkList()
    
    for ; m > 0; m-- {
        var op string
        var k, x int
        fmt.Fscan(in, &op)
        if op == "H" {
            fmt.Fscan(in, &x)
            addToHead(x)
        } else if op == "D"{
            fmt.Fscan(in, &k)
            // 如果删除的节点是头结点，直接将head指向下一位
            if k == 0 {
                head = ne[head]
            } else {
                remove(k - 1)
            }
        } else {
            fmt.Fscan(in, &k, &x)
            add(k - 1, x)
        }
    }
    for i := head; i != -1; i = ne[i] {
        fmt.Fprintf(out, "%d ", e[i])
    }
}

// 初始化链表
func initLinkList() {
    head = -1
    length = 0
}

// 将x插到头结点
func addToHead(x int) {
    e[length] = x
    ne[length] = head
    head = length
    length++
}

// 将x插到下标是k的后面
func add(k, x int) {
    e[length] = x
    ne[length] = ne[k]
    ne[k] = length
    length++
}

// 删除下标k后面的一个节点
func remove(k int) {
    ne[k] = ne[ne[k]]
}
```

### 经典模板题
[826. 单链表](https://www.acwing.com/problem/content/828/)
[707. 设计链表](https://leetcode.cn/problems/design-linked-list/)

## 双链表
双链表比单链表多存储了一个前节点指针，能够在`O(1)`时间找到上一个节点。
### 代码模板
```go
package main

import (
    "fmt"
    "bufio"
    "os"
)

const N = 100010
// l存放i节点的左节点；r存放i节点的右节点
var e, l, r [N]int
var length int

func main() {
    in := bufio.NewReader(os.Stdin)
    out := bufio.NewWriter(os.Stdout)
    defer out.Flush()
    initDuLinkNode()
    var m int
    fmt.Fscan(in, &m)
    for ; m > 0; m-- {
        var op string
        fmt.Fscan(in, &op)
        var k, x int
        if op == "R" {
            fmt.Fscan(in, &x)
            // 1表示最右端点
            // l[1]表示最右端点的左侧
            add(l[1], x)
        } else if op == "L" {
            fmt.Fscan(in, &x)
            // 0表示最左端点
            add(0, x)
        } else if op == "D" {
            fmt.Fscan(in, &k)
            // 删除第k个插入的数
            // 因为预先插入了两个左右端点，所以需+2
            // 又因为下标从0开始，插入数从1开始，所以-1
            remove(k + 1)
        } else if op == "IR" {
            fmt.Fscan(in, &k, &x)
            // 在第k个插入数的右侧插入x
            add(k + 1, x)
        } else if op == "IL" {
            fmt.Fscan(in, &k, &x)
            // 在第k个插入数的左侧插入x
            // 等价于在 第k个插入数的左节点的右侧 插入x
            add(l[k + 1], x)
        }
    }
    for i := r[0]; i != 1; i = r[i] {
        fmt.Fprintf(out, "%d ", e[i])
    }
}

func initDuLinkNode() {
    // 定义第一个和第二个节点为左右端点
    r[0] = 1
    l[1] = 0
    length = 2
}

// 在下标k的右边插入x
func add(k, x int) {
    e[length] = x
    r[length] = r[k]
    l[length] = k
    l[r[k]] = length
    r[k] = length
    length++
}

// 删除下标是k的点
func remove(k int) {
    r[l[k]] = r[k]
    l[r[k]] = l[k]
}
```

### 经典模板题
[827. 双链表](https://www.acwing.com/problem/content/829/)
[707. 设计链表](https://leetcode.cn/problems/design-linked-list/)

## 邻接表
邻接表存储的是一个节点和它的边。存储形式为一个矩阵，每个元素代表一个节点及它的所有边。  
邻接表多用于存储树和图。

# 栈与队列

## 栈
一种先进后出的数据结构。
### 代码模板
```go
package main

import (
    "fmt"
    "bufio"
    "os"
)

const N int = 100010
var st [N]int
var tt int = -1

func main() {
    in := bufio.NewReader(os.Stdin)
    out := bufio.NewWriter(os.Stdout)
    defer out.Flush()
    
    var m int
    fmt.Fscan(in, &m)
    for ; m > 0; m-- {
        var op string
        fmt.Fscan(in, &op)
        if op == "push" {
            var x int
            fmt.Fscan(in, &x)
            push(x)
        } else if op == "pop" {
            _ = pop()
        } else if op == "empty" {
            fmt.Fprintln(out, empty())
        } else {
            fmt.Fprintln(out, query())
        }
    }
}

func push(x int) {
    tt++
    st[tt] = x
}

func pop() int {
    x := st[tt]
    tt--
    return x
}

func empty() string {
    if tt == -1 {
        return "YES"
    }
    return "NO"
}

func query() int {
    return st[tt]
}
```
### 经典模板题
[828. 模拟栈](https://www.acwing.com/problem/content/830/)
[3302. 表达式求值](https://www.acwing.com/problem/content/3305/)
[剑指 Offer II 036. 后缀表达式](https://leetcode.cn/problems/8Zf90G/)

## 单调栈
通常用于解决，寻找每一个数左边离它最近的比它小的数，或寻找每一个数左边离它最近的比它大的数，或右边最近比它小小，诸如此类的问题。

### 代码模板
```go
package main

import (
    "fmt"
    "bufio"
    "os"
)

const N int = 100010
var st [N]int
var tt int = -1

func main() {
    in := bufio.NewReader(os.Stdin)
    out := bufio.NewWriter(os.Stdout)
    defer out.Flush()
    var m int
    fmt.Fscan(in, &m)
    for i := 0; i < m; i++ {
        var x int
        fmt.Fscan(in, &x)
        for tt > -1 && st[tt] >= x {
           tt-- 
        }
        if tt > - 1 {
            fmt.Fprintf(out, "%d ", st[tt])
        } else {
            fmt.Fprint(out, "-1 ")
        }
        tt++
        st[tt] = x
    }
}
```

### 经典模板题
[830. 单调栈](https://www.acwing.com/problem/content/832/)

## 队列
### 代码模板
```go
package main

import "fmt"

const N int = 100010
var que [N]int
var qh, qt int = 0, -1

func main() {
    var m int
    fmt.Scan(&m)
    for ; m > 0; m-- {
        var op string
        fmt.Scan(&op)
        if op == "push" {
            var x int
            fmt.Scan(&x)
            push(x)
        } else if op == "pop" {
            _ = pop()
        } else if op == "empty" {
            fmt.Println(empty())
        } else {
            fmt.Println(query())
        }
    }
}

func push(x int) {
    qt++
    que[qt] = x
}

func pop() int {
    x := que[qh]
    qh++
    return x
}

func empty() string {
    if qh > qt {
        return "YES"
    }
    return "NO"
}

func query() int {
    return que[qh]
}
```

### 经典模板题
[829. 模拟队列](https://www.acwing.com/problem/content/831/)

## 单调队列
经典应用是**求滑动窗口中的最值**。

### 代码模板
```golang
package main

import (
    "fmt"
    "bufio"
    "os"
)

const N int = 1000010
var a [N]int
var deque [N]int

func main() {
    in := bufio.NewReader(os.Stdin)
    out := bufio.NewWriter(os.Stdout)
    defer out.Flush()
    var n, k int
    fmt.Fscan(in, &n, &k)
    for i := 0; i < n; i++ { fmt.Fscan(in, &a[i]) }

	// 求窗口中的最小值
	hh, tt int := 0, -1
    for i := 0; i < n; i++ {
        // 队头元素已经超出窗口范围，将队头元素弹出
        if hh <= tt && i - k + 1 > deque[hh] { hh++ }
        // 重新维护单调队列，从队尾弹出比当前元素大的元素
        for hh <= tt && a[deque[tt]] >= a[i] { tt-- }
        tt++; deque[tt] = i // 队列存储的是下标
        if i >= k - 1 { fmt.Fprintf(out, "%d ", a[deque[hh]]) }
    }
    fmt.Fprintln(out)

	// 求窗口中的最大值
    hh, tt = 0, -1
    for i := 0; i < n; i++ {
        // 队头元素已经超出窗口范围
        if hh <= tt && i - k + 1 > deque[hh] { hh++ }
        // 重新维护单调队列，从队尾弹出比当前元素小的元素
        for hh <= tt && a[deque[tt]] <= a[i] { tt-- }
        tt++; deque[tt] = i
        if i >= k - 1 { fmt.Fprintf(out, "%d ", a[deque[hh]]) }
    }
}
```

### 经典模板题
[154. 滑动窗口](https://www.acwing.com/problem/content/156/)
[239. 滑动窗口最大值](https://leetcode.cn/problems/sliding-window-maximum/)

# kmp算法
[kmp算法](http://hts0000.top/%E5%A6%82%E4%BD%95%E5%AE%9E%E7%8E%B0kmp%E7%AE%97%E6%B3%95/)

