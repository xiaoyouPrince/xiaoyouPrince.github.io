---
title: 并发编程指南- Dispatch Queue
auther: xiaoyouPrince
date: 2022-09-17 23:19:53 +0800
categories: 文档 并发编程指南
tags: 并发 GCD Dispatch Queue thread
layout: post
---

# 并发编程指南- Dispatch Queue

官方地址：[ConcurrencyProgrammingGuide](https://developer.apple.com/library/archive/documentation/General/Conceptual/ConcurrencyProgrammingGuide/Introduction/Introduction.html#//apple_ref/doc/uid/TP40008091)

Grand Central Dispatch (GCD) dispatch queues 是执行任务非常强大的工具，它可以同步/异步执行任意数量的任务。 dispatch queues 几乎可以执行你使用线程可实现的所有操作，其优势就是它更简洁且高效。

## 关于 Dispatch Queues

Dispatch Queues 是一种简单的可执行异步和并发任务的方式。例如：你可以执行计算操作，创建或者修改数据结果，从文件读数据等等，你只需要将对应的任务封装成 block 或者函数，并将它添加到 dispatch queue 即可。

Dispatch Queues 是一种类似对象的结构，用于管理提交给它的任务。所有队列都是 first-in，first-out 的数据结构，所以其中任务的执行顺序和它们被添加的顺序一样。 GCD 提供了几种队列，你也可以自己创建队列，下面是几种队列的分类

1. Serial queues： 串行队列（又称作私有派发队列）同一时间只能执行一个任务，且执行顺序同其被添加的顺序。任务被执行的线程就说队列管理的线程，是明确的。 虽然队列是串行的，但你可以创建多个队列去执行不同的任务，也能达到并发执行任务的效果。
2. Concurrent queues： 并发队列（又称作全局派发队列）可以同时执行1个或多个任务，但任务的启动顺序仍和其被添加的顺序一样。当前被执行的任务会被派发到一个独立的线程中，但确切的被执行的任务数量会依赖系统环境改变。
3. Main dispatch queue： 主派发队列是全局串行队列，它会将任务执行在主线程。主队列和程序的 runloop 一起工作，添加到队列的任务会和 runloop 的其他事件源交错执行。因为它在主线程执行，所以它通常被用作 App 同步的关键点。

使用 GCD dispatch queue 执行并发操作比使用线程的优势如下：

- 更简洁的 `任务-队列` 的编程模型。使用线程你需要处理任务代码和创建管理线程本身的代码。 dispatch queue 则让你更专注要执行的任务。
- dispatch queue 会负责管理线程，并且能根据系统当前的负载/硬件情况创建合适的线程数量
- dispatch queue 本身是系统队列，其任务的执行速度更快

使用 dispatch queue 重构你的多线程代码也十分简单，关键在于设计自包含且能够异步执行的任务。(这对于 dispatch queue 和 thread 都适用)。然而，dispatch queue 的优势在于可预测性。如果你有两个线程并发访问同样的资源，为了保证数据同步，你可能需要锁来避免线程竞争。但是使用 dispatch queue 你可以将两个任务添加到一个串行队列中来确保同时只有一个任务能访问共享资源。**这比使用锁更加高效，因为锁无论在有无争议的情况下都需要执行昂贵的内核陷阱(kernel trap). 而 dispatch queue 则主要在应用程序的进程空间中工作，并且仅在绝对必要的时候才调用内核。**

> 可以理解为：
> 锁：一个线程对资源加锁后，其调用了内核陷阱来阻塞其他的线程执行，当它释放锁，再调用内核陷阱让其他线程可以继续执行。每次调用内核的消耗都是巨大的，应尽量避免的。
> dispatch queue：是一个 App 通用，操作系统管理的队列，它负责了操作在进程空间内的顺序执行，非必要不会有内核调用的开销。
{: .prompt-tip}

你或许已经发现，将两个任务执行到一个串行队列不是并发。但同样的，如果两个线程使同时使用一个锁，那么线程提供的任何并发性都会丢失或者显著降低。更重要的是：**线程模型需要创建两个线程，这将同时消耗内核与用户空间的内存。 dispatch queue 则不会为它使用的线程付出同样的内存代价，且它使用的线程保持繁忙而不被阻塞**

dispatch queue 的其他特点如下：

- dispatch queue 内部的任务执行顺序是 FIFO，但是不同的队列之间是并发执行的。
- 系统决定某一时刻最大的并发数量。如：你可以创建100个队列执行100个任务，此时除非真的有超过100效能核心，否则并不会同时并发执行100个任务。
- 系统启动一个新任务的时候会考虑其队列的优先级。
- 任何被添加到队列的任务必须处于 `ready` 状态。(需要注意，这点和 NSOperation 对象不同)
- **私有队列是引用计数对象**。其需要自行管理引用计数，需要小心的是 dispatch source 也会触发 dispatch queue 引用计数的增加，你需要确保 dispatch source 被取消并且保证 retain/release 的调用平衡。

## 队列相关技术

除了 dispatch queue， GCD 还提供了其他几种队列相关技术来管理你的代码。

1. Dispatch groups：一个 dispatch group 是监控一组 block 调用完成的方法。它的同步机制可以让你按需监听(同步/异步)block的完成状态，并执行依赖结果的其他代码
2. Dispatch semaphores：dispatch semaphore 是类似传统 semaphore 且更加高效的信号量。它只会在派发线程因信号量不可用需要被阻塞的时候才调用到内核陷阱。`传统信号量每次都调用内核，消耗高`
3. Dispatch sources：dispatch source 会在响应特定系统事件的时候生成通知。你可以利用 dispatch source 来监听特定系统事件，如：进程通知/系统信号/文件描述符事件等等。当系统事件发生的时候，dispatch source 会将你设置的监听代码异步提交到指定的 dispatch queue 执行。

## 用 Block 实现要执行的任务

Block 是 C 语言拓展，可以被用在 C/ OC/ 或者C++ 代码中，其简化了任务的创建。 Block 语法看起来像函数指针，实际上它是匿名的 OC 对象，其由编译器创建和管理。编译器会封装你的代码(包含你block中相关的数据)，Block 可以被 copy 到堆区，也可以在App作为参数传递。

Block 的一项关键优势是能访问其作用域之外的变量。Block 的语法就和普通代码块一样，当你将 Block 写到函数或者方法里面的时候，它可以访问其 父作用域 的变量。被 Block 访问的变量会被拷贝到 Block 本身在的堆区数据结构中，所以它能访问外部变量。将 Block 添加到调度队列时，这些值通常必须以只读格式保留。但是，同步执行的块也可以使用预先添加了__block 关键字的变量将数据返回到父调用范围. 下面是 Block 的简单应用，Block 相关的可查看[Blocks Programming Topics](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/Blocks/Articles/00_Introduction.html#//apple_ref/doc/uid/TP40007502)

```objc
int x = 123;
int y = 456;
 
// Block declaration and assignment
void (^aBlock)(int) = ^(int z) {
    printf("%d %d %d\n", x, y, z);
};
 
// Execute the block
aBlock(789);   // prints: 123 456 789
```

当你设计你的 Block 任务的时候需要注意以下几点：

- 如果执行异步任务，block 尽量不要捕获大的数据结构（或指针类型的结构），标量小数据是比较安全的。有可能你中Block中执行的时候，捕获的指针被释放了。
- Dispatch queue 会拷贝 block 到堆区，并在执行完成之后释放它，你不需要手动拷贝block
- dispatch queue 执行效率比自己手动创建 Thread 更高，但是你如果block执行的任务过于小，不如直接在代码中计算
- Block 中不要想要给底层 Thread 绑定数据给其他的Block使用，如果你有此需求，可以使用 dispatch queue 本身提供的 context pointer
- Block 内部如果创建很多 OC 对象，你需要自己管理内存，可以将创建对象的方法用 @autoreleasepool{} 包装起来。 queue 本身虽然有内存管理，但并不能保证啥时候释放池子。

## 创建并管理 Dispatch Queues

将自己的任务添加到队列之前，要先决定使用何种队列。队列可以并行/串行的执行你的任务，你也可以对其设置特定属性来执行特定的任务。

### 获取全局并发队列

并发队列可以让你的任务并行执行。它本身是 FIFO 顺序的，在任意时刻它真正并发执行的任务数是依赖系统负载和硬件可用核心数，还有进程要处理的任务数量，还有其他串行队列中任务数量/任务优先级。基于以上变量：并发队列同一时刻执行的任务数是个变量。

并发队列又称作全局队列，由系统提供，共四种优先级。获取函数如下：

```objc
dispatch_queue_t aQueue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);

// 四种优先级，从高到低如下，指定不同优先级会影响任务的执行顺序。
 DISPATCH_QUEUE_PRIORITY_HIGH 
 DISPATCH_QUEUE_PRIORITY_DEFAULT
 DISPATCH_QUEUE_PRIORITY_LOW 
 DISPATCH_QUEUE_PRIORITY_BACKGROUND 
 
// dispatch_get_global_queue 函数的第二个参数是作为未来拓展的保留参数，当前传 0 即可
```

派发队列本身是引用计数的方式管理内存的。但是对于全局队列的计数由系统管理，你执行的任何 release/retain 方法都会被忽略。

### 创建串行队列

当你需要按特定顺序执行任务的时候，串行队列就很有用了。串行队列同时只会执行一个任务，顺序为 FIFO。**对于共享资源或者可变数据的访问你应该使用串行队列来保护数据一致性，而不是用锁。因为串行队列本身能确保任务按预定的顺序执行。只要你异步的向串行队列中添加任务，它就不会导致死锁。**

串行队列是私有队列，你需要自己创建并管理它。你可以创建任意数量的串行队列，但是需要考虑任务与队列本身的平衡。不要为每一个任务创建一个队列，那会浪费系统资源，如果你要多个任务并发执行的话就用全局并发队列就好。创建串行队列的时候最好标记上队列用途，如数据保护，项目内关键行为同步等。

```objc
dispatch_queue_t queue;
queue = dispatch_queue_create("com.example.MyQueue", NULL);

// dispatch_queue_create 函数共有两个参数，
// 第一个是队列名，可以标记用途，debug 的时候你自己的队列名会被标记出来
// 第二个参数是未来拓展的保留参数，现在传 NULL。
```
 
### 获取运行时刻的通用队列（Common Queues）

Grand Central Dispatch 提供了几个函数来获取项目中的通用队列。

- [dispatch_get_current_queue](https://developer.apple.com/documentation/dispatch/1493248-dispatch_get_current_queue) 函数可用于 debug/测试当前队列的标识符。在队列的 block 中调用，会返回 block 被提交到的队列。在 block 外调用返回 App 的默认并发队列。
- dispatch_get_main_queue 函数获取 App 主队列。其在 Cocoa 项目，或者终端项目手动调用 [dispatch_main](https://developer.apple.com/documentation/dispatch/1452860-dispatchmain) 函数，又或者在主线程中配置了 runloop 的的项目中会自动生成并绑定到主线程。
- dispatch_get_global_queue 函数，可以获取全局并发队列

### 派发队列的 内存管理

Dispatch queues 和其他 dispatch objects 是通过引用计数来管理内存的。当你创建串行队列的时候，它的初始计数为 1，你使用 `dispatch_retain` and `dispatch_release` 函数来增/减其引用计数。当计数为0的时候，它就会被系统回收。

> 注意
> 对于全局并发队列/主队列由系统维护，代码中的release/retain 会被直接忽略

### 给队列保存自定义的上下文信息

所有 dispatch objects (including dispatch queues) 允许你关联自定义信息。可以使用 [dispatch_set_context](https://developer.apple.com/documentation/dispatch/1452807-dispatch_set_context) 和 [dispatch_get_context](https://developer.apple.com/documentation/dispatch/1453005-dispatch_get_context) 函数。系统不会使用你的自定义数据，所以你需要自己维护自定义数据的创建和销毁时机。

你指定的数据可以是随意的数据，或者是 OC 对象的指针。你可以使用队列的终结器函数(finalizer function) 去移除你的自定义数据。

### 给队列提供终结器函数/清理函数(finalizer function)

当你创建串行队列之后，你可以附加一个终结器函数(finalizer function)来执行队列被释放时期的清理工作。队列本身都是引用计数的对象，你可以通过 [dispatch_set_finalizer_f](https://developer.apple.com/documentation/dispatch/1453100-dispatch_set_finalizer_f) 函数来指定队列被销毁时候的关联数据清理工作，当队列引用计数为 0 时，dispatch_set_finalizer_f 指定的函数会被调用(如果你真的指定来它)。 示例如下

```objc
void myFinalizerFunction(void *context)
{
    MyDataContext* theData = (MyDataContext*)context;
 
    // Clean up the contents of the structure
    myCleanUpDataContextFunction(theData);
 
    // Now release the structure itself.
    free(theData);
}
 
dispatch_queue_t createMyQueue()
{
    MyDataContext*  data = (MyDataContext*) malloc(sizeof(MyDataContext));
    myInitializeDataContextFunction(data);
 
    // Create the queue and set the context data.
    dispatch_queue_t serialQueue = dispatch_queue_create("com.example.CriticalTaskQueue", NULL);
    dispatch_set_context(serialQueue, data);
    dispatch_set_finalizer_f(serialQueue, &myFinalizerFunction);
 
    return serialQueue;
}
```

## 将任务(Tasks)添加到队列

要执行任务，你必须将它添加到适合的队列。你可以同步/异步，单独放到队列/放到队列组的方式来派发任务。放入队列后，任务会尽可能快的被执行。

### 添加单任务到队列

创建任务的方式有两种： block 或者 function。 
派发任务的方法也有两种： asynchronously 或者 synchronously.

你应该尽可能使用 `dispatch_async` 和 `dispatch_async_f` 函数来异步派发任务。它们被添加到队列后会在其他线程被调度，你可以继续去干其他的任务。这在主线程需要响应用户事件的场景非常重要。

除了异步派发，你也可以使用同步派发来防止条件竞争或其他同步错误。**`dispatch_sync` 和 `dispatch_sync_f` 方法会将任务同步添加到指定队列，它们会阻塞当前线程，直到特定任务执行完成**

> 注意：
> 如果计划执行任务的队列和你当前调用派发任务的队列相同，永远不要调用 dispatch_sync 和 dispatch_sync_f 函数来派发任务。
> 这对串行队列来说，会直接造成死锁。因为它们会阻塞当前线程，直到特定任务执行完成。同样并发队列也应该避免这样调用

```objc
dispatch_queue_t myCustomQueue;
myCustomQueue = dispatch_queue_create("com.example.MyCustomQueue", NULL);
 
dispatch_async(myCustomQueue, ^{
    printf("Do some work here.\n");
});
 
printf("The first block may or may not have run.\n");
 
dispatch_sync(myCustomQueue, ^{
    printf("Do some more work here.\n");
});
printf("Both blocks have completed.\n");
```

### 任务执行完成后执行一个完成回调

任务别加入到队列后会独立于当前代码执行，所以你可能需要当任务执行完成之后接到通知，以便可以合并结果。在传统的并发编程中，你会需要使用一个回调机制。 在派发队列中你可以使用完成回调。

完成回调是你原任务操作完成时候，提交到队列到另一段代码。调用方在开始任务的时候提供一个完成 block 回调。你需要做的就是当任务完成之后将你的完成回调添加到队列执行。代码如下：

```objc
// 完成任务的时候，指定在某个队列，执行某个回调
void average_async(int *data, size_t len,
   dispatch_queue_t queue, void (^block)(int))
{
   // Retain the queue provided by the user to make
   // sure it does not disappear before the completion
   // block can be called.
   dispatch_retain(queue);
 
   // Do the work on the default concurrent queue and then
   // call the user-provided block with the results.
   dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
      int avg = average(data, len);
      dispatch_async(queue, ^{ block(avg);});
 
      // Release the user-provided queue when done
      dispatch_release(queue);
   });
}
```

### 并行执行循环迭代

当执行固定数量的 loop 迭代也可以使用并发队列来提升性能。例如，for 循环处理事件。

```objc
for (i = 0; i < count; i++) {
   printf("%u\n",i);
}
```

如果每次循环处理的任务都是独立的，并且它们执行的顺序并不重要的时候。你可以使用 `dispatch_apply` or `dispatch_apply_f` 函数替换 for 循环。 你需要提供一个 block/function 和一个 queue 来执行对应的 loop 迭代。如果你将迭代任务提交到并发队列，它就可能并行执行多个任务，从而提升性能。

你也可以将任务提交到串行队列，串行实际执行是按顺序执行的，与你直接 for 循环执行是没有性能提升的。通常是将任务放到并发队列去执行。

> Tip
> 和 for 循环一样，dispatch_apply 和 dispatch_apply_f 函数也会在所有循环执行完才停止。你要自己保证不要将迭代任务提交到当前串行队列中，这样会造成线程死锁。
> 因为这两个函数会阻塞被提交到的线程。所以你如果在主线程调用也要小心，它会阻塞你的主线程及时响应。如果你的任务比较耗时，考虑在子线程中调用。

dispatch_apply 函数的使用语法如下： count 是要总共迭代次数，queue是要执行的队列，i 是当前执行的迭代序号。

```objc
dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
 
dispatch_apply(count, queue, ^(size_t i) {
   printf("%u\n",i);
});
```

作为使用者，需要自己平衡迭代中做的事，如果任务太小，将它添加到队列本身有性能消耗，不如直接就地for循环运行。同样你可以使用跨度（striding）来提升每个迭代处理的工作量。这样你在每个迭代中执行一组迭代任务，并减少总的迭代数量。例如：一个100次的迭代任务，可以等价于：跨度为 4，共计25次的迭代任务。

### 在主线程执行任务

GCD 会在程序的主线程提供特殊的派发队列(主队列)，你可以在主队列执行你的任务。 主队列会自动创建并由创建了 runloop 的程序自动管理其中任务。如果你不是创建的 Cocoa 程序，你就需要自己执行 [dispatch_main](https://developer.apple.com/documentation/dispatch/1452860-dispatchmain) 函数来显式地触发主队列执行，你可以随意往主队列添加任务，但是如果不调用此方法，其任务将不会执行。你可以创建一个 CMD 程序来测试。

你可以使用 `dispatch_get_main_queue` 函数来获取主队列，添加到里面的任务会在主线程执行，你可以用主队列来同步 App 的各个任务的执行结果。

### 在你的任务中使用 OC 对象

GCD 默认支持 Cocoa 的内存管理技术，所以你在 block 任务中可以自由使用 OC 对象。 每个队列会自己维护自己的 autorelease pool 来确保添加到释放池的对象能在某刻释放，但并不保证具体的释放时机。

如果你的程序是内存受限的，并且你的 block 任务会创建较多临时对象。你应该自己在 block 任务中创建 autorelease pool 来自己管理临时对象的内存。

## 队列的挂起 和 重启

如果你要临时暂停队列任务的执行，可以挂起队列。

- [dispatch_suspend](https://developer.apple.com/documentation/dispatch/dispatchobject/1452801-suspend) 挂起队列，增加挂起引用计数
- [dispatch_resume](https://developer.apple.com/documentation/dispatch/dispatchobject/1452929-resume) 重新启动队列，减少挂起引用计数

挂起队列函数，会增加队列本身的 `suspension reference count` (挂起引用计数)，当计数大于0，队列会处于挂起状态，所以你需要自己维护挂起和重启的平衡。**挂起只会影响未被执行的任务，已经执行的任务不回被暂停**

## 使用调度信号量(Dispatch Semaphores)来调节有限资源的使用

如果提交给调度队列的任务访问某个有限资源，则可能需要使用调度信号量来调节同时访问该资源的任务数.

Dispatch semaphore 工作方式和传统信号量很像，只有一个区别。 dispatch semaphore 在资源可用的时候获取锁消耗的时间更少。这是因为 GCD 在资源可用的时候不会向下调用 kernel 陷阱。只有当资源不可用时，操作系统需要暂停你的线程来等待信号量释放的信号。

使用调度信号量的语法如下：
 
- 创建信号量： [dispatch_semaphore_create](https://developer.apple.com/documentation/dispatch/dispatchsemaphore/1452955-init), 你可以指定一个正数，表示可同时访问的资源数量
- 每个任务执行的时候，调用 [dispatch_semaphore_wait](https://developer.apple.com/documentation/dispatch/1453087-dispatch_semaphore_wait) 等待信号量
- 当 dispatch_semaphore_wait 函数返回时，获取资源并执行你的工作
- 当你访问资源并完成任务，释放它，并给信号量发送信号，方法：[dispatch_semaphore_signal](https://developer.apple.com/documentation/dispatch/dispatchsemaphore/1452919-signal)

例如：由于 App 只能访问部分文件描述符。如果你有个任务要处理大量文件。你并不想一次性打开如此多的文件。那你可以使用信号量来限制同时打开文件描述符的最大数量。示例代码如下

```objc
// Create the semaphore, specifying the initial pool size
dispatch_semaphore_t fd_sema = dispatch_semaphore_create(getdtablesize() / 2);
 
// Wait for a free file descriptor
dispatch_semaphore_wait(fd_sema, DISPATCH_TIME_FOREVER);
fd = open("/etc/services", O_RDONLY);
 
// Release the file descriptor when done
close(fd);
dispatch_semaphore_signal(fd_sema);
```
你创建信号量的时候，需要制定可用资源数量。这个数量会成为信号量的初始量。每次你执行 dispatch_semaphore_wait 函数，会将可访问数量减少1， 当结果成为负数，它会调用 kernel 来阻塞当前线程。 相反，dispatch_semaphore_signal 函数会增加可访问量(+1)，如果有任务被阻止并等待资源，其中一个任务随后将被解除阻止并允许执行其工作.

## 等待成组队列任务的完成

Dispatch groups 可以阻塞线程，直到一个或多个任务都执行完。你可以将此特性使用在需要监听所有任务都执行完的地方。例如：对于数据的处理，可能有多个任务要处理数据，而你要在所有任务都处理完成之后处理最终结果。使用 dispatch queue 的另一种方法是作为线程连接(thread joins)的替代方法。您可以将相应的任务添加到调度组，并等待整个组，而不是启动几个子线程，然后加入每个子线程。

```objc
dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
dispatch_group_t group = dispatch_group_create();
 
// Add a task to the group
dispatch_group_async(group, queue, ^{
   // Some asynchronous work
});
 
// Do some other work while the tasks execute.
 
// When you cannot make any more forward progress,
// wait on the group to block the current thread.
dispatch_group_wait(group, DISPATCH_TIME_FOREVER);
 
// Release the group when it is no longer needed.
dispatch_release(group);
```

## 派发队列 与 线程安全

在派发队列的章节谈论线程安全可能会显得奇怪，但它确实是相关的内容。任何时候你在App中实现并发编程，以下几点你都要牢记。

- Dispatch queue 本身是线程安全的，你可以在任何线程中提交任务，而无需提前加锁(或者同步)访问这个队列
- 不要在当前队列调用 [dispatch_sync](https://developer.apple.com/documentation/dispatch/1452870-dispatch_sync) 函数添加新任务，这样会造成当前线程的队列死锁。如果需要在当前队列中添加任务，就使用 dispatch_async 函数。
- 对于添加到队列中的任务，不要使用锁。虽然使用锁是安全的，但是你获取锁的时候，如果锁不可用就会完全阻塞串行队列。同样对于并发队列，等待锁可能会阻塞其他任务执行。如果需要同步部分代码，就使用串行派发队列。
- 任务执行的时候避免获取底层线程的信息(虽然你可以这样做)，更多 GCD 对于线程的兼容性，参考[Compatibility with POSIX Threads](https://developer.apple.com/library/archive/documentation/General/Conceptual/ConcurrencyProgrammingGuide/ThreadMigration/ThreadMigration.html#//apple_ref/doc/uid/TP40008091-CH105-SW18)

--- END ---






