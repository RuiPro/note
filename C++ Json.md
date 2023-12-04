# 什么是Json

json是一种和语言无关的、有标准的序列化的数据结构。

json文件的后缀一般为.json

它的格式比如：

```
{
    "wifi":"开启",
    "蓝牙":"开启",
    "蜂窝网":"开启",
    "蜂窝网已使用数据量":1234,
    "电话卡列表":{
    	"卡数量":2,
        "卡1":["中国移动","12345678910"],
        "卡2":["中国联通","10987654321"]
    },
    "是否支持5G":false,
    "位置支持":["北斗", "GPS","伽利略"]
}
```

json可以存储以下类型的数据：

1. 整形(1234)
2. 浮点型(3.14)
3. 字符串("string")，在使用字符串时，需要使用`""`将字符串括起来才能生效。
4. 布尔类型(true、false)
5. 空值(null)
6. json对象
7. json数组

json对象和json数组内部支持包装其他基础类型。

- json对象：使用`{}`将多个键值对括起来。键值对使用`:`分隔键值，其中键必须使用字符串，值可以是任意json支持的数据类型，包括json对象，也就是说你可以`json对象 in json对象 in json对象 in json对象...`

  但请注意，json对象内的元素必须是键值对！

- json数组：使用`[]`将多个元素括起来，元素可以是任意json支持的数据类型。元素和元素之间的类型不必相同，比如`[123, 3.14, "string", false]`是合法的。同样的，元素也可以套娃。

json对象和json数组之间，各元素使用`,`进行分隔。`,`的作用只是分隔各元素，并不代表一个元素的结束，因此最后一个元素后面不用加`,`。



一个完整的json数据结构至少由一个json对象或json数组构成，且只能存在一个总json对象或json数组。

比如

```
{
	"int_num_array":[123]
}
```

是合法的，这一段只有一个总json对象，这个json对象里套娃了一个json数组，json数组里又套娃了一个整形123。

```
{
	"int_num":123
}
{
	"float_num":3.14
}
```

是非法的，这一段有两个并列的json对象，一个内部存储了一个整形123，另一个存储了一个浮点型3.14



在 JSON 中，空格、制表符、换行符等空白字符在语法上是被忽略的，因此在 JSON中添加这些符号不会影响解析。





# C++使用json

有很多优秀的开源库，但我选择的是JSON for Modern C++。它的官方地址是https://github.com/nlohmann/json/#projects-using-json-for-modern-c

它有很多吸引我的优点：导入项目方便：只有一个hpp头文件，以及类似STL容器的用法和迭代器。它是基于C++11实现的，也可以跨平台。

## 构建一个json

1. 创建一个json变量

   ```
   json data;
   ```

2. 如果你想让data变成一个json对象，那么你只需要对其添加json键值对即可：

   ```
   data["key"] = value;
   ```

   在添加了键值对后，data将自动变成一个json对象。

   如果json对象中存在名称为"key"的键，那么名称为"key"的键值会被更新为value，否则将创建一个键值对`"key": value`。

   注意，我们不能对json对象使用json数组独有的的`[下标]`运算符操作(比如`data[0] = 123;`)，这样会抛出异常`cannot use operator[] with a numeric argument with object`。

   如果你想创建一个空json对象：

   ```
   json data = json::object();
   ```

3. 如果你想让data变成一个json数组，那么你只需要为其提供一个数组(数组可以为空)：

   ```
   data = {123, 456}
   ```

   在JSON for Modern C++中，json数组的设计非常类似于标准库容器vector，你可以使用一些常见的函数，比如`[下标]`运算符用于读取某个元素的值，`size()`用于读取json数组的元素个数。此外，你还可以使用迭代器进行遍历，范围for语句也是可行的。

   相对的，我们不能对json数组使用这些语句比如`data["key"] = 123;`，这些语句是json对象独有的。

   如果你想创建一个空json数组：

   ```
   json data = json::array();
   ```

4. json对象和json数组相结合

   - 我们可以在json对象中嵌套一个json数组：

     ```
     object["key"] = {123，456};
     ```

     也可以这样：先创建一个json数组变量

     ```
     json array = {123，456};
     object["key"] = array;
     ```

   - 我们可以在json对象中嵌套一个json对象：

     ```
     json child_object;
     child_object["child_key"] = child_value;
     object["key"] = child_object;
     ```

   - 我们可以在json数组中嵌套一个json对象：

     ```
     json object;
     object["key"] = value;
     array = {object，123};
     ```

   - 我们可以在json数组中嵌套一个json数组：

     ```
     array = {{1，2，3}，123};
     ```

     或

     ```
     json child_array = {123，456};
     array.push_back(child_array);
     ```

比如，构建上文中的那个json，就可以这样写：

```
#include <iostream>
#include "json.hpp"
using json = nlohmann::json;
int main() {
    json data; // 创建一个空的JSON对象
    data["wifi"] = "开启";
    data["蓝牙"] = "开启";
    data["蜂窝网"] = "开启";
    data["蜂窝网已使用数据量"] = 1234;
    json cards;
    cards["卡数量"] = 2;
    cards["卡1"] = {"中国移动", "12345678910"};
    cards["卡2"] = {"中国联通", "10987654321"};
    data["电话卡列表"] = cards;
    data["是否支持5G"] = false;
    json location_support = {"北斗", "GPS", "伽利略"};
    data["位置支持"] = location_support;
    // 打印JSON对象
    std::cout << data.dump() << std::endl;
    return 0;
}
```



## 序列化与反序列化

