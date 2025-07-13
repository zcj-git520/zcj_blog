---
title: "数据结构-Trie tree go实现"
date: 2022-08-25T21:00:08+08:00
draft: false
image: 1.png
categories:
    - language
    - golang
---
## 前缀树、也称字典树（Trie） for go
- 前缀树是一种多叉树结构，经常用来做快速检索
- 前缀树（Trie树），即字典树，又称单词查找树或键树，
- 它是一种专门处理字符串匹配的数据结构。
- 前缀树可以最大限度地减少无谓的字符串比较（空间换时间），提高查询效率。
- 特点：
    - 根节点不包含任何key
    - 每个子节点包含的key都不相同
## 代码实现如下
```go
    package Trie
    
    import (
    	"fmt"
    )
    
    // 定义前缀树的节点 通过map
    type trieNode struct {
    	ChildNodes map[rune]*trieNode // 定义子节点
    	//CharCount  int                // 字符统计
    	EndNode    bool              // 是否为子节点
    	StrData    string            // 最后节点时，该数据
    }
    
    // 前缀树的定义
    type trieTree struct {
    	root      *trieNode    // 根节点
    	WordCount int          // 词数的数量
    }
    
    // 创建节点
    func creatNode() *trieNode{
    	node := make(map[rune]*trieNode)
    	return &trieNode{
    		ChildNodes: node,
    		//CharCount:  0,
    		EndNode:    false,
    		StrData:    "",
    	}
    }
    
    // 词数的数量
    func (t *trieTree)len() int{
    	return t.WordCount
    }
    
    // 添加节点
    func (t *trieTree)insertData(data string) error{
    	if len(data) == 0{
    		return fmt.Errorf("插入数据为空")
    	}
    	nodeNext := t.root
    	for _, cha := range data{
    		_, ok := nodeNext.ChildNodes[cha]
    		// 是否存在
    		if !ok{
    			// 创建节点
    			nodeNext.ChildNodes[cha] = creatNode()
    		}
    		nodeNext = nodeNext.ChildNodes[cha]
    		//nodeNext.CharCount++
    	}
    	// 判断是否为最后的节点
    	if !nodeNext.EndNode{
    		nodeNext.EndNode = true
    		nodeNext.StrData = data
    		t.WordCount ++
    	}
    	return nil
    }
    
    // 查询某个值是否存在
    func (t *trieTree)FindData(data string) bool{
    	if t.root == nil || data == ""{
    		return false
    	}
    	nodeNext := t.root
    	for _, cha := range data {
    		_, ok := nodeNext.ChildNodes[cha]
    		if !ok{
    			return false
    		}
    		nodeNext = nodeNext.ChildNodes[cha]
    	}
    	if nodeNext.StrData != data{
    		return false
    	}
    	return true
    }
    // 遍历树
    func(t *trieTree)traverseData() ([]string, error){
    	returnData := make([]string,0)
    	if t.root == nil{
    		return returnData, fmt.Errorf("数据为空")
    	}
    	nodeNext := t.root
    	for _, node := range nodeNext.ChildNodes{
    		t.getData(node, &returnData)
    	}
    	return returnData, nil
    }
    
    // 使用递归遍历
    func (t *trieTree)getData(nodeNext *trieNode, triData *[]string) {
    	if nodeNext.EndNode{
    		*triData = append(*triData, nodeNext.StrData)
    		if nodeNext.ChildNodes == nil{
    			return
    		}
    
    	}
    	for _, node := range nodeNext.ChildNodes{
    		 t.getData(node, triData)
    	}
    }
    
    // 清空树
    func (t *trieTree)Clear(){
    	t.root = nil
    	t.WordCount = 0
    }
    
    // 删除数据
    func (t *trieTree)DeleteData(data string) error{
    	if t.root == nil || data == ""{
    		return fmt.Errorf("数据为空")
    	}
    	ok := t.FindData(data)
    	if !ok{
    		return fmt.Errorf("%v：数据不存在", data)
    	}
    	nextNode := t.root
    	t.delete(nextNode, data)
    	return nil
    }
    
    func (t *trieTree)delete(node *trieNode, data string)  {
    	// 从头开始遍历
    	for _, cha := range data{
    		node = node.ChildNodes[cha]
    		// 如果节点为的cha的个数为1时，直接删除节点
    		if node.ChildNodes == nil {
    			delete(node.ChildNodes, cha)
    			return
    		}
    		// 还存在后续节点时，则将此节点不在设置为尾节点
    		if node.EndNode{
    			node.EndNode = false
    		}
    	}
    }
    
    // 根据前缀返回
    func (t *trieTree)TrieData(data string) ([]string, error) {
    	returnData := make([]string,0)
    	if t.root == nil || data == ""{
    		return returnData, fmt.Errorf("数据为空")
    	}
    	nodeNext := t.root
    	for _, cha := range data {
    		nodeNext = nodeNext.ChildNodes[cha]
    	}
    	t.getData(nodeNext, &returnData)
    	return returnData, nil
    }
    
    
    // 初始化树
    func InitTrie() *trieTree {
    	node := creatNode()
    	return &trieTree{
    		root:      node,
    		WordCount: 0,
    	}
    }

```

## 测试代码如下
```go
    package Trie
    
    import (
    	"fmt"
    	"testing"
    )
    
    func TestInsertData(t *testing.T)  {
    	trie := InitTrie()
    	var testStr = []string{"zc", "zzc", "zcj","rr","好的", "zdd", "zcf", "找车估计","张伟"}
    	for _, str := range testStr{
    		err := trie.insertData(str)
    		if err != nil {
    			t.Error(err)
    		}
    	}
    	//trie.Clear()
    	dataS, err := trie.traverseData()
    	if err != nil {
    		t.Error(err)
    	}
    	fmt.Println(dataS)
    	ok := trie.FindData("zcj1")
    	if !ok {
    		fmt.Println("zcj is not  exist" )
    	}else{
    		fmt.Println("zcj is exist" )
    	}
    	//err = trie.DeleteData("rr")
    	//if err != nil {
    	//	t.Error(err)
    	//}
    	dataS, err = trie.traverseData()
    	if err != nil {
    		t.Error(err)
    	}
    	fmt.Println(dataS)
    	fmt.Println("****************************")
    	dataS, err = trie.TrieData("z")
    	if err != nil {
    		t.Error(err)
    	}
    	fmt.Println(dataS)
    
    }
```
## 源码
* [我的github:https://github.com/zcj-git520/DataStructuresAlgorithmsForGo/tree/master/tree/Trie](https://github.com/zcj-git520/DataStructuresAlgorithmsForGo/tree/master/tree/Trie)


