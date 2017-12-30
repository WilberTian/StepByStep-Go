Go中通过结构`struct`来表示聚合型的数据类型。



### 声明和初始化

结构类型是用来描述一组值的，要定义一个结构体的类型，通过`type`关键字和类型`struct`进行声明

例如定义一个结构体类型`person`，它有`age`，`name`这两个字段：

	type person struct {
		age int
		name string
	}


结构体类型定义好之后，就可以进行使用了，可以直接通过`var`关键字声明一个结构体类型的变量
	
	var p person

这种声明的方式，会对结构体`person`里的数据类型默认初始化，也就是使用它们类型的零值，如果要创建一个结构体变量并初始化其为零值时，这种`var`方式最常用。

如果需要指定非零值，就可以使用字面量方式：

	will := person{23, "Will"}

**注意这个值的顺序很重要，必须和结构体里声明字段的顺序一致**，当然也可以不按顺序，但是这时候必须使用`字段名: 字段值`这种键值对的方式：

	will := person{name: "Will", age: 23}



### 结构的赋值

Go语言中的结构是值类型，赋值和传参会复制全部内容。

函数传参是值传递，结构体传递的是其本身以及里面的值的拷贝。

	func main() {
		will := person{23, "Will"}

		fmt.Println(will)
		modify(will)
		fmt.Println(will)
	}

	func modify(p person) {
		p.age = p.age + 1
	}

	type person struct {
		age int
		name string
	}

代码输出：

	{23 Will}
	{23 Will}


通过上面例子的输出可以验证传递的是值的副本。如果要修改例子中的`age`值可以通过传递结构体的指针:

	func main() {
		will := person{23, "Will"}

		fmt.Println(will)
		modify(&will)
		fmt.Println(will)
	}

	func modify(p *person) {
		p.age = p.age + 1
	}

	type person struct {
		age int
		name string
	}

代码输出：

	{23 Will}
	{24 Will}




### 结构的`Tag`

由于Go语言对于可见性的特殊处理（首字母大写位包外可见），所以结构体中的字段名称需要遵循这种规则。

当需要将结构体跟`json`对象进行互相转换的时候，字段的名称或者大小写未必一致，所以Go为结构体成员增加了标签（`Tag`）。

例如，结构体对象转换成`json`字符串：

	func main() {
		type User struct {
	        UserId   int    `json:"user_id"`
	        UserName string `json:"user_name"`
	    }

	    u := &User{UserId: 1, UserName: "will"}
	    j, _ := json.Marshal(u)
	    
	    fmt.Println(string(j))
	}

代码输出：

	{"user_id":1,"user_name":"will"}


将`json`字符串转换为结构体对象：

	func main() {
		type User struct {
	        UserId   int    `json:"user_id"`
	        UserName string `json:"user_name"`
	    }

	    json_string := `{"user_id":1,"user_name":"will"}`

	    // user := &User{}
	    user := new(User)
	    // 将json转换为User结构类型对象
	  	json.Unmarshal([]byte(json_string), user)

	  	fmt.Println(user) 
	}

代码输出：
	
	&{1 will}



### 匿名字段

匿名字段是⼀种语法糖，从根本上说，就是⼀个与成员类型同名的字段。被匿名嵌⼊的可以是任何类型，当然也包括指针。

看一个例子：

	type User struct {
		name string
	}

	type Manager struct {
		User
		role string
	}

	m := Manager{
		User: User{"will"}, 	// 匿名字段的显式字段名，和类型名相同。
		role: "Administrator",
	}


Go中可以像访问普通字段那样访问匿名字段成员，编译器从外向内逐级查找所有层次的匿名字段，直到发现目标或出错。

	type Resource struct {
		id int
	}

	type User struct {
		Resource
		name string
	}

	type Manager struct {
		User
		role string
	}

	var m Manager
	m.id = 1
	m.name = "Will"
	m.role = "Administrator"


注意，外层同名字段会遮蔽嵌⼊字段成员，但是相同层次的同名字段会让编译器报错，解决⽅法是使⽤显式字段名。

