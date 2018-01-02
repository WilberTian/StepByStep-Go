在Go中，和其他语言一样，函数用来定义可重用代码块。但是，Go函数不支持重载(overload)和 默认参数(default parameter)。

### 函数

Go语言中，使用`func`关键字定义方法，模式如下：

	func function_name( [parameter list] ) [return_types] {		// 左⼤括号依旧不能在新的一行
		
	}


下面看一个具体的例子：

	func swap(a, b int) (int, int) {	// 类型相同的相邻参数可合并，多返回值必须⽤括号
		return b, a
	}


定义了`swap`就是一个函数，它的函数签名是`func swap(int, int) (int, int)`。

函数的可见性也遵循Go原因的可见性规则，`swap`函数名称是小写开头的，所以它的作用域只属于所声明的包内使用；如果把函数名以大写字母开头，就可以被其他包调用。


函数是第⼀类对象，可作为参数传递：

	func swap(a, b int) (int, int) {	
		return b, a
	}

	func wrapper (fn func(int, int) (int, int), a, b int) {
		fmt.Println(fn(a, b))
	}

	func main() {
		wrapper(swap, 1, 10)
	}

如果直接将函数的前面写在函数的参数列表中，有时候会比较长，所以建议将复杂签名定义为函数类型：

	func swap(a, b int) (int, int) {
		return b, a
	}

	type swapFunc func(int, int) (int, int)		// 定义函数类型

	func wrapper (fn swapFunc, a, b int) {
		fmt.Println(fn(a, b))
	}

	func main() {
		wrapper(swap, 1, 10)
	}




### 可变参数

Go函数可以是任意多个，这种称之为可变参数，比如常用的fmt.Println()这类函数，可以接收一个可变的参数。

可以变参数，可以是任意多个，可变参数的定义，在类型前加上省略号`...`即可。

看一个例子：

	func main() {
		print("Hello", ",", "wrold")
	}

	func print(strs ...string) {
		for _, v := range strs {
			fmt.Println(v)
		}
	}

可变变参本质上就是`slice`，**只能有⼀个，且必须是最后⼀个**。

当使⽤`slice`对象做变参时，需要通过`...`进行展开：


	func main() {
		s := []string{"Hello", ",", "wrold"}
		print(s...)
	}

	func print(strs ...string) {
		for _, v := range strs {
			fmt.Println(v)
		}
	}




### 多值返回

Go语言支持函数方法的多值返回，例如标准库里的很多方法，都是返回两个值，第一个是函数需要返回的值，第二个是出错时返回的错误信息。

函数声明定义的时候，采用逗号分割，当有多个返回的时候，就用用括号括起来。返回值还是使用`return`关键字，以逗号分割，和返回的声明的顺序一致。

多值返回的函数定义非常简单，看个例子：

	func add(a, b int) (int, error) {
		return a + b, nil
	}

如果返回的值不想使用，可以使用`_`进行忽略：

	result, _ := add(1, 10)




### 命名返回参数

Go函数还支持命名返回参数，命名返回参数可看做与形参类似的局部变量，最后由`return`隐式返回。

	func add(x, y int) (z int) {
		z = x + y
	 	return
	}

	func main() {
	 	fmt.Println(add(1, 2))
	}


命名返回参数允许`defer`延迟调⽤通过闭包读取和修改：

	func add(x, y int) (z int) {
		defer func() {
	 		z += 100
	 	}()

	 	z = x + y
	 	return
	}

	func main() {
		fmt.Println(add(1, 2))
	}

代码输出：
	
	103




#### 匿名函数

匿名函数可赋值给变量，做为结构字段，或在`channel`⾥传送。

	func main() {
	 	// --- function variable ---
		fn := func() { fmt.Println("Hello, World!") }
		fn()

		// --- function as field ---
		d := struct {
			fn func() string
		}{
			fn: func() string { return "Hello, World!" },
		}

		fmt.Println(d.fn())

		// --- channel of function ---
		fc := make(chan func() string, 2)
		fc <- func() string { return "Hello, World!" }
		fmt.Println((<-fc)())
	}


#### 延迟调用

Go函数支持延迟调用这个非常灵活的特性，通过关键字`defer`来注册延迟调⽤，这些调⽤直到`return`前才被执⾏，延迟调用通常⽤于释放资源或错误处理。


	func createFile() error {
		f, err := os.Create("tmp.txt")
	 	if err != nil { return err }

	 	defer f.Close() 	// 注册调⽤，⽽不是注册函数；必须提供参数，哪怕为空

	 	f.WriteString("Hello, World!")
	 	return nil
	}

当有多个`defer`注册的时候，按`FILO`次序执⾏。哪怕函数或某个延迟调⽤发⽣错误，这些调⽤依旧会被执⾏。

	func test(x int) {
		defer fmt.Println("a")
		defer fmt.Println("b")

		defer func() {
			fmt.Println(100 / x) // div0 异常未被捕获，逐步往外传递，最终终⽌进程。
		}()

		defer fmt.Println("c")
	}

	func main() {
		test(0)
	}

代码输出：

	c
	b
	a
	panic: runtime error: integer divide by zero


延迟调⽤参数在注册时求值或复制，可⽤**指针或闭包"延迟"读取**：

	func test() {
		x, y := 10, 20

		defer fmt.Println("defer without closure:", x, y)	// 没有使用闭包下x, y的值

		defer func(i int, j *int) {
			fmt.Println("defer with closure:", i, *j, y) 		// y闭包引⽤
		}(x, &x) 

		x += 10
		y += 100
		fmt.Println("x =", x, "y =", y)
	}

	func main() {
		test()
	}

代码输出为：

	x = 20 y = 120
	defer with closure: 10 20 120
	defer without closure: 10 

	
