# C++多线程

2022.11.18



## 进程和线程的概念

### 程序和进程

进程可以理解为在内存中运行的程序。

- 程序是由指令和参数组成的带有逻辑的可执行二进制文件，是保存在磁盘中的。
- 当我们执行程序时，操作系统就会读取保存在磁盘中的文件到内存中，并开始执行其中的指令代码，就成为了进程。

**进程是操作系统为程序分配系统资源的最小单位**，这里的系统资源包括但不限于内存空间、IO缓冲区等。



### 线程

**线程是程序最小的执行单位**，程序的所有任务都要在线程内执行，CPU按照任务执行。这里的任务指的是代码块。

线程和进程不是相互包含的关系，他们是互相结合的，对程序的运行都很重要。进程中至少有一个线程，如果程序中没有用到多线程，那么主线程就是唯一的线程；线程不能脱离进程而存在，进程退出了或者被截杀了，其内部的线程也就消失了。

在一个进程内，所有线程共用进程的系统资源，非常和睦地共同执行程序任务；但对于CPU时间片来说非常特殊，因为一个CPU核心在同一时刻只能被一个线程占用，所以所有线程都在同时“抢”CPU资源（这里说的抢并不是真正意义上的抢，而是操作系统随机分配的，这其中还涉及到就绪队列等调度原理，此处不细究）。

如果只有一个CPU核心，那么所有线程都会等待被分配这一个CPU核心，但如果有多个核心，那么一个核心被占用了，还有其他核心可以分配，这就是多核心CPU在多线程场景下为什么比单核心CPU高效的原因。



### 多线程

多线程是在硬件上实现针对多任务并发的技术。在理解多线程是什么之前，我们需要先知道什么是并发，以及什么是并行和串行。

- 并发：多个任务的运行时间有重叠，就叫并发。这里的运行时间是指从开始任务到结束任务的那段时间，即使其中某些时间段该任务没有在运行。

  在操作系统中，任务可以看作是线程。并发思想是与处理器核心数量无关的，只要某段时间内存在线程的执行时间重叠就叫并发，这里的执行时间也是线程开始执行到线程结束执行的时间段，与期间线程是否被挂起是无关系的。

  从另一个角度说，并发也可以是在某个时间段内完成多个任务。

- 并行：在某时刻，有多个任务正在被执行，就叫并行。

  在操作系统中，并行一定依赖多线程和多个CPU核心(或CPU有超线程技术)，每个核心分别执行不同的线程，达到并行的效果。

- 串行：多个任务排队有序地被执行，就叫串行。串行必须是一个任务结束后再执行下一个任务。串行非常安全，但它的效率太低了。


并发不一定要使用多线程，但**并发中最高效的模式就是并行**，也就是使用多线程。但是并行需要硬件支持啊，如果任务数量远多于核心数量怎么办？所以人们为了满足并发，提出了分时计算的方式：CPU核心将自己的执行时间分成很多时间片，每个任务都需要抢时间片。抢到了就运行，没抢到就等待，这样也能实现并发。CPU核心数越多，可供任务们抢的时间片也就越多，执行效率也就越高效。CPU由上一个任务切换到下一个任务的这个过程也叫上下文切换。



并发是一种场景和思想，目的是充分使用CPU算力，多线程则是操作系统为了达到并发目的的一种运行方式，后文还会讲到同步和异步，则是线程实现并行的编程手段。



**为什么我们需要使用多线程？用多进程不行吗？**

|          | **优点**                                                     | **缺点**                                                     |
| :------: | :----------------------------------------------------------- | :----------------------------------------------------------- |
| **进程** | 进程间相互隔离，各顾各的运行，非常稳定安全                   | 比较庞大，每次进行上下文切换的时候就要保存进程的全部数据，而且如果我们频繁的切换、创建、退出进程，操作系统就要花大量时间在加载资源、分配资源和销毁资源上面；进程间进行数据传递时要用到套接字、管道等，导致数据传递效率低 |
| **线程** | 创建和销毁线程不会分配资源和销毁资源，对系统资源的冲击小；线程启动速度更快，退出也快；所有进程共享同一系统资源，能够实现高效的数据传递 | 因为共享同一系统资源，容易导致死锁，不安全。需要程序员花更多心思去理清程序运行逻辑。 |

当然，线程数也不是越多越好，在拥有一定量的任务量时，最完美的情况下，需执行线程数和CPU核心数相等。如果线程数过少，会导致CPU使用率低下，造成一核有难八核围观的窘境；如果线程数过多，会导致其他进程很难抢到CPU资源。

> 不同操作系统对线程的实现不一样，Windows对多线程的支持就很好。但在早期的Linux下说没有线程的概念的，直到Linux2.0才逐渐有了线程的概念，线程在Linux下可以看作是一个轻量级的进程。虽然底层不一样，但是这并不影响我们使用多线程。

在操作系统的实际运行中，线程数量一般会远远超过CPU核心数，但我们却感觉不到卡顿，是因为大多数线程都在休眠或等待用户做出指示的状态，主动放弃CPU时间片，把计算资源让给用户。当启动了需要大量消耗CPU资源的应用程序时，比如大型游戏或者大型软件，程序里面的线程就会争夺CPU时间片以争取在最短的时间内完成任务，这时候用户在进行其他操作时就能明显感觉到卡顿。



## C++下的多线程

C++11之前是没有多线程概念的，实现多线程需要用到系统的api，如Windows操作系统的`windows.h`和Linux下的`pthread.h`。但C++11添加了对线程的语言级定义，解决了跨平台问题。

但C++11的线程是基于系统的api封装来的，所以依然需要依赖系统的库，比如Linux下你需要链接pthread库。

> 但windows下没有pthread库，你的项目需要构建跨平台，你已经使用了`std::thread`，但你仍然需要链接系统的库。你不想对每个系统都单独写一个编译脚本，这时时怎么办呢？如果你使用CMake，你可以使用CMake引入的包Threads::Threads：
>
> ```
> target_link_libraries({PROJECT_NAME} Threads::Threads)
> ```
>
> CMake会自动为你匹配系统的多线程库并链接

C++11添加了5个头文件用于操作多线程，5个头文件分别定义了5个类：thread、mutex、atomic、condition_variable和future，**这五个类都在std的命名空间下**。

- thread：用于创建、管理线程

- mutex：互斥锁、读写锁，用于保护共享数据、线程间同步操作

- atomic：原子类操作

- condition_variable：条件变量，和mutex目的一样

- future：用来定义多线程时，获取多线程的返回值。



## Thread库

与C语言的多线程代码不同，C++中的线程被封装成了类，而且将相关函数都封装在了类内部。想要在C++中使用多线程，我们需要先了解线程类thread。

#### class thread

- **构造函数**

    - **`thread() noexcept`** 

        空构造，字面意思是指创建一个线程但啥也不干，如果创建后没有接手其他线程的话，大概率会被编译器优化掉

    - **`thread( thread&& other ) noexcept`**

        将other线程的所有权移交给新创建的线程

    - **`thread( const thread& ) = delete`** 

         因为线程比较特殊，不能存在相同的线程，所以也就不能有相同的线程对象存在。这条语句告诉程序员不能对thread类进行拷贝构造。

    - **`template<class Function, class... Args>`**
        **`explicit thread(Function&& f, Args&&... args)`** 

        这个构造函数是我们在实际开发中用的最多的。创建线程是为了执行多任务。用函数来包装任务再合适不过了。这个构造函数需要我们传入函数和参数，我们为thread传递的函数可以是普通函数、类的成员函数(包括非静态和静态)、lambda匿名函数、仿函数。
        
        > 可变参数模板
        >
        > 此构造函数使用了可变参数模板：**`template<class Function, class... Args>`**。这个模板意味着函数可以接收任意数量、任意类型的参数。
        
        请注意此构造函数前面的`explicit`关键字，这表示用于构造此类的参数不允许被隐式转换。
        
        > **`explicit`关键字**用于类的构造函数和类型转换函数中。当我们对类进行实例化对象时，会调用类的构造函数。比如有一个类：
        >
        > ```
        > class A{
        > 	A(int i){ num = i };
        > 	int num;
        > };
        > ```
        >
        > 当我们使用`A a(100)`来构造一个对象a时，可以创建成功；但是如果我们使用`A a = 100`时，编译器尝试把一个int类型的值赋给A类型的对象，会尝试使用100来创建一个对象，这就是隐式转换，把一个int类型的值转变成了A类型。
        >
        > 有时候这样的隐式转换会导致很多的问题。我们可以在构造函数前面加上`explicit`关键字来阻止隐式转换，一旦参数类型不匹配就会报错。
        >
        > `explicit`关键字只对参数个数为1的构造函数有效，当构造函数的参数多于1个时，不能发生隐式转换，也就没有这个问题。

    **一个thread类一旦被构造，就会立刻变成就绪态请求CPU时间片，然后开始运行。**

- **析构函数**

    **`~thread()`**

    销毁thread对象。如果线程正在运行(`joinable() == true`)，会被终止(`terminate()`)。

- **成员对象**

    thread只有一个成员类：class id。这个id类记录thread对象的线程ID。注意，这个ID并不是操作系统的底层线程ID，而是C++为每个线程定义的ID。要获得操作系统的线程ID，请看下面的`native_handle()`函数。

- **成员函数**

    - **`thread& operator=(thread&& other ) noexcept`** 

        other必须是一个右值，表示用this接管other。如果other正在运行(`other.joinable() == true`)，会被终止(`other.terminate()`)，再被接管。

    - **`thread::id get_id() const noexcept`**

        这个函数用于返回目标thread对象的线程ID

    - **`native_handle_type native_handle()`** 

        这个函数用于返回底层线程句柄，也说白了就是操作系统的线程ID。这是为了和操作系统的相关API进行兼容而加入的。

        > C++11虽然加入了并发支持库，但这是在操作系统提供的API上进行封装得来的，而且在C++11出来之前，多线程的操作系统API就已经很完善了，相比之下C++11的并发支持库就有很多不够完善的地方，比如Linux下的多线程库`pthread.h`就有`terminate()`这个函数用于终止一个线程，但C++11是没有这个函数的。有时候程序员想要使用操作系统为我们提供的API时，就不得不使用操作系统的某些参数，这时候`native_handle()`就派上用场了。

    - **[静态]`static unsigned int hardware_concurrency() noexcept`**

        这个函数用于返回最大支持的线程数量，通俗讲这个值就是CPU的核心数，仅供参考

    - **`bool joinable() const noexcept`**

        若 thread 对象标识活跃的执行线程则为 true ，否则为 false 。

        就是 thread 对象内部在维护一个线程，则为true。

        > 比如thread对象在移动后，旧对象不再维护线程，此时`joinable()`返回false

        具体而言，若 `get_id() != thread::id()` 则返回 true 。

    - **`void join()`**

        这个函数不能被线程自己调用，只能被其他线程调用。

        **被其他线程调用后，调用此函数的线程将被阻塞直到本线程结束运行。**

        如果有多个其他线程都调用了本线程此函数，会导致未定义行为。

        <u>调用后，己线程的id成员对象会变为0，导致`joinable()`会变为`false`。</u>

        如果对`joinable() == false`的线程调用此函数，会导致程序错误。

    - **`void detach()`**

        这个函数和**`void join()`**是相对的。也是不能被线程自己调用，只能被其他线程调用。

        **被其他线程调用后，本线程将会被分离出去独立运行，将管控权移交给C++运行时库，若本线程退出，本线程的所有资源会被操作系统释放，其他线程不能获取到本线程的执行结果。**

        <u>调用后，己线程的id成员对象会变为0，导致`joinable()`会变为`false`。</u>
    
        如果对`joinable() == false`的线程调用此函数，会导致程序错误。
    
        ==**请注意，线程必须要且只能调用join()或detach()中的任意一个，其目的是回收线程的资源，比如线程的内存空间。如果不进行回收，子线程运行完任务函数之后就会变成僵尸线程**==
    
    - **`void yield()`**
    
        当前线程主动放弃占用CPU，由运行态转为就绪态，操作系统调度另一线程继续执行。
    
    - **`sleep_until() 和 sleep_for()`** 
    
        前者是休眠到哪个时间点，后者是休眠一段时间。都要结合chrono库进行使用。

