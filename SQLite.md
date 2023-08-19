## 接触SQLite

SQLite(官方网站www.sqlite.org)是一个由C语言开发的轻量级的数据库管理系统(DBMS)。

SQLite官网上SQLite的自述为体积小、速度快、设备齐全、可靠性高、功能齐全。它具有以下特性：

1. 轻量级：没有安装包，只有绿色可执行文件，开发时只有可数的几个C头文件，可以很容易集成到项目中。
2. 速度快，在TB级别以下的IO场景下，SQLite毫不逊色于MySQL这些DBMS。
3. 支持大部分标准SQL语句，开源，功能齐全。
4. 非常可靠，从2000年到现在已经发展了23年，其官网使用的数据库也是SQLite。
5. 稳定，其承诺到2050年前不会改变数据库的兼容性，这意味着你可以使用2050年的软件打开一个2000年创建的数据库。
6. 数据库是单文件形式，可以放在任何位置，可以非常方便地拷贝、分享。
7. 他是无系统的，这意味着你不需要单独开一个数据库管理系统进程。

因为其轻量级的特性，它不需要像MySQL那样需要一个外置运行的DBMS，它跟随项目一起运行，把单个文件的数据库创建在项目目录里，一个数据库就是一个文件，因此它没有SQL中创建数据库的CREATE DATABASE指令，取而代之的是你可以创建多个数据库文件来创建多个数据库。这一点是和其它DBMS不同的，其官网的描述也是：“我们不与MySQL这样的数据库管理系统竞争，相反，我们与C语言的fopen()函数竞争。”

如果你还在使用fopen()或fstream对象存储和读取数据，不如试试SQLite吧！



## 编译SQLite库

大多数Linux发行版都自带SQLite3软件。但是我们进行开发时不能使用可执行文件，需要库。

```
// Centos安装sqlite库
yum install libsqlite3-dev
```

或者你可以在SQLite官网下载SQLite的源码进行自己编译

www.sqlite.org > Download > Source Code

amalgamation版本只包含了SQLite的C源码。

autoconf不仅包含了源码，还包含了一些编译的配置，允许我们一键编译可执行文件和库。

前者你需要手动编译动态库，但显然选择后者会更方便，我们在Linux下下载后，解压并进入其目录

```
chmod +x configure
./configure
make
```

等待编译完成，其目录下的sqlite3.h头文件和.libs/目录下的.so和.a库文件就是我们使用SQLite开发的必要库资源。



## C++引入SQLite

以3.42.0版本的SQLite为例，先包含头文件

```
#include "sqlite3.h"
```

在CMakeLists.txt中添加SQLite库

```
target_link_libraries(bin_name path/of/lib/libsqlite3.so.0.8.6)
```

开始使用，这是一个简单的小demo，用于打开一个名为test.db的数据库。如果没有，则创建test.db数据库文件。

```
#include <iostream>
#include "sqlite3.h"
#include "function.h"

using namespace std;

int main(int argc, char** argv) {
	cout << "You are using SQLite3 version " << sqlite3_version << endl;
    sqlite3* db;
    char* db_err = 0;
    int return_value;
    // 获取程序路径，打开路径下的test.db，如果没有则创建
    string db_path = GetPath(argv[0]);
    db_path += "test.db";
    return_value = sqlite3_open(db_path.c_str(), &db);
    if (return_value) {
        fprintf(stderr, "Can't open database: %s\n", sqlite3_errmsg(db));
        exit(0);
    }
    else {
        fprintf(stderr, "Opened database successfully\n");
    }
    // 关闭
    sqlite3_close(db);
	return 0;
}
```



## 开始在C/C++中使用SQLite

SQLite的C/C++ API围绕三个步骤展开：打开数据库、执行SQL操作、关闭数据库。在SQLite的C/C++中，数据库是以变量的形式存在的(用C开发的它并不知道类的概念)，即sqlite3类型的变量(其实称为对象更妥)。sqlite3结构定义了与SQLite数据库的连接，包括用于执行SQL语句和管理事务的函数。

我们可以直接定义一个数据库变量：

```
sqlite3* db;
```



### SQLite的错误码

在SQLite的API中，返回值为0意味着API调用成功，返回其他则调用过程中出错。

