使用`kill -l`命令可查看系统支持的信号列表。

The kill(1) command and the kill(2) function just send a signal to a process or process group. Whether or not that signal terminates the process depends on which signal is sent and whether the process has arranged to catch the signal.

When a program is executed, all signals are set to their default action, unless the process that calls exec is ignoring the signal. Specifically(确切地说), the exec functions change the disposition of any signals being caught to their default action and leave the status of all other signals alone. (Naturally, a signal that is being caught by a process that calls exec cannot be caught by the same function in the new program, since the address of the signals for function in the caller probably has no meaning in the new program file that is executed).

## pause

```c
#include <unistd.h>
int pause(void);
```

pause()系统调用可使进程睡眠，直到进程接收到处理或终止进程的信号。