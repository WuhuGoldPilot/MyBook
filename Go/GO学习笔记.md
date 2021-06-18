# GO学习笔记

## Go代码规范

### Getter

go不提供自动的getter和setter，你可以自己提供。需要注意的是，如果你有一个字段owner【小写，不导出的】，那么getter方法应该是Owner() 【大写，导出的】而不是GetOwner()，setter方法为常见的SetOwner()。

examples:

```go
owner := obj.Owner()
if owner != user {
    obj.SetOwner(user)
}
```

### Interface Names

单一方法接口由方法名称加上-er后缀或类似的修饰来构造名词：`Reader`,`Writer`,`Formatter`,`CloseNotifier`.

### MixedCaps

go中习惯使用`MixedCaps`和`mixedCaps`来写多字的复合名。

### Redeclaration and reassignment

```go
f, err := os.Open(name)
d, err := f.Stat()
```

对于err来说，虽然在第二行出现了短变量声明，但是实际上只是一个重赋值，而不是重新声明。`f.Stat`使用的是上面声明的err，并且重新赋值。

### Switch

switch的表达式不需要为常量或者整数，case从上到下计算，直至找到匹配，如果switch没有表达式，则默认为`trrue`。因此这很常用将`if-else`改写为`switch`。

```go
func unhex(c byte) byte {
    switch {
    case '0' <= c && c <= '9':
        return c - '0'
    case 'a' <= c && c <= 'f':
        return c - 'a' + 10
    case 'A' <= c && c <= 'F':
        return c - 'A' + 10
    }
    return 0
}
```

go的switch没有自动降级，但是`cases`可以用`,`分割的列表表示。

```go
func shouldEscape(c byte) bool {
    switch c {
    case ' ', '?', '&', '=', '#', '+', '%':
        return true
    }
    return false
}
```

尽管不像其他类C语言一样常见使用`break`，但是`break`也是可以停止一个`switch`。但是，有时候需要打断周围的循环，而不是`switch`。在go中，可以在循环上放置一个标签并`breaking`该标签来实现。example：

```go
Loop:
	for n := 0; n < len(src); n += size {
		switch {
		case src[n] < sizeOne:
			if validateOnly {
				break
			}
			size = 1
			update(src[n])

		case src[n] < sizeTwo:
			if n+1 >= len(src) {
				err = errShortInput
				break Loop
			}
			if validateOnly {
				break
			}
			size = 2
			update(src[n] + src[n+1]<<shift)
		}
	}
```

当然，`continue`也可以作为可选的标签，但是它只用于循环。

### Type  Switch

`switch`也可以用于发现接口变量的动态类型。这种*type switch*使用类型断言的语法，关键字`type`在括号里面。如果`switch`在表达式中声明了一个变量，则该变量在每个子句中都有对应的类型。在这种情况下重用名称是惯用的，实际上每种情况下都声明了一个具有相同名称但类型不同的新变量，

```go
var t interface{}
t = functionOfSomeType()
switch t := t.(type) {
default:
    fmt.Printf("unexpected type %T\n", t)     // %T prints whatever type t has
case bool:
    fmt.Printf("boolean %t\n", t)             // t has type bool
case int:
    fmt.Printf("integer %d\n", t)             // t has type int
case *bool:
    fmt.Printf("pointer to boolean %t\n", *t) // t has type *bool
case *int:
    fmt.Printf("pointer to integer %d\n", *t) // t has type *int
}
```

### Named result parameters

go函数的返回结果可以命名并作为常规变量，像传入参数一样。当被命名时，它们在函数开始时被初始化为它们的类型的零值，如果函数执行时不带参数的retrun语句，则结果参数的当前值将用作返回值。

命名不是强制性的，但是它们可以使代码更短更清晰，它们是文档，如果我们命名`nextInt`的结果，我们可以知道返回的`int`是哪个。

```go
func nextInt(b []byte, pos int) (value, nextPos int) {
```

因为命名结果已初始化并绑定到未修饰的返回值，它们可以简单和清晰。example：

```go
func ReadFull(r Reader, buf []byte) (n int, err error) {
    for len(buf) > 0 && err == nil {
        var nr int
        nr, err = r.Read(buf)
        n += nr
        buf = buf[nr:]
    }
    return
}
```

### Defer

go的`defer`语句会在`return`之前执行`defer`的函数的回调，这是一种不寻常但是有效的方法去处理诸如资源释放的情况，无论函数在哪条路径返回。example：

