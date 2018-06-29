## Dispatch Queues
GCD处理异步任务和并发任务的关键载体，dispatch queue 是类类结构，对任务的管理遵循FIFO。

* Serial Dispatch Queue(串行队列)：串行队列中的任务按照FIFO执行，实际上为单线程执行，即每次从队列中取出一个task执行，serial dispatch queue 彼此之间是并发的。
* Concurrent Dispatch Queue(并发队列)：并发队列一次并发执行一个活着多个任务，系统提供了4个全局的并发队列。对于concurrent queue，其管理的task可能在多个不同的thread上执行，至于具体管理的thread数量视系统资源而定
* Main Dispatch Queue(主队列)：主队列中的task全部在main thread 中运行

说明：
 
* 同步和异步描述的是task与上下文的关系，所以对于GCD同步队列和异步队列是没有意义的
* 串行队列上的tasks并非只在同一个thread上执行，对于同步请求的任务，其运行的task往往与上下文是同一个thread；对于异步请求的任务，其运行的task往往是不同的thread


## dispatch_sync和dispatch_async

	// dispatch task synchronously

	dispatch_sync(someQueue1, ^{

    // do something 1
    
	});

	// do something 2

	// dispatch task asynchronously

	dispatch_async(someQueue2, ^{

    	// do something 3
    
	});

	// do something 4

同步任务 do something 2 一定会在 do something 1完成之后执行。
异步任务 do something 4 会立即执行，而不会等待 do something 3 执行完。
### 使用时机
* 若派发的task耗时长，并且不想上下文被阻塞就使用dispatch_async
* 如果需要处理的代码比较短，想实现代码保护（线程安全）使用同步任务

##dispatch_barrier_async


	void dispatch_barrier_async(dispatch_queue_t queue, dispatch_block_t block);



dispatch_barrier_asynvc 提交一个任务到指定的并发队列，并在该队列中创建一个同步点，当并发队列调度遇见一个barrier任务，会延迟执行barrier任务，等待所有在barrier之前提交的任务执行结束后，调度barrier任务开始执行。

调用这个函数会在任务提交后立即返回，不会等待任务执行。当barrier任务到并发队列的最前端，他不会立即执行。相反，队列会等到所有当前正在执行的任务执行结束。然后barrier任务才会执行，而且，所有在barrier之后提交的任务会等到barrier任务结束后才执行。


需要说明的是

* barrier任务在并发队列中相当于执行了一段同步的任务，所以需要与barrier任务互斥的任务需要被提交至同一个并发队列才会有效果。
* 如果barrier任务被提交至一个串行队列或者全局并发队列，这个行数等同于dispatch_async函数


##Dispatch Group

### dispatch_group_wait

dispatch_group_wait 会阻塞当前线程，直到组里面的所有任务都完成或者等到某个超时发生。

* 自定义串行队列：适合当一组任务完成时发出通知
* 主队列（串行）：适合这种情况。但如果需要同步地等待所有工作完成时，那么这些任务不应当出现在主队列中，因为不能阻塞主线程。
* 并发队列：适合dispatch group和完成时通知



### dispatch_apply

dispatch_apply 是同步的，和普通的for循环一样，只会在所有工作都完成后才会返回。

当任务内部计算任何给定数量的工作的最佳迭代数量时，必须要注意，因为过多的迭代和每个迭代只有少量的工作会导致大量的开销以致它能够抵消任何因病发带来的收益。

dispatch 适用情形：

* 自定义串行队列：串行队列会完全抵消dispatch_apply的功能
* 主队列：不适合使用dispatch_apply
* 并发队列：对于并发循环来说是很好的选择，特别是当你需要追踪任务的进度时


## deadlock

使用dispatch_async（异步） 的方法向队列提交一个任务，队列会被唤醒。在队列被唤醒后，如果没有没有线程可用，那么每秒产生一个线程来处理队列里的所有任务，直到线程池用完为止。每个线程处理完自己的任务以后，会进入等待状态，直到有任务执行才会被唤醒。

所有队列在初始化时，已经制定了一个global队列来执行自己所有的任务。

