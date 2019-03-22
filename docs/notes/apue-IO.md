# 缓冲IO

## 块

块是IO的基本概念，是文件系统的最小操作单位。在内核中，所有的文件操作都是基于块来执行的。块大小一般是512、1024、2048或4096字节。缓冲IO块置IO缓冲时，把缓冲区大小设置成块的整数倍或约数，保证所有操作都是块对齐，IO性能可以得到提高。

### 用户缓冲IO

用户可以在自己的程序中实现缓冲，实际上很多关键应用就是自己实现了用户缓冲。

用户缓冲IO是在用户空间而不是内核中完成的，可以在应用程序中设置，也可以调用标准IO库。

为提高性能，内核通过延迟写、合并相邻的IO请求以及预读等操作来缓冲数据。用户缓冲IO也是为了提升性能。

## 标准IO

C标准库提供了标准IO库(简称stdio)，它实现了跨平台的用户缓冲方案，使用简单，功能强大。

在进行操作文件时，标准IO使用文件指针而不是文件描述符。在C标准库里，文件指针和文件描述符一一映射。

## 常用的文件操作

```c
#include <stdio.h>
// 打开文件
FILE *fopen(const char *path, const char *mode);  // 成功返回FILE指针，失败返回NULL并设置errno值
// 通过文件描述符打开流
file *fdopen(int fd, const char * mode);  // 成功返回FILE指针，失败返回NULL并设置errno值
// 关闭流
int fclose(FILE *stream);  // 成功返回0，失败返回EOF并设置errno值
// 关闭所有流
int fcloseall(void);  // 始终返回0
// 读数据，每次读取一个字节
int fgetc(FILE *stream);
// 把字符放回流中
int ungetc(int c, FILE *stream);  // 成功返回c, 失败返回EOF
// 读数据，每次读一行
char *fgets(char *str, int size, FILE *stream);  // 成功返回str，失败返回NULL
// 读数据，读二进制
size_t fread(void *buf, size_t size, size_t nr, FILE *stream);  // 返回读到的数据项个数，读取失败或文件结束，返回的数值可能比nr小
```

### fgetc

该函数从stream中读取一个字符，并把它强制转换成unsigned int返回。

fgetc()的返回值必须保存成int类型，把返回值保存为char类型是经常犯的错误，返回int类型是为了能够表示文件结束或错误(EOF)。

### fgets

该函数从stream中读取size-1个字节，并把结果保存到str中，并在str最后写入空字符(\0)。当读到换行符或EOF时会结束读。换行符也会被写入str中。

