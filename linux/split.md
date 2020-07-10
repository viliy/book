# 1. Split - 切割文件


## csv 文件

200w+的csv文件，卡的一逼
用如下命令切割成20多个文件

```shell
split --numeric-suffixes=1 --additional-suffix=.csv -l 100000 sku_origin.csv sku_
```
感觉被拯救了

## split

> 语法： split [OPTION]... [INPUT [PREFIX]]
> 说明： split [参数列表]  [待拆分文件  [拆分文件后生成的文件名前缀]]

### options 

* -a , --suffix-length=N

说明：拆分后生成顺序文件名的长度，默认为2 ，在没有使用-d或--numeric-suffixes[=FROM] 参数时，是为 aa , ab , ac , ad 顺序递增生成拆分文件。



--additional-suffix=SUFFIX

说明：指定拆分后生成的文件名后缀，默认没有后缀；如指定 --additional-suffix=.json 则生成的拆分文件为 aa.json , ab.json , ac.json。



* -b , --bytes=SIZE

说明：指定每个拆分文件的大小，可以使用单位（K , M , G , T , P , E , Z , Y ）或（KB ，MB，GB，TB，PB，EB，ZB，YB）单位进制为 1024字节。



* -C , -line-bytes=SIZE

说明：类似 -b 参数，但这里是在保证每行的完整性下拆分文件，可用单位与 -b 一样。



*  -d , --numeric-suffixes[=FROM]

说明：指定拆分后生成顺序文件名为数字方式，默认为 00 ， 01 ，02，03 。或者指定开始值。

如：

split -l 1000 -d split.json split-tmp-
或

split -l 1000 --numeric-suffixes=10 split.json split-tmp-
注意：如果没有指定拆分生成文件名的前缀默认会在数字上添加 x 如： x00 , x01 , x02



* -e , --elide-empty-files

说明：指定当文件是空的时候不生成拆分文件，只针对使用 -n 或 --number=CHUNKS 参数。



* --filter=COMMAND

说明：调用shell脚本文件过滤处理，脚本获取拆分生成文件名的变量是 $FILE ，获取文件内容使用变量是 $file 。这是一个高级命令，可以使用这两个变量处理最终的拆分文件结构。

注意：使用这个参数后，当前命令是不会生成拆分文件，如果要写拆分文件得通用shell命令来处理。



* -l , --lines=NUMBER

说明：指定拆分后每个文件的最大行数。



* -n , -number=CHUNKS

## FAQ
搜索了下，下面的这个是比较全和实用的
(linux下拆分文件split)[https://blog.51cto.com/php2012web/1660968]