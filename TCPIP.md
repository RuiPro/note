![](https://subingwen.cn/linux/socket/tcp.jpg)

tcp的三次握手和四次挥手的状态变化：

![](https://subingwen.cn/linux/tcp-status/20141015155713390.png)

每一次状态被设置之后都是不可逆的，也无法重复设置状态。因此例如`connect()`、`listen()`这样的函数只能调用一次；每个套接字也必须以`close()`关闭。

服务端

```
#include <iostream>
#include <arpa/inet.h>
#include <unistd.h>
using namespace std;

int main() {
	cout << "I am server." << endl;

	// 1.创建用于监听的套接字
	// socket()函数：参数1指定要通过什么地址通信，参数2指定通信形式，参数3指定通信类型，返回一个用于监听的套接字
	// 这里参数1为IPv4，参数2为流式传输，参数3选择TCP
	int fd = socket(AF_INET, SOCK_STREAM, 0);
	if (fd == -1) {
		printf("创建监听套接字失败！\n");
		return -1;
	}

	// 2.绑定IP和端口
	// 先创建一个用于保存要绑定的IP和端口的结构体sockaddr_in，再进行初始化
	sockaddr_in saddr;
	saddr.sin_family = AF_INET;			// 表示IP和端口是IPv4类型
	saddr.sin_port = htons(12345);		// 绑定端口12345，由于所有socketAPI的非字符型数据都要转成大端，所以转成大端再传入
	saddr.sin_addr.s_addr = INADDR_ANY;	// 和127.0.0.1这样的易读性IP地址不同，在API函数里面IP是一个4字节大小的整形
										// IP和端口结构体内部嵌套了一个地址结构体，所以需要访问sin_addr里面的s_addr成员
										// INADDR_ANY的值为0.0.0.0，表示所有端口，因为值为0，所以不需要转换成大端
	// 把初始化过后的携带地址信息的结构体和套接字进行绑定
	// bind()函数：参数1指定要绑定的监听套接字，参数2传入保存IP和端口的结构体地址，参数3传入参数2结构体的大小，返回值为状态码
	// 由于bind函数只接受sockaddr类型的指针，把sockaddr_in类型的地址强转成sockaddr类型指针传入
	if (bind(fd, (sockaddr*)&saddr, sizeof(saddr)) == -1) {
		printf("绑定地址失败！\n");
		return -1;
	}

	// 3.设置监听
	// listen()函数：参数1指定要绑定的监听套接字，参数2设置同时能检测的最大连接数，最大值为128，返回值为状态码
	if (listen(fd, 128) == -1) {
		printf("设置监听失败！\n");
		return -1;
	}

	// 4.阻塞并等待客户端连接
	// 如果有客户端连接，那么还需要创建一个用于和对应客户端通信的套接字
	// 那么我们还需要创建一个用于保存客户端IP和端口的结构体sockaddr_in
	sockaddr_in caddr;
	// 还需要一个传入+传出的参数保存结构体的大小，用于辅助函数对结构体进行操作
	socklen_t caddr_size = sizeof(caddr);
	// 等待连接
	// accept()函数：参数1指定监听套接字，参数2指定要保存客户端地址信息的结构体，参数3需要传入参数2结构体的大小
	// 函数返回之后，参数3将传出参数2结构体的实际大小，返回值为用于和对应客户端通信的套接字
	int cfd = accept(fd, (sockaddr*)&caddr, &caddr_size);
	if (cfd == -1) {
		printf("创建和客户端进行通信套接字失败！\n");
		return -1;
	}

	// 4.5.如果有客户端连接了，输出一下客户端的地址信息
	// 因为IP在结构体里面是以大端的4字节整形形式保存的，要输出给人能直观看懂的IP地址还需要转成小端再转换成字符串
	char cIP[32];
	inet_ntop(AF_INET, &caddr.sin_addr.s_addr, cIP, sizeof(cIP));
	// 把端口也由大端转成小端
	short cPort = ntohs(caddr.sin_port);
	// 输出
	printf("客户端IP：%s，客户端端口：%d\n", cIP, cPort);

	// 5.与客户端进行通信，直到某一方断开连接
	// 设置接受数据的容器
	char recbuffer[1024];
	// 设置要回复的内容
	char retbuffer[] = "Server received!";
	while (true) {
		// recv()函数：参数1指定通信套接字，参数2指定接受数据的容器地址，参数3需要传入需要接收多少数据，参数4是flag，设为0即可
		// 返回值：大于0表示接受的数据量，等于0表示对方断开了连接，小于0表示错误
		int recvSize = recv(cfd, recbuffer, sizeof(recbuffer), 0);
		if (recvSize > 0) {
			printf("收到客户端的信息：%s\n", recbuffer);
			// 给客户端回话
			// send()函数和recv()函数的参数属性很相似，参数2指定要发送的数据的地址
			send(cfd, retbuffer, sizeof(retbuffer), 0);
		}
		else if (recvSize == 0) {
			printf("客户端断开了连接。\n");
			break;
		}
		else if (recvSize == -1) {
			printf("接收数据失败！\n");
			break;
		}
	}

	// 6.通信结束，关闭套接字
	close(fd);
	close(cfd);

	return 0;
}
```



客户端

```
#include <iostream>
#include <arpa/inet.h>
#include <string>
#include <unistd.h>
using namespace std;

int main() {
	cout << "I am client." << endl;
	// 1.创建用于和服务端通信的套接字
	// socket()函数：参数1指定要通过什么地址通信，参数2指定通信形式，参数3指定通信类型，返回一个用于监听的套接字
	// 这里参数1为IPv4，参数2为流式传输，参数3选择TCP
	int fd = socket(AF_INET, SOCK_STREAM, 0);
	if (fd == -1) {
		printf("创建通信套接字失败！\n");
		return -1;
	}

	// 2.设置要连接的目标服务器
	// 先创建一个用于保存要连接的服务器的IP和端口的结构体sockaddr_in，再进行初始化
	sockaddr_in saddr;
	saddr.sin_family = AF_INET;			// 表示IP和端口是IPv4类型
	saddr.sin_port = htons(12345);		// 绑定端口12345，由于所有socketAPI的非字符型数据都要转成大端，所以转成大端再传入
	string sIP;
	printf("请输入要连接的服务器的地址...\n");
	cin >> sIP;
	// 要把IP地址从小端转成大端
	inet_pton(AF_INET, sIP.c_str(), &saddr.sin_addr.s_addr);

	// 用初始化过后的服务器地址信息进行连接
	// connect()函数：参数1指定要绑定的通信套接字，参数2传入要连接的服务器的IP和端口的结构体地址，参数3传入参数2结构体的大小，返回值为状态码
	// 由于connect函数只接受sockaddr类型的指针，把sockaddr_in类型的地址强转成sockaddr类型指针传入	
	if (connect(fd, (sockaddr*)&saddr, sizeof(saddr)) == -1) {
		printf("连接失败！\n");
		printf("这可能是IP地址未正确输入造成的\n");
		return -1;
	}

	// 3.与服务端进行通信，直到某一方断开连接
	// 设置要与服务器进行通信的内容
	string buffer;
	printf("请输入要给服务器发送的内容...\n");
	// 清空缓冲区
	cin.clear();
	cin.ignore(20480, '\n');
	getline(cin, buffer);
	printf("即将给服务器发送数据：%s\n", buffer.c_str());
	// 设置接受数据的容器
	char recbuffer[1024] = { 0 };

	// 给服务端发送数据
	// send()函数和recv()函数的参数属性很相似，参数2指定要发送的数据的地址
	// 因为字符串还有末位的'\0'符号，所以计算长度还要加1
	send(fd, buffer.c_str(), buffer.size(), 0);

	// 接收来自服务端的数据
	ssize_t recvSize = recv(fd, recbuffer, sizeof(recbuffer), 0);
	if (recvSize > 0) {
		printf("收到客户端的信息：%s\n", recbuffer);
	}
	else if (recvSize == 0) {
		printf("服务器断开了连接。\n");
	}
	else if (recvSize == -1) {
		printf("接收数据失败！\n");
	}

	// 4.关闭套接字
	close(fd);

	return 0;
}
```



为什么要使用大小端？

1. 因为Socket API内部都是按照大端(网络字节序)来对数据进行解析的，如果传入了小端，就会造成错误
2. 如果服务端和客户端都是我们写的，而且操作系统都用的同样的字节序来保存数据，我们当然可以不用担心这个问题；但如果我们不了解对方的操作系统环境或者说处理数据的逻辑，就很容易造成字节序不匹配的情况。因此，IT界明文规定Socket通信使用网络字节序，即大端，数据接收方不管自己收到什么样的数据一律当作大端处理，发送方也要自觉地把数据转成大端再发送。

但是这样我们也很难确定字节序是否会发送冲突，因为大多数主机的字节序是不可见的。解决方法之一就是避免使用除字符串以外的其他形式传值。比如我们要传输一个int num = 123456，那么我们应该把它保存为一个字符串char arr[6] = {'1','2','3','4','5','6'}

基于线程池和做了TCP防粘包措施之后的

防TCP粘包有3个做法：

1. 发送定长的数据。比如规定每个数据包的长度为1024个字节，不到1024个字节补0。这种做法的缺陷是传输效率低，以及对传输的内容有长度限制。
2. 定义数据格式。约定一段数据的开头和结尾的标志。这种做法典型的案例是HTTP，但这种做法需要对传输的正文中出现的标志做转义处理，客户端也要解析转义，比较麻烦。
3. 定义一个数据头，保存要发送的包长度。这个数据头一般是定长的，这样可以很容易解析出数据。这种做法在传输的数据多而长度小的情况下会占用大量带宽。

下面的例子是定义数据头：

服务端：

```
#include <iostream>
#include <arpa/inet.h>
#include <unistd.h>
#include "ThreadPool.hpp"
using namespace std;

// 与客户端对接的函数
void Docking(int cfd, sockaddr_in* caddr);

// 接收数据函数
ssize_t Receive(int& cfd, char* buffer, ssize_t bufferSize);

int main() {
	cout << "I am server." << endl;

	int fd = socket(AF_INET, SOCK_STREAM, 0);
	if (fd == -1) {
		printf("创建监听套接字失败！\n");
		return -1;
	}

	// 先创建一个用于保存要绑定的IP和端口的结构体sockaddr_in，再进行初始化
	sockaddr_in saddr;
	saddr.sin_family = AF_INET;
	saddr.sin_port = htons(12345);
	saddr.sin_addr.s_addr = INADDR_ANY;

	// 设置端口复用
	int opt = 1;
	setsockopt(fd, SOL_SOCKET, SO_REUSEADDR, &opt, sizeof(opt));

	// 绑定端口
	if (bind(fd, (sockaddr*)&saddr, sizeof(saddr)) == -1) {
		printf("绑定地址失败！\n");
		return -1;
	}

	if (listen(fd, 128) == -1) {
		printf("设置监听失败！\n");
		return -1;
	}

	// 创建线程池
	ThreadPool threadPool(1, 32, 128);
	threadPool.Start();

	while (true) {
		// 如果有客户端连接，那么还需要创建一个用于和对应客户端通信的套接字
		// 那么我们还需要创建一个用于保存客户端IP和端口的结构体sockaddr_in
		sockaddr_in* caddr = new sockaddr_in;
		// 还需要一个传入+传出的参数保存结构体的大小，用于辅助函数对结构体进行操作
		socklen_t caddr_size = sizeof(*caddr);
		// 等待连接
		int cfd = accept(fd, (sockaddr*)caddr, &caddr_size);
		if (cfd == -1) {
			printf("创建和客户端进行通信套接字失败！\n");
			continue;
		}
		threadPool.AddTask(&Docking, cfd, caddr);
	}

	// 6.通信结束，关闭套接字
	close(fd);

	return 0;
}

void Docking(int cfd, sockaddr_in* caddr) {
	// 如果有客户端连接了，输出一下客户端的地址信息
	// 因为IP在结构体里面是以大端的4字节整形形式保存的，要输出给人能直观看懂的IP地址还需要转成小端再转换成字符串
	char cIP[32];
	inet_ntop(AF_INET, &caddr->sin_addr.s_addr, cIP, sizeof(cIP));
	// 把端口也由大端转成小端
	unsigned short cPort = ntohs(caddr->sin_port);
	printf("客户端IP：%s，客户端端口：%d\n", cIP, cPort);

	// 一直接收数据，直到客户端断开
	while (true) {
		// 协议：包格式为@[unsigned int][8个字符]@data
		// 其中unsigned int保存包大小（包括unsigned int和两个@在内），用字符串传递，data是数据
		// 接收客户端发来的前10个字节的有效数据量
		unsigned int allSize = 0;
		char allSizeArr[10] = { 0 };
		// MSG_WAITALL标志：必须等到缓冲区中有足够数量的数据才进行读取
		int headSize = recv(cfd, allSizeArr, 10, MSG_WAITALL);
		if (headSize == 0) {
			printf("客户端%s:%d断开了连接...\n", cIP, cPort);
			break;
		}
		if (headSize < 0) {
			printf("接收客户端%s:%d的报头失败！\n", cIP, cPort);
			break;
		}
		if (headSize != 10 || allSizeArr[0] != '@' || allSizeArr[9] != '@') {
			printf("客户端%s:%d未按照协议传输数据！\n", cIP, cPort);
			break;
		}
		// char[1~8]转为unsigned int
		for (int i = 1; i < 8; ++i) {
			allSize += allSizeArr[i] - 48;
			allSize *= 10;
		}
		allSize += allSizeArr[8] - 48;
		printf("客户端%s:%d要发送的数据量为%d字节\n", cIP, cPort, allSize - headSize);

		// 定义接收数据的容器
		char* data = new char[allSize - headSize] {0};
		// 接收数据
		ssize_t recvSize = Receive(cfd, data, allSize - headSize);
		printf("客户端%s:%d实际发送的数据量为%d字节\n", cIP, cPort, recvSize);
		for (int i = 0; i < recvSize; ++i) {
			printf("%c", data[i]);
		}
		printf("\n");
		if (recvSize != allSize - headSize) {
			printf("客户端%s:%d未能完整地传输一条数据！这是本机接收失败或客户端中途断开连接导致的。\n", cIP, cPort);
		}
		delete[] data;
	}

	// 6.通信结束，关闭套接字\销毁资源
	close(cfd);
	delete caddr;
}

ssize_t Receive(int& cfd, char* buffer, ssize_t bufferSize) {
	if (bufferSize <= 0) {
		return -1;
	}
	// 循环接收数据，直到传输完毕或客户端断开连接
	// 保存接受的有效数据量
	ssize_t hasRecv = 0;
	char* ptr = buffer;
	while (hasRecv < bufferSize) {
		auto recvSize = recv(cfd, ptr, bufferSize - hasRecv, 0);
		if (recvSize > 0) {
			// 指向data数组的指针往后移动到数据末尾，下次接收的数据直接就在末尾接上
			ptr += recvSize;
			hasRecv += recvSize;
			continue;
		}
		else {
			break;
		}
	}
	return hasRecv;
}
```

客户端：

```
#include <iostream>
#include <arpa/inet.h>
#include <string>
#include <string.h>
#include <unistd.h>
using namespace std;

ssize_t Send(int& fd, char* msg, ssize_t msgLen);

int main() {
	cout << "I am client." << endl;
	// 1.创建用于和服务端通信的套接字
	// socket()函数：参数1指定要通过什么地址通信，参数2指定通信形式，参数3指定通信类型，返回一个用于监听的套接字
	// 这里参数1为IPv4，参数2为流式传输，参数3选择TCP
	int fd = socket(AF_INET, SOCK_STREAM, 0);
	if (fd == -1) {
		printf("创建通信套接字失败！\n");
		return -1;
	}

	// 2.设置要连接的目标服务器
	// 先创建一个用于保存要连接的服务器的IP和端口的结构体sockaddr_in，再进行初始化
	sockaddr_in saddr;
	saddr.sin_family = AF_INET;				// 表示IP和端口是IPv4类型
	saddr.sin_port = htons(12345);			// 绑定端口12345，由于所有socketAPI的非字符型数据都要转成大端，所以转成大端再传入
	string sIP = "106.13.214.22";
	//printf("请输入要连接的服务器的地址...\n");
	//cin >> sIP;
	// 要把IP地址从小端转成大端
	inet_pton(AF_INET, sIP.c_str(), &saddr.sin_addr.s_addr);

	// 用初始化过后的服务器地址信息进行连接
	// connect()函数：参数1指定要绑定的通信套接字，参数2传入要连接的服务器的IP和端口的结构体地址，参数3传入参数2结构体的大小，返回值为状态码
	// 由于connect函数只接受sockaddr类型的指针，把sockaddr_in类型的地址强转成sockaddr类型指针传入
	if (connect(fd, (sockaddr*)&saddr, sizeof(saddr)) == -1) {
		printf("连接失败！\n");
		return -1;
	}

	// 3.与服务端进行通信，直到某一方断开连接
	// 设置要与服务器进行通信的内容

	// char数组既可以拿来当字节数组存数据用，也可以拿来存字符串
	// 如果是字符串，必须手动添加\0，否则编译器会自动加\0而导致计算数据长度错误
	// 如果是存数据，可以不用加\0，但是要注意编译器是否会自动加\0
	char data[] = "01000.11544845as45f4a5sf45as4f54a5sf";
	// 设置接受数据的容器
	char recBuffer[1024] = { 0 };

	// 发送数据
	ssize_t len = Send(fd, data, sizeof(data));

	// 4.关闭套接字
	close(fd);

	return 0;
}

ssize_t Send(int& fd, char* msg, ssize_t msgLen) {
	if (msg == nullptr || msgLen <= 0 || fd <= 0) {
		return -1;
	}
	// 协议：包格式为@[unsigned int][8个字符]@data
	// 其中unsigned int保存包大小（包括unsigned int和两个@在内），用字符串传递，data是数据
	// ①先计算给服务端发送要发送的数据量
	unsigned int allSize = msgLen + 10;
	printf("准备给服务器发送%d字节的数据\n", allSize - 10);

	// 组合包
	char* package = new char[allSize] {0};
	// 报头
	package[0] = '@';
	package[9] = '@';
	int temp_allSize = allSize;
	for (int i = 8; i > 0; --i) {
		package[i] = (temp_allSize % 10) + 48;
		temp_allSize /= 10;
	}
	// 数据
	memcpy(package + 10, msg, msgLen);

	// 发送
	ssize_t count = allSize;
	while (count > 0) {
		ssize_t len = send(fd, package, allSize, 0);
		if (len == -1) {
			return -1;
		}
		else if (len == 0) {
			return 0;
		}
		package += len;
		count -= len;
	}
	return allSize;
}
```

