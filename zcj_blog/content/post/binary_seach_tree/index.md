---
title: "数据结构-Binary search tree go实现"
date: 2022-01-05T21:00:08+08:00
draft: false
image: 1.gif
categories:
    - language
    - golang
---
## 二叉查找树

### 数据结构定义
```


// 定义树的节点
type binarySearchTreeNode struct {
	info  int    // 定义存储的内容
	left  *binarySearchTreeNode  // 左节点
	right *binarySearchTreeNode  // 右节点
}

// 定义树
type BinarySearchTree struct {
	root   *binarySearchTreeNode   // 定义根节点
	numNodes  int                    // 节点树
}

// 创建节点
func creatNode(data int) *binarySearchTreeNode{
	return &binarySearchTreeNode{
		info:  data,
		left:  nil,
		right: nil,
	}
}

// 节点的数量
func (b *BinarySearchTree)GetNodeNum()int{
	return b.numNodes
}
```

###  数据操作
* 增删改查
#### 数据的插入
```

// 节点的插入
func (b *BinarySearchTree) InsertNode(data int)  {
	newNode := creatNode(data)
	// 如果为空树
	if b.root == nil{
		b.root = newNode
		b.numNodes++
		return
	}
	node := b.root
	for {
		// 当值小于节点的值时，就往左子树插入
		if data < node.info{
			// 左节点为空，直接插入,跳出循环
			if node.left == nil{
				node.left = newNode
				break
			}else{
				// 否则继续往左子树插
				node = node.left
			}
		}else{
			// 当值大于等于节点的值时，就往右子树插入
			// 右节点为空，直接插入,跳出循环
			if node.right == nil{
				node.right = newNode
				break
			}else {
				// 否则继续往右子树插
				node = node.right
			}
		}
	}
	// 节点树+1
	b.numNodes++
}

```

#### 数据的删除
1. 复制删除法：
>将被删除节点的左子树的最大值或者右子树的最小值复制给被删除的节点的数值，
>然后删除左子树的最大值的节点或者右子树的最小值的节点