​	

#### 命名空间this_thread

在 C++11 中不仅添加了线程类，还添加了一个关于线程的命名空间this_thread，在这个命名空间中提供了四个公共的成员函数，通过这些成员函数就可以对当前线程进行相关的操作了。

- **成员函数**

    - **`thread::id get_id() const noexcept`**

        这个函数用于返回当前线程自己的ID，和thread的`get_id()`成员函数功能类似，用法如下

        ```
        int main() {
        	cout << "主线程id：" << this_thread::get_id() << endl;		 // 获得当前线程的线程ID
        	thread t(func, 5);
        	cout << "创建子线程成功，子线程id：" << t.get_id() << endl;    	// 获得其他线程的线程ID
        	t.join();
        	return 0;
        }
        ```
    
    - **`void yield()`**
    
        让当前线程主动放弃占用CPU，由运行态转为就绪态，操作系统调度另一线程继续执行。
    
    - **`sleep_until() 和 sleep_for()`** 
    
        前者是休眠到哪个时间点，后者是休眠一段时间。都要结合chrono库进行使用。



#### 线程的生命周期

进程的5个状态：

- 新建态：进程刚被创建出来的状态

- 等待态：正在等待系统资源的调度，比如读取文件内容

- 就绪态：已经为运行做好充分的准备，在等待获取CPU使用权的状态

- 运行态：正在占用CPU运行的状态

- 终止态：进程运行结束

线程的生命周期和进程非常相似。我们可以把线程的状态分为5个状态：新建态、阻塞态、就绪态、运行态和死亡态。

- 新建态：线程刚被创建出来的状态

- 阻塞态：由于某种原因被阻塞，可以是`sleep()`、线程同步阻塞(锁)、请求资源后等待回应阻塞

- 就绪态：已经为运行做好充分的准备，在等待获取CPU使用权的状态

- 运行态：正在占用CPU运行的状态

- 死亡态：线程的任务函数执行完毕或出现异常后被终止就会进入死亡态。**处于死亡态的线程无法重新运行**。如何标记为死亡态？当`joinable() == true`时，说明线程是“存活”的，你可能不太理解为什么`joinable()`可以判断线程的死活，请看下文的比方。

当一个线程thread对象被创建后，立马就由新建态变成了就绪态。然后就在运行态-就绪态和阻塞态之间来回切换，直到任务函数被执行完毕，进入死亡态。

值得注意的是，线程死亡不代表被析构，线程对象的析构和常规对象一样，为超出作用域时析构。打个比方，假设有一个线程阿伟，阿伟要么度过了他的一生，要么中途夭折，现在他去世了，那么他应该被析构吗？很显然不行，因为阿伟还有尸体留在世上。当宣判了阿伟的死亡后，我们要么为他进行殡葬(`join()`)，在他的葬礼上宣读他的一生都干了什么，然后进行埋葬，从此世界上再无阿伟(通过`join()`来回收线程)；或者阿伟没有任何亲朋好友，完全的自由身(`detach()`)，他死亡后没有人为他举行葬礼，那么他的尸体就只能交给大自然去分解(交给操作系统管理)。这就是为什么必须要对线程进行`join()`或`detache()`操作的原因。当所有人都忘记了阿伟时，阿伟就被析构了，仿佛没有在世界上存在过一样。

如果对线程使用`join()`，当线程死亡时会立刻被其他线程回收，所以`joinable() == true`时，线程还是存活的。如果对线程使用`detache()`，那线程的死活也不归咱管了，而且咱想管也管不着，我们就默认他是死亡的就行。这就是为什么`joinable()`可以判断线程的死活了。

如果不对死亡的线程进行`join()`或`detache()`，那么这个线程就会变成僵尸线程。

> 在Linux下的C语言里，`pthread_join()`函数既可以阻塞调用线程等待子线程，还可以用于回收子线程的运行结果。然而C++11的线程类的`join()`函数并没有回收资源的功能，只能单纯地阻塞等待。
>
> 那么C++11的回收线程运行结果的功能在哪里呢？在future章节会进行讲解。



## 创建线程

创建线程就是实例化thread对象。前文我们了解到thread类的构造函数，抛开移动构造和空构造不讲，真正实例化一个thread对象使用的是这个构造函数：

```
template<class Function, class... Args>
explicit thread(Function&& f, Args&&... args)
```

此构造函数要求我们传入一个任务函数f，且使用了可变参数模板来传入任务函数f的参数args。

任务函数可以是普通函数、函数指针、类的静态成员函数、类的非静态成员函数、匿名函数lambda表达式、对类重载了`()`运算符的仿函数。

假设我们的任务函数为`func()`：

```
int func(int i = 10) {
	thread::id t_id = this_thread::get_id();		// 获得自己的线程ID
	int res = 0;
	for (int j = 1; j <= i; ++j) {
		res += j;
		this_thread::sleep_for(chrono::milliseconds(150));		//休眠150毫秒
	}
	cout << "子线程(" << t_id << ")的运行结果:" << res << endl;
	return res;
}
```

#### 使用普通函数

```
thread t(func, 5);
t.join()
```

#### 使用函数指针

```
int (*pfunc)(int) = func;
thread t(pfunc, 5);
t.join();
```

#### 使用类的成员函数

类的声明

```
class MyFunc {
public:
	//普通成员函数
	int m_func(int i = 10) {
		thread::id t_id = this_thread::get_id();		// 获得自己的线程ID
		int res = 0;
		for (int j = 1; j <= i; ++j) {
			res += j;
			this_thread::sleep_for(chrono::milliseconds(150));		//休眠150毫秒
		}
		cout << "子线程(" << t_id << ")的运行结果:" << res << endl;
		return res;
	}
	// 静态成员函数
	static int static_func(int i = 10) {
		thread::id t_id = this_thread::get_id();		// 获得自己的线程ID
		int res = 0;
		for (int j = 1; j <= i; ++j) {
			res += j;
			this_thread::sleep_for(chrono::milliseconds(150));		//休眠150毫秒
		}
		cout << "子线程(" << t_id << ")的运行结果:" << res << endl;
		return res;
	}
};
```

使用静态成员函数时，可以直接将函数加上作用域当普通函数使用

```
thread t(MyFunc::static_func, 6);
t.join();
```

使用非静态成员函数时有些特殊，调用时需要传入函数地址和对象地址

```
MyFunc mf;
thread t(&MyFunc::m_func,&mf, 6);
t.join();
```

当然，我们也可以使用`bind()`来绑定函数地址和对象地址：

> `bind()`可调用对象包装器被定义在库文件functional中

```
MyFunc mf;
auto m_func_bind = bind(&MyFunc::m_func,&mf);
thread t(m_func_bind, 6);
t.join();
```

**注意**：对象的生命周期必须要比线程对象的声明周期长，否则会发生内存泄漏。编译器不会捕捉这种错误，需要自行注意。

#### 使用匿名函数lambda表达式

```
thread t([=](int i = 5) {
	thread::id t_id = this_thread::get_id();		// 获得自己的线程ID
	int res = 0;
	for (int j = 1; j <= i; ++j) {
		res += j;
		this_thread::sleep_for(chrono::milliseconds(150));		//休眠150毫秒
	}
	cout << "子线程(" << t_id << ")的运行结果:" << res << endl;
	return res;
	}, 6);
t.join();
```

当然，也可以把使用以下形式：

```
auto lambda_func = [=](int i = 5) {
	thread::id t_id = this_thread::get_id();		// 获得自己的线程ID
	int res = 0;
	for (int j = 1; j <= i; ++j) {
		res += j;
		this_thread::sleep_for(chrono::milliseconds(150));		//休眠150毫秒
	}
	cout << "子线程(" << t_id << ")的运行结果:" << res << endl;
	return res;
};
thread t(lambda_func, 6);
t.join();
```

不过你都定义了一个`lambda_func`变量来保存lambda表达式了，为什么不直接写个普通函数呢？难道只是为了能在函数内定义函数吗？

#### 使用对类重载了`()`运算符的仿函数

定义类

```
class MyClass {
public:
	int operator()(int i = 10) {
		thread::id t_id = this_thread::get_id();		// 获得自己的线程ID
		int res = 0;
		for (int j = 1; j <= i; ++j) {
			res += j;
			this_thread::sleep_for(chrono::milliseconds(150));		//休眠150毫秒
		}
		cout << "子线程(" << t_id << ")的运行结果:" << res << endl;
		return res;
	}
};
```

使用

```
thread t(MyClass(), 6);
t.join();
```

#### 任务函数参数的传入

任务函数的形参可以是

- 普通变量(`int i`)：拷贝一个值传入

    ```
    void task(int i){}
    ```

- 指针(`int * i`)：使用指针传入，需要线程内修改参数变量的值时推荐使用这种方法。

    ```
    void task(int * i){}
    ```

- 常量引用(`const int& i`)：使用常量引用传入，**和线程外的引用不同，线程任务函数的引用会发生拷贝**。仅需传入值、不需修改变量时推荐使用这种方法

    ```
    void task(const int& i){}
    ```
    
- 普通引用。但是引用非常特殊，如果使用下面的函数做任务函数，会出现类型不匹配的错误：

    ```
    void task(int& i){}
    ```

    为什么常量引用是可行的，但引用不行？因为线程间虽然共享进程的资源，但是每个线程依然有自己独立的栈空间，一个线程访问其他线程的栈数据是不能被接受的。所以向线程传参时，会进行拷贝操作，假设要传入对象A，那么新线程就会在栈中复制一份相同的变量A，然后新线程访问自己栈空间里的对象A。然而对于引用来说，如果你使用了引用，意思就是你想在当前线程对其他线程的栈空间进行修改，但实际上你修改的东西是当前线程的栈空间，这是歧义的。

    你可能会疑问：为什么我任务函数已经写的是引用传参了，编译器还是要拷贝？其实在根据任务函数创建线程时，编译器是先进行拷贝，再把拷贝的东西放到任务函数里当参数，这时候才检查参数是否匹配。

    ```
    void task(int& i){}
    
    int main(){
    	int num = 100;
    	thread t(task, num);	// 错误！编译器实际上会拷贝一份num，，再将拷贝的num代入任务函数，而不是以原本num的引用去传递
    	t.join();
    	return 0;
    }
    ```
    
    但是我们可以使用`std::ref()`显式地对实参进行取引用后再传入引用，这样是可行的，这个声明表示你想传递一个引用，编译器看到之后不会进行拷贝，参数类型也就匹配上了。
    
    ```
    void task(int& i){}
    
    int main(){
    	int num = 100;
    	thread t(task, std::ref(num));
    	t.join();
    	return 0;
    }
    ```

**如果要在线程内修改其他线程的数据(即使用引用或指针)，最好使用`join()`来回收线程，如果使用`detach()`可能会发生内存非法访问的问题，还要注意线程之间的声明周期。**



## 线程的运行状态

在进程和线程的概念-多线程小节中，我们已经学习了并发和并行，也了解了并行这种高效的运作方式。那么线程是如何实现并行的，为了满足并发要求我们应该如何优化线程，本节将会讲解。

按照并发的思想和并行的方式，我们理想的线程运行方式就是各司其职、**互不干扰**地抢CPU时间片来运行（不是并行），这种最理想的运作方式就是线程异步。**线程默认是异步执行的。**当线程异步运行的时候，不会发生阻塞，执行流按照设定一直运行。

但是如果多个线程想访问共享资源呢？多个线程同时访问共享资源将会损害数据的安全(见下节)，所以人们需要让线程排好队按照顺序访问共享资源。这样一来线程就会被排队阻塞，这就是线程同步。可以说**如果有线程因为访问共享数据而导致执行流遭到了阻塞，就是线程同步**。

线程同步和异步是一种状态，线程只会在访问共享数据的时候是同步的，会礼尚往来排队运行，但其他时间还是异步的，该干啥就干啥。

线程异步能够保持最高效的并发运行状态。但有时候为了保持数据的安全性，人们不得不牺牲一些效率来进行线程同步。随着技术的发展，人们逐渐降低线程同步带来的效率损失，这样既保持了数据的安全性，又能满足高并发的需求。但线程同步的效率只能无限趋近于线程异步，只要是同步就会有效率的损失，不能做到和异步一样高效。



