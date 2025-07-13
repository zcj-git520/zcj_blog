---
title: "数据结构-单向链表c++语言实现"
date: 2021-12-27T22:00:08+08:00
draft: false
image: 1.jpg
categories:
    - language
    - C_C++
---
## 单向链表

### 数据结构的定义

####  数据节点的类的定义
```
class SinglyLinkedNode
{
public:
   SinglyLinkedNode(){
       next = 0;
   };
    // 创建新的node
   SinglyLinkedNode(int data)
   {
       this->data = data;
       next = 0;
   };
   ~SinglyLinkedNode(){};
   int data;             // 链表的数据
   SinglyLinkedNode *next;  // 指向下一个节点
};
```
#### 链表的数据结构(类的定义)
```
class SinglyList
{
private:
    /* data */
    SinglyLinkedNode *head;   // 头节点
    SinglyLinkedNode *tail;   // 尾节点
    int len;                  // 链表长度

public:
    SinglyList(){
        head = 0;
        tail = 0;
        len = 0;
    };
    ~SinglyList();
    // 链表的长度+1
    void setlen(int len)
    {
        this->len += len;
    }; 
    // 返回链表的长度
    int getlen()
    {
        return len;
    };
    // 链表是否为空
    bool ismpty()
    {
        return head == 0;
    }
    // 插入到链表的头部
    void inseartToHead(int data);  
    // 插入到链表的尾部
    void inseartTotail(int data); 
    // 插入到链表的index
    void inseartToindex(int index, int data); 
    // 删除链表的头部元素
    void deleteToHead();
    // 删除链表的尾部元素
    void deleteToTail();
    // 删除链表的index元素
    void deleteToIndex(int index);
    // 查询链表的所有值
    void queryAll();
    // 返回index的值
    int queryIndex(int index);
    // 判断value是否存在
    bool queryValue(int data);
};
```


### 数据的插入

#### 插入到链表的尾部
```
void SinglyList::inseartTotail(int data)
{
    // 定义node
    SinglyLinkedNode *new_node;
    new_node = new SinglyLinkedNode(data);
    // 链表为空
    if (ismpty())
    {
        head = tail = new_node; // 头尾节点都指向新创节点
        setlen(addOne);              // 节点数+1
        return;
    }
    tail->next = new_node;
    tail = tail->next;
    setlen(addOne);
}
```
#### 插入到链表的头部
```
void SinglyList::inseartToHead(int data)
{
    SinglyLinkedNode *node;
    node = new SinglyLinkedNode(data);
    if (ismpty()){
        head = tail = node;
        setlen(addOne);
        return ;
    }
    node->next = head;
    head = node;
    setlen(addOne);
}
```
#### 插入到链表的index
```
void SinglyList::inseartToindex(int index, int data)
{
    int len;
    len = getlen()-1;
     // 如果链表为空, 或者插入到末尾,或者index <0 采用尾插法添加到链表中
    if (ismpty() || index ==len || index < 0)
    {
        inseartTotail(data);
        return;
    }
    if(index == 0){
        inseartToHead(data);
        return;
    }
    SinglyLinkedNode *node;
    node = new SinglyLinkedNode(data);
    int __index = 1;
    for (node = head; node != tail; node = node->next)
    {
        if (__index == index)
        {
            SinglyLinkedNode *new_node;
            new_node = new SinglyLinkedNode(data);
            new_node ->next = node->next;
            node->next = new_node;
            setlen(addOne);
            return;
        }
        __index ++;
    }
}

```

