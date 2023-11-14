# 特征区间

让我们慢下来，先回忆一下目前已经学习过的RUST在面向对象的能力 

 - 通过 impl 在类型上 关联函数和方法: 单独使用本身没有什么意义，就和实现一个函数没有区别
 - 支持静态泛型: 函数和类型上支持静态泛型，可以解决一部分代码冗余，尤其在C语言里面(C里面解决这类问题的方法，经常通过void \*来实现)
 - 多种形式的特征支持: 特征和泛型组合，可以进一步抽象方法，同时可以作为一种类型特征，表达一个类型是否具有某种特征的能力


OK，我们现在有了特征的能力，也有了泛型的能力，可以写出面向对象的方法了吗? 


### 特征区间作用


```
trait AnimalSound {
	fn sound(&self);
}

struct Cat;
struct Dog;

impl AnimalSound for Cat {
	fn sound(&self) {
		println!("miao miao");
	}
}

impl AnimalSound for Dog {
	fn sound(&self) {
		println!("wang wang");
	}
}

//主人拥有一个宠物
struct Master<T>{pet:T}

impl<T> Master<T> {
	
	//主人可以命令宠物发声音
	fn command_sound(&self) {
		let Master{pet} =  self;
		//pet 类型是泛型T，泛型T可以是任何类型  你怎么知道它有没有实现sound方法？
		pet.sound();
	}
}


fn main(){
    let master = Master{pet: Cat };
    master.command_sound();
}
```

特征区间一个最主要作用 是可以约束 泛型，声明泛型具备某个特征的能力，只有这样，才能在泛型种使用该特征中的方法 

没有特征约束的泛型，几乎无法使用，因为根本无法在该泛型上调用任何方法 

### 特征区间声明

特征区间的使用

```
//在没有使用泛型时，可以声明类型为特征对象
trait AnimalSound {
	fn sound(&self);
}

struct Cat;
struct Dog;

impl AnimalSound for Cat {
	fn sound(&self) {
		println!("miao miao");
	}
}

impl AnimalSound for Dog {
	fn sound(&self) {
		println!("wang wang");
	}
}

fn sound(val: impl AnimalSound)  {
    val.sound();
}

fn main(){    
    sound(Cat);
	sound(Dog);
}
 
```

```
//主人拥有一个宠物
struct Master<T>{pet:T}
//在泛型后增加约束
impl<T: AnimalSound> Master<T> {
	
	//主人可以命令宠物发声音
	fn command_sound(&self) {
		let Master{pet} =  self;
		//pet 类型是泛型T，泛型T可以是任何类型  你怎么知道它有没有实现sound方法？
		pet.sound();
	}
}


fn main(){
    let master = Master{pet: Cat };
    master.command_sound();
}
```

```
//主人拥有一个宠物
struct Master<T>{pet:T}
//使用where子句
impl<T> Master<T>
where T: AnimalSound
{
	
	//主人可以命令宠物发声音
	fn command_sound(&self) {
		let Master{pet} =  self;
		//pet 类型是泛型T，泛型T可以是任何类型  你怎么知道它有没有实现sound方法？
		pet.sound();
	}
}


fn main(){
    let master = Master{pet: Cat };
    master.command_sound();
}
```


### 高级特点 
我们已经知道特征可以用来描述 类型应该具有哪些能力，也见到了多种语法，impl语法一般不常用，只会用在类型不太好描述的地方 
比如返回一个闭包,我们这里还没有讲过闭包，不过暂前先认为闭包是一个实现了多种 特征的一个类型

```
//这里先不用关心 闭包的语法，只需要知道 Fn(T) -> U 是一个特征 
fn lazy_adder(a:i32, b:i32) -> impl Fn()->u32 {
	move || a+b
}

fn main(){
	let add_later = lazy_adder(1,2);
	assereq!(3, add_later());
}

```



 



