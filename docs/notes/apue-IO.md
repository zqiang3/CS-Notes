# 缓冲IO

## 链接

<http://lujun9972.github.io/blog/2015/05/15/%E6%A0%87%E5%87%86io%E5%BA%93/>

## 块

块是IO的基本概念，是文件系统的最小操作单位。在内核中，所有的文件操作都是基于块来执行的。块大小一般是512、1024、2048或4096字节。缓冲IO块置IO缓冲时，把缓冲区大小设置成块的整数倍或约数，保证所有操作都是块对齐，IO性能可以得到提高。

## 对齐

内存可看成一个字节数组。

处理理器以特定粒度来访问内存，如2、4、8或16字节

c变量的存储和访问都要求地址对齐，通常编译器会自动对齐所有的数据。

访问未对齐的数据会产生不同程度的性能问题。有些处理器可以访问不对齐的数据，但会有很大的性能损失。有些处理器根本无法访问不对齐的数据，尝试这么做会导致硬件异常。处理器在强制地址对齐时，可能会丢弃低位的数据，从而导致不可预料的行为。

处理结构体，手动执行内存管理，把二进制数据保存到磁盘中，以及网络通信都会涉及对齐问题。 

### 用户缓冲IO

用户可以在自己的程序中实现缓冲，实际上很多关键应用就是自己实现了用户缓冲。

用户缓冲IO是在**用户空间**而不是内核中完成的，可以在应用程序中设置，也可以调用标准IO库。

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
int fputc(int c, FILE *stream);  // 成功返回c, 失败返回EOF并设置errno值
int fputs(const char *str, FILE *stream);  // 成功返回非负整数，失败返回EOF
size_t fwrite(void *buf, size_t size, size_t nr, FILE *stream);  // 成功返回写入的数据项数，出错时返回值小于nr
int fseek(FILE *stream, long offset, int whence);  // 成功返回0找清空EOF，出错返回-1并设置errno
int fsetpos(FILE *stream, fpos_t *pos);  // 成功返回0找清空EOF，出错返回-1并设置errno
void rewind(FILE *stream);
long ftell (FILE *stream);  // 成功返回当前位置，出错返回-1并设置errno
int fgetpos(FILE *stream, fpos_t *pos);  // 成功返回0，失败返回-1并设置errno
int fflush(FILE *stream);  // 成功返回0，失败返回EOF并设置errno
```

| 家族名     | 目的    | 可用于所有流      | 只用于stdin stdout | 内存中的字符串 |
| ------- | ----- | ----------- | --------------- | ------- |
| getchar | 字符输入  | fgetc, getc | getchar         |         |
| putchar | 字符输出  | fputc, putc | putchar         |         |
| gets    | 文本行输入 | fgets       | gets            |         |
| puts    | 文本行输出 | fputs       | puts            |         |
| scanf   | 格式化输入 | fscanf      | scanf           | sscanf  |
| printf  | 格式化输出 | fprintf     | printf          | sprintf |

### fgetc

该函数从stream中读取一个字符，并把它强制转换成unsigned int返回。

fgetc()的返回值必须保存成int类型，把返回值保存为char类型是经常犯的错误，返回int类型是为了能够表示文件结束或错误(EOF)。

### fgets

该函数从stream中读取size-1个字节，并把结果保存到str中，并在str最后写入空字符(\0)。当读到换行符或EOF时会结束读。换行符也会被写入str中。

### fread

如果读取失败或结束，fread会返回一个比nr小的数。不幸的是，必须使用ferror或feof函数，才能确定是失败还是文件结束。

由于变量大小，对齐、填充、字节序这些因素的不同，由一个应用程序写入的二进制文件，另一个应用程序可能无法读取，同一个应用程序在不同机器也可能无法读取。

### fflush

This function causes any unwritten data for the stream to be passed to the kernel. If fp is NULL, this function causes all output streams to be flushed.

理解C库的缓冲区与内核缓冲区的区别，fflush并不保证数据马上被写到物理磁盘。

## STDIN_FILENO, STDOUT_FILENO, STDERR_FILENO

书中描述为：两个常量STDIN_FILENO和STDOUT_FILENO定义在<unistd.h>头文件中，它们指定了标准输入和标准输出的**文件描述符**。

1. STDIN_FILENO等是int类型，是文件描述符，一般定义为0, 1, 2。
2. 属于**系统API**接口库，没有buffer的I/O，**直接进行系统调用**，属于低级I/O，需要自己处理缓冲。
3. 头文件<unistd.h>
4. 使用STDIN_FILENO等的函数有：read, write, close等。
5. STDIN_FILENO等用于系统层的系统调用，操作系统级提供的文件API都是以文件描述符来表示文件。STDIN_FILENO就是标准输入设备（一般是键盘）的文件描述符。

# File I/O(文件IO)

```c
#include <fcntl.h>
int open(const char *pathname, int oflag, ... /* mode_t mode */ );
// Returns: fd id OK, -1 on error