```
#define SQLITE_OK           0   /* Successful result */
#define SQLITE_ERROR        1   /* Generic error */
#define SQLITE_INTERNAL     2   /* Internal logic error in SQLite */
#define SQLITE_PERM         3   /* Access permission denied */
#define SQLITE_ABORT        4   /* Callback routine requested an abort */
#define SQLITE_BUSY         5   /* The database file is locked */
#define SQLITE_LOCKED       6   /* A table in the database is locked */
#define SQLITE_NOMEM        7   /* A malloc() failed */
#define SQLITE_READONLY     8   /* Attempt to write a readonly database */
#define SQLITE_INTERRUPT    9   /* Operation terminated by sqlite3_interrupt()*/
#define SQLITE_IOERR       10   /* Some kind of disk I/O error occurred */
#define SQLITE_CORRUPT     11   /* The database disk image is malformed */
#define SQLITE_NOTFOUND    12   /* Unknown opcode in sqlite3_file_control() */
#define SQLITE_FULL        13   /* Insertion failed because database is full */
#define SQLITE_CANTOPEN    14   /* Unable to open the database file */
#define SQLITE_PROTOCOL    15   /* Database lock protocol error */
#define SQLITE_EMPTY       16   /* Internal use only */
#define SQLITE_SCHEMA      17   /* The database schema changed */
#define SQLITE_TOOBIG      18   /* String or BLOB exceeds size limit */
#define SQLITE_CONSTRAINT  19   /* Abort due to constraint violation */
#define SQLITE_MISMATCH    20   /* Data type mismatch */
#define SQLITE_MISUSE      21   /* Library used incorrectly */
#define SQLITE_NOLFS       22   /* Uses OS features not supported on host */
#define SQLITE_AUTH        23   /* Authorization denied */
#define SQLITE_FORMAT      24   /* Not used */
#define SQLITE_RANGE       25   /* 2nd parameter to sqlite3_bind out of range */
#define SQLITE_NOTADB      26   /* File opened that is not a database file */
#define SQLITE_NOTICE      27   /* Notifications from sqlite3_log() */
#define SQLITE_WARNING     28   /* Warnings from sqlite3_log() */
#define SQLITE_ROW         100  /* sqlite3_step() has another row ready */
#define SQLITE_DONE        101  /* sqlite3_step() has finished executing */
```

当返回错误码时，可以使用`sqlite3_errmsg()`或`sqlite3_errmsg16()`函数获取错误的英文描述。`sqlite3_errmsg()`或`sqlite3_errmsg16()`函数用于获取最近一次SQLite API调用失败时的错误信息。它接受一个`sqlite3*`类型的参数，表示要检查错误信息的数据库连接。它返回一个指向字符串的指针，该字符串包含错误信息。其中`sqlite3_errmsg()`返回的字符串是UTF8编码的，`sqlite3_errmsg16()`则返回UTF16编码的字符串。



### 打开数据库

#### `sqlite3_open()` 与 `sqlite3_open16()`

```
int sqlite3_open(const char *filename, sqlite3 **ppDb);
int sqlite3_open16(const char *filename, sqlite3 **ppDb);
```

- filename：传入要打开的数据库文件的绝对路径字符串，`sqlite3_open16()`需要为UTF8编码。如果是UTF16编码，则需要使用`sqlite3_open16()`。
- ppDb：传出存放数据库对象指针的地址。
- 返回值：返回一个整形结果。

该函数打开一个数据库文件，返回一个用于其他 SQLite 程序的数据库连接对象。

如果filename参数是`NULL`或`:memory:`，那么该函数将会在 RAM 中创建一个内存数据库，且只会在数据库变量的生命周期内有效。如果文件名 filename 不为`NULL`，那么该函数将使用这个参数值尝试打开数据库文件。如果该路径的数据库文件不存在，该函数将在该路径下创建该数据库文件并打开。



#### `sqlite3_open_v2()`

```
int sqlite3_open_v2(
  const char *filename,
  sqlite3 **ppDb,
  int flags,
  const char *zVfs
);
```

- filename：传入要打开的数据库文件的绝对路径字符串。

- ppDb：传出存放数据库变量指针的地址。

