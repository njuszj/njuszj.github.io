---
layout: post
title:  C++防止int32溢出--以反转数字为例
date:   2019-05-15 19:49:00 +0800
categories: note
tag: C++
---

* content
{:toc}



### int32溢出
 + 在C++等语言中int类型的整数占4个字节（一般而言），而一个字节是8bit，所以int型可表示32位的整数，又因为int可以表示负数，所以int的范围是$-2^{31}$~$2^{31}$-1
 + 如果用数字表示，就是：-2147483648 ~ 2147483647
+ 而在C++里可用关键字**INT_MAX**和**INT_MIN**表示上述的int上限和int下限
### 溢出的后果
+ 对于int型的溢出，编译器可能会依二进制计算结果根据补码的规则得到一个错误的结果
+ 可能造成缓冲区溢出、未知的逻辑错误

### 防止int32溢出
> **错误的思路：判断一个int是否大于INT_MAX或小于INT_MIN**

 如果int已经溢出，编译器要么会报错，要么会处理这个数字使其处于int允许的范围内(当然这不是我们想要的数字)，所以这样的比较没有意义，无法判断是否溢出。
> **正确的思路：在无法判断一个操作是否会造成溢出时，在操作之前加一个判断语句。判断什么呢？打个比方，我们要进行 a+b 的操作(a,b>0)，但我们无法判断 a+b 是否会溢出，正如前面所说，我们不能通过 a+b>INT_MAX 这种方式去判断，但是可以转换一下思路，判断 a>INT_MAX-b 是否为真，若为真，a+b会溢出，否则不会**
 
 可能面对具体问题的时候需要一点微调和补充，但基本的思路就是这样，万变不离其宗。

### 示例
 这里给出一个程序用作示例，这是一个很经典的问题--将一个数字反转，转换很简单，现在考虑的是如何防止倒置过程中发生溢出。规定如果发生溢出，返回0。
 ```cpp
 int reverse(int x) {
		int ans = 0;
		int temp = 0;
		int r = 0;
		if (x >= 0)
			while (x) {
				r = x % 10;
				if (INT_MAX / 10 < temp) return 0;
				temp = ans * 10;
				if (INT_MAX - r < temp) return 0;
				ans = temp + r;
				temp = ans;
				x /= 10;
			}
		else 
			while (x) {
					r = x % 10;
					if (INT_MIN / 10 > temp) return 0;
					temp = ans * 10;
					if (INT_MIN - r > temp) return 0;
					ans = temp + r;
					x /= 10;
					temp = ans;	
		}
		return ans;
	}
 ```

 >如有任何问题，欢迎与我交流