#include <unistd.h>
int close(int fields);
// Returns: 0 if OK, -1 on error
off_t lseek(int fd, off_t offset, int whence);
// whence: SEEK_SET, SEEK_CUR, SEEK_END
// Returns: new offset if OK, -1 on error

ssize_t read(int fd, void *buf, size_t nbytes);
// Returns: bytes read if OK, 0 if EOF, -1 on error
ssize_t write(int fd, const void *buf, size_t nbytes);
// Returns: bytes written if OK, -1 on error

// 原子操作
ssize_t pread(int fd, void *buf, size_t nbytes, off_t offset);
// Returns: bytes read if OK, 0 if EOF, -1 on error
ssize_t pwrite(int fd, void *buf, size_t nbytes, off_t offset);
// Returns: bytes written if OK, -1 on error

int dup(int fd);
int dup2(int fd, int fd2);
// Returns: new fd if OK, -1 on error

int fsync(int fd);
int fdatasync(int fd);
// Returns: 0 if OK, -1 on error
void syn(void);

int fcntl(int fd, int cmd, ... /* int arg */ );
// Returns: 若成功依赖于cmd， －1 on error
```

## 文件描述符

内核用文件(fd)描述符标识**进程正在访问的文件**，文件描述符通常是一个非负整数。fd是动态分配的，优先分配未使用的最小值。

新进程执行时，shell会默认分配三个文件描述符，STDIN_FILENO/STDOUT_FILENO/STDERR_FILENO,一般为0/1/2，定义在<unistd.h>中

## 示例

```c
#include <stdio.h>
#include <stdlib.h>
#include <fcntl.h>

char *filename = "foo.txt";
int fd;
fd = open(filename, O_RDONLY | O_CREAT, 0666);
fd = open("temp.txt", O_RDONLY | O_CREAT | O_EXCL, 0666);  # 测试O_EXCL
fd = open(filename, O_RDONLY | O_TRUNC, 0666);  # 测试O_TRUNC标志位
ret = lseek(fd, 0, SEEK_END);
status = truncate(filename, 30);
if (fd < 0)
{
    perror("open");
    exit(1);
}
```

## open

```c
#include<sys/types.h>
#include<sys/stat.h>
#include<fcntl.h>

