---
title: "数据结构-双向链表&&循环双向链表go语言实现"
date: 2021-12-30T22:00:08+08:00
draft: false
image: 1.jpg
categories:
    - language
    - golang
---
## 双向链表

### 数据结构的定义
```
// 双向链表的节点定义
type doubleLinkedNode struct {
	info  interface{}       // 存储的数据
	prev *doubleLinkedNode  // 指向先驱节点
	next *doubleLinkedNode  // 指向后驱几点
}

// 双向链表定义
type DoubleList struct {
	Head *doubleLinkedNode   // 头节点
	Tail *doubleLinkedNode   // 尾节点
	len  int                 // 链表长度
}
```

### 创建节点
```
func createNode(data interface{}) *doubleLinkedNode {
	return &doubleLinkedNode{
		info: data,
		prev: nil,
		next: nil,
	}
}
```

### 链表的长度
```
// 链表的长度
func(d *DoubleList)Len() int{
	return d.len
}
```
### 数据的插入
#### 单个节点的插入
```
// 链表的尾插法
func (d *DoubleList)AddToTail(info interface{}) *DoubleList {
	newNode := createNode(info)  // 创建新节点
	// 链表为空, 头尾节点都指向该节点
	if d.Head == nil{
		d.Head = newNode
		d.Tail = newNode
	}else{
		d.Tail.next = newNode
		newNode.prev = d.Tail
		d.Tail = d.Tail.next
	}
	// 链表长度+1
	d.len++
	return d
}

// 链表的头插法
func (d *DoubleList)AddToHead(info interface{}) *DoubleList  {
	newNode := createNode(info)  // 创建节点
	// 链表为空
	if d.Head == nil{
		d.Head = newNode
		d.Tail = newNode
	}else{
		newNode.next = d.Head    // 新节点的后驱指向头节点
		d.Head.prev = newNode    // 头节点的前驱指向新节点
		d.Head = newNode         // 将新节点设置尾头节点
	}
	// 链表长度+1
	d.len++
	return  d
}

// 链表的index
// 通过index=>插入链表新的节点
// index为正数 为从左->右 // 0表示第一个节点
// index为负数 为从右->左 // -1表示第一个节点
func (d *DoubleList)AddToIndex(index int, info interface{}) *DoubleList {
	// 当链表为空时，采用了尾插入法插入数据
	if d.Head == nil{
		return d.AddToTail(info)
	}
	// 当index大于链表的值时，默认将数据插入到链表的后面
	if int(math.Abs(float64(index))) > d.len-1{
		return d.AddToTail(info)
	}
	// 插入链表的头部
	if index == 0{
		return d.AddToHead(info)
	}
	// 插入到链表的末尾
	if index == -1{
		return d.AddToTail(info)
	}
	// 当index 为负数时，从右到左插入
	if index < 0{
		index += d.len +1
	}
	__index := 1  // 内部的index值
	// 从第二个值开始插入
	for node := d.Head; node != d.Tail; node = node.next{
		if index == __index{
			newNode := createNode(info)
			node.next.prev = newNode   // 节点的后驱节点的先驱指向新节点
			newNode.prev = node        // 新节点的前驱指向节点
			newNode.next = node.next   // 新节点的后驱指向节点的后驱
			node.next = newNode           // 节点的后驱指向新节点
			// 链表的节点数加1
			d.len ++
			return d
		}
		__index ++
	}
	return d
}

```

