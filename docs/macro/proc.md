# 过程宏

虽然 大部分情况下 我们并不会直接使用 `过程宏` 但是他确实是所有宏的基础，过程宏根据使用形式，又被分为三种

 - 类函数宏:  如果定义了一个类函数宏 `custom`，对于他的调用就像是这样 `custom!()` 
 - 派生宏: 如果定义了一个派生宏 `customDerive`, 对于他的调用就像是 `#[derive(customDerive)]`
 - 属性宏: 如果定义了一个属性宏 `customAttribute`, 对于他的调用就像是 `#[customAttribute]`

### 过程宏前置内容

出于某种考虑， 过程宏的代码必须单独的作为一个crate编译 ,并且编译时 必须明确告诉编译器  `crate-type` = `proc-macro` 

```
rustc --crate-type proc-macro

注意: 使用 Cargo 时，定义过程宏的 crate 的配置文件里要使用 proc-macro键做如下设置：
[lib]
proc-macro = true
```

当这样编译一个过程宏的包时，编译器会默认链接 一个名为 `prco_macro`的库，并导入到当前包中，该库提供了关于过程宏所需要的
必要的类型和工具函数


### 过程宏调试

在开发复杂的宏时，我们需要分析代码是否为我们生成了合适的代码，C里面可以在预编译阶段查看，RUST 也提供了类似的方法

```
rustc +nightly -Zunpretty=expanded hello.rs
```

后面会简单演示


### TokenStream 

`prco_macro` 库中 最重要的一个类型为  `TokenStream`,我们简单说明一下，前一个小节我们也简单说明过，过程宏的工作模式

本质上就是消耗一个  `TokenStream` 返回一个新的`TokenStream`,  `TokenStream` 本质上就是一个代码词的流


`TokenStream` 大致相当于 `Vec<TokenTree>` 其中 `TokenTree`可以大致视为词法 `token` 

```
pub enum TokenTree {
    Group(Group),
    Ident(Ident),
    Punct(Punct),
    Literal(Literal),
}

//foo 是标识符(Ident)类型的 token
//10 是一个字面量(Literal)类型的 token
// =  是一个标点符号(Punct)类型的 token
let foo = 10; 

```

下面是我们定义的一个 proc_macro模块: 

```
//文件路径
macro_proc/
├── libmyproc.so
└── lib.rs

编译命令: rustc  --crate-type proc-macro lib.rs    --crate-name myproc 

extern crate proc_macro;

use proc_macro::{TokenStream, TokenTree};

#[proc_macro]
pub fn double(ts: TokenStream) -> TokenStream {
    let mut it = ts.into_iter();

    let a = match it.next() {
        Some(TokenTree::Ident(ident)) => ident.to_string(),
        Some(TokenTree::Literal(lit)) => lit.to_string(),
        _ => panic!("input is not an ident"),
    };

    assert!(it.next().is_none(), "only support one ident");

    format!("{} * 2", a).parse().expect("Error parsing formatted string into token stream")
}

```

下面是我们定义的一个测试模块: 

```
extern crate myproc;
use myproc::double;
fn main() {
	let a = double!(2);
	println!("{}",a);
	println!("{}",double!(a));
}

```



### 过程宏的卫生性 

过程宏是不卫生的 


```
#[proc_macro]
pub fn switch(ts: TokenStream) -> TokenStream {
    let mut it = ts.into_iter();
	let (x,y,temp);
	
    let a = match it.next() {
        Some(TokenTree::Ident(ident)) => ident.to_string(),
        _ => panic!("input is not an ident"),
    };
	let b = match it.next() {
        Some(TokenTree::Ident(ident)) => ident.to_string(),
        _ => panic!("input is not an ident"),
    };

    assert!(it.next().is_none(), "only support two ident");

    format!("
	let temp = {}; // temp = a 
	{} = {}; // a = b 
	{} = {}; // b = temp", a, a, b, b).parse().expect("Error parsing formatted string into token stream")
}
```

```
extern crate myproc;
use myproc::{double,switch};

fn main() {
    let mut  x=10;
    let mut y=20;
    let temp = 100;

    switch!(x,y);

    assert_eq!(x, 20);
    assert_eq!(y, 10);

    println!("temp {}",temp);
    assert_eq!(temp, 10);

}

```

### 类函数过程宏

类函数过程宏是使用宏调用运算符`（!）`调用的过程宏。

这种宏是由一个带有 `proc_macro属性` 和 `(TokenStream) -> TokenStream`签名的 
公有可见性函数定义。输入 `TokenStream` 是由宏调用的`定界符`界定的内容，输出 `TokenStream `将替换整个宏调用。

