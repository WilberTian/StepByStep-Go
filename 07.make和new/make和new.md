在Go语言中，`new`和`make`函数都用来分配内存，下面看看两个函数的区别。


### `new`函数 

介绍`new`函数之前，先看一个例子：

	func main() {
		var i *int
		*i=10
		
		fmt.Println(*i)
	}

代码输出：

	panic: runtime error: invalid memory address or nil pointer dereference

通过错误可以看出，对于引用类型的变量，不仅要声明它，还要为它分配内容空间。（对于值类型的声明不需要）

对上面的代码进行修改：

	func main() {
		var i *int
		i=new(int)

		fmt.Println(*i)
		
		*i=10
		
		fmt.Println(*i)
	}

代码输出：

	0
	10


内建函数`new`本质上其它语言中的同名函数功能一样，通过`new(T)`分配了**零值填充**的`T`类型的内存空间，并且返回其地址，即一个`*T`类型的值。

用Go的术语说，它返回了一个指针，指向新分配的类型`T`的零值。

对于`new`函数，有一点非常重要：**`new`返回指针**。



### `make`函数

内建函数`make(T, args)`与`new(T)`有着不同的功能，make只能用于`slice`、`map`和`channel`的内存创建，并且返回一个有初始值(非零)的`T`类型，而不是`*T`。

本质来讲，导致这三个类型有所不同的原因是，指向数据结构的引用在使用前必须被初始化。例如，一个`slice`，是一个包含指向数据（底层`array`）的指针、长度和容量的三个字段的结构对象，在这些字段被初始化之前，`slice`为`nil`。

对于`slice`、`map`和`channel`来说，`make`初始化了内部的数据结构，填充适当的值，然后返回初始化后的（非零）值。



### `new`和`make`区别

二者都用来分配内存，但是`make`只用于`slice`、`map`以及`channel`的初始化（非零值）；而`new`用于类型的内存分配，并且内存置为零值。

`make`返回的还是这三个引用类型本身；而`new`返回的是指向类型的指针。


另外，`new`这个内置函数并不常用，因为在Go中可以直接通过字面量的方式达到相同的目的，比如：
	
	type User struct { name string }
	u := User{}

这样更简洁方便，而且不会涉及到指针的操作。


