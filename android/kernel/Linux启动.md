[TOC]

# Linux启动

## 0号进程
0号进程，通常也被称为idle进程，或者也称为swapper进程。

0号进程是linux启动的第一个进程，它的task_struct的comm字段为"swapper",所以也成为swpper进程。

    #define INIT_TASK_COMM "swapper"

当系统中所有的进程起来后，0号进程也就蜕化为idle进程，当一个core上没有任务可运行时就会去运行idle进程。一旦运行idle进程则此core就可以进入低功耗模式了，在ARM上就是WFI。

我们本节重点关注是0号进程是如何启动的。在linux内核中为0号进程专门定义了一个静态的 task_struct 的结构，称为init_task。

```C
/*
 * Set up the first task table, touch at your own risk!. Base=0,
 * limit=0x1fffff (=2MB)
 */
struct task_struct init_task
= {
#ifdef CONFIG_THREAD_INFO_IN_TASK
    .thread_info    = INIT_THREAD_INFO(init_task),
    .stack_refcount    = ATOMIC_INIT(1),
#endif
    .state        = 0,
    .stack        = init_stack,
    .usage        = ATOMIC_INIT(2),
    .flags        = PF_KTHREAD,
    .prio        = MAX_PRIO - 20,
    .static_prio    = MAX_PRIO - 20,
    .normal_prio    = MAX_PRIO - 20,
    .policy        = SCHED_NORMAL,
    .cpus_allowed    = CPU_MASK_ALL,
    .nr_cpus_allowed= NR_CPUS,
    .mm        = NULL,
    .active_mm    = &init_mm,
    .tasks        = LIST_HEAD_INIT(init_task.tasks),
    .ptraced    = LIST_HEAD_INIT(init_task.ptraced),
    .ptrace_entry    = LIST_HEAD_INIT(init_task.ptrace_entry),
    .real_parent    = &init_task,
    .parent        = &init_task,
    .children    = LIST_HEAD_INIT(init_task.children),
    .sibling    = LIST_HEAD_INIT(init_task.sibling),
    .group_leader    = &init_task,
    RCU_POINTER_INITIALIZER(real_cred, &init_cred),
    RCU_POINTER_INITIALIZER(cred, &init_cred),
    .comm        = INIT_TASK_COMM,
    .thread        = INIT_THREAD,
    .fs        = &init_fs,
    .files        = &init_files,
    .signal        = &init_signals,
    .sighand    = &init_sighand,
    .blocked    = {{0}},
    .alloc_lock    = __SPIN_LOCK_UNLOCKED(init_task.alloc_lock),
    .journal_info    = NULL,
    INIT_CPU_TIMERS(init_task)
    .pi_lock    = __RAW_SPIN_LOCK_UNLOCKED(init_task.pi_lock),
    .timer_slack_ns = 50000, /* 50 usec default slack */
    .thread_pid    = &init_struct_pid,
    .thread_group    = LIST_HEAD_INIT(init_task.thread_group),
    .thread_node    = LIST_HEAD_INIT(init_signals.thread_head),
};
EXPORT_SYMBOL(init_task);
```

这个结构体中的成员都是静态定义了，为了简单说明，对这个结构做了简单的删减。同时我们只关注这个结构中的以下几个字段，别的先不关注。

```C
.thread_info    = INIT_THREAD_INFO(init_task),      这个结构在thread_info和内核栈的关系中有详细的描述
.stack        = init_stack,             init_stack就是内核栈的静态的定义
.comm        = INIT_TASK_COMM,  0号进程的名称。
```

在这么thread_info和stack都涉及到了Init_stack， 所以先看下init_stack在哪里设置的。

最终发现init_task是在链接脚本中定义的。

```C
#define INIT_TASK_DATA(align)                        \
    . = ALIGN(align);                        \
    __start_init_task = .;                        \
    init_thread_union = .;                        \
    init_stack = .;                            \
    KEEP(*(.data..init_task))                    \
    KEEP(*(.data..init_thread_info))                \
    . = __start_init_task + THREAD_SIZE;                \
    __end_init_task = .;
```

在链接脚本中定义了一个INIT_TASK_DATA的宏。

其中__start_init_task 就是0号进程的内核栈的基地址，当然了
init_thread_union = init_task = __start_init_task

而0号进程的内核栈的结束地址等于 __start_init_task + THREAD_SIZE， THREAD_SIZE 的大小ARM64一般是16K，或者32K。则 __end_init_task 就是0号进程的内核栈的结束地址。