```go
// Contents returns the file's contents as a string.
func Contents(filename string) (string, error) {
    f, err := os.Open(filename)
    if err != nil {
        return "", err
    }
    defer f.Close()  // f.Close will run when we're finished.

    var result []byte
    buf := make([]byte, 100)
    for {
        n, err := f.Read(buf[0:])
        result = append(result, buf[0:n]...) // append is discussed later.
        if err != nil {
            if err == io.EOF {
                break
            }
            return "", err  // f will be closed if we return here.
        }
    }
    return string(result), nil // f will be closed if we return here.
}
```

定义一个`defer`回调函数有两个好处：一是它保证不会让你忘记关闭文件，如果你修改你的方法增加一条新的返回路径，你很容易忘记这件事；二是这意味着你需要将它定义在`open`旁边，这比定义在函数结尾更加清晰。

延迟函数的参数（如果函数是方法，则包含接收者）在延迟执行时计算，而不是在调用执行时计算。除了避免担心在执行时变量会改变值，这意味着单个延迟调用可以延迟多个函数执行。延迟函数以`LIFO`的顺序执行，example：

```go
func trace(s string) string {
    fmt.Println("entering:", s)
    return s
}

func un(s string) {
    fmt.Println("leaving:", s)
}

func a() {
    defer un(trace("a"))
    fmt.Println("in a")
}

func b() {
    defer un(trace("b"))
    fmt.Println("in b")
    a()
}

func main() {
    b()
}
```

输出：

```go
entering: b
in b
entering: a
in a
leaving: a
leaving: b
```

对于习惯其他语言的块级管理资源的程序员来说，`defer`可能看起来很奇怪，但它最强大和最有趣的应用程序恰恰来自它不是基于块的而是基于函数的事实。

### Data

### Allocation with `new` 

go有两个分配内存的原语：`new`和`make`，它们做不同的事且适用不同的对象。

`new`是一个分配内存的内置函数，但与其他语言的同名函数不同，它不会初始化内存，只会将其归零。因此`new(T)`为类型为`T`的新项分配零存储并返回其地址，类型为`*T`的值。在go术语中，它返回一个指向新分配的`T`类型零值的指针。

由于`new`返回的内存已归零，因此在设计数据结构时安排每种类型的零值无需进一步初始化即可使用是有帮助的。这意味着数据结构的用户可以使用`new`创建一个并立即可以使用它。

零值有用的属性是传递性的，example：

```go
type SyncedBuffer struct {
    lock    sync.Mutex
    buffer  bytes.Buffer
}
```

`SyncedBuffer`类型的值可以在分配或者声明后直接使用。在下一个片段中，`p`和`v`无需进一步安排即可正常工作。

```go
p := new(SyncedBuffer)  // type *SyncedBuffer
var v SyncedBuffer      // type  SyncedBuffer
```

### Allocation with `make`

内置函数`make(T,args)`的用途不同于`new(T)`，它只创建切片，映射和通道，它返回一个类型为`T`（不是`*T`）

的初始化（未清零）值。区分的原因是这三种类型在幕后在使用前必须初始化的数据结构引用

。例如切片是一个三项描述符，包含指向数据（在数组内）、长度和容量的指针，并且在这些项被初始化之前，切片为`nil`。对于切片、映射和通道，`make`会初始化内部数据结构并准备要使用的值，例如：

```go
make([]int, 10, 100)
```

分配一个包含100个整数的数组，然后创建一个长度为10、容量为100的切片结构，指向数组的前10个元素。相比之下，`new([]int))`返回一个指向新分配的、归零的切片结构的指针，即指向`nil`切片值的指针。下面是关于`new`和`make`的不同之处的例子：

```go
var p *[]int = new([]int)       // allocates slice structure; *p == nil; rarely useful
var v  []int = make([]int, 100) // the slice v now refers to a new array of 100 ints

// Unnecessarily complex:
var p *[]int = new([]int)
*p = make([]int, 100, 100)

// Idiomatic:
v := make([]int, 100)
```

记得`make`只用于映射、切片、和通道并且不返回指针，使用`new`获得显式指针分配或显式获取变量的地址。

### Arrays

数组在规划内存的详细布局时很有用，有时可以帮助避免分配，但主要是切片的构造块。数组在C和go中有很大的不同，在go中：

- 数组是值，将一个数组分配给另一个会复制所有的值；

