# 基础算法整理(三)


<!--more-->

转载请注明出处：`https://hts0000.github.io/`

欢迎与我联系：`hts_0000@sina.com`

# 双指针
双指针算法一般有两种场景：
- 两个指针分别指向两个不同的集合
- 两个指针指向同一个集合，本质是维护该集合上的一段区间

双指针一般用于将暴力穷举O(n^2)的算法，优化到O(n)。

### 代码模板
```go
package main

import (
    "fmt"
)

func main() {
    var n int
    fmt.Scan(&n)
    nums := make([]int, n)
    for i := 0; i < n; i++ { fmt.Scan(&nums[i]) }
    m := make([]int, 1e5 + 10)
    ans := 0
    for i, j := 0, 0; i < n; i++ {
        m[nums[i]]++
        for m[nums[i]] > 1 {
            m[nums[j]]--
            j++
        }
        ans = max(ans, i - j + 1)
    }
    fmt.Print(ans)
}

func max(a, b int) int {
    if a > b { return a }
    return b
}
```

### 经典模板题
[799. 最长连续不重复子序列](https://www.acwing.com/problem/content/801/)  
[800. 数组元素的目标和](https://www.acwing.com/problem/content/description/802/)  
[3. 无重复字符的最长子串](https://leetcode.cn/problems/longest-substring-without-repeating-characters/)  
[2816. 判断子序列](https://www.acwing.com/problem/content/2818/)

# 位运算
常用的位运算操作：
- 取出n的二进制表示下第k位是多少——`(n>>k)&1`
- 返回n的二进制表示下最后一位1，也叫`lowbit`操作——`n&-n`

## 取出第k位
用0表示第一位。
### 代码模板
```go
package main

import "fmt"

func main() {
    n := 10
    for k := 3; k >= 0; k-- {
        fmt.Print(n>>k&1)
    }
}
```

## lowbit
公式为：`x & -x`。  
原理为：  
负数在计算机中使用补码的方式表示，因此`-x = ^x + 1`，当`x`取反之后，`x`的最后的1会变成0，而最后的1后面的0会变成1，再加上1的话，会进位一直到最后的1处。因此`-x`的二进制表达形——最后一个1前与`x`全部取反，最后一个1及后面全部一致，然后`x & -x`就会得到最后一个1及后面的0组成的二进制数。

一般形式：
```
x      = 0b(10101010000)
^x     = 0b(01010101111)
^x + 1 = 0b(01010110000)
计算过程：
x & -x
= 0b(10101010000) & \
  0b(01010110000)
= 0b(00000010000)
```

`lowbit`的用处有很多，常见的有：快速计算一个数有多少个位为1。

### 代码模板
```go
package main

import "fmt"

// 计算二进制表示中1的个数
func main() {
    var n int
    fmt.Scan(&n)
    for i := 0; i < n; i++ {
        var num, res int
        fmt.Scan(&num)
        for num > 0 {
            num -= lowbit(num)
            res++
        }
        fmt.Printf("%d ", res)
    }
}

func lowbit(x int) int {
    return x & -x
}
```

### 经典模板题
[801. 二进制中1的个数](https://www.acwing.com/problem/content/803/)  
[191. 位1的个数](https://leetcode.cn/problems/number-of-1-bits/)  

# 离散化
这里单独指整数离散化。离散化是指将一个稀疏的区间离散到一个紧凑的区间中。比如有一个区间范围为`[-10^9 ~ 10^9]`，但是里面只有的数只有`10^5`范围，那么显然这个区间是稀疏的，有很多重复或空的值。离散化就是把这个稀疏空间映射到紧凑空间上，降低操作的时空间复杂度。

### 代码模板
```go
func main() {
    // 存储所有需要操作的下标
    alls := make([]int, 0, N)

    // 对所有需要操作的下标排序去重
    quickSort(&alls, 0, len(alls) - 1)
    alls = unique(alls)
    
    // 二分求出离散化后的值
    find(alls, x)
}

func unique(a []int) []int {
    j := 0
    for i := 0; i < len(a); i++ {
        if i == 0 || a[i] != a[i - 1] {
            a[j] = a[i]
            j++
        }
    }
    // 不应该直接返回a的切片，这样会导致底层大数组的引用没有消失，gc就无法回收
    // return a[:j]
    
    // 返回一个新开辟的切片，底层数组不同，gc可以将a回收掉
    temp := make([]int, j)
    copy(temp, a[:j])
    return temp
}

func find(a []int, x int) int {
    l, r := 0, len(a) - 1
    for l < r {
        mid := (l + r) >> 1
        if a[mid] >= x {
            r = mid
        } else {
            l = mid + 1
        }
    }
    return r + 1
}
```

### 经典模板题
[802. 区间和](https://www.acwing.com/problem/content/description/804/)

# 区间合并
合并满足某种条件的区间，本质是贪心算法。  
经典例题是合并有交集的区间，排序后判断左右端点重合则重新维护最小和最大端点。

### 代码模板
```go
func merge(segs []pair) []pair {
    // 首先从小到大排序区间
    sort.Slice(segs, func(i, j int) bool { return segs[i].first < segs[j].first })
    
    temp := make([]pair, 0)
    // 赋予开始和结束端点最小值，避免重复
    var st, ed int = math.MinInt64, math.MinInt64
    // 遍历所有区间
    for _, seg := range segs {
        // 当有区间与开始和结束端点重叠时，更新最大结束端点
        if seg.first <= ed {
            ed = max(ed, seg.second)
        // 如果没有重叠，说明是一个新区间，更新开始和结束端点
        } else {
            // 避免重复
            if st != math.MinInt64 { temp = append(temp, pair{st, ed}) }
            st = seg.first
            ed = seg.second
        }
    }
    // 避免重复，再加入最后一个区间
    if st != math.MinInt64 { temp = append(temp, pair{st, ed}) }
    return temp
}
```

### 经典模板题
[803. 区间合并](https://www.acwing.com/problem/content/805/)  
[56. 合并区间](https://leetcode.cn/problems/merge-intervals/)

