# 智能指针

上一个章节，我们已经见到了 原始指针 和 引用；也介绍了 Deref 和 Drop特征；

本节我们继续了解RUST其他智能指针类型

C语言，指针往往就是指针，不会存在这个指针和那个指针有什么区别，都是通过解引用的方式去使用指针；

指针所代表的值，由程序员自己去解释，你可以malloc 申请一个内存，然后把指针的值赋值给指针；你也可以
把变量的值 通过 `&a`的方式，把该指针值赋值给指针， 甚至可以 通过 `int *a = &10; int **b = &a` 把一个指针
地址在赋值给一个指针，这种行为在RUST中几乎就和原始指针一样 

```
#[derive(Debug)]
struct Apple(i32);

fn main() {
    let mut a = Apple(10);
    let mut a_ptr1  = &mut a as *mut Apple;
    
    let a_ptr_ptr = &mut a_ptr1 as *mut *mut Apple;
    
    println!("{:p}",&a); 
    println!("{:p}",a_ptr1);

    println!("{:p}",&a_ptr1);
    println!("{:p}",a_ptr_ptr);
    
    unsafe {
        println!("{:?}",*(*a_ptr_ptr));
        println!("{:?}",*a_ptr1);
    }
}
```

原始指针一般不会直接使用，之前我们也看到了，原始指针的解引用必须要在unsafe模式执行，RUST不会再负责
指针的安全

引用已经是我们看到过的一个智能指针类型，而且我们也看到了引用使用的严苛性，这种严苛性保证引用使用指针是安全的

RUST 还提供了其他几种类型的智能指针，用于再不同场合下指针的使用，同时保证了内存安全 


###Box<T>
Box智能指针几乎是最简单的智能指针了 Box智能指针主要实现: 

 - 实例是从堆上分配的(Box类型通过定义一个 Allocator内存分配器完成这件工作)
 - Box指针符合RUST对于堆上内存安全的要求(释放时机、使用安全)


Box通过实现Drop特征，可以自动释放内存，同时会调用内部类型的Drop 

```
struct Apple(i32);

impl Drop for Apple{

	fn drop(&mut self) {
		println!("drop apple");
	}
}

fn main() {
	let _apple = Box::new(Apple(10)); //等于堆上分配了一个初始化过的APPLE 结构体
	
} //当box变量所有权销毁，drop会释放堆上的内存 同时调用子类型的drop
``` 


Box通过实现Deref特征，使用它 就像使用一般的指针一样 

```
#[derive(Debug)]
struct Apple(i32);

fn main() {
	let mut apple_ptr = Box::new(Apple(10)); //等于堆上分配了一个初始化过的APPLE 结构体
	
	println!("{:?}",*apple_ptr);
	
	let apple_ptr_ref = &apple_ptr;
	
	println!("{:?}",*apple_ptr_ref); // 回顾deref的引用归一
	
	
	(*apple_ptr).0 = 100; // *(apple_ptr.derefmut()) 等于使用了 Apple的可变引用
	
	println!("{:?}",*apple_ptr); 
	
	println!("{:?}",*apple_ptr_ref); // 回顾引用的使用规则
}
``` 

Box通过实现Deref特征, 保证对于变量的访问 符合引用的使用规则

```
#[derive(Debug)]
struct Apple(i32);

fn main() {
	let mut apple_ptr = Box::new(Apple(10)); //等于堆上分配了一个初始化过的APPLE 结构体
	
	let apple_ref: &Apple = &apple_ptr;  // apple_ptr.deref() 等于使用了 Apple的 不可变引用
	
	let mut_apple_ref: &mut Apple = &mut apple_ptr;// apple_ptr.derefmut() 等于使用了 Apple的 可变引用
	
	 // println!("{:?}",apple_ref ); //回顾引用使用约束
	 
	 
	(*mut_apple_ref).0 =  100;
	println!("{:?}",mut_apple_ref );
}
```

解引用移动？复制

```
#[derive(Debug)]
struct Apple(i32);

fn main() {
	let mut apple_ptr = Box::new(Apple(10)); //等于堆上分配了一个初始化过的APPLE 结构体
	
	let apple2 = *apple_ptr; //解引用赋值 
	
	println!("{:?}",apple_ptr);
}

```
解引用移动？复制

```
fn main() {
	let mut int_ptr = Box::new(10); //等于堆上分配了一个初始化过的i32
	
	let int2 = *int_ptr; //解引用赋值 
	
	println!("{:?}",int_ptr);
}
```

总结: 移动还是赋值，取决于T 的类型 是否实现了copy



Box支持所有权移动语义 虽然BOX肯能会放在栈里 但是它包裹着一个堆上的内存

```
fn main() {
	let b = Box::new(10);
	
	let a = b;
	
	println!("{}",b);
}


```


### 智能指针其他特点

由于智能指针本身特点，一般会提供和其他指针的互转: 

 - Box可以使用 `into_raw`转为 原始指针，box被消耗(所有权被转移)
 - 由于原始指针不会被自动销毁，再使用完之后，需要再通过 `from_raw` 得到新的BOX，由box自动销毁内存
 
```
pub fn into_raw(b: Box<T, A>) -> *mut T

fn main() {
	let x = Box::new(String::from("Hello"));
	let ptr: *mut String = Box::into_raw(x);
	
	unsafe { println!("{}", *ptr) };
	
	//如果不把指针再塞回去 会造成 内存泄漏
	let _x = unsafe { Box::from_raw(ptr) };
}

```



### 练习Box实现简单的链表

链表支持 push pop,每一次都是再头部添加和挪出

```
struct List<T> {
	head: Option<Box<Node<T>>>,
} 

#[derive(Debug)]
struct Node<T> {
	data: T,
	next: Option<Box<Node<T>>>,
} 

impl<T: std::fmt::Debug> List<T> {
	fn new() -> List<T> {
		List{head: None}
	}
	
	fn push(&mut self, data:T) {
		let mut new_node = Node {data, next:None};
		if self.head.is_none() {
			//替换new node 为Head，head 挪动到newnode -> next 
			self.head = Some(Box::new(new_node))
		} else {
			//替换new node 为Head，head 挪动到newnode -> next 
			new_node.next = Some(self.head.take().unwrap());
			self.head = Some(Box::new(new_node))
		}
	}
	
	fn pop(&mut self) -> Option<T> {
        let head = self.head.take()?;
        let ans = Some(head.data);
        self.head = head.next;
        ans
	}
	
	
	fn show(&self) {
		let mut index = 0;
		let mut node_option_ref  = self.head.as_ref();
		loop {
		    if node_option_ref.is_some() {
		        let node_ref = node_option_ref.unwrap();
				println!("index[{}] : {:?}", index, node_ref.data);
				node_option_ref = node_ref.next.as_ref();
				index+=1;
				continue;
			}
			break;
		}
	}
}

#[derive(Debug)]
struct Apple(i32);


fn main() {
	let mut list = List::new();
	list.push(Apple(1));
	list.push(Apple(2));
	list.push(Apple(3));
	list.show();
	list.pop();
	list.show();
	
	list.pop();
	list.pop();
	list.show();
}

```

  






