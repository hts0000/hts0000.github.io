# 二分查找


<!--more-->

转载请注明出处：`https://hts0000.github.io/`

欢迎与我联系：`hts_0000@sina.com`

## 二分查找
对于任何有序集合，我们可以使用二分查找的方式来快速定位到想要的元素。

二分查找的时间复杂度为`O(logn)`

二分查找的实现难点在于边界条件的判断，要注意集合的闭合区间。实现时闭合区间分为两种：`[left,right]`和`[left,right)`，两种不同的闭合区间的代码实现也不同。

### 二分查找的边界条件
[left,right]

```golang
// BinarySearch 查找nums中是否存在target，并返回其下标，如果不存在返回-1
func BinarySearch(nums []int, target int) int {
	length := len(nums)
	left, right := 0, length - 1
	
	for left <= right {
		mid := left + (right - left) / 2
		if nums[mid] < target {
			left = mid + 1
		} else if nums[mid] > target {
			right = mid - 1
		} else {
			return mid
		}
	}
	return -1
}
```

[left,right)

```golang
// BinarySearch 查找nums中是否存在target，并返回其下标，如果不存在返回-1
func BinarySearch(nums []int, target int) int {
	length := len(nums)
	left, right := 0, length
	
	for left < right {
		mid := left + (right - left) / 2
		if nums[mid] < target {
			left = mid + 1
		} else if nums[mid] > target {
			right = mid
		} else {
			return mid
		}
	}
	return -1
}
```

### 二分查找其他应用
二分查找不仅可以用于有序序列，还可以用于部分有序序列，比如[leetcode 153.寻找旋转排序数组中的最小值](https://leetcode-cn.com/problems/find-minimum-in-rotated-sorted-array/)也可以使用二分法。

由此我们可知，二分查找只是一种查找的思想。对于查找一个值，我们要首先想到如何能跳过一些无意义的比较，收缩左右边界，以达到加快搜索的过程。

