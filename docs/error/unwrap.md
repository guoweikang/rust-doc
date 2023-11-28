# 更多解包方法

我们已经知道，标准库提供了Result 和 Option 两个泛型枚举类型，也已经知道，枚举类型可以通过使用
`if let Some()/Err()/Ok() ` 或者是 `match` 语法对结果进行匹配处理，如果仅仅如此让我们看一个之前的代码


下面代码是我们开发中经常需要处理的情况，判断函数返回值，然后对不同返回情况做出不一样的处理

```
use std::path::Path;
use std::io::prelude::*;
use std::fs::File;


fn read_file_content(filename:&str) -> Result<String,String> {
	let path = Path::new(filename);
	let mut file = match File::open(&path) {
		Ok(file) => file,
		Err(err) => return Err(format!("{}",err)),
	};
	
	let mut s = String::new();
	match file.read_to_string(&mut s) {
		Ok(_) => Ok(s),
		Err(err) => Err(format!("{}",err)),
	}
}

fn main() {
    match read_file_content("a.txt") {
        Ok(content) => println!("read content: {}",content),
        Err(err) => println!("read content err: {} ",err),
    }
}
```

可以看到，对于 枚举类型的错误类型 处理起来有几个问题

 - 错误和值被包装了一层，实际在使用时，往往都是需要先解压在使用，而解压动作必须增加一个block 
 - 错误和正确的情况一般情况下都需要分开处理，由于错误处理的代码占据函数越多空间，则代码可读性越差； 

为了解决上述两个问题，RUST提供了更多的方法处理Option 和Result 


### map&map_err
map用于封装类型转换的函数

option的map 定义如下： 

```
pub fn map<U, F>(self, f: F) -> Option<U>
where
    F: FnOnce(T) -> U,
{
    match self {
        Some(x) => Some(f(x)),
        None => None,
    }
}
```
从map的实现上来开，有几个泛型需要注意: 

 - F:代表一个函数类型，该函数接收一个 泛型T，然后输出泛型U 
 - T：代表 Some包裹的类型
 - U: 代表函数输出的类型 

result的map 定义如下： 和option类似，只处理Ok的情况

```
pub fn map<U, F: FnOnce(T) -> U>(self, op: F) -> Result<U, E> {
    match self {
        Ok(t) => Ok(op(t)),
        Err(e) => Err(e),
    }
}
```

当然也还有针对Err情况的Map(仅针对Result类型) 

```
pub fn map_err<F, O: FnOnce(E) -> F>(self, op: O) -> Result<T, F> {
    match self {
        Ok(t) => Ok(t),
        Err(e) => Err(op(e)),
    }
}
```

那么 我们一开始的代码可以修改为: 

```
use std::path::Path;
use std::io::prelude::*;
use std::fs::File;

fn err_to_string(err: std::io::Error) -> String {
	format!("{}",err)
}

fn read_file_content(filename:&str) -> Result<String,String> {
	let path = Path::new(filename);
	let mut file = match File::open(&path).map_err(err_to_string) {
		Ok(file) => file,
		Err(err) => return Err(err),
	};
	
	let mut s = String::new();
	match file.read_to_string(&mut s).map_err(err_to_string) {
		Ok(_) => Ok(s),
		Err(err) => Err(err),
	}
}

fn main() {
    match read_file_content("a.txt") {
        Ok(content) => println!("read content: {}",content),
        Err(err) => println!("read content err: {} ",err),
    }
}
```


### map_or&map_or_else

`map_or`和 `map_or_else` 用于直接解压，并且对内部类型进行归一，需要传递一个默认值

我们知道 option 和 Result 如果解压 会有可能得到两个不同的类型，但是有时，我们可能会让程序正常执行；然后给错误类型一个
默认值，这两个方法用于完成这个动作，同时还支持传入闭包或者函数 对解压值进行转换

```
pub fn map_or<U, F>(self, default: U, f: F) -> U
where
    F: FnOnce(T) -> U,
{
    match self {
        Some(t) => f(t),
        None => default,
    }
}

pub fn map_or<U, F: FnOnce(T) -> U>(self, default: U, f: F) -> U {
    match self {
        Ok(t) => f(t),
        Err(_) => default,
    }
}


pub fn map_or_else<U, D, F>(self, default: D, f: F) -> U
where
    D: FnOnce() -> U,
    F: FnOnce(T) -> U,
{
    match self {
        Some(t) => f(t),
        None => default(),
    }
}

pub fn map_or_else<U, D: FnOnce(E) -> U, F: FnOnce(T) -> U>(self, default: D, f: F) -> U {
    match self {
        Ok(t) => f(t),
        Err(e) => default(e),
    }
}
```



### and&and_then&or&or_esle 
结果如果表示成功或者失败，利用 `and` 和 `or` 可以对结果进行类似 条件运算 比如  `true & false; true or false`