- flags：传入标志位，它必须至少包含以下三个标志组合之一：

  - `SQLITE_OPEN_READONLY`：以只读模式打开数据库。如果数据库不存在，则返回错误。
  - `SQLITE_OPEN_READWRITE`：如果可能，以读写模式打开数据库，或者如果文件受操作系统的写保护，则以只读模式打开。在任何一种情况下，数据库都必须已经存在，否则返回错误。
  - `SQLITE_OPEN_READWRITE | SQLITE_OPEN_CREATE`：以读写模式打开数据库，并在不存在时创建它。这是`sqlite3_open()`和`sqlite3_open16()`始终使用的行为。

  除了必需的标志外，还支持以下可选标志：

  - `SQLITE_OPEN_URI`：如果设置了此标志，则可以将文件名解释为URI。
  - `SQLITE_OPEN_MEMORY`：数据库将作为内存中的数据库打开。

- zVfs：传入`sqlite3_vfs`对象的名称，它定义了新数据库连接应使用的操作系统接口。如果此参数为`NULL`，则将使用默认的`sqlite3_vfs`对象，一般都使用`NULL`值。

- 返回值：返回一个整形结果。

该函数打开一个数据库文件，返回一个用于其他 SQLite 程序的数据库连接对象。

如果filename参数是`NULL`或`:memory:`，那么该函数将会在 RAM 中创建一个内存数据库，且只会在数据库变量的生命周期内有效。如果文件名 filename 不为`NULL`，那么该函数将使用这个参数值尝试打开数据库文件。如果该路径的数据库文件不存在，该函数将在该路径下创建该数据库文件并打开。



### 执行SQL操作

#### `sqlite3_exec()`

```
int sqlite3_exec(
  sqlite3* db,
  const char *sql,
  int (*callback)(void*,int,char**,char**),
  void * arg,
  char **errmsg
);
```

- db：传入一个已经打开的数据库变量的地址。

- sql：传入UTF8编码的SQL语句字符串。

- callback：传入回调函数的函数指针，当SQL语句返回的结果集中有数据时，对每一条(行)数据都会调用这个回调函数。

  如果没有结果集，则不会调用这个回调函数，设置为`NULL`即可。比如插入数据、创建表时。

  且这个回调函数的格式需要固定为

  ```
  int (*callback)(void* arg, int column, char** data_set, char** column_set);
  
  int callback(void* arg, int column, char** data_set, char** column_set){
  	//todo
  }
  ```

  - arg：即`sqlite3_exec()`的第四个参数arg

  - column：列数，即关系型数据库的字段数量

  - data_set：存放该行数据的字符串数组，大小是column。值得一提的是，即使数据在数据库中存放的数据类型为INT，也会被当作字符串来读取，你需要手动根据下面的column_set和SQL语句`PRAGMA table_info(table_name)`来判断处理。

    > `PRAGMA table_info(table_name)`
    >
    > 此SQL语句查询表的各字段属性。
    >
    > 查询结果类似这样：
    >
    > ```
    > 0|id|INT UNSIGNED|1||0
    > 1|name|char(10)|1||0
    > 2|gender|char(1)|1||0
    > 3|class|char(10)|0||0
    > 4|college_id|INT UNSIGNED|0||0
    > 5|tuition|INT UNSIGNED|0||0
    > 6|headmaster|INT UNSIGNED|0||0
    > ```
    >
    > 第一个字段是字段名，第二个是字段类型，第三个为是否为`NOT NULL`(0为可以，1为不可以)，第四个为默认值。可以根据这个SQL语句先执行一次SQL操作得到字段属性，再根据属性进行数据的SQL操作。

  - column_set：字段名数组，大小是column。

  - 此回调函数返回0时表示回调成功，**请务必返回一个返回值！**

- arg：即传给上面回调函数的自定义参数。

- errmsg：一个指向错误消息字符串的指针的地址。如果执行SQL语句时发生错误，将在此处返回错误消息。如果不需要获取错误消息，则可以将此参数设置为`NULL`。

- 返回值：返回一个整形结果。

这个函数可以用来执行单个SQL语句。它接受一个数据库连接、要执行的SQL语句、回调函数和错误消息指针作为参数。如果执行成功，它将返回`SQLITE_OK`。



#### `sqlite3_prepare_v2()`和`sqlite3_step()`

