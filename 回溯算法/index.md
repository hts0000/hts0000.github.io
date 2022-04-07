# 回溯算法


<!--more-->

转载请注明出处：`https://hts0000.github.io/`

欢迎与我联系：`hts_0000@sina.com`

### 回溯
回溯就是暴力穷举，对于一些组合、切割、求子集、n皇后等问题，只能使用暴力穷举的方式遍历所有可能性求解。

回溯用于解决n层for循环的问题

对于一个长度为n的序列，要求出它所有的子集，需要写出n层for循环。因为序列长度n未知，所以连用for暴力穷举的代码都写不出来。

这是就要用到回溯，通过递归的方式，不断深入，开启n层for循环。

n层递归的循环，就像是一颗树，循环的次数是树的宽度，递归的次数是树的深度

![](https://cdn.jsdelivr.net/gh/hts0000/images/202203092026827.png)

### 回溯模板
```go
func demo() {
    // 存放结果
    res := make([]int, 0)
    // 存放临时结果
    path := make([]int, 0)

    // 回溯函数的参数很难提前确定
    // 一般在写具体实现是根据需要补上
    var backTracking func(所需参数)
    backTracking = func(所需参数) {
        if 满足要求 {
            // 新申请内存空间，避免底层数组一致
            res = append(res, append([]int{}, path...)...)
            return
        }
        for 遍历序列 {
            path = append(添加临时结果)
            backTracking(所需参数)
            path = path[:len(path)-1]
        }
    }
}
```

### 回溯的去重
去重首先需要序列有序，然后在for循环内部判断当前元素与前一元素是否相同，如果相同就continue

![](https://cdn.jsdelivr.net/gh/hts0000/images/202203092035947.png)

### 标识当前元素已被使用

![](https://cdn.jsdelivr.net/gh/hts0000/images/202203102208044.png)

```go
used := make([]bool, length)

backTracking = func() {
    for i := 0; i < len(nums); i++ {
        if used[i] {
            continue
        }
        used[i] = true
        ...
        backTracking()
        used[i] = false
        ...
    }
}
```
