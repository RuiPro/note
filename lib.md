# libcurl

本章节记录libcurl的使用。

libcurl是一个跨平台的轻量级的网络请求库，它是由C语言构建的。你可以用它进行多种网络协议的传输，比如http/https、ftp。之所以是网络请求库而不是网络库，是因为它只能用于主动发送请求而不能创建服务，因此它只能被用于设计成客户端。



## 准备你的libcurl

1. 首先，你需要把curl的仓库克隆下来。仓库一般都是最新的DEV开发版，你也可以选择release

   ```
   git clone github.com/curl/curl.git
   ```

   或者你可以直接下载zip文件再解压。

2. 准备openssl

   到openssl的官方仓库克隆仓库

   ```
   git clone git://git.openssl.org/openssl.git
   git clone https://github.com/openssl/openssl.git
   ```

   然后

   ```
   ./Configure --prefix=/usr/local/ssl --openssldir=/usr/local/ssl
   make
   make install
   ```

   

3. 开始编译库文件。

   windows直接使用projects目录下的sln文件在visual studio下编译即可。Linux需要执行以下命令

   ```
   autoreconf -fi
   ./configure --prefix=目标目录 --可选参数
   ./configure --with-ssl=/usr/local/ssl --with-libssl-prefix=/usr/local/ssl/lib64
   make
   make install
   ```

   然后在[目标目录]下就出现了库文件和对应的头文件，就可以用于开发了。

   你的libcurl第一次可能不会成功，你需要根据提示，在`./configure`后添加可选参数，比如提示你确实了SSL，那么你需要安装openssl或者忽略SSL。这需要根据你的实际情况来定，忽略SSL意味着不能使用HTTP1.x以外的HTTP协议。但是大多数情况下，HTTP1.x已经够用了。centos安装openssl的命令为:

   ```
   yum install openssl-devel
   ```

   

4. 在你的项目中添加库文件和头文件

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

5. 开始享用curl吧！



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

https://curl.se/libcurl/c/curl_easy_setopt.html

https://everything.curl.dev/libcurl/options/all

https://blog.csdn.net/yupu56/article/details/43566085

#### curl的四个回调函数

- `CURLOPT_WRITEFUNCTION`：此回调函数用于接收响应数据。

  ```
  size_t function(char* buffer, size_t size, size_t nmemb, void* user_data)
  ```
  
  当libcurl从服务器接收到数据时，就会调用这个回调函数。你可以在这个函数里面将接收到的数据进行处理和保存。
  
  - buffer：接受到的数据数组的头节点
  - size：每个数据块的大小，单位字节，一般为1
  - nmemb：有多少个数据块
  - user_data：用户指定的参数
  - 返回值：接收了多少数据，一般为`size * nmemb`。如果不等说明发生了错误导致传输中断
  
- `CURLOPT_HEADERFUNCTION`：此回调函数用于处理响应报头。

  ```
  size_t function(char* buffer, size_t size, size_t nmemb, void* user_data)
  ```

  当libcurl接收到报头数据时，就会调用这个回调函数。请注意，这个函数会接收整个HTTP报头，也就是请求头(HTTP/1.1 200 OK)和所有请求行(Key: Value)。

  - buffer：接受到的数据数组的头节点
  - size：每个数据块的大小，单位字节，一般为1
  - nmemb：有多少个数据块
  - user_data：用户指定的参数
  - 返回值：接收了多少数据，一般为`size * nmemb`。如果不等说明发生了错误导致传输中断

- `CURLOPT_READFUNCTION`：此回调函数用于发送请求数据。

  ```
  size_t function(char* ptr, size_t size, size_t nitems, void* user_data)
  ```

  当libcurl需要发送数据时，就会调用这个回调函数。这个函数需要你实现：1. 将你需要发送的数据写入到buffer中；2. 返回数据量。

  - ptr：一块缓冲区，需要你尽可能多地将数据写入，但不能超过`size * nmemb`
  - size：每个数据块的大小，单位字节，一般为1
  - nmemb：最多可以写入多少个数据块
  - user_data：用户指定的参数
  - 返回值：你写入了多少数据。如果返回0，表示你已经写完了，curl会停止传输

- `CURLOPT_PROGRESSFUNCTION`：此回调函数用于报告操作进度。

  ```
  int function(void* clientp, double dltotal, double dlnow, double ultotal, double ulnow)
  ```

  当libcurl进行数据传输时，就会定期调用这个回调函数。你可以在这个函数中更新进度信息。返回值应该是0，如果返回非0值，libcurl会认为发生了错误，并终止请求。**这个选项是过时的，7.32.0以后的版本推荐使用下面的CURLOPT_XFERINFOFUNCTION，和这个语法一样**

- `CURLOPT_XFERINFOFUNCTION`：此回调函数用于报告操作进度。

  ```
  int function(void* clientp, double dltotal, double dlnow, double ultotal, double ulnow)
  ```

  - clientp：用户指定的参数
  - dltotal：总下载大小
  - dlnow：已经下载了多少
  - ultotal：总上传大小
  - ulnow：现在上传了多少
  - 返回值：返回0代表继续传输，返回非0代表终止传输





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

`event_base_loop()`：开启监听事件循环，可以设置标志位

`event_base_dispatch()`函数是`event_base_loop()`函数的一个包装器，它等同于调用`event_base_loop(base, 0)`

**多线程操作**

如果你要对一个`event_base`进行多线程操作，请在创建`event_base`之前调用`evthread_use_pthreads();`



## libevent定时器

libevent为我们封装了IO事件、定时器事件、信号事件。接下来介绍定时器事件。

```
// 创建事件树
event_base* base = event_base_new();

// 创建定时器事件。你可以创建多个定时器事件，也可以在event_base_dispatch()调用之后添加事件
// 不过要注意回多个事件中如果有一个调函数如果是阻塞的，可能会导致后面的定时超时，你需要在回调函数中使用多线程
event* timer = evtimer_new(m_ev_main_base, TickEventCB, this);

// 为定时器事件设置时间
// 请注意，定时器在设置时间之后才能被检测，如果没有对其的时间进行设置，这个定时器将永远不会被触发
timeval tv;
evutil_timerclear(&tv);
tv.tv_sec = 5;

// 把定时器事件挂到事件树上。
evtimer_add(m_ev_timer, &tv);

// 开始事件循环
event_base_dispatch(m_ev_main_base);

// 把事件从事件树上取下，不再检测该事件，但也不会释放该事件的资源
event_del(m_ev_timer);
// 把事件从事件树上取下并释放该事件的资源
event_free(m_ev_timer);
```

一旦定时器事件被触发，他将自动地从事件树上被取下

如果你要修改某个定时器的时间，你需要先从事件树上取下该事件，修改后再挂到事件树上。

`event_del()`和`event_free()`无需传入`event_base`，因为每个事件在创建时已经绑定了自己属于哪个`event_base`
