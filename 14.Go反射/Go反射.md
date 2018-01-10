反射提供了运行时获取对象类型和值的能力，在Go语言中，反射是由`reflect`包支持的，Go反射定义了两个重要的类型：`Type`和`Value`。

- Type is the representation of a Go type
- Value is the reflection interface to a Go value

下面看看Go反射相关的细节。




### 反射和接口

开始反射的`Type`和`Value`介绍之前，先回顾下Go语言中的接口类型。

在Go中，接口类型是一种抽象的类型，一个非常重要的接口类型就是空接口：

    interface{}

空接口可以保存任意值，在Go中空接口类型变量维护了一个`pair`，`pair`由两部分组成的，一个是接口的具体类型，一个是具体类型对应的值。

    var num int = 23
    i interface{} := num

对于上面的代码，变量`i`的`pair`表示就是`<Value, Type>`，其中`Value`为变量的值`23`，`Type`为变量的类型`int`。


下面回到反射的介绍，**Go反射其实就是一种检查存储在接口变量中的`<Value, Type>`对的机制**。

`reflect`包中提供两个函数`reflect.Typeof`和`reflect.Valueof`，这两个函数用来获取任意对象的`Type`和`Value`。




### `reflect.TypeOf`函数

函数`reflect.TypeOf`接受任意`interface{}`类型的对象，并返回对象对应的`Type`：

    func TypeOf(i interface{}) Type

通过下面的例子看看如何使用`reflect.TypeOf`函数获取对象的`Type`：

    import (
        "fmt"
        "reflect"
    )

    type User struct {
        Name string
        Age int
    }

    func main() {
        num := 24
        wilber := User{"Wilber", 23}

        fmt.Println(reflect.TypeOf(num))
        fmt.Println(reflect.TypeOf(wilber))

        fmt.Println(reflect.TypeOf(&num))
        fmt.Println(reflect.TypeOf(&wilber))
    }

代码输出为：

    int
    main.User
    *int
    *main.User





### `reflect.ValueOf`函数

函数`reflect.ValueOf`接受任意`interface{}`类型的对象，并返回对象的`Value`：

    func ValueOf(i interface{}) Value

看一段代码：

    import (
        "fmt"
        "reflect"
    )

    type User struct {
        Name string
        Age int
    }

    func main() {
        num := 24
        wilber := User{"Wilber", 23}

        fmt.Println(reflect.ValueOf(num))
        fmt.Println(reflect.ValueOf(wilber))

        fmt.Println(reflect.ValueOf(&num))
        fmt.Println(reflect.ValueOf(&wilber))
    }

代码输出为:

    24
    {Wilber 23}
    0xc4200160b0
    &{Wilber 23}


### `Type`的常用方法

在Go反射中，`Type`类型提供了很多方法，下面看几个比较常用的。

比如，可以通过`NumField()`和`NumMethod()`获得一个结构体类型的字段和方法：

    type User struct {
        Name string `json:"name"`
        Age  int `json:"age"`
    }

    func (self User) GetName() string {
        return self.Name
    }

    func main() {
        wilber := User{"Wilber", 23}

        t := reflect.TypeOf(wilber)

        for i := 0; i < t.NumField(); i++ {
            fmt.Println(t.Field(i).Name)
            fmt.Println(t.Field(i).Tag.Get("json"))
        }
        for i := 0; i < t.NumMethod(); i++ {
            fmt.Println(t.Method(i).Name)
        }
    }

代码输出为：

    Name
    name
    Age
    age
    GetName




### `Value`的常用方法

`Value`类型也提供了很多方法，下面看看几个比较常用的方法。



#### `Type()`和`Kind()`方法

`Type()`和`Kind()`这两个方法用来获取对象的类型，但是二者是有区别的，看下面代码：

    type User struct {
        Name string
        Age int
    }

    func main() {
        wilber := User{"Wilber", 23}

        v := reflect.ValueOf(wilber)

        fmt.Println(v.Type())
        fmt.Println(v.Kind())
    }

代码输出为：

    main.User
    struct

`Value`类型的`Type()`方法返回的是`reflect.Value`的`Type`，这个方法返回的是对象的静态类型，即`static type`，例子中就是`User`。

而`Kind()`方法返回的是对象的底层类型，即`underlying type`，例子中就是`struct`。



#### `Interface()`方法

对于`Value`类型，`Interface()`也是一个比较常用的方法，可以把`Value`类型转换成`interface{}`，相当于`reflect.ValueOf`的逆操作：

    func (v Value) Interface() (i interface{})

看一个例子：

    type User struct {
        Name string
        Age int
    }

    func main() {
        wilber := User{"Wilber", 23}

        v := reflect.ValueOf(wilber)

        fmt.Println(v.Interface())
    }



#### 通过反射修改对象

既然通过反射可以获得对象的`Value`，那么通过反射也可以对对象进行修改，看下面代码：

    type User struct {
        Name string
        Age  int
    }

    func main() {
        wilber := User{"Wilber", 23}

        v := reflect.ValueOf(wilber)

        fmt.Println(v.Field(0))             // 通过Field()方法和下标获取对于字段的值
        fmt.Println(v.FieldByName("Age"))   // 通过FieldByName()方法和字段名获取对于字段的值

        v.Field(0).SetString("Will")
    }

代码输出为：

    Wilber
    23
    panic: reflect: reflect.Value.SetString using unaddressable value

上面例子中，尝试修改对象的时候遇到了错误，是因为`reflect.ValueOf`函数返回的是一份值的拷贝。

在Go中，如果要修改反射类型对象，其值必须要是“addressable”的，对上面代码进行简单的修改：

    type User struct {
        Name string
        Age  int
    }

    func main() {
        wilber := User{"Wilber", 23}

        v1 := reflect.ValueOf(wilber)
        fmt.Println(v1.CanSet())        // Value的CanSet()方法用来判断反射对象是否可以修改

        pv := reflect.ValueOf(&wilber)
        v2 := pv.Elem()                 // 通过Elem()方法获取地址对应的值
        fmt.Println(v2.CanSet())

        v2.Field(0).SetString("Will")
        v2.FieldByName("Age").SetInt(24)

        fmt.Println(v2)
    }

代码输出为：

    false
    true
    {Will 24}




### Go语言反射的三条定律

- 反射可以将“接口类型变量”转换为“反射类型对象”
- 反射可以将“反射类型对象”转换为“接口类型变量”
- 如果要修改反射类型对象，其值必须要是“addressable”的

