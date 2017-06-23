---
title: 'C++11多线程std::thread的使用'
date: '2014/10/2 14:00:00'
tags:
  - c++
abbrlink: 52fca7b5
---

cocos2d-x 3.0移除了`pthread`的支持，使用c++11的`std::thread`下面介绍以下简单用法  

<!-- more -->
代码需包含头文件`<thread>`

## 创建

	...
	std::thread t1(&HelloWold::myThread,this,1,2)；//创建一个分支线程，回调到myThread函数里
	...

	void HelloWorld::myThread(int a,int b)
	{
	}


## join()

	t1.join();
等待子线程myThread执行完之后，主线程才会继续执行下去，此时主线程会释放掉执行完后的子线程资源。
## detach()

	std::thread t1(&HelloWold::myThread,this,1,2)；
	t1.detache();
不等待子线程，将子线从主线程里分离，子线程执行完成后会释放掉自己的资源。分离后的子线程，主线程将对它没有控制权了。
## Sleep()

	Sleep(100)

## 利用互斥对象mutex同步数据
### 初始化  

	std::mutext mutex;//线程互斥对象

### 使用

	mutex.lock();//加锁
	//...
	mutex.unlock();//解锁

### 注意

lock之后一定要unlock，特别需要注意可能出现的异常或者线程直接退出
