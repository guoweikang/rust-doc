# 枚举类型

枚举类型对于C或者其他开发人员一定也不陌生了，RUST 对于枚举类型 做了更多加强，让我们一起进入enum

### 最简单的枚举

RUST中的枚举类型，和C有一个非常类似的点，隐含有一个 **判别值** 

```
enum Animal {
    Dog,
    Cat,
}

fn main() {
	let mut a: Animal = Animal::Dog;
    println!("{}",a as isize ); //这里使用到了类型转换 
	a = Animal::Cat;
    println!("{}",a as isize );
}

```

RUST 枚举 和C 类似，会默认从0给 每个枚举变体(枚举实例: Animal 是类型，Animal::Dog 是变体) 编号

也和C一样，可以修改枚举默认判别值

练习: 

```

enum Foo {
    Bar,            // 0
    Baz = 123,      // 123
    Quux,           // 124
}

fn main() {
	let baz_discriminant = Foo::Baz as u32;
	assert_eq!(baz_discriminant, 123);
}

```

枚举的标识号默认大小为 isize, 可以通过 ** #[repr(u8)] ** 主动降低标识号内存

```
#[repr(u8)]
enum OverflowingDiscriminantError {
    Max = 255,
    MaxPlusOne // 应该是256，但枚举溢出了
}

#[repr(u8)]
enum OverflowingDiscriminantError2 {
    MaxMinusOne = 254, // 254
    Max,               // 255
    MaxPlusOne         // 应该是256，但枚举溢出了。
}
```
### 复杂类型枚举

RUST 允许枚举变体携带变量，基本形式为 变体名(T)  有以下注意点
 
 - 枚举变体所占内存由 枚举变体变量自身所占内存 加上 变体标识号 
 - 如果使用了复杂枚举变体，则无法直接获变体标识号

```

enum Animal {
    Dog(i32,f64), // Dog 携带了 一个无名元组变体，
    Cat {lenth: i32, weight: f64 }, // cat 携带了一个类结构体变体
} 

fn main() {
	let mut a: Animal = Animal::Dog(10, 37.2);
	a = Animal::Cat { lenth: 20, weight: 2.7 };
}

```




