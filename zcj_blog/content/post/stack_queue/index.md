---
title: "数据结构-栈(基于单项链表)&&队列(基于双向链表)go实现"
date: 2021-12-31T22:00:08+08:00
draft: false
image: 1.jpg
categories:
    - language
    - golang
---
## 栈
基于单向链表实现
### 数据结构定义
```
type stackData struct {
	list *Singly_linked_list.LinkedList
}

// 获得栈的长度
func (s *stackData)Len() int{
	return s.list.Len()
}
```

###  数据操作
```
// 将数据插入栈顶
func (s *stackData)Push(data interface{}) {
	s.list.AddToHead(data)
}

// 将数据从栈顶取出，并删除数据
func (s *stackData)Pop()interface{}{
	data := s.list.QuireIndex(0)
	// 栈不为空, 删除栈顶数据
	if data != nil{
		s.list.DeleteToHead()
	}
	return data
}

// 将数据取出，不删除数据
func (s *stackData)GetTopValue()interface{}{
	return s.list.QuireIndex(0)
}

// 展示栈
func (s *stackData)ShowStack() {
	s.list.QuireAll()
}
```
### 创建栈
```
func NewStackData()*stackData{
	return  &stackData{list:Singly_linked_list.NewLinkedList()}
}
```
## 队列
* 基于双向链表实现
### 数据结构定义
```
type queueData struct {
	list *double_linked_list.DoubleList
}

// 获得队列的大小
func (q *queueData)Len() int{
	return q.list.Len()
}
```
### 数据的操作
```
// 将数据队尾
func (q *queueData)EnQueue(data interface{}) {
	q.list.AddToTail(data)
}

// 将数据从队首取出，并删除数据
func (q *queueData)DeQueue()interface{}{
	data := q.list.QuireIndex(0)
	// 栈不为空, 删除栈顶数据
	if data != nil{
		q.list.DeleteToHead()
	}
	return data
}

// 将数据取出，不删除数据
func (q *queueData)GetTopQueueValue()interface{}{
	return q.list.QuireIndex(0)
}

// 展示队列
func (q *queueData)ShowQueue() {
	q.list.QuireAll()
}
```
### 创建队列
```
func NewQueueData()*queueData{
	return  &queueData{list:double_linked_list.NewDoubleLinkedList()}
}
```
## 源码
* [我的github:https:https://github.com/zcj-git520/DataStructuresAlgorithmsForGo/tree/master/stack_queue/queue](https://github.com/zcj-git520/DataStructuresAlgorithmsForGo/tree/master/stack_queue/queue)
* [我的github:https:https://github.com/zcj-git520/DataStructuresAlgorithmsForGo/tree/master/stack_queue/stack](https://github.com/zcj-git520/DataStructuresAlgorithmsForGo/tree/master/stack_queue/stack)

