# Test

源文件 \_test.go 为后缀时表示测试代码。

* 测试函数：Test 开头。
* 基准测试函数：Benchmark 开头。
* 示例函数：Example 开头。

go test 命令会遍历所有 \*\_test.go 文件中符合上述命名规则的函数，生成临时 main 函数，构建、运行、报告测试结果，最后清理临时文件。

## 测试函数

函数名 Test 后面必须以大写字母开头。

```go
func Test${Name}(t *testing.T) {
    // 继续执行后面的代码
    t.Error()
    t.Errorf()
    
    // 停止，必须和测试函数在同一个 goroutine 中调用
    t.Fatal()
    t.Fatalf()
}
```



```text
go test [build/test flags] [packages] [build/test flags & test binary flags]

// 当前目录对应的包
go test

-v // 打印详情
-run="${regx}" // 仅测试函数名匹配的测试函数
-coverprofile=c.out // 输出测试覆盖率文件
-covermode=count // 覆盖率插入的代码是计数器而不是 bool，可以衡量热点代码


go tool cover -html=c.out // 生成覆盖率报告
```

表格形式的测试。

随机测试。

黑盒、白盒测试。

若在测试代码中修改了全局变量，记得改回去。

外部测试包可以避免循环依赖，对于集成测试应用比较广泛。若外部测试包需要做白盒测试，可以通过内部测试包导出相关变量，如 fmt 包中的 export\_test.go 文件。

## 基准测试

go test 默认不执行基准测试，-bench=${regex} 指定执行的基准测试的函数名。

循环在基准测试内实现，而不是测试框架，这样就可以做一些初始化操作。

```bash
go test
-bench=. 
-benchmem // 统计内存相关数据
```