## Linux内核的启动
熟悉linux内核的朋友都知道，linux内核的启动 ，一般都是有bootloader来完成装载，bootloader中会做一些硬件的初始化，然后会跳转到linux内核的运行地址上去。

如果熟悉ARM架构的盆友也清楚，ARM64架构分为EL0, EL1, EL2, EL3。正常的启动一般是从高特权模式向低特权模式启动的。通常来说ARM64是先运行EL3，再EL2，然后从EL2就trap到EL1，也就是我们的Linux内核。

我们来看下Linux内核启动的代码。

代码路径：arch/arm64/kernel/head.S 文件中
```C
/*
 * Kernel startup entry point.
 * ---------------------------
 *
 * The requirements are:
 *   MMU = off, D-cache = off, I-cache = on or off,
 *   x0 = physical address to the FDT blob.
 *
 * This code is mostly position independent so you call this at
 * __pa(PAGE_OFFSET + TEXT_OFFSET).
 *
 * Note that the callee-saved registers are used for storing variables
 * that are useful before the MMU is enabled. The allocations are described
 * in the entry routines.
 */
 
    /*
     * The following callee saved general purpose registers are used on the
     * primary lowlevel boot path:
     *
     *  Register   Scope                      Purpose
     *  x21        stext() .. start_kernel()  FDT pointer passed at boot in x0
     *  x23        stext() .. start_kernel()  physical misalignment/KASLR offset
     *  x28        __create_page_tables()     callee preserved temp register
     *  x19/x20    __primary_switch()         callee preserved temp registers
     */
ENTRY(stext)
    bl    preserve_boot_args
    bl    el2_setup            // Drop to EL1, w0=cpu_boot_mode
    adrp    x23, __PHYS_OFFSET
    and    x23, x23, MIN_KIMG_ALIGN - 1    // KASLR offset, defaults to 0
    bl    set_cpu_boot_mode_flag
    bl    __create_page_tables
    /*
     * The following calls CPU setup code, see arch/arm64/mm/proc.S for
     * details.
     * On return, the CPU will be ready for the MMU to be turned on and
     * the TCR will have been set.
     */
    bl    __cpu_setup            // initialise processor
    b    __primary_switch
ENDPROC(stext)
```

上面就是内核在调用 start_kernel 之前做的主要工作了。

preserve_boot_args
用来保留bootloader传递的参数，比如ARM上通常的dtb的地址

el2_setup
从注释上来看是， 用来trap到EL1，说明我们在运行此指令前还在EL2

__create_page_tables
用来创建页表，linux才有的是页面管理物理内存的，在使用虚拟地址之前需要设置好页面，然后会打开MMU。
目前还是运行在物理地址上的

__primary_switch
主要任务是完成MMU的打开工作
```C
__primary_switch:
    adrp    x1, init_pg_dir
    bl    __enable_mmu
    ldr    x8, =__primary_switched
    adrp    x0, __PHYS_OFFSET
    br    x8
ENDPROC(__primary_switch)
```
主要是调用 __enable_mmu 来打开mmu，之后我们访问的就是虚拟地址了

调用 __primary_switched 来设置0号进程的运行内核栈，然后调用 start_kernel 函数
```C
/*
 * The following fragment of code is executed with the MMU enabled.
 *
 *   x0 = __PHYS_OFFSET
 */
__primary_switched:
    adrp    x4, init_thread_union
    add    sp, x4, #THREAD_SIZE
    adr_l    x5, init_task
    msr    sp_el0, x5            // Save thread_info
 
    adr_l    x8, vectors            // load VBAR_EL1 with virtual
    msr    vbar_el1, x8            // vector table address
    isb
 
    stp    xzr, x30, [sp, #-16]!
    mov    x29, sp
 
    str_l    x21, __fdt_pointer, x5        // Save FDT pointer
 
    ldr_l    x4, kimage_vaddr        // Save the offset between
    sub    x4, x4, x0            // the kernel virtual and
    str_l    x4, kimage_voffset, x5        // physical mappings
 
    // Clear BSS
    adr_l    x0, __bss_start
    mov    x1, xzr
    adr_l    x2, __bss_stop
    sub    x2, x2, x0
    bl    __pi_memset
    dsb    ishst                // Make zero page visible to PTW
 
    add    sp, sp, #16
    mov    x29, #0
    mov    x30, #0
    b    start_kernel
ENDPROC(__primary_switched)
```

