# 生命周期

生命周期是为引用服务;还记得我们讲引用时原则吗？ 

 - 引用生效期间  不应该超过 被引用变量的有效期，否则使用引用指向的内存 是无效的  
 - 在引用使用某个变量期间，变量不能被转移 
 - 在共享引用使用期间(生效时)，不能被修改 也不能有可变引用
 - 在可变引用生效期间 ，变量不能有其他引用
 

本节要探讨的问题是：

 - RUST是如何知道引用什么时候生效 什么时候不生效的？
 - 在什么情况 我们需要明确告诉编译器 某个引用的生命周期？


### 概念讲解

作用域:  作用域指某个变量的可见区间，当变量离开作用域之后会被销毁

```
//离开作用域之后 变量不可见
fn main() {
	let a = 10;
	{
		let a_ref = &a;
	}
	
	println!("{}",a_ref); //cannot find value `a_ref` in this scope
}
``` 

借用检查器: RUST 编译器在编译阶段，会检查每个引用变量 `引用的内存`的有效性  防止引用无效内存 

```

fn main() {
	let a_ref: &i32;
	{
		let a:i32 = 10;
		a_ref = &a;
	}
	println!("a_ref {}",a_ref);
}

``` 

当然，也会检查同一块内存 是否被多个引用使用，以及是否否和多个引用的约束 



### 生命周期标注

大部分情况下，rust 编译器可以通过 代码推出 引用的生命周期，比如上述例子，但是有些情况确无法推断出 

先来一个简单的:

```
#[derive(Debug)]
struct Class {
	id:i32,
	num:i32,
}

#[derive(Debug)]
struct Student<'a> {
	class: &'a Class,
}

fn main() {
	let student_a: Student; 
	{
		let class1:Class = Class {id:1, num:0};
		student_a = Student{class: &class1};
	}
	println!("{:?}", student_a)
}
```

这个例子演示了，如果某个类型包含外部的引用， 必须要明确给出 生命周期占位符，因为该引用可能来自于不同的生命周期 

当结构体通过使用`<'a>` 生命周期占位符以后，表示明确编译器，结构体变量的生命周期不能比`<'a>` 引用的生命周期长

换个方法说，引用的生命周期 `<''a>` 必须要比结构体实例的生命周期长 


稍微复杂点的: 这也是C里面经常会遇到的情况，一个结构体包含另外一个结构体，但是另外一个结构体，也需要保存指向包含结构体的 对象指针


```
#[derive(Debug)]
struct Student<'a> {
    name: String,
    class_ref: &'a Class<'a>,
}

#[derive(Debug)]
struct Class<'a> {
    id: i32,
    students: Vec<Student<'a>>,
}

impl<'a> Class<'a> {
    fn add_student(&'a mut self, name: &str) {
        let students_ref = &mut self.students;
        let nwe_stu = Student {
            name: String::from(name),
            class_ref: self,
        };
        students_ref.push(nwe_stu);
    }
}

fn main() {
    let mut class1 = Class {
        id: 1,
        students: Vec::new(),
    };
    
    class1.add_student("Jack");
    class1.add_student("Rose");
}
```


这个问题，目前还没有办法解决，但是接下来的智能指针 或许能够给我们一些帮助 





