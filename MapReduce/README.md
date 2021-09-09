# MapReduce

MapReduce产生最初由google设计，制造和使用的系统，mapreduce的论文可以追溯到2004年，他们面临的问题是，他们正在以TB为单位运行大量计算,对于数TB的数据，例如创建网络所有的索引或分析链接整个网络的结构以便识别最重要的页面，或者你所知道的最权威的页面，对整个网络这段时间里产生的数十亿兆字节的数据构建索引基本相当与对整个数据进行排序。
google寻求一种使得非专业人员也能够轻松开发可运行大型分布式计算系统

Map函数将一个Key和Value作为参数

``` go
func Map(filename string, value interface{})
```


Reduce函数会根据这个key所关联的所有map对应的值得到一个vector数组

``` go
func Reduce(key interface{}, value []int) int {
    return len(value)
}
```


## 运行流程
首先，这些运行在哪里，假设这里有1000台服务器，其中大量的服务器用作worker，还有一个master来组织整个计算,在master上有5000个输入文件，这5000个输入文件会在空闲时期由主服务器分发给不同的worker，master会想worker7发送一条消息：在这样一个输入文件上运行map函数，然后就是运行属于MapReduce的worker上的函数。
对MapReduce有了足够的理解，接下来就是读取输入的文件，并且使用文件名作为map函数的参数来调用该函数，然后worker进程执行我们在map函数中的实现逻辑，每次map调用emit时，worker进程都会将此数据写入到本地磁盘上的文件。(当map调用emit时，会在调用此map的worker本地磁盘上生成文件，这些文件会不断追加该worker上运行map所生成的键和值)，在map的最后阶段，我们会有所有这些worker机器上map函数所生成的输出，然后这些MapReduce workers会安排将数据移动到需要进行reduce操作的位置上。因此，在一个典型的大型计算中，reduce需要所有的map输出的结果，在这个简单的例子中就是，这个reduce需要所有关于key为a的map输出，通常在我们运行这个rreduce函数之前，一个MapReduce worker会与其他几千台服务器的每一个进行通信，reduce worker将会从每个worker中取出key为a的所有实例，一旦他结束全部的实例收集，他就可以调用reduce(),并且reduce本身会调用emit函数，并将输出结果写入google的文件集群服务器(GFS)中的一个文件

虽然由于网络失败或者负载或者其他一些原因导致master就近处理数据不可能百分之百做到，但**几乎所有的map人物和它对应存储数据的地方都在同一台机器上**

``` go
[]interface{}{a, b, c}
[]interface{}{b}
[]interface{}{a, c}

[]interface{}{{a, 1}, {a, 1}}
[]interface{}{{b, 1}, {b, 1}}
[]interface{}{{c, 1}, {c, 1}}
```

将行存储转换为列存储的过程，在论文中将之称为 shuffle
    需要将整个网络中的所有数据从它的map服务器转移到它的reduce服务器，这个过程也是MapReduce非常消耗性能的一部分 
