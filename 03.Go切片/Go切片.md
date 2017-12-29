在Go语言中，数组长度是固定不变的，为了增加灵活性，Go中提供了切片`slice`这种数据类型，可以根据需要自动改变大小，使用这种结构，可以更方便的管理和使用数据集合。



### 内部实现

表面上看，切片和数组非常相似，因为切片是基于数组实现的，它的底层是数组。

但是，本质上，`slice`并不是数组或数组指针。它通过内部指针和相关属性引⽤数组⽚段，以实现变⻓⽅案。

**切片对象非常小，是因为它是只有3个字段的数据结构**，这三个字段，就是Go语言操作底层数组的元数据：

- 一个是指向底层数组的指针
- 一个是切片的长度 
- 一个是切片的容量

Go源码中`slice`的结构对象：

	struct Slice
	{ 
		byte* array; 	// actual data
	    uintgo len; 	// number of elements
	    uintgo cap; 	// allocated number of elements
	};



### 声明和初始化

切片的声明和初始化通常使用`make`，同时可以指定切片的长度和容量。通过`make`创建切片时，如果不指定字面值，默认值就是数组的元素的零值。
	
	// len = 5, cap = 5
	slice := make([]int, 5)

	// len = 5, cap = 10
	slice := make([]int, 5, 10)


除了使用`make`，还可以使用字面量的方式直接声明并初始化切片：

	slice := []int{1, 2, 3, 4, 5}


跟数组类似，也可以直接使用索引来设置特定索引位置的值：

	slice := []int{4: 1}


切片还有`nil`切片和空切片，它们的长度和容量都是0，但是它们指向底层数组的指针不一样，`nil`切片意味着指向底层数组的指针为`nil`，而空切片对应的指针是个地址。

	//nil切片
	var nilSlice []int

	//空切片
	slice := []int{}


切片另外一个用的比较多的创建方式是基于现有的数组或者切片创建。

	slice := []int{1, 2, 3, 4, 5}
	slice1 := slice[:]
	slice2 := slice[0:]
	slice3 := slice[:5]

	fmt.Println(slice1)
	fmt.Println(slice2)
	fmt.Println(slice3)


当基于原数组或者切片创建一个新的切片后，对于底层数组容量是`k`的切片`slice[i:j]`来说:
	- 长度：j-i
	- 容量：k-i

例如： 

	slice := []int{1, 2, 3, 4, 5}
	newSlice := slice[1:3]

	fmt.Printf("newSlice长度:%d,容量:%d",len(newSlice),cap(newSlice))

比如我们上面的例子`slice[1:3]`,长度就是`3-1=2`，容量是`5-1=4`。

上面的例子是基于一个数组或者切片使用2个索引创建新切片的方法，此外还有一种3个索引的方法，第3个用来限定新切片的容量，其用法为`slice[i:j:k]`:

	slice := []int{1, 2, 3, 4, 5}
	newSlice := slice[1:2:3]

这样我们就创建了一个长度为`2-1=1`，容量为`3-1=2`的新切片，不过第三个索引，不能超过原切片的最大索引值`5`。



### 切片的赋值

切片是引⽤类型，通过内部实现指向底层数组。**但切片本质是结构体，所以赋值或者传参的时候，都是值拷⻉传递**。

从下面的代码可以看到，赋值后的切片跟原有的切片指向同样的底层数据：

	slice := []int{1, 2, 3, 4, 5}
	newSlice := slice[1:3]
	newSlice[0] = 10
		
	fmt.Println(slice)
	fmt.Println(newSlice)

代码输出：

	[1 10 3 4 5]
	[10 3]


因为切片是三个字段构成的结构类型，所以在函数间以值的方式传递的时候，占用的内存非常小，成本很低。

在赋值、传递切片的时候，其底层数组不会被复制，也不会受影响，复制只是复制的切片本身，不涉及底层数组。



### 使用切片

切片的使用很方便，可以直接通过`[i]`操作获取或者修改特定索引位置的值；也可以方便的通过`[i:j]`的操作生成一个新的切片。


#### `append`函数

切片算是一个动态数组，所以它可以按需增长，使用内置`append`函数即可。

`append`函数可以为一个切片追加一个元素，至于如何增加、返回的是原切片还是一个新切片、长度和容量如何改变这些细节，`append`函数会自动处理。

	slice := []int{1, 2, 3, 4, 5}
	newSlice := slice[1:3]
		
	newSlice = append(newSlice, 10)
	fmt.Println(newSlice)
	fmt.Println(slice)

代码输出：
	
	[2 3 10]
	[1 2 3 10 5]


**如果切片的底层数组，没有足够的容量时，就会新建一个底层数组，把原来数组的值复制到新底层数组里，再追加新值，这时候就不会影响原来的底层数组了**。

	func main() {
		arr := [...]int{1, 2, 3, 4, 5}

		s := arr[:3]
		fmt.Println(s)

		// 对于切片的修改，会影响底层数组arr
		s[1] = 100
		fmt.Println(s)
		fmt.Println(arr)

		s = append(s, 10, 11, 12, 13, 14, 15)

		fmt.Println(s)

		// 对于切片的修改，不会影响原始底层数组arr，因为append已经新建了一个底层数组
		s[2] = 200
		fmt.Println(s)
		fmt.Println(arr)
	}

代码输出：

	[1 2 3]
	[1 100 3]
	[1 100 3 4 5]
	[1 100 3 10 11 12 13 14 15]
	[1 100 200 10 11 12 13 14 15]
	[1 100 3 4 5]

内置的`append`也是一个可变参数的函数，所以可以同时追加好几个值:

	newSlice = append(newSlice, 10, 20, 30)


还可以通过`...`操作符，把一个切片追加到另一个切片里:

	slice := []int{1, 2, 3, 4, 5}
	newSlice := slice[1:2:3]
	newSlice=append(newSlice, slice...)

	fmt.Println(newSlice)
	fmt.Println(slice)


#### `copy`函数

函数`copy`在两个切片间复制数据，复制⻓度以`len`⼩的为准，两个切片可指向同⼀底层数组，允许元素区间重叠。

	data := [...]int{0, 1, 2, 3, 4, 5, 6, 7, 8, 9}
	s1 := data[8:]
	s2 := data[:5]

	copy(s2, s1) 

	fmt.Println(s2)
	fmt.Println(data)

输出：

	[8 9 2 3 4]
	[8 9 2 3 4 5 6 7 8 9]



### 切片遍历

对于切片的遍历可以使用`for`循环遍历，也可以直接使用`for range`遍历：

	slice := []int{1, 2, 3, 4, 5}

	for i,v:=range slice{
		fmt.Printf("索引:%d,值:%d\n",i,v)
	}



