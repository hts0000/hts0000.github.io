# 树的递归遍历


<!--more-->

转载请注明出处：`https://hts0000.github.io/`

欢迎与我联系：`hts_0000@sina.com`

树的遍历方式分为如下几种：
- 前序遍历
- 中序遍历
- 后序遍历
- 层序遍历

其中前序遍历可以求出树的深度，而后序遍历可以求出树的高度
![](https://cdn.jsdelivr.net/gh/hts0000/images/202202211634775.png)



树的遍历也可以借用辅助数据结构来完成：
- 栈
- 队列

栈适合模拟前中后序遍历，队列适合模拟层序遍历

### 前序遍历求深度
```golang
func maxDepth(root *TreeNode) int {
	if root == nil {
		return 0
	}
	leftDepth := maxDepth(root.Left)
	rightDepth := maxDepth(root.Right)
	return max(leftDepth, rightDepth) + 1
}
```

