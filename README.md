[合集 \- .NET9 \& C\#13(1\)](https://github.com)1\..NET9 \- 新功能体验（一）11\-21收起
被微软形容为“迄今为止最高效、最现代、最安全、最智能、性能最高的.NET版本”——.NET 9已经发布有一周了，今天想和大家一起体验一下新功能。


![](https://img2024.cnblogs.com/blog/386841/202411/386841-20241121001414091-1682627959.png)


此次.NET 9在性能、安全性和功能等方面进行了大量改进，包含了数千项的修改，今天主要和大家一起体验我们常用有变化的功能。


# ***01***、安装


首先可以在命令行中通过dotnet \-\-list\-sdks指令查看是否以安装.NET9。


![](https://img2024.cnblogs.com/blog/386841/202411/386841-20241121001421735-999325891.png)


安装方式主要有两种，第一种通过直接下载.NET9 SDK安装。


![](https://img2024.cnblogs.com/blog/386841/202411/386841-20241121001428486-1572180636.png)


第二种方式通过更新IDE Visual Studio至17\.12\.0或之后的版本，比如我的IDE是17\.11\.6，选择更新即可，如果是更老的版本，比如17\.11\.4则需要更新两次才行。


![](https://img2024.cnblogs.com/blog/386841/202411/386841-20241121001436123-1559035637.png)


再次执行dotnet \-\-list\-sdks命令，发现已安装成功，结果如下：


![](https://img2024.cnblogs.com/blog/386841/202411/386841-20241121001442806-969399181.png)


# ***02***、新的转义序列


本次C\#13新引入了 \\e 作为 ESC (Escape)字符 Unicode U\+001B 的字符文本转义序列。而以前使用的是 \\u001b 或 \\x1b。


ESC (Escape) 字符（ASCII 编码值为 27，十六进制为 0x1b）是一种控制字符，可以用于 终端控制和文本格式化。它本身代是不可见字符，主要作用标志着后续字符序列的开始，这些序列会指定终端执行特定的操作，如改变文本颜色、设置文本样式、清屏、移动光标等。


我们举一个例子，比如“ESC\[31m ”表示把文本颜色设置为红色，我们把ESC改为转义序列，代码如下：



```
static void Main(string[] args)
{
    var colorRed = "\x1b[31m红色文本";
    Console.WriteLine(colorRed);
    Console.ReadKey();
}

```

执行效果如下：


![](https://img2024.cnblogs.com/blog/386841/202411/386841-20241121001453900-1625026884.png)


如果单独使用ESC (Escape)字符的转义序列则表示其后接着的字符不可见，我们可以用\\u001b和\\x1b看看效果，结果如下，发现看字没有展示出来：


![](https://img2024.cnblogs.com/blog/386841/202411/386841-20241121001500641-328994903.png)


然后我们把\\u001b和\\x1b改为\\u001b1和\\x1b1，再看一下效果：


![](https://img2024.cnblogs.com/blog/386841/202411/386841-20241121001507679-987336479.png)


从上图可以发现使用\\u001b1结果还是\\u001b后面一个字符看不见，而使用\\x1b1后变成问号“？”了，这是因为\\x1b1整体被识别为一个更长的十六进制序列，而不是单纯的控制字符了，这就产生了歧义，因此不建议使用\\x1b，因而这次新增了转义字符\\e，效果如下。


![](https://img2024.cnblogs.com/blog/386841/202411/386841-20241121001514859-160991548.png)


# ***03***、隐式索引访问


现在对象初始值设定项表达式中允许隐式“从末尾开始”索引运算符\[^]。


我们来看一个例子，首先创建了一个ImplicitIndex类并且包含一个Numbers属性，该属性是一个长度为 5 的整数数组。


现在我们可以在初始化类ImplicitIndex时初始化属性Numbers，并使用“从末尾开始”索引运算符来填充数组值。


我们先看看在.NET8中的效果，鼠标移到错误上可以看到老版本是不支持该功能的，如下图：


![](https://img2024.cnblogs.com/blog/386841/202411/386841-20241121001522479-1086833474.png)


而使用.NET9则可以。


# ***04***、params参数增强


之前params只支持数组，而现在params参数支持更多的集合类型，包括 Span、ReadOnlySpan、IEnumerable 等等。这使得我们可以更加灵活的传入集合参数，而不用先把集合转成数组后再传输。



```
public static void PrintNumbers(params int[] numbers) { }
public static void PrintNumbers(params List<string> numbers) { }
public static void PrintNumbers(params HashSet<int> numbers) { }
public static void PrintNumbers(params SortedSet<int> numbers) { }
public static void PrintNumbers(params IList<int> numbers) { }
public static void PrintNumbers(params ICollection<int> numbers) { }
public static void PrintNumbers(params IEnumerable<int> numbers) { }
public static void PrintNumbers(params Span<int> numbers) { }
public static void PrintNumbers(params ReadOnlySpan<int> numbers) { }

```

params 关键字允许方法接收一个可变数量的参数，通常是单一类型的参数集合。params 的参数通常可以是数组、集合或其他实现了 IEnumerable 接口的类型，但有一些限制，比如以下类型虽然实现了IEnumerable 接口，但是并不受支持。



```
public static void PrintNumbers(params Dictionary<int, int> numbers) { }
public static void PrintNumbers(params SortedList<int, int> numbers) { }
public static void PrintNumbers(params LinkedList<int> numbers) { }
public static void PrintNumbers(params Queue<int> numbers) { }
public static void PrintNumbers(params Queue numbers) { }
public static void PrintNumbers(params Stack<int> numbers) { }
public static void PrintNumbers(params Stack numbers) { }
public static void PrintNumbers(params Hashtable numbers) { }

```

# ***05***、锁对象


本次更新引入新的锁类型System.Threading.Lock，用于实现互斥。在之前的版本中通常通过object类型进行加锁，而现在有了专门的Lock类型用来加锁。


新的Lock类型会使得代码更干净、更安全、更高效。


在新的锁定机制中EnterScope替换了Monitor底层实现。同时它遵循Dispose模式返回ref struct，因此可以与using语句结合使用。


我们一起看看下面代码示例：



```
// .NET 9 之前
public class LockExampleNET9Before
{
    private readonly object _lock = new();
    public void Print()
    {
        lock (_lock)
        {
            Console.WriteLine("我们是老的锁");
        }
    }
}
// .NET 9
public class LockExampleNET9
{
    private readonly Lock _lock = new();
    public void Print()
    {
        lock (_lock)
        {
            Console.WriteLine("我们是 .NET 9 新锁");
        }
    }
    public async Task LogMessageAsync(string message)
    {
        using (_lock.EnterScope())
        {
            Console.WriteLine("我们是 .NET 9 新锁，可以和using一起使用");
        }
    }
}

```

# ***06***、生成UUID v7


我们经常在实体中使用Guid作为主键，并且通过Guid.NewGuid()可以很方便的生成一个新的Guid，而此方法生成的Guid是依据UUID第四个版本规范生成的。


当前已经可以通过Guid.CreateVersion7()方法创建UUID第七个版本，这个版本UUID主要功能就是包含了时间戳，数据结构如下：


\| 48位时间戳 \| 12位随机 \| 62位随机 \|


这也意味着v7版本的UUID可以按时间排序了，在数据库中使用起来更方便，同时Guid.CreateVersion7()方法还有一个重载方法接收DateTimeOffset类型时间戳，用来通过指定时间创建UUID。



```
static void Main()
{
    // v4 UUID
    var guid_v4 = Guid.NewGuid();
    // v7 UUID
    var guid_v7 = Guid.CreateVersion7();
    // v7 UUID with timestamp
    var guid_v7_time = Guid.CreateVersion7(TimeProvider.System.GetLocalNow()); 
    Console.ReadKey();
}

```

***注***：测试方法代码以及示例源码都已经上传至代码库，有兴趣的可以看看。[https://gitee.com/hugogoos/Planner](https://github.com):[westworld加速](https://tianchuang88.com)


