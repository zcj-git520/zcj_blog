---
title: "数据结构-双向链表&&循环双向链表 c++实现"
date: 2021-12-30T21:00:08+08:00
draft: false
image: 1.jpg
categories:
    - language
    - C_C++
---
## 双向链表

### 数据结构的定义
```
template<class T>
class TwoWayLinkedNode
{
public:
    TwoWayLinkedNode(/* args */){}; 
    TwoWayLinkedNode(T data)
    {
        this->data = data;
        this->prev = 0;
        this->next = 0;
    }
    ~TwoWayLinkedNode(){};
    T data;       // 数据
    TwoWayLinkedNode *next;  // 指向后继节点
    TwoWayLinkedNode *prev;   // 指向前驱
};

template<class T>
class TwoWayList
{
private:
    /* data */
    TwoWayLinkedNode<T> *head;   // 头节点
    TwoWayLinkedNode<T> *tail;   // 尾节点
    int  len;                    // 链表长度
public:
    TwoWayList()
    {
        head = 0;
        tail = 0;
        len = 0;
    };
    ~TwoWayList();
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
    // 清空链表
    void clear();
    // 链表是否为空
    bool isEmpty()
    {
        return head == 0;
    }
    // 插入到链表的头部
    void inseartToHead(T data);  
    // 插入到链表的尾部
    void inseartTotail(T data); 
    // 插入到链表的index
    void inseartToindex(int index, T data); 
    // 删除链表的头部元素
    void deleteToHead();
    // 删除链表的尾部元素
    void deleteToTail();
    // 删除链表的index元素
    void deleteToIndex(int index);
    // 查询链表的所有值
    void queryAll();
    // 返回index的值
    bool queryIndex(int index, T *data);
    // 判断value是否存在
    bool queryValue(T data);
};
```
###  数据的操作
```

template<class T>
void TwoWayList<T> ::inseartToHead(T data)
{
   TwoWayLinkedNode<T> *new_Node = new TwoWayLinkedNode<T>(data); 
   // 链表为空
   if (isEmpty())
   {
      head = tail = new_Node;
   }
   else
   {
      new_Node->next = head;   // 新节点的后驱指向头节点
      head->prev = new_Node;   // 头节点的前驱指向新节点
      head = new_Node;         // 将新节点设置新的头节点
   }
   setlen(addOne);
}

template<class T>
void TwoWayList<T> ::inseartTotail(T data)
{
   TwoWayLinkedNode<T> *new_Node = new TwoWayLinkedNode<T>(data);
   if(isEmpty())
   {
      head = tail = new_Node;
   }
   else
   {
      tail->next = new_Node;  // 尾节点的next指向新节点
      new_Node->prev = tail;  // 新节点的前驱指向尾节点
      tail = new_Node;        // 将新节点设置为新的尾节点
   }
   setlen(addOne);
}

template<class T>
void TwoWayList<T> ::inseartToindex(int index, T data)
{
   int len = getlen();
   if(index == 0)
   {
      inseartToHead(data);
      return;
   }
   // 链表为空，当index 小于0 或者大于等于链表数，采用尾插法
   if (isEmpty() || index < 0 || index >= len)
   {
      inseartTotail(data);
      return;
   }
   TwoWayLinkedNode<T> *node = head;
   int __index = 1;
   while (__index != index)
   {
      node = node->next;
      __index ++;
   }
   TwoWayLinkedNode<T> *new_node = new TwoWayLinkedNode<T>(data);
   node->next->prev = new_node;   // node节点的后继节点的先驱要指向新节点
   new_node->prev = node;         // 新节点的先驱要指向node
   new_node->next = node->next;   // 新节点的后继要指向node的后继
   node->next = new_node;         // node的后继要指向新节点
   setlen(addOne);
}

template<class T>
void TwoWayList<T> ::clear()
{
   while (!isEmpty())
   {
      TwoWayLinkedNode<T> *node = head->next;
      delete head;
      head = node;
   }
   
}

template<class T>
void TwoWayList<T> ::deleteToHead()
{
   // 链表为空就返回
   if(isEmpty())
   {
      return;
   }
   // 链表只存在一个节点
   if(head == tail)
   {
      delete head;
      head = tail = 0;
   }
   else{
       head = head->next;                // 将下一节点设置头节点
      delete head->prev;                // 释放新的头节点的前驱节点
      head->prev = 0;                   // 头节点的先驱设置nil
   }
   setlen(ReductOne);
}

template<class T>
void TwoWayList<T> ::deleteToTail()
{
    // 链表为空就返回
   if(isEmpty())
   {
      return;
   }
   // 链表只存在一个节点
   if(head == tail)
   {
      delete head;
      head = tail = 0;
   }
   else{
      tail = tail->prev;                // 尾节点的先驱节点重新设置为新的尾节点
      delete tail->next;                // 释放新的尾节点的后驱
      tail->next = 0;                   // 新的尾节点的后驱指向尾nil
   }
   setlen(ReductOne);
   
}

template<class T>
void TwoWayList<T> ::deleteToIndex(int index)
{
   len = getlen();
   // 链表为空,就返回
   if(isEmpty())
   {
      return;
   }
   // index >= len || index < 0 采用删除末尾值
   if (index >= len || index < 0)
   {
      deleteToTail();
      return;
   }
   if(index == 0)
   {
      deleteToHead();
      return;
   }
   if (index == len - 1)
   {
      deleteToTail();
      return;
   }
   int __index = 1;
   TwoWayLinkedNode<T> *node = head->next;
   while (__index != index)
   {
      node = node->next;
      __index ++;
   }
   node->prev->next = node->next;  // node节点的先驱节点的后驱指向node的后驱
   node->next->prev = node->prev;  // node节点的后驱节点的先驱指向node的先驱
   delete node;  // 释放node
   setlen(ReductOne);
}

template<class T>
void TwoWayList<T> ::queryAll()
{
   if (isEmpty())
   {
        cout << "The list length is empty" << endl;
        return;
    }

   cout << "The length of the two way list is zero:" << getlen() << endl;
   int __index = 1;
   TwoWayLinkedNode<T> *node = head;
   while (node != tail->next)
   {
      cout << "node num is:" << __index << "  data:" << node->data << endl;
      node = node->next;
      __index++; 
   }
   
}

template<class T>
bool TwoWayList<T> ::queryIndex(int index, T *data)
{
   len = getlen();
   // 链表为空,index >= len || index < 0  就返回
   if(isEmpty() || index >= len || index < 0)
   {
      return false;
   }
   int __index = 0;
   for(TwoWayLinkedNode<T> *node = head; node != tail->next; node = node->next)
   {
      if (__index == index)
      {
         *data = node->data;
         return true;
      }
      __index++;

   }
   return false;

}

template<class T>
bool TwoWayList<T> ::queryValue(T data)
{
    if(isEmpty())
   {
      return false;
   }
   for(TwoWayLinkedNode<T> *node = head; node != tail->next; node = node->next)
   {
      if (node->data == data)
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
   TwoWayList<int> newList;
   newList.inseartToHead(3);
   newList.inseartToHead(2);
   newList.inseartToHead(1);
   newList.inseartToHead(-11);
   newList.inseartTotail(4);
   newList.inseartTotail(5);
   newList.inseartTotail(6);
   newList.inseartTotail(7);
   newList.inseartTotail(8);
   newList.queryAll();
   cout << "****************************************************"<< endl;
   newList.inseartToindex(2, 11);
   newList.inseartToindex(0, 2);
   newList.inseartToindex(1, 3);
   newList.queryAll();
   cout << "****************************************************"<< endl;
   newList.deleteToHead();
   newList.deleteToHead();
   newList.deleteToHead();
   newList.queryAll();
   // newList.clear();
   cout << "****************************************************"<< endl;
   newList.deleteToTail();
   newList.deleteToTail();
   newList.deleteToTail();
   newList.deleteToTail();
   newList.queryAll();
   cout << "****************************************************"<< endl;
   newList.deleteToIndex(4);
   newList.queryAll();
   cout << "****************************************************"<< endl;
   int index = 1;
   int info;
   if(newList.queryIndex(index, &info))
   {
      cout << "exits index: " << index << " => value is " << info<<  endl;
   }
   else
   {
      cout <<" no exits index:" << index << endl;
   }
   cout << "****************************************************"<< endl;
   newList.queryAll();
   cout << newList.queryValue(1)<< endl;
   system("pause");
   return 0;
}

```

