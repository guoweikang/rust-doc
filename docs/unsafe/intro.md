# 不安全介绍

关于unsafe，我们之前的学习中，或多或少见过他的一些身影 本节我们更加全面的对他进行一个介绍

### 再谈安全

安全对于一门编程语言或许有很多种解释，包括不限于 内存安全、类型安全等

内存安全我们已经强调过很多次了，内存安全 可以保证程序不会写入 禁用的内存地址，比如空指针，也不会访问无效的内存，比如悬空指针
 
类型安全意味着程序不会允许用户为字符串分配数字，比如RUST 中的String，不会允许你写入非UTF-8编码的字符 

并发安全意味着程序再执行多个线程时 不会因为条件竞争而修改共享状态

一般而言，如果一门语言可以再自身提供所有这些层面的安全，就被认为是安全的。 我们再前面章节也已经看到了  RUST再安全模式下，确实可以做到这些， 


不安全的程序是指程序再运行时 破坏不变量或者触发未定义的行为，其中一些是程序员自身造成的，例如逻辑错误，而有一些，是由于语言本身又或者是编译器自身造成的

比如C/C++并没有规定不可以使用未初始化的变量 

```

int main(void) 
{
	bool var;
	if (var) {
		//do something
	} else {
		//do something
	}
}
```

程序此时的行为是未定义的，还比如访问数组越界，合理的处理应该是立即崩溃，但是C/C++对于这种行为也是允许的
 
 
虽然RUST 可以让程序免受一些主要的不安全因素影响，但是再某些情况下，即使RUST程序是安全的，也不能保证程序是正确的，尤其是逻辑错误

 - 使用浮点数，由于浮点数标准自身原因，可能出现精度的舍入误差，这种情况需要程序员自己处理
 - 不适用锁机制，访问共享数据
 - 死锁问题等

 
 
 
### 不安全模式

既然RUST 是一门安全的语言，而且做了如此多努力，但是某些情况下，RUST也不得不进入另外一个世界；

当RUST 需要与其他语言交互时，RUST 其实并不清楚其他语言的特性，比如类型系统，两种语言可能完全是不相同的，但是又不得不和其他语言发生交互

RUST提供了一个叫做不安全模式的概念，再不安全模式下，程序员可以额外获得一些功能(比如访问裸指针) 这些功能再C/C++中被称为未定义的行为
不过能力越大，责任越大，再RUST中使用不安全模式，安全的责任就落在了程序员身上，RUST不得不相信 程序员能够保证相关操作是安全的

当然，这种不安全特性，必须以一种受控的方式进行，RUST中 不安全的代码必须用`unsafe` 关键字标识，这样开发人员可以再阅读代码 一眼看到

除了需要和其他语言交互，再某些情况下，RUST也会用到不安全模式，例如: 将一个字节序`Vec<U8>`转为String类型，RUST 提供了String::from_utf8方法，
该方法会检查数组是否符合UTF-8编码，意味着一定会带来一些性能开销，但是作为程序员自身，如果你可以保证`Vec<U8>`一定是一个有效的UTF-8编码，也可以使用 
String::from_utf8_unchecked 这个不安全方法，减少性能开销，当然这种情况，有一个前提，程序员比编译器更了解某些细节的实现


最后，RUST会认为这些情况是不安全的: 

 - 更新静态可变变量
 - 解引用(访问)原始指针
 - 调用不安全的函数
 - 从联合类型中读取值
 - 再extern 代码块中调用某个声明的函数 --- 该函数来自其他语言

再上述情况中，编译器会放宽某些安全规则，但是借用检查依然生效； 为了让用户能够使用这些能力，RUST提供了 `unsafe`关键字，并且该关键字只能使用再规定的
场景

 - 函数和方法
 - 不安全的代码块 unsafe {}
 - 特征
 - 实现代码块 

### 不安全的函数和代码块

尝试编译和运行下列代码

```
fn get_value (i: *cosnt i32) -> i32 {
	*i
}

fn main() {
	let foo = &1013 as *const i32;
	let _bar = get_value(foo);
}
```

按照提示 修改后为: 

```
unsafe fn get_value (i: *cosnt i32) -> i32 {
	*i
}

fn main() {
	let foo = &1013 as *const i32;
	let _bar = unsafe {get_value(foo)};
}
```

这里用到了unsafe 的两种形式， unsafe标记不安全的函数，以及调用不安全函数，必须要标记为不安全的代码块

但是还有一种更为普遍的方法: 将不安全行为封装再API内部，从而外部调用不需要再标注为不安全代码块

```
fn get_value (i: *const i32) -> i32 {
	unsafe {*i}
}

fn main() {
	let foo = &1013 as *const i32;
	let _bar = get_value(foo);
	
	let _bar = get_value(4 as *const i32 ); //注意这里会发生什么

}
```

注意上面代码可能发生的情况

注意:虽然不安全的行为可以被封装在一个安全的API ，必须要符合一个条件: 该函数内部可以保证不会出现不安全的行为，该API会保证函数虽然使用了 不安全标记，但是行为是安全的



### 不安全的特征和实现

有些时候需要把特征标记为不安全，需要不安全的特征的原因并不是很明显。。。用处之一是，标记无法在线程之间共享的类型(Send Sync) 

不安全的特征 可以包含不安全的方法，也可以包含安全的方法(安全的特征，也可以包含不安全的方法)

```
struct Apple(i32);

unsafe trait UnsafeTrait {
	unsafe fn unsafe_func(&self);
	
	fn safe_func(&self) {
		println!("this is ok");
	}
}

trait SafeTrait {
	unsafe fn unsafe_func(&self);
}

unsafe impl UnsafeTrait for Apple {
	unsafe fn unsafe_func(&self) {
		println!("unsafe trait: unsafe_func");
	}
}

impl SafeTrait for Apple {
	unsafe fn unsafe_func(&self) {
		println!("safe trait: unsafe_func");
	}
}

fn main(){
	let app = Apple(32);
	
	app.safe_func();
	unsafe {UnsafeTrait::unsafe_func(&app) }
	unsafe {SafeTrait::unsafe_func(&app) }
}
```












 





