---
layout: post
title: "golang base基础－常量与变量"
description: ""
category: 
tags: []
---
{% include JB/setup %}

常量与变量

    package main
    
    import (
        "fmt"
    )
    
    /**
        变量声明，可用于全局和局部变量声明
        var vName type
        
        只能用于全局变量声明
        var (
            vName1 type
            vName2 type
            ...
        )
        
        变量初始化
        var vName type = value
        var vName = value
        vName := value
        
        常量
        const cName type ＝ value
        const cName = value
        const (
            cName1 = value
            cName2 = value
        )
        
        预定义常量: true false iota
        const (
            a = 1 << iota
            b = 1 << iota
            c = 1 << iota
        )
    */
    
    const (
        a = iota
        b
        c
        d
    )
    
    // 枚举
    const (
        Sunday = iota
        Monday
        Tuesday
        Wednesday
        Thursday
        Friday
        Saturday
    )
    
    func main() {
        fmt.Println(a, b, c, d)
    }


