## GO语言介绍

### 1. 下载与安装

官网：https://golang.org

国内下载：https://studygolang.com/dl

国内镜像配置：https://goproxy.cn/

### 2. 基础语法

#### 2.1 变量定义

```go
package main

import "fmt"

// 包内部变量，不能使用 :=
// var aa = 3
// var ss = "kkk"
// var bb = true
var (
	aa = 3
	ss = "kkk"
	bb = true
)

// 变量定义，%q输出空字符串为""
//定义变量不设置初始值
func variableZeroValue() {
	var a int
	var s string
	fmt.Printf("%d %q\n", a, s)
}

// 变量默认初始值
func variableInitValue() {
	var a, b int = 3, 4
	var s string = "abc"
	fmt.Print(a, s, b)
}

// 编译器自动识别类型
func variableTypeDeduction() {
	var a, b, c, s = 3, 4, true, "def"
	fmt.Print(a, s, b, c, s)
}

// x := 相当于 var x =
func variableShorter() {
	a, b, c, s := 3, 4, true, "def"
	b = 5
	fmt.Print(a, s, b, c, s)
}

func main() {
	fmt.Print("Hello World")
	variableZeroValue()
	variableInitValue()
	variableTypeDeduction()
	variableShorter()
	fmt.Print(aa, bb, ss)
}
```

#### 2.2 内建变量类型

- bool，string
- (u)int，(u)int8，(u)int16，(u)int32，(u)int64，uintptr
  - u：表示无符号
  - 8，16等：规定长度。无符号（int）按照操作系统的长度来定，64位操作系统位64，32位操作系统位32
  - uintptr中ptr表示的指针，指针长度由操作系统决定
- byte，rune
  - rune：表示字符型，go语言的char，rune长度是32位，byte是8位
- float32，float64，complex64，complex128
  - complex64中实部和虚部是32位的，在complex128中实部和虚部是64位的。**复数**。

复数是二维平面中的数字

![image-20230214201523292](https://winnxudong.oss-cn-shanghai.aliyuncs.com/images/image-20230214201523292.png)

![image-20230214202641249](https://winnxudong.oss-cn-shanghai.aliyuncs.com/images/image-20230214202641249.png)

```go
// 欧若拉公式
func euler() {
	// 复数计算
	c := 3 + 4i
	fmt.Print(cmplx.Abs(c))
	// 欧若拉公式
	fmt.Print(cmplx.Pow(math.E, 1i*math.Pi) + 1)
}
```

#### 2.3  强制类型转换

- 类型转换是强制的

```go
func triangle() {
	var a, b int = 3, 4
	var c int
	c = int(math.Sqrt(float64(a * a + b * b)))
	fmt.Print(c)
}
```

#### 2.4 常量

- const 数值可作为各种类型使用

```go
const filename = "abc.txt"

func consts() {
	const a, b = 3, 4
	var c int
	c = int(math.Sqrt(a*a + b*b))
	fmt.Println(filename, c)
}

func consts() {
	const (
		a, b = 3, 4
	)
	var c int
	c = int(math.Sqrt(a*a + b*b))
	fmt.Println(filename, c)
}
```

#### 2.5 枚举

- 普通枚举类型
- 自增枚举类型

```go
func enums() {
	const (
		cpp = 0
		java = 1
		python = 2
		golang = 3
	)
	fmt.Println(cpp, java, python, golang)
}
====等价
func enums() {
	const (
		cpp = iota
		java
		python
		golang
	)
	fmt.Println(cpp, java, python, golang)
}
====iota表示从0自增

func enums() {
	const (
		cpp = iota
		_
		java
		python
		golang
	)
	fmt.Println(cpp, java, python, golang)

	// b, kb, mb, gb, tb, pb
	const (
		b = 1 << (10 * iota)
		kb
		mb
		gb
		tb
		pb
	)
}
```

![image-20230214204729930](https://winnxudong.oss-cn-shanghai.aliyuncs.com/images/image-20230214204729930.png)

#### 2.6 条件语句

##### if语句

- if条件里不需要括号
- if的条件里可以赋值
- if条件里赋值的变量作用域就在这个if语句中

```go
// 方式一
func main() {
	const filename = "abc.txt"
	contents, err := ioutil.ReadFile(filename)
	if err != nil {
		fmt.Println(err)
	} else {
		fmt.Printf("%s\n", contents)
	}
}

// 方式二
func main() {
	const filename = "abc.txt"
	if contents, err := ioutil.ReadFile(filename); err != nil {
		fmt.Println(err)
	} else {
		fmt.Printf("%s\n", contents)
	}
}
```

##### switch语句

- switch会自动break，除非使用fallthrough会让switch继续执行
- switch后可以没有表达式

```go
// 传入一个分数返回一个string字符串
func grade(score int) string {
	g := ""
	switch {
	case score < 60:
		g = "F"
	case score < 80:
		g = "C"
	case score < 90:
		g = "B"
	case score < 100:
		g = "A"
	default:
		panic(fmt.Sprintf("Wrong score: %d", score))
	}
	return g
} 
```

#### 2.7 循环

- for的条件里不需要括号
- for的条件里可以省略初始条件，结束条件，递增表达式

```go
// 二进制转换 忽略初始条件
func convertToBin(n int) string {
	result := ""
	for ; n > 0; n /= 2 {
		lsb := n % 2
		result = strconv.Itoa(lsb) + result
	}
	return result
}

// 忽略起始条件与递增条件 相当于while
func printFile(filename string) {
	file, err := os.Open(filename)
	if err != nil {
		panic(err)
	}

	scanner := bufio.NewScanner(file)

	for scanner.Scan() {
		fmt.Println(scanner.Text())
	}
}

// 忽略结束条件无限循环
func forever() {
	for  {
		fmt.Println("abc")
	}
}

func main() {
	fmt.Println(
		convertToBin(5),
		convertToBin(13),
	)
	printFile("abc.txt")
}
```

#### 2.8 函数

- 函数可以返回多个值（div）
  - 返回多个值时可以起名字
  - 仅用于非常简单的函数
  - 对于调用者而言没有区别

- 函数可作为参数（apply）
- 函数参数可设置可变参数列表
- 函数返回值类型写在最后面
- 函数没有默认参数，可选参数

