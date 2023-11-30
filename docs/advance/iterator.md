# 迭代器

我们在讲代码控制流那章节时，没有讲for 的使用，因为RUST 对于for 是 作为语法糖解释的 其中会用到迭代器特征



### 迭代器特征

```
pub trait Iterator {
    type Item;
	fn next(&mut self) -> Option<Self::Item>;
	
	//省略了其他大量的默认方法
}
```

直观看这个特征的定义，该特征可以从一个 实现了该特征的类型上，通过`next` 的方法, 得到一个 `Item`自定义类型的值 

下面是基于迭代器上面，对于`for`的进一步说明

```
struct Collec {
    start: u32,
    end:u32,
    current:u32,
}

impl Iterator for Collec {
    type Item = u32;
    fn next(&mut self) -> Option<Self::Item> {
        if self.current <  self.start {
            self.current = self.start;
            Some(self.current)
        } else if self.current < self.end {
            self.current+=1;
            Some(self.current)
        } else {
           return None;
        }
        
        
    }

}

impl Collec {
    fn new(start:u32, end:u32) -> Collec {
         Collec {start, end, current:0}
    }
}

fn main() {
    let c = Collec::new(1, 20);

    for i in c {
        println!("{}",i);
    }
}

```

如果 遍历一个迭代器，RUST 会对`for` 进行展开

```
fn main() {
    let c = Collec::new(1, 20);

	while let Some(i) = c.next() {
		 println!("{}",i);
	}
}
```

### 类型和迭代器的转换

上面我们举得例子，Collec 自身就是一个迭代器，实际编程中，我们更多会对希望迭代的对象 生成一个新的迭代器类型 

一般约定，从一个类型到 迭代器类型，原类型可以通过三种方法实现

 - iter(&self): 该方法通过原类型的**不可变引用**，返回一个迭代器，迭代器得到的是内部元素的引用
 - iter_mut(&mut self): 该方法，可以获取内部元素的可变引用 
 - into_iter(self): 该方法可以获得内部值的元素所有权，同时会消耗掉原类型 原类型不可以使用

```
pub trait IntoIterator {
    type Item;
    type IntoIter: Iterator<Item = Self::Item>;

    // Required method
    fn into_iter(self) -> Self::IntoIter;
}
```

如果for 语法糖中的类型实现了`IntoIterator`, RUST 会尝试对语法糖解包

```

fn main() {
	for i in 0..20 {
		println!("{}",i);
	}
	//0..20 其实是一个语法糖，实际是定义了一个Range类型 该类型还有很多其他变体
	assert_eq!((0..20), std::ops::Range { start: 0, end: 20 })
}

fn main() {
	let mut ranage = std::ops::Range { start: 0, end: 20 };
	{
		let result = match IntoIterator::into_iter(ranage) {
			mut iter => loop {
				let next;
				match iter.next() {
					Some(val) => next = val,
					None => break,
				};
				let i = next;
				let () = { println!("{i}"); };
			},
		};
		result
	}
}

```










