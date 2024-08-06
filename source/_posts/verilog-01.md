---
title: Verilog 学习笔记 01 - Verilog 基础语法
tags:
  - Verilog
  - EE
date: 2024-04-10 22:09:15
description: Verilog 基础语法
---


## Verilog 模块

一个 Verilog 项目由数个模块构成，每个模块由 `module` 开始，`endmodule` 结束。例如：

```verilog
module top_module( input in, output out );

endmodule
```

在上面这个例子中，`top_module` 就是模块名称，后面括号里则是输入输出端口。

## Wire 类型和 Assign 关键字

Verilog 中的 wire 是定向的，从一段流向另一段，而且只能用于传递数据而不能存储。模块中定义的输入与输出默认是 wire 类型：

```verilog
module my_module(input a, input b, output c);
    // 这里的 a, b, c 都默认是 wire 类型的信号
    // 可以在模块内部直接使用这些信号进行逻辑操作
    assign c = a & b;
endmodule
```

`assign` 关键字用于向 wire 连续赋值——想象下：当导线的一端改变时，另一端也将立即改变，这也符合 wire 不存储只传递的特性。`wire` 类型可以先定义再用 `assign` 赋值，也可以在定义时直接赋值，不用 `assign`。

{% asset_img img01.png %}

`assign` 语句实际上描述了模块内各部分的*连接*，而不是赋值，所以，`assign` 语句的先后顺序并不影响分配的结果，同时，`assign` 并不是导线，它只是把导线连到一起，导线是单独声明的（使用 `wire` 关键字）。

下面这个例子展示了一个输入信号可以连接到多个输出信号（想象一个端点连接两条线到两个不同的端点）。

```verilog
module top_module (
 input a,
 input b,
 input c,
 output w,
 output x,
 output y,
 output z  );
 
 assign w = a;
 assign x = b;
 assign y = b;
 assign z = c;

 // If we're certain about the width of each signal, using 
 // the concatenation operator is equivalent and shorter:
 // assign {w,x,y,z} = {a,b,b,c};
endmodule
```

{% asset_img img02.png %}

如注释里写的，这里也可以用连接运算符，形式是 `{signal1, signal2, …, signalN}`，这个符号的作用是把几个信号连起来组成一个带宽更大的信号，比如 `{1'b0, 1'b1}` 就等同于 `{2'b01}`。

## 按位操作符

与 C 语言一样，Verilog 也有按位操作符，非 `~`、与 `&` 、或 `|` 和异或 `^`。有了这些，就可以组合出其他的逻辑门了，比如异非或：

```verilog
module xnor(input a, input b, output out);
 assign out = ~(a ^ b);
endmodule
```
