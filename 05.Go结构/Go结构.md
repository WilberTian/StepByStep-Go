### 声明和初始化


结构类型是用来描述一组值的，要定义一个结构体的类型，通过type关键字和类型struct进行声明，以上我们就定义了一个结构体类型person,它有age,name这两个字段数据。

type person struct {
	age int
	name string
}


结构体类型定义好之后，就可以进行使用了，我们可以用过var关键字声明一个结构体类型的变量。
var p person

这种声明的方式，会对结构体person里的数据类型默认初始化，也就是使用它们类型的零值，如果要创建一个结构体变量并初始化其为零值时，这种var方式最常用。


如果我们需要指定非零值，就可以使用我们字面量方式了。

will := person{23, "Will"}

示例这种我们就为其指定了值，注意这个值的顺序很重要，必须和结构体里声明字段的顺序一致，当然我们也可以不按顺序，但是这时候我们必须为字段指定值。

will := person{name: "Will", age: 23}
使用冒号:分开字段名和字段值即可，这样我们就不用严格的按照定义的顺序了。


### 结构作为函数参数

函数传参是值传递，所以对于结构体来说也不例外，结构体传递的是其本身以及里面的值的拷贝。

func main() {
	will := person{23, "Will"}
	fmt.Println(will)
	modify(will)
	fmt.Println(will)
}
func modify(p person) {
	p.age =p.age + 23
}
type person struct {
	age int
	name string
}

以上示例的输出是一样的，所以我们可以验证传递的是值的副本。如果上面的例子我们要修改age的值可以通过传递结构体的指针，我们稍微改动下例子

func main() {
	will := person{23, "Will"}
	fmt.Println(will)
	modify(&will)
	fmt.Println(will)
}
func modify(p *person) {
	p.age =p.age + 23
}
type person struct {
	age int
	name string
}


### struct tag



