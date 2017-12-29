Go语言中，通过`map`这种数据结构来存储无序的键值对集合。



### 声明和初始化

跟切片一样，`map`可以通过`make`函数声明和初始化，也可以直接使用字面量进行初始化。

	dict := make(map[string]int)


	dict := map[string]int{"will": 23}


如果不指定任何键值对，就是一个空map：
	
	dict := map[string]int{}


`nil`的`map`是未初始化的，只是声明了一个`map`类型的变量：

	var dict map[string]int

对于`nil`的`map`是不能操作存储键值对的，必须要初始化后才可以，比如使用`make`函数，为其申请一块可以存储数据的内存：

	var dict map[string]int

	dict = make(map[string]int)
	dict["will"] = 23
	fmt.Println(dict)

`map`的键可以是任何值，键的类型可以是内置的类型，也可以是结构类型，但是不管怎么样，这个键可以使用`==`运算符进行比较，所以像切片、函数以及含有切片的结构类型就不能用于`map`的键了，因为他们具有引用的语义，不可比较。



### `map`的赋值

跟切片一样，`map`也是引用类型，对`map`类型对象进行赋值或者传递函数参数，是不会拷贝一个该`map`的副本的。

也就是说对新的`map`变量做了修改，那么这个`map`的所有引用都会感知到这个修改。

	func main() {
		dict := map[string]int{"will": 23, "july": 20}
		modify(dict)
		fmt.Println(dict["will"])
	}

	func modify(dict map[string]int) {
		dict["will"] = 10
	}

代码输出：

	map[will:10 july:20]



### 使用`map`

`map`的使用很简单，和数组、切片差不多，只不过数组、切片是使用索引，`map`是通过键。

	dict := make(map[string]int)
	dict["will"] = 23

对于`map`，如果我们获取一个不存在的键的值，也是可以的，返回的是值类型的零值。

但是这样就会导致我们不知道是真的存在一个为零值的键值对呢，还是说这个键值对就不存在。对此，`map`为我们提供了检测一个键值对是否存在的方法。

	age, exists := dict["july"]


如果我们想删除一个map中的键值对，可以使用Go内置的`delete`函数。`delete`函数接受两个参数，第一个是要操作的`map`，第二个是要删除的`map`的键。
	
	delete(dict, "will")




### `map`遍历

想要遍历`map`的话，可以使用`for range`遍历：

	dict := map[string]int{"will": 23}

	for key, value := range dict {
		fmt.Println(key, value)
	}




### `map`值的特殊性

从`map`中取回的是⼀个`value`临时复制品，对其成员的修改是没有任何意义的。

例如下面的代码：

	func main() {
		type user struct{ name string }

		m := map[int]user{ 
		   1: {"will"}, 
		} 

		fmt.Println(m)

		m[1].name = "wilber"
	}

代码输出：

	cannot assign to struct field m[1].name in map

如果想要达到修改的目的，只能是完整替换`value`或使⽤指针:

完整替换：

	func main() {
		type user struct{ name string }

		m := map[int]user{ 
		   1: {"will"}, 
		} 

		m[1] = user{"july"}

		fmt.Println(m)
	}

使用指针：

	func main() {
		type user struct{ name string }

		m := map[int]*user{ 
		   1: &user{"will"}, 
		} 

		m[1].name = "july"

		fmt.Println(m)
	}


