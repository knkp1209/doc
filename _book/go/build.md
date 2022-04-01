`go build` 是带有缓存的

例：`/usr/local/includes/stdlib.h` 删除之后，还会报错，这里因为 build 命令缓存 编译过程的中间结果

![image-20220318220547492](https://knkp-doc.oss-cn-guangzhou.aliyuncs.com/doc/image-20220318220547492.png)

缓存目录 可以通过 `go env | grep CACHE` 获取

![image-20220318220902625](https://knkp-doc.oss-cn-guangzhou.aliyuncs.com/doc/image-20220318220902625.png)

删除缓存命令 `go clean -cache`

`go test` 也是带有缓存的，删除缓存命令为 `go clean -testcache`