## 线程同步

为什么要进行线程同步？要回答这两个问题，我们需要先了解线程之间的竞争。

前文说到，所有线程共享进程的系统资源。如果一些资源同时被多个线程使用，那么我们称之为公共资源，也叫临界资源。最常见的就是内存啦。因为线程与线程之间是分开的，而且线程之间获得CPU时间片的概率也不一样且是动态变化的，所以有可能在同一时刻内，多个线程对临界资源同时进行了访问，这会导致某些问题。比如线程A和线程B同时对某块内存进行写操作，假设这块内存保存的是int类型，初始值为100，线程A对其加50，线程B对其加80。那么结果可能不会是230，而是150或180。

范例：

```
int func(int * p) {
	thread::id t_id = this_thread::get_id();		// 获得自己的线程ID
	for (int j = 1; j <= 100; ++j) {
		++(*p);
		cout << "子线程(" << t_id << ")的运行结果:" << *p << endl;
		this_thread::sleep_for(chrono::milliseconds(100));		//休眠100毫秒
	}
	return 0;
}

int main() {
	int num = 0;
	thread t1(func, &num), t2(func, &num);
	t1.join();
	t2.join();
	cout << "最终结果是" << num << endl;
	return 0;
}
```

在这个程序中，线程t1和t2同时对num进行操作，各操作100次，那么最终结果应该是200。然而因为线程没有进行同步导致一个线程的某些操作被另一个线程“覆盖”了，就会导致结果不是200：

