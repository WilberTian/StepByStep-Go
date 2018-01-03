在Go语言中，函数是指不属于任何结构体、类型的方法，也就是说，函数是没有绑定对象实例的（没有接收者）。

而⽅法总是绑定对象实例，并隐式将实例作为第⼀实参(receiver)。




### 方法定义

方法的声明和函数类似，唯一的区别是：方法在定义的时候，会在`func`关键字和方法名之间增加一个参数，这个参数就是接收者(receiver)，通过这种方式定义的方法就和接收者绑定在一起了，称之为这个接收者的方法。

看一个例子：

	type person struct {
		name string
	}

	func (p person) String() string {
		return "the person's name is " + p.name
	}

	func main() {
		p := person{name: "will"}
		fmt.Println(p.String())
	}

调用的方法非常简单，使用类型的变量进行调用即可，类型变量和方法之前是一个`.`操作符，表示要调用这个类型变量的某个方法。

关于方法的定义，有一些注意点：

- 只能为当前包内命名类型定义⽅法
- 参数`receiver`可任意命名，如⽅法中未曾使⽤，可省略参数名
- 参数`receiver`类型可以是`T`或`*T`，基类型`T`不能是接⼝或指针
- 不⽀持⽅法重载，`receiver`只是参数签名的组成部分（即使`receiver`的类型不同，方法名也不能相同）
- 可⽤实例`value`或`pointer`调⽤全部⽅法，编译器⾃动转换


下面具体看看方法的接收者。




### 方法的接收者

Go语言里有两种类型的接收者：值接收者和指针接收者。

	func (s *MyStruct) pointerMethod() { } 	// method on pointer

	func (s MyStruct) valueMethod() { } 	// method on value


使用值类型接收者定义的方法，在调用的时候，使用的其实是值接收者的一个副本，所以对该值的任何操作，不会影响原来的类型变量。

	func main() {
		p:=person{name:"will"}
		p.modify() 					// 指针接收者，修改有效

		fmt.Println(p.String())
	}

	type person struct {
		name string
	}

	func (p person) String() string{
		return "the person's name is " + p.name
	}

	func (p *person) modify(){
		p.name = "july"
	}

代码输出：

	the person's name is july

*在调用方法的时候，传递的接收者本质上都是副本，只不过一个是这个值副本，一是指向这个值指针的副本。指针具有指向原有值的特性，所以修改了指针指向的值，也就修改了原有的值。*


在上面的例子中，在调用指针接收者方法的时候，使用的也是一个值的变量，并不是一个指针，如果换成指针也是可以正常工作的：

	p := person{name:"will"}
	(&p).modify() 

Go语言中没有强制使用指针进行方法调用，Go的编译器自动转换指针，以满足接收者的要求。

同样的，如果是一个值接收者的方法，使用指针也是可以调用的，Go编译器自动会解引用，以满足接收者的要求，比如例子中定义的`String()`方法，也可以这么调用：

	p := person{name:"will"}
	fmt.Println((&p).String())

总之，方法的调用，既可以使用值，也可以使用指针。

那么什么时候用值方法，什么时候用指针方法?

- 如果方法需要修改`receiver`，那么使用指针方法
- 如果`receiver`是一个很大的结构体，考虑到效率，应该使用指针方法




### 方法的本质

⽅法不过是⼀种特殊的函数，为了进一步理解方法，以及`receiver`为`T`和`*T`的差别，可以看看下面的例子：

	type Data struct {
		x int
		y *int
	}

	func (self Data) ValueTest() { 			// func ValueTest(self Data);
		fmt.Printf("Value method: %p\n", &self)
		fmt.Printf("d.y address in value method: %p\n", self.y)
	}

	func (self *Data) PointerTest() { 		// func PointerTest(self *Data);
		fmt.Printf("Pointer method: %p\n", self)
	}

	func main() {
		d := Data{0, new(int)}
		p := &d

		fmt.Printf("d address: %p\n", p)
		fmt.Printf("d.y address: %p\n\n", d.y)

		fmt.Println("Call methods with value")
		d.ValueTest()   	// ValueTest(d)
		d.PointerTest() 	// PointerTest(&d)

		fmt.Println()

		fmt.Println("Call methods with pointer")
		p.ValueTest()   	// ValueTest(*p)
		p.PointerTest() 	// PointerTest(p)
	}

代码输出为：
	
	d address: 0xc42000e1d0
	d.y address: 0xc4200160b0

	Call methods with value
	Value method: 0xc42000e1e0
	d.y address in value method: 0xc4200160b0
	Pointer method: 0xc42000e1d0

	Call methods with pointer
	Value method: 0xc42000e1f0
	d.y address in value method: 0xc4200160b0
	Pointer method: 0xc42000e1d0
	

为了理解方法，可以把方法想象成`receiver`对象为第一个参数的函数。

- 当通过值类型对象调用`receiver`为指针类型的方法时，Go编译器自动将值类型对象通过`&`操作符转换成指针类型，例如`PointerTest(&d)`
- 当通过指针类型对象调用`receiver`为值类型的方法时，Go编译器自动将指针类型对象通过`*`操作符转换成值类型，例如`ValueTest(*p)`




### 匿名字段的方法

在[](Go结构)文章的介绍中了解到，Go中可以像访问普通字段那样访问匿名字段成员；同样，也可以可以像字段成员那样访问匿名字段⽅法，编译器负责查找。

通过匿名字段，可获得和继承类似的复⽤能⼒。依据编译器查找次序，只需在外层定义同名⽅法，就可以实现`override`。


看一个例子：

	type User struct {
		id   int
		name string
	}

	type Manager struct {
		role string
		User
	}

	func (self User) ToString() string { 		
		return fmt.Sprintf("User receiver obj: %p, %v", &self, self)
	}

	/*
	// override
	func (self Manager) ToString() string { 		
		return fmt.Sprintf("Manager receiver obj: %p, %v", &self, self)
	}
	*/

	func (self *User) ChangeName(name string) string { 			// receiver is &(Manager.User)
		self.name = name
		return fmt.Sprintf("*User receiver obj: %p, %v", self, self)
	}

	func main() {
		m := Manager{"manager", User{1, "Will"}}
		pm := &m

		fmt.Printf("Manager obj address: %p\n", &m)
		fmt.Printf("User field obj address: %p\n", &(m.User))

		fmt.Println();

		fmt.Println(m.ToString())
		fmt.Println(pm.ToString())

		fmt.Println();

		fmt.Println(m.ChangeName("july"))
		fmt.Println(pm.ChangeName("july"))
	}

代码输出为：

	Manager obj address: 0xc42007a180
	User field obj address: 0xc42007a190

	User receiver obj: 0xc42000a060, {1 Will}
	User receiver obj: 0xc42000a0a0, {1 Will}

	*User receiver obj: 0xc42007a190, &{1 july}
	*User receiver obj: 0xc42007a190, &{1 july}



