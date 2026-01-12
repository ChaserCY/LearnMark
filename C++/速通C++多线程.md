
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