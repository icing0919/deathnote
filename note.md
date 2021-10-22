1.包的使用：
一个目录下的统计文件归属一个包。package的声明要一致
package声明的包和对应的目录可以不一致，但一般上是写成一致的
包可以嵌套
同包下的函数不需要导入包，可以直接使用
main包，main（）函数所在的包，其他的包不能使用
导入包的时候，路径要从src下开始写

----------------------------------------------------
2.init（）函数和main（）函数：
这两个函数都是go语言中的保留函数。init用来初始化信息。main用于作为程序的入口
这两个函数定义的时候：不能有参数、返回值。只能由go程序自动调用，不能被引用
init函数可以定义在任意的包中，可以有多个。main函数只能在main包下，并且只能有一个
执行顺序
    A.先执行init函数，后执行main函数
    B.对于同一个go文件中，调用顺序是从上到下的，也就是说，先写的先执行，后写的后执行
    C.对于同一个包下，将文件名按照字符串进行排序，之后顺序调用各个文件中的init函数
    D.对于不同包下
        如果不存在依赖，按照main包中import的顺序来调用对应包中init函数
        如果存在依赖，最后被依赖的最先被初始化
            导入顺序：main——>A——>B——>C
            执行顺序：C——>B——>A——>main
存在依赖的包之间，不能循环导入
一个包可以被其他多个包import，但是只能被初始化一次。
_操作，其实是引入该包，而不直接使用包里面的函数，仅仅是调用了该包里的init（）

----------------------------------------------------
3.目录/文件操作：
路径：
    相对路径：relative
        ab.txt
        相对于当前工程
    绝对路径：absolute
        /e/go/src/aa.txt
创建文件夹，如果文件夹存在，创建失败
    os.MkDir()，创建一层
    os.MkDirAll()，可以创建多层
创建文件，Create采用模式0666（任何人都可读写，不可执行）创建一个名为name的文件，如果文件已存在会截断它（为空文件）    os.Create（），创建文件

----------------------------------------------------
4.打开文件：让当前的程序，和指定的文件之间建立一个连接
    os.Open(filename)
    os.OpenFile(filename,mede,perm)


file3 ,err := os.Open(fileName1) 只读的
if err != nil{
    fmt.Println("err:",err)
    reurn
}
fmt.Println(file3)

第一个参数：文件名称
第二个参数：文件的打开方式 RDONLY只读 WRONLY只写 RDWR读写 ······
第三个参数：文件的权限:文件不存在创建文件，需要指定权限

----------------------------------------------------
5.关闭文件：程序和文件之间的链接断开
    file.Close()
----------------------------------------------------
6.删除文件或文件夹：
err := os.Remove("路径")
if err != nil{
    fmt.Println("err:",err)
    return
}
fmt.Println("删除文件成功")

os.Remove(),删除文件或空目录
os.RemoveAll(),删除所有（慎用，不经过回收站，找不回来了）
----------------------------------------------------
7.读取数据：
    Reader接口：
        Reader(p []byte)(n int, error)

    读取本地aa.txt文件中的数据
    step1：打开文件
        fileName := "路径/aa.txt"
        file,err := os.open(fileName)
        if err != nil{
            fmt.Println("err:",err)
            return
        }

    step3：关闭文件
        defer file.Close()

    step2：读取数据
        bs := make([]byte,4,4)
        n,err := file.Read(bs)
        fmt.Println(err)
        fmt.Println(n)
        fmt.Println(bs)
现在是根据指定的长度（4）输出一次，循环多次执行的话，可以顺序导出后续所有数据。
也可以通过别的参数来从某一位置等导出数据。
----------------------------------------------------
8.写出数据
step1：打开文件
    fileName := "路径/ab.txt"
    file,err := os.Open(fileName)   
    if err != nil{
        fmt.Println("err:",err)
        return
    }
    //file,err := os.OpenFile(fileName,os.O_CREATE|os.O_WRONLY,os.ModePerm)
    //如果没有这个文件，只用Open是会报错的，用OpenFile可以创建并打开

step3：关闭文件
    defer file.Close()

step2：写出数据
    bs := []byte{65,66,67,68,69,70}   ABCDEF
    n,err := file.Write(bs)
    fmt.Println(n)
    HandleErr(err)

    func HandleErr(err error){
        if err != nil{
            log.Fatal(err)
        }
    }
现在的输出是覆盖模式的，如果是要追加数据，要再加一个模式 |os.O_APPEND 。
n,err := file.Write(bs[:2])  //用切片写出部分想要的数据
n,err = file.WriteString("helloworld")   //直接写出字符串
n,err := file.Write([]byte("hello"))    //拿byte写字符串，转换成字节切片
ReaderAt从哪开始读  WriterAt从哪开始写
----------------------------------------------------
9.复制文件
----------------------------------------------------
10.断点续传
    文件的传递：文件复制
Seek(offset int64, whence int) (int64, error)   设置指针光标的位置
    第一个参数：偏移量
    第二个参数：如何设置
        0：seekstart，表示相当于文件开始
        1：seekcurrent，表示相对于当前位置的偏移量
        2：seekend，表示相对于末尾
通过seek方法设置光标，可以读取文件任意位置的数据，也可以在任意位置写出数据

----------------------------------------------------
11.runtime
runtime.GOROOT()   获取goroot目录
runtime.GOOS     获取操作系统
runtime.NumCPU()    获取逻辑cpu的数量
runtime.GOMAXPROCS(runtime.NumCPU())   设置go程序执行的最大的cpu的数量：[1,256]
runtime.Gosched()    让出时间片，先让别的goroutine执行
runtime.Goexit()   终止当前的goroutine
----------------------------------------------------
同步等待：  WaitGroup：同步等待组
    var wg sync.WaitGroup   创建同步等待组的对象
    Add（），设置等待组中要执行的子goroutine的数量
    Wait（），让主goroutine出于等待
    wg.Done()  给wg等待组中的counter数值减1.   同wg.Add(-1).

----------------------------------------------------
互斥锁：
多个goroutine访问一个资源，这个资源叫临界资源，处理不当会产生恐慌，可以通过上锁来另资源被一个goroutine访问时，别的goroutine无法访问
mutex.Lock()  上锁
mutex.Unlock()   解锁
也可以叫他“同步”，通过上锁解锁保证同一时间只有一个goroutine访问资源。
在使用互斥锁的时候，对资源操作完，一定要解锁。否则会出现程序异常，死锁等问题。
建议defer语句解锁。
----------------------------------------------------
读写锁
针对于读写操作的互斥锁，和普通互斥锁的区别是，他能分别针对读操作和写操作上锁解锁。
两大原则：
    可以随便读，多个goroutine同时读，读时不能写。
    写的时候，啥也不能干，不能读也不能写。
----------------------------------------------------
通道channel
----------------------------------------------------
缓冲通道：
    非缓冲通道：make(chan type)
        都次发送，一次接收，都是阻塞的
    缓冲通道：make(chan type , capacity)
        发送：缓冲区的数据满了，才会阻塞
        接收：缓冲区的数据空了，才会阻塞

----------------------------------------------------
6
----------------------------------------------------
----------------------------------------------------
----------------------------------------------------
----------------------------------------------------
----------------------------------------------------
----------------------------------------------------
