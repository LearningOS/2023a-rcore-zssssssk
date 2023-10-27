# Lab1

## sys_task_info功能实现

- struct TaskControlBlock中加入三个公共字段start_time、time、syscall_times，其中start_time记录了task的开始执行时间，剩下两个与TaskInfo结构体里的内容一致

- 每个task的start_time字段会在第一次run的时候初始化

- 每个task的time和syscall_times字段会在系统调用的时候进行计算，我选择在trap_handler里的Exception::UserEnvCall中计算，因为之后就是调用用户库的syscall了。传入一个syscall_id即可对这两个字段进行计算

- 之后在task/mod.rs中暴露对外的接口即可

- 最后返回0


### 问答题

1.[rustsbi] RustSBI version 0.3.0-alpha.2, adapting to RISC-V SBI v1.0.0

[kernel] PageFault in application, bad addr = 0x0, bad instruction = 0x804003c4, kernel killed it.
[kernel] IllegalInstruction in application, kernel killed it.
[kernel] IllegalInstruction in application, kernel killed it.

(0x0 as *mut u8).write_volatile(0)这段代码会尝试对地址0x0进行写操作。然而，地址0x0通常是一个无效的内存地址，操作系统并没有为其分配任何内存。

在用户态使用core::arch::asm!("sret")会导致错误，因为sret是一个特权指令，它用于从特权模式返回到用户模式。

`core::arch::asm!("csrr {}, sstatus", out(reg) sstatus)`是Rust的内联汇编代码，用于读取RISC-V架构下的`sstatus`寄存器的值。`sstatus`寄存器是特权级寄存器，只能在特权级模式下访问。在用户态下尝试访问这些寄存器，会触发异常，因为用户态程序通常不被允许直接访问硬件或者执行某些特权指令。

2.

1. a0代表传入的第一个参数。应用程序执行之前的准备与异常处理完回到应用程序会用到
2. 
   - sstatus：记录进入Trap之前cpu处于哪个特权级
   - sepc：记录Trap发生之前执行的最后一条指令的地址
   - sscratch：记录用户区的栈顶指针

3.`x2` 是栈指针（Stack Pointer），`x4` 是线程指针（Thread Pointer）。这些寄存器通常被操作系统或运行时环境用于特殊目的，因此在一般的应用程序代码中可能会避免直接使用它们。

4.sp指向用户区的栈顶，sscratch指向内核区的栈顶

5.sret。`sret` 是 RISC-V 指令集中的一条特权级指令，用于从异常或中断处理程序返回。它会从 `sepc` 寄存器加载程序计数器（PC），并从 `sstatus` 寄存器恢复状态位，然后跳转到 `sepc` 所指向的地址继续执行。

在这个过程中，`sstatus` 寄存器的 SPP（之前的特权级）位被清零，这意味着处理器将切换到用户模式。因此，当 `sret` 指令执行后，处理器将进入用户态，并开始执行用户程序。

6.sp指向内核区的栈顶，sscratch指向用户区的栈顶

### **荣誉准则**

1. 在完成本次实验的过程（含此前学习的过程）中，我曾分别与 **以下各位** 就（与本次实验相关的）以下方面做过交流，还在代码中对应的位置以注释形式记录了具体的交流对象及内容：

   > 无

2. 此外，我也参考了 **以下资料** ，还在代码中对应的位置以注释形式记录了具体的参考来源及内容：

   > rCore-Tutorial-Book 第三版（主要参考了第三章练习答案）

3. 我独立完成了本次实验除以上方面之外的所有工作，包括代码与文档。 我清楚地知道，从以上方面获得的信息在一定程度上降低了实验难度，可能会影响起评分。

4. 我从未使用过他人的代码，不管是原封不动地复制，还是经过了某些等价转换。 我未曾也不会向他人（含此后各届同学）复制或公开我的实验代码，我有义务妥善保管好它们。 我提交至本实验的评测系统的代码，均无意于破坏或妨碍任何计算机系统的正常运转。 我清楚地知道，以上情况均为本课程纪律所禁止，若违反，对应的实验成绩将按“-100”分计。