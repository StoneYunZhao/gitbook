# Module

## Basic

init 会在当前目录创建一个 go.mod 文件，当前目录的导入路径为 example.com/hello，子目录的的导入路径在后面加上子目录就行。

还会生成一个 go.sum文件，用于检测模块中的文件是否一致。

```go
go mod init example.com/hello
```

在代码中引用了第三方依赖后，执行 go test、go  build 等命令后，首先会在 go.mod 中查询相关依赖，若没有，则会自动查询包含那个 package 的模块，并使用最新的版本，添加到 go.mod 文件。版本的查找优先级为：打标签的最新的 release 版本 &gt; 打标签的最新的预发布版本 &gt; 最新的未打标签的版本。

```go
import "rsc.io/quote"
```

可以列出当前模块和它依赖的模块。

```go
go list -m all
```



{% embed url="https://blog.golang.org/using-go-modules" %}

[https://stackoverflow.com/questions/57355929/what-does-incompatible-in-go-mod-mean-will-it-cause-harm](https://stackoverflow.com/questions/57355929/what-does-incompatible-in-go-mod-mean-will-it-cause-harm)

