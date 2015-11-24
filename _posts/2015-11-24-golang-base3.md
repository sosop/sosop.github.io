---
layout: post
title: "golang基础－面向对象"
description: ""
category: "go"
tags: []
---
{% include JB/setup %}

面向对象：

    package main
    
    import (
        "fmt"
    )
    
    // 定义类型
    type Integer int
    
    //为Integer添加方法
    func (this Integer) GreatThan(other Integer) bool {
        return this > other
    }
    
    // 修改对象时，才必须传指针
    
    /**
    值语义和引用语义
    注：数组也是值语义
    */
    
    // 结构体
    type Object struct {
    }
    
    // 初始化
    /**
    o = new(Object)
    o = Object{}
    o = &Object{}
    */
    
    // 匿名组合
    // 首字母大小写决定可见性
    
    // 非侵入式接口：一个类只要实现了接口要求的所有函数，那么这个类就实现类该接口
    // 接口赋值：对象实例赋值给接口，接口对象赋值给接口
    // 接口查询：
    /**
    obj, ok := object.(interface)
    */
    
    // 类型查询
    /**
    v.(type)
    */
    // 接口组合
    // any类型
    
    func main() {
        var a Integer = 10
        fmt.Println(a.GreatThan(8))
    }
    