![](http://md.ruimix.top/md/20221130171225.png)

而且两个线程都要使用cout输出，但因为cout对象只有一个，所以也会导致某些输出操作被覆盖，比如下图子线程(696)的运行结果就被吞掉了：

![](http://md.ruimix.top/md/20221130171407.png)



为什么会这样？这还得从CPU说起。在运行一个程序时，CPU需要和内存进行数据交互。但是CPU并不直接操作内存，而是讲内存中的值复制到L3缓存中(L3缓存在CPU中仅一个，是所有核心共享的)，再复制到L2、L1缓存中(L2、L1缓存是每个核心都有且独享的)，然后再对L1缓存中的数据进行操作，然后就是把操作之后的数据写入到L2缓存、L3缓存中，最后再写入内存。因为需要层层传值，所以内存中的数据从开始到最终被修改会有一段时间，即使这个时间人感觉不出来，但对于以纳秒为单位的计算机来说是很大的时间间隔了。如果两个核心同时对内存中的同一块内存进行修改，那么其中一个核心的操作可能会被“覆盖”掉。

> 为什么CPU要有三级缓存？因为CPU对数据的处理速度比内存的读写速度快的多！如果CPU直接对内存进行操作，那么CPU就有很大一部分时间被浪费在了等待内存回应上。为了让内存的速度不至于浪费CPU的算力，现代CPU都有缓存。缓存被集成在CPU内，其响应速度比内存快，当CPU需要读写内存中的信息时，就会先把数据请求读取到缓存中，这至少有两点可以提速：
>
> 1、因为内存中的数据都是成块批量读取到缓存中，缓存也是将数据成块打包写入内存中，相比随机读写内存中的数据这种方式更快；
>
> 2、如果CPU频繁地对某块内存进行读写，那么在内存被搬到缓存中后，CPU就可以频繁地对缓存而非内存进行操作，这避免了频繁访问内存而导致的延迟。

如何避免这个情况？很简单，让内存在某一时刻只能被一个线程访问不就行了呗。要让临界资源在某一时刻内只能被一个线程访问，那么换一个角度说，就要让线程依次访问临界资源，这就是线程同步。那么具体的操作有两个，一是使用锁，二是原子操作。这两者我们会在下文进行讲解。

线程同步有几种方法：

- 互斥锁：在mutex库中包含

- 读写锁：在C++14标准中添加的共享锁(`shared_lock`和`shared_timed_mutex`)、C++17中添加的`shared_mutex`就相当于读写锁

- 条件变量：在condition_variable库中包含

- 信号量：C++20之前没有提供信号量，但是可以利用操作系统的API，比如Linux的semaphore.h。C++20加入了信号量，在semaphore库中包含

- 原子变量：在atomic库中包含



### 互斥锁

互斥锁是一种用于管理临界区的锁。其本质上是一个互斥锁类型的对象或变量，和临界区是没有任何关系的。

- 如果互斥锁没有被占用，那么所有线程都可以占用这个互斥锁。
- 一旦有线程占用了这个互斥锁，这个线程就拥有了这个互斥锁的控制权。在同一时间内，只能存在一个线程占用互斥锁。
- 只有占有了互斥锁的线程可以解开互斥锁，互斥锁被解开后，回归到没有被占用的状态，等待其他线程占用。
- 如果线程尝试占用已被占用的互斥锁，那么这个线程会阻塞在互斥锁这里。
- 互斥锁需要被所有访问临界区的线程访问到，因此需要注意互斥锁的访问空间和生命周期。

打个比方，假设只有一个厕所，但是要为十个人服务，如果我们不对厕所进行加锁，那么就会出现一个人在上厕所时另一个人闯入的情况。如果我们加了一个单向锁，就不会出现这种情况了。而且这个单向锁只有厕所里面的人可以解开，外面的人如果尝试解开，就是在白费力气，只能在门外等待。

那么我们该如何使用锁呢？其实很简单，比如我们要锁一个房间里面的东西，我们会一一对房间内的桌子椅子和床等锁起来吗？很显然不会吧，我们只需要锁门就可以了。有人需要独享房间时，只需要在使用时将房间反锁，在不用时将锁开开离开就行了。我们的互斥锁也是如此，只需要**在开始访问临界区的位置对互斥锁进行加锁，在结束访问临界区时解锁互斥锁即可**。严格意义上来说，临界区和互斥锁是一一匹配的，就像一个房间配备一把锁那样。

互斥锁被定义在mutex头文件内，有四个类型：普通互斥锁`mutex`、带等待时间的互斥锁`timed_mutex`、递归互斥锁`recursive_mutex`、带等待时间的递归互斥锁`recursive_timed_mutex`。

对于任意的互斥锁类，我们不需要关心它的构造函数是怎样的（因为它只有无参构造），也不需要关心它的析构。所有的互斥锁类型都无法拷贝构造，因为不存在两个一模一样的互斥锁；也无法被移动。

##### 普通互斥锁`mutex`

- `void lock()`

    对互斥锁对象进行锁定操作，已经占用了普通互斥锁的线程不可以重复进行加锁操作，否则会死锁

- `void unlock()`

    对互斥锁对象进行解锁操作，已经占用了普通互斥锁的线程不可以重复进行解锁操作，否则会死锁

- `bool try_lock()`

    尝试对互斥锁对象进行锁定，如果该互斥锁是未锁定状态，则会对该互斥锁进行锁定操作，并且返回true；如果该互斥锁已被其他线程锁定，则返回false。

    如果线程已经占用了这个互斥锁对象，那么函数的行为未定义

- `native_handle_type native_handle()`

    返回底层的句柄

##### 带等待时间的互斥锁`timed_mutex`

带等待时间的互斥锁`timed_mutex`拥有普通互斥锁`mutex`的所有成员函数，这些成员函数的功能是一样的，同时还多了两个成员函数：

- `bool try_lock_for(const TIME)`

- `bool try_lock_until(const TIME)`

    这两个函数的作用是尝试对互斥锁进行加锁操作，如果该互斥锁是未锁定状态，则会对该互斥锁进行锁定操作，并且返回true；如果该互斥锁已被其他线程锁定，那么会阻塞（等待一段时间/等到某个时刻），如果这段阻塞时间内成功对互斥锁进行加锁操作，则返回true，如果到阻塞结束都未进行加锁操作，则返回false

    如果线程已经占用了这个互斥锁对象，那么函数的行为未定义

##### 递归互斥锁`recursive_mutex`

递归互斥锁`recursive_mutex`拥有普通互斥锁`mutex`的所有成员函数，这些成员函数的功能是一样的。与普通互斥锁`mutex`不同的是，已经占用了普通互斥锁的线程不可以重复进行加锁和解锁操作，否则会死锁；而递归互斥锁`recursive_mutex`可以被同一个线程上锁和解锁多次。

当某个线程占用递归互斥锁时，每上锁一次/解锁一次都会被递归互斥锁记录，**只有在解锁次数等于上锁次数时**，递归互斥锁才能被释放

递归互斥锁的使用次数并不是无限多的，其最大使用次数也未被指定。当抵达最大上锁次数时，`lock()`会抛出`std::system_error`而`try_lock()`会返回false

##### 带等待时间的递归互斥锁`recursive_timed_mutex`

带等待时间的递归互斥锁`recursive_timed_mutex`就是递归互斥锁`recursive_mutex`加上带等待时间的互斥锁`timed_mutex`的功能。拥有带等待时间的互斥锁`timed_mutex`的所有成员函数，这些成员函数的功能是一样的；拥有递归互斥锁`recursive_mutex`对已经占用的线程的全部特性。



##### 实例

下面这段程序代码创建了10个线程，并且每个线程都对主线程中的`num`变量进行加1操作，加10次。

```
#include <iostream>
#include <thread>
#include <chrono>
#include <mutex>
#include <vector>
using namespace std;

int func(int * p) {
	thread::id t_id = this_thread::get_id();		// 获得自己的线程ID
	for (int j = 0; j < 10; ++j) {
		++(*p);
		cout << "子线程(" << t_id << ")对参数加一，结果:" << *p << endl;
		this_thread::sleep_for(chrono::milliseconds(100));		//休眠100毫秒
	}
	return 0;
}

int main() {
	int num = 0;
	thread T[10];
	for (int i = 0; i < 10; ++i) {
		T[i] = thread(func, &num);
	}
	for (int i = 0; i < 10; ++i) {
		T[i].join();
	}
	return 0;
}
```

运行结果如下：

![](http://md.ruimix.top/md/20221210162318.png)

可以看到未加锁的情况下，各线程访问临界区是杂乱无章的。

加锁后：

```
#include <iostream>
#include <thread>
#include <chrono>
#include <mutex>
#include <vector>
using namespace std;

mutex m;				// 创建互斥锁对象

int func(int * p) {
	thread::id t_id = this_thread::get_id();		// 获得自己的线程ID
	for (int j = 0; j < 10; ++j) {
		m.lock();				// 加锁
		++(*p);
		cout << "子线程(" << t_id << ")对参数加一，结果:" << *p << endl;
		m.unlock();				// 解锁
		this_thread::sleep_for(chrono::milliseconds(100));		//休眠100毫秒
	}
	return 0;
}

int main() {
	int num = 0;
	thread T[10];
	for (int i = 0; i < 10; ++i) {
		T[i] = thread(func, &num);
	}
	for (int i = 0; i < 10; ++i) {
		T[i].join();
	}
	return 0;
}
```

运行结果如下：

![](http://md.ruimix.top/md/20221210162608.png)



### 死锁

死锁就是在使用互斥锁时，一组线程都被阻塞在了锁前、且在等待一个永远也不会为真的条件，导致没有线程能够离开阻塞区。

常见的死锁如：

1. 某个线程对普通互斥锁重复加锁，因为在第一次加锁后，锁就被锁定了，再次进行加锁会导致该线程阻塞；而因为该线程阻塞了，也就不会释放互斥锁，导致其他要使用该互斥锁的线程也跟着一起阻塞。
2. 线程A占用了临界区1，并对互斥锁①进行了加锁；线程B占用了临界区2，并对互斥锁②进行了加锁。此时如果线程A访问临界区2尝试对互斥锁②进行加锁，线程B也访问临界区1尝试对互斥锁①进行加锁，两个线程就会僵持不下，导致死锁。
3. 线程在加锁后忘记解锁，导致其他线程也请求不到临界区，被阻塞
4. 某个线程本身就存在阻塞情况，比如在主线程中创建了一个子线程，同时主线程和子线程都要访问临界资源，主线程调用了`join()`来回收子线程，这样就可能发生死锁：如果主线程比子线程获得锁，且访问完了临界资源，最终等待回收子线程，而子线程因为没有获得锁一直都在阻塞，导致主线程和子线程都被阻塞
5. ...

死锁是难以预测和调试的，因为线程使用CPU的执行轨迹是千变万化的，但是死锁可以完全避免，这和程序员的编程能力和对细节的处理能力挂钩。在使用互斥锁时，我们一定要多检查，尽量使用`try_lock()`和`std::lock()`

`void std::lock()`是一个函数，它接收多于1个的可变参数的锁对象，并且对他们进行锁定，它能同时锁定这些锁而且不会发生死锁：遍历并锁定参数锁，当所有锁都能被锁定时，成功锁定所有锁，一旦其中有一个锁没法锁定或抛出异常，它会回滚并解开之前它锁上的锁，阻塞直到所有锁都被上锁成功。这能避免一些因为上锁时机导致的死锁。

`int std::try_lock()`是一个函数，它接收多于1个的可变参数的锁对象，并且尝试对他们进行锁定，它能同时锁定这些锁而且不会发生死锁：遍历并锁定参数锁，当所有锁都能被锁定时，成功锁定所有锁，一旦其中有一个锁没法锁定或抛出异常，它会回滚并解开之前它锁上的锁，然后返回第一个异常锁的从0开始的下标，不会阻塞。

示例：

```
#include <iostream>
#include <thread>
#include <mutex>
using namespace std;

int num1 = 0;
int num2 = 0;
mutex m1;		//m1用于锁num1
mutex m2;		//m2用于锁num2
void func1(){
	  lock(m1, m2);		// 因为不确定m1和m2哪个需要先被锁
	  for (int i = 0; i < 10000000; i++){
	      ++num1;
	      ++num2;
	  }
	  // 需要手动解锁
	  m1.unlock();
	  m2.unlock();
}

int main(){
	thread t1(func);
	thread t2(func);
	t1.join();
	t2.join();
	return 0;
}
```



### 基于RAII机制的封装锁类

RAII机制就是”资源获取即初始化“，简单来说就是到手即用，不用即销毁。是C++中一种用于避免资源浪费、防止内存泄漏的机制。因为很多程序员没有良好的编程规范，就导致这些程序员对资源只管拿来用、不管销毁，比如某程序员在堆区申请了一块内存，却直到程序结束也没给它释放掉，这就造成了资源浪费；或者我们上文提过的对互斥锁加锁了之后忘记解锁就导致了死锁。

RAII机制的引入给程序员带来很大的便利性，比如C++11中加入的智能指针，申请一块内存我给你记着，什么时候离开这块内存的作用域了，我自动帮你释放，不用你操心。

> 永远不要过于自信说你可以记得所有资源的回收！当业务复杂且控制流分支多时，即使老手也会决定棘手，你不确定你可能会漏掉了哪个控制流出口！

C++中互斥锁也引入了这个概念，比如封装锁类型`lock_guard`，被定义在mutex头文件中

- **`lock_guard`**

    `lock_guard`类型是一个泛型的封装锁。它没有多余的成员函数，且无法拷贝和移动。它接收四种互斥锁类型，而且需要使用一个互斥锁对象进行构造。就像这样：

    ```
    mutex my_mutex;
    lock_guard<mutex> auto_locker(my_mutex);
    ```

    它会在构造时锁定互斥锁，如果此时互斥锁已被占用，则会在此处阻塞直到加锁成功；在离开作用域析构时解开互斥锁。

    我们以上个实例为例，看看使用封装锁的效果：
    
    ```
    #include <iostream>
    #include <thread>
    #include <chrono>
    #include <mutex>
    #include <vector>
    using namespace std;
    
    mutex m;				// 创建互斥锁对象
    
    int func(int * p) {
    	thread::id t_id = this_thread::get_id();		// 获得自己的线程ID
    	for (int j = 0; j < 10; ++j) {
    		lock_guard<mutex> auto_locker(m);		// 创建封装锁
    		++(*p);
    		cout << "子线程(" << t_id << ")对参数加一，结果:" << *p << endl;
    		this_thread::sleep_for(chrono::milliseconds(100));		//休眠100毫秒
    		// 每次for循环一次，封装锁都会被重新定义，并在当次循环结束后释放
    	}
    	return 0;
    }
    
    int main() {
    	int num = 0;
    	thread T[10];
    	for (int i = 0; i < 10; ++i) {
    		T[i] = thread(func, &num);
    	}
    	for (int i = 0; i < 10; ++i) {
    		T[i].join();
    	}
    	return 0;
    }
    ```

    结果如下：
    
    ![](http://md.ruimix.top/md/20221210165417.png)
    
    
    
- **`unique_lock`**

    `unique_lock`是`lock_guard`的升级版，与`lock_guard`不同的是，`lock_guard`不能控制上锁和释放的时机，而`unique_lock`支持。`unique_lock`支持移动，但不支持拷贝。

    `unique_lock`虽然比`lock_guard`灵活很多，但在效率上要差一点，内存占用也多一点。
    
    `unique_lock`的构造不仅可以传入一个互斥锁，还可以传入锁定互斥锁的时机。有三种时机：`std::defer_lock`、`std::try_to_lock`、`std::adopt_lock`，他们都被定义在mutex头文件中
    
    ```
    mutex m;
    ① unique_lock<mutex> ul(m);		// 立即锁定互斥锁，如果不能锁定则阻塞
    ② unique_lock<mutex> ul(m,std::defer_lock);		// 不在构造时锁定互斥锁，稍后手动指定，互斥锁必须以无锁的状态传入
    ③ unique_lock<mutex> ul(m,std::try_to_lock);	// 尝试锁定互斥锁，如果能锁定则锁定，如果不能锁定也不会阻塞
    ④ unique_lock<mutex> ul(m,std::adopt_lock);		// 不在构造时锁定互斥锁，而是默认此线程已经占用了互斥锁，但在析构时解锁
    ```
    
    `unique_lock`还有很多成员函数供我们使用，这些函数和我们上文讲过的互斥锁的成员函数很类似
    
    - `void lock()`
    
    - `void unlock()`
    
    - `bool try_lock()`
    
    - 【当互斥锁带等待时间时才有】`bool try_lock_for(const TIME)`
    
    - 【当互斥锁带等待时间时才有】`bool try_lock_until(const TIME)`
    
    - `void swap(unique_lock& other)`
    
        与另一个`unique_lock`对象交换
    
    - `mutex_type* release()`
    
        请注意，这个函数不会释放互斥锁。
    
        与互斥锁切断关系，不再负责互斥锁的释放，之后此对象的状态为无关联互斥锁，如果对其再调用`lock()`、`mutex()`等成员函数都是无意义的，甚至可能导致死锁和未定义行为。
    
    - `mutex_type* mutex()`
    
        返回指向互斥锁的指针，如果无关联互斥锁则返回空
    
    - `bool owns_lock()`
    
        检查互斥锁是否已经被本线程锁定，如果是返回true，否返回false

    `unique_lock`在手动调用了`lock()`和`unlock()`之后，在超出其作用域时并不会发生死锁。
    
    ```
    mutex mtx;
    int func() {
    	unique_lock<mutex> auto_lock(mtx);
    	auto_lock.unlock();
      	return 0;	//不会导致死锁
    }
    ```
    
    但是对`mutex`来说，重复释放锁会导致死锁：
    
    ```
    mutex mtx;
    int func() {
    	mtx.lock();
    	mtx.unlock();
    	mtx.unlock();	//死锁
      	return 0;
    }
    ```
    
    这是因为`unique_lock`的内部存在一个bool类型的变量`_is_Own`存储互斥锁的数学。在调用``lock()`和`unlock()`函数时，该值会变化，只有当其为true时，锁才会被释放，且释放后改成false。这样就可以避免多次调用`unlock()`函数而导致的死锁问题。
    
    
    
    这是一种`unique_lock`的用法，与`lock_guard`不同，这不会对线程进行阻塞：
    
    ```
    mutex m;
    
    void func(){
    	unique_lock<mutex> ul(m,std::try_to_lock);	// 尝试锁定互斥锁，如果能锁定则锁定，如果不能锁定也不会阻塞
    	if(ul.owns_lock()){
    		// 如果已经占用锁
    	}
    	else{
    		// 如果没有占用锁
    	}
    }
    ```
    
    上述两个RAII机制的封装锁类在使用时应该避免绕过封装锁直接对互斥锁进行操作，这样可能导致死锁。比如
    
    ```
    mutex m;
    unique_lock<mutex> auto_lock(m);	// m已经上锁
    m.lock();	// 死锁
    ```
    

**请注意，`unique_lock`并不是线程安全的，这意味着你在多个线程中使用一个`unique_lock`会导致未定义行为**



- **`shared_lock`**

    这是一种带共享属性的封装锁类，与`unique_lock`非常相似，我们将在读写锁章节进行学习。



### call_once()和once_flag

在某些情境下，我们只能让一些函数仅调用一次，比如初始化资源。而我们恰好又想把这个函数放到线程中去运行，该怎么办呢？如果我们不管，那么每个线程都会调用一次函数，这显然不是我们想要的。如果我们对这个函数进行加锁，定义一个变量，如果变量为0就说明未执行过该函数，如果变量非0就说明已经执行过该函数，然后对这个变量进行加锁...这样是可行的，而且C++11已经帮我们做好了这个功能：`call_once()`和`once_flag`。

`call_once()`和`once_flag`被定义在mutex头文件中，其中`once_flag`本质上是一个保存着布尔值的互斥锁类，用于辅助`call_once()`。`call_once()`是一个函数模板，我们需要传入一个`once_flag`类型的对象、一个函数以及函数的可变参数列表。

> 请注意`once_flag`类型对象的生命周期和访问作用域

如下示例：

如果没有使用`call_once()`和`once_flag`，目标函数被多次调用：

![](http://md.ruimix.top/md/20221210204108.png)

使用了`call_once()`和`once_flag`之后，目标函数只被调用了一次：

![](http://md.ruimix.top/md/20221210203936.png)



### 读写锁

互斥锁通过串行的方式很好地保护了临界区，然而这容易导致效率降低。在某些场景下，串行是唯一的方式，比如排队等公共厕所；但我们有些场景如果串行，效率就太低了。比如看电影，以前电影院没有现在这么高级，在放映电影时，每一个观众都能随时自由地进出放映厅，但在管理员更换电影或调试的时候才禁止观众进入。很显然放映厅是临界区吧，但观众可以并行地观看电影，但管理员只能依次操作电影。在计算机中也一样，某些操作我们无需修改临界区的值，只需读取，这种操作是肯定不会导致数据混乱的，而我们只需要对写入或修改的操作进行加锁串行即可。这就是读写锁。

可能你会疑问：不能只对写操作加锁吗？答案是不行，因为在进行写操作时，其他写和读操作都要被阻塞，只有在读操作时，其他读操作才不会被阻塞，因此读操作需要分情况阻塞，不能只对其中一种操作进行加锁。

下面这个示例演示了不加锁的情况下读写的耗时：

```
#include <iostream>
#include <thread>
#include <chrono>
#include <mutex>
#include <vector>
using namespace std;

mutex m;

int read_func(int* p) {			// 读取num的值
	for (int i = 0; i < 15; ++i) {
		m.lock();
		cout << "读取到num的值为" << *p << endl;
		this_thread::sleep_for(chrono::milliseconds(50));		// 假设每次读取要耗时50毫秒
		m.unlock();
	}
	return 0;
}

int write_func(int* p) {		// 修改num的值
	for (int i = 0; i < 5; ++i) {
		m.lock();
		++(*p);
		cout << "修改num的值为" << *p << endl;
		m.unlock();
		this_thread::sleep_for(chrono::milliseconds(200));		//每200毫秒要写一次数据，写5次
	}
	return 0;
}

int main() {
	int num = 666;
	thread T[3];
	auto startTime = std::chrono::system_clock::now();
	for (int i = 0; i < 2; ++i) {
		T[i] = thread(read_func, &num);
	}
	T[2] = thread(write_func, &num);
	for (int i = 0; i < 3; ++i) {
		T[i].join();
	}
	std::chrono::duration<double> runTime = std::chrono::system_clock::now() - startTime;
	cout << "所耗时间为：" << runTime.count() << "s" << endl;
	return 0;
}
```

运行结果：耗时2.37秒

![](http://md.ruimix.top/md/20221211111156.png)



C++14为我们提供了共享锁类`shared_timed_mutex`和封装锁类`shared_lock`，C++17又加入了`shared_mutex`，可以实现读写锁。这三者都被定义在头文件shared_mutex中。

共享锁有两个权：共享权和自由权，当有线程拥有锁的私有权时，其他线程不能获得锁的任何权，但是当锁的共享权被占有时，其他线程可以同时占有共享权，但不能在此时占有私有权。这一特点和读写锁的特性非常吻合。

#### 【C++14】带等待时间的共享锁类`shared_timed_mutex`

成员函数如下，其中大部分同名的成员函数的功能和互斥锁相似，值得我们注意的是带shared的版本

- `void lock()`

    给共享锁类对象上锁并占用私有权，本线程完全占有此共享锁，不允许其他线程操作

    当线程已经占用此共享锁的共享权，应当调用`unlock_shared()`交出共享权，再占用私有权，不允许直接由共享权变为私有权

    当其他线程占用此共享锁的私有权和共享权时，本线程会被阻塞

- `void unlock()`

    给共享锁类解锁私有权

- `bool try_lock()`

- `bool try_lock_for(const TIME)`

- `bool try_lock_until(const TIME)`

- `void lock_shared()`

    给共享锁类对象上锁并占用共享权，和其他线程占用此共享锁的共享权并不冲突

    当其他线程占用此共享锁的私有权时，本线程会被阻塞

- `void unlock_shared()`

    给共享锁类解锁共享权

- `bool try_lock_shared()`

- `bool try_lock_shared_for(const TIME)`

- `bool try_lock_shared_until(const TIME)`

#### 【C++14】封装锁类`shared_lock`

`shared_lock`和`unique_lock`非常相似，但功能不同。`shared_lock`只能传入共享锁类作为构造参数，且只能对共享锁的共享权进行操作。以下是它的成员函数

> 请注意和`lock_shared()`进行区分，这是共享锁类的成员函数！

- `void lock()`

    占有共享锁的共享权，如果锁被私有地占用，那么此函数会阻塞直到占用共享权成功

- `void unlock()`

- `bool try_lock()`

- 【当互斥锁带等待时间时才有】`bool try_lock_for(const TIME)`

- 【当互斥锁带等待时间时才有】`bool try_lock_until(const TIME)`

- `void swap(unique_lock& other)`

    与另一个`unique_lock`对象交换

- `mutex_type* release()`

    与互斥锁切断关系，不再负责互斥锁的释放，之后此对象的状态为无关联互斥锁

- `mutex_type* mutex()`

    返回指向互斥锁的指针，如果无关联互斥锁则返回空

- `bool owns_lock()`

    检查互斥锁是否已经被本线程锁定，如果是返回true，否返回false

#### 【C++17】共享锁类`shared_mutex`

共享锁类`shared_mutex`和带等待时间的共享锁类`shared_timed_mutex`功能一样，只不过少了`bool try_lock_for(const TIME)`和`bool try_lock_until(const TIME)`这些函数。



#### 实例

我们以上文中检测读写的例子为原型，在此基础上使用共享锁进行改造，实现读写锁并优化：

1. 使用`shared_timed_mutex`

    ```
    #include <iostream>
    #include <thread>
    #include <chrono>
    #include <mutex>
    #include <shared_mutex>
    #include <vector>
    using namespace std;
    
    shared_timed_mutex stm;
    
    int read_func(int* p) {			// 读取num的值
    	for (int i = 0; i < 15; ++i) {
    		stm.lock_shared();				// 加读锁
    		cout << "读取到num的值为" << *p << endl;
    		this_thread::sleep_for(chrono::milliseconds(50));		// 假设每次读取要耗时50毫秒
    		stm.unlock_shared();			// 解读锁
    	}
    	return 0;
    }
    
    int write_func(int* p) {		// 修改num的值
    	for (int i = 0; i < 5; ++i) {
    		stm.lock();				// 加写锁
    		++(*p);
    		cout << "修改num的值为" << *p << endl;
    		stm.unlock();				// 解写锁
    		this_thread::sleep_for(chrono::milliseconds(200));		//每200毫秒要写一次数据，写5次
    	}
    	return 0;
    }
    
    int main() {
    	int num = 666;
    	thread T[3];
    	auto startTime = std::chrono::system_clock::now();
    	for (int i = 0; i < 2; ++i) {
    		T[i] = thread(read_func, &num);
    	}
    	T[2] = thread(write_func, &num);
    	for (int i = 0; i < 3; ++i) {
    		T[i].join();
    	}
    	std::chrono::duration<double> runTime = std::chrono::system_clock::now() - startTime;
    	cout << "所耗时间为：" << runTime.count() << "s" << endl;
    	return 0;
    }
    ```

    结果：耗时1.2秒

    ![](http://md.ruimix.top/md/20221211141331.png)

2. 使用`shared_lock`

    ```
    #include <iostream>
    #include <thread>
    #include <chrono>
    #include <mutex>
    #include <shared_mutex>
    #include <vector>
    using namespace std;
    
    shared_timed_mutex stm;
    
    int read_func(int* p) {			// 读取num的值
    	for (int i = 0; i < 15; ++i) {
    		shared_lock<shared_timed_mutex> write_locker(stm);				// 加读锁
    		cout << "读取到num的值为" << *p << endl;
    		this_thread::sleep_for(chrono::milliseconds(50));		// 假设每次读取要耗时50毫秒
    	}
    	return 0;
    }
    
    int write_func(int* p) {		// 修改num的值
    	for (int i = 0; i < 5; ++i) {
    		unique_lock<shared_timed_mutex> write_locker(stm);				// 加写锁
    		++(*p);
    		cout << "修改num的值为" << *p << endl;
    		write_locker.unlock();									//写完后立即释放锁
    		this_thread::sleep_for(chrono::milliseconds(200));		//每200毫秒要写一次数据，写5次
    	}
    	return 0;
    }
    
    int main() {
    	int num = 666;
    	thread T[3];
    	auto startTime = std::chrono::system_clock::now();
    	for (int i = 0; i < 2; ++i) {
    		T[i] = thread(read_func, &num);
    	}
    	T[2] = thread(write_func, &num);
    	for (int i = 0; i < 3; ++i) {
    		T[i].join();
    	}
    	std::chrono::duration<double> runTime = std::chrono::system_clock::now() - startTime;
    	cout << "所耗时间为：" << runTime.count() << "s" << endl;
    	return 0;
    }
    ```

    结果：耗时1.2秒

    ![](http://md.ruimix.top/md/20221211141722.png)



### 条件变量

C++11为我们提供了条件变量，这是除锁之外的另一种线程同步方式。严格意义上说，它不保护临界区，而是通过控制线程来保证数据不会被非法操作，当条件满足时，相关线程会被阻塞而另一方的线程将会被唤醒运行。因为条件变量不管临界区，临界区仍然需要我们使用锁来保护。

条件变量最经典的应用场景就是生产者-消费者模型。

#### 生产者-消费者模型

我们假设这样一个情景：

> 某工厂里面有8条流水线，其中3条将原材料加工成半成品，5条将半成品加工成成品。同时，该工厂有一个半成品存放仓库。按照工厂的管理规定，同一时间内只能有一条流水线的工人进入仓库。而且每条流水线都是独立运作的，一条流水线的状态不会影响其他流水线。
>
> - 当仓库满时，加工原材料为半成品的流水线暂停；
> - 当仓库空时，加工半成品为成品的流水线暂停；
>
> 现在你是管理员，你该如何设计算法调度所有流水线工作和进出仓库？

这就是一个经典的生产者-消费者模型。我们假设加工原材料为半成品的流水线为生产者线程，加工半成品为成品的流水线为消费者线程，仓库为任务队列。那么我们可以写出以下代码：

```
#define WarehouseSize 10

class Factory {
private:
	int goods[WarehouseSize];		// 仓库
	int goods_num = 0;				// 货物数量
	mutex lock;						// 仓库锁
	thread p[3];					// 生产者流水线
	thread c[5];					// 消费者流水线
public:
	Factory();						// 构造，在构造中创建线程
	~Factory();						// 析构，在析构中回收线程
	void producer();				// 生产者
	void consumer();				// 消费者者
};

Factory::Factory() {
	for (int i = 0; i < 3; ++i) {
		p[i] = thread(&Factory::producer,this);
	}
	for (int i = 0; i < 5; ++i) {
		c[i] = thread(&Factory::consumer, this);
	}
}

Factory::~Factory() {
	for (int i = 0; i < 3; ++i) {
		p[i].join();
	}
	for (int i = 0; i < 5; ++i) {
		c[i].join();
	}
}

void Factory::producer() {
	while (true) {
		this_thread::sleep_for(chrono::milliseconds(500));		// 生产一个半成品需要500单位时间
		unique_lock<mutex> ulock(lock);							// 占用仓库存入半成品
		goods[goods_num] = rand() % 100;
		++goods_num;
		cout << "生产了一个半成品：" << goods[goods_num - 1] << "，当前货物数量：" << goods_num << endl;
	}
	return;
}

void Factory::consumer() {
	while (true) {
		unique_lock<mutex> ulock(lock);							// 占用仓库取出半成品
		--goods_num;
		cout << "消费了一个半成品：" << goods[goods_num + 1] << "，当前货物数量：" << goods_num << endl;
		ulock.unlock();											// 取出后手动释放互斥锁
		this_thread::sleep_for(chrono::milliseconds(830));		// 消耗一个半成品需要830单位时间
	}
	return;
}
```

如果我们不进行限制的话，会出现货物存量为负值或者货物存量超过仓库容量这种结果：

![image-20221211213410275](http://md.ruimix.top/md/image-20221211213410275.png)



我们无法通过简单地使用锁来实现这个模型，因为锁不会唤醒阻塞的线程(线程阻塞是指放弃抢夺CPU时间片，如果线程里有个循环一直检测，不算阻塞)。但是条件变量可以轻松实现。

#### 条件变量类

C++11为我们提供了两个条件变量类：`condition_variable`和`condition_variable_any`，它们被定义在头文件`condition_variable`中。我们不必关心它的构造和析构，也不能对其进行拷贝和移动。

- **`condition_variable`**

    - `void notify_one()`

        唤醒一个被条件变量阻塞的线程，请注意，唤醒的线程是完全随机的，只要被该条件变量阻塞，就有可能会被唤醒。所以，它并不知道什么是生产者和消费者，如果生产者和消费者共用同一个条件变量，当一个生产者生产了产品时，它可能会唤醒另一个阻塞的生产者，而没有消费者被唤醒，如果此时队列处于满的状态，却没有消费者去进行消费，就会导致死锁。

        避免这个情况的做法是使用`notify_all()`或创建两个条件变量[推荐]：一个用于生产者，另一个用于消费者。这样，生产者和消费者就可以分别唤醒对方，而不是唤醒同类。

    - `void notify_all()`

        唤醒所有被条件变量阻塞的线程

    - `void wait(unique_lock<mutex>&, [可选]Predicate)`

        这个函数接受一个`unique_lock`的封装锁类对象，和一个可选的、值类型为布尔类型的条件函数。

        当没有传入条件函数或传入的条件函数的返回值为false时，此函数**进入睡眠状态**(也叫被该条件变量阻塞)，并解锁`unique_lock`的互斥锁，直到被唤醒，它才脱离睡眠状态，**重新**判断传入的条件函数的返回值(如果有)，锁定互斥锁对象并向下运行。（如果它没有成功占用互斥锁，那么它会进入阻塞；或者如果条件函数的返回值为false，那么它会重新睡眠）

        当传入的条件函数的返回值为true时，此函数不会睡眠并直接向下运行

        > 睡眠状态和阻塞的区别：睡眠状态是暂停运行，直到被唤醒才开始运行；而阻塞的线程实际上一直在运行，只是因为各种原因无法向下运行而已。这就好比堵车时，睡眠状态是司机睡觉，就算前面道路畅通也不会察觉，直到交警来把司机叫醒；而阻塞是司机一直醒着，但由于堵住了导致车辆无法前进，只要路况好起来车辆就会跟进。

        

    - `bool wait_for(unique_lock<mutex>&, const chrono::duration<Rep,Period>&, [可选]Predicate)`

        `bool wait_until(unique_lock<mutex>&, const chrono::duration<Rep,Period>&, [可选]Predicate)`

        和上面的`wait()`功能一样，不过带了超时功能，如果超出时间限制还在阻塞返回false，否则返回true

- **`condition_variable_any`**

    和`condition_variable`功能一样，不过它可以接受四种互斥锁类型，而`condition_variable`只接收`mutex`类型的`unique_lock`。



#### 实例

我们以上文中生产者-消费者模型的例子为原型，在此基础上使用条件变量进行改造优化：

```
#include <iostream>
#include <thread>
#include <chrono>
#include <mutex>
#include <vector>
#include <condition_variable>
using namespace std;
#define WarehouseSize 10

class Factory {
private:
	int goods[WarehouseSize];		// 仓库
	int goods_num = 0;				// 货物数量
	mutex lock;						// 仓库锁
	thread p[3];					// 生产者流水线
	thread c[5];					// 消费者流水线
	condition_variable cv;			// 创建条件变量[此处只创建了一个是有缺陷的]
public:
	Factory();						// 构造，在构造中创建线程
	~Factory();						// 析构，在析构中回收线程
	void producer();				// 生产者
	void consumer();				// 消费者者
};

int main() {
	Factory f;
	return 0;
}

Factory::Factory() {
	for (int i = 0; i < 3; ++i) {
		p[i] = thread(&Factory::producer, this);
	}
	for (int i = 0; i < 5; ++i) {
		c[i] = thread(&Factory::consumer, this);
	}
}

Factory::~Factory() {
	for (int i = 0; i < 3; ++i) {
		p[i].join();
	}
	for (int i = 0; i < 5; ++i) {
		c[i].join();
	}
}

void Factory::producer() {
	while (true) {
		this_thread::sleep_for(chrono::milliseconds(500));		// 生产一个半成品需要500单位时间
		unique_lock<mutex> ul(lock);
		cv.wait(ul, [this]() {return goods_num < WarehouseSize; });		// 当库存没超过10时，生产者可以入库
		goods[goods_num] = rand() % 100;
		cout << "生产了一个半成品：" << goods[goods_num] << "，当前货物数量：" << goods_num + 1 << endl;
		++goods_num;
		ul.unlock();
		cv.notify_one();										// 生产了产品，唤醒消费者函数线程
	}
	return;
}

void Factory::consumer() {
	while (true) {
		unique_lock<mutex> ul(lock);
		cv.wait(ul, [this]() {return goods_num > 0; });			// 当库存大于0时，不阻塞消费者函数出库
		cout << "消费了一个半成品：" << goods[goods_num - 1] << "，当前货物数量：" << goods_num - 1 << endl;
		--goods_num;
		ul.unlock();											// 取出后手动释放互斥锁
		cv.notify_one();										// 消费了产品，唤醒生产者函数线程
		this_thread::sleep_for(chrono::milliseconds(830));		// 消耗一个半成品需要830单位时间
	}
	return;
}
```

运行结果：![image-20221211232140105](http://md.ruimix.top/md/image-20221211232140105.png)



```
condition_variable.wait(lock, 条件);
```

和下面这段语句功能是一样的 

```
while (!条件) {
	condition_variable.wait(lock)
}
```

都是在判断：如果条件成立，那么不阻塞继续往下运行；如果条件不成立就阻塞。

值得注意的是，下面必须使用`while`循环检测，而不能用`if`检测一次，因为`if`在阻塞被唤醒后不会再对条件进行检测，而是直接往下跑了，可能会有很多线程抢资源，如果资源比较少，那有些线程就会处于“条件成立了，实际上却没有抢到资源”的窘境。如果使用while循环检测的话，再被唤醒后还会再检测一下条件，看看能不能满足再运行。`wait()`函数在传入条件函数时，被唤醒时也是会重新检测条件函数的，所以行得通。



### 信号量

C++20加入了信号量库semaphore。信号量也是一种线程同步的手段，和条件变量非常相似，条件变量通过判断条件来阻塞或唤醒相关线程，来实现线程间协调工作；而信号量则是通过一个变量来控制相关线程。

和条件变量一样，信号量也不会管理临界区，管理临界区需要和互斥锁搭配使用。

*由于现在我用不了C++20的东西(VisualStudio2019无法更新/Linux下更新了GCC但是没有semaphore头文件)，因此无法示范*

**信号量类`counting_semaphore`**

信号量类内部有一个计数器，并提供了加减计数器的函数。计数器有取值范围，为0~用户构造时的最大值。当计数器值为0时，阻塞减小计数器的线程。

信号量类需要使用参数构造，无法被拷贝。

- 构造函数

    `counting_semaphore<信号量最大值> 信号量对象名称{信号量初始值}`

    构造一个信号量对象

- `constexpr std::ptrdiff_t max()`

    返回信号量内部计数器可达的最大值

- `void release(std::ptrdiff_t update = 1 )`

    对信号量内的计数器进行加操作，默认参数是1，不可传入负值，也不可传入超过(最大值-当前值)的值

- `void acquire()`

    对信号量内的计数器进行减操作。只能减一，如果计数器大于0将不会阻塞，否则将阻塞直到计数器大于0可以实现减一的时候

- `bool try_acquire_for(rel_time)`

    `bool try_acquire_until(abs_time)`

    和`void acquire()`一样，不过加了等待时间和返回值



### 原子变量

原子变量具有原子性，是C++11中添加的为了保证线程在访问临界区时不会发生争抢的一种功能。和线程同步的互斥锁非常类似，原子变量在同一时刻只能被一个线程操作，你不可能观察到原子操作进行到一半的状态。

> **原子操作和原子性**
>
> 原子是自然界中不可再分的粒子。那么原子操作也就是不可再分的操作。这里的分指的是CPU的上下文切换，也就是线程之间的切换。
>
> 打个比方，把垃圾丢进垃圾桶里需要三步：①打开垃圾桶 ②把垃圾放进去 ③关上垃圾桶。假如不止你一个人想丢垃圾，就可能发生争抢：你打开了垃圾桶，张三顺势丢了垃圾，轮到你时垃圾桶恰好满了。你肯定不希望这样，这时候使用互斥锁可以解决这个问题，每个人在丢垃圾时，都要在旁边的牌子上写：xxx正在丢垃圾，禁止插手。这样是比较麻烦的，因为每个人都要在操作时看一下垃圾桶是不是被别人占用了。
>
> 现在办公室引进了智能自动垃圾桶，只要你对着它丢垃圾，它就会自动打开垃圾桶，等垃圾进去以后又自动关上，一气呵成，别人想插手也来不及，也不需要插手了。这就是原子操作，一气呵成，中途不发生任何的上下文切换。那么这种智能自动垃圾桶就带有原子性。
>
> 原子操作一旦开始，就会持续到结束，其实这是CPU提供的功能。在多核 CPU 下，当某个 CPU 核心开始运行原子操作时，会先暂停其它 CPU 内核对内存的操作，以保证原子操作不会被其它 CPU 内核所干扰。

原子变量虽然也是串行，该堵车的时候还是要堵，但还是比锁要高效很多。

原子变量使用了特殊的CPU指令，在运行时只允许一个线程访问原子变量。这导致原子变量比普通变量效率低一些。

C++11为我们提供了原子模板类`atomic`，被定义在头文件atomic中。原子变量不能拷贝。

原子模板类接收任何整形作为模板参数，比如int、long、char(本质上是一个字节的整数)、bool(本质上是0-1变量)等。自定义类型、浮点型都是不允许的。定义之后，大多数原子变量能够和平时我们使用的整形变量一样操作：加减乘除等等。除了这些，原子变量还有很多成员函数，这些都是带有原子性的。

- `store(value)`

    因为`=`运算符在类中相当于拷贝，而原子变量不允许拷贝，那赋值时就需要使用到`store()`将值value赋值给自己

- `load(value)`

    返回原子变量的值

- 还有很多操作和重载的运算符，这些都是原子操作

    ![image-20221212222057623](http://md.ruimix.top/md/image-20221212222057623.png)

    

    比如：

    ```
    atomic<int> i;
    i.store(100); 		// 原子操作
    ```

    使用原子变量在临界区内不用上锁，我们直接用就行了。

    我们使用互斥锁小节的例子，将其修改为使用原子变量：

    ```
    #include <iostream>
    #include <thread>
    #include <chrono>
    #include <atomic>
    #include <vector>
    using namespace std;
    
    int func(atomic<int>* p) {
    	thread::id t_id = this_thread::get_id();		// 获得自己的线程ID
    	for (int j = 0; j < 10; ++j) {
    		++(*p);
    		printf("子线程%d对结果加一：%d\n", t_id, (int)*p);
    		this_thread::sleep_for(chrono::milliseconds(100));		//休眠100毫秒
    	}
    	return 0;
    }
    
    int main() {
    	atomic<int> num = 0;
    	thread T[10];
    	for (int i = 0; i < 10; ++i) {
    		T[i] = thread(func, &num);
    	}
    	for (int i = 0; i < 10; ++i) {
    		T[i].join();
    	}
    	return 0;
    }
    ```

    结果：

    ![image-20221212223524501](http://md.ruimix.top/md/image-20221212223524501.png)



## 线程间的数据传递

现在我们还有一个问题需要解决，那就是对线程运行结果进行回收。在之前的实例中，我们的任务函数都没有返回值或返回了无意义的值。然而在实际开发中，我们很有可能需要用到这个返回值。比如在主线程中，创建子线程对一串值进行复杂的数学计算，再接收计算结果使用。但是线程不能相互访问对方的栈数据，这使得线程间的数据传输变得困难。

在Linux下的C函数中使用`pthread_join()`就行：`int pthread_join(pthread_t 要回收的线程ID, void **返回结果)`。`pthread_join()`可加入一个`void**`类型的二级指针形参，用来传入一个指针，将运行结果通过指针的形式写入到主线程指定的位置中，且成功回收返回0，失败返回错误号。在C++中`join()`并没有这样的功能。但是我们依然可以学习这种思想：在任务函数中使用一个指针参数，任务函数把需要计算的内容计算完成之后，把运行结果保存到这个指针指向的位置。

实例：

```
#include <iostream>
#include <thread>
#include <mutex>
using namespace std;

int task(int i, int * ret) {
	printf("子线程开始运行\n");
	int res = 0;
	while (i > 0) {
		res += i;
		--i;
		this_thread::sleep_for(60ms);
	}
	*ret = res;
	printf("子线程运行完毕\n");
	return res;
}

int main() {
	int num = 100;
	int num_ret = 0;
	thread t(task, num, &num_ret);
	for (int i = 0; i < 5; ++i) {
		printf("主线程运行中...\n");
		this_thread::sleep_for(500ms);
	}
	cout << "子线程销毁" << endl;
	t.join();						// 等待子线程运行完毕才能读取结果
	cout << num_ret << endl;
	return 0;
}
```

这样看起来可行，但这存在很多问题：

1. 无法回收线程在异步时的返回值，因为你不知道异步的线程什么时候结束。

2. 如果要用到子线程的结果，必须使用`join()`等待子线程运行结束才能保证数据准确。但是很多时候我们并不希望使用`join()`，比如在类的构造函数中创建线程，类的析构函数中调用`join()`，这时候你想在类里面调用`join()`是不可能的。同时这将导致`detach()`线程的结果不好回收。`pthread_join()`也有这个问题，你想回收你就必须得等子线程运行结束。

3. 在`pthread_join()`中，参数是在`pthread_join()`中使用的，`pthread_join()`这个函数会阻塞一直到子线程退出，且有返回值，可以保证指针传进去了就一定会有东西出来，如果线程中途夭折了，程序员也可以依靠返回值进行判断。但是在C++中，我们只能把参数放在任务函数中传入，缺乏判断数据有效性的依据，还要程序员自行加入判断逻辑。

   

从这些可以得出使用全局变量和指针传入的方式来获取线程的结果有很多缺陷，为此，C++11为我们提供了方便的在线程之间传输数据的方式：`future`类和`promise`类，被定义在头文件future中。

`future`类和`promise`类都是模板类，可以按照实际线程间需要传递的值来定义，既可以传递值，也可以传递异常。

**`future`类和`promise`类需要搭配使用**。而且一对`future`对象和`promise`对象只能传一次值。如果甲线程需要获得乙线程的数据，那么甲线程需要创建一个`promise`对象，然后再根据`promise`对象创建出一个相捆绑的`future`对象，然后再把`promise`对象传给乙线程。然后乙线程可以给`promise`对象进行赋值，甲线程可以调用`future`对象的相关函数来判断乙线程是否给`promise`对象赋值了，也可以将乙线程放在`promise`对象中的数据在相捆绑的`future`对象中取出。但是这种情形只能单向传输数据，如果要互相传输数据只能用线程同步了。

*因为这种传递数据的方式只能传一次，并且只能单向传递，所以多被用于线程运行结果的传递。*

> 这有点像电子取餐号牌。餐厅中有和叫号机(`promise`对象)绑定的若干电子号牌(根据`promise`对象创建出一个相捆绑的`future`对象)，顾客点完菜之后，拿着电子号牌到座位上等待。等菜品被做出来之后，前台只要选择目标顾客的号牌按呼叫，顾客手上的号牌就会发出提示。但是号牌没有主动呼叫功能，所以只能被动的接受信息，不能主动发送信息。

**`future`对象**：内部存储了一个将来会被赋值的值。但是现在不知道值是多少；如果`future`对象被对方线程通过捆绑的`promise`对象赋值了，那么当前线程就可以通过`future`类读取子线程的结果，如果对方未对捆绑的`promise`对象进行赋值，那么当前线程尝试访问`future`类的结果就会被阻塞。

**`promise`对象**：和`future`对象搭配使用，`promise`对象作为参数传递给对方线程。对方给`promise`对象赋值，那么当前线程就可以通过相捆绑的`future`对象获得值了。

接下来我们来看`future`类和`promise`类的详细结构：



### `future`类

`future`类是一个模板类，需要我们提供数据类型来进行构造。我们无需关心`future`类的构造和析构是怎样的，同时，`future`类只支持移动，不支持拷贝，这意味着每个`future`对象是独一无二的，只能供一个线程使用。如果想要让传递的数据被多个线程接收，可以使用`shared_future`。

**成员函数：**

- `std::shared_future<T> share()`

  返回一个`shared_future`对象，把数据的接受权共享给其他线程，共享之后，此`future`对象将会失效。

  `shared_future`没有`std::shared_future<T> share()`函数，和`future`在使用时的功能是一样的。

- `bool valid()`

  检查`future`对象是否有效(已经绑定了`promise`对象)

- `void wait()`

  如果数据已经传达，可以读取时不会阻塞；如果数据还没传达，将会阻塞当前线程

  如果`valid() == false`会抛出异常

- `future_status wait_for(time)`  `future_status wait_until(time)` 

  如果数据已经传达，可以读取时不会阻塞；如果数据还没传达，将会阻塞当前线程直到超出时间

  其返回值有三个：

  `future_status::deferred`	对方线程还没开始运行
  `future_status::ready`	数据已经传达
  `future_status::timeout`	对方线程正在运行，但是超过了等待时间也没等到数据

- `void get()` `T get()` `T& get()`

  返回`future`对象内部的数据

  如果数据已经传达，会读取数据；如果数据还没传达，将会阻塞当前线程直到传达，再读取数据

  如果`valid() == false`行为未定义

  

### `promise`类

`promise`类也是一个模板类，需要我们提供数据类型来进行构造。我们无需关心`promise`类的构造和析构是怎样的，同时，`promise`类只支持移动，不支持拷贝，这意味着每个`promise`对象是独一无二的，只能供一个线程使用。

**成员函数：**

- `future<T> get_future()`

  返回一个和`promise`对象绑定的`future`对象。

- `void set_value(const T& value)  ` `void set_value(T&& value)` `void set_value(T& value)` `void set_value()`

  存储要传递的值，并将绑定的`future`对象设置为读取数据就绪状态。这个操作是原子操作。

- `void set_value_at_thread_exit(const T& value)  ` `void set_value_at_thread_exit(T&& value)` 

  `void set_value_at_thread_exit(T& value)` `void set_value_at_thread_exit()`

  存储要传递的值，但不将绑定的`future`对象设置为读取数据就绪状态，而是等子线程执行完被销毁之后再设置为就绪状态。存储数据的过程是原子操作。



### 实例

```
#include <iostream>
#include <thread>
#include <mutex>
#include <future>
using namespace std;

int task(int i, promise<int>& p) {
	printf("子线程开始运行\n");
	int res = 0;
	while (i > 0) {
		res += i;
		--i;
		this_thread::sleep_for(60ms);
	}
	p.set_value(res);					// 子线程把结果保存至promise
	printf("子线程运行完毕\n");
	return res;
}

int main() {
	int num = 100;
	int num_ret = 0;
	promise<int> p;						// 定义 promise
	future<int> f = p.get_future();		// 定义和promise绑定的future
	thread t(task, num, ref(p));		// 把promise按引用传给子线程
	for (int i = 0; i < 5; ++i) {
		printf("主线程运行中...\n");
		this_thread::sleep_for(500ms);
	}
	num_ret = f.get();
	cout << num_ret << endl;
	cout << "子线程销毁" << endl;
	t.join();
	return 0;
}
```





## 线程池

多线程能够充分利用CPU资源，实现效率最大化。但是创建线程仍然需要消耗时间，因为线程的创建需要划分资源。

为了对多线程进行优化，人们发明了线程池。线程池是一种可以复用线程和动态调整线程数量的数据结构。

一个线程池由以下结构构成：

- 线程池的相关数据，比如线程池中最大、最小的线程数量，储存任务队列的大小，以及临界区的锁
- 任务队列，用于保存放入线程池中的任务
- 一个管理者线程，动态地调整线程，当线程不够时可以创建线程，过多线程闲置时可以释放闲置的线程
- N个工作线程，工作线程是线程池中执行任务的主体，内部有一个循环，通过回调不断地从任务队列中取任务并执行，不销毁线程即可实现多任务处理



### 如何保存任务函数

我们可以像C那样，把函数的参数打包成一个结构体，定义一个函数指针，再定义一个结构体指针，这样就可以传入了。然而这样的方法需要程序员在将任务函数和参数传入线程池的时候需要打包，在执行任务函数的时候对参数进行解包，这样未免有点复杂。

C++11为我们提供了可调用对象包装器，被定义在头文件functional中

- 可变参数模板

  可变参数模板为我们提供了一个可以接受任意数量和类型参数的函数。这符合线程池的工作线程的任务函数要求，因为你不知道线程池需要执行什么样的任务

  不过需要注意的是，模板类\模板类成员函数和普通函数不一样，普通函数是客观存在的，可以随时调用；而模板函数需要等到调用的时候，才会被编译器用参数类型实例化出一个对应参数的函数，再去调用这个对应的函数。

  因为C++编译器是先将每个文件编译，编译完了再进行汇编和链接，假设函数声明被放在`func.h`头文件中，函数实现被放在`func.cpp`源文件中(`func.cpp`源文件包含了`func.h`头文件)，在`main.cpp`里面包含了`func.h`头文件并且调用了函数。根据编译器的特性，编译过程中在链接阶段之前会生成两个可成重定位二进制文件：`main.o`和`func.o`。

  - 如果函数是普通函数。因为普通函数客观存在，函数在`func.o`中可以随时调用，编译器很容易就找到了函数的实现并链接地址。

  - 如果函数是模板函数，在`main.o`文件中，因为调用了模板函数，模板函数的声明能够被顺利实例化，但是没有实现；在`func.o`文件中，因为没有调用模板函数，所以模板函数的实现不能实例化。所以在链接阶段，链接器会找不到函数的对应的参数的具体实现，就报错了。

  解决的方法是不使用分文件声明+实现的方式，我们可以直接把模板函数的声明和实现写在一个文件里面，我们约定这样的文件后缀名为.hpp

- `std::bind`

  为我们提供了一个将函数和参数绑定在一起的方式，或者将可调用对象和对象绑定在一起的方式。

  `bind()`会形成一个重载了调用运算符的类，也就是仿函数。其返回值为空，参数也为空。`void Object.operator() ();`

- `template<class T>  std::function<T>`

  为我们提供了一个保存和延迟调用函数的方式。曾经当我们手上有一个可调用对象时，要么调用它，要么释放它。现在我们可以通过`std::funtion`来保存这个可调用对象了。

  其中，我们需要隆重介绍`function<void()>`，这个类型可以保存返回值为void、参数列表为空的函数，甚至可以把可调用对象保存在数组里面。这恰好符合我们线程池的要求。

  

实例：

```
#include <iostream>
#include <functional>
using namespace std;

void MyFunc(int i, char j) {
    cout << "MyFunc function called, a int arg:" << i << " ,a char arg:" << j << endl;
}

// 可变参数模板
template<class F, class... Args>
void callback(F func, Args... args) {
    cout << "callback function called" << endl;

    function<void()> arr[5];
    // function<void()>类型包装了返回值为void、参数列表为空的函数
    arr[0] = bind(func, args...);
    // bind()把函数和参数绑定起来，形成一个仿函数：返回值为void、参数列表为空void     void Object.operator() ();
    // 如果是成员函数，还能把成员函数对象和成员函数绑定起来，再和参数绑定起来
    // 这样刚好就能被function<void()>包装存储起来
    arr[0]();
    return;
}

class MyClass {
public:
    void m_func() {
        cout << "MyClass::m_func function called" << endl;
    }
    void m_func(int i) {
        cout << "MyClass::m_func function called, a int arg:" << i << endl;
    }
};

int main() {
    callback(MyFunc, 555, 'v');
    MyClass mc;
    // 如果存在重载，那么必须指定重载的是哪一个
    callback((void(MyClass::*)())&MyClass::m_func,&mc);
    callback((void(MyClass::*)(int))&MyClass::m_func,&mc,100);
    callback([](int i){cout << "lambda function called, a int arg:" << i << endl; }, 666);
    return 0;
}
```

请注意，`std::function`绑定类的非静态成员函数需要提供类的成员函数名称和类的指针，那么如果`std::function`绑定了一个类的非静态成员函数后，先析构类对象，再调用绑定的可调用对象会发生未定义行为，你应该避免这种行为，因此你需要注意类的生命周期，使用智能指针、RAII机制都能避免这个问题。



### 线程池的具体实现

ThreadPool.hpp

```
#include <atomic>
#include <thread>
#include <mutex>
#include <condition_variable>
#include <queue>
#include <list>
#include <vector>
#include <functional>
#include <chrono>

#ifndef THREADPOOL_H
#define THREADPOOL_H

class WorkerThread;

class ThreadPool {
    friend WorkerThread;
public:
    ThreadPool(int min_thread_num, int max_thread_num, int max_task_num);     // 构造
    ~ThreadPool();                                                            // 析构
    template<class F, class... Args>
    int AddTask(F func, Args... args);                                        // 【模板】添加任务
    int Start();                                                              // 开始运作
    void Terminate();                                                         // 结束运作
    void SafelyExit(bool);                                                    // 设置强制结束
    void ReceiveAllTask(bool);                                                // 设置队列满时阻塞添加任务
    int GetMaxThreadNum() const;
    int GetMinThreadNum() const;
    int GetMaxTaskNum() const;

private:
    int max_thread_num;                         // 线程池内能达到的最大线程数量
    int min_thread_num;                         // 线程池内至少存在的线程数量
    int max_task_num;                           // 线程池存储的最大任务数量，0代表无上限
    int wave_range = 3;                         // 每次调整加/减多少线程
    int alive_thread_num;                       // 存活的线程数，只有管理者线程修改，不需加锁
    std::atomic<int> working_thread_num{};      // 正在工作的线程数
    std::atomic<int> exit_thread_num{};         // 准备退出的线程数
    std::queue<std::function<void()>> *task_queue = nullptr;    // 任务队列
    std::mutex task_queue_mutex;                // 锁任务队列
    std::list<WorkerThread *> *worker_list = nullptr; // 存放线程的链表
    std::thread *manager = nullptr;             // 管理者线程
    std::condition_variable cond;               // 任务条件变量
    std::vector<std::thread::id> *to_destroy = nullptr;          // 存放需要进行销毁的线程ID
    std::mutex to_destroy_mutex;                // 锁销毁队列
    short state_code;                           // 状态码：0创建但未运行 1正在运行 2结束准备销毁
    bool destroy_with_no_task = true;           // 是否在销毁线程池之前执行完任务队列中剩下的任务，默认是
    bool block_task_when_full = true;           // 在任务队列满的时候是否阻塞添加任务函数，默认是
    // 如果选择否，那么在任务队列满的时候添加任务会返回-1

    void manager_func();                        // 管理者线程函数
};

// 把工作线程包装成一个类，类有构造和析构可以用于创建和销毁
class WorkerThread {
    friend ThreadPool;
    std::thread::id thread_id;              // 线程自己的线程ID
    std::thread *thread_ptr = nullptr;     // 自己要做的事
    ThreadPool *pool = nullptr;            // 自己属于哪个线程池

    void worker_func();
    explicit WorkerThread(ThreadPool *);
    ~WorkerThread();
};

/***************以下是类的成员函数实现***************/

ThreadPool::ThreadPool(int min_thread_num, int max_thread_num, int max_task_num) {
    if (max_thread_num < min_thread_num || max_thread_num <= 0 ||
        min_thread_num < 0 || max_task_num < 0)
        return;
    this->max_thread_num = max_thread_num;
    this->min_thread_num = min_thread_num;
    this->max_task_num = max_task_num;
    alive_thread_num = 0;
    working_thread_num.store(0);
    exit_thread_num.store(0);
    state_code = 0;

    do {
        this->to_destroy = new std::vector<std::thread::id>;
        if (this->to_destroy == nullptr) break;
        this->task_queue = new std::queue<std::function<void()>>;
        if (this->task_queue == nullptr) break;
        this->worker_list = new std::list<WorkerThread *>;
        if (this->worker_list == nullptr) break;
        std::cout << "创建线程池成功，线程数量为" << this->min_thread_num << "~" << this->max_thread_num << "个"
                  << std::endl;
        return;
    } while (false);
    delete this->to_destroy;
    delete this->task_queue;
    delete this->worker_list;
}

ThreadPool::~ThreadPool() {
    if (state_code != 2) {
        Terminate();
    }
    delete this->to_destroy;
    delete this->task_queue;
    delete this->manager;
    delete this->worker_list;
}

template<class F, class... Args>
int ThreadPool::AddTask(F func, Args... args) {
    // 线程池还没开启的时候不可以添加任务
    if (state_code != 1) return -1;
    std::unique_lock<std::mutex> uniqueLock(task_queue_mutex);
    if (block_task_when_full) {
        // 满的时候休眠
        while (task_queue->size() >= max_task_num) {
            cond.wait(uniqueLock);
            // 当被唤醒之后发现不允许添加任务立即返回
            if (max_task_num == -1) {
                uniqueLock.unlock();
                return -1;
            }
        }
    } else {
        // 当不允许添加任务立即返回
        if (max_task_num == -1) {
            uniqueLock.unlock();
            return -1;
        }
        // 当max_task_num不为0并且任务数量抵达上限时返回-1
        if (max_task_num != 0 && task_queue->size() >= max_task_num) {
            uniqueLock.unlock();
            return -1;
        }
    }
    task_queue->push(std::bind(func, args...));
    uniqueLock.unlock();

    cond.notify_all();
    return 0;
}

int ThreadPool::Start() {
    if (state_code != 0) {
        return -1;
    }
    state_code = 1;
    // 创建管理者线程
    this->manager = new std::thread(&ThreadPool::manager_func, this);
    // 创建工作线程
    for (int i = 0; i < min_thread_num; ++i) {
        worker_list->push_back(new WorkerThread(this));
        ++alive_thread_num;
    }
    return 0;
}

void ThreadPool::Terminate() {
    if (state_code == 0) {
        state_code = 2;
        return;
    }
    if (state_code == 1) {
        // 如果要放弃剩下的任务，清空任务队列
        if (!destroy_with_no_task) {
            task_queue_mutex.lock();
            delete task_queue;
            task_queue = new std::queue<std::function<void()>>;
            task_queue_mutex.unlock();
        }
        // 阻止继续添加任务
        max_task_num = -1;
        // 如果还有线程在工作，则等待
        while (working_thread_num.load() != 0) {
            std::this_thread::sleep_for(std::chrono::milliseconds(100));
        }
        // 让管理者线程和工作线程退出循环
        state_code = 2;
        // 回收管理者线程
        manager->join();
        // 唤醒正在睡眠的子线程
        cond.notify_all();
        // 回收工作线程
        for (auto x: *worker_list) {
            delete x;
        }
        return;
    }
}

void ThreadPool::manager_func() {
    std::cout << "管理者线程开始工作" << std::endl;
    // 当线程池开启时一直工作
    while (state_code == 1) {
        // 每2秒监视一次
        std::this_thread::sleep_for(std::chrono::seconds(2));
        printf("管理者线程监视一次，当前存活线程数量%d个，正在工作线程%d个，任务%zu个\n", alive_thread_num,
               working_thread_num.load(), task_queue->size());
        // 线程不够时创建线程：存活数<最大数 并且 所有线程都在工作
        if (alive_thread_num < max_thread_num &&
            alive_thread_num - working_thread_num.load() == 0) {
            // 创建的线程数量 = 调整量 或 最大-当前
            int addNum = wave_range < max_thread_num - alive_thread_num ?
                         wave_range : max_thread_num - alive_thread_num;
            for (int i = 0; i < addNum; ++i) {
                worker_list->push_back(new WorkerThread(this));
                ++alive_thread_num;
            }
            continue;
        }

        // 线程过多时销毁线程：存活数>最小数 并且 一些线程没事干在睡眠
        // 让一些线程主动退出
        task_queue_mutex.lock();    // 为了防止在清点闲置线程的时候突然来任务干扰，把任务队列锁起来
        if (alive_thread_num > min_thread_num &&
            working_thread_num.load() + wave_range < alive_thread_num) {
            // 销毁的线程数量 = 调整量 或 当前 - 最小
            int destroyNum = wave_range < alive_thread_num - min_thread_num ?
                             wave_range : alive_thread_num - min_thread_num;
            exit_thread_num.store(destroyNum);
            task_queue_mutex.unlock();
            // 唤醒对应个睡眠的线程
            // 在此情景下，任务队列应该是空的，添加任务的函数那里不会休眠，所以不会被唤醒
            for (int i = 0; i < destroyNum; ++i) {
                cond.notify_one();
            }
            // 等待要被回收的线程就绪
            while (exit_thread_num.load() != 0) {
                std::this_thread::sleep_for(std::chrono::milliseconds(10));
            }
            // 回收主动退出的线程
            to_destroy_mutex.lock();
            for (auto x: *to_destroy) {
                for (auto iter = worker_list->begin(); iter != worker_list->end(); ++iter) {
                    if (x == (*iter)->thread_id) {
                        delete (*iter);
                        worker_list->erase(iter);
                        --alive_thread_num;
                        break;
                    }
                }
            }
            to_destroy->clear();
            to_destroy_mutex.unlock();
        } else {
            task_queue_mutex.unlock();
        }
    }
    std::cout << "管理者线程结束工作" << std::endl;
}

void ThreadPool::SafelyExit(bool arg) {
    if(state_code == 0)
        this->destroy_with_no_task = arg;
}

void ThreadPool::ReceiveAllTask(bool arg) {
    if(state_code == 0)
        this->block_task_when_full = arg;
}

int ThreadPool::GetMaxThreadNum() const {
    return this->max_thread_num;
}
int ThreadPool::GetMinThreadNum() const {
    return this->min_thread_num;
}
int ThreadPool::GetMaxTaskNum() const {
    return this->max_task_num;
}

WorkerThread::WorkerThread(ThreadPool *pool) {
    this->pool = pool;
    this->thread_ptr = new std::thread(&WorkerThread::worker_func, this);
    this->thread_id = thread_ptr->get_id();
    printf("创建了一个线程++++++++++++++++++++++\n");
}

WorkerThread::~WorkerThread() {
    this->thread_ptr->join();
    delete this->thread_ptr;
    printf("销毁了一个线程-----------------------\n");
}

void WorkerThread::worker_func() {
    std::cout << "工作线程开始工作" << std::endl;
    // 当线程池开启时一直工作
    while (pool->state_code == 1) {
        // 给任务队列加锁
        std::unique_lock<std::mutex> uniqueLock(pool->task_queue_mutex);
        // 当任务队列为空时睡眠等待
        while (pool->task_queue->empty()) {
            pool->cond.wait(uniqueLock);
            // 当线程醒来发现需要有线程退出
            if (pool->exit_thread_num.load() != 0) {
                --pool->exit_thread_num;
                uniqueLock.unlock();
                // 在要销毁的线程列表中存入自己的线程ID
                pool->to_destroy_mutex.lock();
                pool->to_destroy->push_back(this->thread_id);
                pool->to_destroy_mutex.unlock();
                return;
            }
            // 当线程醒来发现要关闭
            if (pool->state_code != 1) return;
            // 先判断需不需要退出再判断是不是要关闭线程池
            // 防止关闭线程池时管理者还在等待工作线程退出，造成管理者一直阻塞等待
        }
        // 从任务队列取一个任务运行
        std::function<void()> task = pool->task_queue->front();
        pool->task_queue->pop();
        // 释放任务队列锁
        uniqueLock.unlock();
        // 唤醒添加任务的函数
        pool->cond.notify_one();
        // 工作线程数量加一
        ++pool->working_thread_num;
        // 执行任务
        task();
        // 执行完任务，工作线程数量减一
        --pool->working_thread_num;
    }
}

#endif

```

测试main.cpp

```
#include <iostream>
#include "ThreadPool.hpp"

/* 某工厂生产一种食品，该产品有淡季和旺季。春秋是淡季，夏冬是旺季。
 * 当淡季时，订单数会有所下降，旺季则会激增
 * 为了最小化人力财政支出，该工厂除了几个长工外，其余均为临时工
 * 其中长工为3人，整个工厂最多只能有15个员工
 * 假设每个季节只有30秒，其中每一笔订单需5秒完成
 * 春季的订单速度为1.5秒每单，夏季为0.2秒每单，秋季为3秒每单，冬季为0.5秒每单
 * 那么春季总计20单，夏季150单，秋季10单，冬季60单
 * 预计需要人力春季4人，夏季25人，秋季2人，冬季10人
 */

void Order(int num) {
    printf("有一笔订单，订单编号：%d\n", num);
    std::this_thread::sleep_for(std::chrono::seconds(5));
    printf("订单编号：%d 的订单完成！\n", num);
}

int main() {
    system("chcp 65001");
    ThreadPool tp(3, 15, 30);
    //tp.SafelyExit(false);
    //tp.ReceiveAllTask(false);
    tp.Start();
    printf("========================春季来临========================\n");
    for (int i = 0; i < 20; ++i) {
        int j = rand();
        if(tp.AddTask(&Order, j) == -1){
            printf("订单%d被退回\n",j);
        }
        std::this_thread::sleep_for(std::chrono::milliseconds(1500));
    }
    printf("========================夏季来临========================\n");
    for (int i = 0; i < 150; ++i) {
        int j = rand();
        if(tp.AddTask(&Order, j) == -1){
            printf("订单%d被退回\n",j);
        }
        std::this_thread::sleep_for(std::chrono::milliseconds(200));
    }
    printf("========================秋季来临========================\n");
    for (int i = 0; i < 10; ++i) {
        int j = rand();
        if(tp.AddTask(&Order, j) == -1){
            printf("订单%d被退回\n",j);
        }
        std::this_thread::sleep_for(std::chrono::milliseconds(3000));
    }
    printf("========================冬季来临========================\n");
    for (int i = 0; i < 60; ++i) {
        int j = rand();
        if(tp.AddTask(&Order, j) == -1){
            printf("订单%d被退回\n",j);
        }
        std::this_thread::sleep_for(std::chrono::milliseconds(500));
    }
    printf("========================今年结束========================\n");
    return 0;
}
```

