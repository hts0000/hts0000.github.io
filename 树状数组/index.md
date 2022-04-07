# 树状数组


<!--more-->

转载请注明出处：`https://hts0000.github.io/`

欢迎与我联系：`hts_0000@sina.com`

[参考](https://halfrost.com/binary_indexed_tree/)

## 树状数组是什么？
树状数组，或称Binary Indexed Tree, Fenwick Tree，**是一种用于高效处理对一个存储数字的列表进行更新及求前缀和的数据结构**。

**树状数组所能解决的典型问题就是存在一个长度为n的数组，我们如何高效进行如下操作：**
- update(idx, delta)：将num加到位置idx的数字上。
- prefixSum(idx)：求从数组第一个位置到第idx（含idx）个位置所有数字的和。
- rangeSum(from_idx, to_idx)：求从数组第from_idx个位置到第to_idx个位置的所有数字的和

如果用暴力求解数组区间和，时间复杂度为`O(n)`，更新数组值为`O(1)`。
```go
// [left,right]
func rangeSum(left, right int) int {
	// 时间复杂度O(n)，与for left<= right 没有数量级上的差距
	return sum(right) - sum(left - 1)
}

// [0,index]
func sum(index int) int {
	if index < 0 || index >= len(nums) {
		return 0
	}
	sum := 0
	for i := index; i >= 0; i-- {
		sum += nums[i]
	}
	return sum
}

// O(1)
func update(index, delta int) {
	nums[index] += delta
}
```

用前缀和求解区间和的时间复杂度为`O(1)`，更新数组值为`O(n)`
```go
type Presum struct {
	nums []int
}

func NewPreSum(nums []int) Presum {
	pre := make([]int, len(nums))
	pre[0] = nums[0]
	for i := 1; i < len(nums); i++ {
		pre[i] = pre[i-1] + nums[i]
	}
	return Presum{
		nums: pre,
	}
}

func (p *Presum) Update(index, val int) {
	// O(n)
	for i := index; i < len(p.nums); i++ {
		p.nums[i] += val
	}
}

func (p *Presum) SumRange(left, right int) int {
	// O(1)
	return p.nums[right] - p.nums[left-1]
}
```

树状数组求解区间和的时间复杂度为`O(logn)`，更新数组值为`O(logn)`。因为修改值之后需要重新维护树状数组。
```go
type BinaryIndTree struct {
	Nums []int
	Tree []int
}

func NewBinaryIndTree(nums []int) BinaryIndTree {
	// 用前缀和快速构造树状数组
	length := len(nums)
	tree := make([]int, length+1)
	preSum := make([]int, length+1)
	for i := 1; i <= length; i++ {
		preSum[i] = preSum[i-1] + nums[i-1]
		tree[i] = preSum[i] - preSum[i-lowbit(i)]
	}
	return BinaryIndTree{
		Nums: nums,
		Tree: tree,
	}
}

func lowbit(i int) int {
	return i & (-i)
}

func (b *BinaryIndTree) Add(index, delta int) {
	// O(logn)
	for i := index; i < len(b.Tree); i += lowbit(i) {
		b.Tree[i] += delta
	}
}

func (b *BinaryIndTree) Update(index, val int) {
	b.Add(index+1, val-b.Nums[index])
	b.Nums[index] = val
}

func (b *BinaryIndTree) Sum(index int) int {
	sum := 0
	// O(logn)
	for i := index; i > 0; i -= lowbit(i) {
		sum += b.Tree[i]
	}
	return sum
}

func (b *BinaryIndTree) SumRange(left, right int) int {
	return b.Sum(right+1) - b.Sum(left)
}
```

树状数组适用于对数组查询和修改都有性能要求的场景。

## 树状数组如何高效？
![](https://cdn.jsdelivr.net/gh/hts0000/images/202204062253873.png)
看上面这张图，可以把树状数组想象成一棵树。所有奇数下标的都是独立子节点。**子节点通过`i+lowbit(i)`来找到其父节点，父节点存储的就是它和它所有子节点的和**。通过这样一棵树，我们可以用`O(logn)`的时间复杂度来获得一个区间的和。

为了方便表示，tree被画成了不连续的样子，实际上tree在内存中仍是连续的数组。

下面这样图可以比较容易理解树状数组在更新节点值时，如何知道那些节点值是应该更新的。

![](https://cdn.jsdelivr.net/gh/hts0000/images/202204062204495.png)

比如我们要对下标`i`的值进行更新，除了对`i`本身，还有其上面覆盖了`i`(或者说管辖`i`)的所有节点进行更新。比如`i=1`，则需要更新的节点为`1、2、4、8`

### lowbit原理
`lowbit(i)`用一句话来解释就是——**截断`i`在二进制表示中最低位1前面的数**。

比如：i=6=b(110)，截断最低位1前面的数得到b(10)=2

lowbit实现很简单
```go
func lowbit(i int) int {
	// return i & ((~i)+1) 两者等价
	return i & (-i)
}
```

为什么`&`上自己的负数可以截断最低位1前面的数呢？因为负数在计算机中是用补码来表示的，所以我们需要对补码、原码有一些了解。
- 原码：用最高位表示符号位，如`int8(-8) = 原(10001000)`
- 补码：对原码除符号位之外的所有位取反并加1，如`int8(-8) = 补(11111000)`
- 反码：对原码除符号位之外的所有位取反，如`int8(-8) = 反(11110111)`
这里我们以6和-6为例，假设他们都是有符号的8位数。
![](https://cdn.jsdelivr.net/gh/hts0000/images/202204071821893.png)

可以发现一个数的正数与负数，它们除了最低位的1及后面的数之外，每一位的数都不相同，因此它们相`&`可以得到我们想要的结果——**截断`i`在二进制表示中最低位1前面的数**。

## 树状数组功能一览
根据节点维护的数据含义不同，树状数组可以提供不同的功能来满足各种区间场景。

### 1. 单点增减+区间求和
[LeetCode-307. 区域和检索 - 数组可修改](https://leetcode-cn.com/problems/range-sum-query-mutable/submissions/)
### 2. 区间增减+单点查询
### 3. 区间增减+区间求和
### 4. 单点增减+区间最值
[HDU-1754 I Hate It](http://acm.hdu.edu.cn/showproblem.php?pid=1754)
### 5. 区间叠加+单点最值
[LeetCode-218. 天际线问题](https://leetcode-cn.com/problems/the-skyline-problem/)

## 树状数组常见应用
1. 求逆序对
2. 求区间逆序对
3. 求树上逆序对

## 二维树状数组

## 完整代码
```go
type BinaryIndTree struct {
	Nums []int
	Tree []int
}

func NewBinaryIndTree(nums []int) BinaryIndTree {
	// 用前缀和快速构造树状数组
	length := len(nums)
	tree := make([]int, length+1)
	preSum := make([]int, length+1)
	for i := 1; i <= length; i++ {
		preSum[i] = preSum[i-1] + nums[i-1]
		tree[i] = preSum[i] - preSum[i-lowbit(i)]
	}
	return BinaryIndTree{
		Nums: nums,
		Tree: tree,
	}
}

func (b *BinaryIndTree) update(index, delta int) {
	for i := index; i < len(b.Tree); i += lowbit(i) {
		b.Tree[i] += delta
	}
}

func (b *BinaryIndTree) Update(index, val int) {
	b.update(index+1, val-b.Nums[index])
	b.Nums[index] = val
}

func (b *BinaryIndTree) Sum(index int) int {
	sum := 0
	for i := index; i > 0; i -= lowbit(i) {
		sum += b.Tree[i]
	}
	return sum
}

func (b *BinaryIndTree) SumRange(left, right int) int {
	return b.Sum(right+1) - b.Sum(left)
}

func lowbit(i int) int {
	return i & (-i)
}
```


