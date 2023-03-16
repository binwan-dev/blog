---
title: Rust 变量
date: 2022-09-03 22:28:33
categories: 
- Rust
---

## 变量 ##

Rust 变量默认是无法改变的。这是Rust推动的特点之一，这样可以利用Rust提供的安全、易并发的方式来编写代码。不过我们也可以选择让变量可变。

当变量不可变时，一旦将值绑定到变量上，就无法更改该值。例如以下示例

首先创建一个项目 variables 
``` bash
cargo new variables
```

在 variables 文件夹中找到文件 src/main.rs
``` rust
fn main(){
	let x = 5;
	println!("The value of x is: {}", x);
	x = 6;
	println!("The value of x is: {}", x);
}
```
保存后运行命令 `cargo run`。会得到以下报错信息：
``` bash
$ cargo run
   Compiling variables v0.1.0 (/Users/binwan/Documents/binwan-dev/rust-hello/variables)
error[E0384]: cannot assign twice to immutable variable `x`
 --> src/main.rs:4:2
  |
2 |     let x = 5;
  |         -
  |         |
  |         first assignment to `x`
  |         help: consider making this binding mutable: `mut x`
3 |     println!("The value of x is: {}", x);
4 |     x = 6;
  |     ^^^^^ cannot assign twice to immutable variable

For more information about this error, try `rustc --explain E0384`.
error: could not compile `variables` due to previous error
```

`cannot assign twice to immutable variable 'x'` 指示变量x不能被分配两次。
当我们尝试在代码中更改一个不可更改的值时，编译器将会提示错误，我们必须重视这个错误。如果代码的一部分假设某个值永远不会更改，而代码的另一部分更改该值，则代码的第一部分可能无法执行其设计的操作。这种错误很难在事后跟踪，特别是第二段代码更改该值时。


可变形有时也是非常有用的，可以使代码编写起来更方便。Rust中可以通过添加 `mut` 在变量名称前面。添加该指示代码后变量将可以被更改。
例如以下示例：

在文件 `src/main.rs` 中：
``` rust
fn main() {
    let mut x = 5;
    println!("The value of x is: {}", x);
    x = 6;
    println!("The value of x is: {}", x);
}
```
运行代码 `cargo run` 后输出以下：
``` bash
$ cargo run
   Compiling variables v0.1.0 (/Users/binwan/Documents/binwan-dev/rust-hello/variables)
    Finished dev [unoptimized + debuginfo] target(s) in 0.14s
     Running `target/debug/variables`
The value of x is: 5
The value of x is: 6
```
在使用更改值时，通常有许多因素需要考虑。例如，在使用大型数据结构时，使用 `mut` 就地更改要比复制和返回新实例更快。对于较小的数据结构，创建新实例并以更具函数式的编程风格编写可能更容易思考。


## 常量 ##
和变量相似，常量也是声明后无法更改，但是它们之间又有一些不同。
常量是任何情况下都不能更改的，可以使用 `const` 代码声明一个常量，同时常量需要指定数据类型（例如：u32 等）。
常量只能设置为常量表达式，不能设置为运行时计算的结果。
例如：
``` rust
	const THREE_HOURS_IN_SECONDS: u32 = 60 * 60 * 3;
```

## 重影（shadowing） ##
变量重影可以申明一个同名的变量，并将前一变量的值重影。并且在作用域不同的情况下，可以局部重影。使用关键字 `let` 。
示例：src/main.rs
``` rust
fn main(){
	let x = 5;
	let x = x +1;
	{
		let x = x * 2;
		println!("The value of x is the inner scope is: {}", x);
	}
	
	println!("The value of x is: {}", x);
}
```
运行命令 `cargo run` 输出如下：
``` bash
$ cargo run
   Compiling variables v0.1.0 (/Users/binwan/Documents/binwan-dev/rust-hello/variables)
    Finished dev [unoptimized + debuginfo] target(s) in 0.12s
     Running `target/debug/variables`
The value of x is the inner scope is: 12
The value of x is: 6
```
可以看出在局部作用域的重影变量并没有改变全局的重影，这说明重影作用范围是可以局部的。
重影和变量标记不同，重影是可以更改变量类型的，而`mut`只能改变值但不能更改类型。
例如：
``` rust
let spaces = "     ";
let spaces = spaces.len();
```
这是被允许的。
当我们使用`mut` 时：
``` rust
let mut spaces = "      ";
spaces = spaces.len();
```
运行后会出现`mismatched types`错误。
``` bash
$ cargo run
   Compiling variables v0.1.0 (file:///projects/variables)
error[E0308]: mismatched types
 --> src/main.rs:3:14
  |
3 |     spaces = spaces.len();
  |              ^^^^^^^^^^^^ expected `&str`, found `usize`

For more information about this error, try `rustc --explain E0308`.
error: could not compile `variables` due to previous error
```

[官方原文](https://doc.rust-lang.org/book/ch03-01-variables-and-mutability.html#shadowing)