- 特别是，如果你将一个数组传给一个函数，他会收到一个数组的副本，而不是一个指向它的指针；

- 数组的大小是其类型的一部分，`[10]int`和`[20]int`的类型是不同的。

  

`value`属性可以很有用也可以很昂贵，如果你想要类似C的行为和效率，你可以传递一个指向数组的指针：

```go
func Sum(a *[3]float64) (sum float64) {
    for _, v := range *a {
        sum += v
    }
    return
}

array := [...]float64{7.0, 8.5, 9.1}
x := Sum(&array)  // Note the explicit address-of operator
```

但即使是这种风格也不是go惯用的，改用切片更加好。

### Slices 

切片包装数组，为数据序列化提供更通用、更强大、更方便的接口。除了具有显式维度的项（例如转换矩阵），go中的大多数数组编程都是使用切片而不是简单数组完成的。

切片保存对底层数组的引用，如果将一个切片分配给另一个切片，则它们都引用同一个数组。如果一个函数接受一个切片参数，它对切片元素所做的更改将对调用者可见，类似于传递一个指向底层数组的指针。因此，`Read`函数可以接受切片参数而不是指针和计数，切片内的长度设置了要读取的数据量的上限。下面是在`os`包的`File`类型的`Read`方法的签名：

```go
func (f *File) Read(buf []byte) (n int, err error)
```

该方法返回读取的字节数和错误值（如果有），要读入较大缓冲区的前32个字节，请对缓冲区进行切片：

```go
n, err := f.Read(buf[0:32])
```

这种切片很常见也很高效。切片的长度可以改变，只要它仍然适合底层数组的限制。只需将其分配给自身的一部分，切片的容量，可通过内置函数`cap`访问，报告切片可能采用的最大长度。下面是一个将数据附加到切片的函数，如果数据超出容量，则重新分配切片，返回结果切片。该函数使用`len`和`cap`在应用于`nil`切片时合法的事实，并返回0.

```go
func Append(slice, data []byte) []byte {
    l := len(slice)
    if l + len(data) > cap(slice) {  // reallocate
        // Allocate double what's needed, for future growth.
        newSlice := make([]byte, (l+len(data))*2)
        // The copy function is predeclared and works for any slice type.
        copy(newSlice, slice)
        slice = newSlice
    }
    slice = slice[0:l+len(data)]
    copy(slice[l:], data)
    return slice
}

```

之后我们必须返回切片，因为虽然`Append`可以修改切片的元素，但是切片本身（保存指针、长度和容量的运行时数据结构）是按值传递的。附加数据都切片中是非常有用的，它被`append`内置函数捕获。

### Two-dimensional slices

go的数组和切片是一维的，要创建二维数组或切片的等效项，必须定义一个数组数组或切片数组，如下所示：

```go
type Transform [3][3]float64  // A 3x3 array, really an array of arrays.
type LinesOfText [][]byte     // A slice of byte slices.	
```

因为切片是可变长度的，所以可以让每个内部切片的长度不同。这是一个常见的情况，如下例子所示，每行都有一个独立的长度：

```go
text := LinesOfText{
	[]byte("Now is the time"),
	[]byte("for all good gophers"),
	[]byte("to bring some fun to the party."),
}
```

有时候分配一个2D切片是有必要的，有两个方法可以实现它。一种是独立分配每个切片；另一种是分配单个数组并将各个切片指向其中。使用哪种方式取决于你的应用，如果切片可能会增长或者缩小，则应单独分配以避免覆盖下一行；如果没有，使用单个分配构造对象会更有效。下面是两个分配方式的例子：

一次一行：

```go
// Allocate the top-level slice.
picture := make([][]uint8, YSize) // One row per unit of y.
// Loop over the rows, allocating the slice for each row.
for i := range picture {
	picture[i] = make([]uint8, XSize)
}
```

一个分配，分成几行：

```go
// Allocate the top-level slice, the same as before.
picture := make([][]uint8, YSize) // One row per unit of y.
// Allocate one large slice to hold all the pixels.
pixels := make([]uint8, XSize*YSize) // Has type []uint8 even though picture is [][]uint8.
// Loop over the rows, slicing each row from the front of the remaining pixels slice.
for i := range picture {
	picture[i], pixels = pixels[:XSize], pixels[XSize:]
}
```

### Maps

