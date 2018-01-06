Go语言不支持继承，而是提倡通过组合的方式实现代码复用；也就是将已有类型嵌入到新的类型中，通过类型嵌套到达代码复用的目的。


### 使用嵌套类型

嵌套类型的使用比较简单，就是通过类型的组合、嵌套来定义新的类型，看一个例子：

	type User struct {
		id   int
		name string
	}

	type Manager struct {
		User
		role string
	}

	func (self User) GetId() string {
		return fmt.Sprintf("User id is %d", self.id)
	}

	func (self User) GetInfo() string {
		return fmt.Sprintf("User name is %v", self.name)
	}

	func (self Manager) GetInfo() string {
		return fmt.Sprintf("Manager role is %v", self.role)
	}

	func main() {
		m := Manager{
			User: User{1, "will"}, 	
			role: "Administrator",
		}

		fmt.Println(m.GetId())
		fmt.Println(m.GetInfo())
		fmt.Println(m.User.GetInfo())
	}

代码输出为：

	User id is 1
	Manager role is Administrator
	User name is will

从上面例子可以看到，对于内部类型属性和方法的访问，可以用外部类型直接访问，也可以通过内部类型进行访问；但是外部类型新增的方法和属性字段，只能使用外部类型访问。

另外，外部类型也可以声明同名的属性或方法，来覆盖内部类型的响应属性或方法。




### 嵌套类型和接口

前面一篇中介绍了Go接口的使用，对于嵌套类型，还有一点非常灵活的是：**如果内部类型实现了某个接口，那么外部类型也被认为实现了这个接口**。

在接口介绍的文章中提到了“方法集”这个概念，当使用嵌套类型的时候，方法集的规则如下：

- 如类型`S`包含匿名字段`T`，则`S`⽅法集包含`receiver T`⽅法
- 如类型`S`包含匿名字段`*T`，则`S`⽅法集包含`receiver T + *T`⽅法
- 不管嵌⼊`T`或`*T`，`*S`⽅法集总是包含`receiver T + *T`⽅法

为了理解上面关于类型嵌套时候的方法集规则，先看一个例子，`Manager`类型包含`User`类型：

	type Stringer interface {
		Vstring() string
	}

	type User struct {
		id   int
		name string
	}

	type Manager struct {
		User
		role string
	}

	func (self User) Vstring() string {
		return fmt.Sprintf("Vstring: %d, %v", self.id, self.name)
	}

	func main() {
		var m1 Stringer = Manager{
			User: User{1, "will"}, 	
			role: "Administrator",
		}

		fmt.Println(m1.Vstring())

		var m2 Stringer = &Manager{
			User: User{2, "will"}, 	
			role: "Administrator",
		}

		fmt.Println(m2.Vstring())
	}

代码输出：

	Vstring: 1, will
	Vstring: 2, will


再看一个例子，`Manager`类型包含`*User`类型：

	type Stringer interface {
		Vstring() string
		Pstring() string
	}

	type User struct {
		id   int
		name string
	}

	type Manager struct {
		*User
		role string
	}

	func (self User) Vstring() string {
		return fmt.Sprintf("Vstring: %d, %v", self.id, self.name)
	}

	func (self *User) Pstring() string {
		return fmt.Sprintf("Pstring: %d, %v", self.id, self.name)
	}

	func main() {
		var m1 Stringer = Manager{
			User: &User{1, "will"}, 	
			role: "Administrator",
		}

		fmt.Println(m1.Vstring())
		fmt.Println(m1.Pstring())


		var m2 Stringer = &Manager{
			User: &User{2, "will"}, 	
			role: "Administrator",
		}

		fmt.Println(m2.Vstring())
		fmt.Println(m2.Pstring())
	}

代码输出：

	Vstring: 1, will
	Pstring: 1, will
	Vstring: 2, will
	Pstring: 2, will


从两个例子中都可以看到，`User`类型实现了`Stringer`接口，由于`Manager`包含了`User`类型，所以`Manager`也相当于实现了`Stringer`接口。

