### 声明和初始化

Map的创建有make函数，Map字面量。

dict := make(map[string]int)


dict := map[string]int{"张三":43}


当然我们可以不指定任何键值对，也就是一个空map。
dict := map[string]int{}


nil的Map是未初始化的，所以我们可以只声明一个变量，既不能使用map字面量，也不能使用make函数分配内存。
var dict map[string]int
这样就好了，但是这样我们是不能操作存储键值对的，必须要初始化后才可以，比如使用make函数,为其开启一块可以存储数据的内存，也就是初始化。


Map的键可以是任何值，键的类型可以是内置的类型，也可以是结构类型，但是不管怎么样，这个键可以使用==运算符进行比较，所以像切片、函数以及含有切片的结构类型就不能用于Map的键了，因为他们具有引用的语义，不可比较。



### 使用map

在Go Map中，如果我们获取一个不存在的键的值，也是可以的，返回的是值类型的零值，这样就会导致我们不知道是真的存在一个为零值的键值对呢，还是说这个键值对就不存在。对此，Map为我们提供了检测一个键值对是否存在的方法。

age, exists := dict["李四"]


如果我们想删除一个Map中的键值对，可以使用Go内置的delete函数。
delete(dict,"张三")
delete函数接受两个参数，第一个是要操作的Map，第二个是要删除的Map的键。


### map遍历

想要遍历Map的话，可以使用for range风格的循环

dict := map[string]int{"张三": 43}
for key, value := range dict {
	fmt.Println(key, value)
}



### map作为函数参数


函数间传递Map是不会拷贝一个该Map的副本的，也就是说如果一个Map传递给一个函数，该函数对这个Map做了修改，那么这个Map的所有引用，都会感知到这个修改。

func main() {
	dict := map[string]int{"王五": 60, "张三": 43}
	modify(dict)
	fmt.Println(dict["张三"])
}
func modify(dict map[string]int) {
	dict["张三"] = 10
}