### 数据的删除
#### 删除链表的头部元素
```
void SinglyList::deleteToHead()
{
    // 链表为空
    if(ismpty())
    {
        return;
    }
    SinglyLinkedNode *tmp = head;
    // 链表只存在一个值
    if(head == tail)
    {
        head = tail = 0;
    }
    else
    {
        head = head->next;
    }
    delete tmp; // 释放资源
    setlen(ReductOne);
}
```
#### 删除链表的尾部元素
```
void SinglyList::deleteToTail()
{
    // 链表为空
    if(ismpty())
    {
        return;
    }
    // 链表只存在一个值
    if(head == tail)
    {
        delete head;
        setlen(ReductOne);
        head = tail = 0;
        return;
    }
    SinglyLinkedNode *tmp = head;
    while (tmp->next != tail)
    {
        tmp  = tmp->next;
    }
    delete tail;
    tail = tmp;
    tail->next = 0;
    setlen(ReductOne);
}
```
#### 删除链表的index元素
```
void SinglyList::deleteToIndex(int index)
{
    // 链表为空
    if(ismpty())
    {
        return;
    }
    // 链表只存在一个值
    if(head == tail)
    {
        delete head;
        setlen(ReductOne);
        head = tail = 0;
        return;
    }
    int len = getlen();
    if (index >= len-1 || index < 0)
    {
        deleteToTail();
        return;
    }
    if (index == 0)
    {
        deleteToHead();
        return;
    }

    SinglyLinkedNode *node = head;
    SinglyLinkedNode *tmp = head->next;
    int __index = 1;
    while (__index != index)
    {
        node = node->next;
        tmp = tmp->next;
        __index ++;
    }
    node->next = tmp->next;
    delete tmp;
    setlen(ReductOne);
}

```
### 数据的查询

#### 查询链表的所有值
```
void SinglyList::queryAll()
{
    if (ismpty()){
        cout << "The list length is empty" << endl;
        return;
    }
    int i = 1;
    cout << "The length of the list is zero:" << getlen() << endl;
    SinglyLinkedNode *node;
    for(node = head; node != tail->next; node = node->next){
        cout << "node num is:" << i << "  data:" << node->data << endl;
        i++;
    }
}
```
#### 返回index的值
```
int SinglyList::queryIndex(int index)
{
    int len = getlen();
    // 链表为空, 索引不存在
    if(ismpty() || index > len -1 || index < 0)
    {
        return -1;
    }
   if (index == 0)
   {
       return head ->data;
   }
   if (index == len -1)
   {
       return tail ->data;
   }
   int __index = 1;
   for (SinglyLinkedNode *node = head->next; node != tail; node = node->next)
   {
       if (index == __index)
       {
           return node->data;

       }
        __index ++; 
   }
   return -1;
}
```
#### 判断value是否存在
```
bool SinglyList::queryValue(int value)
{
    // 链表为空
    if(ismpty())
    {
        return false;
    }
    // 链表只存在一个值
    if(head == tail)
    {
        if (head->data == value){
            return true;
        }
        return false;
    }

    for(SinglyLinkedNode *node = head; node != tail->next; node = node->next)
    {
        if(node->data == value)
        {
            return true;
        }
    }
    return false;
}

```

### 测试
```
int main()
{
    SinglyList newList;
    newList.inseartTotail(1);
    newList.inseartTotail(2);
    newList.inseartTotail(3);
    newList.inseartToHead(0);
    newList.inseartToHead(-11);
    newList.inseartToHead(-12);
    newList.inseartToHead(-13);
    newList.inseartToindex(6,5);
    newList.inseartToindex(-7,51);
    newList.queryAll();
    cout << "************************************************"<< endl;
    newList.deleteToHead();
    newList.queryAll();
    cout << "************************************************"<< endl;
    newList.deleteToTail();
    newList.queryAll();
    cout << "************************************************"<< endl;
    newList.deleteToTail();
    newList.queryAll();
    cout << "************************************************"<< endl;
    newList.deleteToIndex(3);
    newList.queryAll();
    cout << "************************************************"<< endl;
    cout << newList.queryIndex(5) << endl;
     cout << "************************************************"<< endl;
    cout << newList.queryValue(21) << endl;
    system("pause");
    return 0;
}

```
## 源码
* [我的github:https://github.com/zcj-git520/DataStructuresAlgorithmsForC/tree/master/linked_list/singly_linked_list](https://github.com/zcj-git520/DataStructuresAlgorithmsForC/tree/master/linked_list/singly_linked_list)

