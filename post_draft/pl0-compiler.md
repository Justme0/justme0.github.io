

一个小型编译器的实现

本文将从代码上讲解一个小型编译器的实现，其中涉及词法分析、语法分析（含类型检查）、中间代码生成，以及后端的程序虚拟机，写作目的在于帮助理解“高级程序语言”这一概念。

# What
什么是编译器？从我的角度看，

# 效果展示
``` c
/* bsearch.pl0 */

type Arr = array[0..9] of integer; /* typedef an array */
var a: Arr; /* define a global variable */

/*
** Find value in [first, last).
** If found, return its index; otherwise return last index.
*/
function bsearch(first: integer; last: integer; value: integer): integer;
var i: integer; /* define local variables */
    ret: integer;
begin
    ret := last;
    while (first < last) do begin
        i := first + (last - first) div 2;
        if (a[i] = value) then begin
            bsearch := i;   /* similar to return */
        end else if (a[i] < value) then begin
            first := i + 1;
        end else begin
            last := i;
        end;
    end;

    bsearch := ret;
end;

/* main() */
begin
    a[0] := -3;
    a[1] := -2;
    a[2] := 0;
    a[3] := 1;
    a[4] := 1;
    a[5] := 7;
    a[6] := 8;
    a[7] := 11;
    a[8] := 22;
    a[9] := 27;

    write(bsearch(0, 10, 22));  /* 8 */
    write(bsearch(0, 10, -1));  /* 10 */
end.
```

# 整体架构


## S1. 词法分析


## S2. 语法分析


## S3. 中间代码生成


## S4. 后端虚拟机（即解释器）
看到虚拟机这个词，可能大家觉得高大上，笼统地说，虚拟一个执行环境，就可以称为虚拟机。主要分为两类。
 - 提供环境执行操作系统：如VMware，QEMU
 - 提供环境执行应用程序：如Java虚拟机，LLVM。本文介绍的PL/0语言的虚拟机即属于这类，命名为PVM。

### 实现
PVM的输入是S3得到的中间代码，它可以是可读的，也可以是一坨二进制。为了简化说明，就以可读的代码作为输入。


# 展望
oo












