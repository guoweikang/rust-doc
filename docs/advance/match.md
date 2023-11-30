# 高级匹配

`match` 关键字我们已经在Option Result 以及 枚举类型中见到过它的基本使用方法，类似于C里面的switch case，

但是`match` 在RUST中还有更加高级的用法



### 模式

要理解match 首先需要理解模式

模式：模式基于 给定数据结构去匹配值 并 **解构**，并可选地将变量和这些结构中匹配到的值绑定起来。
模式也用在变量声明上和函数（包括闭包）的参数上

下面示例中的模式完成四件事：


 - 测试 person 是否在其 car字段中填充了内容。
 - 测试 person 的 age字段（的值）是否在 13 到 19 之间，并将其值绑定到给定的变量 person_age 上。
 - 将对 name字段的引用绑定到给定变量 person_name 上。
 - 忽略 person 的其余字段。其余字段可以有任何值，并且不会绑定到任何变量上。

```
if let
    Person {
        car: Some(_),
        age: person_age @ 13..=19,
        name: ref person_name,
        ..
    } = person
{
    println!("{} has a car and is {} years old.", person_name, person_age);
}
```

模式可以被使用在 

 - let 声明
 - 函数和闭包的参数
 - 匹配(match)表达式
 - if let表达式
 - while let表达式
 - for表达式 
  
### 模式的解构

模式可以解构 结构体(struct)、枚举(enum)和元组，
占位符(`_`) 代表一个数据字段，而通配符 `..` 代表特定变量/变体(variant)的所有剩余字段

解构的应用,这部分我们在讲解结构体和元组已经讲解过了 

```
enum Color {
	RED(i32),
	BLUE(i32),
}

fn main() {
	let a = (10,20,30,40,50);
	
	let (x,y,_,..) = a; // let 可以使用模式 对元组解构  _忽略第三个数据，..忽略剩余的其他字段
	
	println!("{} {} ",x,y);
	
	let red = Color::RED(100);
	
	if let Color::RED(x) = red { // if let 对enum 解构
	    println!("{}",x);
	}
	
    if let (x,y,_,..) = a { // if let 对元组解构
        println!("{} {} ",x,y);
    }
}
```

解构是模式的基本使用方法，能实现功能比较单一 一般都要继续结合**值匹配**使用



### 模式的匹配

可反驳性：当一个模式有可能和它匹配的值不相同时，我们说这个模式具备  **可反驳性**  

不可反驳性：当一个模式总是和它匹配的值相同时，我们说这个模式具备  **不可反驳性**  

在我们前一个小节的例子中中，请说明哪个是可反驳 哪个是不可反驳的

```
let (x, y) = (1, 2);               // "(x, y)" 是一个不可反驳型模式

if let (a, 3) = (1, 2) {           // "(a, 3)" 是可反驳型的, 将不会匹配
    panic!("Shouldn't reach here");
} else if let (a, 4) = (3, 4) {    // "(a, 4)" 是可反驳型的, 将会匹配
    println!("Matched ({}, 4)", a);
}
```

没有条件的解构，一般都是不可反驳的，因为能匹配一切，带有条件的模式匹配，一般都是具备不可反驳性

接下来 我们讲从最简单的匹配开始讲起 


### 字面量模式

字面量是最简单的匹配, 支持: 数字、bool、char、string 

`@` 表示变量绑定，如果匹配成功，匹配到的值会绑定在 变量上 

`_` 是通配符匹配，表示可以匹配任何值

```
fn main() {
	let x = 10_i32;
	
	if let y @ 10 = x {
		println!("y is 10 : {}",y);
	}
	
	if let y @ 20 = x {
		println!("y is 20 : {}",y);
	}
	
	if let _ @ 10 = x {
		println!("y is 10");
	}
	
	let y = "abc";
	
	match y {
		"123" => println!("this is 123"),
		 z @ "abc" => println!("this is abc {}",z),
		 _  => println!("this is other "), 
	}
}
```

### 标识符模式

