## 1. MPI 基础概念 | MPI Basic Concepts

### 什么是 MPI? | What is MPI?

MPI (Message Passing Interface) 是一个用于并行计算的库，主要用于 C 语言程序中。它提供了必要的函数和常量，使我们能够将<font color="#7030a0">串行代码转换为并行代码</font>，并且让进程之间能够更容易地进行通信。

MPI is a library that we can use in C programs. It provides the necessary functions and constants to <font color="#7030a0">allow us to convert serial code to parallel</font>. Primarily, it enables processes to communicate with each other much more easily.

### MPI 标准 | MPI Standard

MPI 是关于如何在并行编程中实现消息传递的定义。标准描述了以下内容：

- 发送消息的协议
- 函数调用的语法
- 发送信息时可以使用的数据类型

The MPI standard describes the following:

- The protocol for sending messages
- The syntax of the function calls
- Some data types that can be used when sending information

### MPI 实现 | MPI Implementation

Open MPI 是 MPI 的一个开源实现，可以安装在大多数计算机上，但有时候安装和正确设置可能很困难。在课程中，我们将通过已经安装了 MPI 的服务器（beckett4.ucd.ie）来使用它。

Open MPI is an open-source implementation available at [https://www.open-mpi.org/](https://www.open-mpi.org/). Although it can be installed on most computers, it can sometimes be difficult to install and set up correctly. In this course, we will use it through a server that already has it installed (beckett4.ucd.ie).

## 2. 使用 MPI | Using MPI

将程序更改为使用 MPI 需要以下几个步骤：

1. **包含必要的头文件** - `mpi.h`
2. **初始化 MPI 环境** - `MPI_Init`
3. **获取有关进程的信息** - `MPI_Comm_size`/`MPI_Comm_rank`
4. **通信** - `MPI_Send` / `MPI_Recv`
5. **清理 MPI 环境** - `MPI_Finalize`

To change a program to use MPI, we must follow these steps:

1. Include the required header file - `mpi.h`
2. Initialize the MPI environment - `MPI_Init`
3. Find out about the processes - `MPI_Comm_size`/`MPI_Comm_rank`
4. Communicate - `MPI_Send` / `MPI_Recv`
5. Clean up the MPI environment - `MPI_Finalize`

### 基本程序示例（无通信） | Basic Example (No Communication)

```C
\#include <mpi.h>
\#include <stdio.h>

int main(int argc, char **argv) {
    // 存储进程总数的变量
    int procCount;
    // 存储当前进程编号的变量
    int procNum;

    // 将命令行参数传递给 MPI
    MPI_Init(&argc, &argv);

    // 询问进程总数
    //对于第一个，我们将使用 MPI 库 MPI_COMM_WORLD 提供的常量 
    //对于第二个，必须创建一个 int 变量并传递一个指针作为形参
    MPI_Comm_size(MPI_COMM_WORLD, &procCount);
    // 询问当前进程的编号
    MPI_Comm_rank(MPI_COMM_WORLD, &procNum);

    // 每个进程将打印以下消息
    printf("Hello, I am process %d out of %d\n", procNum, procCount);

    // 结束 MPI 环境
    MPI_Finalize();
    return 0;
}
```

这个例子展示了 MPI 的基本结构，但没有进程间通信。每个进程只是打印出自己的编号和总进程数。

This example shows the basic structure of an MPI program without inter-process communication. Each process simply prints out its own number and the total number of processes.

## 3. 编译和运行 MPI 程序 | Compiling and Executing MPI Programs

### 编译 MPI 程序 | Compiling MPI Programs

使用 `mpicc` 命令编译 MPI 程序：

```Shell
mpicc program.c -o program
```

### 运行 MPI 程序 | Executing MPI Programs

使用 `mpirun` 命令运行 MPI 程序：

```Shell
mpirun -np 4 ./program
```

其中 `-np 4` 指定要启动的进程数量（本例中为 4 个）。

Use the `mpirun` command to execute MPI programs:

```Shell
mpirun -np 4 ./program
```

where `-np 4` specifies the number of processes to start (4 in this example).

## 4. MPI 通信 | MPI Communication

### 点对点通信 | Point-to-Point Communication

点对点通信是在两个进程之间进行的通信，主要使用 `MPI_Send()` 和 `MPI_Recv()` 函数。

Point-to-point communication occurs between two processes using primarily the `MPI_Send()` and `MPI_Recv()`functions.

### MPI_Send 函数

**1. `MPI_Send` - 寄送包裹**

想象一下你正在寄送一个包裹，你需要告诉快递员一些信息：

- `void *buf`: 你要寄送的**东西**是什么？（包裹里的**内容**的起始位置，`发送缓冲区起始地址`）
- `int count`: 你要寄送多少**件**这样的东西？（包裹里物品的**数量**，`要发送的元素数量`）
- `MPI_Datatype datatype`: 这些东西是**什么类型**的？（是文件、书本、还是其他，`发送数据类型`）
- `int dest`: 这个包裹要寄给**谁**？（收件人的**地址**，即目标进程的 rank，`目标进程的rank`）
- `int tag`: 你想在这个包裹上贴一个**特殊的标记**或编号，方便收件人识别这是哪个包裹吗？（`消息标签`）
- `MPI_Comm comm`: 你使用的是**哪个快递公司**或邮政系统？（确保寄件人和收件人都在同一个系统里，`通信域`）

所以，记忆 `MPI_Send` 就是记住： 把 **(什么数据)** 寄送给 **(谁)** 带上 **(什么标记)** 在 **(哪个通信网络)** 上。

`buf`, `count`, `datatype` (什么数据) | `dest` (给谁) | `tag` (标记) | `comm` (网络)

```C
int MPI_Send(
    void* data,           // 要发送的数据的指针
    int count,            // 要发送的数据项数量
    MPI_Datatype datatype,// 发送数据的类型
    int destination,      // 目标进程的编号
    int tag,              // 消息标签（用于区分不同消息）
    MPI_Comm communicator // 通信器（通常是 MPI_COMM_WORLD）
)


//该函数将向编号为 0 的进程发送一个 int 值
int someNumber = result_of_calculation();
MPI_Send( &someNumber, 1, MPI_INT, 0, MPI_ANY_TAG, MPI_COMM_WORLD);

//例子：发送多个 int
int data[3];
data[0] = result_of_calculation();
data[1] = result_of_other_calculation();
data[2] = result_of_last_calculation();
MPI_Send( &data, 3, MPI_INT, 0, MPI_ANY_TAG, MPI_COMM_WORLD);
```

### MPI_Recv 函数
- `void *buf`: 你准备把收到的东西放**在哪里**？（一个准备好的空间，`接收缓冲区起始地址`）
- `int count`: 你准备的这个空间**最大能放多少**东西？（为了不撑爆你的空间，`最大可接收元素数量`）
- `MPI_Datatype datatype`: 你期望收到**什么类型**的东西？（`接收数据类型`）
- `int source`: 你期望这个包裹是**从谁那里寄来**的？（寄件人的**地址**，即源进程的 rank，或者你可以说**“随便谁寄来的都行”**，`源进程的rank（或MPI_ANY_SOURCE）`）
- `int tag`: 你期望收到的包裹的**标记**是什么？（或者你可以说**“任何标记的包裹都收”**，`消息标签（或MPI_ANY_TAG）`）
- `MPI_Comm comm`: 你在**哪个快递公司**或邮政系统那里等待包裹？（`通信域`）
- `MPI_Status *status`: 包裹收到后，请**告诉我一些关于它的详细信息**，比如实际是从谁寄来的、实际的标记是什么、实际收到了多少件东西等等。（`状态对象`，这是一个输出参数，MPI 会填写信息）
```C
int MPI_Recv(
    void* buff,           // 接收数据的缓冲区
    int count,            // 最大可接收的数据项数量
    MPI_Datatype datatype,// 接收数据的类型
    int source,           // 源进程的编号
    int tag,              // 消息标签
    MPI_Comm communicator,// 通信器
    MPI_Status* status    // 接收状态信息
)

int value;
MPI_Recv( &value, 1, MPI_INT, 1, MPI_ANY_TAG, MPI_COMM_WORLD,MPI_STATUS_IGNORE); 
This function will receive a single int value from the process numbered 1

例子：发送多个 int
int data[3];
MPI_Recv( data, 3, MPI_INT, 1, MPI_ANY_TAG, MPI_COMM_WORLD, MPI_STATUS_IGNORE);
该函数将从编号为 1 的进程接受三个 int 值
```

### Sending and Receiving 例子

```C
\#include <stdio.h>
\#include <stdlib.h>
\#include <time.h>
\#include <mpi.h>

int main(int argc, char **argv) {
    int procCount;
    int procNum;
	//MPI初始化-过程信息
	//初始化MPI环境，确定进程总数，并为进程分配一个唯一ID
    MPI_Init(&argc, &argv);
    MPI_Comm_size(MPI_COMM_WORLD, &procCount);
    MPI_Comm_rank(MPI_COMM_WORLD, &procNum);
	//使用当前系统时间time（NULL）加上进程排名procNum
	//为随机数生成器设置种子生成一个介于0和999之间的随机数
	//打印进程id及其生成的数字
    srand(time(NULL) + procNum);
    int r = rand() % 1000;
    printf("Process %d generated the number %d\n", procNum, r);

    int temp;
    if (procNum == 1) {
        MPI_Send(&r, 1, MPI_INT, 0, 0, MPI_COMM_WORLD);
        printf("Process %d is sending the number %d to 0\n", procNum, r);
    } else if (procNum == 0) {
        MPI_Recv(&temp, 1, MPI_INT, 1, 0, MPI_COMM_WORLD, MPI_STATUS_IGNORE);
        printf("Received %d from process 1\n", temp);
        r = r + temp;
        printf("The sum of the numbers was %d\n", r);
    }

    MPI_Finalize();
    return 0;
}
```

## 7. 考试准备 | Exam Preparation

根据 10-1 PPT，考试将会涵盖 MPI 相关内容，特别是点对点通信，包括 MPI_Send() 和 MPI_Recv() 函数以及编译和执行命令。重点准备以下内容：

1. MPI 的基本概念和用途
2. MPI 程序的基本结构和初始化过程
3. 点对点通信函数的参数和用法
4. 如何编译和运行 MPI 程序
5. MPI 与 OpenMP 的区别和适用场景

Based on the 10-1 PPT, the exam will cover MPI-related content, especially point-to-point communication including MPI_Send() and MPI_Recv() functions and compile/execution commands. Focus on preparing the following:

1. Basic concepts and purposes of MPI
2. Basic structure and initialization process of MPI programs
3. Parameters and usage of point-to-point communication functions
4. How to compile and run MPI programs
5. Differences between MPI and OpenMP and their application scenarios

希望这些笔记对你的期末考试准备有所帮助！如果有任何问题，请随时提问。

I hope these notes will help with your final exam preparation! If you have any questions, please feel free to ask.