---
title: Linux 下 wait_pid 和 vfork 使用
date: 2020-12-08 21:14:48
tags:
	- Linux
categories:
	- Linux
description:
	- Linux 下使用 vfork 创建子进程，waitpid 获取子进程返回状态
---
## 调用 vfork 创建子进程
```c
pid_t pid = vfork();
if (pid < 0)
{
    perror("vfork failed.");
}
else if (pid == 0)
{
    printf("this is parent process.");
}
else if (pid > 0)
{
    printf("this is child process.");
}
```
vfork 用于创建一个新进程，和 fork 不同， vfork 不将父进程的地址空间完全复制到子进程中，不会复制页表
vfork 执行一次会返回两个返回值
1. 在子进程中返回0
2. 在父进程中返回子进程 pid_t
执行失败返回 -1

## 调用 waitpid 等待子进程结束

```c
waitpid(pid, &status, 0);
if (WIFEXITED(status))
{
    printf("child process exit normally, return code is %d.", WEXITSTATUS(status));
}
else if (WIFSIGNALED(status))
{
    printf("child process exit abnormally, return code is %d.", WTERMSIG(status));
}
```
waitpid() 会暂时停止目前程序的执行，知道有信号来到或子进程结束
其中 pid 为子进程 pid, status 为要返回的子进程的结束状态
有如下几个宏可以用来判别结束情况

- `WIFEXITED(status)` 如果子进程正常结束则为非 0 值。
- `WEXITSTATUS(status)` 取得子进程 exit()返回的结束代码,一般会先用 WIFEXITED 来判断是否正常结束才能使用此宏。
- `WIFSIGNALED(status)` 如果子进程是因为信号而结束则此宏值为真
- `WTERMSIG(status`) 取得子进程因信号而中止的信号代码,一般会先用 WIFSIGNALED 来判断后才使用此宏。
- `WIFSTOPPED(status)` 如果子进程处于暂停执行情况则此宏值为真。一般只有使用 WUNTRACED 时才会有此情况。
- `WSTOPSIG(status)` 取得引发子进程暂停的信号代码,一般会先用 WIFSTOPPED 来判断后才使用此宏。
- 如果执行成功则返回子进程识别码(PID) ,如果有错误发生则返回返回值-1。失败原因存于 errno 中。