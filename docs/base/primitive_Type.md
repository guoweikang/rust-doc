# 原始类型

### C语言的世界
原始类型这个概念应该很好理解，C里面的整型 、浮点型、char、指针，定长数组 

为什么叫原始类型？在C语言里面，类型的概念并不完备，C里面基本除了原始类型 其余我们见到的应该只有struct
这一种自定义类型了，原始类型的解释 一般归编译器解释，
**在编译阶段，决定应该给变量(总是有类型的) 分配的栈大小和存放位置**


### RUST原始类型

RUST的原始类型有: 

整型: 

 - i8/i16/i32/i64/i128： 有符号整数，数字表示该变量的bit数，下同
 - u8/u16/u32/u64/u128： 无符号整数
 - isize/usize: 非固定bit的整数，取决于平台(32位系统=i32/u32  64位系统=i64/u64)  
 
浮点型:

  - f32/f64: 符合IEEE 754标准的浮点类型 

布尔型: 

 - bool: 值有 false/true两种
 
数组:

 - [T,N] ：和C类似，指定一个类型，以及大小，数组是在栈上分配的 

元组类型: 

 - （A,B,C,D）：元组类型可以包含不同类型，数量不能超过10  

文本类型: 

  - char: 字符类型 一个4byte的内存单元(符合UTF-8标准的字符类型，关于UTF-8请自行查阅资料)
  - str: 一个加强版本的[u8]类型(切片类型 见下)

原始指针类型:  原始指针类型访问 不受RUST 安全机制管理 

 - *const T ：指向类型T的常量指针 不能修改内容 
 - *mut T:  指向类型T的可变指针 可以修改内容 

函数指针类型:
 
  - fn: 后面我们再讲闭包的时候会带一下 

目前为止，上面几种类型(除了str) 我们在其他语言中应该都见过了，这里的原始指针类型可能比较拗口，我们先理解为
普通指针就好(RUST 为了保证内存安全，对于指针严格管理 我们后续会讲)

  
切片类型: 

 - [T] : 和数组定义很像，但是没有大小，切片类型是一个**动态大小类型 DST ** 切片类型用来表示一段内存视图
         我们后续会讲关于动态大小的事情
		 
引用类型: 指针的一个变体
 
  - 共享引用: &T
  - 唯一直接(可变)引用: &mut T 

单元类型: 

 - (): 一个空类型，仅仅用来处理代表返回值是空的函数(比如一个空函数)

从不返回类型: 

 - !: 一个小类型， 仅仅用来处理函数永远不会返回的类型(比如一个panic/死循环函数)


### 理解库方法类型
大家一般在讲类型的时候，会给出一些示例代码，这些示例代码不可避免会用到一些方法，我们需要知道，类型和
该类型支持的方法是两个部分，后者一般是在 rust基础库中实现的，而类型是原生编译器的定义 

```
fn main() {
	/// i8::MAX 中的MAX 是在RUST core 库中为 类型定义的一个常量 
    assert_eq!(i8::MAX, __); 
    assert_eq!(u8::MAX, __); 

    println!("Success!");
	
	///下面是源代码的实现: 位于: rust-src/core/num/mod.rs 
	impl i8 {
    int_impl! {
        Self = i8,
        ActualT = i8,
        UnsignedT = u8,
        BITS = 8,
        BITS_MINUS_ONE = 7,
        Min = -128,
        Max = 127,
        rot = 2,
        rot_op = "-0x7e",
        rot_result = "0xa",
        swap_op = "0x12",
        swapped = "0x12",
        reversed = "0x48",
        le_bytes = "[0x12]",
        be_bytes = "[0x12]",
        to_xe_bytes_doc = "",
        from_xe_bytes_doc = "",
        bound_condition = "",
    }
}	
}
```

另外，基础库里面也实现了大量的类型: String Vec HashMap这些，我认为这些不在我的笔记内容范围之内 
这些类型的使用大部分可以通过查阅 [API 文档手册](https://doc.rust-lang.org/src/core/num/uint_macros.rs.html) 得到答案

本笔记更加关注的是背后的实现原理，而库中类型的实现，应该作为我们的一个代码参考，看RUST社区高手是如何
利用RUST特性开发的

### 目前的难点

在类型介绍中，我们可能已经遇到了从来没有见过的术语，而这些术语的出现，是由于RUST 设计原则以及为了保证
安全才诞生的，如果和其他语言都一样，那RUST就没有特殊性了 

我们会在后续的笔记中，逐步对里面的内容展开 相信你不会失望


### 数字变量的使用
再阅读本节之前，建议先阅读完下一章节 再回来看，本节会用到下一节的知识 

整性变量可以使用: 

 - 单位表达式 100_000
 - 可以使用 **_** 声明类型
 - 可以使用 0x:16进制 0o:八进制 0b:2进制
 
练习1: 类型

```
fn main() {
	// 标准的变脸声明: let variable: type = value;
	let a: i32 = 1;
	is_i32(a);
	// 数字类型支持通过 *_*显示声明类型 尝试修复它 
	let b: i32 = 2_u32;
	is_i32(b);
	// RUST支持默认类型推导，该推导发生在编译阶段，整性默认类型为i32 
	let c = 3;
	is_u32(c);
	let d = 4_u32; 
	is_i32(c);
	
	// 浮点类型默认类型是f64 尝试修复它
	let e = 1.0;
	is_f32(e);
	let f = 1.0_f64;
	is_f64(e);
}

fn is_i32(val: i32) -> () {}
fn is_u32(val: i32) -> () {}
fn is_f32(val: f32) -> () {}
```

练习2: 进制表达式 

```
// Modify `assert!` to make it work
fn main() {
    let v = 1_024 + 0xff + 0o77 + 0b1111_1111;
    assert!(v == 1579);

    println!("Success!");
}
```


### 字符类型的使用

 - 字符类型是UTF-8编码的，也就意味着代表的内存是4字节大小 
 - 变量需要使用 **''**  和C 一样
 
练习1: 关于内存大小 
```

// Make it work
use std::mem::size_of_val;
fn main() {
    let c1 = 'a';
    assert_eq!(size_of_val(&c1),1); 

    let c2 = '中';
    assert_eq!(size_of_val(&c2),3); 

    println!("Success!");
} 

```

练习2: 关于声明 
```

// Make it work
use std::mem::size_of_val;
fn main() {
    let c1 = 'a';
    assert_eq!(size_of_val(&c1),1); 

    let c2 = '中';
    assert_eq!(size_of_val(&c2),3); 

    println!("Success!");
} 

```

### 数组的使用
和C基本上是一样的

 - 数组类型声明为 [TYPE:SIZE] 
 - 可以使用 [val;size] 的方式一次性初始化数组内存 
 - 为了保证数组内存大小的计算， 数组元素类型必须是相同的，
 - 通过数组下标访问元素
 - 通过数组下标访问的元素 必须是要支持 copy 特征的(暂时先不用关注)


```
 fn main() {
    // Fill the blank
    let list: [char; 100] = ['a';100] ;

    assert!(list[0] == 'a');
    assert!(list.len() == 100);

    println!("Success!");
}
```
