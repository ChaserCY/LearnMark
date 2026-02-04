
# 一，创建，运行线程
std::thread <线程名>(<函数名>，<参数1>，...)
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

# 五，lock_guard与unipue_lock

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
## std::unipue_lock
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

# 六，call_once与使用场景