## 循环双向链表
### 数据结构的定义
```
// 循环双向链表节点
template<class T>
class CircularDoubleLinkedNode
{
public:
    CircularDoubleLinkedNode()
    {
        prev = next = 0;
    }
    CircularDoubleLinkedNode(T data)
    {
        info = data;
        prev = next = 0;
    }
    ~CircularDoubleLinkedNode(){};
    T info;     // 数据
    CircularDoubleLinkedNode *prev;   // 先驱节点
    CircularDoubleLinkedNode *next;   // 后继节点
};

template<class T>
class CircularDoubleList
{
private:
    CircularDoubleLinkedNode<T> *current_node;  // 指向当前节点
    int len;    // 链表的长度
public:
    CircularDoubleList()
    {
        current_node = 0;
        len = 0;
    }
    ~CircularDoubleList();
    void addLen()
    {
        len++;
    }
    void reduceLen()
    {
        len--;
    }
    int getLen()
    {
        return len;
    }
    bool isEmpty()
    {
        return current_node == 0;
    }
    void inseartNode(T data);
    bool deleteNode();
    void queryAll();
    // 判断value是否存在
    bool queryValue(T data);
};
```
### 数据的操作
```

template<class T>
void CircularDoubleList<T> ::inseartNode(T data)
{
    CircularDoubleLinkedNode<T> *new_node = new CircularDoubleLinkedNode<T>(data);
    // 如果链表为空
    if(isEmpty())
    {
        // 当前节点为新的节点
        current_node = new_node;
        current_node->next = current_node; // 后驱指向自己
        current_node->prev = current_node; // 前驱也指向自己
    }
    else
    {
        /*与双向链表中的插入到链表中是一样的*/
       current_node->next->prev = new_node;
       new_node->prev = current_node;
       new_node->next = current_node->next;
       current_node->next = new_node;
       current_node = new_node;
    }
    addLen();
}

template<class T>
bool CircularDoubleList<T> ::deleteNode()
{
    // 链表为空
    if(isEmpty())
    {
        return false;
    }
    int len = getLen();
    // 链表中只有一个元素
    if(len == 1)
    {
        delete current_node;
        current_node = 0;
        reduceLen();
        return true;
    }
    // 链表中只有两个元素
    if (len == 2)
    {
        CircularDoubleLinkedNode<T> *node = current_node;
        current_node = current_node->next;
        current_node->next = current_node;
        current_node->prev = current_node;
        delete node;
        reduceLen();
        return true;
    }
    // 链表元素达到3个及3个以上
    CircularDoubleLinkedNode<T> *node = current_node;
    current_node->prev->next = current_node->next;
    current_node->next->prev = current_node->prev;
    current_node = current_node->next;
    delete node;
    reduceLen();
    return true;
}

template<class T>
void CircularDoubleList<T> ::queryAll()
{
    if(isEmpty())
    {
        cout << "circular double list is empty" <<  endl;
        return;
    }

    cout << "circular double list len is:" << getLen() << endl;
    int __index = 1;
    for (CircularDoubleLinkedNode<T> *node = current_node->next; node != current_node; node = node->next)
    {
        cout << "index:" << __index << "  data is: " << node->info << endl;
        __index++;
    }
    cout << "index:" << __index << "  data is: " << current_node->info << endl;
}

template<class T>
bool CircularDoubleList<T> ::queryValue(T data)
{
    // 判断链表为空
    if(isEmpty())
    {
        return false;
    }
    // 当前节点的值== data
    if(current_node->info == data)
    {
        return true;
    }
    // 循环遍历当节点的value== data then retuen ture
    for(CircularDoubleLinkedNode<T> *node = current_node->next; node != current_node; node = node->next)
    {
        if(node->info == data)
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
    CircularDoubleList<int> cirList;
    cirList.inseartNode(0);
    cirList.inseartNode(1);
    cirList.inseartNode(2);
    cirList.inseartNode(3);
    cirList.queryAll();
    cout << "*****************************************************" << endl;
    cirList.deleteNode();
    cirList.deleteNode();
    // cirList.deleteNode();
    // cirList.deleteNode();
    cirList.queryAll();
    cout << "******************************************************" <<endl;
    int data = 11;
    if(cirList.queryValue(data))
    {
        cout << data << "  is exists" << endl;
    }
    else
    {
        cout << data << "  is not exists" << endl;
    }
   system("pause");
   return 0;
}

```
## 源码
* [我的github:https://github.com/zcj-git520/DataStructuresAlgorithmsForC/tree/master/linked_list/two_way_linked_list](https://github.com/zcj-git520/DataStructuresAlgorithmsForC/tree/master/linked_list/two_way_linked_list)
* [我的github:https://github.com/zcj-git520/DataStructuresAlgorithmsForC/tree/master/linked_list/circular_double_linked_list](https://github.com/zcj-git520/DataStructuresAlgorithmsForC/tree/master/linked_list/circular_double_linked_list)