```
sqlite3_prepare(
  sqlite3 *db,
  const char *zSql,
  int nByte,
  sqlite3_stmt **ppStmt,
  const char **pzTail
);
int sqlite3_prepare_v2(
  sqlite3 *db,
  const char *zSql,
  int nByte,
  sqlite3_stmt **ppStmt,
  const char **pzTail
);
int sqlite3_prepare_v3(
  sqlite3 *db,
  const char *zSql,
  int nByte,
  unsigned int prepFlags,
  sqlite3_stmt **ppStmt,
  const char **pzTail
);
// 以上三个函数只能接收UTF8编码的字符串
// 此外，还存在sqlite3_prepare16()、sqlite3_prepare16_v2()、sqlite3_prepare16_v3()，他们是上面三个函数的UTF16变体，功能上一样但是只能接收UTF16编码的字符串
```

- db：传入一个已经打开的数据库变量的地址。

- zSql：传入要进行解析编译的SQL语句。

- nByte：传入一个整数，表示`zSql`的长度。如果`zSql`是以空字符结尾的字符串，则可以将此参数设置为-1。

- ppStmt：传入一个指向`sqlite3_stmt`类型的指针的地址，用于传出解析后的保存SQL语句的`sqlite3_stmt`结构体。

  `sqlite3_stmt`结构体保存了解析后的SQL语句，可以被多次执行，每次执行都可能具有不同的绑定参数值，它的主要作用是提高了查询性能。解析后的 SQL 语句可以被重复使用，而不用每次都重新解析SQL语句。这种预编译的过程可以提前检查SQL语法的正确性，大大提高了程序的效率。

  `sqlite3_stmt`结构体包含了许多信息，包括：

  - SQL 语句的文本。
  - 绑定到 SQL 语句中变量的值。
  - 查询结果中的当前行。
  - 查询结果中的列信息，包括列名称和数据类型。
  - 与 SQL 语句相关的数据库连接。

  因此，你不能单纯地把它看作是解析后的SQL语句，它更像是一个记录员，记录了目标、进度、下一步要干什么。

  可以使用 `sqlite3_reset()` 函数来重置一个 `sqlite3_stmt` 对象。该函数将重置 `sqlite3_stmt` 对象的状态，使其可以被重新执行，就好像播放器的进度条被拉回了开始一样。

  **不过，`sqlite3_stmt`结构体变量在使用完毕后也需要使用`sqlite3_finalize()`手动进行资源释放。**

- pzTail：传出一个指向字符串指针的指针。如果`zSql`包含多个SQL语句，则在此处返回未使用部分的内容。如果不需要获取未使用部分，则可以将此参数设置为`NULL`。

- prepFlags：传入一个整数，表示预处理标志。可以使用多个标志的按位或组合。

  - `SQLITE_PREPARE_PERSISTENT`：这个标志表示预处理语句对象可以在多次调用`sqlite3_reset()`或`sqlite3_finalize()`函数后继续使用。
  - `SQLITE_PREPARE_NORMALIZE`：这个标志表示在编译SQL语句之前，应对其进行规范化。规范化会删除SQL语句中不必要的空格和换行符，并将所有关键字转换为大写。
  - `SQLITE_PREPARE_NO_VTAB`：这个标志表示禁止使用虚拟表。

  可以根据需要选择适当的预处理标志。如果不需要任何特殊行为，则可以将`prepFlags`参数设置为0。

- 返回值：返回一个整形结果。

这三个函数可以用来执行预编译的SQL语句。其中`sqlite3_prepare()`和`sqlite3_prepare_v2()`函数都用于将SQL语句编译为预处理语句对象。它们的区别在于对模式错误的处理方式不同。当数据库模式发生更改时，`sqlite3_prepare()`函数可能会返回`SQLITE_SCHEMA`错误。在这种情况下，应用程序需要重新准备语句并再次尝试执行。相比之下，`sqlite3_prepare_v2()`函数会自动检测模式错误并重新准备语句。这意味着，当使用`sqlite3_prepare_v2()`函数时，应用程序不需要检查`SQLITE_SCHEMA`错误并手动重新准备语句。除了对模式错误的处理方式不同，`sqlite3_prepare()`和`sqlite3_prepare_v2()`函数的功能和用法基本相同。建议使用`sqlite3_prepare_v2()`函数，因为它更方便且更安全。`sqlite3_prepare_v3()`函数在功能上和`sqlite3_prepare_v2()`函数相同，但是可以自定义处理标志。

