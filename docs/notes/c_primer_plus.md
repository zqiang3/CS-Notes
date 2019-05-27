# 第10章 数组和指针

## 要点

数组的初始化：C99增加了指定初始化项目。

数组的边界

多维数组，二维数组的初始化

指针和数组，指针和多维数组

处理数组的函数

指针操作

C99标准增加了变长数组

C99增加了复合文字：(int [2]{10, 20} 表示包含两个int值的无名数组)

## 总结

声明一个数组：type name[size], 传统上size是一个常量整数表达式

当需要存储同种类型的许多元素时，数组是最佳选择。

C把数组名解释为数组首元素的地址。使用数组名传参时，是将数组的地址传递给函数。

数组和指针是紧密联系的，指针符号和数组符号往往可以互换使用。

如果不想修改原数组，在函数声明中可以为形式参量加上关键字const。

对指针进行增量运算时，指针值的改变是以指针相应类型的字节大小为单位的。

# 第13章 文件输入/输出



* fopen(), getc(), putc(), exit(), fclose(), fprintf(), fscanf(), fgets(), fputs(), rewind(), fseek(), ftell(), fflush(), fgetpos(), fsetpos(), feof(), ferror(), ungetc(), setvbuf(), fread(), fwrite()

## 概要

文件是什么：C将文件看成是连续的字节序列，每个字节都可被单独读取。

文本视图和二进制视图。使用文本视图时，会将行尾换行符进行相应转换。

标准文件：标准输入、标准输出、标准错误

exit, EXIT_SUCESS, EXIT_FAILURE

fopen()，如果文件不能打开，返回NULL

文件指针，FILE，文件指针指向一个关于文件信息的数据包，其中包括文件I/O使用的缓冲区信息。

getc(), putc()

文件结尾 EOF(-1)

fclose(), 成功返回0，否则返回EOF

fprintf(), fscanf(), fgets(), fputs()

fseek(), ftell() SEEK_SET, SEEK_CUR, SEEK_END，可移植性，文件大小只能在long类型表示范围之内

UNIX文本文件通常不包含Ctrl+Z和\r。

fgetpos() fsetpos()

I/O内幕

fread(), fwrite()

feof() ferror()