映射是一种方便且强大的内置数据结构，它将一种类型（键）的值与另一种类型（元素或值）的值相关联。
键可以是定义了相等运算符的任何类型，例如整数、浮点数和复数、字符串、指针、接口（只要动态类型支持相等）、结构和数组。切片不能用作映射键，因为它们没有定义相等性。像切片一样，映射保存对底层数据结构的引用。如果将`map`传递给更改`map`内容的函数，则更改将在调用方中可见。
可以使用带有冒号分隔的键值对的常用复合文字语法构建映射，因此在初始化期间很容易构建它们。

```go
var timeZone = map[string]int{
    "UTC":  0*60*60,
    "EST": -5*60*60,
    "CST": -6*60*60,
    "MST": -7*60*60,
    "PST": -8*60*60,
}
```

分配和获取映射值在语法上看起来就像对数组和切片执行相同的操作，只是索引不需要是整数。

```go
offset := timeZone["EST"]
```

尝试使用映射中不存在的键获取映射值将返回映射中类型的零值。例如，如果映射包含整数，查找不存在的键将返回 0。集合可以实现为值类型为 bool 的映射。将映射条目设置为 true 以将值放入集合中，然后通过简单的索引对其进行测试。

```go
attended := map[string]bool{
    "Ann": true,
    "Joe": true,
    ...
}

if attended[person] { // will be false if person is not in the map
    fmt.Println(person, "was at the meeting")
}
```

有时您需要将缺失的值与零值区分开来。是否有“UTC”的条目或者是 0，因为它根本不在`map`中？你可以用多重赋值的形式来区分。

```go
var seconds int
var ok bool
seconds, ok = timeZone[tz]
```

出于显而易见的原因，这被称为“逗号确定”习语。在这个例子中，如果 tz 存在，秒将被适当地设置并且 ok 为真；如果没有，秒将被设置为零，确定将是假的。这是一个将它与一个很好的错误报告放在一起的函数：

```go
func offset(tz string) int {
    if seconds, ok := timeZone[tz]; ok {
        return seconds
    }
    log.Println("unknown time zone:", tz)
    return 0
}
```

要在不担心实际值的情况下测试`map`中是否存在，您可以使用空白标识符 (_) 代替该值的常用变量。

```go
_, present := timeZone[tz]
```

要删除`map`的值，请使用 delete 内置函数，其参数是要删除的`map`和键。

```go
delete(timeZone, "PDT")  // Now on Standard Time
```

### Printing 

go的格式化打印和c的`printf`类似但是比它更丰富和更普遍。这些方法放在`fmt`包中，并且有大写字母的名字：`fmt.Printf`, `fmt.Fprintf`, `fmt.Sprintf`等。字符串函数（`Sprintf`等）返回一个字符串而不是填充提供的缓冲区。
你不需要提供格式化字符串，对于`Printf`, `Fprintf`和`Sprintf`中的每一个，都有另一对函数，例如`Print`和`Println`。这些函数不采用格式化字符串，而是为每个参数生产默认格式。`Println`函数在每个参数之前插入一个空格并在输出中附加一个换行符，而`Print`仅当两边参数都是字符串时才添加空格。下面这个例子，每一行的输出都是一样的：

```go
fmt.Printf("Hello %d\n", 23)
fmt.Fprint(os.Stdout, "Hello ", 23, "\n")
fmt.Println("Hello", 23)
fmt.Println(fmt.Sprint("Hello ", 23))
```

格式化打印函数`fmt.Fprintf`和他们的同类将任何实现`io.Writer`接口的对象作为第一个参数，变量`os.Stout`和`os.Stderr`是熟悉的实例。
下面是不同于C的部分。首先，诸如%d之类的数字格式不采用符号或大小的标志；其次，打印线程使用参数的类型来决定这些属性。

```go
var x uint64 = 1<<64 - 1
fmt.Printf("%d %x; %d %x\n", x, x, int64(x), int64(x))
```

打印：

```go
18446744073709551615 ffffffffffffffff; -1 -1
```

如果你只想要默认的转换，比如整数的十进制，你可以使用笼统的格式`%v`（value），结果正是`Print`和`Println`所产生的结果。此外，此格式可以打印任何值，包括数组，切片，结构和映射。这是上一节定义时区地图的打印语句：

```go
fmt.Printf("%v\n", timeZone)  // or just fmt.Println(timeZone)
```

输出：

```go
map[CST:-21600 EST:-18000 MST:-25200 PST:-28800 UTC:0]
```