- 序列化

  序列化就是将json变量序列化为标准字符串的过程。

  可以使用`nlohmann::json::dump()`这个函数将json变量转为string类型：

  ```
  string str_data = json_data.dump();
  ```

  `nlohmann::json::dump()`这个函数还支持传入一个整形参数，这个整形关系到制表符的宽度，用于确定转出的字符串的格式。他会对json进行人性化输出，比如

  ```
  {[[],[{"key":123}]]}
  dump(4):
  {
  	[
  		[],
  		[
  			{"key":123}
  		]
  	]
  }
  ```

  

- 反序列化

  反序列化就是将字符串或流解析为json变量的过程。

  可以使用`nlohmann::json::parse()`这个函数将字符串或流转为json变量。

  这个函数有三个重载：
  
  ```
  // 重载1：主要用于解析string
  static json parse(const std::string& s, const parser_callback_t cb = nullptr,
                    const bool allow_exceptions = true, const bool ignore_comments = false);
  
  // 重载2：主要用于解析流对象
  template<typename InputType>
  static json parse(InputType&& i, const parser_callback_t cb = nullptr,
                    const bool allow_exceptions = true, const bool ignore_comments = false);
                    
  //你可以使用
  ifstream json_file(file_path);
  json j;
  json_file >> j;
  
  
  // 重载3：主要用于解析string中的片段
  template<typename IteratorType>
  static json parse(IteratorType first, IteratorType last, const parser_callback_t cb = nullptr,
                    const bool allow_exceptions = true, const bool ignore_comments = false);
  ```
  
  该函数接受以下参数：
  - `s`：要解析的json字符串。
  - `cb`：回调函数，用于在解析过程中处理 JSON 对象的每个成员。默认值为 `nullptr`。
  - `allow_exceptions`：指示是否允许抛出异常。默认值为 `true`。如果值为true，解析json出错时会抛出一个异常。如果为false，解析json出错时会返回一个空json变量。
  - `ignore_comments`：指示是否忽略 JSON 字符串中的注释。默认值为 `false`。
  其中，`cb` 和 `ignore_comments` 参数的默认值在 JSON for Modern C++ 3.10.0 版本中新增。在早期版本中，`nlohmann::json::parse()` 函数没有这些参数。
  - `i`：需要用于解析json的流对象。比如文件流fstream。
  - `first`：要用于解析json的字符串的迭代器头部。
  - `last`：要用于解析json的字符串的迭代器尾部。
  
  
  
  除此之外，你还可以通过在字面量后面直接添加`_json`字段来实现从字符串反序列化为json，如：
  
  ```
  json j = "{ \"happy\": true, \"pi\": 3.141 }"_json;
  
  auto j2 = R"(
    {
      "happy": true,
      "pi": 3.141
    }
  )"_json;
  ```
  
  这样做推荐使用C++11的用户定义字面量特性。否则你需要对每一个`"`进行转义。



## 使用已有的json

使用已有的json非常简单！你只需反序列化以获得一个json变量。然后通过这些函数来解析json

- `size()`函数：用于获取json数组或对象的元素数量。
- `empty()`函数：用于判断json数组或对象是否为空。
- `clear()`函数：用于清空json数组或对象的元素。
- `type()`函数：用于获取某个json变量的类型。
- `is_array()` 函数：如果 JSON 变量是 JSON 数组，返回 `true`，否则返回 `false`
-  `is_object()` 函数：如果 JSON 变量是 JSON 对象，返回 `true`，否则返回 `false`
- `is_null()`：如果 JSON 变量是 null 类型，返回 `true`，否则返回 `false`
-  `is_boolean()`：如果 JSON 变量是 bool 类型，返回 `true`，否则返回 `false`
-  `is_number()`：如果 JSON 变量是 number 类型（int、float、double），返回 `true`，否则返回 `false`
- `is_string()`：如果 JSON 变量是 string 类型，返回 `true`，否则返回 `false`

- `at("key")`方法用来查找json对象对应键的值，`[下标]`的方法用来查找对应的json数组的元素。请注意，对json对象使用`[下标]`或对json数组使用`at("key")`会抛出异常。请注意下标也是从0开始
-  `contains("key")`：检查一个键是否存在于json对象中。如果对json对象使用，会抛出异常。
- `find("string")`函数：用于在某个json数组或对象中查找某个元素。如果存在，返回指向目标元素的迭代器，否则返回`end()`迭代器
- `erase("string")`函数：用于删除json数组或对象的某个元素
- `count("string")`函数：用于获取某个元素在json数组或对象中出现的次数
- 此外，你还可以使用迭代器。

例如，我要获取上例中卡2的号码，代码如下：

```
#include <iostream>
#include <string>
#include "json.hpp"

using json = nlohmann::json;
using namespace std;

int main() {
    system("chcp 65001");
    json data = R"(
        {
            "wifi":"开启",
            "蓝牙":"开启",
            "蜂窝网":"开启",
            "蜂窝网已使用数据量":1234,
            "电话卡列表":{
    	        "卡数量":2,
                "卡1":["中国移动","12345678910"],
                "卡2":["中国联通","10987654321"]
            },
            "是否支持5G":false,
            "位置支持":["北斗", "GPS","伽利略"]
        }
    )"_json;

    string card2_num;
    if(data.is_object() && data.at("电话卡列表").is_object()){
        auto iter = data.at("电话卡列表").find("卡2");
        if(iter != data.at("电话卡列表").end()){
            card2_num = (*iter)[1];
        }
        else{
            cout << "无卡2" << endl;
        }
    }
    cout << (data.type() == json::object()) <<endl;
    cout << "卡2的号码：" << card2_num << endl;

    return 0;
}
```

