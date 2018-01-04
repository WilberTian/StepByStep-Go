Go语言中提供了接口这个抽象类型，接口只是定义行为集合，不涉及实现。所有的实现都是通过实现接口的实例来完成。



### 接口定义

接口是一个抽象的类型，声明了⼀个或多个⽅法签名的集合，任何类型的⽅法集中只要拥有与之对应的全部⽅法，就表⽰这个类型“实现”了该接⼝，⽆须在该类型上显式添加接⼝声明。

所谓对应⽅法，是指有相同名称、参数列表(不包括参数名)以及返回值，除了接口中声明的所有方法，类型还可以有其他⽅法。

看一个例子，例子中定义了两个接口，其中`Printer`嵌入了`Stringer`接口：

	type Stringer interface {
		String() string
	}

	type Printer interface {
		Stringer 
		Print()
	}

	type User struct {
		id   int
		name string
	}

	func (self *User) String() string {
		return fmt.Sprintf("user id is %d, user name is %s", self.id, self.name)
	}

	func (self *User) Print() {
		fmt.Println(self.String())
	}

	func main() {
		var t Printer = &User{1, "Will"} 
		t.Print()
	}

代码输出为：

	user id is 1, user name is Will




### 空接口（empty interface）

Go语言中，空接⼝`interface{}`没有任何⽅法签名，也就意味着任何类型都实现了空接⼝，其作⽤类似⾯向对象语⾔中的根对象`object`。

	var v1 interface{} = 1
	var v2 interface{} = "abc"
	var v3 interface{} = struct{ X int }{1}

如果函数打算接收任何数据类型，则可以将参考声明为`interface{}`，最典型的例子就是标准库`fmt`包中的`Print`和`Fprint`系列的函数：

	func Fprint(w io.Writer, a ...interface{}) (n int, err error) 
	func Fprintf(w io.Writer, format string, a ...interface{})
	func Fprintln(w io.Writer, a ...interface{})
	func Print(a ...interface{}) (n int, err error)
	func Printf(format string, a ...interface{})
	func Println(a ...interface{}) (n int, err error)

匿名接⼝可⽤作变量类型，或结构成员：

	type Stringify struct {
		s interface {
			String() string
		}
	}

	type User struct {
		id   int
		name string
	}

	func (self *User) String() string {
		return fmt.Sprintf("user id is %d, user name is %s", self.id, self.name)
	}

	func main() {
		t := Stringify{&User{1, "Will"}}
		fmt.Println(t.s.String())
	}

代码输出为：

	user id is 1, user name is Will


注意，`[]T`不能直接赋值给`[]interface{}`，会遇到变异错误：

    t := []int{1, 2, 3, 4}
    var s []interface{} = t

需要通过遍历子元素进行赋值：

	t := []int{1, 2, 3, 4}
	s := make([]interface{}, len(t))
	for i, v := range t {
	    s[i] = v
	}




### 接口的本质

接口是一个抽象的类型，接口只是定义方法而没有实现。

接口对象本质上是拥有两个字段的数据结构，接口也是一个引用类型：

- 一个字段是接口表指针，这个内部表里存储的有实体类型的信息以及相关联的方法集
- 一个字段是指向存储的实体类型值的数据指针

看看`runtime.h`里面关于接口的定义

	struct Iface
	{
		Itab* tab;
		void* data;
	};

	struct Itab
	{
		InterfaceType* inter;
		Type* type;
		void (*fun[])(void);
	};

接⼝表存储元数据信息，包括**接⼝类型、动态类型，以及实现接⼝的⽅法指针**。⽆论是反射还是通过接⼝调⽤⽅法，都会⽤到这些信息。  

数据指针指向目标对象的只读复制品，复制完整对象或指针。

另外，接⼝转型返回临时对象，只有使⽤指针才能修改其状态，看下面的例子：

	type User struct {
		id   int
		name string
	}

	func main() {
		u := User{1, "Will"}
		var i interface{} = u
		
		u.id = 2
		u.name = "July"

		fmt.Printf("%v\n", u)
		fmt.Printf("%v\n", i.(User))
	}

代码输出：
	
	{2 July}
	{1 Will}




### 接口的转换

在Go语言中，利⽤类型推断，可判断接⼝对象是否某个具体的接⼝或类型。看一个例子：

	type User struct {
		id   int
		name string
	}

	func (self *User) String() string {
		return fmt.Sprintf("%d, %s", self.id, self.name)
	}

	func main() {
		var o interface{} = &User{1, "Will"}
		if i, ok := o.(fmt.Stringer); ok { 
			fmt.Println(i)
		}
		u := o.(*User)
		// u := o.(User) 	// panic: interface is *main.User, not main.User
		fmt.Println(u)
	}

超集接⼝对象可转换为⼦集接⼝，反之出错。例子中`Printer`接口内嵌了`Stringer`接口，所以实现`Printer`接口的实例可以赋值给`Stringer`接口：

	type Stringer interface {
		String() string
	}

	type Printer interface {
		String() string
		Print()
	}

	type User struct {
		id   int
		name string
	}

	func (self *User) String() string {
		return fmt.Sprintf("%d, %v", self.id, self.name)
	}

	func (self *User) Print() {
		fmt.Println(self.String())
	}

	func main() {
		var o Printer = &User{1, "Will"}

		var s Stringer = o

		fmt.Println(s.String())
	}





### 方法集

在实现一个接口的时候，就实现了这个接口的所有方法；但是实现方法的时候，可以使用指针接收者实现，也可以使用值接收者实现，这两者是有区别的。

看下面的例子：

	type Stringer interface {
		Vstring() string
	}

	type User struct {
		id   int
		name string
	}

	func (self User) Vstring() string {
		return fmt.Sprintf("Vstring: %d, %v", self.id, self.name)
	}

	func main() {
		var u1 Stringer = User{1, "Will"}
		fmt.Println(u1.Vstring())

		var u2 Stringer = &User{2, "Will"}
		fmt.Println(u2.Vstring())
	}

代码输出为：

	Vstring: 1, Will
	Vstring: 2, Will

稍微修改下上面的代码：

	type Stringer interface {
		Vstring() string
		Pstring() string
	}

	type User struct {
		id   int
		name string
	}

	func (self User) Vstring() string {
		return fmt.Sprintf("Vstring: %d, %v", self.id, self.name)
	}

	func (self *User) Pstring() string {
		return fmt.Sprintf("Pstring: %d, %v", self.id, self.name)
	}


	func main() {
		// var u1 Stringer = User{1, "Will"}
		// cannot use User literal (type User) as type Stringer in assignment:
		// User does not implement Stringer (Pstring method has pointer receiver)

		var u2 Stringer = &User{2, "Will"}
		fmt.Println(u2.Vstring())
		fmt.Println(u2.Pstring())
	}

代码输出为：

	Vstring: 2, Will
	Pstring: 2, Will


这里就涉及到了方法集的概念，在Go中不同的类型就会关联不同的方法集，一般规则如下：

- 类型`T`⽅法集包含全部`receiver T`⽅法
- 类型`*T`⽅法集包含全部`receiver T + *T`⽅法


**⽤实例`value`和`pointer`调⽤⽅法(含匿名字段)不受⽅法集约束，编译器总是查找全部⽅法，并⾃动转换`receiver`实参**。