在得到解析后的`sqlite3_stmt`变量后，就可以使用`sqlite3_step()`执行SQL语句了。

```
int sqlite3_step(sqlite3_stmt* pStmt);
```

该函数接受一个指向解析后的`sqlite3_stmt`变量的指针，返回执行的状态码。当执行SELECT语句时，`sqlite3_step()`函数将返回`SQLITE_ROW`，表示查询返回了一行结果，当查询没有更多结果时，`sqlite3_step()`函数将返回`SQLITE_DONE`。当执行不返回结果的语句（如INSERT、UPDATE或DELETE）时，`sqlite3_step()`函数将直接返回`SQLITE_DONE`，表示操作已完成。

根据这个函数的英文我们可以看出来这个函数是逐步行进的。每一步走完后，都会返回一个返回值。此时，我们可以使用`sqlite3_column_*()`函数来获取查询结果中的字段值，如下：

```
const void* sqlite3_column_blob(sqlite3_stmt*, int iCol);			//获取BLOB字段值。
double sqlite3_column_double(sqlite3_stmt*, int iCol);				//获取浮点字段值。
int sqlite3_column_int(sqlite3_stmt*, int iCol);					//获取整数字段值。
sqlite3_int64 sqlite3_column_int64(sqlite3_stmt*, int iCol);		//获取64位整数字段值。
const unsigned char *sqlite3_column_text(sqlite3_stmt*, int iCol);	//获取UTF8字符串字段值。
const void* sqlite3_column_text16(sqlite3_stmt*, int iCol);			//获取UTF16字符串字段值。
sqlite3_value* sqlite3_column_value(sqlite3_stmt*, int iCol);		//获取值，并保存在sqlite3_value结构体里

sqlite3_column_count(sqlite3_stmt* pStmt)				//获取字段数量
sqlite3_column_bytes(sqlite3_stmt* pStmt, int iCol)		//获取BLOB或UTF8字符串字段值的大小。
sqlite3_column_bytes16(sqlite3_stmt* pStmt, int iCol)	//获取BLOB或UTF16字符串字段值的大小。
// 以上两个函数在获取大小时会发生类型转换，比如int值123会被识别为char[3]
sqlite3_column_type(sqlite3_stmt* pStmt, int iCol)		//获取字段的数据类型。
const char* sqlite3_column_name(sqlite3_stmt*, int N);	//获取字段名，也就是列名。
```

- pStmt：传入一个`sqlite3_stmt*`类型的参数，表示要获取数据的已经预处理过的SQL语句对象。

- iCol：传入一个整数，表示要获取的字段的索引（从0开始），表示你想获取哪一列的数据。如果超出了列数，会返回错误码

`sqlite3_column_count()`用于获取字段数量，也就是获取数据库有几列；`sqlite3_column_name()`用以获取字段名。

 `sqlite3_column_type()` 函数来确定查询结果中每一列的数据类型。该函数返回一个整数，表示该列的数据类型。返回值可能是以下几种之一：`SQLITE_INTEGER`整形，`SQLITE_FLOAT`浮点型，`SQLITE_TEXT`字符串，`SQLITE_BLOB`BLOB类型 或 `SQLITE_NULL`空值。然后，可以根据返回的数据类型使用对应的 `sqlite3_column_*()` 函数来获取该列的值。

当然，SQLite3也为我们提供了`sqlite3_value`用于保存各种类型的值，不过，在获取到值之后你任然需要使用`sqlite3_value_type()`进行类型判断，再调用对应的函数`sqlite3_value_*()`来取出其中的值。这是一个示例：

```
sqlite3_value *value = sqlite3_column_value(stmt, 0);
int type = sqlite3_value_type(value);
if (type == SQLITE_INTEGER) {
    int int_value = sqlite3_value_int(value);
    // 处理整数值
} else if (type == SQLITE_TEXT) {
    const unsigned char *text_value = sqlite3_value_text(value);
    // 处理文本值
} // 其他类型
```

因为这和`sqlite3_column_*()`对比就是在脱裤子放屁，所以一般直接使用`sqlite3_column_*()`即可。

