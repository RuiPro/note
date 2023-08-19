# libcurl

本章节记录libcurl的使用。

libcurl是一个跨平台的轻量级的网络请求库，它是由C语言构建的。你可以用它进行多种网络协议的传输，比如http/https、ftp。之所以是网络请求库而不是网络库，是因为它只能用于主动发送请求而不能创建服务，因此它只能被用于设计成客户端。



## 准备你的libcurl

1. 首先，你需要把curl的仓库克隆下来。

   ```
   git clone github.com/curl/curl.git
   ```

   或者你可以直接下载zip文件再解压。

2. 开始编译库文件。

   windows直接使用projects目录下的sln文件在visual studio下编译即可。Linux需要执行以下命令

   ```
   ./configure --prefix=目标目录 --可选参数
   ./configure --with-ssl=/usr/bin/openssl --with-libssl-prefix=/usr/lib/x86_64-linux-gnu/
   make
   make install
   ```

   然后在[目标目录]下就出现了库文件和对应的头文件，就可以用于开发了。

   你的libcurl第一次可能不会成功，你需要根据提示，在`./configure`后添加可选参数，比如提示你确实了SSL，那么你需要安装openssl或者忽略SSL。这需要根据你的实际情况来定，忽略SSL意味着不能使用HTTP1.x以外的HTTP协议。但是大多数情况下，HTTP1.x已经够用了。centos安装openssl的命令为:

   ```
   yum install openssl-devel
   ```

   

3. 在你的项目中添加库文件和头文件

   比如Cmake就可以使用

   ```
   add_executable(项目目标 源文件)
   target_link_libraries(项目目标 libcurl.so文件的路径)
   ```

   来添加库文件，然后在源文件中添加头文件即可。注意，`target_link_libraries`要在`add_executable`之后。

   此外，Cmake中的库文件通常需要绝对路径，你可能还需要使用项目根目录变量`${PROJECT_SOURCE_DIR}`来找到你的库文件。

   在Cmake中添加头文件，你可以把整个头文件文件夹拷贝到你的项目里，直接使用以下方式：

   ```
   #include "curl/curl.h"
   ```

   或者，你可以把头文件拷贝到某个特定的文件夹下，比如当多个项目都使用了此头文件时，你可以把它放在/opt/env/include下。

   当然，你还需要做一些额外的配置，gcc/g++只会寻找/usr/include和/usr/local/include下的头文件。想要让gcc找到你的头文件，你需要对gcc添加`-L<path/to/include>`参数。或者，你可以在CMakeLists中添加：

   ```
   include_directories(path/to/include)
   ```

   根据curl开发团队的文档，你只需要包含curl/curl.h这一个头文件就包含了所有curl的api

4. 开始享用curl吧！



## libcurl食用指南

在开始之前，如果你是一位C++程序员，你应该在使用libcurl前注意以下几点：

libcurl是用C语言构建的。因此它不知道std::string类型，这意味着如果需要给libcurl的api传入使用string，需要使用c_str()方法把它转成char*类型再传入。其次，它不知道类的概念，这意味着你不能使用类的非静态成员函数做回调。

### 开始食用

curl的食用步骤为：创建curl句柄——配置curl句柄——开始使用curl进行协议传输数据。

下面的小Demo揭示了此三个过程。

