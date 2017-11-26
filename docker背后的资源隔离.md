| Namespace | 系统调用参数 | 隔离内容 |
| ------------- |:-------------:| -----:|
| UTS(UNIX Timesharing System)      | CLONE_NEWUTS       | 主机名与域名 |
| IPC      | CLONE_NEWIPC       | 信号量、消息队列和共享内存 |
| PID      | CLONE_NEWPID       | 进程编号 |
| Network  | CLONE_NEWNET       | 网络设备、网络栈、端口等等 |
| Mount    | CLONE_NEWNS        | 挂载点（文件系统）|
| User     | CLONE_NEWUSER      | 用户和用户组 |