init_thread_union 就是我们在链接脚本中定义的，也就是0号进程的内核栈的栈底
add    sp, x4, #THREAD_SIZE： 设置堆栈指针SP的值，就是内核栈的栈底+THREAD_SIZE的大小。现在SP指到了内核栈的顶端
最终通过b start_kernel就跳转到我们熟悉的linux内核入口处了。
至此0号进程就已经运行起来了。


## 1号进程
当一条b start_kernel指令运行后，内核就开始的内核的全面初始化操作

```C
asmlinkage __visible void __init start_kernel(void)
{
    char *command_line;
    char *after_dashes;
 
    set_task_stack_end_magic(&init_task);
    smp_setup_processor_id();
    debug_objects_early_init();
 
    cgroup_init_early();
 
    local_irq_disable();
    early_boot_irqs_disabled = true;
 
    /*
     * Interrupts are still disabled. Do necessary setups, then
     * enable them.
     */
    boot_cpu_init();
    page_address_init();
    pr_notice("%s", linux_banner);
    setup_arch(&command_line);
    /*
     * Set up the the initial canary and entropy after arch
     * and after adding latent and command line entropy.
     */
    add_latent_entropy();
    add_device_randomness(command_line, strlen(command_line));
    boot_init_stack_canary();
    mm_init_cpumask(&init_mm);
    setup_command_line(command_line);
    setup_nr_cpu_ids();
    setup_per_cpu_areas();
    smp_prepare_boot_cpu();    /* arch-specific boot-cpu hooks */
    boot_cpu_hotplug_init();
 
    build_all_zonelists(NULL);
    page_alloc_init();
    。。。。。。。
    acpi_subsystem_init();
    arch_post_acpi_subsys_init();
    sfi_init_late();
 
    /* Do the rest non-__init'ed, we're now alive */
    arch_call_rest_init();
}
 
void __init __weak arch_call_rest_init(void)
{
    rest_init();

start_kernel 函数就是内核各个重要子系统的初始化，比如mm, cpu, sched, irq等等。
最后会调用一个 rest_init 剩余部分初始化

noinline void __ref rest_init(void)
{
    struct task_struct *tsk;
    int pid;
 
    rcu_scheduler_starting();
    /*
     * We need to spawn init first so that it obtains pid 1, however
     * the init task will end up wanting to create kthreads, which, if
     * we schedule it before we create kthreadd, will OOPS.
     */
    pid = kernel_thread(kernel_init, NULL, CLONE_FS);
    /*
     * Pin init on the boot CPU. Task migration is not properly working
     * until sched_init_smp() has been run. It will set the allowed
     * CPUs for init to the non isolated CPUs.
     */
    rcu_read_lock();
    tsk = find_task_by_pid_ns(pid, &init_pid_ns);
    set_cpus_allowed_ptr(tsk, cpumask_of(smp_processor_id()));
    rcu_read_unlock();
 
    numa_default_policy();
    pid = kernel_thread(kthreadd, NULL, CLONE_FS | CLONE_FILES);
    rcu_read_lock();
    kthreadd_task = find_task_by_pid_ns(pid, &init_pid_ns);
    rcu_read_unlock();
 
    /*
     * Enable might_sleep() and smp_processor_id() checks.
     * They cannot be enabled earlier because with CONFIG_PREEMPT=y
     * kernel_thread() would trigger might_sleep() splats. With
     * CONFIG_PREEMPT_VOLUNTARY=y the init task might have scheduled
     * already, but it's stuck on the kthreadd_done completion.
     */
    system_state = SYSTEM_SCHEDULING;
 
    complete(&kthreadd_done);
 
}
```

在这个rest_init函数中我们只关系两点：

```C
pid = kernel_thread(kernel_init, NULL, CLONE_FS);
pid = kernel_thread(kthreadd, NULL, CLONE_FS | CLONE_FILES);
/*
 * Create a kernel thread.
 */
pid_t kernel_thread(int (*fn)(void *), void *arg, unsigned long flags)
{
    return _do_fork(flags|CLONE_VM|CLONE_UNTRACED, (unsigned long)fn,
        (unsigned long)arg, NULL, NULL, 0);
}
```

很明显这是创建了两个内核线程，而 kernel_thread 最终会调用do_fork根据参数的不同来创建一个进程或者内核线程。
关系 do_fork 的实现我们在后面会做详细的介绍。当内核线程创建成功后就会调用设置的回调函数。

当 kernel_thread(kernel_init) 成功返回后，就会调用 kernel_init 内核线程，其实这时候1号进程已经产生了。接下来看下 kernel_init 主要做什么事情

