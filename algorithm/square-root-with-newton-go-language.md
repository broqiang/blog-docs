+++
title = "用 Go 语言实现用牛顿法平方根函数"
author = "BroQiang"
github_url = "https://broqiang.com"
head_img = ""
created_at = 2018-04-04T01:35:06
updated_at = 2018-04-04T01:35:06
description = ""
tags = ["go", "算法"]
+++

```go
package main

import "fmt"
import "math"

func Sqrt(x float64) float64 {
    z := 1.0

    for {
        tmp := z - (z*z-x)/(2*z)
        fmt.Println(tmp)

        if tmp == z || math.Abs(tmp-z) < 0.000000000001 {
            break
        }

        z = tmp
    }

    return z
}

func main() {
    fmt.Println(Sqrt(2.0))
    fmt.Println(math.Sqrt(2.0))
}
```