# use

use的使用建立在 模块树的建立基础上(无论是内部的 还是外部的) 

use声明用来创建一个或多个与程序项路径同义的本地名称绑定。
通常使用 use声明来缩短引用模块所需的路径。这些声明可以出现在模块和块中，但通常在作用域顶部。


### use基本使用 

use 声明可以将一个完整的路径绑定到一个新的名字，从而更容易访问。

```
// 将 `deeply::nested::function` 路径绑定到 `other_function`。
use deeply::nested::function as other_function;

fn function() {
    println!("called `function()`");
}

mod deeply {
    pub mod nested {
        pub fn function() {
            println!("called `deeply::nested::function()`")
        }
    }
}

fn main() {
    // 更容易访问 `deeply::nested::funcion`
    other_function();

    println!("Entering block");
    {
        // 这和 `use deeply::nested::function as function` 等价。
        // 此 `function()` 将遮蔽外部的同名函数。
        use deeply::nested::function;
        function();

        // `use` 绑定拥有局部作用域。在这个例子中，`function()`
        // 的遮蔽只存在这个代码块中。
        println!("Leaving block");
    }

    function();
}
```

use 也支持 `{}` 花括号的写法 一次可以导入多个 程序项 

use 也支持 `self` ,当使用self 之后，使用该模块的函数 就和在自己模块中一样 


### use更多使用

参考
 -  https://kaisery.github.io/trpl-zh-cn/ch07-04-bringing-paths-into-scope-with-the-use-keyword.html
 - https://rustwiki.org/zh-CN/reference/items/use-declarations.html#use%E5%8F%AF%E8%A7%81%E6%80%A7






