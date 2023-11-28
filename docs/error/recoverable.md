# 可恢复异常

RUST中的大多数错误处理 都是通过 Option Result 类型或者是该类型的扩展实现的；

该类型本质上是通过RUST 对于Enum基元类型，可以包装 内部类型的能力实现的 

```
pub enum Option<T> {
    None,
    Some(T),
}

pub enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

两种类型 都是std 标准库提供的自定义类型 并非RUST 内置类型

### 简单的enmum匹配

先熟悉一些 枚举类型的基本匹配方法 ，option 和 Result 同样都适用 

```
#[derive(Debug)]
enum Color {
	RED,
	GREEN,
}

#[derive(Debug)]
struct Apple(Color);

fn main() {
	let color1 = Color::GREEN;
	//match 匹配所有可能值 
	let a = match color1 {
		Color::RED => {
			println!("color is RED");
			Apple(Color::RED)
		},
		Color::GREEN => {
			println!("color is GREEN");
			Apple(Color::GREEN)
		},
	};
	println!("{:?}",a);
	
	//if let 可以单个匹配 
	let color1 = Color::RED;

	if let Color::RED = color1 {
			println!("color is RED");
	}
}
```


### Option 
option 是一个枚举类型，该类型由两个 变体: None(不带值) Some(附带一个泛型)

```
pub enum Option<T> {
    None,
    Some(T),
}
```

最简单的使用: 说明错误原因

```
fn main() {
	let mut op = Option::None;
	op = Option::Some(10);
	
	let mut op =  Option::None;
}
```

None 和Some 已经被rust 通过prelude 导入
```
fn main() {
	let mut op = None;
	op = Some(10);
}
```

应用1：大量用于返回值类型

```
#[derive(Debug)]
struct Dog {
	name: String,
}

struct DogHome {
	dogs: Vec<Dog>,
}

impl DogHome {
	fn put_dog(&mut self, name: &str) {
        self.dogs.push(Dog{name: name.to_string()});
	}
	
	fn get_dog(&mut self, name: &str) -> Option<&Dog> {
		for dog in &self.dogs {
			if dog.name == name {
                    return Some(dog);
			}
		}
		None
	}
}

fn main(){
	let mut doghome =DogHome{dogs: Vec::new()};
	doghome.put_dog("john");

	let john = doghome.get_dog("john");
	println!("{:?}",john);
	
	let jack = doghome.get_dog("jack");
	println!("{:?}",jack);
}
```

应用2：用于结构体中，之前的链表实验我们已经见到过了 


Option除了适用enum的match 和 if let 还提供了unwrap和except 方法，区别是如果值为None，会触发panic

一般情况下，会认为panic的解压不安全，在以下两种情况下建议使用: 

 - 明确了值不可能为None
 - 确实希望产生panic
  
```
#[derive(Debug)]
struct Dog {
	name: String,
}

struct DogHome {
	dogs: Vec<Dog>,
}

impl DogHome {
	fn put_dog(&mut self, name: &str) {
        self.dogs.push(Dog{name: name.to_string()});
	}
	
	fn get_dog(&mut self, name: &str) -> Option<&Dog> {
		for dog in &self.dogs {
			if dog.name == name {
                    return Some(dog);
			}
		}
		None
	}
}

fn main(){
	let mut doghome =DogHome{dogs: Vec::new()};
	doghome.put_dog("john");

	let john = doghome.get_dog("john");
	println!("{:?}",john.unwrap());
	
	let jack = doghome.get_dog("jack");
	println!("{:?}",jack.expect("jack is none"));
}
```

###Result 

RESULT 也是一个枚举类型，和option 类似 该类型也由两个 变体: OK(附带一个泛型) Err(附带一个泛型) 构成

```
pub enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

从变体名称上可以直观看到，OK和Err 分别表示成功和失败的两种情况 该类型相比Option 从使用场景更加固定一些，那就是函数的返回值 

说明代码问题
```
fn main() {
	let result = Ok(32);
	let result2 = Err("invalid param");
}
```

很多标准库提供的返回值 都是result 类型  说明下面代码问题原因

```
use std::path::Path;

fn main() {
	let path = Path::new("data.txt");
	let file = File::open(&path);
	let mut s = String::new();
	file.read_to_string(&mut s);
	
	println!("Message : {}",s);
}

```

第一次修改
```
use std::path::Path;

fn main() {
	let path = Path::new("data.txt");
	let file = match File::open(&path) {
		Ok(file) => file,
		Err(err) => panic!("Error opening file: {}",err),
	};
	
	let mut s = String::new();
	let _ = file.read_to_string(&mut s); //主动忽略掉错误处理
	
	println!("Message : {}",s);
}
```
