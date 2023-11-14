# 关联函数&方法

这是我们看到RUST 面向对象的第一个小节，而且这一个章节 主要介绍最基础的一些内容，为后面的章节做一些准备

我们无法一次把RUST 关于某个部分的内容在一个章节里面讲清楚 因为这其中的内容有一些互相依赖


### impl和关联函数

我们已经学习过了函数，也学习过了struct和枚举类型，面向对象中最重要的一个思想就是 数据和操作数据的方法绑定 

impl 关键字用来完成这种绑定

 - impl 可以给某个类型实现某个方法，通过 类型::方法 语法调用该方法

```
#[derive(PartialEq,Debug)] //先不用管他
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    // Complete the new methd，return a Rectangle
    fn new(width:u32, height: u32) -> Rectangle  {
		Rectangle {
			width,
			height,
		}
	}	
}

fn main() {
    let rect1 = Rectangle { width: 30, height: 50 };
	let rect2 = Rectangle::new(30,50); // 关联函数调用 
    assert_eq!(rect1, rect2);

    println!("Success!");
}
```

### 方法
我们已经见过了关联函数，关联函数一般用于创建新对象，或者确实就是为某个类型实现一个方法，其实依然就是一个函数，只不过
作用域从全局作用域 变成了 类型作用域 

方法则更加符合面向对象的常见语法 

 - 类型中的方法，Self 关键字表示 当前调用者的类型 
 - 方法的第一个参数 必须是self 或者 &self 或者 &mut self，关于后两者 我们先可以认为是变量指针 先不用管他
 - 方法必须要有类型的实例作为参数，调用形式有两种:  
 - 方法和关联函数实现上 区别其实并不大 我们几乎可以认为两者的函数声明都类似 TrafficLight_new/TrafficLight_show_state

```

#[derive(PartialEq,Debug)] //先不用管他 马上就会用到
enum TrafficLight {
    Red,
	Green,
	Yellow,
}

impl TrafficLight {
    // Using `Self` to fill in the blank.
    pub fn show_state(self: &Self)  {
        println!("the current state is {:?}", self);
    }
    //写法2
    pub fn show_state2(self: &TrafficLight)  {
        println!("the current state is {:?}", self);
    }
    
    //写法3 省略类型
    pub fn show_state3(&self)  {
        println!("the current state is {:?}", self);
    }
    // Fill in the blank, DON'T use any variants of `Self`.
    pub fn switch_state(self: TrafficLight) -> TrafficLight {
        if self ==  TrafficLight::Green  {
			 TrafficLight::Yellow 
		} else if self ==  TrafficLight::Yellow  {
			 TrafficLight::Red 
		} else {
			TrafficLight::Green 
		}
    }
    
    // 写法3 省略类型
    pub fn switch_state2(self) -> TrafficLight {
        if self ==  TrafficLight::Green  {
			 TrafficLight::Yellow 
		} else if self ==  TrafficLight::Yellow  {
			 TrafficLight::Red 
		} else {
			TrafficLight::Green 
		}
    }
}
fn main() {
    let color = TrafficLight::Red;
    color.show_state();
    let color = color.switch_state();
    color.show_state2(); //通过实例调用
    let color = TrafficLight::switch_state2(color); // 通过类型::方法调用
    TrafficLight::show_state3(&color);// 通过类型::方法调用
    println!("Success!");
}
```

 

