### 内部实现

切片是基于数组实现的，它的底层是数组，它自己本身非常小，可以理解为对底层数组的抽象。

切片对象非常小，是因为它是只有3个字段的数据结构：一个是指向底层数组的指针，一个是切片的长度，一个是切片的容量。这3个字段，就是Go语言操作底层数组的元数据，有了它们，我们就可以任意的操作切片了。


### 声明和初始化

slice := make([]int, 6)

slice := make([]int, 5, 10)


slice := []int{1, 2, 3, 4, 5}

slice := []int{4: 1}



//nil切片
var nilSlice []int
//空切片
slice:=[]int{}


切片另外一个用处比较多的创建是基于现有的数组或者切片创建。

slice := []int{1, 2, 3, 4, 5}
slice1 := slice[:]
slice2 := slice[0:]
slice3 := slice[:5]
fmt.Println(slice1)
fmt.Println(slice2)
fmt.Println(slice3)


我们基于原数组或者切片创建一个新的切片后，那么新的切片的大小和容量是多少呢？这里有个公式：

对于底层数组容量是k的切片slice[i:j]来说
长度：j-i
容量:k-i

比如我们上面的例子slice[1:3],长度就是3-1=2，容量是5-1=4。不过代码中我们计算的时候不用这么麻烦，因为Go语言为我们提供了内置的len和cap函数来计算切片的长度和容量。

slice := []int{1, 2, 3, 4, 5}
newSlice := slice[1:3]
fmt.Printf("newSlice长度:%d,容量:%d",len(newSlice),cap(newSlice))
以上基于一个数组或者切片使用2个索引创建新切片的方法，此外还有一种3个索引的方法，第3个用来限定新切片的容量，其用法为slice[i:j:k]。


slice := []int{1, 2, 3, 4, 5}
newSlice := slice[1:2:3]
这样我们就创建了一个长度为2-1=1，容量为3-1=2的新切片,不过第三个索引，不能超过原切片的最大索引值5。



### 使用切片

切片算是一个动态数组，所以它可以按需增长，我们使用内置append函数即可。append函数可以为一个切片追加一个元素，至于如何增加、返回的是原切片还是一个新切片、长度和容量如何改变这些细节，append函数都会帮我们自动处理。

slice := []int{1, 2, 3, 4, 5}
newSlice := slice[1:3]
	
newSlice = append(newSlice, 10)
fmt.Println(newSlice)
fmt.Println(slice)
//Output
[2 3 10]
[1 2 3 10 5]


如果切片的底层数组，没有足够的容量时，就会新建一个底层数组，把原来数组的值复制到新底层数组里，再追加新值，这时候就不会影响原来的底层数组了。



内置的append也是一个可变参数的函数，所以我们可以同时追加好几个值。

newSlice = append(newSlice, 10, 20, 30)


我们还可以通过...操作符，把一个切片追加到另一个切片里。

slice := []int{1, 2, 3, 4, 5}
newSlice := slice[1:2:3]
newSlice=append(newSlice, slice...)
fmt.Println(newSlice)
fmt.Println(slice)


### 切片遍历

slice := []int{1, 2, 3, 4, 5}
for i,v:=range slice{
	fmt.Printf("索引:%d,值:%d\n",i,v)
}


### 切片作为函数参数

我们知道切片是3个字段构成的结构类型，所以在函数间以值的方式传递的时候，占用的内存非常小，成本很低。在传递复制切片的时候，其底层数组不会被复制，也不会受影响，复制只是复制的切片本身，不涉及底层数组。


func main() {
	slice := []int{1, 2, 3, 4, 5}
	fmt.Printf("%p\n", &slice)
	modify(slice)
	fmt.Println(slice)
}
func modify(slice []int) {
	fmt.Printf("%p\n", &slice)
	slice[1] = 10
}

打印的输出如下：

0xc420082060
0xc420082080
[1 10 3 4 5]