```C
static int __ref kernel_init(void *unused)
{
    int ret;
 
    kernel_init_freeable();
    /* need to finish all async __init code before freeing the memory */
    async_synchronize_full();
    ftrace_free_init_mem();
    free_initmem();
    mark_readonly();
 
    /*
     * Kernel mappings are now finalized - update the userspace page-table
     * to finalize PTI.
     */
    pti_finalize();
 
    system_state = SYSTEM_RUNNING;
    numa_default_policy();
 
    rcu_end_inkernel_boot();
 
    if (ramdisk_execute_command) {
        ret = run_init_process(ramdisk_execute_command);
        if (!ret)
            return 0;
        pr_err("Failed to execute %s (error %d)\n",
               ramdisk_execute_command, ret);
    }
 
    /*
     * We try each of these until one succeeds.
     *
     * The Bourne shell can be used instead of init if we are
     * trying to recover a really broken machine.
     */
    if (execute_command) {
        ret = run_init_process(execute_command);
        if (!ret)
            return 0;
        panic("Requested init %s failed (error %d).",
              execute_command, ret);
    }
    if (!try_to_run_init_process("/sbin/init") ||
        !try_to_run_init_process("/etc/init") ||
        !try_to_run_init_process("/bin/init") ||
        !try_to_run_init_process("/bin/sh"))
        return 0;
 
    panic("No working init found.  Try passing init= option to kernel. "
          "See Linux Documentation/admin-guide/init.rst for guidance.");
}
```

kernel_init_freeable 函数中就会做各种外设驱动的初始化
最主要的工作就是通过execve执行/init可以执行文件。
我们通常将init称为1号进程，其实在刚才kernel_init的时候1号线程已经创建成功，也可以理解kernel_init是1号进程的内核态，而我们所熟知的init进程是用户态的。

至此1号进程就完美的创建成功了，而且也成功执行了init可执行文件。

1. init进程会调用 prepare_namespace() 函数去挂载根文件系统，根文件系统的存储位置由bootcmdline指定
(例如：root=/dev/mmcblk0p2)；
2. 挂载上根文件系统后，去查找应用层的初始化函数，应用层的初始化函数也是通过 bootcmdline 指定
(例如：init=/linuxrc)；
3. init进程调用 run_init_process() 函数去执行应用层的初始化函数，然后init进程就从内核态转变成用户态；
4. 打开控制台 "/dev/console" ，此时得到文件描述符(因为是打开的第一个文件，文件描述符是0)，然后将文件描述符赋值两遍，得到文件描述符0、1、2，也就是标准输出、标准输入、标准出错。
5. init进程构建了交互界面，启动了login进程、命令行进程、shell进程



## 2号进程
2号进程，是由1号进程创建的。而且2号进程是所有内核线程父进程。
内核的守护进程，始终运行在内核空间, 负责全部内核线程的调度和管理。

2号进程就是刚才 rest_init 中创建的另外一个内核线程。
kernel_thread(kthreadd, NULL, CLONE_FS | CLONE_FILES);

当 kernel_thread(kthreadd) 返回时，2号进程已经创建成功了。而且会回调kthreadd函数

```C
int kthreadd(void *unused)
{
    struct task_struct *tsk = current;
 
    /* Setup a clean context for our children to inherit. */
    set_task_comm(tsk, "kthreadd");
    ignore_signals(tsk);
    set_cpus_allowed_ptr(tsk, cpu_all_mask);
    set_mems_allowed(node_states[N_MEMORY]);
 
    current->flags |= PF_NOFREEZE;
    cgroup_init_kthreadd();
 
    for (;;) {
        set_current_state(TASK_INTERRUPTIBLE);
        if (list_empty(&kthread_create_list))
            schedule();
        __set_current_state(TASK_RUNNING);
 
        spin_lock(&kthread_create_lock);
        while (!list_empty(&kthread_create_list)) {
            struct kthread_create_info *create;
 
            create = list_entry(kthread_create_list.next,
                        struct kthread_create_info, list);
            list_del_init(&create->list);
            spin_unlock(&kthread_create_lock);
 
            create_kthread(create);
 
            spin_lock(&kthread_create_lock);
        }
        spin_unlock(&kthread_create_lock);
    }
 
    return 0;
}
```

