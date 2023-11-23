# 按位输出的小程序

```
#include <iostream>
#include <bitset>
#include <iomanip>
#include <string>

using namespace std;

template<class T>
void printbin(const T& x) {
	int n = sizeof(x) * 8;
	cout
		<< " [Type: "
		<< setw(8) << left << typeid(x).name()
		<< setw(6) << " Size:"
		<< setw(4) << left << n
		<< setw(6) << " Addr:"
		<< setw(18) << left << &x
		<< setw(7) << " Value:" 
		<< setw(8) << left << x
		<< " ] ";
	char* bit_arr = (char*)&x;
	for (int i = sizeof(x) - 1; i >= 0; --i) {
		char byte = bit_arr[i];
		for (int j = 0; j < 8; ++j) {
			char bit = (byte << j) & -128;
			if (bit == 0) cout << "0";
			else cout << "1";
		}
		cout << " ";
	}
	cout << endl;
}

int main() {
	float a = 0.25;
	cout << "a ";
	printbin(a);
	float b = 0.125;
	cout << "b ";
	printbin(b);
	float c = 0.625;
	cout << "c ";
	printbin(c);
	float d = 0.1875;
	cout << "d ";
	printbin(d);

	return 0;
}
```



# 输入流输出流判断

```
#include <iostream>
using namespace std;

#define default 1000;

void test01(){
	int num = default;
	cout << "Please input an integer, the default is 1000: " << endl;
	if (cin.peek() != '\n') {
		cin >> num;
		if(!cin.good())
			cout << "[!] Please input an integer!" << endl;
			num = default;
	}
	cout << "num = " << num << endl;
}

void test02(){
	int num = default;
	cout << "Please input an integer, the default is 1000: " << endl;
	while (cin.peek() != '\n') {
		cin >> num;
		if(!cin.good()){
			cout << "[!] Your input is not an integer! Do you want to input again? ( Y / N )";
			cin.clear();
			cin.ignore(4096,'\n');
			char retry;
			cin >> retry;
			cin.ignore(4096,'\n');
			if(retry == 'Y' || retry == 'y'){
				cout << "Please input an integer again!" << endl;
				continue;
			}
			else {
				num = default;
				break;
			}
		}
	}
	cout << "num = " << num << endl;
}

void test03(){
	int num = default;
	cout << "Please input an integer, the default is 1000: " << endl;
	while (cin.peek() != '\n') {
		cin >> num;
		if(!cin.good() || num < 0){
			cout << "[!] 你输入的值不是一个正整数！是否要重新输入？ ( Y / N )";
			cin.clear();
			cin.ignore(4096,'\n');
			char retry;
			cin >> retry;
			cin.ignore(4096,'\n');
			if(retry == 'Y' || retry == 'y'){
				cout << "请重新输入一个正整数：" << endl;
				continue;
			}
			else {
				num = default;
				break;
			}
		}
	}
	cout << "num = " << num << endl;
}

int main() {
//	test01();
	test03();
	return 0;
}
```



# 可变参数模板

```
#include <iostream>
using namespace std;

template<class ...Args>
void func(const Args&... args) {
	cout << "The quantity of args : " << sizeof ...(args) << endl;

	(cout << ... << args);
}

int main() {
	func(1,2,"hello",3,4,5,6);
	return 0;
}
```



# main函数的参数

```
#include <iostream>
using namespace std;

int main(int argc, char* argv[]) {
	for (int i = 0; i < argc; ++i) {
		cout << "第" << i << "个参数为：" << argv[i] << endl;
	}
	return 0;
}
```



# lambda，仿函数和容器

```
#include <iostream>
#include <set>
using namespace std;

class MyCompareClass {
public:
	bool operator() (const int a, const int b) const {
		cout << "仿函数调用: " << a << " & " << b << endl;
		return a > b;
	}
};

int main() {
	system("chcp 65001 > nul");
	//使用仿函数充当容器排序规则
	set<int, MyCompareClass> s1;
	for (int i = 1; i <= 5; ++i) {
		auto ret = s1.insert(i);
		if (ret.second) cout << "插入了元素: " << i << endl << endl;
	}
	for (auto x : s1) {
		cout << x << endl;
	}


	/*使用lambda表达式充当容器排序规则时，必须要将lambda表达式的类型当参数传入
	同时，容器在进行排序时，会调用传入准则函数的构造函数，而在C++中，lambda表达式没有构造函数
	这时就要同步地将lambda表达式传入容器的构造函数*/
	auto func = [](const int a, const int b){
		cout << "lambda表达式调用: " << a << " & " << b << endl;
		return a > b;
	};
	cout << typeid(func).name() << endl;	//lambda表达式的类型是不确定的临时类型
	set<int, decltype(func)> s2(func);
	for (int i = 1; i <= 5; ++i) {
		auto ret = s2.insert(i);
		if (ret.second) cout << "插入了元素: " << i << endl << endl;
	}
	for (auto x : s2) {
		cout << x << endl;
	}

	return 0;
}
```



# 线程

```
#include <iostream>
#include <thread>
#include <chrono>
using namespace std;

int func(int i) {
	thread::id t_id = this_thread::get_id();        // 获得自己的线程ID
	for (int j = 1; j <= 100; ++j) {
		//++i;
		cout << "子线程(" << t_id << ")的运行结果:" << i << endl;
		this_thread::sleep_for(chrono::milliseconds(150));        //休眠150毫秒
	}
	return 0;
}

int main() {
	int num = 0;
	cout << "主线程id：" << this_thread::get_id() << endl;
	thread t(func, num);
	// 获得其他线程的线程ID
	cout << "创建子线程成功，子线程id：" << t.get_id() << endl;

	t.join();
	return 0;
}
```



# 在类内部定义静态数组

```
class MyClass{
public:
	static constexpr const char* arr[] = {
		"主页",
		"学生管理",
		"教学计划",
		"成绩管理"
	};		// 必须带constexpr
};
```



# C++定义多维数组

一维数组可以使用

```
int* arr = new int[n];
```

其中n可以动态指定。

二维数组的定义方式：

```
int (*table)[n] = new int[m][n];
```

但m和n必须使用常量指定，确保**在编译阶段就能确定值**。

```
int main() {
    const int m = 6;
    const int n = 5;
    int (*table)[n] = new int[m][n];
    delete[] tabel;
}
```

但这种是不允许的，

```
int main() {
    int m = 6;
    int n = 5;
    int (*table)[n] = new int[m][n];
}
```

如果你需要，你可以先定义一个一维指针数组作为行，然后在定义很多一维数组作为列。但是这样你需要使用循环，且析构时也需要使用循环。
