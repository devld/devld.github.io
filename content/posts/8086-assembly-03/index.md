---
title: "8086 汇编笔记 - 常用指令 - 03"
date: "2017-11-16T12:47:00+08:00"
categories:
- 汇编
tags:
- 汇编
- notes
---

### 数据传送

1. MOV dest, src

  将 src 中的数据传送至 dest.

  ```x86asm
  MOV AX, BX  ; CPU 的通用寄存器之间的数据传送
  MOV AX, 1234H ; 立即数 -> 寄存器
  MOV DS:[1234H] 5678H  ; 立即数 -> 存储单元

  MOV AX, DS:[1234H]  ; 存储单元 -> 寄存器
  MOV DS:[1234H], BX  ; 寄存器 -> 存储单元

  MOV AX, ES  ; 段寄存器 -> 通用寄存器
  MOV DS, AX  ; 通用寄存器 -> 段寄存器
  MOV ES, DS:[BX] ; 存储单元 -> 段寄存器
  MOV DS:[1234H], CS  ; 段寄存器 -> 存储单元

  MOV WORD PTR DS:[0], 12H ; 将 12H 当作 字型数据 放入 DS:[0]
  ```

  <!-- more -->

  使用 MOV 指令时需注意以下几点:

  - MOV 可传送 8 位或 16 位数据, 取决于寄存器.

     如果 MOV 指令中无法判断出操作数的位数, 则需要显式声明数据位数.
     比如上面代码的最后一行, 表明操作数是一个 16 位的字型数据, 相应的还有 `BYTE PTR` 字节型数据.

  - 一条 MOV 指令无法在存储单元与存储单元之间传送数据.

  - CS 和 IP 不能作为 dest.

  - 不能在段寄存器之间传送数据. 比如 `MOV DS, ES` 会报错.

  - 立即数不能作为 dest.

  - 不能向段寄存器传送立即数. 比如 `MOV DS, 1234H` 会报错. 可以通过通用寄存器作为中转.

    ```x86asm
    MOV AX, 1234H
    MOV DS, AX
    ```

2. XCHG dest, src

  用于交换数据, 相当于三条 MOV 指令. 指令中的两个操作数可以是两个寄存器或寄存器与存储单元.

  使用 XCHG 需要注意以下几点:

  - 两个操作数不能同时为存储单元

  - 任何一个操作数都不能为段寄存器和立即数

3. LEA r16, mem

  取有效地址送入 16 位寄存器 r16.

  ```x86asm
  LEA SP, [BP][DI]  ; 将 [BP+DI] 寻址方式的偏移地址送入 SP
  ```

  下面的代码的作用相同, 但后一条指令使用了 OFFSET 伪指令, 由编译器在编译时赋值.

  ```x86asm
  LEA BX, VAR
  MOV BX, OFFSET VAR
  ```

4. LDS 和 LES

  ```x86asm
  LDS dest, src
  LES dest, src
  ```

  装入地址指令

  LDS 和 LES 都将 src 指向的内存中连续的 4B 内容的低 16 位送入 dest 指定的通用寄存器.
  LDS 将 上述 4B 的高16为装入 DS, LES 则装入 ES.

  dest 必须是通用寄存器, src 比如是内存操作数.

5. LAHF 和 SAHF

  标志传送指令

  LAHF 将[标志寄存器][2]的低 8 位放入 AH

  ![LAHF][1]

  SAHF 将 AH 的内容送入标志寄存器的低 8 位

6. PUSH 和 POP

  ```x86asm
  PUSH src
  POP  dest
  ```

  操作堆栈

  src 和 dest 可以为 寄存器或存储单元

  PUSH 将 src 中的 16 位数据放入 SS:[SP] 的位置, 并将 SP - 2.
  POP 将 SS:[SP] 的 16 位数据放入 dest, 并将 SP + 2.

7. CBW 和 CWD

  字扩展指令 CBW

  将 AL 中的字节数据扩展到字型数据, 高 8 位 放入 AH.

  字扩展双字指令 CWD

  将 AX 中的字型数据扩展到双字型, 高 16 位放入 DX.

  ```x86asm
  MOV AL, 12H
  CBW ; AL 符号位为 0, 所以 0 -> AH

  MOV AL, AAH
  CBW ; AL 符号位为 1, 所以 FF -> AH

  MOV AX, 1234H
  CWD ; 0 -> DX

  MOV AX, AAAAH
  CWD; FFFF -> DX
  ```


### 算术运算指令

