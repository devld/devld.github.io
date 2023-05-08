---
title: "8086 汇编笔记 - 程序代码结构 - 02"
date: "2017-10-29T17:28:00+08:00"
categories:
- 汇编
tags:
- 汇编
- notes
---

> 这里的汇编是 16位汇编, 在 Win7 64位 及以后已经不支持
目前有两种方法: 一是使用虚拟机, 另外就是 安装一个 `DosBox`
工具合集: [AsmTool.zip][1] 包含: masm 5和6 以及 DosBox

先来一个 Hello World

```x86asm
ASSUME CS:CODE      ; 声明代码段

DATA SEGMENT        ; 数据段
    HELLOWORLD   DB 'Hello World', '$'    ; HelloWorld 字符串
DATA ENDS

STACK SEGMENT STACK ; 堆栈段
    DB 32 DUP(0)    ; 声明一个32 bytes的连续空间当作堆栈段
STACK ENDS

CODE SEGMENT        ; 代码段
START:
    MOV AX, DATA    
    MOV DS, AX      ; 设置 DS 为数据段

    MOV AH, 09H
    MOV DX, OFFSET HELLOWORLD
    INT 21H         ; 调用 DOS 的中断, 将DX 指向的字符串打印出来

    MOV AX, 4C00H
    INT 21H         ; 程序退出
CODE ENDS
END START           ; 汇编结束,  并表示 程序起始位置为 标号 START 处
```

简化版 (MASM 6.x+)

<!-- more -->

```x86asm
.MODEL SMALL        ; 程序的存储模式
.STACK              ; 定义栈
.DATA               ; 定义数据段
	HELLOWORLD DB 'Hello World', '$'

.CODE               ; 定义代码段
.STARTUP            ; 定义程序起始位置

	MOV AH, 09H
	MOV DX, OFFSET HELLOWORLD
	INT 21H

.EXIT 0             ; 程序退出
	END               ; 汇编结束
```

#### 程序存储模式 (MODEL)

```x86asm
.MODEL 存储模式 [, 语言类型] [, 操作系统类型] [, 堆栈选项]
```

 模式   | 代码段大小 | 非代码段大小 | 说明
:------:|:----------:|:------------:|-------------------------------------------------------------------------------------------
 TINY   |    64kb    |     64kb     | 代码段和数据段的寄存器都被设置为同一值, 程序大小不能超过 64kb
 SMALL  |    64kb    |     64kb     | 只能有一个代码段和一个数据段, 程序最大 128kb
COMPACT |    64kb    |    64kb+     | 只能有一个代码段, 可以有多个数据段
MEDIUM  |   64kb+    |     64kb     | 可以有多个数据段, 只能有一个代码段
 LARGE  |   64kb+    |    64kb+     | 代码段和数据段都可以有多个, 但全部的静态数据(不能改变的数据)仍限制在 64kb 内
 HUGE   |   64kb+    |    64kb+     | 与 LARGE 模式相同, 但静态数据没有限制
 FLAT   |     -      |      -       | 只能运行在 32位的 x86 CPU上, DOS下无法使用. Windows 9.x, Windows NT 的程序必须使用此模式. 

#### 简化定义伪指令

- 段定义

  ```x86asm
  ; 定义堆栈段, 段名为 STACK, 大小默认为 1kb
  .STACK [大小]

  ; 定义数据段, 段名是 _DATA. 用于定义变量
  .DATA

  ; 定义无初值变量段, 段名为 _BS. 使用这个可以减小 EXE文件大小
  .DATA ?

  ; 定义只读的常量数据段, 段名为 CONST
  .CONST

  ; 创建一个代码段
  .CODE [段名]
  ```

- 程序开始伪指令

  ```x86asm
  .STARTUP
  ```

  .STARTUP 伪指令安装给定的 CPU 类型, 根据 .MODEL 选择的存储模式, 操作系统类型和堆栈类型, 产生程序开始执行的代码；同时指定了程序开始执行的起始位置. 
  在DOS 模式下, 它还会初始化 DS 的值, 调整 SS 和 SP 的值. 

  在 SMALL 存储模式下,  .STARTUP 会被编译成如下启动代码:

  ```x86asm
  MOV DX, DGROUP      ; DGROUP 表示数据段组的段地址
  MOV DS, DX
  MOV BX, SS
  SUB BX, DX
  SHL BX, 1
  SHL BX, 1
  SHL BX, 1
  SHL BX, 1
  CLI                 ; 关中断
  MOV SS, DX          ; 调整 SS = DS, 这是 SMALL 模式的规定
  ADD SP, BX          ; SP 也需要相应的调整
  STI                 ; 开中断
  ```

- 程序结束未指定

  ```x86asm
  . EXIT [返回码]
  ```

  终止程序的的执行, 返回操作系统, 它对应的代码是

  ```x86asm
  MOV AX, 4C00H
  INT 21H
  ```

- 汇编结束伪指令

  ```x86asm
  END [标号]
  ```

  指示汇编程序 MASM 到此结束汇编过程. 
  源程序最后一行必须有一条 END 指令, 可选的标号表示程序开始执行的位置. 
  例如 START, 连接程序据此设置 CS:IP 的值. 


[1]: https://pan.baidu.com/s/1qXNIHPy