在获取到值之后，我们可以再次调用`sqlite3_step()`函数来获得下一条数据，直到没有数据。因此，大部分时间我们都会写一个while循环判断`sqlite3_step()`函数的返回值，在循环体内部使用`sqlite3_column_*()`来获取和处理每一条数据。



#### `sqlite3_prepare_v2()`、`sqlite3_bind_*()`和`sqlite3_step()`

这些函数可以用来执行带参数的预编译的SQL语句。首先，使用`sqlite3_prepare_v2()`函数将带参数的SQL语句编译为预处理语句对象。然后，使用`sqlite3_bind_*()`函数绑定参数值。最后，使用`sqlite3_step()`函数执行预处理语句对象。在使用方式上，比上一小节多了`sqlite3_bind_*()`函数，用于后续绑定一些SQL语句内的值。

为什么会多出这样一种形式？

在准备SQL语句时，你可能会对SQL语句进行字符串拼接。这对C++来说并不算什么难事，C++的`string`类重载了`+`号运算符。但C语言并不能简单地使用`+`来拼接字符串。因此，当你想自定义SQL时，会感到很繁琐。

比如下面这个SQL语句意在插入一条教师数据，id为1，姓名为陈老师，薪资为3000。

```
int id = 1;
string name = "陈老师";
int salary = 3000;
string sql = "INSERT INTO teacher VALUES (";
sql += to_string(id);
sql.push_back(',');
sql.push_back('\'');
sql += name;
sql.push_back('\'');
sql.push_back(',');
sql += to_string(salary);
sql.push_back(')');
sql.push_back('\;');
```

你可以看到这非常繁琐，而且还是数据量小、SQL逻辑简单的情况下。

如果再让你插入王老师的数据，你会怎么办？再使用+++push_back+++吗？

SQLite3为我们提供了一个方法，让我们可以后绑定数据。那就是`sqlite3_bind_*()`函数

```
int sqlite3_bind_blob(sqlite3_stmt* pStmt, int index, const void*, int n, void(*)(void*));
int sqlite3_bind_blob64(sqlite3_stmt* pStmt, int index, const void*, sqlite3_uint64, void(*)(void*));
int sqlite3_bind_double(sqlite3_stmt* pStmt, int index, double);
int sqlite3_bind_int(sqlite3_stmt* pStmt, int index, int);
int sqlite3_bind_int64(sqlite3_stmt* pStmt, int index, sqlite3_int64);
int sqlite3_bind_null(sqlite3_stmt* pStmt, int index);
int sqlite3_bind_text(sqlite3_stmt* pStmt,int index,const char*,int,void(*)(void*));
int sqlite3_bind_text16(sqlite3_stmt* pStmt, int index, const void*, int, void(*)(void*));
int sqlite3_bind_text64(sqlite3_stmt* pStmt, int index, const char*, sqlite3_uint64, void(*)(void*), unsigned char encoding);
int sqlite3_bind_value(sqlite3_stmt* pStmt, int index, const sqlite3_value*);
int sqlite3_bind_pointer(sqlite3_stmt* pStmt, int index, void*, const char*,void(*)(void*));
int sqlite3_bind_zeroblob(sqlite3_stmt* pStmt, int index, int n);
int sqlite3_bind_zeroblob64(sqlite3_stmt* pStmt, int index, sqlite3_uint64);
```

- pStmt：传入一个`sqlite3_stmt*`类型的参数，表示要给预处理过后的SQL语句绑定值。

- index：索引。

  首先你需要在原SQL语句中使用占位符`?`、`?id`、`:tag`、`$tag`、`@tag`这五种形式来进行占位操作。比如

  ```
  INSERT INTO teacher VALUES (?,?,?);
  INSERT INTO teacher VALUES (?12,?34,?56);
  INSERT INTO teacher VALUES (:id,:name,:salary);
  INSERT INTO teacher VALUES ($id,$name,$salary);
  INSERT INTO teacher VALUES (@id,@name,@salary);
  ```

  这样，你就可以在`sqlite3_bind_*()`函数的第二个参数中填入索引了。

  如果你使用`?`、`?id`进行占位，可以直接输入id。对于只使用`?`的占位符，从SQL语句的左到右是从1开始的一个递增整数序列，而对于自定义id的占位形式`?id`，直接填入自定义的id即可。

  如果使用tag占位方式`:tag`、`$tag`、`@tag`进行索引，你需要先使用 `sqlite3_bind_parameter_index()` 函数来获取该tag的整型索引值，然后再使用该整型索引值来绑定值。

  ```
  int sqlite3_bind_parameter_index(sqlite3_stmt*, const char *tag);
  ```

  该函数传入`sqlite3_stmt*`类型的参数和tag名称，即可返回整型索引值。

  虽然你可以混合使用这5种占位方式，但这无疑会降低SQL语句的可读性，因此不提倡混合使用。

  除此之外，你还可以使用`int sqlite3_bind_parameter_count(sqlite3_stmt*);`来获取你在SQL语句中使用了几个占位符，使用`const char *sqlite3_bind_parameter_name(sqlite3_stmt*, int);`来获取第几个索引的tag名称。

