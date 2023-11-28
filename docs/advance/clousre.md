# 闭包

闭包在RUST 种被定义为一个类型，该类型实现了特定的 特征；

闭包也是一个表达式，关于表达式和语句 我们已经介绍过了 

闭包的使用类似于函数的调用，但是闭包可以捕获当前作用域内的变量




### 闭包基本语法

`(move) || expression`, 其中 `||`用来声明入参 

```
fn func<T, F: FnOnce(T)-> T,  > (val: T ,  f: F) -> T{
	f(val)
}

fn double_func<T: std::ops::Mul<i32, Output = T>>(val:T) -> T {
	val*2
}

fn main() {
	let double = |x| x * 2; //闭包可以作为变量绑定 

	let doubled = double(10); // 通过变量 调用闭包 
	
	let doubled = func(10, |x| x*2); //闭包作为参数传递

    println!("{}",doubled);
	
	let doubled = func(10, double_func); //函数作为参数传递
	
	println!("{}",doubled);
}


```

上述用法，基本上和函数使用是一样的，但是闭包最重要的功能是 闭包的语句能够捕获(使用)当前作用域内的变量 

```

fn create_error() -> Result<(),String> {
	Err("invalid param".to_string())
}

fn main() {
	let str_prefix = "main module: ";
	
	let result = create_error().map_err(|mut err|  format!("{} {}",str_prefix, err));
	
	println!("{:?}",result)
}

```

在传递给map_err的闭包函数种，我们定义了入参 `mut err`,但是我们也在表达式中用到了 `str_prefix`，该变量作用域是在当前main的作用域

如果使用函数实现类似功能，则无法像这样使用，必须要把 `str_prefix` 传递出去才可以 


### 闭包类型探讨

本小节我们尝试透过闭包类型，来看闭包是如何实现 作用域参数捕获的 

在前一个小节的例子中，当我们定义了闭包时，类似的会生成这样一个类型

```
struct Closure<'a> {
    s : &'a str,
}

impl<'a> FnOnce<(String)> for Closure<'a> {
    type Output = String;
    fn call_once(self, err: String) -> String {
        format!("{} {}",self.s, err)
    }
}

//这里类似于:let clousre  = Closure{s: str_prefix}; 
// create_error().map_err(clousre);
let result = create_error().map_err(|mut err|  format!("{} {}",str_prefix, err)); 


pub fn map_err<F, O: FnOnce(E) -> F>(self, op: O) -> Result<T, F> {
      match self {
          Ok(t) => Ok(t),
          Err(e) => Err(op(e)), //这里类似会发生 op.call_once(e);
      }
}
```

通过上述对于闭包的展开，我们知道 闭包对于作用域内变量使用，其实是在定义闭包类型是 初始化为闭包结构体的元素

 


