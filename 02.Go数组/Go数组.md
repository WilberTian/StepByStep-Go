### 声明和初始化

var nums [6]int


var nums [6]int
nums = [6]int{1, 2, 3, 4, 5, 6}


nums := [6]int{1, 2, 3, 4, 5, 6}


nums := [...]int{1, 2, 3, 4, 5, 6}


array := [6]int{0, 1, 0, 4, 0, 0}
array := [6]int{1: 1, 3: 4}



### 数组遍历

func main() {
	array := [5]int{1: 1, 3: 4}
	for i := 0; i < 5; i++ {
		fmt.Printf("索引:%d,值:%d\n", i, array[i])
	}
}


func main() {
	array := [5]int{1: 1, 3: 4}
	for i, v := range array {
		fmt.Printf("索引:%d,值:%d\n", i, v)
	}
}


### 数组作为函数参数

同样类型的数组是可以相互赋值的，不同类型的不行，会编译错误。那么什么是同样类型的数组呢？Go语言规定，必须是长度一样，并且每个元素的类型也一样的数组，才是同样类型的数组。

array := [5]int{1: 1, 3: 4}
var array1 [5]int = array //success
var array2 [4]int = array1 //error


func main() {
	array := [5]int{1: 2, 3:4}
	modify(array)
	fmt.Println(array)
}
func modify(a [5]int){
	a[1] =3
	fmt.Println(a)
}


func main() {
	array := [5]int{1: 2, 3:4}
	modify(&array)
	fmt.Println(array)
}
func modify(a *[5]int){
	a[1] =3
	fmt.Println(*a)
}



