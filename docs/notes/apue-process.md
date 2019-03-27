# 进程

```c
#include <sys/types.h>
#include <unistd.h>
pid_t getpid(void);
pid_t getppid(void);
// 两个系统调用都不会返回错误
int execl(const char *path, const char *arg, ...);
int execlp(const char *file, const char *arg, ...);
int execle(const char *path, const char *arg, ..., char * const envp[]);
int execv(const char *path, char *const argv[]);
int execvp(const char *file, char *const argv[]);
int execve(const char *filename, char *const argv[], char *const envp[]);

pid_t fork(void);
void exit(int status);
int atexit(void (*func)(void));
void on_exit(void (*func)(int, void*), void *arg);

pid_t wait(int *status);
pid_t waitpid(pid_t pid, int *status, int options);
int waitid(idtype_t idtype, id_t id, siginfo_t *infop, int options);
int system(const char *command);
```



大多数操作系统只提供了单个系统调用来启动新的进程，而UNIX提供了两个系统调用：fork和exec。

内核运行的第一个进程是init进程，其pid值为1。

如果用户没有指定一个init程序，内核会尝试执行合适的init程序

1. /sbin/init
2. /etc/init
3. /bin/init
4. /bin/sh：没有找到init程序时，就会尝试执行sh程序。

在以上四个位置，最先被发现的会当作init运行。如果四个都失败了，内核就会报警，然后系统挂起。

内核交出控制权后，init会接着完成后续的启动过程。包括初始化系统，启动各种服务以及启动登录进程。

**pid最大值**：/prec/sys/kernel/pid_max

## 进程体系

用户和组

进程组

pid_t: `<sys/types.h>`

## 运行新进程

在UNIX中，把程序载入内存并执行程序映像的操作与创建新进程的操作是分离的。

exec系统调用会把二进制程序加载到内存中，替换地址空间原来的内容，并开始执行。这个过程称为“执行”。

fork系统调用用于创建一个新的进程，它相当于复制其父进程。通常情况下，新的进程会立即执行新的程序。它需要两个步骤：首先创建一个新的进程，然后通过exec系统调用把新的二进制程序加载到该进程中。

## exec

在Linux中，exec系函数只有一个是真正的系统调用，其他都是基于该系统调用在C库中封装的函数。execve()是唯一的系统调用。