1. 加法

  - 不带进位加法

    ```x86asm
    ADD dest, src ; (src) + (dest)  -> dest
    ```

    + ADD 指令的 src 和 dest 不能同时为两个存储单元
    + 段寄存器之间也不能相加
    + 主要影响 CF, ZF, OF, SF 标志寄存器

  - 带进位的加法

    ```x86asm
    ADC dest, src ; (src) + (dest) + (CF) -> dest
    ```

  - 加 1 指令

    ```x86asm
    INC reg/mem ; (reg/mem) + 1 -> reg/mem
    ```

    + 操作数可以是寄存器或存储单元, 但不能为立即数
    + 影响 AF, OF, PF, SF, ZF, 但不影响 CF
    + INC 指令会将操作数视为无符号数

2. 减法

  - 不带进位减法

    ```x86asm
    SUB dest, src ; (dest) - (src) -> dest
    ```

  - 带进位减法

    ```x86asm
    SBB dest, src ; (dest) - (src) - (CF) -> dest
    ```

  - 减 1 指令

    ```x86asm
    DEC reg/mem ; (reg/mem) - 1 -> reg/mem
    ```

    DEC 指令不影响 CF 标志.

3. 取补

  ```x86asm
  NEG reg/mem ; 0 - (reg/mem) -> reg/mem
  ```

  将操作数以及符号为逐位取反, 然后加 1.

4. 乘法

  - 无符号乘法

    ```x86asm
    MUL reg/mem
    ```

    如果操作数为 8 位, (AL) * (reg/mem) -> AX
    如果操作数为 16 位, (AX) * (reg/mem) - > DX, AX

  - 有符号乘法

    ```x86asm
    IMUL reg/mem
    ```

5. 除法

  - 无符号除法

    ```x86asm
    DIV src
    ```

    + 当 src 为 8 位时

      (AX) / (src) 的商 -> AL
      (AX) / (src) 的余 -> AH

    + 当 src 为 16 位时

      (DX, AX) / (src) 的商 -> AX
      (DX, AX) / (src) 的余 -> DX

    注意: 当除法的商过大(超过 AL, AX 的范围时, 会产生异常中断)

    ![Divide overflow][3]

  - 有符号除法

    ```x86asm
    IDIV src
    ```

    与 DIV 功能相同, 但除数, 被除数都被当作有符号数.

6. 比较指令

  ```x86asm
  CMP dest, src ; (dest) - (src)
  ```

  将目标操作数与源操作数相减, 不送回结果, 只更改标志位.

  - 判断两数相等

    根据 ZF 标志位, 若 ZF = 1 则 相等

  - 判断无符号数大小

    根据 CF 标志位, 若 CF = 1, 则 (dest) < (src)

  - 判断有符号数大小

    可根据 SF 和 OF 标志判断, 若 SF ⊕ OF = 1, 即 SF 与 OF 不同时, dest < src


### 逻辑指令

```x86asm
AND   dest, src ; 按位且 (dest) & (src) -> dest
OR    dest, src ; 按位或 (dest) | (src) -> dest
XOR   dest, src ; 按位异或 (dest) ^ (src) -> dest
NOT   reg/mem   ; 按位取反 ~(reg/mem) -> reg/mem
TEST  dest, src ; 按位与, 只设置标志位
```


### 移位指令

```x86asm
SAL   reg/mem, 1/CL ; 算数左移
SAR   reg/mem, 1/CL ; 算数右移

SHL   reg/mem, 1/CL ; 逻辑左移
SHR   reg/mem, 1/CL ; 逻辑右移

; 算数左/右移与逻辑左/右移的区别在于前者会保持符号位不变.

ROL   reg/mem, 1/CL ; 循环左移
ROR   reg/mem, 1/CL ; 循环右移

RCL   reg/mem, 1/CL ; 带进位循环左移
RCR   reg/mem, 1/CL ; 带进位循环左移

; 带进位与不带进位的区别是
; RCL/RCR 将 CF 标志位也加入了移位操作中
; 变成了 9 位 的循环移位操作

```

### 处理器指令

```x86asm
CLC   ; 0 -> CF
STC   ; 1 -> CF

CLD   ; 0 -> DF
STD   ; 1 -> DF

CLI   ; 0 -> IF
STI   ; 1 -> IF

CMC   ; !CF -> CF, CF 取反
```

### 转移指令

- 无条件转移指令

  ```x86asm
  JMP   dest
  JMP   reg/m16
  ```

  JMP 分为三种转移方式

  + 短转移

    在段内短距离(-128 ~ 127)转移

  + 段内转移

    CS 不变, 给出转移的 16 位偏移地址

  + 段间转移

    CS 和 IP 都改变的转移
    给出一个双字的数据, 低 16 位 -> IP, 高 16 位 -> CS

