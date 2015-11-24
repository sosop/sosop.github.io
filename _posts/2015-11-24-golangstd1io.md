---
layout: post
title: "golang标准库io"
description: ""
category: 
tags: []
---
{% include JB/setup %}

io包：

    package libTest
    
    import (
        "bufio"
        "bytes"
        "io"
        "os"
        "strings"
    )
    
    /**
    Reader接口：http://docscn.studygolang.com/pkg/io/#Reader
    Reader接口的Read方法返回错误时，不代表没有读取到任何数据
    */
    func ReadFrom(reader io.Reader, num int) ([]byte, error) {
        bytes := make([]byte, num)
        n, err := reader.Read(bytes)
        if n > 0 {
            return bytes, nil
        }
        return bytes, err
    }
    
    /**
    Writer接口：http://docscn.studygolang.com/pkg/io/#Writer
    遇到的引起写入提前停止的错误，返回的 n < len(p)，它就必须返回一个非nil的错误
    */
    
    /**
    实现了Reader和Writer接口的类型
    os.File os.Stdin os.Stdout os.Stderr
    os.File 同时实现了io.Reader和io.Writer
    strings.Reader 实现了io.Reader
    bufio.Reader/Writer 分别实现了io.Reader和io.Writer
    bytes.Buffer 同时实现了io.Reader和io.Writer
    bytes.Reader 实现了io.Reader
    compress/gzip.Reader/Writer 分别实现了io.Reader和io.Writer
    crypto/cipher.StreamReader/StreamWriter 分别实现了io.Reader和io.Writer
    crypto/tls.Conn 同时实现了io.Reader和io.Writer
    encoding/csv.Reader/Writer 分别实现了io.Reader和io.Writer
    mime/multipart.Part 实现了io.Reader
    LimitedReader、PipeReader、SectionReader实现了Reader
    PipeWriter实现了Writer
    */
    
    /**
    ReaderAt接口：http://docscn.studygolang.com/pkg/io/#ReaderAt
    ReadAt 从基本输入源的偏移量 off 处开始，将 len(p) 个字节读取到 p 中
    WriterAt接口：http://docscn.studygolang.com/pkg/io/#WriterAt
    将 len(p) 个字节写入到偏移量 off 处的基本数据流中
    */
    func testReadAt() ([]byte, error) {
        reader := strings.NewReader("study in golang")
        bytes := make([]byte, 8)
        _, err := reader.ReadAt(bytes, 6)
        if err != nil {
            panic(err)
        }
        return bytes, nil
    }
    
    func testWriteAt() int {
        f, e := os.Create("test")
        if e != nil {
            panic(e)
        }
        defer f.Close()
        f.WriteString("stude in golang")
        n, err := f.WriteAt([]byte("java"), 9)
        if err != nil {
            panic(err)
        }
        return n
    }
    
    /**
    ReaderFrom接口：http://docscn.studygolang.com/pkg/io/#ReaderFrom
    ReadFrom方法不会返回err == EOF
    WriterTo接口：http://docscn.studygolang.com/pkg/io/#WriterTo
    
    */
    func testReaderFrom() {
        f, e := os.Open("test")
        if e != nil {
            panic(e)
        }
        defer f.Close()
        writer := bufio.NewWriter(os.Stdout)
        writer.ReadFrom(f)
        writer.Flush()
    }
    
    func testWriterTo() {
        reader := bytes.NewReader([]byte("stude in golang"))
        reader.WriteTo(os.Stdout)
    }
    
    /**
    Seeker接口：http://docscn.studygolang.com/pkg/io/#Seeker
    const (
        SEEK_SET int = 0 // seek relative to the origin of the file
        SEEK_CUR int = 1 // seek relative to the current offset
        SEEK_END int = 2 // seek relative to the end
    )
    */
    func testSeeker() int32 {
        reader := strings.NewReader("golang学习")
        reader.Seek(2, os.SEEK_SET)
        c, _, _ := reader.ReadRune()
        return c
    }
    
    /**
    Closer接口：http://docscn.studygolang.com/pkg/io/#Closer
    应该将defer file.Close()放在错误检查之后
    */
    
    /**
    ByteReader和ByteWriter接口
    http://docscn.studygolang.com/pkg/io/#ByteReader
    http://docscn.studygolang.com/pkg/io/#ByteWriter
    实现了io.ByteReader或io.ByteWriter的类型：
    bufio.Reader/Writer 分别实现了io.ByteReader和io.ByteWriter
    bytes.Buffer 同时实现了io.ByteReader和io.ByteWriter
    bytes.Reader 实现了io.ByteReader
    strings.Reader 实现了io.ByteReader
    */
    
    /**
    ByteScanner、RuneReader和RuneScanner接口：
    http://docscn.studygolang.com/pkg/io/#ByteScanner
    http://docscn.studygolang.com/pkg/io/#RuneReader
    http://docscn.studygolang.com/pkg/io/#RuneScanner
    */
    
    /**
    SectionReader类型：http://docscn.studygolang.com/pkg/io/#SectionReader
    SectionReader 只是内部（内嵌）ReaderAt表示的数据流的一部分：从 off 开始后的n个字节
    这个类型的作用是：方便重复操作某一段(section)数据流；或者同时需要ReadAt和Seek的功能
    */
    
    /**
    LimitedReader类型：http://docscn.studygolang.com/pkg/io/#LimitedReader
    从 R 读取但将返回的数据量限制为 N 字节
    */
    
    /**
    Copy 和 CopyN 函数：http://docscn.studygolang.com/pkg/io/#Copy http://docscn.studygolang.com/pkg/io/#CopyN
    CopyN 将 n 个字节从 src 复制到 dst
    */
    func testCopy() {
        io.Copy(os.Stdout, strings.NewReader("study in golang"))
    }
    
    /**
    WriteString 函数：http://docscn.studygolang.com/pkg/io/#WriteString
    方便写入string类型提供的函数
    */
    func testWriteString() {
        io.Wr, strings.NewReader("study in golang"))
    }
    
    /**
    WriteString 函数：http://docscn.studygolang.com/pkg/io/#WriteString
    方便写入string类型提供的函数
    */
    func testWriteString() {
        io.Writestrings.NewReader("study in golang"))
    }
    
