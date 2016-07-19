---
title: "C++ 中的 Lambda 表达式"
comments: true
---
前些天买了本《程序设计语言理论》，看了简介，Lambda 演算贯穿整个理论，尤其在函数式语言中具有重要作用。C++11 中也加入了 Lambda 表达式，下面做个总结。

<p style="text-align:center"><img src="http://img.blog.csdn.net/20140507213330515?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvSnVzdG1lMA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" align="middle" width="204" height="300" alt=""></p>
<br />

<!--
![](http://img.blog.csdn.net/20140507213330515?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvSnVzdG1lMA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
-->

# 1. 一个简单的例子
一个简单的 Lambda 表达式如下：

{% highlight cpp %}
[] {};
{% endhighlight %}

这就定义了一个<span style="color:rgb(255,0,0)">**对象**</span>，这个对象匿名，再强调一下，Lambda 表达式是对象，不是类型。本例中，该对象的类型是 'anonymous-namespace'::\<lambda0>，这是编译器给它设的一个类型名。

# 2. 实现
一般地，编译器实现 Lambda 表达式时，将其转化为函数对象（仿函数 functor）。比如上面的例子将转化为

{% highlight cpp %}
struct A {
public:
	void operator()() const {}
};

A();
{% endhighlight %}

其中 A() 是一个函数对象，而 A 是类型，这些要明确区分。

# 3. 状态
仿函数和 Lambda 表达式是可以有状态的，也就是说它们可以含数据成员，而函数指针就无状态可言，比如下面这种情况就无法用函数指针，只能用仿函数或 Lambda 表达式。

{% highlight cpp %}
/********************************************************************
created:	2014/05/07 22:31
filename:	main.cpp
author:		Justme0 (http://blog.csdn.net/justme0)

purpose:	涉及状态时，只能用仿函数或 Lambda 表达式，无法用函数指针
*********************************************************************/

#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

/*
** 输出 vec 中大于 standard 的元素
*/
void print_use_lambda(const vector<int> &vec, int standard) {
	for_each(vec.begin(), vec.end(), [standard](int elem) {
		if (standard < elem) {
			cout << elem << ' ';
		}
	});
}

struct Printor {
private:
	int m_standard;

public:
	Printor(int standard) : m_standard(standard) {}

	void operator()(int elem) const {
		if (m_standard < elem) {
			cout << elem << ' ';
		}
	}
};

/*
** 输出 vec 中大于 standard 的元素
*/
void print_use_functor(const vector<int> &vec, int standard) {
	for_each(vec.begin(), vec.end(), Printor(standard));
}

int main(int argc, char **argv) {
	int arr[] = {5, 2, 1, 4, 5, 6, 7, 2};
	int size = sizeof arr / sizeof *arr;
	vector<int> vec(arr, arr + size);

	cout << "需要输出大于几的数？" << endl;
	int standard;
	cin >> standard;

	print_use_functor(vec, standard);
	cout << endl;
	print_use_lambda(vec, standard);
	cout << endl;

	system("PAUSE");
	return 0;
}
{% endhighlight %}

程序中调用 for_each 时，只输出大于某个数的元素，而这个数事先不知道，这个数代表了仿函数的状态，而若用函数指针则无法获得这个数（状态），注意 for_each 的第三个参数是一元的。

# 4. 捕捉列表中的值传递与引用传递
按值方式传递捕捉列表与按引用方式传递列表不同，以下面的程序为例：

{% highlight cpp %}
/********************************************************************
	created:	2014/05/06 21:01
	filename:	main2.cpp
	author:		Justme0 (http://blog.csdn.net/justme0)

	purpose:	Lambda 的捕捉列表
*********************************************************************/

#include <iostream>
using namespace std;

int main(int argc, char **argv) {
	int i = 0;
	auto by_val_lambda = [i] {
		return i;
	};
	auto by_ref_lambda = [&i] {
		return i;
	};

	cout << by_val_lambda() << endl;	// 0
	cout << by_ref_lambda() << endl;	// 0

	++i;

	cout << by_val_lambda() << endl;	// 0
	cout << by_ref_lambda() << endl;	// 1

	system("PAUSE");
	return 0;
}
{% endhighlight %}

要解释这个问题，把它们转化为仿函数（编译器的实现）就一目了然了：

{% highlight cpp %}
/********************************************************************
created:	2014/05/06 20:39
filename:	main.cpp
author:		Justme0 (http://blog.csdn.net/justme0)

purpose:	用 Functor 代替 Lambda
*********************************************************************/

#include <iostream>
using namespace std;

struct Value {
private:
	int m_i;

public:
	Value(int i) : m_i(i) {}

	int operator()() const {
		return m_i;
	}
};

struct Reference {
private:
	int &m_i;

public:
	Reference(int &i) : m_i(i) {}	// 必须用初始化列表

	int operator()() const {
		return m_i;
	}
};

struct Pointer {
private:
	int *m_pi;

public:
	Pointer(int *pi) : m_pi(pi) {}

	int operator()() const {
		return *m_pi;
	}
};

int main(int argc, char **argv) {
	int i = 0;
	auto by_val_functor = Value(i);
	auto by_ref_functor = Reference(i);
	auto by_ptr_functor = Pointer(&i);

	cout << by_val_functor() << endl;	// 0
	cout << by_ref_functor() << endl;	// 0
	cout << by_ptr_functor() << endl;	// 0

	++i;

	cout << by_val_functor() << endl;	// 0
	cout << by_ref_functor() << endl;	// 1
	cout << by_ptr_functor() << endl;	// 1

	system("PAUSE");
	return 0;
}
{% endhighlight %}

值传递时 Lambda 保存了一个副本，与形参就无关了。引用传递时，Lambda 保存的是形参的引用，所以后面改变形参 ++i 时，Lambda 内保存的引用指向的 i 变了（同一个 i），注意指针与引用类似，用指针来解释引用再好不过了。

# 5. 捕捉类别的限制
Lambda 捕捉列表仅能捕捉父作用域的自动变量。

标准是这么定义的，但有的编译器不遵守，笔者对具体的规则也存有疑惑。但总的来说，Lambda 的思想是局限于一个局部作用域中使用，若需全局共享，则用函数指针（无状态）或仿函数（有状态）。

参考资料：<a href="https://www.ibm.com/developerworks/community/groups/service/html/communityview?communityUuid=12bb75c9-dfec-42f5-8b55-b669cc56ad76" target="_blank">《深入理解 C++11》</a>

(End)
