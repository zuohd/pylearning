# Day 26：Go 环境 + 基础语法
> 学习目标：搭建 Go 开发环境，掌握基础语法
>

---

## 1. Go 语言简介
```
Go 语言特点：
- Google 开发，2009 年发布
- 编译型语言，性能接近 C
- 语法简洁，上手快
- 原生支持并发（goroutine）
- 丰富的标准库
- 跨平台编译

适用场景：
- 后端服务
- 微服务
- 命令行工具
- 云原生应用
```

---

## 2. 环境搭建
```bash
# 1. 下载安装
# 访问 https://go.dev/dl/ 下载对应系统版本

# 2. 验证安装
go version
# go version go1.21.0 linux/amd64

# 3. 设置环境变量（Linux/Mac）
export GOPATH=$HOME/go
export PATH=$PATH:$GOPATH/bin

# 4. 创建项目
mkdir hello && cd hello
go mod init hello

# 5. 创建文件 main.go
# 6. 运行
go run main.go
```

---

## 3. Hello World
```go
// main.go
package main

import "fmt"

func main() {
    fmt.Println("Hello, World!")
}
```

```bash
# 运行
go run main.go

# 编译
go build -o hello
./hello
```

---

## 4. 变量和常量
```go
package main

import "fmt"

func main() {
    // 变量声明方式
    var a int = 10
    var b = 20        // 类型推断
    c := 30           // 简短声明（最常用）

    // 多变量声明
    var x, y int = 1, 2
    m, n := 3, 4

    // 常量
    const PI = 3.14159
    const (
        StatusOK = 200
        StatusNotFound = 404
    )

    fmt.Println(a, b, c)
    fmt.Println(x, y, m, n)
    fmt.Println(PI, StatusOK)
}
```

---

## 5. 基本数据类型
```go
package main

import "fmt"

func main() {
    // 整数
    var i int = 42
    var i8 int8 = 127
    var i64 int64 = 9223372036854775807

    // 无符号整数
    var u uint = 42
    var u8 uint8 = 255

    // 浮点数
    var f32 float32 = 3.14
    var f64 float64 = 3.141592653589793

    // 布尔
    var b bool = true

    // 字符串
    var s string = "Hello"

    // 类型转换（必须显式）
    var x int = 10
    var y float64 = float64(x)

    fmt.Printf("int: %d\n", i)
    fmt.Printf("float: %f\n", f64)
    fmt.Printf("bool: %t\n", b)
    fmt.Printf("string: %s\n", s)
    fmt.Printf("转换后: %f\n", y)
}
```

---

## 6. 条件语句
```go
package main

import "fmt"

func main() {
    age := 20

    // if 语句
    if age >= 18 {
        fmt.Println("成年人")
    } else {
        fmt.Println("未成年")
    }

    // if-else if-else
    score := 85
    if score >= 90 {
        fmt.Println("优秀")
    } else if score >= 80 {
        fmt.Println("良好")
    } else if score >= 60 {
        fmt.Println("及格")
    } else {
        fmt.Println("不及格")
    }

    // if 带初始化语句
    if num := 10; num > 5 {
        fmt.Println("num 大于 5")
    }

    // switch
    day := 3
    switch day {
    case 1:
        fmt.Println("周一")
    case 2:
        fmt.Println("周二")
    case 3:
        fmt.Println("周三")
    default:
        fmt.Println("其他")
    }
}
```

---

## 7. 循环
```go
package main

import "fmt"

func main() {
    // 基本 for 循环
    for i := 0; i < 5; i++ {
        fmt.Print(i, " ")
    }
    fmt.Println()

    // while 风格
    j := 0
    for j < 5 {
        fmt.Print(j, " ")
        j++
    }
    fmt.Println()

    // 无限循环
    // for {
    //     fmt.Println("无限")
    // }

    // range 遍历
    nums := []int{1, 2, 3, 4, 5}
    for index, value := range nums {
        fmt.Printf("索引: %d, 值: %d\n", index, value)
    }

    // 只要值
    for _, v := range nums {
        fmt.Print(v, " ")
    }
    fmt.Println()

    // break 和 continue
    for i := 0; i < 10; i++ {
        if i == 3 {
            continue  // 跳过 3
        }
        if i == 7 {
            break     // 到 7 停止
        }
        fmt.Print(i, " ")
    }
}
```

---

## 8. 切片（Slice）
```go
package main

import "fmt"

func main() {
    // 创建切片
    var s1 []int                    // 声明
    s2 := []int{1, 2, 3}            // 字面量
    s3 := make([]int, 5)            // make 创建，长度 5
    s4 := make([]int, 3, 10)        // 长度 3，容量 10

    fmt.Println(s1, s2, s3, s4)

    // 追加元素
    s1 = append(s1, 1, 2, 3)
    fmt.Println(s1)

    // 切片操作
    nums := []int{0, 1, 2, 3, 4, 5}
    fmt.Println(nums[1:4])   // [1 2 3]
    fmt.Println(nums[:3])    // [0 1 2]
    fmt.Println(nums[3:])    // [3 4 5]

    // 长度和容量
    fmt.Printf("长度: %d, 容量: %d\n", len(nums), cap(nums))

    // 遍历
    for i, v := range nums {
        fmt.Printf("索引 %d: %d\n", i, v)
    }
}
```

---

## 9. Map
```go
package main

import "fmt"

func main() {
    // 创建 map
    m1 := make(map[string]int)
    m2 := map[string]int{
        "apple":  5,
        "banana": 3,
    }

    // 添加/修改元素
    m1["one"] = 1
    m1["two"] = 2

    // 获取元素
    fmt.Println(m2["apple"])

    // 检查键是否存在
    value, exists := m2["orange"]
    if exists {
        fmt.Println("orange:", value)
    } else {
        fmt.Println("orange 不存在")
    }

    // 删除元素
    delete(m2, "apple")

    // 遍历
    for key, value := range m2 {
        fmt.Printf("%s: %d\n", key, value)
    }

    // 长度
    fmt.Println("长度:", len(m2))
}
```

---

## 今日小结
| 知识点 | 要点 |
| --- | --- |
| 环境 | go mod init、go run、go build |
| 变量 | var、:=、const |
| 类型 | int、float64、bool、string |
| 条件 | if、switch |
| 循环 | for、range |
| 切片 | []T、append、切片操作 |
| Map | map[K]V、make |

---

## 课后练习
完成练习题 26-28（见 [exercises.md](exercises.md)）

---
