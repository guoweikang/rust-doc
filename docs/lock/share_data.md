# 数据共享

事实上，线程之间进行数据共享是有很多种形式的，熟悉C语言编程的，应该知道的有

 - 跨进程的 IPC 通信: socket , pipe, sharemem
 - 进程内的共享变量访问，必须要增加锁的保护， mutex,semphore, 读写锁等等等
 
内核态的数据共享应该不涉及到IPC的通信(都在一个地址空间) 因此只会涉及到关于数据的原子访问 常见有

 - atomic 原子变量 
 - spinlock保护的共享变量
 - 其他锁保护的共享变量(太多了)
 
我们要看一下RUST 是如何定义数据共享的

### 多所有权模型

RC我们已经很熟悉了，RUST 提供的一个模型 就是使用多所有权模型 引用计数指针 

由于Rc的引用计数更新不是原子的，因此只能适用于单线程模型，多线程模型 提供了他的孪生模型 Arc,

Arc保证了原子计数的更新是原子的

回到上一个小节的例子

```
use std::thread;
use std::sync::Arc;

fn main() {
	let nums = Arc::new(vec![1,2,3,4]);

	for n in 0..4 {
		let ns = Arc::clone(&nums);
		let child = thread::spawn( move || {
			println!("Thread num: {}",  ns[n]);
		});
		child.join().expect("Failed join child thread");
	}
	println!("main exit");
}
```
 
### 共享数据的保护

回忆引用计数指针的问题，我们之前讲过 如果希望修改引用计数内的类型，需要通过Refcell
```
use std::thread;
use std::sync::Arc;
use std::cell::RefCell;

fn main() {
	let nums = Arc::new(vec![1,2,3,4]);

	for n in 0..4 {
		let ns = Arc::clone(&nums);
		let child = thread::spawn( move || {
			println!("Thread num: {}",  ns.get(n).unwrap());
			ns.clear();
		});
		child.join().expect("Failed join child thread");
	}
	println!("main exit");
}
```

我们可以看到上面代码报错了，提示无法修改Arc，请回答一下为什么 

是否可以利用RefCell修改他的值? 试试看 为什么 


### 互斥

RUST 提供了可以互斥的类型 保证共享变量的访问，当然实际上互斥类型可以有非常多种 根据锁的类型不同

我们这里简单介绍一种 Mutex

```
use std::sync::Mutex;
use std::thread;


fn main() {
	let nums = Mutex::new(vec![1,2,3,4]);

	let child = thread::spawn( move || {
		println!("Thread num: {}",  nums.lock().unwrap().get(0).unwrap());
	});
	child.join().expect("Failed join child thread");
	println!("main exit");
}

```
Mutex的变量必须要先通过lock()方法 上锁成功以后才可以访问

### 共享可变性 

有了Mutex的基础，再加上Arc 可以实现一个数据再多个所有权下的访问 

```
use std::thread;
use std::sync::Arc;
use std::sync::Mutex;

fn main() {
	let nums = Arc::new(Mutex::new(vec![]));

	for n in 0..4 {
		let ns = nums.clone();
		let child = thread::spawn( move || {
		    let mut v = ns.lock().unwrap();// lock 
		    v.push(n);
			println!("Thread num: {:?}",  v);
			// v lifetime end， will unlock self
		});
		child.join().expect("Failed join child thread");
	}
	println!("main exit");
}
```

### 通过消息传递进行通信 

除了之前讲过的，利用多所有权 和 互斥机制 实现变量可以共享访问之外，RUST 还提供了叫做消息传递的通信机制

利用了标准库中的 `std::sync::mpsc` 提供了一个无锁定的多生产者，单订阅者队列，作为彼此通信的共享消息队列


 - channel: 一种异步的无限缓冲通道
 - sync_channel : 同步的有界 缓冲通道
 
 
```
use std::thread;
use std::sync::mpsc::channel;

fn main() {
	
	let(tx,rx) = channel();
	
	let child = thread::spawn( move || {
		while  let Ok(n) = rx.recv() {
			println!("recived: {}",n);
			if n == 9 {
				break;
			}
 		}
	});
	
	for i in 0..10 {
		tx.send(i).unwrap();
	}
	child.join().expect("Failed join child thread");
	
	println!("main exit");
}
```

关于异步和同步: 
 
 - 异步通道，send不会阻塞(如果消费者速度小于生产者，内存可能耗尽)；同步通道，大小有限，可以阻塞

关于多生产者和单一消费者: 

 - tx 支持Copy语义，可以被复制多份，也就可以被多个线程共同持有 
 - rx 不支持Copy，也就是所有权唯一，只能一个线程接收消息
 