串行队列是一个特殊的队列和一个特殊的任务。串行队列被作为一个任务放入global队列的链表里进行操作，在执行到串行队列时，会使用循环挨个执行串行队列中的任务，所以串行队列的所有任务都是在一个线程里执行。

dispatch_sync提交的任务是在当前线程中执行的。实际上dispatch_sync在当前线程a提交了一个任务到目的队列，然后阻塞当前线程a，目的队列如果为并发队列，立即唤醒当前线程a，并将任务作为函数在线程a执行（这么做的原因是基于thread－localside effects， garbage collection等考虑），如果目的队列为串行队列，则等待队列中前面的任务完成后，唤醒线程a，执行任务（如果串行队列中之前的任务有需要在当前线程a中执行的，就会造成死锁）。但是，提交到主线程的同步任务必然是在主线程执行的，而不是当前线程。据查阅的大部分博客与guide doc，向一个串行队列提交同步任务的并没有现实的意义，并且向主队列提交同步任务会导致死锁，避免这样做。


* 案例一：


		NSLog(@"1"); // 任务1

		dispatch_sync(dispatch_get_main_queue(), ^{

    		NSLog(@"2"); // 任务2
    
		});

		NSLog(@"3"); // 任务3



向主队列提交一个同步任务，dispatch_sync向主队列提交了任务2，然后阻塞了主线程，主队列在等待任务2的完成来唤醒，任务2的执行需要主线程被唤醒，但此时主线程处于阻塞状态，造成死锁。

*  案例二：


		NSLog(@"1"); // 任务1

		dispatch_sync(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_HIGH, 0), ^{

    	NSLog(@"2"); // 任务2
    
		});

		NSLog(@"3"); // 任务3


在当前线程向全局并发队列提交同步任务2，dispatch_sync会阻塞当前线程，任务2会暂停DISPATCH_QUEUE_PRIORITY_HIGH以及该队列所在线程，然后唤醒当前线程，执行任务2，执行任务3.


* 案例三：



		dispatch_queue_t queue = dispatch_queue_create("com.demo.serialQueue", DISPATCH_QUEUE_SERIAL);

		NSLog(@"1"); // 任务1

		dispatch_async(queue, ^{

    		NSLog(@"2"); // 任务2
    
    		dispatch_sync(queue, ^{  
        		NSLog(@"3"); // 任务3
    		});
    
    		NSLog(@"4"); // 任务4
		});

		NSLog(@"5"); // 任务5


在当前线程向自定义串行队列提交异步任务［2，3，4］，在线程池中新建线程b来执行异步任务。 当执行到dispatch_sync时，向串行队列提交任务3，会阻塞当前线程b，任务3会暂停串行队列，然后唤醒当前线程b。但是任务3的执行需要等待任务4的完成，但是线程b被阻塞，任务4无法执行，导致死锁。

* 案例四：

		NSLog(@"1"); // 任务1

		dispatch_async(dispatch_get_main_queue(), ^{

    		NSLog(@"2"); // 任务2
    		dispatch_sync(dispatch_get_global_queue(0, 0), ^{
        		NSLog(@"3"); // 任务3
    		});
    		NSLog(@"4"); // 任务4
		});

		NSLog(@"5"); // 任务5



在当前线程向全局并发队列提交［1，异步任务，5］，异步任务中的任务是［2，同步任务，4］。


* 案例5


		dispatch_async(dispatch_get_global_queue(0, 0), ^{

    		NSLog(@"1"); // 任务1
    		dispatch_sync(dispatch_get_main_queue(), ^{
        		NSLog(@"2"); // 任务2
    		});
    		NSLog(@"3"); // 任务3
		});

		NSLog(@"4"); // 任务4

		while (1) {
		}

		NSLog(@"5"); // 任务5


1.主队列添加任务［异步任务，任务4，死循环，任务5］，向全局并发队列提交异步任务［1，2，3］

2.死循环会阻塞主线程，导致提交至主队列的任务2无法执行，任务3也无法执行

3.任务5由于主队列被阻塞也无法执行


# NSOperation and NSOperation Queue



## some concepts

* task: 一个简单的任务
* thread：一个操作系统提供的允许app在同一时间执行多个任务的方法
* process：一段正在执行的代码，可以由多个线程组成











	











