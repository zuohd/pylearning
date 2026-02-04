# Day 27：Go 函数 + Struct
> 学习目标：掌握 Go 函数定义和结构体使用
>

---

## 1. 函数基础
```go
package main

import "fmt"

// 基本函数
func greet() {
    fmt.Println("Hello!")
}

// 带参数
func add(a int, b int) int {
    return a + b
}

// 参数类型简写
func multiply(a, b int) int {
    return a * b
}

// 多返回值
func divide(a, b int) (int, int) {
    return a / b, a % b
}

// 命名返回值
func calculate(a, b int) (sum int, diff int) {
    sum = a + b
    diff = a - b
    return  // 自动返回命名的变量
}

func main() {
    greet()
    fmt.Println(add(3, 5))
    fmt.Println(multiply(4, 6))

    quotient, remainder := divide(10, 3)
    fmt.Printf("商: %d, 余数: %d\n", quotient, remainder)

    s, d := calculate(10, 3)
    fmt.Printf("和: %d, 差: %d\n", s, d)
}
```

---

## 2. 可变参数
```go
package main

import "fmt"

// 可变参数
func sum(nums ...int) int {
    total := 0
    for _, n := range nums {
        total += n
    }
    return total
}

// 混合参数
func printf(format string, args ...interface{}) {
    fmt.Printf(format, args...)
}

func main() {
    fmt.Println(sum(1, 2, 3))
    fmt.Println(sum(1, 2, 3, 4, 5))

    // 切片展开
    nums := []int{1, 2, 3, 4, 5}
    fmt.Println(sum(nums...))

    printf("Name: %s, Age: %d\n", "Tom", 20)
}
```

---

## 3. 匿名函数和闭包
```go
package main

import "fmt"

func main() {
    // 匿名函数
    add := func(a, b int) int {
        return a + b
    }
    fmt.Println(add(3, 5))

    // 立即执行
    func(msg string) {
        fmt.Println(msg)
    }("Hello!")

    // 闭包
    counter := func() func() int {
        count := 0
        return func() int {
            count++
            return count
        }
    }()

    fmt.Println(counter())  // 1
    fmt.Println(counter())  // 2
    fmt.Println(counter())  // 3
}
```

---

## 4. 结构体基础
```go
package main

import "fmt"

// 定义结构体
type Person struct {
    Name string
    Age  int
    City string
}

func main() {
    // 创建结构体
    p1 := Person{"张三", 20, "北京"}
    p2 := Person{Name: "李四", Age: 21, City: "上海"}
    p3 := Person{Name: "王五"}  // 其他字段为零值

    // 指针
    p4 := &Person{Name: "赵六", Age: 22}

    // 访问字段
    fmt.Println(p1.Name, p1.Age)
    fmt.Println(p2)
    fmt.Println(p3)
    fmt.Println(p4.Name)  // Go 自动解引用

    // 修改字段
    p1.Age = 21
    fmt.Println(p1)
}
```

---

## 5. 结构体方法
```go
package main

import "fmt"

type Rectangle struct {
    Width  float64
    Height float64
}

// 值接收者方法
func (r Rectangle) Area() float64 {
    return r.Width * r.Height
}

// 指针接收者方法（可修改结构体）
func (r *Rectangle) Scale(factor float64) {
    r.Width *= factor
    r.Height *= factor
}

// 构造函数（Go 没有构造函数，用普通函数代替）
func NewRectangle(w, h float64) *Rectangle {
    return &Rectangle{Width: w, Height: h}
}

func main() {
    rect := NewRectangle(10, 5)
    fmt.Printf("面积: %.2f\n", rect.Area())

    rect.Scale(2)
    fmt.Printf("缩放后: %+v\n", rect)
    fmt.Printf("新面积: %.2f\n", rect.Area())
}
```

---

## 6. 结构体嵌套
```go
package main

import "fmt"

type Address struct {
    City    string
    Street  string
    ZipCode string
}

type Employee struct {
    Name    string
    Age     int
    Address Address  // 嵌套结构体
}

// 匿名嵌套（组合）
type Manager struct {
    Employee        // 匿名嵌套
    Department string
}

func main() {
    emp := Employee{
        Name: "张三",
        Age:  30,
        Address: Address{
            City:    "北京",
            Street:  "长安街",
            ZipCode: "100000",
        },
    }
    fmt.Printf("员工: %s, 城市: %s\n", emp.Name, emp.Address.City)

    mgr := Manager{
        Employee: Employee{
            Name: "李四",
            Age:  40,
            Address: Address{City: "上海"},
        },
        Department: "技术部",
    }
    // 匿名嵌套可以直接访问
    fmt.Printf("经理: %s, 部门: %s, 城市: %s\n",
               mgr.Name, mgr.Department, mgr.City)
}
```

---

## 7. 接口基础
```go
package main

import (
    "fmt"
    "math"
)

// 定义接口
type Shape interface {
    Area() float64
}

// 实现接口（隐式）
type Circle struct {
    Radius float64
}

func (c Circle) Area() float64 {
    return math.Pi * c.Radius * c.Radius
}

type Rectangle struct {
    Width, Height float64
}

func (r Rectangle) Area() float64 {
    return r.Width * r.Height
}

// 使用接口
func printArea(s Shape) {
    fmt.Printf("面积: %.2f\n", s.Area())
}

func main() {
    c := Circle{Radius: 5}
    r := Rectangle{Width: 10, Height: 5}

    printArea(c)
    printArea(r)

    // 接口切片
    shapes := []Shape{c, r}
    for _, s := range shapes {
        printArea(s)
    }
}
```

---

## 8. 错误处理
```go
package main

import (
    "errors"
    "fmt"
)

// 返回错误
func divide(a, b float64) (float64, error) {
    if b == 0 {
        return 0, errors.New("除数不能为零")
    }
    return a / b, nil
}

// 自定义错误
type ValidationError struct {
    Field   string
    Message string
}

func (e ValidationError) Error() string {
    return fmt.Sprintf("%s: %s", e.Field, e.Message)
}

func validateAge(age int) error {
    if age < 0 {
        return ValidationError{Field: "age", Message: "不能为负数"}
    }
    if age > 150 {
        return ValidationError{Field: "age", Message: "不能超过150"}
    }
    return nil
}

func main() {
    // 处理错误
    result, err := divide(10, 0)
    if err != nil {
        fmt.Println("错误:", err)
    } else {
        fmt.Println("结果:", result)
    }

    // 自定义错误
    if err := validateAge(-5); err != nil {
        fmt.Println("验证失败:", err)
    }
}
```

---

## 今日小结
| 知识点 | 要点 |
| --- | --- |
| 函数 | 多返回值、命名返回值 |
| 可变参数 | ...T |
| 闭包 | 匿名函数、状态保持 |
| 结构体 | type struct |
| 方法 | 值接收者、指针接收者 |
| 接口 | 隐式实现 |
| 错误处理 | error 接口 |

---

## 课后练习
完成练习（见 [exercises.md](exercises.md)）

---
