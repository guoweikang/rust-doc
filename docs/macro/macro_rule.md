# 声明式宏

上一个小节，我们介绍了过程宏，可以看到，过程宏的定义都是以一个 `fn macro_name(TokenStrem) -> TokenStrem` 函数的形式存在

过程宏的实现，需要了解 `TokenStrem`的相关方法的使用；同时过程宏 是非卫生的，会受外部代码影响，也可能影响外部代码 

本节内容将介绍 `声明式宏`，相信在之前的基础上 这一章内容不会很难


### macro_rules!

我们先介绍 声明式宏的基本定义形式: 
```
macro_rules! IDENTIFIER MacroRulesDef
```

 - `macro_rules!` 开头，实际上它也是一个宏，该宏的定义内嵌在编译器中
 - `IDENTIFIER` 声明宏的名称，每个声明宏都一个名称
 - `MacroRulesDef` 宏规则定义 每个宏可以有一条或多条规则
 

我们从外往里解析 `MacroRulesDef` , MacroRulesDef可以有三种形式 
```
macro_rules! double  {MacroRules}
macro_rules! double  [MacroRules];
macro_rules! double  (MacroRules);
```
上面三种形式功能是一样的，习惯于使用第一种形式 在继续分解 `MacroRules`

```
macro_rules! double  {
	MacroRule; 
	MacroRule; 
}
```

这里可以看到有多条宏的展开，这是因为`声明式宏`可以根据不同情况有多种展开形式

下面例子说明了这一点

```
#[derive(Debug)]
struct Apple(i32);

impl Drop for Apple {
        fn drop(&mut self) {
            println!("drop apple {:?}",self);
        }
}
macro_rules! double  [
    ($x: ident) => {let app = Apple(100); $x *= 2}; //说明卫生
    ($x: expr, $y: expr) => {$x*=2; $y*=2};
    ($x: literal) => {panic!("aaa")};
];


fn main() {
    let mut a = 10;
    let app = Apple(10);
    double!(a);
    let mut b = 10;
    double!(a,b);
    double!(10);

    println!("still use app {:?}",app);//说明卫生
    println!("a {} b {}",a,b);//说明卫生
}
``` 

让我们继续对`MacroRule` 进行分解  可以看到由一个 `MacroMatcher` + `=> ` + `MacroTranscriber` 组成

宏在展开时，会先根据token 尝试匹配 `MacroMatcher`，如果匹配到预期的 `MacroMatcher` 则进行对应的展开 
```
macro_rules! double  {
	MacroMatcher => MacroTranscriber;
	MacroMatcher => MacroTranscriber;
}
```

### MacroMatcher
由于匹配器的内容较多 我们单独增加一个小节 尝试说明

当一个宏被调用时，`macro_rules!` 解释器将按照声明顺序一一检查规则。

对每条规则，它都将尝试将输入标记树的内容与该规则的` matcher `进行匹配。某个` matcher `必须与输入完全匹配才被认为是一次匹配。

如果所有规则均匹配失败，则宏展开失败并报错。

最简单的例子是空 matcher：

```
macro_rules! four {
    () => { 1 + 3 };
}

//当且仅当匹配到空的输入时，匹配成功，即 four!()、four![] 或 four!{} 三种方式调用是匹配成功的 。
```

下面是一个稍微复杂的匹配

```
macro_rules! gibberish {
    (4 fn ['spang "whammo"] @_@) => {...};
}
//使用下述匹配可以匹配成功
gibberish!(4 fn ['spang "whammo"] @_@])
```

从上述两个例子，可以看到，macro_rules 的匹配 本质上在匹配 宏调用时传入的 `tokenStream`

### 元变量

有时，我们除了匹配，还想要复用 输入流 中的元素(大部分情况下是这样的),这样的变量 我们也叫`元变量`，匹配器可以在匹配的同时 绑定匹配内容到一个变量

捕获的书写方式是：先写美元符号`$`，然后跟一个`标识符`，然后是冒号 `:`，最后是捕获方式，比如 `$e:expr`。

一旦想要捕获变量，需要描述捕获方式，捕获方式有以下类型

 - block：一个块（比如一块语句或者由大括号包围的一个表达式）
 - expr：一个表达式 (expression)
 - ident：一个标识符 (identifier)，包括关键字 (keywords)
 - item：一个条目（比如函数、结构体、模块、impl 块）
 - lifetime：一个生命周期注解（比如 'foo、'static）
 - literal：一个字面值（比如 "Hello World!"、3.14、'🦀'）
 - meta：一个元信息（比如 #[...] 和 #![...] 属性内部的东西）
 - pat：一个模式 (pattern)
 - path：一条路径（比如 foo、::std::mem::replace、transmute::<_, int>）
 - stmt：一条语句 (statement)
 - tt：单棵标记树
 - ty：一个类型
 - vis：一个可能为空的可视标识符（比如 pub、pub(in crate)）

