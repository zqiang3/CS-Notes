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

## 进程控制原语

原语表示是一个原子操作，执行期间不能被中断，是一个不可分割的基本单位。

进程控制原语包括进程的创建，进程的终止，进程的阻塞和唤醒，进程的切换。

## 进程体系

用户和组

进程组

pid_t: `<sys/types.h>`

## 进程的创建

子进程可以继承父进程拥有的资源。

创建一个新进程的过程如下（创建原语)：

1. 为新进程分配一个唯一的进程标识号，并申请一个空白的PCB(PCB是有限的)。若PCB申请失败则创建失败。
2. 为进程分配资源，为新进程的程序和数据、以及用户栈分配必要的内存空间（在PCB 中体现）。注意：这里如果资源不足（比如内存空间），并不是创建失败，而是处于”等待状态“，或称为“阻塞状态”，等待的是内存这个资源。
3. 初始化PCB,主要包括初始化标志信息、初始化处理机状态信息和初始化处理机控制信息，以及设置进程的优先级等。
4. 如果进程就绪队列能够接纳新进程，就将新进程插入到就绪队列，等待被调度运行。

## 进程的终止

操作系统终止进程的过程如下（撤销原语）：

1. 根据被终止进程的标识符，检索PCB，从中读出该进程的状态。
2. 若被终止进程处于执行状态，立即终止该进程的执行，将处理机资源分配给其他进程。
3. 若该进程还有子进程，则应将其所有子进程终止。
4. 将该进程所拥有的全部资源，或归还给其父进程或归还给操作系统。
5. 将该PCB从所在队列（链表）中删除。

当被阻塞进程所期待的事件出现时，如它所启动的I/O操作已完成或其所期待的数据已到达，则由有关进程（比如，提供数据的进程）调用唤醒原语(Wakeup)，将等待该事件的进程唤醒。

唤醒原语的执行过程是：

1. 在该事件的等待队列中找到相应进程的PCB。
2. 将其从等待队列中移出，并置其状态为就绪状态。
3. 把该PCB插入就绪队列中，等待调度程序调度。

需要注意的是，Block原语和Wakeup原语是一对作用刚好相反的原语，必须成对使用。 Block原语是由被阻塞进程自我调用实现的，而Wakeup原语则是由一个与被唤醒进程相合作或被其他相关的进程调用实现的。

## 进程切换

对于通常的进程，其创建、撤销以及要求由系统设备完成的I/O操作都是利用系统调用而进入内核，再由内核中相应处理程序予以完成的。进程切换同样是在内核的支持下实现的，因此可以说，任何进程都是在操作系统内核的支持下运行的，是与内核紧密相关的。

进程切换是指处理机从一个进程的运行转到另一个进程上运行，这个过程中，进程的运行环境产生了实质性的变化。

进程切换的过程如下：

1. 保存处理机上下文，包括程序计数器和其他寄存器。
2. 更新PCB信息。
3. 把进程的PCB移入相应的队列，如就绪、在某事件阻塞等队列。
4. 选择另一个进程执行，并更新其PCB。
5. 更新内存管理的数据结构。
6. 恢复处理机上下文。

注意，进程切换与处理机模式切换是不同的，模式切换时，处理机逻辑上可能还在同一进程中运行。如果进程因中断或异常进入到核心态运行，执行完后又回到用户态刚被中断的程序运行，则操作系统只需恢复进程进入内核时所保存的CPU现场，无需改变当前进程的环境信息。但若要切换进程，当前运行进程改变了，则当前进程的环境信息也需要改变。



## 运行新进程

在UNIX中，把程序载入内存并执行程序映像的操作与创建新进程的操作是分离的。

exec系统调用会把二进制程序加载到内存中，替换地址空间原来的内容，并开始执行。这个过程称为“执行”。

fork系统调用用于创建一个新的进程，它相当于复制其父进程。通常情况下，新的进程会立即执行新的程序。它需要两个步骤：首先创建一个新的进程，然后通过exec系统调用把新的二进制程序加载到该进程中。

## exec

在Linux中，exec系函数只有一个是真正的系统调用，其他都是基于该系统调用在C库中封装的函数。execve()是唯一的系统调用。

## fork

在UNIX里，fork是怎么做的呢？它是以一次一页的方式，把父进程的地址空间内容完全地拷贝到子进程里，再从父进程那里继承各种资源，像打开的文件，当前工作目录等。

子进程创建后被设置为就绪态，并把它放入到就绪队列。做完这些工作后fork在子进程返回值为0，在父进程返回子进程的pid。fork执行完后，原来的一个进程就一分为二，一个是父进程，一个是新创建的子进程。

在UNIX里实现fork花费的时间比较长，linux对fork进行了改善。Linux是怎么改善的呢？Linux使用了一个技术，叫做写时复制技术(Copy-On-Write)。

COW（写时复制）

在fork之后exec之前父进程和子进程用的是相同的物理空间（内存区），子进程的代码段、数据段、堆栈都是指向父进程的物理空间。两者的虚拟空间不同，但对应的物理空间是同一个。当父子进程中有更改相识段的行为发生时，再为子进程相应的段分配物理空间。如果更改行为不是因为执行exec，内核会给子进程的数据段、堆栈段分配相应的物理空间，而代码段继续共享父进程的物理空间（两者的代码完全相同）。如果是因为执行exec，由于两者的代码不同，子进程的代码段也会分配单独的物理空间。

