昨天花了一天的时间写一个项目[zanshin](https://github.com/tiancaiamao/zanshin)。不知道这是自己第多少次写类似的东西了。这次做的事情是，用scheme写字节码编译器，然后用Go写虚拟机。大部分只是拿以前的代码过来修修改改，没什么新意。

之所以会又拿出来写一次，是想在项目里嵌入一个简单的虚拟机用于调试。Go语言不是一个REPL的环境，**运行起来之后的调试有点蛋疼**。如果我能给它加一个虚拟机，就可以在服务运行的时候连接上去的查询和修改一些数据。大概就是这样一个想法。

我们竞价服务的项目，会加载一些数据到内存，竞价的时候会用这些数据过滤决定是否出价。比如调试为什么没有出价？某个数据到底加载了没有？代码到底会不会运行某一过滤条件。如果可以连一个虚拟机了去里面查，调试起来会方便一些。

当然不是将之前整个逻辑迁移到虚拟机之上，只是调试的时候连上去，绑定几个调试函数和运行环境，所以性能不用考虑。

为这个去引入另一门语言我觉得太重了。只用Go写一个虚拟机就行，不会超过几百行，其它东西可以扔到外面去。

我把REPL设计成这样：scheme那端通过网络连接上Go那端，在scheme中编译好字节码，直接将字节码发给Go，在Go写的虚拟机中运行。

这样子会有一个多文件编译的问题。每次输入一行，都会生成一次的字节码，这些字节码不会被“链接”到一个文件中。所以在函数调用的时候，仅仅保存一个PC寄存器(记录在字节码内部的偏移量)是不够的，我需要保存(哪一份字节码 + 在字节码内的偏移)两个信息才能记录函数返回地址。

另一个问题是全局变量。我们知道一个可执行文件，会有代码段和数据段，我们的字节码也同样有些信息，全局变量就对应于数据段。但是由于我们是REPL的编译的，数据段应该累积起来。具体做法是，从scheme连接上Go，到断开连接，这个期间算一次调试的session。在这个session内，全局变量的可见性应该是持久的，也就是说后一次编译环境是在前一次编译环境基础之上的。

编译到字节码，执行字节码的虚拟机，都是相对独立的。用scheme写编译，可以省掉parser。至于用Go实现虚拟机，可以省掉垃圾回收。

目前还很粗糙，只把基本的架子搭起来了，等完善好了才能用到我们项目上面去。

zanshin是日语的剑道中的一个词汇--残心，不要问为什么，随便想到的（巫剑神威控中这个招式很帅^_^）。