int open( const char * pathname, int flags);
int open( const char * pathname,int flags, mode_t mode);
```

**flags**

1. O_RDONLY 以只读方式打开文件
2. O_WRONLY 以只写方式打开文件
3. O_RDWR 以可读写方式打开文件。上述三种旗标是互斥的，也就是不可同时使用，但可与下列的旗标利用OR(|)运算符组合。
4. O_CREAT 若欲打开的文件不存在则自动建立该文件。
5. O_EXCL 如果O_CREAT也被设置，此指令会去检查文件是否存在。文件若不存在则建立该文件，若**文件存在**将导致打开文件错误。此外，若O_CREAT与O_EXCL同时设置，并且欲打开的文件为符号连接，则会打开文件失败。
6. O_TRUNC 若文件存在并且以可写的方式打开时，此旗标会令文件长度清为0，而原来存于该文件的资料也会消失。
7. O_APPEND 当读写文件时会从文件尾开始移动，也就是所写入的数据会以附加的方式加入到文件后面。
8. O_NONBLOCK 以不可阻断的方式打开文件，也就是无论有无数据读取或等待，都会立即返回进程之中。
9. O_NDELAY 同O_NONBLOCK。

**文件描述符**

open 返回的文件描述符一定是最小的未被使用的描述符。

**文件权限**

创建文件时的权限会受到umask 值所影响, 因此该文件权限应该为 (mode-umaks).

## read

```c
ssize_t read(int fd, void * buf, size_t count);
```

**说明**

read()会把参数fd所指的文件传送count 个字节到buf 指针所指的内存中 

**返回值**

成功返回读到的字节数；如果返回0, 表示已到达文件尾或是无可读取的数据；出错返回-1

多种情况会导致读到的字节数少于要求读的字节数：

1. 没读够就到文件尾了。例如想要100bytes，但到文件尾还有30bytes，会返回30（实际读到的字节数）；

2. 已到文件尾，返回0（实际读到的字节数）

3. 从特殊文件读，有限制：
   终端设备，通常最多1行；
   网络设备，缓冲机制能到导致没有那么多数据可读；
   管道或FIFO，没那么多数据可读；
   某些记录设备，一次最多返回1个记录；

4. 读时被信号中断

   ​

## write

```c
ssize_t write (int fd, const void * buf, size_t count);
```

**说明**

write()会把参数buf所指的内存写入count个字节到参数放到所指的文件内 

**返回值**

成功返回写入的字节数；出错返回-1，并写errno。

出错原因一般是磁盘满或超过文件长度限制

## fcntl

F_DUPFD： 复制文件描述符

F_GETFD: 文件描述符标志

F_SETFD: 设置文件描述符标志

F_SETFL: 设置文件状态标志

## 内核IO相关的数据结构

进程表项（包括文件描述符列表，指向文件表项）

文件表项，包含v节点指针

v节点表项。

**文件表项**

内核为所有打开文件维持一张文件表，包括:

1. 文件状态标志（读、写、添写、同步和非阻塞等）
2. 文件当前偏移量
3. 指向该文件V节点的指针（linux没有V节点）

**v-node与i-node**

每个文件都有，保存在磁盘上，与文件对应，打开文件时获取的，主要包括文件的所有者、文件长度、指向文件实际数据块在磁盘所在位置的指针等。

v-node是与文件系统无关的，所以单独提出来。linux里没有v-node，而是采用**“与文件系统无关的i节点”+“与文件系统有关的i节点”**的方式。

## sync, fsync和fdatasync

sync只是将所有修改过的块缓冲区排入写队列，然后返回，并不等待实际写磁盘操作结束。

fsync对fd的单一文件起作用，并且等待写磁盘操作结束，然后返回

fdatasync类似于fsync，但只影响文件的数据部分，而fsync还会同步更新文件的属性。

# Files and Directories

```c
#include <unistd.h>
#include <sys/stat.h>
int stat(const char *restrict pathname, struct stat *restrict buf);
int fstat(int filedes, struct stat *buf);
int lstat(const char *restrict pathname, struct stat *restrict buf);
// All three return: 0 if OK, -1 on error
int access(const char *pathname, int mode);  // Returns: 0 if OK, -1 on error
mode_t umask(mode_t cmask);  // Returns: previous file mode creation mask
int chmod(const char *pathname, mode_t mode);
int fchmod(int filedes, mode_t mode);  // Both return: 0 if OK, -1 on error
```

## stat structure

```c
struct stat {
  mode_t st_mode;
  ino_t  st_ino;
  dev_t  st_dev;
  dev_t  st_rdev;
  nlink_t  st_nlink;
  uid_t  st_uid;
  gid_t  st_gid;
  off_t  st_size;
  time_t st_atime;
  time_t st_mtime;
  time_t st_ctime;
  blksize_t  st_blksize;
  blkcnt_t   st_blocks;
};
```

## User IDs and group IDs associated with process

|                         |                                        |
| ----------------------- | -------------------------------------- |
| real user ID            | who we really are                      |
| real group ID           | who we really are                      |
| effective user ID       | used for file access permission checks |
| effective group ID      | used for file access permission checks |
| supplementary group IDs | used for file access permission checks |
| saved set-user-ID       | saved by exec functions                |
| saved set-group-ID      | saved by exec functions                |



## file access permission bits

from <sys/stat.h>

| st_mode mask | meaning       |
| ------------ | ------------- |
| S_IRUSR      | user-read     |
| S_IWUSR      | user-write    |
| S_IXUSR      | user-execute  |
| S_IRGRP      | group-read    |
| S_IWGRP      | group-write   |
| S_IXGRP      | group-execute |
| S_IROTH      | other-read    |
| S_IWOTH      | other-write   |
| S_IXOTH      | other-execute |

# standard I/O Library（标准IO库）

```c
#include <stdio.h>
int fwide(FILE *fp, int mode);
void setbuf(FILE *restrict fp, char *restrict buf);
int setvbuf(FILE *restrict fp, char *restrict buf, int mode, size_t size);
// returns 0 if OK, nonzero if error
FILE *fopen(const char *restrict pathname, const char *restrict type);
FILE *freopen(const char *restrict pathname, const char *restrict type, FILE *restrict fp);
FILE *fdopen(int fieldes, const char *type);
// All three return: file pointer if OK, NULL on error
int fclose(FILE *fp);  // returns: 0 if OK, EOF on error
int getc(FILE *fp);  // 可用宏实现
int fgetc(FILE *fp);
int getchar(void);
// All three return: next character if OK, EOF on end of file or error
int ferror(FILE *fp);
int feof(FILE *fp);
// Both return: nonzero (true) if condition is true, 0 (false) otherwise
void clearerr(FILE *fp);
int ungetc(int c, FILE *fp);  // Returns: c if OK, EOF on error
int putc(int c, FILE *fp);  // 可用宏实现
int fputc(int c, FILE *fp);
int putchar(int c);
// All three return: c if OK, EOF on error
char *fgets(char *restrict buf, int n, FILE *restrict fp);
char *gets(char *buf);  // the 'gets' function is dangerous and should not be used
// Both return: buf if OK, NULL on end of file or error
int fputs(const char *restrict str, FILE *restrict fp);
int puts(const char *str);
// Both reutrn: none-negative value if OK, EOF on error
long ftell (FILE *fp);  // Returns: current file postion indicator if OK, -1L on error
int fseek(FILE *fp, long offset, int whence);  // Returns: 0 if OK, nonzero on error
void rewind(FILE *fp);
off_t ftello(FILE *fp); // Returns: current file postion indicator if OK, -1 on error
int fseeko(FILE *fp, off_t offset, int whence);  // Returns: 0 if OK, nonzero on error
int fgetpos(FILE *restrict fp, fpos_t *restrict pos);
int fsetpos(FILE *fp, const fpos_t *pos);
// Both return: 0 if OK, nonzero on error
char *tmpnam(char *ptr);  // Returns: pointer to unique pathname
FILE *tmpfile(void);  // Returns: file pointer if OK, NULL on error

