## YAML

YAML和JSON的作用类似，都是一种和语言无关的、有标准的序列化的数据结构。

YAML的文件后缀一般为.yml或.yaml

它的格式比如：

```
languages:
    - C
    - C++
    - Python 
websites:
    YAML: yaml.org 
    C++: cppreference.com
    Python: python.org
```

YAML是大小写敏感的

和JSON不同，YAML使用每一行前面加空格缩进表示层级，而JSON则使用中括号花括号嵌套来表示层级。请注意，必须使用空格进行缩进，不允许使用制表符来控制。多少个空格不重要，只要前面的空格数量相等(即左侧对齐)，就表示属于一个层级。

YAML也有数据类型：纯量、数组和对象，是不是很像JSON？不过YAML支持的类型多一些

YAML使用`#`注释一行



### YAML纯量

也就是YAML存储的最小的数据，和JSON一样，YAML也是使用键值对来存储数据，即`key: value`

- 字符串，不过和JSON不同，YAML的字符串不需要使用`""`包起来
- bool值，true或false
- 整形，1234
- 浮点型，3.14
- 时间，采用ISO8601格式，yyyy-mm-ddThh:mm:ss+时区，注意中间有个字符T
- 日期，复合 iso8601 格式，yyyy-mm-dd
- NULL，`null`或`~`

YAML允许使用`!!str`来将类型强制锁定为字符串

```
# value是字符串
key: !!str value
```

当字符串太长时，你也可以使用多行的形式.

1. 使用`>`号，这种形式每一行的结束和下一行的开始都会使用一个空格分隔开

   ```
   ALongString: >
     line1
     line2
     line3
   ```

   ```
   "line1 line2 line3\n"
   ```

   如果你不想要结尾的`\n`，你可以使用`>-`:

   ```
   ALongString: >-
     line1
     line2
     line3
   ```

2. 使用`|`号，这种形式每一行的结束和下一行的开始都会使用一个换行符分隔开，即字面量

   ```
   ALongString: |
     line1
     line2
     line3
   ```

   这样解析出来就是`line1\nline2\nline3\n`



### YAML数组

YAML数组是以`- `开头的多行，请注意`-`后面还有一个空格，比如

```
nums:
  - 123
  - 456
  - 789
```

YAML数组的元素缩进可以和数组的键名保持一致：

```
nums:
- 123
- 456
- 789
```

当然，你也可以使用`[]`:

```
nums: [123, 456, 789]
```



### YAML对象

YAML对象就是键值对(的集合)，如果这个对象不止一个元素，这些元素需要保持在同一个缩进层级下。

对象的每个元素都由键值对组成，其中值又可以是纯量、数组或对象。

比如

```
key: value
```

是一个对象，但这个对象只有一个键值对元素，键是`key`，值是`value`

```
section: 
  key1: value1
  key2: value2
  key3: value3
```

是一个对象，只有一个键值对元素，键是`section`，值又是一个对象，子对象又有三个键值对元素。

当然你也可以使用`{}`:

```
section: {key1:value1, key2:value2, key3: value3}
```



**复杂的形式**

你可以将纯量、数组和对象搭配使用

```
section: 
  key1: value1
  key2: value2
  key3:
    - 123
    - 456
    - 789
  key4: value4
  key5:
    ckey1: cvalue1
    ckey2: cvalue2
  key6:
    - 
     arr_key1: arr_value1
     arr_key2: arr_value2
    - 
     arr_key3: arr_value3
     arr_key4: arr_value4
     arr_key5: arr_value5
```



## GitHub Actions

GitHub Actions是GitHub推出的一个在线执行的功能。他会根据你配置的工作流文件实现自动云端跑脚本，实现云端编译，云端发布等等。它是免费的，而且性能很足，每个工作流都使用E5两个核心+7G内存的虚拟机，而且还有万兆网。

GitHub Actions由很多工作流`workflow`组成。工作流`workflow`由任务`job`组成，任务`job`由步骤`step`组成，`step`由动作`action`组成。



## workflow 文件

workflow文件就是你要运行云端脚本的配置文件。GitHub Actions依赖workflow文件，在仓库的`.github/workflows`目录下存放的任何workflow文件都会由GitHub在满足条件时自动运行。

workflow文件使用的是YAML格式，文件名可以任意取，但是后缀名统一为`.yml`。一个仓库可以有多个workflow文件。

workflow文件的格式和关键字很多，这是它的官方文档：https://docs.github.com/zh/actions/using-workflows/workflow-syntax-for-github-actions

https://www.ruanyifeng.com/blog/2019/09/getting-started-with-github-actions.html



