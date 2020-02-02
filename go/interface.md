# 接口约定

__借口类型__:借口类型是一种抽象类型,他不会暴露出他所代表的队形的内部值的结构和这个对象支持的基础操作集合,它们只会展示出它们自己的方法.

`Fprintf`的前缀F表示文件(File)也表明格式化输出结果应该被写入第一个参数提供的文件中.在`Printf`和`Sprintf`中都调用了`Fprintf`

``` go
package fmt
func Fprintf(w io.Writer, format string, args ...interface{})(int, error)
func Printf(format string, args ...interfaceP{})(int, error){
    return Fprintf(os.Stdout, format,args...)
}
func Sprintf(format string, args ...interface{}) string{
    var buf bytes.Buffer
    Fprintf(&buf,format,args...)
    return buf.String()
}
```



`Printf`中的第一个参数os.Stdout是*os.File类型,把结果写入标准输出中.

`Sprintf`中的第一个参数&buf是一个可以指向可以写入字节的内存缓冲区,最后返回一个字符串

`Fprintf`函数的第一个参数是一个借口类型__io.Writer__类型

``` go
package io

type Writer interface{
    .......
    Write(p []byte) (n int,err error)
}
```

__io.Writer__类型定义了函数`Fprintf`和这个函数的调用者之间的约定:

+ 调用者提供具体类型的值就像\*os.FIle和*bytes.Buffer,这些类型都有一个特定签名和行为的Write函数
+ 保证`Fprintf`接受任何满足io.Writer借口的值都可以工作.

`Fprintf`函数可能没有指定写入的是一个文件或者是一段内存,而是写入一个可以调用Writer函数的值.

**一个类型可以自由的使用另一个满足同样接口的类型来进行替换被称作可替换性(LSP里氏替换)**这是一个面向对象的特征

``` go
type ByteCounter int
 
func (c *ByteCounter ) Write(p []byte ) ( int,error){
    *c += ByteCounter(len(p))
    return len(p),nil
}
```

*ByteCounter满足io.Writer的约定,拥有对应的Write方法

把他传入`Fprintf`函数中,执行字符串格式化的过程中不会去关注ByteCounter正确的累加结果的长度.

``` go
var c ByteCounter
c.Write([]byte("hello"))
fmt.Println(c)
c = 0
var name = "Dolly"
fmt.Fprintf(&c,"hello, %s",name)
fmt.Println(c)
```



