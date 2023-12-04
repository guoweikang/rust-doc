# C语言交互

rust在和外部语言交互，有两种情况

 - RUST 调用C语言接口
 - C调用 Rust 接口 
 
我们将分别针对上述两种情况进行说明


### 类型差异

先要清楚RUST 的FFI定义: Rust 的 FFI（Foreign Function Interface）功能来调用 C 代码

FFI 首先需要解决的一个就是关于类型的问题: 两种语言如何在类型一致上达成共识

关于字符串的类型差异: 
Rust 使用 String 类型来表示拥有的字符串，使用 str 原始码来表示字符串的借用片段。这两种字符串始终采用 UTF-8 编码，
中间可能包含 nul 字节，也就是说，如果查看组成字符串的字节，其中可能有一个 0 字节。String 和 str 都显式地存储其长度；字符串的末尾不像 C 语言那样有 nul 结束符。

C 字符串与 Rust 字符串不同点: 

 - 编码不同:  Rust 字符串使用 UTF-8，但 C 字符串可能使用其他编码。
   如果使用的是 C 语言字符串，则应明确检查其编码，而不是像 Rust 中那样假定其为 UTF-8。

 - 字符大小: C 语言字符串可以使用 char 或 wchar_t 大小的字符；请注意，C 语言的 char 与 Rust 的 char 不同。
   C 标准对这些类型的实际大小不作解释，但为每种字符类型组成的字符串定义了不同的应用程序接口。
   Rust 字符串始终是 UTF-8，因此不同的 Unicode 字符将以不同的字节数进行编码。
   Rust 类型 char 代表 "Unicode 标量值"，与 "Unicode 代码点 "相似，但并不相同。
 - nul 结束符和隐式字符串长度： 通常情况下，C 语言字符串都是 nul 结束符，即字符串末尾有一个 0 字符。
    要计算字符串的长度，C 代码必须手动调用类似 strlen() 的函数（用于基于字符的字符串），
	或 wcslen() 的函数（用于基于 wchar_t 的字符串）。这些函数返回字符串中不包括 nul 结束符的字符数，
	因此缓冲区长度实际上是 len+1 个字符。Rust 字符串没有 nul 结束符；它们的长度始终存储，无需计算。
	在 Rust 中，访问字符串的长度是一个 O(1) 运算（因为长度已存储）；
	而在 C 语言中，访问字符串的长度是一个 O(n) 运算，因为需要通过扫描字符串以查找 nul 结束符来计算长度。
 - 内部 nul 字符: 当 C 语言字符串具有 nul 结束符时，通常意味着中间不能有 nul 字符, 
    nul 字符基本上会截断字符串。Rust 字符串中间可以有 nul 字符，因为在 Rust 中，nul 不一定是字符串的结束符。

### 类型支持

RUST std::ffi 模块 提供了关于类型的定义 
该模块提供跨非 Rust 接口（如其他编程语言和底层操作系统）处理数据的实用程序。
它主要用于 FFI（外来函数接口）绑定以及需要与其他语言交换类似 C 语言字符串的代码。


从 Rust 到 C：CString 代表一个自有的、对 C 语言友好的字符串：它以 nul 结尾，内部没有 nul 字符。
   Rust 代码可以从普通字符串中创建 CString（前提是字符串中间没有 nul 字符），
   然后使用各种方法获得原始 *mut u8，并将其作为参数传递给使用 C 语言字符串约定的函数。

从 C 到 Rust：CStr 表示借用 C 语言的字符串；您可以用它来封装从C语言函数中获取的原始 *const u8。
   CStr 保证是一个以空字节结尾的字节数组。一旦有了 CStr，如果它是有效的 UTF-8，就可以将其转换为 Rust &str，
   或者通过添加替换字符进行有损转换。


其他类型:

- c_char	Equivalent to C’s char type.
- c_double	Equivalent to C’s double type.
- c_float	Equivalent to C’s float type.
- c_int	Equivalent to C’s signed int (int) type.
- c_long	Equivalent to C’s signed long (long) type.
- c_longlong	Equivalent to C’s signed long long (long long) type.
- c_schar	Equivalent to C’s signed char type.
- c_short	Equivalent to C’s signed short (short) type.
- c_uchar	Equivalent to C’s unsigned char type.
- c_uint	Equivalent to C’s unsigned int type.
- c_ulong	Equivalent to C’s unsigned long type.
- c_ulonglong	Equivalent to C’s unsigned long long type.
- c_ushort	Equivalent to C’s unsigned short type.

- c_void: Enums Equivalent to C’s void type when used as a pointer.


### RUST 调用C

首先我们定义一个C的函数接口 把他编译为 一个静态库
```
// gcc 
unsigned int mystrlen(char *str) {
	unsigned int c;
	for (c = 0; *str!= '\0'; c++,*str++);
	return c ;
}
```

在rust中声明C接口 

```
extern "C" {
	fn mystrlen(str: *const c_char) -> c_uint;
}


fn main() {
	let c_string = CString::new("C from Rust").expect("failed");
	let count = unsafe {
		mystrlen(c_string.as_ptr())
	};
	
	println!("lenth: {}",count);
}
```

编译命令:

```
 gcc -shared -o liblen.so len.c
 export LD_LIBRARY_PATH=./
 rustc  main.rs -L./  -l len
guoweikang@ubuntu-virtual-machine:~/code/rust/c$ tree
.
├── len.c
├── liblen.so
├── main
└── main.rs

```

###  C调用RUST

首先 如果是C调用RUST，调用的RUST代码，需要先编译为一个动态库 通过指定 `crate-type`
```
rustc crate-type=cdylib
```

让我们重点关注一下代码这边

```
use std::ffi::Cstr;
use std::os::raw::c_char;

#[repr(C)] // 布局遵循C语言的布局模式 
pub euum Order {
	Gt,
	Lt,
	Eq
}

#[no_mangle]//ABI符合C语言的ABI
pub extern "C" fn compare_str(a: *const c_char, b: *const c_char) -> Order 
{
	let a = unsafe {Cstr::from_ptr(a).to_bytes()};
	let b = unsafe {Cstr::from_ptr(b).to_bytes()};

	if a > b {
		Order::Gt
	} else if a < b {
		Order::Lt
	} else {
		Order::Eq
	}
}
```


```
#include <stdint.h>
#include <stdio.h>

int32_t compare_str(const char* val1,const char* val2);

int main() {
	printf("%d\n",compare_str("amanda","brian"));
	return 0;
}
```

编译命令:

```
rustc  --crate-type=cdylib  ./cmp.rs
export LD_LIBRARY_PATH=./
gcc main.c  -L ./ -lcmp -o main
```