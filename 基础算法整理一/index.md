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

寻找左边界

```go
func bsearch(nums []int, target int) int {
 if len(nums) == 0 {
  return -1
 }
 l, r := 0, len(nums) - 1
 for l < r {
  mid := (l + r) >> 1
  if nums[mid] >= target {
   r = mid
  } else {
   l = mid + 1
  }
 }
 if nums[l] != target {
  return -1
 }
 return l
}
```

寻找右边界

```go
func bsearch(nums []int, target int) int {
 if len(nums) == 0 {
  return -1
 }
 l, r := 0, len(nums) - 1
 for l < r {
  mid := (l + r + 1) >> 1
  if nums[mid] <= target {
   l = mid
  } else {
   r = mid - 1
  }
 }
 if nums[r] != target {
  return -1
 }
 return r
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