```
#include <iostream>
#include <string>
#include "curl/curl.h"

using namespace std;

// 回调函数的格式为：size_t callback(char* ptr, size_t size, size_t nmemb, void* userdata)
// 其中ptr指向服务器返回的数据
// size表示每个数据块的大小，比如4就是每块数据为4字节
// nmemb表示数据块的数量，比如123就是分成了123块
// userdata用于传递用户在CURLOPT_WRITEDATA属性中自定义的数据
size_t callback(char* ptr, size_t size, size_t nmemb, void* userdata) {
	((std::string*)userdata)->append(ptr, size * nmemb);
	return size * nmemb;
}

int main() {
	// 1.创建curl句柄
	CURL* handle = curl_easy_init();
	if (handle == nullptr) {
		cout << "CURL init faild" << endl;
		return 0;
	}

	// 2.配置curl句柄属性
	curl_easy_setopt(handle, CURLOPT_URL, "http://www.baidu.com");	// 设置URL
	curl_easy_setopt(handle, CURLOPT_CUSTOMREQUEST, "GET");				// 设置请求方法
	//curl_slist* pList = NULL;											// 设置自定义字段
	//pList = curl_slist_append(pList, "Accept-Encoding:gzip");
	//pList = curl_slist_append(pList, "Accept-Language:zh-CN,zh,en;q=0.8");
	//curl_easy_setopt(handle, CURLOPT_HTTPHEADER, pList);

	std::string reply_data;												// 接收数据的容器
	curl_easy_setopt(handle, CURLOPT_WRITEDATA, &reply_data);			// 设置得到请求结果后的回调函数的最后一个参数
	curl_easy_setopt(handle, CURLOPT_WRITEFUNCTION, callback);			// 得到请求结果后的回调函数

	// 3.开始使用curl进行协议传输数据
	CURLcode ret_code = curl_easy_perform(handle);						 // 执行请求
	if (ret_code == CURLE_OK) {
		long reply_code;
		curl_easy_getinfo(handle, CURLINFO_RESPONSE_CODE, &reply_code);
		//正确响应后，输出数据
		if (reply_code == 200 || reply_code == 201) {
			std::cout << "code: " << reply_code << std::endl;
			std::cout << "data: " << reply_data << std::endl;
		}
	}
	// 4.释放资源
	curl_easy_cleanup(handle);
	return 0;
}
```



### 创建curl句柄

### 配置curl句柄

[所有选项 - 所有卷曲 (curl.dev)](https://everything.curl.dev/libcurl/options/all)

https://blog.csdn.net/yupu56/article/details/43566085

### 开始传输数据





# libevent

本章节记录libevent的使用。

libevent是一个跨平台的轻量级的事件通知库，它是由C语言构建的。你可以用它进行一些事件驱动的模型。它参照了反应堆模型，内部实现了io多路复用。比如监听套接字时，你可以把套接字包装好再放到libevent里面，并设置回调函数。当套接字可以接收/发送/发生错误时，对应的回调函数会被调用，这就是事件驱动。事件驱动比阻塞监听要好得多，如果使用阻塞监听，你需要一直判断套接字是否可以接收/发送/发生错误。



## 准备你的libevent

libevent的准备和libcurl很类似。你可以使用下面的命令克隆源码：

```
git clone https://github.com/libevent/libevent.git
```

或者你可以下载压缩包。

在linux下编译：

```
$ mkdir build && cd build
$ cmake ..     # Default to Unix Makefiles.
$ make
$ make verify  # (可选)
```

在windows下编译：

```
$ md build && cd build
$ cmake -G "Visual Studio 10" ..   # Or use any generator you want to use. Run cmake --help for a list
$ cmake --build . --config Release # Or "start libevent.sln" and build with menu in Visual Studio.
```

然后在build/lib目录下，就生成了库文件。

和libcurl不同，你需要把bulid/include和源码下的include中的内容合并在一起才是完整的头文件。



此外，和libcurl不同，libevent的头文件里，包含其他头文件的格式全都是`#include <header.h>`，这意味着你必须把这个文件放到/usr/include或/usr/local/include下，或者在Cmake中使用include_directories函数定向头文件目录，而不能直接把头文件放到项目中使用。



## 开始食用

先来介绍一下libevent中两个重要的结构体：event_base和event

- event_base是用来监听的一个event集合，就像select和poll、epoll需要一个socket数组/树来进行监听
- event是要监听的事件，实际上内部封装了文件描述符或信号



```
#include <iostream>
#include <string>
#include <event.h>
#include <evhttp.h>

using namespace std;

void http_request_cb(struct evhttp_request* req, void* arg) {
	if (evhttp_request_get_command(req) == EVHTTP_REQ_GET) {
		evbuffer* buffer = evbuffer_new();
		evbuffer_add_printf(buffer, "hello");
		evhttp_send_reply(req, 200, "OK", buffer);
	}
}

int main() {
	cout << EVENT__VERSION << endl;

	// libevent中两个重要的结构体：event_base和event
	// event_base是用来监听的一个event集合，就像select和poll、epoll需要一个socket数组/树来进行监听
	// event是要监听的事件，实际上内部封装了文件描述符或信号

	// 创建监听集合
	event_base* ev_base = event_base_new();
	evhttp* http = evhttp_new(ev_base);
	evhttp_set_cb(http, "/", http_request_cb, NULL);
	evhttp_bind_socket(http, "0.0.0.0", 12345);
	event_base_dispatch(ev_base);

	return 0;
}
```

