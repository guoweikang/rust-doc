# 执行流控制

执行流控制，一般指在一个代码块/函数内部的 指令跳转，对应到最终的汇编，都是jump指令；
函数调用也是一种跳转，只是函数的调用往往会伴随函数返回，入栈出栈、以及上下文保存恢复等；

本小节主要解释第一种，RUST中使用到的函数内部代码块的跳转，关于for的跳转，我们将在学习完迭代器之后再使用

### if else  
和其他语言一样，RUST 和C相比，特殊的有: 

 - 不需要加**()** 包裹条件
 - if else 可以作为表达式做 右值，右值的返回类型必须是相同
 
```
// Fix the errors
fn main() {
    let n = 5;

    let big_n =
        if n < 10 && n > -10 {
            println!(", and is a small number, increase ten-fold");

            10 * n
        } else {
            println!(", and is a big number, halve the number");
            n / 2.0 ;
        }

    println!("{} -> {}", n, big_n);
} 
```

### loop 
loop作为循环表达式，配合break continue退出循环或者执行下一次循环，RUST 的loop有以下优点: 

 - loop 可以做为 表达式，配合break 可以当作右值使用
 - loop 嵌套使用，支持通过 **`label_name** 标识循环，达到类似C语言中 goto的效果

练习: 熟悉loop的基本使用
```
// Fill in the blanks
fn main() {
    let mut count = 0u32;

    println!("Let's count until infinity!");

    // Infinite loop
    loop {
        count += 1;

        if count == 3 {
            println!("three");

            // Skip the rest of this iteration
            __;
        }

        println!("{}", count);

        if count == 5 {
            println!("OK, that's enough");

            __;
        }
    }

    assert_eq!(count, 5);

    println!("Success!");
}
```

练习: loop作为右值表达式使用 


```
// Fill in the blank
fn main() {
    let mut counter = 0;

    let result = loop {
        counter += 1;

        if counter == 10 {
            __;
        }
    };

    assert_eq!(result, 20);

    println!("Success!");
}
```

练习: loop使用breka lable 跳出循环

```
// Fill in the blank
fn main() {
    let mut count = 0;
    'outer: loop {
        'inner1: loop {
            if count >= 20 {
                // This would break only the inner1 loop
                break 'inner1; // `break` is also works.
            }
            count += 2;
        }

        count += 5;

        'inner2: loop {
            if count >= 30 {
                // This breaks the outer loop
                break 'outer;
            }

            // This will continue the outer loop
            continue 'outer;
        }
    }

    assert!(count == __);

    println!("Success!");
}
```


### while 

while 是一种加了条件的跳转 

 - while 不能做右值
 - 因为loop基本上实现了while 的能力，而且更加强大，一般不怎么使用while 

```
// Fill in the blanks to make the last println! work !
fn main() {
    // A counter variable
    let mut n = 1;

    // Loop while the condition is true
    while n __ 10 {
        if n % 15 == 0 {
            println!("fizzbuzz");
        } else if n % 3 == 0 {
            println!("fizz");
        } else if n % 5 == 0 {
            println!("buzz");
        } else {
            println!("{}", n);
        }


        __;
    }

    println!("n reached {}, so loop is over",n);
}

```
