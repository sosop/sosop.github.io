---
layout: post
title: "go数据结构－线性表"
description: ""
category: "go"
tags: []
---
{% include JB/setup %}

线性表：顺序存储和链式存储

    package dataStructure
    
    import (
        "errors"
        "strconv"
    )
    
    const (
        CAPACITY = 20
    )
    
    type Book struct {
        ID   int
        Name string
    }
    
    // 线性表接口
    type List interface {
        Get(index int) (obj interface{}, err error)
        Add(obj interface{}) (index int, err error)
        AddIndex(index int, obj interface{}) error
        Delete(index int) error
        Size() int
    }
    
    // 顺序存储-begin
    // 顺序存储结构
    type ArrayList struct {
        data   [CAPACITY]interface{}
        length int
    }
    
    // 获取元素，复杂度O(1)
    func (list *ArrayList) Get(index int) (interface{}, error) {
        if index >= list.length || index < 0 {
            return nil, errors.New("index out of bound")
        }
        return list.data[index], nil
    }
    
    // 尾部插入元素，复杂度O(1)
    func (list *ArrayList) Add(obj interface{}) (int, error) {
        if list.length == CAPACITY {
            return -1, errors.New("can not insert element, because the table is full")
        }
        index := list.length
        list.length++
        list.data[index] = obj
        return index, nil
    }
    
    // 指定下标插入元素，复杂度O(n)
    func (list *ArrayList) AddIndex(index int, obj interface{}) error {
        if list.length == CAPACITY || index < 0 || index >= CAPACITY {
            return errors.New("table full or index out of bound!")
        }
        if index >= list.length {
            list.data[index] = obj
        } else {
            for i := list.length - 1; i >= index; i-- {
                list.data[i+1] = list.data[i]
            }
            list.data[index] = obj
        }
        list.length++
        return nil
    }
    
    // 删除指定下标，复杂度O(n)
    func (list *ArrayList) Delete(index int) error {
        if index < 0 || index >= list.length {
            return errors.New("index out of bound!")
        }
        for i := index; i < list.length-1; i++ {
            list.data[i] = list.data[i+1]
        }
        list.length--
        return nil
    }
    
    func (list *ArrayList) Size() int {
        return list.length
    }
    
    // 顺序存储-end
    
    // 链式存储-begin
    type Node struct {
        data interface{}
        next *Node
        pre  *Node
    }
    type LinkedList struct {
        header  *Node
        current *Node
        tail    *Node
        length  int
    }
    
    func (node Node) hasNext() bool {
        if node.next == nil {
            return false
        }
        return true
    }
    
    // 获取元素，复杂度O(n)
    func (list *LinkedList) Get(index int) (interface{}, error) {
        if index >= list.length || index < 0 {
            return nil, errors.New("index out of bound")
        }
        list.current = list.header
        i := 0
        for list.current.hasNext() {
            list.current = list.current.next
            if i == index {
                return list.current, nil
            }
            i++
        }
        return nil, errors.New("can not find index " + strconv.Itoa(index))
    }
    
    // 尾部插入元素，复杂度O(1)
    func (list *LinkedList) Add(obj interface{}) (int, error) {
        if list.length == CAPACITY {
            return -1, errors.New("can not insert element, because the table is full")
        }
        index := list.length
        node := obj.(*Node)
        list.tail.next = node
        node.pre = list.tail
        list.tail = node
        list.length++
        return index, nil
    }
    
    // 指定下标插入元素，复杂度O(n)
    func (list *LinkedList) AddIndex(index int, obj interface{}) error {
        if list.length == CAPACITY || index < 0 || index >= CAPACITY {
            return errors.New("table full or index out of bound!")
        }
        insertNode := obj.(*Node)
        if index >= list.length {
            list.tail.next = insertNode
        } else {
            node, err := list.Get(index)
            if err != nil {
                panic(err)
            }
            node.(*Node).next = insertNode
        }
        insertNode.pre = list.tail
        list.tail = insertNode
        list.length++
        return nil
    }
    
    // 删除指定下标，复杂度O(n)
    func (list *LinkedList) Delete(index int) error {
        if index < 0 || index >= list.length {
            return errors.New("index out of bound!")
        }
        node, err := list.Get(index)
        if err != nil {
            panic(err)
        }
        current := node.(*Node)
        if index == list.length-1 {
            list.tail = current.pre
        } else {
            current.pre.next = current.next
        }
        current = nil
        list.length--
        return nil
    }
    
    func (list *LinkedList) Size() int {
        return list.length
    }
    
    // 链式存储-end
    /*
    func main() {
        var sqList List
        var books [CAPACITY]interface{}
        sqList = &ArrayList{books, 0}
        for i := 0; i < CAPACITY; i++ {
            index := i + 1
            _, e := sqList.Add(Book{index, "study in go " + strconv.Itoa(index)})
            if e != nil {
                panic(e)
            }
        }
    
        book, err := sqList.Get(2)
        if err != nil {
            panic(err)
        }
        
        fmt.Println(book.(Book))
    }
    */


