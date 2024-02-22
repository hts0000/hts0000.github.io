# 排序算法


<!--more-->

转载请注明出处：`https://hts0000.github.io/`

欢迎与我联系：`hts_0000@sina.com`

十大排序算法一览
![](https://cdn.jsdelivr.net/gh/hts0000/images/202204032131522.png)


# 比较排序
## 平均时间复杂度O(n^2)

### 冒泡排序
将最大的数放到最后，然后序列范围-1，继续把当前序列内最大的放到最后
```go
func BubbleSort[T constraints.Ordered](nums []T) {
	length := len(nums)
	for i := 0; i < length; i++ {
		for j := 1; j < length-i; j++ {
			if nums[j-1] > nums[j] {
				nums[j], nums[j-1] = nums[j-1], nums[j]
			}
		}
	}
}
```

### 选择排序

### 插入排序

## 平均时间复杂度O(nlogn)

### 归并排序

### 快速排序(quick sort)
#### 概念

#### C语言中的API
```
void qsort(void *base, size_t nitems, size_t size, int (*compar)(const void *, const void*));
```

| 参数   | 说明                                                     |
| ------ | -------------------------------------------------------- |
| base   | 指向要排序的数组的第一个元素的指针                       |
| nitems | 由base指向的数组中的元素个数                             |
| size   | 数组中每个元素的大小，以字节为单位                       |
| compar | 用来比较两个元素的函数，即函数指针（比较算法的回调函数） |

**使用`void *`是因为要适配任何类型**

compar函数实现
```c
/*
  如果compar返回值小于0，则p1所指向元素会被排在p2所指向元素的左面
  如果compar返回值等于0，则p1所指向元素与p2所指向元素的顺序不确定
  如果compar返回值大于0，则p1所指向元素会被排在p2所指向元素的右面
*/
int compar(const void *pi, const void *p2);

// 递增排序实现
int cmp(const void *p1, const void *p2) {
	int v1 = *(int *)p1;
	int v2 = *(int *)p2;
	if (v1 < v2) {
		return -1;
	} else if (v1 > v2) {
		return 1;
	}
	return 0;
}

// 递增排序实现简化写法（需保证相减不会超过32位整型）
int cmp(const void *p1, const void *p2) {
	return (*(int *)p1) - (*(int *)p2);
}

/*******************************************************************/

// 递减排序实现
int cmp(const void *p1, const void *p2) {
	return (*(int *)p2) - (*(int *)p1);
}

/*******************************************************************/

// 偶数在前奇数在后
int Qua(int x) {
	return x % 2;
}

int cmp(const void *p1, const void *p2) {
	return Qua(*(int *)p1) - Qua(*(int *)p2);
}
```

### 堆排序

### 希尔排序


# 非比较排序

### 基数排序

### 计数排序

### 桶排序