and 的定义和使用 Option类似 不做演示
```
pub fn and<U>(self, res: Result<U, E>) -> Result<U, E> {
    match self {
        Ok(_) => res,
        Err(e) => Err(e),
    }
}


let x: Result<u32, &str> = Ok(2);
let y: Result<&str, &str> = Err("late error");
assert_eq!(x.and(y), Err("late error")); // Ok 和 Err and 运算之后得到 Err

let x: Result<u32, &str> = Err("early error");
let y: Result<&str, &str> = Ok("foo");
assert_eq!(x.and(y), Err("early error")); // Err 和 OK运算之后得到Err


let x: Result<u32, &str> = Err("not a 2");
let y: Result<&str, &str> = Err("late error");
assert_eq!(x.and(y), Err("not a 2")); // Err 和Err 运算之后 得到的是 self的Err

let x: Result<u32, &str> = Ok(2);
let y: Result<&str, &str> = Ok("different result type");
assert_eq!(x.and(y), Ok("different result type")); // OK 和 Ok运算之后 得到的是 y的Ok
```


`and_then` 的定义和使用: `and_then` 支持在得到结果后 进一步对结果进行运算 增加了and运算的map

```
pub fn and_then<U, F: FnOnce(T) -> Result<U, E>>(self, op: F) -> Result<U, E> {
    match self {
        Ok(t) => op(t),
        Err(e) => Err(e),
    }
}

pub fn and_then<U, F>(self, f: F) -> Option<U>
where
    F: FnOnce(T) -> Option<U>,
{
    match self {
        Some(x) => f(x),
        None => None,
    }
}
```

### 其他方法
类型库还提供了 `unwarp/unwrap_or/unwrap_or_esle` 总之 当你觉得 Result 和Option造成了代码长度增加，请尝试去
标准库找找是否有对应的方法可以优化代码


### as_ref&as_mut&take
我们在上面看到的所有解包，请你仔细观察他的`self` 入参,Option不支持Copy语义 意味着 每次解包等于消耗了Option自身 

有时我们并不希望消耗掉Option  我们希望使用他的引用 比如下面的示例 

```
#[derive(Debug)]
struct Node {
	next: Option<Box<Node>>,
}


fn main() {
	let node1 = Node {
		next: None,
	};
	let node_head = Node {next: Some(Box::new(node1))};

	//print node_head next
	println!("next: {:?} ", node_head.next.unwrap());
	
	//what happended
	println!("next: {:?} ", node_head.next.expect("no content"));
}
```

有时我们也真的希望移动 Option的值 但是又不希望破环掉Option

```
#[derive(Debug)]
struct List {
	head: Option<Box<Node>>,
}

#[derive(Debug)]
struct Node {
	data: i32,
	next: Option<Box<Node>>,
}

impl List {
	fn new() -> List {
		List{head:None}
	}
	
	//模拟入栈
	fn push(&mut self, data: i32)  {
		let mut node = Node {next: None, data: data };
		match self.head {
			None => self.head = Some(Box::new(node)),
			Some(_) => {
				node.next = self.head.take();
				self.head = Some(Box::new(node));
			}
		}
	}
	
	fn show(&self) {
		let mut cur_ref = self.head.as_ref();
		loop {
			match cur_ref {
				Some(box_ref) =>  {
					println!("data:{}",box_ref.as_ref().data);
					cur_ref = box_ref.next.as_ref();
					continue;
				}
				None =>  break,
			}
		}
	}
	
	
}

fn main() {
	let mut li = List::new();
	li.push(3);
	li.push(2);
	li.push(1);

	li.show();
	li.show();
}
```


### ok&ok_or 
Option 可以通过`ok_or` `ok_or_else `方法 转为 Result 类型 

```
pub fn ok_or<E>(self, err: E) -> Result<T, E> {
    match self {
        Some(v) => Ok(v),
        None => Err(err),
    }
}

   pub fn ok_or_else<E, F>(self, err: F) -> Result<T, E>
    where
        F: FnOnce() -> E,
    {
        match self {
            Some(v) => Ok(v),
            None => Err(err()),
        }
    }

```
result 可以通过`ok` `err ` 方法 转为 Option 类型 

```
pub fn ok(self) -> Option<T> {
    match self {
        Ok(x) => Some(x),
        Err(_) => None,
    }
}
```



### 及早返回&`？`

及早返回有点像 java的异常透传，我们在解包时，如果返回正确则解包，否则直接把错误类型透传到上一层 提前返回

我们一开始的代码可以这样修改

```
use std::path::Path;
use std::io::prelude::*;
use std::fs::File;

fn err_to_string(err: std::io::Error) -> String {
	format!("{}",err)
}

fn read_file_content(filename:&str) -> Result<String,String> {
	let path = Path::new(filename);
	let mut file =  File::open(&path).map_err(err_to_string)?;
	
	let mut s = String::new();
	file.read_to_string(&mut s).map_err(err_to_string)?;
	Ok(s)
}

fn main() {
    match read_file_content("a.txt") {
        Ok(content) => println!("read content: {}",content),
        Err(err) => println!("read content err: {} ",err),
    }
}
```