下面是一个简单是的示例 下面的宏定义忽略它的输入，并将函数 answer 输出到它的作用域。
```
extern crate proc_macro;
use proc_macro::TokenStream;

#[proc_macro]
pub fn make_answer(_item: TokenStream) -> TokenStream {
    "fn answer() -> u32 { 42 }".parse().unwrap()
}
```

类函数过程宏可以在任何宏调用位置调用，这些位置包括

 - 语句、表达式、模式、类型表达式、程序项可以出现的位置 
 - 包括extern块里、固有(inherent)实现里和 trait实现里、以及 trait声明里）。

至于如何使用`TokenStream` 建议可以参考标准宏库的使用还有 `rust for linux`的宏的使用


### 派生宏

派生宏是最常用的过程宏 

派生宏为派生(derive)属性定义新输入。
这类宏在给定输入结构体(struct)、枚举(enum)或联合体(union) token流的情况下创建新程序项。
它们也可以定义派生宏辅助属性。

自定义派生宏由带有 `proc_macro_derive` 属性和 `(TokenStream) -> TokenStream`签名的公有可见性函数定义。

派生宏定义
```
extern crate proc_macro;
use proc_macro::TokenStream;

#[proc_macro_derive(AnswerFn)]
pub fn derive_answer_fn(_item: TokenStream) -> TokenStream {
    "fn answer() -> u32 { 42 }".parse().unwrap()
}
```

派生宏使用

```
extern crate proc_macro_examples;
use proc_macro_examples::AnswerFn;

#[derive(AnswerFn)]
struct Struct;

fn main() {
    assert_eq!(42, answer());
}
```

### 属性宏
属性宏由带有 `proc_macro_attribute`属性和 `(TokenStream, TokenStream) -> TokenStream`签名的公有可见性函数定义。
签名中的第一个 `TokenStream `是属性名称后面的定界` token树(delimited token tree)（不包括外层定界符）`。
如果该属性作为裸属性(bare attribute)给出，则第一个 `TokenStream 值为空`。
第二个 `TokenStream `是程序项的其余部分，包括该程序项的其他属性。
输出的 TokenStream 将此属性宏应用的程序项替换为任意数量的程序项。

例如，下面这个属性宏接受输入流并按原样返回，实际上对属性并无操作。

```
#![crate_type = "proc-macro"]
extern crate proc_macro;
use proc_macro::TokenStream;

#[proc_macro_attribute]
pub fn return_as_is(_attr: TokenStream, item: TokenStream) -> TokenStream {
    item
}
```
下面示例显示了属性宏看到的字符串化的 `TokenStream`。
输出将显示在编译时的编译器输出窗口中。（具体格式是以 "out:"为前缀的）输出内容也都在后面每个示例函数后面的注释中给出了。


```
#[proc_macro_attribute]
pub fn show_streams(attr: TokenStream, item: TokenStream) -> TokenStream {
    println!("attr: \"{}\"", attr.to_string());
    println!("item: \"{}\"", item.to_string());
    item
}
```

使用示例
```
// src/lib.rs
extern crate my_macro;

use my_macro::show_streams;

// 示例: 基础函数
#[show_streams]
fn invoke1() {}
// out: attr: ""
// out: item: "fn invoke1() { }"

// 示例: 带输入参数的属性
#[show_streams(bar)]
fn invoke2() {}
// out: attr: "bar"
// out: item: "fn invoke2() {}"

// 示例: 输入参数中有多个 token 的
#[show_streams(multiple => tokens)]
fn invoke3() {}
// out: attr: "multiple => tokens"
// out: item: "fn invoke3() {}"

// 示例:
#[show_streams { delimiters }]
fn invoke4() {}
// out: attr: "delimiters"
// out: item: "fn invoke4() {}"
```


### 总结

本节重点在于介绍过程宏的 生成替换原理， 以及过程宏的三种形式；

一般如果要实现一个真正的过程宏，按照以下流程去实现: 

 - 需要实现什么功能？决定是定义 函数宏？ 属性宏？ 派生宏？
 - 预期希望实现的最终代码 希望是什么样子的？ 
 - 定义自己的宏
 - 利用 `proc_macro` 提供的关于宏的各种 类型和方法，对输入进行解释
 - 利用 `proc_macro` 提供的关于宏的各种 类型和方法，生成最终的代码
 - 利用调试手段，观察是否符合预期 并进行调整


我们在这里没有展开讲 关于`proc_macro` 库的使用，这个只能是在实际开发中，通过参考其他人的使用慢慢积累

我们更多时候可能使用的是下一个小节的内容 `声明式宏`

