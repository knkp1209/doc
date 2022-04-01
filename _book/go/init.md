```go
// 包的初始化
// 变量按声明顺序初始化，在依赖已解析完毕的情况下，根据依赖的顺序进行
// 包可由多个文件组成，任何文件可以包含任意数量 init 函数
// 函数 init 是不能被调用的和被引用的
// 方法与函数是不一样的，将方法命名为 init 是可以像正常方法使用的

var a = b + c // 最后把 a 初始化为 3
var b = f()   // 通过调用 f 接着把 b 初始化为 2
var c = 1     // 首先初始化为 1
func f() int { return c + 1 }


// 函数
func init() {
  
}

type P struct {
   a int
}

// 方法
func (p P) init(a int) {
   p.a = a
   fmt.Printf("%v", p.a)
}

func main() {
	var p1 = P{
		a: 2,
	}
	p1.init(10) // 方法名为 init 是可调用的
  init() // 函数名为 init 是不可调用的
}
```