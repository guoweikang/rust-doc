# 泛型

有了impl 的基础我们可以来看RUST 的泛型实现了，RUST中的泛型和其他面向对象语言 相比较 
最大特点是采用了 零抽象的原则 为什么说是零抽象？ 

泛型本质上就是在代码阶段 通过类型符号(严格意义上不是变量) 实现了类型占位符 

其实C里面也有实现泛型的类似方法,想想是什么？

```
#define  min(a,b) (a)<(b)? (a) : (b)
```

作为一名C语言开发工程师，之前我是很烦面向对象的，原因在于接触了java语言后，似乎程序员的工作从编写
代码变成了需要熟知各种类型的API，更重要的原因我总结如下

 - C语言中的API接口，只要你看到源码，可以清楚知道它在做什么 是怎么实现的
 - 面向对象中的API接口，你看到源码，会看到它又引用或者继承了更高一个层级的(一般叫父类型)的API，又因为
   这些代码中大量使用了更高级别的抽象语言，导致如果你没有把它的语法学明白，几乎无法读基础库的源码 

这形成了一个悖论，java 这种提高生产力的开发语言，目标是希望开发人员可以用极低的学习成本加速开发过程，
如果你想成为高深的java程序员，你需要把java的语言特性和语法学习明白(大部分人是不行的，因为本来就是上层应用开发人员) 

什么是类型系统？类型是对模型的抽象，这其实对我们并不遥远，即使是C语言开发人员 也知道 u8 是对0-255数字的抽象

### 类型泛型

知识点: 泛型是一个类型占位符，没有实例化(使用该泛型的解构、函数等)的泛型是没有意义的
RUST会在编译阶段，根据具体使用泛型的代码，完成泛型类型的替换，生成实际的类型、函数


练习1: 结构体泛型 

```
struct GenricA<T> (T); // 定义了一个元组结构体，包含类型T 

 // 定义了一个POINT 泛型结构体，包含类型T U  
 // 代码中并没有使用到这个类型，因此该类型不会被实例化
struct GenricPoint<T,U>  {
	x:T,
	y:U,
}


fn main() {
    //使用两种方法修复它
	let a: GenricA<i32> = GenricA(12_u32);
	let GenricA(x) = a; //解构元组结构体 
	
	println!("{}",x);
}

```

练习2: 函数泛型 

```
//先不用关注这里了 std::ops::Add 后面会讲 
fn add <T: std::ops::Add<Output = T>> (a:T, b:T) -> T {
	a+b
}

fn main() {
	let c = add(10_i32,20_i32);
	let d = add(10_f32,20_f32);
}

```

练习3: 方法实现泛型

- 在使用impl 为泛型类型实现方法时，必须要显示为impl声明泛型

```
struct GenricPoint<T,U>  {
	x:T,
	y:U,
}

//必须在impl 之后增加泛型声明
impl<T,U> GenricPoint<T,U> {

	//函数需要用到其他泛型，需要在函数继续声明泛型
	fn mix<V,W>(self: GenricPoint<T,U> , mixval: GenricPoint<V,W>) -> GenricPoint<T,W> {
		GenricPoint{
			x: self.x,
			y: mixval.y,
		}
	}
}


fn main() {
	let a : GenricPoint<i32,i32> = GenricPoint{x:10, y:10};
	let b : GenricPoint<f32,f32> = GenricPoint{x:1.0_f32, y:2.0_f32};
	let _c : GenricPoint<i32,f32> = a.mix(b);
	
    //上述代码在真正运行的时候，想的简单一点，会为 每个类型 实例化一个函数 
	//GenricPoint_mix(GenricPoint<i32,i32>,GenricPoint<f32,f32>)
	
}
```

### const值泛型

基于泛型类型本质上就是一个占位符，在编译阶段会被常量化，RUST也支持值的常量化 

```
//声明了一个N占位符，N 在使用的时候 必须是一个常量，或者是一个在编译阶段可以处理的常量表达式 
//由于它的静态性，该值初始化不应该依赖运行态 比如需要调用函数
struct Arr<T, const N: usize> ([T;N]);

fn main(){
	let _a: Arr<i32,12> = Arr([0;12]);
}

```