# 线程安全

本节更多探讨RUST 关于线程安全的设计 


### 线程安全

线程安全，指多个线程再竞争访问同一个数据内容时，不会出现数据损坏从而引发意外行为 

RUST只对数据安全执行了保护，并不会防止死锁问题

```
#![feature(mutex_unlock)]


use std::thread;
use std::sync::Arc;
use std::sync::Mutex;

fn main() {

	let a = Arc::new(Mutex::new(10));
	
	let c = a.lock().unwrap();// a is locked
	let a1 = a.clone();
	let child = thread::spawn( move || {
		println!("{}",a1.lock().unwrap()) ;//try to lock a, will failed ，unless a release lock
	});
	

    Mutex::unlock(c);// call	drop(c); 
	child.join().expect("Failed join child thread");
	println!("main exit");
}
```

那么RUST是如何保证 数据的共享和正确访问呢？

线程创建的时候，会约束闭包使用变量的类型 

```
pub fn spawn<F, T>(f: F) -> JoinHandle<T>
where
    F: FnOnce() -> T + Send + 'static,
    T: Send + 'static,
```

可以看到，F 和 T 类型都被约束为 必须要支持Send 特征；如果闭包中使用了不支持Send特征的类型，编译不会通过


### Send 特征
通过Send特征标记 一个数据类型是否可以被 发送(移动) 到线程中 表明该类型是一个移动类型

支持Send特征的类型，因为所有权再发送给线程之后 可以转移，因此是安全的 
```
pub unsafe auto trait Send { }
```
可以看到该特征是一个 auto 类型，因此类型默认都会默认实现该特征，同时 unsafe 表明了该特征是不安全特征
需要开发人员明确自定义类型是支持线程同步访问的 

RC被明确的声明为 不支持Send 特征，因此Rc指针不可以被发送给线程
```
impl<T: ?Sized, A: Allocator> !Send for Rc<T, A> {}
```

指针也是不支持Send特征的类型 

```
use std::thread;

const A:i32 = 10;
fn main() {

	let a = 10;
	let b = &a as *const i32;
	let child = thread::spawn( move || {
		println!("{}", *b) ;
	});
	child.join().expect("Failed join child thread");
	println!("main exit");
}
```

引用也无法传递(借用声明周期约束) 

```
#![feature(negative_impls)]

use std::thread;

#[derive(Debug)]
struct Apple;

impl !Sync for Apple {}

fn main() {

	let a = Apple;
	let b = &a;
	let child = thread::spawn( move || {
		println!("{:?}", *b) ;
	});
	child.join().expect("Failed join child thread");
	println!("main exit");
}
```


### Sync 特征

Sync特征的存在，主要是解决数据共享类型的，Sync标识的是所有权转移 而Sync标识的是同一个内存可以再不同
的线程访问

 - 如果类型支持Sync 标记，那么该类型的 不可变引用 支持Send标记
 - 如果类型支持Send，那么该类型的可变引用 也支持send 
 - 如果类型支持Sync 那么该类型的引用和不可变引用 支持Send

```
// 不可变引用 &String，String 是 Sync，因此 &String 是 Send

fn main() {
    let s = String::from("Hello");
    let reference = &s;

    std::thread::spawn(move || {
        println!("{}", reference);
    }).join().unwrap();
}


// 可变引用 &mut Vec<i32>，Vec<i32> 是 Send，因此 &mut Vec<i32> 是 Send
fn main() {
    let mut vec = vec![1, 2, 3];
    let reference = &mut vec;

    std::thread::spawn(move || {
        reference.push(4);
        println!("{:?}", reference);
    }).join().unwrap();
}

#[derive(Debug)]
struct Apple;

const DATA :Apple = Apple;

fn main() {
    let refd:&'static Apple = &DATA;
    std::thread::spawn(move || {
        println!("captured {:?} by value", *refd)
    }).join().unwrap();
    
    println!("{:?}",DATA);
}
```







