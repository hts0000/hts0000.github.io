# 基础算法整理(一)


<!--more-->

转载请注明出处：`https://hts0000.github.io/`

欢迎与我联系：`hts_0000@sina.com`
# 1. 排序
## 1.1 快速排序
### 代码模板
```go
func sortArray(nums []int) []int {
    qucikSort(nums, 0, len(nums) - 1)
    return nums
}

func qucikSort(nums []int, l, r int) {
    if l >= r { return }
    pivot := nums[(l + r) / 2]
    i, j := l - 1, r + 1
    for i < j {
        for i++; nums[i] < pivot; i++ {}
        for j--; nums[j] > pivot; j-- {}
        if i < j { nums[i], nums[j] = nums[j], nums[i] }
    }
    qucikSort(nums, l, j)
    qucikSort(nums, j + 1, r)
}
```

### 经典模板题
[912. 排序数组](https://leetcode.cn/problems/sort-an-array/)  

## 1.2 归并排序
### 代码模板
递归版
```go
func sortArray(nums []int) []int {
    mergeSort(nums, 0, len(nums) - 1)
    return nums
}

func mergeSort(nums []int, left, right int) {
    if left >= right { return }
    mid := (left + right) >> 1
    mergeSort(nums, left, mid)
    mergeSort(nums, mid + 1, right)
    temp := merge(nums[left:mid+1], nums[mid+1:right+1])
    copy(nums[left:], temp)
}

func merge(left, right []int) []int {
    temp := make([]int, len(left) + len(right))
    i, j, k := 0, 0, 0
    for i < len(left) || j < len(right) {
        if i >= len(left) {
            copy(temp[k:], right[j:])
            return temp
        }
        if j >= len(right) {
            copy(temp[k:], left[i:])
            return temp
        }
        if left[i] <= right[j] {
            temp[k] = left[i]
            k++; i++
        } else {
            temp[k] = right[j]
            k++; j++
        }
    }
    return temp
}
```

迭代版
```go
func sortArray(nums []int) []int {
    mergeSort(nums)
    return nums
}

func mergeSort(nums []int) {
    for step := 1; step < len(nums); step += step {
        for i := 0; i + step < len(nums); i += 2 * step {
            temp := merge(nums[i:i+step], nums[i+step:min(i+2*step, len(nums))])
            copy(nums[i:], temp)
        }
    }
}

func merge(left, right []int) []int {
    temp := make([]int, len(left) + len(right))
    i, j, k := 0, 0, 0
    for i < len(left) || j < len(right) {
        if i >= len(left) {
            copy(temp[k:], right[j:])
            break
        }
        if j >= len(right) {
            copy(temp[k:], left[i:])
            break
        }
        if left[i] <= right[j] {
            temp[k] = left[i]
            k++; i++
        } else {
            temp[k] = right[j]
            k++; j++
        }
    }
    return temp
}

func min(a, b int) int {
    if a < b { return a }
    return b
}
```

### 经典模板题
[912. 排序数组](https://leetcode.cn/problems/sort-an-array/)  
[剑指 Offer 51. 数组中的逆序对](https://leetcode.cn/problems/shu-zu-zhong-de-ni-xu-dui-lcof/)

# 2. 二分
## 2.1 整数二分
二分的本质是将一个区间分为两块，一块满足要求，另一块不满足要求，然后就可以寻找这两块区间的边界。  
每次缩小的时候，都要确保剩余区间内有答案。

### 代码模板
```go
// 左闭右开区间
// lowerBound 返回最小的满足 nums[i] >= target 的 i
// 如果数组为空，或者所有数都 < target，则返回 nums.length
// 要求 nums 是非递减的，即 nums[i] <= nums[i + 1]
func lowerBound(a []int, x int) int {
    // [l : r)
    // 循环不变量：
    // a[l - 1] < x
    // a[r] >= x
    l, r := 0, len(a)
    for l < r {
        m := (l + r) >> 1
        if a[m] < x {
            l = m + 1 // 范围缩小到 [m + 1 : r)
        } else {
            r = m // 范围缩小到 [l : m)
        }
    }
    return l // 或者返回 r
}
```

`golang`的标准库中内置了二分查找的`API`——`sort.SearchInts()`，其实现代码跟模板代码类似，也是`lower_bound`的写法，但是进行了抽象。  
把`if a[m] < x`这一句代码进行了抽象，改为传入一个闭包函数，当条件不满足时，缩小左区间，否则缩小右区间。
```go
// src/sort/search.go;l=124
func SearchInts(a []int, x int) int {
    // 闭包函数把 x 包进去了
    return Search(len(a), func(i int) bool { return a[i] >= x })
}

func Search(n int, f func(int) bool) int {
    // 定义循环不变量
    // Define f(-1) == false and f(n) == true.
    // Invariant: f(i-1) == false, f(j) == true.
    i, j := 0, n
    for i < j {
        h := int(uint(i+j) >> 1) // avoid overflow when computing h
        // i ≤ h < j
        if !f(h) {
            i = h + 1 // preserves f(i-1) == false
        } else {
            j = h // preserves f(j) == true
        }
    }
    // i == j, f(i-1) == false, and f(j) (= f(i)) == true  =>  answer is i.
    return i
}
```

一些常用的使用方法
- 寻找`x`的左边界: `l := sort.SearchInts(nums, x)`，如果不存在`x`，则`l = len(nums)`
- 寻找`x`的右边界: `r := sort.SearchInts(nums, x + 1) - 1`，如果不存在`x`，则`r = len(nums)`
- 寻找第一个`>x`的: `sort.SearchInts(nums, x + 1)`
- 寻找第一个`<x`的: `sort.SearchInts(nums, x) - 1`

### 关于 lower_bound 和 upper_bound
`lower_bound`是寻找第一个`>=x`的下标  
`upper_bound`是寻找第一个`>x`的下标

`golang`实现`lower_bound`和`upper_bound`
```go
// 如果不存在 x，返回的下标是 0
func lowerBound(a []int, x int) int {
    return sort.Search(len(a), func(i int) bool { return a[i] >= x })
}

func upperBound(a []int, x int) int {
    return sort.Search(len(a), func(i int) bool { return a[i] > x })
}
```

`lower_bound() - 1`是`<x`的最大下标，`-1`表示不存在  
`upper_bound() - 1`是`<=x`的最大值下标，`-1`表示不存在

### 关于循环不变量
循环不变量指的是在循环中，**性质不变**的量。

比如我们定义`L`的左侧是`<x`的，`R`的右侧是`>=x`的，保证这两个变量的性质，在循环时和循环后不改变，这就叫循环不变量。

### 模板实践
[34. 在排序数组中查找元素的第一个和最后一个位置](https://leetcode.cn/problems/find-first-and-last-position-of-element-in-sorted-array/)
```go
func searchRange(nums []int, target int) []int {
    // 寻找左边界
    l := lowerBound(nums, target)
    if l == len(nums) || nums[l] != target {
        return []int{-1, -1}
    }
    // 左边界存在，右边界肯定存在
    // 寻找右边界，第一个 >=target+1 的下标，这个下标-1 就是 target 的右边界
    r := lowerBound(nums, target + 1) - 1
    return []int{l, r}
}

func lowerBound(a []int, x int) int {
    l, r := 0, len(a)
    for l < r {
        m := (l + r) >> 1
        if a[m] < x {
            l = m + 1
        } else {
            r = m
        }
    }
    return l
}
```

用标准库函数
```go
func searchRange(nums []int, target int) []int {
    l := sort.SearchInts(nums, target)
    if l == len(nums) || nums[l] != target {
        return []int{-1, -1}
    }
    r := sort.SearchInts(nums, target + 1) - 1
    return []int{l, r}
}
```

### 经典模板题
[704. 二分查找](https://leetcode.cn/problems/binary-search/)  
[34. 在排序数组中查找元素的第一个和最后一个位置](https://leetcode.cn/problems/find-first-and-last-position-of-element-in-sorted-array/)

## 2.2 浮点数二分
### 代码模板
```go
func mySqrt() {
    var x float64
	fmt.Scanf("%f", &x)
	var l, r float64 = 0, x
	for r - l > 1e-8 {
		mid := (l + r) / 2
		if mid * mid >= x {
			r = mid
		} else {
			l = mid
		}
	}
	fmt.Printf("%f", l)
}
```

### 经典模板题
[69. x 的平方根](https://leetcode.cn/problems/sqrtx/)  
[790. 数的三次方根](https://www.acwing.com/problem/content/792/)

