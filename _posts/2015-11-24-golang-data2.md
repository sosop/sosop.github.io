---
layout: post
title: "go数据结构－栈于队列"
description: ""
category: "go"
tags: []
---
{% include JB/setup %}

栈与队列

    package dataStructure
    
    import (
        "errors"
    )
    
    /**
    栈：表尾进行增加删除的线性表
    */
    
    type Linear interface {
        pop() interface{}
        push(e interface{}) (int, error)
        size() int
    }
    
    type NodeStack struct {
        data interface{}
        next *NodeStack
    }
    
    type Stack struct {
        top    NodeStack
        length int
    }
    
    func (stack *Stack) pop() interface{} {
        if stack.length == 0 {
            return nil
        }
        node := stack.top
        stack.top = *node.next
        stack.length--
        return node
    }
    
    func (stack *Stack) push(e interface{}) (int, error) {
        if stack.length >= CAPACITY {
            return -1, errors.New("stack is full")
        }
        node := stack.top
        newTop := e.(NodeStack)
        stack.top = newTop
        newTop.next = &node
        stack.length++
        return 1, nil
    }
    
    func (stack *Stack) size() int {
        return stack.length
    }
    
    // 循环实现斐波那契数
    func Fbi(n int) (int, error) {
        if n <= 0 {
            return -1, errors.New("bound of index")
        }
        if n <= 2 {
            return 1, nil
        }
        fb := make([]int, n, n)
        fb[0] = 1
        fb[1] = 1
        
        for i := 2; i < n; i++ {
            fb[i] = fb[i-1] + fb[i-2]
        }
        return fb[n-1], nil
    }
    
    // 递归实现斐波那契数
    func FbiRecursive(n int) int {
        if n <= 0 {
            panic("bound of index")
        }
        if n <= 2 {
            return 1
        }
        return FbiRecursive(n-1) + FbiRecursive(n-2)
    }
