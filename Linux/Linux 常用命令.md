# Linux 常用命令

## 目录
* [find](#find)

## <h2 id="find">find</h2>

* `find dir -name name` (根据文件名查询)
* `find -size [+-]N[bcwkMG]` (根据文件大小查询)
```
# b,c(byte),w,k,M,G 是表示文件大小的单位
# + 表示 size>N, - 表示 size <= N, 无符号表示 N-1 < size <= N  
```
*  (查询到文件并对这些文件执行 `ll -h` 命令，显示文件信息)

`find dir -name name -exec -ls -l {} \;`

`find dir -name name | xargs ls -lh`


