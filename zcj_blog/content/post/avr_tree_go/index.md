---
title: "数据结构-AVL tree go实现"
date: 2022-02-15T21:00:08+08:00
draft: false
image: 1.png
categories:
    - language
    - golang
---
## 平衡二叉树

### 数据结构定义
```
// 定义树的节点
type AVLTreeNode struct {
	info   int  // 定义存储的内容
	height int  // 树的高度
	right  *AVLTreeNode  // 右节点
	left   *AVLTreeNode  // 左节点
}

// 定义树
type AVLTree struct {
	root *AVLTreeNode  // 定义根节点
}

// 创建节点
func creatNode(data int) *AVLTreeNode{
	return &AVLTreeNode{
		info:   data,
		height: -1,
		right:  nil,
		left:   nil,
	}
}
```

###  数据操作
* 增删改查
#### 数据的插入
```

// 数据的插入
func getHeight(node *AVLTreeNode) int {
	if node == nil{
		return -1
	}
	return node.height
}
// 左子树调整
func (a *AVLTree)leftTreeAdjust(node *AVLTreeNode, data int) *AVLTreeNode {
	// 判断是否需要调整树
	if getHeight(node.left) - getHeight(node.right) > 1{
		// 插入的节点在左边上，进右旋
		if data < node.left.info{
			return a.lR(node)
		}else{
			// 进行左右旋
			return a.rLR(node)
		}
	}
	return  node
}

// 右子树调整
func (a *AVLTree)rightTreeAdjust(node *AVLTreeNode, data int) *AVLTreeNode  {
	// 判断是否需要调整树
	if getHeight(node.right) - getHeight(node.left) > 1{
		// 插入的节点在右边上，进左旋
		if data >= node.right.info{
			return a.rR(node)
		}else{
			// 进行右左旋
			return  a.lRR(node)
		}
	}
	return node
}

func (a *AVLTree)insertNode(node **AVLTreeNode, data int)  {
	if *node == nil{
		*node = creatNode(data)
		(*node).height = maxValue(getHeight((*node).left), getHeight((*node).right)) + 1
		return
	}
	// 插入左子树
	if data < (*node).info{
		a.insertNode(&(*node).left, data)
		// 调整树
		*node = a.leftTreeAdjust(*node, data)

	}else{
		// 插入右子树
		a.insertNode(&(*node).right, data)
		// 调整树
		*node = a.rightTreeAdjust(*node, data)
	}
	// 调整节点的高度
	(*node).height = maxValue(getHeight((*node).left), getHeight((*node).right)) + 1
}

func (a *AVLTree)InsertData(data int )  {
	// 如果树为空
	if a.root == nil{
		a.root = creatNode(data)
		a.root.height = 0
		return
	}
	a.insertNode(&a.root, data)
}
```

#### 数据的删除
```
// 清空树
func (a *AVLTree)Clear(){
	a.root = nil
}

// 删除节点
func (a *AVLTree)deleteNode(node **AVLTreeNode, data int)  {
	if *node == nil{
		fmt.Printf("no find data is %d \n", data)
		return
	}
	if (*node).info == data{
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
			a.leftTreeAdjust(*node, data)
		}
		if *node != nil{
			// 调整节点的高度
			(*node).height = maxValue(getHeight((*node).left), getHeight((*node).right)) + 1
		}
		return
	}else if (*node).info > data{
		a.deleteNode(&(*node).left, data)
		a.leftTreeAdjust(*node, data)
	}else {
		a.deleteNode(&(*node).right, data)
		a.rightTreeAdjust(*node,data)
	}
	(*node).height = maxValue(getHeight((*node).left), getHeight((*node).right)) + 1
}

func (a *AVLTree)Remove(data int)  {
	if a.root == nil{
		return
	}
	a.deleteNode(&a.root, data)
}

// 得到最大值
func maxValue(value, value1 int) int {
	if value >= value1{
		return value
	}
	return value1
}
```
#### 数据的查询
* 前序遍历、中序遍历、后续遍历、值查询
```
// 树的遍历
// 前序遍历
func (a *AVLTree)nLR(node *AVLTreeNode){
	// 如果节点为空就返回
	if node == nil{
		return
	}
	fmt.Printf("%d->", node.info)
	// 遍历左子树
	a.nLR(node.left)
	// 遍历右子树
	a.nLR(node.right)
}

// 中序遍历
func (a *AVLTree)lNR(node *AVLTreeNode){
	// 如果节点为空就返回
	if node == nil{
		return
	}
	a.lNR(node.left)
	fmt.Printf("%d->", node.info)
	a.lNR(node.right)
}

// 后序遍历
func (a *AVLTree)lRN(node *AVLTreeNode){
	if node == nil{
		return
	}
	a.lRN(node.left)
	a.lRN(node.right)
	fmt.Printf("%d->", node.info)
}

// 前序显示
func (a *AVLTree)ShowNLR(){
	if a.root == nil{
		fmt.Println("zhe tree is empty")
		return
	}
	fmt.Print("NLR:")
	node := a.root
	a.nLR(node)
	fmt.Println()
}

// 中序显示
func (a *AVLTree)ShowLNR(){
	if a.root == nil{
		fmt.Println("zhe tree is empty")
		return
	}
	fmt.Print("LNR:")
	node := a.root
	a.lNR(node)
	fmt.Println()
}

// 后序显示
func (a *AVLTree)ShowLRN(){
	if a.root == nil{
		fmt.Println("zhe tree is empty")
		return
	}
	fmt.Print("LRN:")
	node := a.root
	a.lRN(node)
	fmt.Println()
}

```

