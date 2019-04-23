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

## signal

```;c
#include <signal.h>
void (*signal(int signo, void (*func)(int)))(int);
typedef void Sigfunc(int);
Sigfunc *signal(int, Sigfunc*);
int kill(pid_t pid, int signo);
int raise(int signo);  // Both return: 0 if OK, -1 on error
unsigned int alarm(unsigned int seconds);  // Returns: 0 or number or seconds until previously set alarm
int pause(void);  // Returns: -1 with errno set to EINTR
int sigemptyset(sigset_t *set);
int sigfillset(sigset_t *set);
int sigaddset(sigset_t *set, int signo);
int sigdelset(sigset_t *set, int signo);  // All four return: 0 if OK, -1 on error
int sigismember(const sigset_t *set, int signo);  // Returns:1 if true, 0 if false, -1 on error
int sigprocmast(int how, const sigset_t *restrict set, sigset_t *restrict oset);  // Returns: 0 if OK, -1 on error
```