#### 链表的合并
```
// 将新的链表插入头部
func (d *DoubleList)AddListToHead(list *DoubleList)*DoubleList{
	// 两个链表都为空时
	if d.Head == nil && list.Head == nil{
		return d
	}
	// 合并表为空
	if d.Head != nil && list.Head == nil{
		return d
	}
	// 主表为空
	if d.Head == nil && list.Head != nil{
		return list
	}
	// 合并表的尾指针指向主表的头
	list.Tail.next = d.Head
	d.Head.prev = list.Tail
	list.Tail = d.Tail
	// 表节点数合并
	list.len += d.len
	return list
}

// 将新的链表插入到尾部
func (d *DoubleList)AddListToTail(list *DoubleList)*DoubleList{
	// 两个链表都为空时
	if d.Head == nil && list.Head == nil{
		return d
	}
	// 合并表为空
	if d.Head != nil && list.Head == nil{
		return d
	}
	// 主表为空
	if d.Head == nil && list.Head != nil{
		return list
	}
	// 将新表插入到主表之后
	d.Tail.next = list.Head
	list.Head.prev = d.Tail
	d.Tail = list.Tail
	d.len += list.len
	return d
}

// 经新的表插入到index
func (d *DoubleList)AddListToIndex(index int, list *DoubleList) *DoubleList {
	// 两个链表都为空时
	if d.Head == nil && list.Head == nil{
		return d
	}
	// 合并表为空
	if d.Head != nil && list.Head == nil{
		return d
	}
	// 主表为空
	if d.Head == nil && list.Head != nil{
		return list
	}
	if int(math.Abs(float64(index))) > d.len{
		fmt.Println("错误的index")
		return d
	}
	if index == 0{
		return d.AddListToHead(list)
	}
	if  index == -1{
		return d.AddListToTail(list)
	}
	if index < 0{
		index += d.len
	}
	__index := 1
	for node := d.Head; node != d.Tail; node = node.next{
		if index == __index{
			node.next.prev = list.Tail
			list.Tail.next = node.next
			node.next = list.Head
			list.Head.prev = node
			d.len += list.len  // 链表的数值相加
			return d
		}
		__index ++
	}
	return d
}

```
### 数据的删除
```
// 链表头删除法
func (d *DoubleList)DeleteToHead()*DoubleList{
	// 链表为空
	if d.Head == nil{
		return d
	}
	// 链表中只有一个数
	if d.Head == d.Tail{
		d.Head = nil
		d.Tail = nil
		d.len  = 0
		return d
	}
	d.Head = d.Head.next
	d.Head.prev = nil
	// node数减一
	d.len --
	return d
}

// 链表尾删除法
func (d *DoubleList)DeleteToTail()*DoubleList{
	// 链表为空
	if d.Tail == nil{
		return d
	}
	// 链表中只有一个数
	if d.Head == d.Tail{
		d.Head = nil
		d.Tail = nil
		d.len = 0
		return d
	}
	d.Tail = d.Tail.prev
	d.Tail.next = nil
	// node数减一
	d.len --
	return d
}

// 通过值=>删除链表的节点(第一个)
func (d *DoubleList)DeleteToAValue(value interface{})*DoubleList{
	// 链表为空
	if d.Head == nil{
		fmt.Println("链表为空")
		return d
	}
	// value == head.info
	// 就采用头删法
	if value == d.Head.info{
		return  d.DeleteToHead()
	}
	if value == d.Tail.info{
		return  d.DeleteToTail()
	}
	// 中间采用轮询查找value, 从第二个开始轮询到倒数第二个结束
	for node := d.Head.next; node != d.Tail; node = node.next{
		if node.info == value{
			//  删除node
			node.next.prev = node.prev
			node.prev.next = node.next
			d.len --
			return d
		}
	}
	fmt.Println("链表中：值不存在")
	return d
}

// 通过值=>删除链表的节点(所有)
func (d *DoubleList)DeleteToValue(value interface{})*DoubleList{
	// 链表为空
	if d.Head == nil{
		fmt.Println("链表为空")
		return d
	}
	node := d.Head         // 当前的node
	for {
		// 当下一个节点是尾节点时，就判断首位和末尾是为需要删除的node
		if node == d.Tail {
			if d.Head.info == value{
				d.DeleteToHead()
			}
			if d.Tail.info == value{
				d.DeleteToTail()
			}
			return d
		}
		if node.info == value{
			// 删除node， 非头节点
			if node.prev != nil{
				node.next.prev = node.prev
				node.prev.next = node.next
				d.len --
			}else{
				// 头节点
				d.DeleteToHead()
			}
		}
		node = node.next

	}
}

// 通过index=>删除链表的节点
// index为正数 为从左->右 // 0表示第一个节点
// index为负数 为从右->左 // -1表示第一个节点
func (d *DoubleList)DeleteToIndex(index int)*DoubleList{
	// 链表为空
	if d.Head == nil{
		fmt.Println("链表为空")
		return d
	}
	// index 超过链表数
	if int(math.Abs(float64(index))) > d.len-1{
		fmt.Println("错误的index")
		return d
	}
	// 删除第一个node
	if index == 0{
		return d.DeleteToHead()
	}
	if index < 0{
		index += d.len
	}
	if index == d.len -1{
		return d.DeleteToTail()
	}
	_index := 1
	node := d.Head.next
	for {
		if index == _index{
			node.next.prev = node.prev
			node.prev.next = node.next
			d.len --
			return d
		}
		node = node.next
		_index ++
	}
}

```
### 数据的查询
```
// 遍历链表
func (d *DoubleList) QuireAll() {
	if d.Head == nil{
		fmt.Println("链表数据为空")
		return
	}
	node := d.Head
	for {
		if node == nil{
			return
		}
		fmt.Println(node.info)
		if node == d.Tail{
			return
		}
		node = node.next
	}
}

// 判断valve是否存在
func (d *DoubleList)QuireValue(value interface{}) bool{
	// 表为空
	if d.Head == nil{
		return false
	}
	// 表中只存在一个值
	if d.Head == d.Tail{
		if d.Head.info == value{
			return true
		}
		return false
	}
	// 遍历查值
	for node := d.Head; node != d.Tail.next; node = node.next{
		if node.info == value{
			return true
		}
	}
	return false
}

// 根据索引返回值
func (d *DoubleList)QuireIndex(index int) interface{} {
	// 链表为空
	if d.Head == nil{
		return nil
	}
	if int(math.Abs(float64(index))) > d.len -1 {
		fmt.Println("索引值错误")
		return nil
	}
	if index == 0{
		return d.Head.info
	}
	if index == d.len -1 || index == -1{
		return d.Tail.info
	}
	if index < 0{
		index += d.len
	}
	__index := 1
	for node := d.Head.next; node != d.Tail; node = node.next{
		if __index == index{
			return node.info
		}
		__index ++
	}
	//fmt.Println("未能找到！！！！")
	return nil
}

```

