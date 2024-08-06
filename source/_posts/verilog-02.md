---
title: Verilog 学习笔记 02 - Verilog 中的向量
tags:
  - Verilog
  - EE
description: Verilog 中的向量
date: 2024-08-06 11:03:46
---


## 向量（Vector）的基本用法

之前使用 [[01 - Verilog 基础语法#Wire 类型和 Assign 关键字 |wire]] 关键字声明的变量都是一比特的，想要声明多比特的变量就要用到向量，比如 `wire[7:0] w` 就声明了一个 8bit 的 wire 向量，可以表示一个 8 位宽的地址总线。同时，也可以把向量中的某一位分配到端口上（想象下从总线上分出一路线来），比如 `assign out = w[0]`。

也可以用连接运算符将数个端口接到总线上，比如

```verilog
wire[0:2] bus;
assign {o0, o1, o2} = bus;
```

## 向量的细节

```verilog
wire [7:0] w;         // 8bit wire 小端序
reg  [4:1] x;         // 4bit reg，范围可以不从零开始
output reg [0:0] y;   // 1bit 的向量也是向量
input wire [3:-2] z;  // 范围可以是负数
output [3:0] a;       // I/O 端口默认是 wire
wire [0:7] b;         // 8bit wire 大端序
```

向量的字节顺序（Endianness）区分大端序（Big Endian）`[lower, upper]` 与小端序（Little Endian）`[upper, lower]`，对于同一个变量，端序必须一致，在同一项目中最好保持一致。

例如，对于数据 1A2B3C4D:

内存地址增长方向====>

- 大端序：1A2B3C4D
- 小端序：D4C3B2A1

## 向量切片

可以通过切片访问向量的一部分：

```verilog
w[3:0]      // w 的低四位
x[1]        // x 的最低位
x[1:1]      // 同上
z[-1:-2]    // z 的低二位
b[3:0]      // 无效，端序不符
b[0:3]      // b 的高四位（大端序）
```

## 避免隐式连接

看下面的例子：

```verilog
wire [2:0] a, c; // Two vectors 
assign a = 3'b101; // a = 101 
assign b = a; // b = 1 implicitly-created wire 
assign c = b; // c = 001 <-- bug 
my_module i1 (d,e); // d and e are implicitly one-bit wide if not declared.  
// This could be a bug if the port was intended to be a vector.
```

在这个例子中，`a` 是一个 3bit 的 wire 向量，而 `b` 只是个 1bit 的 wire 变量，所以将 `a` 分配到 `b` 时，b 只被分配了 `a[0]`，再将 `b` 分配到 3bit wire 向量 `c` 时，`c` 只能是 `001` 而非 `101`。

在模块文件中添加：

```verilog
`default_nettype none
```

来使上述代码中的第二行报错。

## 封装与未封装的数组（Unpacked Vs. Packed Arrays）

```verilog
reg [7:0] mem [255:0];
// 256 个未封装的元素（向量），每个元素是个 8 比特的向量
reg mem2 [28:0];
// 29 个未封装的比特
```

封装定义在名称前，表示有多少位被“打包”在一起，未封装定义在名称后，通常用于定义内存数组。

## 按位运算与逻辑运算

[[01 - Verilog 基础语法#按位操作符]] 中提到了按位运算，此外，还有一种逻辑运算符：逻辑非 `!`、逻辑和 `&&`、逻辑或 `||`，没有逻辑异或。

对于 1bit 的值而言，按位运算和逻辑运算的结果并无区别，但对于向量而言，逻辑运算会将整个向量当作布尔值处理，非零值为 `True`，零值为 `False`，并只输出一个 1bit 布尔值。按位运算符则将两个 N bit 的值按位做运算，返回的也是一个 N bit 的向量。

{% asset_img wavedrom01.svg %}

从上图可以看出按位或与逻辑或的区别。

## 向量的连接

可以用连接运算符 `{a, b, c}` 将向量连接在一起。

```verilog
wire [2:0] a, b;
wire [5:0] c;
assign a = 3'b000;
assign b = 3'b111;
c = {a, b} // 6'b000111
```

连接运算符可以在赋值的左右两侧使用。

```verilog
input [15:0] in;
output [23:0] out;
assign {out[7:0], out[15:8]} = in;
// 交换 out 的两个字节
assign out[15:0] = {in[7:0], in[15:8]};
// 交换 in 的两个字节
assign out = {in[7:0], in[15:8]};
// 右侧的两个字节被拓展以匹配左侧的 24 比特向量，此时 out[23:16] 全为 0
```

## 向量的复制

与连接运算符类似，可以使用复制运算符复制向量（同时扩充长度）。

```verilog
{5{1'b1}}
// 5'b11111
{2{a,b,c}}
// 即 {a,b,c,a,b,c}
{3'd5, {2{3'd6}}}
// 9'b101_110_110. 是一个 3'b101 与两个 3'b110 的复制的组合
```