设置当前进程的名字为 "kthreadd" ，也就是 task_struct 的comm字段
然后就是while循环，设置当前的进程的状态是TASK_INTERRUPTIBLE 是可以中断的
判断 kthread_create_list 链表是不是空，如果是空则就调度出去，让出cpu
如果不是空，则从链表中取出一个，然后调用 kthread_create 去创建一个内核线程。
所以说所有的内核线程的父进程都是2号进程，也就是kthreadd。


## 总结：
1. linux启动的第一个进程是0号进程，是静态创建的
2. 在0号进程启动后会接连创建两个进程，分别是1号进程和2和进程。
3. 1号进程最终会去调用可init可执行文件，init进程最终会去创建所有的应用进程。
4. 2号进程会在内核中负责创建所有的内核线程
5. 所以说0号进程是1号和2号进程的父进程；1号进程是所有用户态进程的父进程；2号进程是所有内核线程的父进程。

我们通过ps命令就可以详细的观察到这一现象。

    root@ubuntu:zhuxl$ ps -eF
    UID         PID   PPID  C    SZ   RSS PSR STIME TTY          TIME CMD
    root          1      0  0 56317  5936   2 Feb16 ?        00:00:04 /sbin/init
    root          2      0  0     0     0   1 Feb16 ?        00:00:00 [kthreadd]

上面很清晰的显示：PID=1的进程是init，PID=2的进程是kthreadd。而他们俩的父进程PPID=0，也就是0号进程。

    UID         PID   PPID  C    SZ   RSS PSR STIME TTY          TIME CMD
    root          4      2  0     0     0   0 Feb16 ?        00:00:00 [kworker/0:0H]
    root          6      2  0     0     0   0 Feb16 ?        00:00:00 [mm_percpu_wq]
    root          7      2  0     0     0   0 Feb16 ?        00:00:10 [ksoftirqd/0]
    root          8      2  0     0     0   1 Feb16 ?        00:02:11 [rcu_sched]
    root          9      2  0     0     0   0 Feb16 ?        00:00:00 [rcu_bh]
    root         10      2  0     0     0   0 Feb16 ?        00:00:00 [migration/0]
    root         11      2  0     0     0   0 Feb16 ?        00:00:00 [watchdog/0]
    root         12      2  0     0     0   0 Feb16 ?        00:00:00 [cpuhp/0]
    root         13      2  0     0     0   1 Feb16 ?        00:00:00 [cpuhp/1]
    root         14      2  0     0     0   1 Feb16 ?        00:00:00 [watchdog/1]
    root         15      2  0     0     0   1 Feb16 ?        00:00:00 [migration/1]
    root         16      2  0     0     0   1 Feb16 ?        00:00:11 [ksoftirqd/1]
    root         18      2  0     0     0   1 Feb16 ?        00:00:00 [kworker/1:0H]
    root         19      2  0     0     0   2 Feb16 ?        00:00:00 [cpuhp/2]
    root         20      2  0     0     0   2 Feb16 ?        00:00:00 [watchdog/2]
    root         21      2  0     0     0   2 Feb16 ?        00:00:00 [migration/2]
    root         22      2  0     0     0   2 Feb16 ?        00:00:11 [ksoftirqd/2]
    root         24      2  0     0     0   2 Feb16 ?        00:00:00 [kworker/2:0H]

再来看下，所有内核线性的PPI=2， 也就是所有内核线性的父进程都是kthreadd进程。

    UID         PID   PPID  C    SZ   RSS PSR STIME TTY          TIME CMD
    root        362      1  0 21574  6136   2 Feb16 ?        00:00:03 /lib/systemd/systemd-journald
    root        375      1  0 11906  2760   3 Feb16 ?        00:00:01 /lib/systemd/systemd-udevd
    systemd+    417      1  0 17807  2116   3 Feb16 ?        00:00:02 /lib/systemd/systemd-resolved
    systemd+    420      1  0 35997   788   3 Feb16 ?        00:00:00 /lib/systemd/systemd-timesyncd
    root        487      1  0 43072  6060   0 Feb16 ?        00:00:00 /usr/bin/python3 /usr/bin/networkd-dispatcher --run-startup-triggers
    root        489      1  0  8268  2036   2 Feb16 ?        00:00:00 /usr/sbin/cron -f
    root        490      1  0  1138   548   0 Feb16 ?        00:00:01 /usr/sbin/acpid
    root        491      1  0 106816 3284   1 Feb16 ?        00:00:00 /usr/sbin/ModemManager
    root        506      1  0 27628  2132   2 Feb16 ?        00:00:01 /usr/sbin/irqbalance --foreground

所有用户态的进程的父进程PPID=1，也就是1号进程都是他们的父进程。






