# 函数

RUST中的函数基本上和其他语言是一样的。稍微有些不一样的地方在于，RUST由于
强类型和安全的原因，任何函数都需要明确返回值类型，即使一个没有返回的函数

### 表达式和语句 
再继续函数之前，先说明一下RUST中的表达式和语句的区别，C语言是没有这个特性的，但是一般
面向对象的语言常常会实现这个特性(由于闭包、语法糖的原因，这个后面再讲)

一个有左值 有右值的表达式几乎可以说就是一个语句；语句一般是不产生任何结果的；

```
// C语言中只有声明和语句
int main() {
	int a; //这是一个声明
	a = 10; //这是一个赋值语句 一般左边我们叫左值 右边叫右值
	int b = 11;
	int c = a> b ? c: a;   // 右边是一个三元表达式，左边是左值 
}
```

C语言不支持表达式独立作为一个语句存在，必须是一个完整的有右值赋值的表达式，才可以作为语句 

很多高级语言 为了支持闭包的特性，一般都支持表达式独立作为语句 


 - RUST 支持表达式独立成句，几乎可以表达式独立成句  等于一个 return 表达式
 - RUST 的每一条block 语句，隐含有返回一个 unit type(单元类型: ())
 
```
fn main() {
    let x = 5u32; // 这是一个语句

    let y = {
        let x_squared = x * x; //这是一个语句
        let x_cube = x_squared * x;

        // This expression will be assigned to `y`
        x_cube + x_squared + x //这是一个表达式，可以作为右值赋值给变量y
    };

    let z = {
        // The semicolon suppresses this expression and `()` is assigned to `z`
        2 * x; // 这是一个block 语句，隐含有block 的返回类型是()
    };

    println!("x is {:?}", x);
    println!("y is {:?}", y);
    println!("z is {:?}", z);
}
```

### 函数定义

 - 函数声明格式为: fn func_name(val1: type1, val2,type2)  -> re_Type ;
 - 函数声明中 类型必须要指定
 
练习1: 修复错误
```
fn main() {
    // Don't modify the following two lines!
    let (x, y) = (1, 2);
    let s = sum(x, y);

    assert_eq!(s, 3);

    println!("Success!");
}

fn sum(x, y: i32) {
    x + y;
}

```

### 无返回值函数

RUST中明确了无返值的函数的类型为 单元类型(unit),这里也解释了基元类型的unit type的使用和作用

```
fn main() {
    // Don't modify the following two lines!
    let (x, y) = (1, 2);
    let s = return_unit(x, y);

    assert_eq!((), s);

    println!("Success!");
}

fn return_unit(x: i32, y: i32) -> () {
    x + y;
}
```

### 不返回函数
有些函数可能是永远不会返回的，比如退出函数、panic函数，未实现的函数

 - RUST使用 **!** 关键字表示never type，也解释了基元类型的never type的使用和作用
 - 使用  panic! unimplemented! todo! 可以实现无返回类型的函数
 
```
// Solve it in two ways
// DON'T let `println!` works
fn main() {
    never_return();

    println!("Failed!");
}

fn never_return() -> ! {
    // Implement this function, don't modify the fn signatures
    // todo!()
	//  unimplemented!()
}
```
