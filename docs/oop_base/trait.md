# 特征


### 继承和动态类型
我们已经学习了 类型方法、以及泛型的概念，其实已经解决了很大一部分C语言的冗余代码的问题 

但其实我们知道面向对象语言里面最习惯使用的一个类型抽象叫做继承，继承固然有很多优点，继承解决了不同
关联类型的代码冗余，比如最经典的继承模型: 关于四腿兽、猫、狗的问题；

 - 继承允许类型中的数据类型继承，比如四脚兽一定有四条腿，那么猫狗继承四脚兽，也一定有四条腿 
 - 继承允许方法的继承，比如四脚兽都会叫，猫狗也都会叫 
 
RUST 遵循内存强安全的原则并且是向性能妥协的，因此没有实现继承 

 让我们假设一个java中很常见的场景，猫和狗都属于四脚兽 
 
 - 下面代码在cat 和 dog 分别重新实现了父类animal的 sound 方法
 - animal_sound 接受的参数是 animal父类型
 - 如果是常见的面向对象实现中，会通过利用叫做多态的特性实现对参数类型的判定 从而调用各自的方法实现
 - 多态的特性必然需要用到为每个类型保存一个各自的函数指针列表
 - 这样做的后果就是类型自身需要抽象，体现就是需要占用内存，并且在运行中动态判定，自然执行效率会下降 
 - 之前我们在学习泛型的时候知道了，rust 会把泛型实现为静态类型，这是因为RUST 遵循零抽象的原则，虽然代码
   写的时候是泛型，但是实际编译阶段会把这种动态抽象转换为 静态类型
 
 
```
//这是一段伪代码 
struct animal;
struct cat;
struct dog;

impl animal  {
	fn sound(self) {
		println!("i am a animal");
	}
}

impl dog  {
	fn sound(self) {
		println!("i am a dog");
	}
}

impl cat  {
	fn sound(self) {
		println!("i am a cat");
	}
}


fn animal_sound(struct animal) {
	animal.sound();
}

```

### 特征初探

我们已经知道，RUST 不支持继承，RUST采用了另外一种技术支持类似的功能: 特征 

特征是用来描述一组抽象接口的定义，特征本身也是一个类型，特征的元素就是一个函数指针列表 

下面是一个示例，假设我们需要实现一个MP3和一个MP4，很容易想到他们需要支持播放和暂停的功能，这种共性
可以通过 特征 抽象为一组接口的定义 


练习1：简单使用
```
trait Playable {
	//特征中的方法 可以是一个声明，实现特征的类型 必须要实现该方法
	//可以使用self关键字，fn play(self: &Self);因为特征必须要具体的类型实现才有意义 
	fn play(&self); 
	
	//特征中的方法 可以有一个默认实现 
	//特征中的方法 也可以是一个关联方法 必须通过 TYPE::pause 调用
	fn pause() {
		println!("Paused");
	}
}

struct MP3;
struct MP4;

impl Playable for MP3 {
	fn play(&self) {
		println!("Mp3 playing");
	}
}

impl Playable for MP4 {
	fn play(&self) {
		println!("Mp4 playing");
	}
	
	//特征中的方法 可以重写 
	fn pause() {
		println!("MP4 Paused");
	}
}

fn main() {
	let mp3 = MP3;
	let mp4 = MP4;
	mp3.play();
	MP3::pause();
	mp4.play();
	MP4::pause();
}
```

练习2：特征可以定义常量泛型


```
trait Number {
	const Max: usize;
}

struct U8;
struct U16;

impl Number for U8 {
	const Max: usize = 255;
}

impl Number for U16 {
	const  Max: usize = u16::MAX  as usize ;
}

fn main() {
}
```

### 特征继承

RUST 为特征实现了继承的能力，说是继承，实际上是一个单向的过程，RUST 会静态检查类型在实现某个特征的时候，是否
同时实现了该特征依赖的特征 

```
enum CarType {
	SUV,
	OffRoad,
}

trait Vehicle {
	const Price: u32;  
	fn get_price()-> u32 {
		Self::Price
	}
}

//表明类型在实现car的特征的同时 必须要实现 Vehicle
trait Car: Vehicle {
	fn car_type(&self) -> CarType;
}
struct TesilaModelY;
struct Jeep;

impl Car for TesilaModelY {
	fn car_type(&self) -> CarType {
		CarType::SUV
	}
}
impl Vehicle for TesilaModelY {
 	const Price: u32 = 100;  
}
impl Car for Jeep {
	fn car_type(&self) -> CarType {
		CarType::OffRoad
	}
}
impl Vehicle for Jeep {
 	const Price: u32 = 200;  
}

fn main() {
    println!("{}",Jeep::get_price());
    println!("{}",TesilaModelY::get_price()) 

}
```

### 特征的形式

我们在使用特征时，可以参考标准库中的用法，其实很多RUST 讲的高级特性，都是在RUST 标准库中实现的；

我们如果想实现自己的类型，并且希望能够做到像标准库的类型一样好用和安全，利用好标准库必不可少 


标记特征: 标记特征的形式 一般都是空的形式，存在的意义有两种 

 - 作为一个标记，表明实现该标记特征的类型 具备某种能力，类似打标签
 - 作为组合，标记特征也可以用来组合(继承)一些特征，作为某几种特征的更高级别的抽象 
 
泛型特征: 特征中也可以使用泛型 

```
trait CreateFrom<T> {
	fn create_from(value:T) -> Self;
}

struct Generic<T>(T); 

impl<T: std::fmt::Display> CreateFrom<T> for Generic<T> {
	fn create_from(value: T) -> Self {
		Generic(value)
	}
}


fn main(){
    let a8  = Generic::create_from(1_u8);
}
```
 
关联类型特征 : 和泛型特征类似，只是把类型作为特征的一个参数

- 在使用关联类型特征，需要使用  Self::type 引用该类型

```
trait CreateFrom {
	type In;
	fn create_from(value: Self::In) -> Self;
}

#[derive(Debug)]
struct Generic<T>(T); 

impl<T> CreateFrom for Generic<T> {
	//实现类型必须要初始化类型 In 
	type In = T;
	
	fn create_from(value:  Self::In) -> Self {
		Generic(value)
	}
}


fn main(){
    let a8  = Generic::create_from(1_u8);
    let a32  = Generic::create_from(10_i32);
    
    println!(" {:?}  {:?}",a8,a32);
}
```

