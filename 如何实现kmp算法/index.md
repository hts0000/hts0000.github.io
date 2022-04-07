# 如何实现kmp算法


<!--more-->

转载请注明出处：`https://hts0000.github.io/`

欢迎与我联系：`hts_0000@sina.com`

# 什么是KMP算法？
`kmp`算法是一种用于高效匹配字符串子串是否存在的算法，比如想知道`文本串s`中是否存在`模式串p`，就可以使用kmp算法。

# 如何实现KMP算法？
实现kmp算法分两步
1. 基于模式串初始化next数组
2. 使用next数组加速字符串匹配

## 什么是next数组？
next数组，又称为prefix数组、前缀表，其作用是记录给定字符串的所有子串中**相等的前后缀长度**。

### 什么是前缀后缀？
对于字符串`cbccbcbccc`，其前缀为**包含首字符不包含尾字母**的任意子串，其后缀为**包含尾字符不包含首字符**的任意子串。

**前缀**
```
c
cb
cbc
cbcc
...
cbccbcbcc
```
**后缀**
```
c
cc
ccc
bccc
...
bccbcbccc
```

对于字符串`cbccbcbccc`的每一个子串，我们求出它们的最长相等前后缀长度，其中子串`c`初始化为0。
![最长相等前后缀](https://cdn.jsdelivr.net/gh/hts0000/images/202202111728900.jpg "最长相等前后缀")

## next数组具体实现
求next数组的代码如下
```go
func getNext(s string) []int {
    sLen := len(s)
    next := make([]int, sLen)
    next[0] = 0
    j := 0

    for i := 1; i < sLen; i++ {
        // j要保证大于0，因为下面有取j-1作为数组下标的操作
        for j > 0 && s[i] != s[j] {
            // 回退前一位
            j = next[j - 1]
        }
        if s[i] == s[j] {
            j++
        }
        next[i] = j
    }
    return next
}
```

求出next数组后，我们就可以利用它加速字符串的匹配。

当遇到不匹配时，就往前找一个最长相等前后缀，并移动模式串指针到他的后面，再尝试匹配
![不匹配](https://cdn.jsdelivr.net/gh/hts0000/images/202202111925536.png "不匹配")

![匹配](https://cdn.jsdelivr.net/gh/hts0000/images/202202111926824.png "匹配")

当`i==j==5`时，`文本串s[5] != 模式串p[5]`，这时我们在next数组中往前找一位，next数组中记录了**p[0:5]这个子串最长相等前后缀长度为2**，意味着我们j指针只需要回退到下标为2的位置就可以继续匹配了，跳过了前面相同的部分

# kmp具体实现
```go
func kmp(haystack string, needle string) int {
	if len(needle) == 0 {
		return 0
	}
	next := getNext(needle)
    // 模式串的起始位置 next为0 因此也为0
	j := 0
	for i := 0; i < len(haystack); i++ {
		for j > 0 && haystack[i] != needle[j] {
            // 寻找下一个匹配点
			j = next[j-1]
		}
		if haystack[i] == needle[j] {
			j++
		}
        // j指向了模式串的末尾
		if j == len(needle) {
			return i - len(needle) + 1
		}
	}
	return -1
}

func getNext(s string) []int {
    sLen := len(s)
    next := make([]int, sLen)
    next[0] = 0
    j := 0

    for i := 1; i < sLen; i++ {
        // j要保证大于0，因为下面有取j-1作为数组下标的操作
        for j > 0 && s[i] != s[j] {
            // 回退前一位
            j = next[j - 1]
        }
        if s[i] == s[j] {
            j++
        }
        next[i] = j
    }
    return next
}
```

