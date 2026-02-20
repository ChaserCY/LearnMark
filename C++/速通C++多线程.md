
# 一，创建，运行线程
std::thread <线程名>(<函数名>，<参数1>，...) //==创建会拷贝参数，如果传入参数不能拷贝，必须使用std::move()==
<线程名>.join();  //等待子线程运行结束，才会继续主线程
<线程名>.detach();//子与主同时运行，把子挂在后台的
<线程名>.joinable();   //返回bool值，判定是否可以进行join或者detach

# 二，线程函数中的数据未定义错误
## 1.在线程中传递引用类型(目的是修改，而线程的方法默认是拷贝)
std::thread <线程名>(<函数名>，std::ref(<参数1>),...）;
方法：std::ref()     //括号里填参数，一般用在函数传值中，引用传值，可以修改该参数
## 2.传递指针或引用指向局部变量的问题
```cpp
#include<iostream>
#include<thread>

std::thread t;


void foo(int& x) {
	x += 1;
}

void test() {
	int a = 1;
	t = std::thread(foo, std::ref(a));
}

int main() {
	test();
	t.join();
	return 0;
}
```
## 3.传递指针或引用已释放的内存的问题
线程创建后，紧跟清除内存，导致指针的引用为空，那么可能造成野指针
```cpp
#include<iostream>
#include<thread>

std::thread t;

void foo(int* x) {
	*x += 1;
	std::cout << *x << std::endl;
}


int main() {
	int* ptr = new int(1);
	std::thread t(foo, ptr);
	delete ptr;//这个位置
	t.join();
	return 0;
}
```
## 4.类成员函数作为入口函数，类对象被提前释放

创建方法：
std::thread(&类名::函数名，&对象，参数....)


```cpp
#include<iostream>
#include<thread>
#include<memory>
class A {
public:
	void foo() {
		std::cout<<"Hello World !" << std::endl;
	}
};
int main() {
	std::shared_ptr<A> a = std::make_shared<A>();
	std::thread t(&A::foo, a);
	//A a;
	//std::thread t(&A::foo, &a);//直接这样写，可能这个a会在t.join();之前的代码中把a对象释放掉，导致没有a的地址，那就使用智能指针，使得a自己在线程结束后在析构函数中释放
	t.join();
	return 0;
}
```

## 5.入口函数为类的私有成员函数
就是想用线程来执行类中的==私有==函数时，会报错


```cpp
#include<iostream>
#include<thread>
#include<memory>
class A {
private://原因是这里无法被访问
	void foo() {
		std::cout<<"Hello World !" << std::endl;
	}
};

int main() {
	std::shared_ptr<A> a = std::make_shared<A>();

	std::thread t(&A::foo, a);//这里会报错
	
	t.join();
	
	return 0;
}
```

解决：使用友元（friend）来暂时给别的函数当前私有函数的执行权限
```cpp
#include<iostream>
#include<thread>
#include<memory>
class A {
private:
	friend void thread_foo();//这里使用了友元，在私有中“授权”
	void foo() {
		std::cout<<"Hello World !" << std::endl;
	}
};

void thread_foo() {
	std::shared_ptr<A> a = std::make_shared<A>();

	std::thread t(&A::foo, a);
	
	t.join();
}

int main() {
	thread_foo();
	return 0;
}
```

# 三，互斥量解决多线程数据共享问题
即多个子线程创建同时修改了同一数据引发的问题

解决方法：在操作前后使用==互斥锁（mutex）==
```cpp
#include<iostream>
#include<thread>
#include<mutex>

int a = 0;
std::mutex mtx;
void func() {
	for (int i = 0; i < 100; i++) 
	{
		mtx.lock();   //在操作前锁住
		a += i;
		mtx.unlock();   //操作完成后解锁
	}
}
int main() {
	std::thread t1(func);
	std::thread t2(func);

	t1.join();
	t2.join();
	
	std::cout << a << std::endl;
	return 0;
}
```

# 四，互斥量死锁
即两个互斥量都需要对方锁住的资源，同时进行等待，锁死，时间线上的重叠

错误代码：
```cpp
#include<iostream>
#include<thread>
#include<mutex>

int a = 0;
std::mutex m1,m2;
void func_1() 
{
	m1.lock();
	m2.lock();
	m1.unlock();
	m2.unlock();
}

void func_2()
{
	m2.lock();
	m1.lock();
	m1.unlock();
	m2.unlock();
}


int main() {
	std::thread t1(func_1);
	std::thread t2(func_2);

	t1.join();
	t2.join();
	
	std::cout << a << std::endl;
	return 0;
}
```

## AI 解释：
### 1. 死锁发生在什么地方？

死锁不是发生在某一行代码上，而是发生在**两个线程的时间线交错**时。具体来说，卡在 **`func_1` 的第 10 行** 和 **`func_2` 的第 18 行**。

让我们看一个最坏情况的“剧本”：

1. **时刻 1**：线程 `t1` 启动，执行 `func_1`。它成功拿到了 `m1` 的锁（执行完第 9 行 `m1.lock()`）。
    
    - 此时：`t1` 手里有 **m1**。
        
2. **时刻 2**：操作系统突然切换线程（Context Switch），暂停了 `t1`，让 `t2` 开始跑。
    
3. **时刻 3**：线程 `t2` 启动，执行 `func_2`。它成功拿到了 `m2` 的锁（执行完第 17 行 `m2.lock()`）。
    
    - 此时：`t2` 手里有 **m2**。
        
4. **时刻 4（死锁发生）**：
    
    - `t1` 醒来，想继续执行第 10 行 `m2.lock()`。但是 `m2` 在 `t2` 手里，所以 `t1` **被卡住（阻塞）**，必须等 `t2` 放手。
        
    - `t2` 继续跑，想执行第 18 行 `m1.lock()`。但是 `m1` 在 `t1` 手里，所以 `t2` 也**被卡住**，必须等 `t1` 放手。
        

**结果**：`t1` 等 `t2`，`t2` 等 `t1`，谁也不让谁，程序永久卡死。

### 2. `m1`, `m2` 锁住的是什么东西？

这是一个非常棒的初学者问题！

**直觉上**：你可能以为 `m1` 锁住了 `a`，或者 `m2` 锁住了某个具体的变量。  
**实际上**：**锁（Mutex）本身不与任何变量强绑定。**

- `std::mutex` 就像是一个**“进出门的令牌”**。
    
- 它**锁住的是“代码的执行权”**。
    

在你的代码中，`m1.lock()` 的意思是：“只要我拿着 `m1` 这个令牌，其他任何想要拿 `m1` 的人（线程）都得在原地等着，直到我调用 `m1.unlock()` 把令牌扔回去。”

虽然你定义了全局变量 `int a = 0;`，但在这个例子里，你的锁其实**什么数据也没保护**。因为在 `lock` 和 `unlock` 之间，你并没有去修改 `a` 或者其他变量。你只是为了演示死锁，写了纯粹的加锁逻辑。

如果这是真实业务代码，通常会写成这样，那锁住的就是 `a`：

cpp

```cpp
m1.lock();
a = a + 1; // 此时 m1 保护了 a，别人不能同时改 a
m1.unlock();
```

### 3. 这个 lock/unlock 的排队顺序是什么意思？

这里的顺序是导致死锁的**根本原因**。

- **`func_1` 的顺序（1 -> 2）**：
    
    - 我要先拿令牌 1。
        
    - 手里攥着令牌 1 的情况下，我再去申请令牌 2。
        
    - 事情办完后，还回 1，再还回 2。
        
- **`func_2` 的顺序（2 -> 1）**：
    
    - 我要先拿令牌 2。
        
    - 手里攥着令牌 2 的情况下，我再去申请令牌 1。
        

**为什么这很危险？**  
这就像两个人在过独木桥：

- A 穿着左脚鞋（拿着 m1），非要等拿到右脚鞋（m2）才肯走。
    
- B 穿着右脚鞋（拿着 m2），非要等拿到左脚鞋（m1）才肯走。
    

**如何修复？（黄金法则）**  
为了避免死锁，所有线程获取锁的**顺序必须一致**。  
如果你把 `func_2` 改成和 `func_1` 一样：**先锁 m1，再锁 m2**，死锁就立刻解决了。因为如果 `t1` 拿到了 `m1`，`t2` 在第一步申请 `m1` 时就会被卡住，根本没机会去拿 `m2`，从而避免了互相卡死。

# 五，lock_guard与unique_lock
- **std::lock_guard**：轻量级，构造时锁，析构时开，功能简单，性能最好。
- **std::unique_lock**：重量级，功能强大（支持中途解锁、延迟锁定、配合条件变量等），由于内部要维护状态，**性能比 lock_guard 差**。

## std::lock_guard
- 创建：
	std::lock_guard<std::mutex>  新锁的名(mutex锁的名)；
- 特点：
	不能复制与移动
	自动加锁，自动解锁
```cpp
#include<iostream>
#include<thread>
#include<mutex>

int share_data = 0;
std::mutex mtx;

void func()
{
	for (int i = 0; i < 50; i++) {
		
		std::lock_guard<std::mutex> lg(mtx);//自动加锁，结束自动解锁
		share_data++;
		
		//mtx.lock();
		//share_data += i;
		//mtx.unlock();
	}
}


int main() {
	std::thread t1(func);
	std::thread t2(func);

	t1.join();
	t2.join();
	
	std::cout << a << std::endl;
	return 0;
}
```
## std::unique_lock
支持自动上锁和自动解锁
而且还有更多功能，更常用，更安全
- 构造1（自动加解锁）：
	std::unique_lock<std::mutex> lg(mtx);
- 构造2（延迟/等待加锁）：
	- std::unique_lock<std::mutex> ==lg==(mtx, std::defer_lock);//取消了“构造即加锁”的能力，若能锁后面会自动解锁
	- ==lg==.try_lock_for(std::chrono::seconds(5));//等待5s来锁mtx，5s内若是等到了返回true,没有就返回false
	- 延迟加锁，并且会定时放弃，正常的锁会一直等待

```cpp
#include<iostream>
#include<thread>
#include<mutex>

int share_data = 0;
std::mutex mtx1;
std::timed_mutex mtx;
void func()
{
	for (int i = 0; i < 2; i++) {
		
		//std::lock_guard<std::mutex> lg(mtx);//自动加锁，结束自动解锁
		//std::unique_lock<std::mutex> lg(mtx);
		std::unique_lock<std::timed_mutex> lg(mtx, std::defer_lock);//取消了“构造即加锁”的能力，若能锁后面会自动解锁
		if (lg.try_lock_for(std::chrono::seconds(2))) //等待2s来锁mtx，2s内若是等到了返回true,没有就返回false流程继续走
		{
			std::this_thread::sleep_for(std::chrono::seconds(1));
		}
		share_data++;
		/*mtx.lock();
		share_data ++;
		mtx.unlock();*/
	}
}


int main() {
	std::thread t1(func);
	std::thread t2(func);

	t1.join();
	t2.join();
	
	std::cout << share_data << std::endl;
	return 0;
}
```

# 六，call_once单例模式与使用场景(多线程)
## 格式
```cpp
#include <thread>

std::once_flag fff; //状态标记命名，几个不同的单例事件，就命名几个

std::call_once(fff, <函数/线程>); //启动call_once功能，传入状态标记，看现在是什么情况


```
## 懒汉模式
这里有关Log对象的返回，参数都要用==引用&==
![](图片/C++/速通C++多线程/IMG-20260208224653767.png)
- **生存期 (Lifetime)**：普通局部变量的生存期仅限于函数执行期间；而 ==static 变量的生存期是 **从首次初始化开始，直到整个程序结束**==。它**常驻内存**（数据段或 BSS 段）。
    
- **Meyers' Singleton**：这种写法以 C++ 专家 Scott Meyers 的名字命名。它是实现单例模式最优雅、最简洁、且**默认线程安全**的方式，比手写 if (instance == nullptr) { lock(); ... }（双重检查锁定）要高效且安全得多。

## 饿汉模式
![](图片/C++/速通C++多线程/IMG-20260210004415986.png)
## 与普通锁(mutex)的区别
![](图片/C++/速通C++多线程/IMG-20260211181313104.png)
![](图片/C++/速通C++多线程/IMG-20260211181351528.png)

# 七，condition_variable 与其使用场景
## 概念
**std::condition_variable** 是 C++ 标准库 <condition_variable> 头文件中定义的同步原语。

- **功能**：它允许一个或多个线程 **阻塞（Block/Sleep）**，直到另一个线程修改了共享变量（条件），并通知（Notify）该条件变量。
    
- **机制**：
    
    - **等待端 (Consumer)**：持有锁 -> 检查条件 -> 如果条件不满足，调用 wait() -> **自动释放锁** 并进入睡眠 -> 被唤醒后 **自动重新拿锁** -> 再次检查条件。
        
    - **通知端 (Producer)**：持有锁（可选，但推荐） -> 修改条件（如向队列塞数据） -> 调用 notify_one() 或 notify_all() 唤醒等待的线程。
        
- **核心方法**：
    
    - wait（lock， predicate）： 阻塞等待。
        
    - notify_one(): 唤醒一个正在等待的线程。
        
    - notify_all(): 唤醒所有正在等待的线程。
## 示例模型
![](图片/C++/速通C++多线程/IMG-20260212223732182.png)
## 应用方式
```cpp
#include<mutex>
#include<condition_variable>  //所需头文件

std::mutex mtx;//命名某一标志锁   
std::condition_variable g_cv;//命名某一个消费等待标志

std::unique_lock<std::mutex> lockk(mtx);//给线程的两处共同操作加了个自动锁

g_cv.wait(lockk, []() {return !g_queue.empty();}); // 第二个参数：false：开始等待，之后接收到通知(g_cv.notify_one()执行)，等待结束。true:不等待

g_cv.notify_one();//符合某先后条件，开始通知等待的线程

```
## 源码参考
![](图片/C++/速通C++多线程/IMG-20260212224858384.png)

# 八，跨平台线程池
可以使用单例模式，智能指针，call_once拓展线程池
思路：
![](未命名.canvas)

源码：
```cpp
#include <iostream>
#include <thread>
#include <mutex>
#include <condition_variable>
#include <vector>
#include <functional>
#include <queue>
class ThreadPool {
public:
	ThreadPool(int numTreads) :stop(false) {   //线程池构造函数，创建num个线程
		for (int i = 0; i < numTreads; i++) {
			threads.emplace_back([this] {//添加线程，构造std::thread 即启动线程
				while (1) 
				{
					std::unique_lock<std::mutex> lock(mtx);
					condition.wait(lock, [this] { return !tasks.empty() || stop; });//当队列为空且线程池停止时，线程等待，wait的false是等待
					
					if (stop && tasks.empty())return;//当队列为空且线程池停止时，线程退出循环

					std::function<void()> task(std::move(tasks.front()));
					tasks.pop(); //用于清理队列中已被清理队列中已被移动的“空移动的“空壳”元素<壳”元素
					lock.unlock();//手动解锁
					task();//执行任务
				}
				
				});
			
		}
	}
	~ThreadPool() {
		{
			std::unique_lock<std::mutex> lock(mtx);
			stop = true;
		}//这个中括号出现，是为了手动限制锁的作用域

		condition.notify_all();
		for (auto& t : threads) 
		{
			t.join();
		}
	}

	template<class F, class... Args>//标记下一个函数，不是具体代码，根据实体输入来读取代码
	void enqueue(F&& f, Args&&... args) {//F&& f:万能引用，能接受任意类型的一个函数，Args&&... args:万能引用，表示任意参数数量和类型
		std::function<void()>task =//添加更改了“任意参数”的函数对象（泛型函数对象）
			std::bind(std::forward<F>(f), std::forward<Args>(args)...);//bind:将一个函数对象和一组参数绑定到一起，返回一个新的函数对象，这个函数对象可以像原函数一样被调用，但是会将参数绑定到一起。
		//forward:“完美转发”，将参数属性(左右值)不变，传递给外层函数。不用的话：当是右值(f为Lambda)时，将会退化为左值，进行多了一次拷贝，浪费性能
		{
			std::unique_lock<std::mutex> lock(mtx);
			tasks.emplace(std::move(task));
		}
		condition.notify_one();
	}



private:
	std::vector<std::thread> threads;//线程数组
	std::queue<std::function<void()>> tasks; //任务列表队列

	std::mutex mtx;
	std::condition_variable condition;

	bool stop;
};



int main() {
	ThreadPool pool(4);

	for (int i = 0; i < 10; i++) {
		pool.enqueue([i] {
			std::cout << "task : " << i << " is running " << std::endl;
			std::this_thread::sleep_for(std::chrono::seconds(1));
			std::cout << "task : " << i << " is done " << std::endl;

			});
	}

	return 0;
}

```

# 九，异步并发编程
## 1.**std::future** 和 **std::async**
<#include \<future>> 头文件
![](图片/C++/速通C++多线程/IMG-20260219233619472.png)
- 与Thread对比
![](图片/C++/速通C++多线程/IMG-20260219233207666.png)
## 2.packaged_task（thread与future的桥梁）
这里packaged_task的意义：
你可手动的去启动线程，而不是像async直接启动线程
![](图片/C++/速通C++多线程/IMG-20260220165217075.png)
## 3.std::promise
在C++中，promise是一个类模板，用于在一个线程中产个值，并在另一个线程中获取这个值。promise通常与future和async一起使用，用于实现异步编程。
```cpp
void fun1(std::promise<int> &f) {

	f.set_value(1000);
}

int main()
{

	std::promise<int> f; //创建一个promise对象，promise对象是一个模板类，可以用来传递一个值或者一个异常，promise对象可以用来创建一个future对象，future对象可以用来获取promise对象传递的值或者异常
	auto future_result = f.get_future();

	//std::thread t1(fun1, std::move(f));//std::promise对象是不可复制的，所以需要移动promise对象，std::move(f)将promise对象f转换为右值引用，传递给线程函数fun1
	std::thread t1(fun1, std::ref(f)); //std::ref(f)将promise对象f转换为引用，传递给线程函数fun1
	t1.join();

	std::cout << future_result.get() << std::endl;

	return 0;
}
```
![](图片/C++/速通C++多线程/IMG-20260220171110359.png)

# 十，std::atomic 原子操作
头文件：\<atomic>
==不用自己加锁，效率更高==
std:atomic是C++11标准库中的一个模板类，用于实现多线程环境下的原子操作。它提供了一种线程安全的方式来访问和修改共享变量，可以避免多线程环境中的数据竞争问题。
![](图片/C++/速通C++多线程/IMG-20260220181329131.png)