#### 树的旋转

* 单左旋 单右旋 左右旋 右左旋
```
// 单右旋
func (a *AVLTree)lR(node *AVLTreeNode) *AVLTreeNode {
	// 临时node 指向节点的左节点
	temp := (*node).left
	// node的左节点指向temp 的右节点
	(*node).left = temp.right
	// temp 的右节点指向node（node 为temp的右节点的）
	temp.right = node
	// 调整高度
	(*node).height = maxValue(getHeight((*node).left), getHeight((*node).right)) + 1
	temp.height = maxValue(getHeight(temp.left), getHeight(temp.right)) + 1
	return  temp
}

// 单左旋
func (a *AVLTree)rR(node *AVLTreeNode) *AVLTreeNode {
	// 临时node 指向节点的右节点
	temp := (*node).right
	// node的右节点指向temp的左节点
	(*node).right = temp.left
	// temp的左节点指向node
	temp.left = node
	// 调整树的高度
	(*node).height = maxValue(getHeight((*node).left), getHeight((*node).right)) + 1
	temp.height = maxValue(getHeight(temp.left), getHeight(temp.right)) + 1
	return temp
}

// 左右旋(先左旋在右旋)
func (a *AVLTree)rLR(node *AVLTreeNode) *AVLTreeNode {
	node.left =  a.rR(node.left)
	return a.lR(node)
}

// 右左旋(先右旋再左旋)
func (a *AVLTree)lRR(node *AVLTreeNode) *AVLTreeNode {
	node.right = a.lR(node.right)
	return a.rR(node)
}

```
### 测试
```

func TestAVRTree_InsertData(t *testing.T) {
	avl := CreatAVLTree()
	avl.InsertData(2)
	avl.InsertData(12)
	avl.InsertData(24)
	avl.InsertData(1)
	avl.InsertData(65)
	avl.InsertData(72)
	avl.InsertData(32)
	avl.InsertData(21)
	avl.ShowNLR()
	avl.ShowLNR()
	avl.ShowLRN()
}

func TestAVLTree_Remove(t *testing.T) {
	avl := CreatAVLTree()
	avl.InsertData(1)
	avl.InsertData(2)
	avl.InsertData(3)
	avl.InsertData(4)
	avl.InsertData(5)
	avl.InsertData(6)
	avl.InsertData(7)
	avl.InsertData(8)
	avl.ShowNLR()
	avl.ShowLNR()
	avl.ShowLRN()
	fmt.Println("**********************")
	avl.Remove(4)
	avl.ShowNLR()
	avl.ShowLNR()
	avl.ShowLRN()
}

func TestAVLTree_IsValue(t *testing.T) {
	avl := CreatAVLTree()
	avl.InsertData(1)
	avl.InsertData(2)
	avl.InsertData(3)
	avl.InsertData(4)
	avl.InsertData(5)
	avl.InsertData(6)
	avl.InsertData(7)
	avl.InsertData(8)
	avl.ShowNLR()
	avl.ShowLNR()
	avl.ShowLRN()
	data := 4
	is := avl.IsValue(data)
	if is{
		fmt.Printf("%d :is exits\n", data)
	}else{
		fmt.Printf("%d :is not exits\n", data)
	}
}

```
## 源码
* [我的github:https://github.com/zcj-git520/DataStructuresAlgorithmsForGo/tree/master/tree/AVL_tree](https://github.com/zcj-git520/DataStructuresAlgorithmsForGo/tree/master/tree/AVL_tree)