2.合并删除法：
>从删除的节点的两棵子树合并未一棵树，然后将这颗树连接到删除节点的父节点
>具体操作：左子树的最大值左作为有子树的父节点，左子树的根节点作为这棵树的根节点
>或右子树的最小值作为左子树的父节点，右子树的根节点作为这棵树的根节点
```

// 清空树
func (b *BinarySearchTree) Clear(){
	b.root = nil
	b.numNodes = 0
}

//合并删除法：
//从删除的节点的两棵子树合并未一棵树，然后将这颗树连接到删除节点的父节点
//具体操作：左子树的最大值左作为有子树的父节点，左子树的根节点作为这棵树的根节点
//或右子树的最小值作为左子树的父节点，右子树的根节点作为这棵树的根节点
//
func (b *BinarySearchTree)deleteByMerge(node **binarySearchTreeNode) {
	// 如果右节点为nil
	if (*node).right == nil{
		*node = (*node).left

	}else if (*node).left == nil{
		*node = (*node).right
	}else{
		// 找出左子树的最大值
		temp := (*node).left
		for ; temp.right != nil; temp = temp.right{}
		// 左子树的最大值的右节点指向右子树
		temp.right = (*node).right
		// node的头指针指向这个树既指向左子树的头节点
		*node = (*node).left
	}


}

// 移除节点，通过合并的方法
func (b *BinarySearchTree)RemoveByMerge(data int) bool{
	// 如果为空树
	if b.root == nil{
		return false
	}
	// 找到值的节点
	node := b.root
	prev := b.root
	for {
		if node == nil{
			return false
		}
		if node.info == data{
			break
		}
		prev = node
		if data < node.info{
			node = node.left
		}else {
			node = node.right
		}
	}
	if node == b.root{
		b.deleteByMerge(&b.root)
	}else if node == prev.left{
		b.deleteByMerge(&prev.left)
	}else{
		b.deleteByMerge(&prev.right)
	}
	// 节点树-1
	b.numNodes--
	return true
}

/*
复制删除法：
将被删除节点的左子树的最大值或者右子树的最小值复制给被删除的节点的数值，
然后删除左子树的最大值的节点或者右子树的最小值的节点
*/
func (b *BinarySearchTree)deleteByCopy(node **binarySearchTreeNode) {
	// 如果右节点为nil
	if (*node).right == nil{
		*node = (*node).left
	}else if (*node).left == nil{
		*node = (*node).right
	}else{
		// 找出左子树的最大值
		temp := (*node).left
		prev := *node  // 左子树最大值的最前驱节点
		for ; temp.right != nil; temp = temp.right{
			prev = temp
		}
		// 左子树的最大值复制给node
		(*node).info = temp.info
		if prev == *node{ // 如果左子树的最大值为node的node的子节点
			prev.left = temp.left  // 先驱节点的左节点指向左子树最大节点的左节点
		}else{
			prev.right = temp.left  // 先驱节点的右节点指向左子树最大节点的右节点
		}

	}


}

// 移除节点，通过复制的方法
func (b *BinarySearchTree)RemoveByCopy(data int) bool{
	// 如果为空树
	if b.root == nil{
		return false
	}
	// 找到值的节点
	node := b.root
	prev := b.root
	for {
		if node == nil{
			return false
		}
		if node.info == data{
			break
		}
		prev = node
		if data < node.info{
			node = node.left
		}else {
			node = node.right
		}
	}
	//b.deleteByCopy(&node)
	if node == b.root{
		b.deleteByCopy(&b.root)
	}else if node == prev.left{
		b.deleteByCopy(&prev.left)
	}else{
		b.deleteByCopy(&prev.right)
	}
	// 节点树-1
	b.numNodes--
	return true
}
```
#### 数据的查询
* 前序遍历、中序遍历、后续遍历、值查询
```
// 前序遍历
func (b *BinarySearchTree) nLR(node *binarySearchTreeNode){
	if node == nil{
		return
	}
	fmt.Printf("%d->", node.info)
	b.nLR(node.left)
	b.nLR(node.right)
}

// 中序遍历
func (b *BinarySearchTree)lNR(node *binarySearchTreeNode){
	if node == nil{
		return
	}
	b.lNR(node.left)
	fmt.Printf("%d->", node.info)
	b.lNR(node.right)
}

// 后序遍历
func (b *BinarySearchTree)lRN(node *binarySearchTreeNode){
	if node == nil{
		return
	}
	b.lRN(node.left)
	b.lRN(node.right)
	fmt.Printf("%d->", node.info)
}

// 前序显示
func (b *BinarySearchTree)ShowNLR(){
	if b.root == nil{
		fmt.Println("zhe tree is empty")
		return
	}
	fmt.Print("NLR:")
	node := b.root
	b.nLR(node)
	fmt.Println()
}

// 中序显示
func (b *BinarySearchTree)ShowLNR(){
	if b.root == nil{
		fmt.Println("zhe tree is empty")
		return
	}
	fmt.Print("LNR:")
	node := b.root
	b.lNR(node)
	fmt.Println()
}

// 后序显示
func (b *BinarySearchTree)ShowLRN(){
	if b.root == nil{
		fmt.Println("zhe tree is empty")
		return
	}
	fmt.Print("LRN:")
	node := b.root
	b.lRN(node)
	fmt.Println()
}

// 查找值是否存在
func (b *BinarySearchTree)IsValue(data int) bool {
	if b.root == nil{
		return false
	}
	node :=  b.root
	for {
		if node == nil{
			return false
		}
		if node.info == data {
			return  true
		}
		if data < node.info{
			node = node.left
		}else {
			node = node.right
		}
	}
}
```

### 树的创建
```
func NewBinarySearchTree() *BinarySearchTree {
	return &BinarySearchTree{
		root:     nil,
		numNodes: 0,
	}
}
```
## 源码
* [我的github:https://github.com/zcj-git520/DataStructuresAlgorithmsForGo/tree/master/tree/binary_search_tree](https://github.com/zcj-git520/DataStructuresAlgorithmsForGo/tree/master/tree/binary_search_tree)