- 条件转移指令

  + 无符号数

    ```x86asm
    JZ
    JE  ; ZF = 1

    JNE
    JNZ ; ZF = 0

    JA
    JNBE  ; CF = 0 && ZF = 0
    ; Jump if Not Below or Equal

    JAE
    JNB   ; CF = 0
    ; Jump if Not Below

    JB
    JNAE  ; CF = 1 && ZF = 0
    ; Jump if Not Above or Equal

    JBE
    JNA   ; CF = 1
    ; Jump if Not Above
    ```

  + 有符号数

    ```x86asm
    JG
    JNLE  ; SF = OF && ZF = 0
    ; Jump if Not Less or Equal

    JGE
    JNL   ; SF = OF
    ; Jump if Not Less

    JL
    JNGE  ; SF != OF
    ; Jump if Not Greater or Equal

    JLE
    JNG   ; SF != OF || ZF = 1
    ; Jump if Not Greater
    ```

  + 特殊算数标志位

    ```x86asm
    JC    ; CF = 1
    JNC   ; CF = 0

    JO    ; OF = 1
    JNO   ; OF = 0

    JP
    JPE   ; PF = 1
    JNP
    JPO   ; PF = 0

    JS    ; SF = 1
    JNS   ; SF = 0

    JCXZ  ; CX = 0
    ```

### 循环指令

1. LOOP label

  执行 LOOP 指令时, 处理器会先将 CX - 1, 然后判断 若 CX != 0, 则继续循环, 否则退出循环.

2. LOOPE/LOOPZ label

  与 LOOP 相同, 首先 CX - 1 (ZF 不受影响), 然后判断:
  ```
    若 CX = 0
      跳出循环
    否则
      若 ZF = 0
        跳出循环
      否则
        继续循环
  ```

3. LOOPNE/LOOPNZ label

  与 LOOPE/LOOPZ 不同的地方在 当 ZF = 1 时跳出循环


### 串处理操作指令

```x86asm
MOVSB
MOVSW
; 将 DS:SI 所指向的 字节/字 数据传送到 ES:DI 指向的位置
; 然后根据 DF 的值确定 递增(DF = 0)或递减(DF = 1) SI 与 DI 的值

CMPSB
CMPSW
; 将 SI 指向的 字节/字 数据与 DI 指向的数据相减比较
; 然后设置相关的标志位(AF, CF, OF, PF, SF, ZF)
; 然后根据 DF 改变 SI 与 DI

SCASB
SCASW
; 将 AL/AX 的内容与 DI 指向的数据相减比较, 设置标志位
; 然后根据 DF 改变 DI

LODSB
LODSW
; 将 SI 指向的 字节/字 数据放入 AL/AX

STOSB
STOSW
; 将 AL/AX 的内容放入 DI 指向的内存单元
; 然后根据 DF 改变 DI
```

重复前缀

- REP

  无条件执行跟在后面的指令, 由 CX 指定重复次数

- REPE/REPZ

  当后面的指令的结果使 ZF = 1 时, 重复执行, 并且由 CX 指定最大重复次数
  该指令与 CMPS 和 SCAS 配合使用

- REPNE/REPNZ

  与 REPE/REPZ 条件相反, ZF = 0 时, 重复执行


### 子程序调用返回指令

- 定义子程序

  ```x86asm
  <name> PROC [NEAR/FAR]
      ; code
  <name> ENDP
  ```

- 子程序的调用

  ```x86asm
  CALL <name>
  ```

  根据调用出近调用还是远调用, 将 IP / (CS, IP) 压入栈中, 然后转去执行子程序.

- 返回指令

  ```x86asm
  RET [n]
  ```

  将堆栈顶部的 IP / (IP, CS) 弹出, 然后返回到调用的地方.
  如果指定了参数 n, 则在弹出堆栈之后, (SP) + n -> SP.

### 8086 软中断

```x86asm
INT n
```

执行这条指令时 CPU 会将标志寄存器压入堆栈,
禁止新的可屏蔽中断和单步中断(IF = 0, TF = 0),
并将当前的 CS 和 IP 压入堆栈.
然后会在 0000H 段中的偏移地址为 4n 的地方找到中断处理程序的 CS 和 IP.
4n 处为 IP, 4n + 2 处为 CS, 然后转去执行.

```x86asm
IRET
```

中断返回指令, 会自动将 IP, CS, FLAGS 出栈.

[1]: lahf-flags-to-ah.png
[2]: /2017/10/29/8086-assembly-01/#8086-寄存器
[3]: 8086-asm-div-overflow.png