标识符模式将它们匹配的值绑定到一个变量上。此标识符在该模式中必须是唯一的。
该变量会在作用域中遮蔽任何同名的变量。这种绑定的作用域取决于使用模式的上下文。

在上面我们已经见到了，标识符模式主要用于变量的绑定，`@` `mut` `ref` 都是标识符绑定

```
fn main() {
	let mut a = 10; // mut 是一个标识符模式，标识绑定变量a的时候，使用可变模式
	let ref a_ref = a; // ref 是一个标识符模式，标识绑定变量 a_ref 的时候，使用的是a的引用 等于 let a_ref= &a;
	
	if let y @ 10 = a {  // @是一个标识符模式，标识把匹配到的值 绑定在 一个变量上 
		println!("a is 10 : {}",y);
	}
}
```

在执行变量绑定时，非ref 模式下，会尝试把copy 原有变量的值，当然如果原有变量没有实现 Copy 则会执行移动语义


```
#[derive(Debug)]
struct Apple(i32);

fn main() {
	let app = Some(Apple(10));
	
	match app {
		Some(a) => println!("move apple, apple not use"),
		None=> (),
	}
	//println!("{:?}",app);//what will happen
	 
	let app = Some(Apple(10));
	
	match app {
		Some(ref a) => println!("borrow apple, apple can use"),
		None=> (),
	}
	println!("{:?}",app);//what will happen

	let mut app = Some(Apple(10));
	
	match app {
		Some(ref mut a) => a.0 = 100,
		None=> (),
	}
	println!("{:?}",app);//what will happen
}

```

`@` 绑定模式一般用于解构的同时 同时增加约束 希望绑定内部变量 

```
#[derive(Debug)]
struct Apple(i32);

fn main() {
	let app = Some(Apple(20));
	
	match app {
		Some(Apple(size @ 10)) =>  println!("apple size is 10 {}",size),
		Some(Apple(size)) => println!("apple size is {}",size),
		None=> (),
	}
}
```

### 引用模式
引用模式可以实现对引用的解引用  关键字为 `@@@@`

```
#[derive(Debug)]
struct Apple(i32);

fn main() {
	let app = Apple(20);
	
	match &app {
		&Apple(size @ 20) =>  println!("apple size is 20 {}",size),
		_ => (),
		&apple => println!("apple is {:?}",apple),
	}
}

``` 

### match匹配

在我们学习完模式的使用之后 在回头来看match，基本上就能够对match的使用有一个基本的认识了

match guard： 模式守卫允许在match 完成模式匹配之后 进一步改进匹配标准。 
模式守卫出现在模式的后面，由关键字 if 后面的布尔类型表达式组成。

```
let message = match maybe_digit {
    Some(x) if x < 10 => process_digit(x),
    Some(x) => process_other(x),
    None => panic!(),
};
```



### 练习

定义一个消息类型格式 如下 
```
#[derive(Debug)]
enum Message {
    Quit,
    Move {x: i32, y: i32 },
    Write(String),
}
```

需求为: 

 - 如果消息退出，打印 `quit`
 - 如果消息为移动：如果 `x = 0 y = 0 ` 打印 `don't move` 
 - 如果消息为移动：如果 `x > 0 y = 0`  打印 `move right` 
 - 如果消息为移动：如果 `x < 0 y = 0`  打印 `move left` 
 - 如果消息为移动：如果 `x = 0 y > 0`  打印 `move up` 
 - 如果消息为移动：如果 `x = 0 y < 0`, 打印`move down`
 - 如果消息为移动：不符合上述条件 打印坐标
 - 如果消息为Write：打印内容(需要保证Message在 匹配完以后 依然可以使用)
 


```
#[derive(Debug)]
enum Message {
    Quit,
    Move {x: i32, y: i32 },
    Write(String),
}

fn match_msg(message: &Message) {
}

fn main() {
    let msg = Message::Quit;
	match_msg(&msg);
	
	let msg = Message::Move(0,0);
	match_msg(&msg);

	let msg = Message::Write("hello world".into());
	
	match_msg(&msg);
	
	println!("{}",msg);
}
```