还有个细节问题，fork之后内核会将子进程放在队列的前面，以让子进程先执行，以免父进程更改数据导致写时复制，而后子进程执行exec系统调用，之前的数据复制就是无意义的。





# process control

```c
#include <unistd.h>

pid_t getpid(void);  // Returns: process ID of calling process
pid_t getppid(void);  // Returns: parent process ID of calling process
uid_t getuid(void);  // Returns: real user ID of calling process
uid_t geteuid(void);  // Returns: effective user ID of calling process
gid_t getgid(void);  // Returns: real group ID of calling process
gid_t getegid(void);  // Returns: effective group ID of calling process

pid_t fork(void);  // Returns: 0 in child, process ID of child in parent, -1 on error
pid_t vfork(void);  // Returns: 0 in child, process ID of child in parent, -1 on error

#include <sys/wait.h>
pid_t wait(int *statloc);
pid_t waitpid(pid_t pid, int *statloc, int options);
// Both return: process ID if OK, 0(see later), or -1 on error

int execl(const char *pathname, const char *arg0, ... /* (char *)0 */);
int execv(const char *pathname, char *const argv[]);
int execle(const char *pathname, const char *arg0, ... /* (char *)0, char *const envp[] */);
int execve(const char *pathname, char *const argv[], char *const envp[]);
int execlp(const char *filename, const char *arg0, ... /* (char *)0 */);
int execvp(const char *filename, char *const argv[]);
// All six return: -1 on error, no return on success

int setuid(uid_t uid);
int setgid(gid_t gid);
// Both return: 0 if OK, -1 on error
```



## process identifiers

Although unique, process IDs are reused. Most UNIX systems implement algorithms to delay reuse however, so that newly created processes are assigned IDs different from those used by processes that terminated recently.

Process ID 0 is usually the scheduler process and is often known as the swapper.  No program on disk corresponds to this process, which is part of the kernel and is known as a system process.

Process ID 1 is usually the init process and is invoked by the kernel at the end of the bootstrap procedure. init usually reads the files in /etc/init.d-and is a normal user process, not a system process within the kernel, like the swapper, init becomes the parent process of any orpaned child process.

## fork Function

This function is called once but returns twice. 这样做的原因是一个进程可以有多个子进程，但并没有函数可以获取子进程的ID。而每个进程都可以调用getppid获取父进程的ID，因此子进程中fork调用返回0。

Both the child and the parent continue executing with the instruction that follows the call to fork.

The child is a copy of the parent. For example, the child gets a copy of the parent's data space, heap, and stack. The parent and the child share the text segment.

Current implementations don't perform a complete copy of the parent's data, stack, and heap, since a fork is often followed by an exec. A technique called copy-on-write is used. These regions are shared by the parent and child and have their protection chaned by the kernel to read-only. If either process tries to modify these regions, the kernel then makes a copy of that piece of memory only, typically a "page" in a virtual memory system.

有关线程的讨论，参见后续章节。



## User IDs

在Linux中，每个文件都有其所属的用户和用户组，默认情况下是文件的创建者，也可以根据chown和chgrp来修改文件所属的用户和用户组。文件的属性存放在属性结构stat中，其中有st_uid和st_gid标志着。

先举个例子。假设现在系统中有两个用户liz和hlf，有一个程序文件file的所属用户为hlf。然后使用liz用户登录系统，运行file文件，则运行进程process有一个实际用户和有效用户，实际用户默认为当前登录用户，即liz。而有效用户呢？有效用户是哪个得看file文件的属性了，属性结构stat里有个st_mode文件模式字，其中有一个设置-用户-ID位，如果没有设置这个位的话，那任何执行file文件的进程的有效用户就是其实际用户；如果设置了这个位的话，则执行file文件的进程的有效用户就是该file文件所属用户了。这里，如果设置了该位，则有效用户就为hlf，否则就为实际用户liz了。

这里需要说明实际用户ID和有效用户ID的作用。**实际用户是用来标识进程是谁，是谁在执行进程**，一般是登录用户；而**有效用户ID则标识这该进程的访问权限**，假设进程的实际用户为liz，而有效用户为hlf，则进程可以可以访问hlf用户可以访问的文件，即拥有与hlf一样的权限（注意不是能够访问hlf的文件）。

## setuid

There are rules for who can change th IDs. 

1. If the process has superuser privileges, the setuid function sets the real user ID, effective user ID, and saved set-user-ID to uid.
2. If the process does not have superuser privileges, but **uid equals either the real user ID or the saved set-user-ID**, setuid sets only the effective user ID to uid.
3. If neither of these two condition is true, errno is set to EPERM, and -1 to returned.

We can make a few statements about the three user IDs that the kernel maintains.

1. Only a superuser process can change the real user ID. Normally, the real user ID is set by the login(1) program when we log in and never changes. Because login is a superuser process, it sets all three user IDs when it calls setuid.
2. The effective user ID is set by the exec functions only if **the set-user-ID bit is set for the program file**. If the set-user-ID bit is not set, the exec funtions leave the effective user ID as its current value. We can call setuid at any time to set the effective  user ID to either the real user ID or the saved set-user-ID. Naturally, we can't set the effective user ID to any random value.
3. The saved set-user-ID is copied from the effective user ID by exec. If the file's set-user-ID bit is set, this copy is saved after exec stores the effective user ID from the file's user ID.