```
macro_rules! times_five {
    ($e:expr) => { 5 * $e }; //通过捕获表达式的方式，定义元变量e 然后再 宏展开中 可以通过 $e 使用元变量
}

fn main() {
	let a = 10;
	let b = times_five!(a);
	
	println!("{}",b);
}
```

更多关于各个捕获方式的定义，比如什么是表达式 什么是标识符 参考文档

元变量支持捕获多个，并且元变量支持重复使用

```
macro_rules! discard {
    ($e:expr) => {};
}
macro_rules! repeat {
    ($e:expr) => { $e; $e; $e; };
}
macro_rules! multiply_add {
    ($a:expr, $b:expr, $c:expr) => { $a * ($b + $c) };
}
```

### 反复捕获 
Ok，如果你学习过正则表达式，对于贪婪匹配一定不陌生，宏的匹配器也支持贪婪匹配 基本语法为

```
$ ( ... ) sep rep
```

 - `$` 是字面上的美元符号标记
 - `( ... )` 是被反复匹配的模式，由小括号包围
 - `sep` 是可选的分隔标记。它不能是括号或者反复操作符 rep。常用例子有` ,` 和` ; ` 
 - `rep`  是必须的重复操作符。当前可以是
 
    - `?` 表示最多一次重复，所以此时不能前跟分隔标记
    - `*` 表示零次或多次重复
	- `+` 表示一次或多次重复

```
macro_rules! vec_strs {
    (
        // 开始反复捕获
        $(
            // 每个反复必须包含一个表达式
            $element:expr
        )
        // 由逗号分隔
        ,
        // 0 或多次
        *
    ) => {
        // 在这个块内用大括号括起来，然后在里面写多条语句
        {
            let mut v = Vec::new();

            // 开始反复捕获
            $(
                // 每个反复会展开成下面表达式，其中 $element 被换成相应被捕获的表达式
                v.push(format!("{}", $element));
            )*

            v
        }
    };
}

fn main() {
    let s = vec_strs![1, "a", true, 3.14159f32];
    assert_eq!(s, &["1", "a", "true", "3.14159"]);
}
```


你可以在一个反复语句里面使用多次和多个元变量，只要这些元变量以相同的次数重复。所以下面的宏代码正常运行：

```
macro_rules! repeat_two {
    (
		// 开始反复捕获
		$(
			// 每个反复必须包含一个表达式
			$i:ident
		)
		//没有分隔符 直接匹配0次或多次 标识符 ident 
		*
		//裸字符 匹配 `,` 
		, 
		// 开始反复捕获
		$($i2:ident)*) 
		
		=> {
		 // 在这个块内用大括号括起来，然后在里面写多条语句
			$( let $i: (); let $i2: (); )*
		}
}

fn main () {
    repeat_two!( a b c d e f, u v w x y z );
}
```

尝试修复下面代码
```
macro_rules! repeat_two {
    ($($i:ident)*, $($i2:ident)*) => {
        $( let $i: (); let $i2: (); )*
    }
}

fn main() {
    repeat_two!( a b c d e f, x y z );
}

```

### 元变量表达式

元变量表达式为 代码翻译器 提供了关于元变量的信息 —— 这些信息是不容易获得的。

目前除了 `$$` 表达式外，它们的一般形式都是 `$ { op(...) }`, 即除了 $$ 以外的所有元变量表达式都涉及反复。

可以使用以下表达式（其中 `ident` 是所绑定的元变量的名称，而 `depth` 是整型字面值）：


 - ${count(ident)}：最里层反复 $ident 的总次数，相当于 ${count(ident, 0)}
 - ${count(ident，depth)}：第 depth 层反复 $ident 的次数
 - ${index()}：最里层反复的当前反复的索引，相当于 ${index(0)}
 - ${index(depth)}：在第 depth 层处当前反复的索引，向外计数
 - ${length()}：最里层反复的重复次数，相当于 ${length(0)}
 - ${length(depth)}：在第 depth 层反复的次数，向外计数
 - ${ignore(ident)}：绑定 $ident 进行重复，并展开成空
 - $$：展开为单个 $，这会有效地转义 $ 标记，因此它不会被展开（转写）
 


### 再谈分类符