char *tempnam(const char *derectory, const char *prefix);  // Returns: pointer to unique pathname
int mkstemp(char *template);  // Returns: file descriptor if OK, -1 on error
```



## Streams 

With the standard I/O library, the discussion centers around streams. When we open or create a file with the standard I/O library, we say that we have associated a **stream** with the file.

ASCII字符集中一个字符用一个字节表示，国际字符集中一个字符可以用多个字节表示。

stream's orientation: byte-orientation, wide-orientation. Throughtout the rest of this book, we will deal only with byte-orientation streams.

## FILE Objects

File object is normally a structure that contains all the information required by the standard I/Olibrary to manage the stream: 

* the **file descriptor** used for the actural I/O
* **a pointer to a buffer** for the stream
* the **size of the buffer**
* a **count of the number of characters** currently in the buffer
* **error flag**

Standard I/O library has already been ported to a wide variety of ther operating systems other than UNIX system. In this book, we will talk about its typical implementation on a UNIX system.

## Standard Input, Standard Output, and Standard Error

标准输入，标准输出，标准错误是三种预定义的流，是进程自动打开的流。这三种流使用文件指针stdin, stdout, stderr表示，在`<stdio.h>`文件中定义。

stdin, stdout, stderr属于高级IO,带缓冲

这三种流分别有相应的文件描述符。

stdin: STDIN_FILENO

stdout: STDOUT_FILENO

stderr: STDERR_FILENO

## Buffering

The goal of the buffering provided by the standard I/O library is to use the minimum number of read and write calls.

标准IO库的缓冲也是最让人困惑的地方。

有三种类型的缓冲：

1. Fully buffered. 当缓冲区被填满时才会执行真正的IO操作。文件相关的操作默认采用这种方式。The buffer used is usually obtained by one of the standard I/O functions calling malloc the first time I/Ois performed on a stream.
2. Line buffered. 当遇到换行符时才执行真正IO操作。
3. Unbuffered. 标准IO库不进行缓冲。

大多数实现默认采用以下缓冲方式：

1. 标准错误是Unbuffered。不管是否遇到换行符，错误信息总是立即被显示。
2. 指向终端设置的流是Line buffered，其他流是Fully buffered.

### Opening a Stream

when a file is opened for reading and writing(the plus sign in the type), the following restrictions apply.

* Output cannot be directly followed by input without an intervening fflush, fseek, fsetpos, or rewind.
* Input cannot be directly followed by output without an intervening, fseek, fsetpos, or rewind, or an input operation that encounters an end of file.

在有些系统中测试，发现对于普通文件（非socket的操作），似乎不遵守这个规则读写也正常。然而，为了程序的可移植性和健壮性，依然建议遵守标准的规定。

man fopen中的一段话

> Reads and writes may be intermixed on read/write streams in any order.  Note that ANSI C requires that a file positioning function intervene between output  and  input,  unless  an  input  operation
> ​     encounters end-of-file.  (If this condition is not met, then a read is allowed to return the result of writes other than the most recent.)  Therefore it is good practice (and indeed sometimes neces‐
> ​     sary under Linux) to put an fseek(3) or fgetpos(3) operation between write and read operations on such a stream.  This operation may be an apparent no-op (as in fseek(..., 0L, SEEK_CUR)  called  for
> ​     its synchronizing side effect).

### freopen

This function is typically used to open a specified file as one of the predefined streams: standard input, standard output, or standard error.

### fdopen

Thie function is often used with descriptors that are returned by the functions that create pipes and network communication channels. Because these specail types of files cannot be opened with the standard I/O fopen function, we have to call the device-specific function to obtain a file descriptor, and then associate this descriptor with a standard I/O stream using fdopen.