对于映射，`Printf`和它相似的方法按字典排序对输出进行排序。
打印结构体时，修改后的格式`%+v`用它们的名称注释结构体的字段，对于任何值，替代格式`%#v`以完整的go语法打印值。

```go
type T struct {
    a int
    b float64
    c string
}
t := &T{ 7, -2.35, "abc\tdef" }
fmt.Printf("%v\n", t)
fmt.Printf("%+v\n", t)
fmt.Printf("%#v\n", t)
fmt.Printf("%#v\n", timeZone)
```

打印：

```go
&{7 -2.35 abc   def}
&{a:7 b:-2.35 c:abc     def}
&main.T{a:7, b:-2.35, c:"abc\tdef"}
map[string]int{"CST":-21600, "EST":-18000, "MST":-25200, "PST":-28800, "UTC":0}
```

(注意&符号)当应用于`string`或`[]byte`类型的值时，引用的字符串格式也可以通过`%q`获得。如果可能，替代格式`%#q`将使用反引号（`q%`格式也适用于整数和`rune`，生成单引号`rune`常量）。此外。`%x`适用于字符串，字节数组和字节切片以及整数，生成一个长的十六进制字符串，也可以使用`% x`格式在每个字节之中放置空格。

另一种便利的格式`%T，它打印值的类型。

```go
fmt.Printf("%T\n", timeZone)
```

打印：

```go
map[string]int
```

如果你想要控制自定义类型的默认格式，只需要在类型上定义一个带有签名`String() string`的方法。对于简单类型`T`，可能如下：

```go
func (t *T) String() string {
    return fmt.Sprintf("%d/%g/%q", t.a, t.b, t.c)
}
fmt.Printf("%v\n", t)
```

打印：

```go
7/-2.35/"abc\tdef"
```

如果你需要打印`T`类型的值以及指向`T`的指针，则`String`的接收者必须是值类型，这个例子使用了一个指针，因为它对于结构类型更加有效和惯用。
我们的`String`方法可以调用`Sprintf`是因为打印线程是完全可重入的并且可以以这种方式包装。然而，关于这种方法有一个重要的细节需要理解：不要通过调用`Sprintf`会循环调用你的`String`方法去构造一个`String`方法。如果`Sprintf`调用尝试将接收器直接打印为字符串，则可能发生这种情况，而后者又会再次调用该方法。如下例所示：这是一个常见且容易犯的错误：

```go
type MyString string

func (m MyString) String() string {
    return fmt.Sprintf("MyString=%s", m) // Error: will recur forever.
}
```

修复也很容易：将参数转换为基本字符串类型，它没有方法调用。

```go
type MyString string
func (m MyString) String() string {
    return fmt.Sprintf("MyString=%s", string(m)) // OK: note conversion.
}
```

```go
func Printf(format string, v ...interface{}) (n int, err error) {}
```

在函数`Printf`中，v的作用类似于`[]inteface{}`类型的变量，但如果将其传递给另一个可变参数函数，它的作用类似于常规参数列表。下面的例子是我们使用上面的函数实现的，它将参数直接传递给`fmt.Sprintln`以实现格式化：

```go
// Println prints to the standard logger in the manner of fmt.Println.
func Println(v ...interface{}) {
    std.Output(2, fmt.Sprintln(v...))  // Output takes parameters (int, string)
}
```

我们在对`Sprintln`的嵌套调用中在v之后写入`...`告诉编译器将v视为参数列表，否则它只会将v作为单个切片参数传递。

`...`参数也可以指定类型，例如`...int`用于选择整数列表中最小者的`min`函数：

```go
func Min(a ...int) int {
    min := int(^uint(0) >> 1)  // largest int
    for _, i := range a {
        if i < min {
            min = i
        }
    }
    return min
}
```

### Append

`append`的签名与我们上面自定义的`Append`函数不同：

```go
func append(slice []T, elements ...T) []T
```

其中`T`是任何给定类型的占位符，你实际上无法在go中编写类型`T`由调用者确定的函数。这就是`append`内置的原因：它需要编译器的支持。
`append`的作用就是将元素追加到切片的末尾并返回结果，结果需要返回是因为与我们手写的`Append`一样，底层数组可能会改变。一个简单的例子：

```go
x := []int{1,2,3}
x = append(x, 4, 5, 6)
fmt.Println(x)
```

打印`[1 2 3 4 5 6]`。所以`append`和`Printf`一样，接受任意数量的参数。

