[[toc]]

# 要解决的问题

如何降低异步编程的复杂度

异步编程的复杂度本质上是因为引入多时间线导致的时序混乱问题，程序员作为人类所具有的单时间、线性思维方式在此遇到了智力瓶颈。并在此背景下，实际编程还涉及另一个问题：通信，即时间线之间如何产生影响或进行协同。

# 解决方案

通过高阶的编程模型，让程序员以一种更自然、简单的方式认知多时间线问题，从低阶的 callback/thread 中脱离出来。

# 解决方案案例

## Reactive Programming

ReactiveX 针对各平台提供了统一的响应式编程组件，它同时支持了声明式和函数式编程范式。

### 基于流的抽象

时间是程序面临的本质问题之一，当下最为普遍方法是通过维护内部状态，记录下信息随着时间而产生的变化：打车时乘客所处不同阶段需要有不同的行为支持、工单在不同阶段也会有不同的处理方法、用户账户余额随着使用不断减少。
主流的编程语言使用赋值元语为状态提供了很好的支持，但也随着状态的膨胀，不同状态被不同时间线所影响，复杂度也成指数级上升。
以赋值为基础的抽象类似一个自变量由输入(i)、时间(t)组成的函数，因为程序员大脑思维无法对具有时间的函数进行成像（特别是该自变量同时被多个函数使用），所以我们希望能通过变换，产生能够符合心智的模型，这样就能大大降低该类场景的编程难度。

流是时间的抽象，它是随着时间流逝而状态不断变化的表征函数，其自变量是状态的变化函数。从流的视角来看，随着时间变化的状态像是空间中排列完好的序列，并且它将时间变量从原有的函数中抽离形成新的函数，这样互相影响的时间线就是一个坐标系中以不同的变化函数作为常量、时间作为自变量的曲线。
点击事件是流、map/reduce 是流、并发请求时流、余额变化也是流。

### 构成

| 构成 | 解释 |
| --- | --- |
| 事件 | 导致多时间线的触发动作，可以是同步也可是异步 |
| Observable | 基于流的抽象概念，提供事件或数据的访问|
| Subscribe | 通过订阅 Observable，进行响应 |
| Operator | 类似函数式编程，对流进行变换 |

### UI 交互

赞同和反对对投票的影响为两条平行的时间线，可以将它们分别视为流，并随着时间的前进更新投票数

<<< @/asynchronous-programming/reactive-programming-sample.js

### 并行计算

<<< @/asynchronous-programming/ParallelComputation.java

### RPC 并发请求

<<< @/asynchronous-programming/ConcurrentRpc.java

### 数据流

从上个示例可以看出，流不仅仅是时间抽象的有效工具，结合函数式编程的特性，序列计算逻辑也能够得到很好地表达。

### 通信

两条时间线可能会进行通信，可以共享状态、可以发送消息，复杂度体现在某个时刻两条时间线的状态纠葛。如果以流的视角对这种情况建模，那么可以将所有「某个时刻」视为单独的时间线（流），从而与其他时间线区分。
从上面「UI交互」的示例中能够看出，这种理解是一种非常自然且简单的方式。

## CSP (Golang)

基于 Thread 的并发过程通过共享内存进行通信，随之而来的既是同步、锁、并发控制的问题。CSP 将过程的输入输出视为通信本身，并可将其应用于并发过程。
举一个爬虫的例子： 对关键字进行搜索，其结果有多页，而每一页也会有多个抓取链接，最终将链接正文输出。

### 线程抽象

如果使用线程作为工具对其抽象，分页抓取和链接抓取为并行的两个独立线程，两者共用某块内存空间，前者写后者读，其中需要对共享结构进行空间控制、同步控制（锁），进而会引起可能的死锁。

### goroutine & channel

分页抓取和链接抓取分别为两个过程，前者将抓取到的链接传递给后者，后者根据接收到链接进行解析。编程语言支持更高阶的抽象，提供对应的原语，从而让程序员以更自然、简单的方式思考并发问题。
对于关键字、正文也可使用同样的方式，将收集用户输入、持久化视为过程，输入结果与分页抓取通信、输出结果与输出过程通信，整体形成「关键字->搜索页->链接->正文」的模型，而不用过多的考虑其他细节。

<<< @/asynchronous-programming/goroutine.go



