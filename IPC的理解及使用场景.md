### IPC(inter process communication) 是指不同间进程的信息交换或传播
  * #### 管道方式
  * #### 消息队列
  * #### 信号量
  * #### 共享存储
  * #### unix domain socket
  * #### socket(不同主机)
  * #### streams(不同主机)
* ### 管道
  通常指无名管道，是unix系统IPC最古老的形式
  * #### 特点：
    * #### 半双工（即数据只能在一个方向上流动）具有固定的读端和写端。
    * #### 他只能用于具有沁园关系的进程之间的通信（也就是父子进程或者兄弟进程）。
    * #### 他可以看成是一种特殊的文件，对于他的读写也可以使用普通的read,write等函数。但是他不是普通的文件，并不属于其他任何文件系统，并且只存在于内存中
  * #### 原型
    ```
    #include <unistd.h>
    int pipe(int fd[2]);    // 返回值：若成功返回0，失败返回-1
    ```
    当一个管道建立时，它会创建两个文件描述符：fd[0]为读而打开，fd[1]为写而打开。如下图：
    ![](http://img.blog.csdn.net/20150419222058628?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbGlzb25nbGlzb25nbGlzb25n/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)
    关闭管道只需将这两个文件描述符关闭即可
    * #### 例子
      单个进程中的管道几乎没有任何用处。所以，通常调用 pipe 的进程接着调用 fork，这样就创建了父进程与子进程之间的 IPC 通道。如下图所示：
      ![](http://img.blog.csdn.net/20150419223853807?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbGlzb25nbGlzb25nbGlzb25n/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

      若要数据流从父进程流向子进程，则关闭父进程的读端（fd[0]）与子进程的写端（fd[1]）；反之，则可以使数据流从子进程流向父进程。
      ```
      #include<stdio.h>
      #include<unistd.h>

      int main() {
        int fd[2]; //两个文件描述符
        pid_t pid;
        char buff[20];

        if(pipe(fd) < 0) //创建管道
          printf("Create Pipe Error!\n");
        if ((pid = fork()) < 0>) //创建子进程
          printf("Fork Error!\n");
        else if (pid > 0) // 父进程
        {
          close(fd[0]); //关闭读端
          write(fd[1], "hello world\n", 12);
        } else {
          close(fd[1]); //关闭写端
          read(fd[0], buf, 20);
          printf("%s", buff);
        }
      }
      ```
      * #### node的管道例子