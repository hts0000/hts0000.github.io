# Bit数组/bitmap


<!--more-->

转载请注明出处：`https://hts0000.github.io/`

欢迎与我联系：`hts_0000@sina.com`

# 什么是bit数组？
bit数组通常为一个无符号数的数组，用每一个元素的每一个bit位来存储数据。

`var bitSlice []uint64{0, 0, 0}`
![bit数组](https://cdn.jsdelivr.net/gh/hts0000/images/202202151428267.png "bit数组")

当我们存储数据，比如存储2时，对64取商和模为0和2，可知需要将2存储在bitSlice[0]的第2个bit位中，因此我们把`bitSlice[0] & 1<<2`即可标识2被存储了

![bit数组](https://cdn.jsdelivr.net/gh/hts0000/images/202202151430776.png "把2存储到bitSlice中")

对于更大的数字，比如66，对64取商和模为1和2，可知需要将66存储在bitSlice[1]的第2个bit位中，因此我们把`bitSlice[1] & 1<<2`即可标识66被存储了
![](https://cdn.jsdelivr.net/gh/hts0000/images/202202151434594.png "把66存储到bitSlice中")

# bit数组有什么用？
bit数组可以使用很小的存储空间，存储更多的数据。对于uint64而言，如果直接存储，一个数要使用8byte空间，64个数则要64*8byte=512byte，而用bit来存，一个uint64的变量可以存储64个数，空间节省了64倍。

如果我们要对40亿条数据进行排序去重，但机器可用内存只有4G，要怎么做呢？

我们先来计算一下，4,000,000,000对64取商和模为62,500,000和0，可知数组最大需要62,500,000个元素，而`62,500,000 * 8byte = 500,000,000byte ≈ 480M`

如果用[]uint32切片直接存储，则需要`4,000,000,000 * 4byte = 16,000,000,000byte ≈ 15G`

使用bit数组，存储时只会将数所在的bit位 置1，所以只要按顺序取出数据，就是排序去重后的数据。

# bit数组应用
bit数组在很多地方都有应用，比如redis中的bitmap就是使用了bit数组的方式实现的。

# bit数组实现
```golang
package bitmap

import (
	"bytes"
	"fmt"
)

// 参考go语言圣经6.5章节

// UINTSIZE 判断当前机器是32位还是64位
// 64位机器^uint(0) = 64位全1，>> 63之后变成只有第一位为1，32 << 1 = 64
// 32位机器>> 63之后全部位变为0，32 << 0 = 32
const UINTSIZE = 32 << (^uint(0) >> 63)

// An IntSet is a set of small non-negative integers.
// Its zero value represents the empty set.
type IntSet struct {
	words []uint
}

// Has reports whether the set contains the non-negative value x.
func (s *IntSet) Has(x int) bool {
	word, bit := x/UINTSIZE, uint(x%UINTSIZE)
	// words长度大于数据所在元素下标，且所在bit位不为0
	return word < len(s.words) && s.words[word]&(1<<bit) != 0
}

// Add adds the non-negative value x to the set.
func (s *IntSet) Add(x int) {
	// 对要存储的数取商和模
	word, bit := x/UINTSIZE, uint(x%UINTSIZE)
	// 如果words数组长度小于商，则扩容至能存下为止
	for word >= len(s.words) {
		s.words = append(s.words, 0)
	}
	// 将数存储到指定bit位
	s.words[word] |= 1 << bit
}

// UnionWith sets s to the union of s and t.
// 设置s为s和t的并集
func (s *IntSet) UnionWith(t *IntSet) {
	for i, tword := range t.words {
		if i < len(s.words) {
			// 取并集
			// 101 | 010 = 111
			s.words[i] |= tword
		} else {
			s.words = append(s.words, tword)
		}
	}
}

// IntersectWith sets s to the intersect of s and t
// 设置s为s和t的交集
// 元素在s集合t集合均出现
func (s *IntSet) IntersectWith(t *IntSet) {
	// 如果s比t长，把s截取到t的长度
	if len(s.words) > len(t.words) {
		s.words = s.words[:len(t.words)]
	}
	// 如果t比s长，也只会比较s中有的部分
	for i, tword := range t.words {
		if i < len(s.words) {
			s.words[i] &= tword
		}
	}
}

// DifferenceWith sets s to the difference of s and t
// 设置s为s和t的差集
// 元素出现在s集合，未出现在t集合
func (s *IntSet) DifferenceWith(t *IntSet) {
	for i, tword := range t.words {
		if i < len(s.words) {
			// ^: 异或运算，值不同则为1，相同为0
			// B           A
			// 100011010 ^ 110101001 = 010110011
			// 010110011 & 110101001 = 010100001
			// 正好是A中有而B中没有的
			s.words[i] &= s.words[i] ^ tword
		}
	}
}

// SymmetricDifference sets s to the symmetric difference of s and t
// 设置s为s和t的并查集
// 元素出现在s但没有出现在t，或者出现在t没有出现在s
func (s *IntSet) SymmetricDifference(t *IntSet) {
	for i, tword := range t.words {
		if i < len(s.words) {
			s.words[i] ^= tword
		} else {
			// 如果s元素个数小于t，则s要把t中多的部分补上
			s.words = append(s.words, tword)
		}
	}
}

// String returns the set as a string of the form "{1 2 3}".
// String方法，fmt.Printf()函数对实现了String方法的类型会直接调用其String()方法进行输出
func (s *IntSet) String() string {
	var buf bytes.Buffer
	buf.WriteByte('{')
	for i, word := range s.words {
		if word == 0 {
			continue
		}
		// 逐位遍历，如果为1，则将其写入缓冲区
		for j := 0; j < UINTSIZE; j++ {
			if word&(1<<uint(j)) != 0 {
				if buf.Len() > len("{") {
					buf.WriteByte(' ')
				}
				// UINTSIZE*i+j还原数实际大小
				// 比如2存储在[0]的第2个比特位
				// UINTSIZE*0+2=2
				fmt.Fprintf(&buf, "%d", UINTSIZE*i+j)
			}
		}
	}
	buf.WriteByte('}')
	return buf.String()
}

// Len return the number of elements
// 返回当前存储了多少个数
func (s *IntSet) Len() int {
	l := 0
	for _, word := range s.words {
		if word == 0 {
			continue
		}
		for j := 0; j < UINTSIZE; j++ {
			if word&(1<<uint(j)) != 0 {
				l++
			}
		}
	}
	return l
}

// Elems returns a slice containing the elements of the set
func (s *IntSet) Elems() []int {
	res := make([]int, s.Len())
	k := 0
	for i, word := range s.words {
		if word == 0 {
			continue
		}
		for j := 0; j < UINTSIZE; j++ {
			if word&(1<<j) != 0 {
				res[k] = UINTSIZE*i + j
				k++
			}
		}
	}
	return res
}

// Remove remove x from the set
// 删除指定数
func (s *IntSet) Remove(x int) {
	word, bit := x/UINTSIZE, uint(x%UINTSIZE)
	// 如果要删除的数超出长度或words为空
	if word > len(s.words) || len(s.words) == 0 {
		return
	}

	// 将指定bit位置0，^为取反符号
	// ^(1<<2)
	// = ^(001 << 2)
	// = ^100
	// = 011
	s.words[word] &= ^(1 << bit)
}

// Clear remove all elements from the set
func (s *IntSet) Clear() {
	for i := range s.words {
		s.words[i] = 0
	}
}

// Copy return a copy of the set
func (s *IntSet) Copy() *IntSet {
	c := new(IntSet)
	c.words = make([]uint, len(s.words))
	copy(c.words, s.words)
	return c
}

// AddAll adds all the non-negative value num from nums to the set.
func (s *IntSet) AddAll(nums ...int) {
	for _, num := range nums {
		s.Add(num)
	}
}
```