然后，你就可以根据实际值来确定使用哪个`sqlite3_bind_*()`函数进行绑定数据了。比如

```
sqlite3_stmt *stmt;
const char *sql = "INSERT INTO mytable (column1, column2) VALUES (?, ?)";	// 定义了两个占位符
sqlite3_prepare_v2(db, sql, -1, &stmt, NULL);

int value1 = 123;
const char *value2 = "hello";
sqlite3_bind_int(stmt, 1, value1);							// 把第一个占位符改成整型123
sqlite3_bind_text(stmt, 2, value2, -1, SQLITE_TRANSIENT);	// 把第二个占位符改成字符串"hello"

sqlite3_step(stmt);
```





### 关闭数据库

可以使用 `sqlite3_close()` 函数来关闭之前调用 `sqlite3_open()` 打开的数据库连接。所有与连接相关的语句都应在连接关闭之前完成。如果还有查询没有完成，`sqlite3_close()` 将返回 `SQLITE_BUSY` 禁止关闭的错误消息。

```
int sqlite3_close(sqlite3* db);
```



## SQLite的默认表sqlite_master

每个SQLite数据库文件都存在一个默认表`sqlite_master`，保存了数据库的信息。

`sqlite_master` 表有以下五个字段：

| type                                                     | name       | tbl_name           | rootpage                                                     | sql                 |
| -------------------------------------------------------- | ---------- | ------------------ | ------------------------------------------------------------ | ------------------- |
| 对象的类型，可以是 ‘table’、‘index’、‘view’ 或 ‘trigger’ | 对象的名称 | 对象所属的表的名称 | 对象的根页号。对于表和索引，这是存储数据的 B-tree 的根页号。对于视图和触发器，此值为0 | 创建对象的 SQL 语句 |

- 列出所有表

  ```
  SELECT name FROM sqlite_master WHERE type='table' ORDER BY name;
  ```

  



















## 附录

### C++获取Linux可执行文件的位置的函数

```
#include <string>
#include <iostream>
#include <unistd.h>
using std::cin;
using std::cout;
using std::endl;
using std::string;

// 这个argv0是main(int argc, char** argv)里的argv[0]
std::string GetPath(char* argv0) {
	std::string env_path;
	if (argv0[0] == '.' && argv0[1] == '/') {       // 如果程序位置在当前路径下的子路径
		char path[1024]{ 0 };
		if (getcwd(path, 1024) == nullptr) {        // 程序路径 = 当前工作路径 拼接 程序执行路径
			cout << "Failed to get app path!" << endl;
			return env_path;
		}
		env_path = path;
		env_path += argv0 + 1;
	}
	else if (argv0[0] == '.' && argv0[1] == '.') {  // 如果程序位置在父路径下的子路径下
		char path1[1024]{ 0 };
		char path2[1024]{ 0 };
		if (getcwd(path1, 1023) == nullptr) {       // 程序路径 = 求绝对路径(当前工作路径 拼接 程序执行路径)
			cout << "Failed to get app path!" << endl;
			return env_path;
		}
		env_path = path1;
		env_path.push_back('/');
		env_path += argv0;
		realpath(env_path.c_str(), path2);
		env_path.clear();
		env_path = path2;
	}
	else {                                          // 如果程序位置是绝对路径，直接转换
		env_path = argv0;
	}
	while (env_path.back() != '/') {                 // 去掉程序名称，得到程序所在的目录
		env_path.pop_back();
	}
	return env_path;
}
```