`block` 分类符只匹配 block 表达式。

块 (block) 由 `{ `开始，接着是一些语句，最后是可选的表达式，然后以 `} `结束。 
块的类型要么是最后的值表达式类型，要么是 () 类型。

```
macro_rules! blocks {
    ($($block:block)*) => ( $(println!("{}", stringify!($block));)* );
}

fn main() {
    blocks! {
        {}
        {
            let zig;
        }
        { 2 }
    }
}

```

`expr `分类符用于匹配任何形式的表达式 [expression](https://doc.rust-lang.org/reference/expressions.html)

```
macro_rules! expressions {
    ($($expr:expr)*) => ( $(println!("{}", stringify!($expr));)* );
}
fn main() {
	expressions! {
		"literal"
		funcall()
		future.await
		break 'foo bar
	}
}
```

`ident` 分类符用于匹配任何形式的标识符 (identifier) 或者关键字

```
macro_rules! idents {
    ($($ident:ident)*) => ( $(println!("{}", stringify!($ident));)* );
}

fn main() {

	idents! {
		// _ <- `_` 不是标识符，而是一种模式
		foo
		async
		O_________O
		_____O_____
	}
}
```

`item `分类符只匹配 Rust 的 [item](https://doc.rust-lang.org/reference/items.html) 的 定义 ， 
不会匹配指向 item 的标识符 (identifiers)。例子：

```
macro_rules! items {
    ($($item:item)*) => ($(println!("{}", stringify!($item));)* );
}

fn main() {

	items! {
		struct Foo;
		enum Bar {
			Baz
		}
		impl Foo {}
		/*...*/
	}
}
```


`lifetime `分类符用于匹配生命周期注解或者标签。 它与 ident 很像，但是 lifetime 会匹配到前缀 `'' `


```
macro_rules! lifetime {
    ($($lifetime:lifetime)*) => ($(println!("{}", stringify!($lifetime));)* );
}

fn main() {

	lifetime! {
		'static
		'shiv
		'_
	}
}
```

`literal`分类符用于匹配字面表达式 

```
macro_rules! literals {
    ($($literal:literal)*) => ($(println!("{}", stringify!($literal));)* );
}

fn main() {

	literals! {
		-1
		"hello world"
		2.3
		b'b'
		true
	}
}
```

`meta`分类符用于匹配属性 (attribute)， 准确地说是属性里面的内容。
通常你会在 `#[$meta:meta] ` 或 `#![$meta:meta]` 模式匹配中 看到这个分类符。

```
macro_rules! metas {
    ($(#[$meta:meta])*) => ($(println!("{}", stringify!($meta));)* );
}

fn main() {

	metas! {
		#[cfg]
		#[dbg]
	}
}
```
针对文档注释简单说一句： 文档注释其实是具有 `#[doc="…"]` 形式的属性，
`...`实际上就是注释字符串， 这意味着你可以在在宏里面操作文档注释！


`pat`分类符用于匹配任何形式的模式 [pattern](https://doc.rust-lang.org/reference/patterns.html)，
包括 2021 edition 开始的 `or-patterns`

```
macro_rules! patterns {
    ($($pat:pat)*) =>  ($(println!("{}", stringify!($pat));)* );
}

fn main() {

	patterns! {
	    "literal"
		_
		0..5
		ref mut PatternsAreNice
		0 | 1 | 2 | 3 
	}
}
```

从 2021 edition 起， or-patterns 模式开始应用，这让 pat 分类符不再允许跟随 `|`

为了避免这个问题或者说恢复旧的 pat 分类符行为，你可以使用 pat_param 片段，
它允许 `|` 跟在它后面，因为 pat_param 不允许 top level 或 or-patterns。
```
macro_rules! patterns {
    (pat: $pat:pat) => {
        println!("pat: {}", stringify!($pat));
    };
    (pat_param: $($pat:pat_param)|+) => {
        $( println!("pat_param: {}", stringify!($pat)); )+
    };
}
fn main() {
    patterns! {
       pat: 0 | 1 | 2 | 3
    }
    patterns! {
       pat_param: 0 | 1 | 2 | 3
    }
}
```

```
macro_rules! patterns {
    ($( $( $pat:pat_param )|+ )*)  =>  
    ($( $( println!("{}", stringify!($pat)); )+)* );
}

fn main() {

	patterns! {
	    "literal"
		_
		0..5
		ref mut PatternsAreNice
		0 | 1 | 2 | 3 
	}
}
```


### 更多细节

更多细节参考 [宏小册](https://zjp-cn.github.io/tlborm/decl-macros.html)