### 结构的入口
```
func NewDoubleLinkedList()*DoubleList{
	// 创建的链表头尾节点都为空
	return &DoubleList{
		Head: nil,
		Tail: nil,
		len: 0,
	}
}
```

## 循环双向链表

### 数据结构的定义
```
// 节点的定义
type circleDoubleNode struct {
	info  interface{}  // 数据结构的定义
	prev  *circleDoubleNode  // 前驱指针
	next  *circleDoubleNode  // 后驱指针
}

// 双向循环链表的定义
type circleDoubleList struct {
	currentNode *circleDoubleNode  // 指向链表的指针
	len  int    // 链表的长度
}
```
### 创建节点
```
func createNode(data interface{}) *circleDoubleNode{
	return &circleDoubleNode{
		info: data,
		prev: nil,
		next: nil,
	}
}
// 得到链表的长度
func (c *circleDoubleList)GetLen()int{
	return c.len
}
```
### 节点的插入
```
func (c *circleDoubleList)InsertNode(data interface{}) {
	newNode := createNode(data)
	// 如果链表为空
	if c.currentNode == nil{
		c.currentNode = newNode  // 新创建的节点为当前节点
		// 节点的前驱与后驱都指向自己
		c.currentNode.next = c.currentNode
		c.currentNode.prev = c.currentNode
	}else{
		// 节点的插入
		c.currentNode.next.prev = newNode
		newNode.prev = c.currentNode
		newNode.next = c.currentNode.next
		c.currentNode.next = newNode
		c.currentNode = newNode
	}
	// 链表长度 +1
	c.len ++
}
```

### 当前节点的删除
```
func (c *circleDoubleList)DeleteNode() bool{
	// 链表为空
	if c.currentNode == nil{
		return false
	}
	// 当链表值存在一个元素时
	if c.len == 1{
		c.currentNode = nil
		c.len --
		return true
	}
	// 当链表只存在两个元素时
	if c.len == 2{
		c.currentNode = c.currentNode.next
		c.len --
		return true
	}
	// 当前节点的后驱节点的前驱指针指向当前节点的的前驱节点
	c.currentNode.next.prev = c.currentNode.prev
	// 当前节点的前驱节点的后驱指针指向当前节点的后驱节点
	c.currentNode.prev.next = c.currentNode.next
	// 设置当前节点的后驱节点为当前节点
	c.currentNode = c.currentNode.next
	c.len --
	return true
}
```
### 遍历说有节点
```
func (c *circleDoubleList)QuireAll(){
	if c.currentNode == nil{
		fmt.Println("circle double linked list is empty")
		return
	}
	//__index := 1
	//for node := c.currentNode.next; node != c.currentNode; node = node.next{
	//	fmt.Printf("index is %d, node value is %v", __index, node.info)
	//	__index ++
	//}
	fmt.Printf("circle double linked list len is %d \n", c.len)
	node := c.currentNode.next
	for i := 1; i < c.len; i++{
		fmt.Printf("index is %d, node value is %v \n", i, node.info)
		node = node.next
	}
	fmt.Printf("index is %d, node value is %v \n", c.len, c.currentNode.info)
}

// 判断值是否存在
func (c *circleDoubleList)QuireValue(data interface{}) bool{
	if c.currentNode == nil{
		return false
	}
	node := c.currentNode
	for i := 1; i <= c.len; i++{
		if node.info == data{
			return true
		}
		node = node.next
	}
	return  false
}
```
### 链表实例化
```
func NewCircleDoubleList()*circleDoubleList{
	return &circleDoubleList{
		currentNode: nil,
		len:         0,
	}
}
```
## 源码
* [我的github:https://github.com/zcj-git520/DataStructuresAlgorithmsForGo/tree/master/Linked_list/double_linked_list](https://github.com/zcj-git520/DataStructuresAlgorithmsForGo/tree/master/Linked_list/double_linked_list)
* [我的github:https://github.com/zcj-git520/DataStructuresAlgorithmsForGo/tree/master/Linked_list/circle_double_linked_list](https://github.com/zcj-git520/DataStructuresAlgorithmsForGo/tree/master/Linked_list/circle_double_linked_list)

