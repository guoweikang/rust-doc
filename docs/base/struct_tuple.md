# 自定义类型

这里我们将学习两种最基础的组合类型 : struct 和 tuple 

struct 对于C语言开发同学一定不陌生了，元组类型对于python的开发同学一定也不陌生

struct 和 tuple 还是有一点类似的，struct 是有名的变量集合，tuple 更像是无名的struct(只可以使用一次)


### 元组

元组一般用来组合 一组不同类型，但是有关联性的变量，RUST 借鉴了其他语言的优点，支持元组类型 

 - 元组也是在栈上申请内存的，意味着一旦定义，其大小不允许在发生变化
 - 元组通过 t.0,t.1,t.2 下标访问
 - 最新rust版本支持元组的解构 以及定义一个变量元组，然后解构
 - 元组长度目前仅支持10个元素(考虑栈上内存有限)

练习1: 元组的声明和访问
 
```
fn main() {
    let _t0: (u8,i16) = (0, -1);
    // Tuples can be tuple's members
    let _t1: (u8, (i16, u32)) = (0, (-1, 1));
    // Fill the blanks to make the code work
    let t: (u8, __, i64, __) = (1u8, 2u16, 3i64);
    println!("Success!");
	
	let t = ("i", "am", "sunface");
    assert_eq!(t.1, "sunface");

    println!("Success!");
}

```

练习2: 使用元组解构初始化

```
fn main() {
    let tup = (1, 6.4, "hello");

    // Fill the blank to make the code work
    let () = tup;

    assert_eq!(x, 1);
    assert_eq!(y, "hello");
    assert_eq!(z, 6.4);

    println!("Success!");
}

```

练习3: 定义一个变量元组 稍后初始化
```
fn main() {

    let (x, y, z): (i32,i32,i32);

    // Fill the blank
    __ = (1, 2, 3);
    
    assert_eq!(x, 3);
    assert_eq!(y, 1);
    assert_eq!(z, 2);

    println!("Success!");
}

```


### 结构体
RUST 结构体基本上和其他语言类似 

 - struct 默认在栈上申请内存
 - RUST 支持空元素的结构体
 - RUST 支持匿名元素结构体，类似元组
 - RUST 支持 sturct 解构 
 - RUST 不允许 声明结构体中某个变量为可变，只允许整个结构体声明为可变
 
练习1: struct基本使用

```
// Fix the error 必须初始化所有字段
struct Score{
    math: i32,
    english: i32,
}
fn main() {
    let eng_score = 80;
	let math_score = 90;
    let p = Score {
        math: math ,
    };
    println!("Success!");
} 

```

练习2: 允许空的结构体 

```
struct Score;

fn main() {
    let emptr_score = Score; //空的结构体 内存为0
    println!("Success!");
} 
```

 
练习3: 允许元素匿名的结构体 

```
//匿名元素 结构体很像元组
struct RGB(u8,u8,u8);

fn main() {
    let color: RGB = RGB(0,0,255); //匿名结构体初始化类似元组
	let red = color.0; //通过下标访问匿名结构体 类似元组
	let RGB(red,green,black) = color; //支持结构体解构 
    println!("Success!");
} 
```

练习4: 支持有名结构体的解构 


```
//匿名元素 结构体很像元组
struct Score{
    math: i32,
    english: i32,
}
fn main() {
    let eng_score = 80;
	let math_score = 90;
    let p = Score {
        math: math_score ,
		english: eng_score,
    };
	let Score{math:x, english:y} = p; // 支持结构体解构 变量
	let Score{math, english} = p; // 支持结构体解构 变量

    println!("{}{}",x,y);
}
```


