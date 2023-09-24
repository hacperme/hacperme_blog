---
id: 1928
title: 'FreeRTOS 中的栈溢出检测机制'
date: '2021-04-11T23:26:09+08:00'
author: hacper
layout: post
guid: 'https://hacperme.com/?p=1928'
permalink: /2021/04/11/freertos_stack_overflow_checking/
views:
    - '1540'
categories:
    - 笔记
tags:
    - '8910'
    - dump
    - EC200U
    - FreeRTOS
    - RTOS
    - 'stack overflow'
    - Unisoc
    - 展锐
---

## FreeRTOS中的任务栈结构介绍

在FreeRTOS中，创建任务A、B、C三个任务，以栈的生长方向从高到低为例，其任务栈结构如下图所示：

```c
                           +----------->+--------+        Low
                           |            |        |         ^
                           |            |No used |   SP    |
                           |            |--------|<------  |
                           |            |  regs  |         |
                           |            +--------+         |
                           |            | Func 4 |         |
Low   +---------------+    |            +--------+         |
 ^    |               |    |            | Func 3 |         |
 |    | Task C stack  |    |            +--------+         |
 |    +- - - - - - - -+    |            | Func 2 |         |
 |    |     TCB       |    |            +--------+         |
 |    +---------------+----+            | Func 1 |         |
 |    |               |           +---->+--------+        High
 |    | Task B stack  |           |
 |    +- - - - - - - -+-----------+     Task B stack
 |    |     TCB       |           
 |    +---------------+
 |    |               |
 |    | Task A stack  |
 |    +- - - - - - - -+
 |    |     TCB       |
High  +---------------+
```

在内存上可以先简单看作是每个任务按任务控制块（TCB）+ stack 依次存储，任务在执行过程中的上下文数据会保存在栈中，比如函数的局部变量，传递参数，返回地址等都会存储在栈里面。在task B的栈空间里，函数调用关系是 Func 1 调用 → Func 2  调用 → Func 3 调用 → Func 4。在这样的栈存储结构，任务在运行的过程中可能会出现下面两个问题：

1. 栈溢出导致的问题，如果任务中函数调用的层级过深，或者函数内部有定义占用空间较大的局部变量，则有可能在压栈的时候使用的空间超出了任务的栈空间大小，例如task B 中的函数调用层级过多，会导致task B 的数据写到task C的堆栈空间里，从而破坏task C的栈空间。
2. 局部变量溢出导致的问题，以task B为例，如果在Func 4函数里面操作局部变量，不小心把局部变量“写穿了”，则有可能会把返回地址给覆盖了，Func 4返回时会跳转的错误的地址而出现异常，Func 4执行完也就无法返回到Func 3了，如果覆盖的空间较大，则有可能将数据写到task A的栈空间，从而导致task A的栈空间也被破坏。

## FreeRTOS的栈溢出检测机制

根据FreeRTOS的官方文档介绍，FreeRTOS提供两种栈溢出的检测方法，可以通过配置configCHECK_FOR_STACK_OVERFLOW 这个宏来选择使用哪种方法，当kernel检测到栈溢出的情况之后，会调用钩子函数vApplicationStackOverflowHook，该函数的原型如下：

```c
void vApplicationStackOverflowHook( TaskHandle_t xTask,
                                    signed char *pcTaskName );
```

### 栈溢出检测方式1

第一种检测方式的原理是检查当前任务的栈顶指针是否在该任务的栈空间范围之内，如果栈顶指针不在任务的栈空间，则会触发钩子函数vApplicationStackOverflowHook，通过配置宏configCHECK_FOR_STACK_OVERFLOW为1使用这种检测方式，具体可以看下include/stack_macros.h路径的源码：

```c
#if ( ( configCHECK_FOR_STACK_OVERFLOW == 1 ) && ( portSTACK_GROWTH < 0 ) )

/* Only the current stack state is to be checked. */
    #define taskCHECK_FOR_STACK_OVERFLOW()                                                            \
    {                                                                                                 \
        /* Is the currently saved stack pointer within the stack limit? */                            \
        if( pxCurrentTCB->pxTopOfStack <= pxCurrentTCB->pxStack )                                     \
        {                                                                                             \
            vApplicationStackOverflowHook( ( TaskHandle_t ) pxCurrentTCB, pxCurrentTCB->pcTaskName ); \
        }                                                                                             \
    }

#endif /* configCHECK_FOR_STACK_OVERFLOW == 1 */
/*-----------------------------------------------------------*/

#if ( ( configCHECK_FOR_STACK_OVERFLOW == 1 ) && ( portSTACK_GROWTH > 0 ) )

/* Only the current stack state is to be checked. */
    #define taskCHECK_FOR_STACK_OVERFLOW()                                                            \
    {                                                                                                 \
                                                                                                      \
        /* Is the currently saved stack pointer within the stack limit? */                            \
        if( pxCurrentTCB->pxTopOfStack >= pxCurrentTCB->pxEndOfStack )                                \
        {                                                                                             \
            vApplicationStackOverflowHook( ( TaskHandle_t ) pxCurrentTCB, pxCurrentTCB->pcTaskName ); \
        }                                                                                             \
    }

#endif /* configCHECK_FOR_STACK_OVERFLOW == 1 */
/*-----------------------------------------------------------*/
```

在分析代码之前要留意一下栈的生长方向，其中宏 portSTACK_GROWTH 表示栈的生长方向，当配置portSTACK_GROWTH < 0 时，说明栈的生长方向是从高地址向低地址生长，反之，当配置portSTACK_GROWTH > 0 是，表示栈是从低地址向高地址增长。pxTopOfStack 记录了当前任务的栈顶指针，pxStack是当前任务栈的起始地址，而pxEndOfStack是当前任务栈的结束地址。从代码可以看出此方式是直接通过判断当前堆栈指针是否超出了任务栈空间的合法范围来检测溢出的，如果当前任务栈被其他任务破坏了，这种方式就检测不出来。

### 栈溢出检测方式2

方式2的检测原理是在任务栈初始化的时候，填充固定数据0xA5，kernel 会检查任务栈的最后16个字节的数据是否被篡改，如果发现这16个字节不是0xA5，则会触发vApplicationStackOverflowHook的调用，通过配置configCHECK_FOR_STACK_OVERFLOW >1 来使用此方式。以下是该检测方式的源码，分析时要注意栈的增长方向：

```c
#if ( ( configCHECK_FOR_STACK_OVERFLOW > 1 ) && ( portSTACK_GROWTH < 0 ) )

    #define taskCHECK_FOR_STACK_OVERFLOW()                                                            \
    {                                                                                                 \
        const uint32_t * const pulStack = ( uint32_t * ) pxCurrentTCB->pxStack;                       \
        const uint32_t ulCheckValue = ( uint32_t ) 0xa5a5a5a5;                                        \
                                                                                                      \
        if( ( pulStack[ 0 ] != ulCheckValue ) ||                                                      \
            ( pulStack[ 1 ] != ulCheckValue ) ||                                                      \
            ( pulStack[ 2 ] != ulCheckValue ) ||                                                      \
            ( pulStack[ 3 ] != ulCheckValue ) )                                                       \
        {                                                                                             \
            vApplicationStackOverflowHook( ( TaskHandle_t ) pxCurrentTCB, pxCurrentTCB->pcTaskName ); \
        }                                                                                             \
    }

#endif /* #if( configCHECK_FOR_STACK_OVERFLOW > 1 ) */
/*-----------------------------------------------------------*/

#if ( ( configCHECK_FOR_STACK_OVERFLOW > 1 ) && ( portSTACK_GROWTH > 0 ) )

    #define taskCHECK_FOR_STACK_OVERFLOW()                                                                                                \
    {                                                                                                                                     \
        int8_t * pcEndOfStack = ( int8_t * ) pxCurrentTCB->pxEndOfStack;                                                                  \
        static const uint8_t ucExpectedStackBytes[] = { tskSTACK_FILL_BYTE, tskSTACK_FILL_BYTE, tskSTACK_FILL_BYTE, tskSTACK_FILL_BYTE,   \
                                                        tskSTACK_FILL_BYTE, tskSTACK_FILL_BYTE, tskSTACK_FILL_BYTE, tskSTACK_FILL_BYTE,   \
                                                        tskSTACK_FILL_BYTE, tskSTACK_FILL_BYTE, tskSTACK_FILL_BYTE, tskSTACK_FILL_BYTE,   \
                                                        tskSTACK_FILL_BYTE, tskSTACK_FILL_BYTE, tskSTACK_FILL_BYTE, tskSTACK_FILL_BYTE,   \
                                                        tskSTACK_FILL_BYTE, tskSTACK_FILL_BYTE, tskSTACK_FILL_BYTE, tskSTACK_FILL_BYTE }; \
                                                                                                                                          \
                                                                                                                                          \
        pcEndOfStack -= sizeof( ucExpectedStackBytes );                                                                                   \
                                                                                                                                          \
        /* Has the extremity of the task stack ever been written over? */                                                                 \
        if( memcmp( ( void * ) pcEndOfStack, ( void * ) ucExpectedStackBytes, sizeof( ucExpectedStackBytes ) ) != 0 )                     \
        {                                                                                                                                 \
            vApplicationStackOverflowHook( ( TaskHandle_t ) pxCurrentTCB, pxCurrentTCB->pcTaskName );                                     \
        }                                                                                                                                 \
    }

#endif /* #if( configCHECK_FOR_STACK_OVERFLOW > 1 ) */
/*-----------------------------------------------------------*/
```

在任务栈初始化的时候填充已知的固定数据0xA5，除了做栈溢出检测之外，还可以作为评估线程运行时占用栈空间大小的方法。其原理是在线程运行一段时间之后，从栈的低地址统计有多少字节的0xA5得到栈空间未使用的大小（假设栈从高地址向低地址生长），用栈的总大小减去未使用的空间便可得到该任务运行时栈可能的最大使用大小。

## kernel 在什么时候检查栈溢出呢？

在 tasks.c 文件可以查到taskCHECK_FOR_STACK_OVERFLOW在 void vTaskSwitchContext( void )函数中被调用，也就是在任务上下文切换的时候做检查，从这点可以看出软件检查栈溢出的方式具有一定的滞后性，任务栈被破坏了并不能马上检测到问题。

```c
void vTaskSwitchContext( void )
{
    if( uxSchedulerSuspended != ( UBaseType_t ) pdFALSE )
    {
        /* The scheduler is currently suspended - do not allow a context
         * switch. */
        xYieldPending = pdTRUE;
    }
    else
    {
        xYieldPending = pdFALSE;
        traceTASK_SWITCHED_OUT();

        #if ( configGENERATE_RUN_TIME_STATS == 1 )
            {
                #ifdef portALT_GET_RUN_TIME_COUNTER_VALUE
                    portALT_GET_RUN_TIME_COUNTER_VALUE( ulTotalRunTime );
                #else
                    ulTotalRunTime = portGET_RUN_TIME_COUNTER_VALUE();
                #endif

                /* Add the amount of time the task has been running to the
                 * accumulated time so far.  The time the task started running was
                 * stored in ulTaskSwitchedInTime.  Note that there is no overflow
                 * protection here so count values are only valid until the timer
                 * overflows.  The guard against negative values is to protect
                 * against suspect run time stat counter implementations - which
                 * are provided by the application, not the kernel. */
                if( ulTotalRunTime > ulTaskSwitchedInTime )
                {
                    pxCurrentTCB->ulRunTimeCounter += ( ulTotalRunTime - ulTaskSwitchedInTime );
                }
                else
                {
                    mtCOVERAGE_TEST_MARKER();
                }

                ulTaskSwitchedInTime = ulTotalRunTime;
            }
        #endif /* configGENERATE_RUN_TIME_STATS */

        /* Check for stack overflow, if configured. */
        taskCHECK_FOR_STACK_OVERFLOW();

        /* Before the currently running task is switched out, save its errno. */
        #if ( configUSE_POSIX_ERRNO == 1 )
            {
                pxCurrentTCB->iTaskErrno = FreeRTOS_errno;
            }
        #endif

        /* Select a new task to run using either the generic C or port
         * optimised asm code. */
        taskSELECT_HIGHEST_PRIORITY_TASK(); /*lint !e9079 void * is used as this macro is used with timers and co-routines too.  Alignment is known to be fine as the type of the pointer stored and retrieved is the same. */
        traceTASK_SWITCHED_IN();

        /* After the new task is switched in, update the global errno. */
        #if ( configUSE_POSIX_ERRNO == 1 )
            {
                FreeRTOS_errno = pxCurrentTCB->iTaskErrno;
            }
        #endif

        #if ( configUSE_NEWLIB_REENTRANT == 1 )
            {
                /* Switch Newlib's _impure_ptr variable to point to the _reent
                 * structure specific to this task.
                 * See the third party link http://www.nadler.com/embedded/newlibAndFreeRTOS.html
                 * for additional information. */
                _impure_ptr = &( pxCurrentTCB->xNewLib_reent );
            }
        #endif /* configUSE_NEWLIB_REENTRANT */
    }
}
/*-----------------------------------------------------------*/
```

## 栈溢出实例分析

### 任务控制块（TCB）中的与栈相关的成员介绍

FreeRTOS的TCB数据结构在 tasks.c 文件中定义，源码如下：

```c
/*
 * Task control block.  A task control block (TCB) is allocated for each task,
 * and stores task state information, including a pointer to the task's context
 * (the task's run time environment, including register values)
 */
typedef struct tskTaskControlBlock       /* The old naming convention is used to prevent breaking kernel aware debuggers. */
{
    volatile StackType_t * pxTopOfStack; /*< Points to the location of the last item placed on the tasks stack.  THIS MUST BE THE FIRST MEMBER OF THE TCB STRUCT. */

    #if ( portUSING_MPU_WRAPPERS == 1 )
        xMPU_SETTINGS xMPUSettings; /*< The MPU settings are defined as part of the port layer.  THIS MUST BE THE SECOND MEMBER OF THE TCB STRUCT. */
    #endif

    ListItem_t xStateListItem;                  /*< The list that the state list item of a task is reference from denotes the state of that task (Ready, Blocked, Suspended ). */
    ListItem_t xEventListItem;                  /*< Used to reference a task from an event list. */
    UBaseType_t uxPriority;                     /*< The priority of the task.  0 is the lowest priority. */
    StackType_t * pxStack;                      /*< Points to the start of the stack. */
    char pcTaskName[ configMAX_TASK_NAME_LEN ]; /*< Descriptive name given to the task when created.  Facilitates debugging only. */ /*lint !e971 Unqualified char types are allowed for strings and single characters only. */

    #if ( ( portSTACK_GROWTH > 0 ) || ( configRECORD_STACK_HIGH_ADDRESS == 1 ) )
        StackType_t * pxEndOfStack; /*< Points to the highest valid address for the stack. */
    #endif

    #if ( portCRITICAL_NESTING_IN_TCB == 1 )
        UBaseType_t uxCriticalNesting; /*< Holds the critical section nesting depth for ports that do not maintain their own count in the port layer. */
    #endif

    #if ( configUSE_TRACE_FACILITY == 1 )
        UBaseType_t uxTCBNumber;  /*< Stores a number that increments each time a TCB is created.  It allows debuggers to determine when a task has been deleted and then recreated. */
        UBaseType_t uxTaskNumber; /*< Stores a number specifically for use by third party trace code. */
    #endif

    #if ( configUSE_MUTEXES == 1 )
        UBaseType_t uxBasePriority; /*< The priority last assigned to the task - used by the priority inheritance mechanism. */
        UBaseType_t uxMutexesHeld;
    #endif

    #if ( configUSE_APPLICATION_TASK_TAG == 1 )
        TaskHookFunction_t pxTaskTag;
    #endif

    #if ( configNUM_THREAD_LOCAL_STORAGE_POINTERS > 0 )
        void * pvThreadLocalStoragePointers[ configNUM_THREAD_LOCAL_STORAGE_POINTERS ];
    #endif

    #if ( configGENERATE_RUN_TIME_STATS == 1 )
        uint32_t ulRunTimeCounter; /*< Stores the amount of time the task has spent in the Running state. */
    #endif

    #if ( configUSE_NEWLIB_REENTRANT == 1 )
        /* Allocate a Newlib reent structure that is specific to this task.
         * Note Newlib support has been included by popular demand, but is not
         * used by the FreeRTOS maintainers themselves.  FreeRTOS is not
         * responsible for resulting newlib operation.  User must be familiar with
         * newlib and must provide system-wide implementations of the necessary
         * stubs. Be warned that (at the time of writing) the current newlib design
         * implements a system-wide malloc() that must be provided with locks.
         *
         * See the third party link http://www.nadler.com/embedded/newlibAndFreeRTOS.html
         * for additional information. */
        struct  _reent xNewLib_reent;
    #endif

    #if ( configUSE_TASK_NOTIFICATIONS == 1 )
        volatile uint32_t ulNotifiedValue[ configTASK_NOTIFICATION_ARRAY_ENTRIES ];
        volatile uint8_t ucNotifyState[ configTASK_NOTIFICATION_ARRAY_ENTRIES ];
    #endif

    /* See the comments in FreeRTOS.h with the definition of
     * tskSTATIC_AND_DYNAMIC_ALLOCATION_POSSIBLE. */
    #if ( tskSTATIC_AND_DYNAMIC_ALLOCATION_POSSIBLE != 0 ) /*lint !e731 !e9029 Macro has been consolidated for readability reasons. */
        uint8_t ucStaticallyAllocated;                     /*< Set to pdTRUE if the task is a statically allocated to ensure no attempt is made to free the memory. */
    #endif

    #if ( INCLUDE_xTaskAbortDelay == 1 )
        uint8_t ucDelayAborted;
    #endif

    #if ( configUSE_POSIX_ERRNO == 1 )
        int iTaskErrno;
    #endif
} tskTCB;
```

这里仅介绍几个与任务栈相关的几个成员。

- pxTopOfStack

pxTopOfStack 存储任务栈的栈顶地址。

- pxStack

pxStack 存储任务栈空间的起始地址，也就栈的最低地址。

- pxEndOfStack

pxEndOfStack 存储任务栈空间的结束地址，也就是栈的最高地址。

### EC200U 栈溢出导致memory dump 分析

EC200U模组（Unisoc 8910Dm）的open sdk使用的RTOS是FreeRTOS，模组死机时导出的memory dump 文件可以通过trace 32工具分析，以下是一个栈溢出的dump实例。

通过trace 32加载dump文件，首先看到死机前函数栈帧里有调用vApplicationStackOverflowHook函数，说明kernel在任务上下文切换时检测到了栈溢出问题。

![](https://jsd.cdn.zzko.cn/gh/hacperme/picx_hosting@master/images/qns/Untitled.3ljv1vf7grq0.png)

查看线程列表，正在执行的task是rm_app_start这个线程，也就是在这个任务死机了。

![](https://jsd.cdn.zzko.cn/gh/hacperme/picx_hosting@master/images/qns/Untitled-1.vqzir71hzpc.png)

接着查看rm_app_start的任务控制块数据，其中有三个数据需要关注：

1. TCB的起始地址→0x80bf78b8

还记得前面介绍的FreeRTOS栈存储结构吗？是按照TCB+stack方式存储的。EC200U的栈增长方向是按高地址向地址增长的。

TCB+stack紧挨着，所以知道了TCB的起始地址也就知道了栈的结束地址（最高地址，栈底）。

2. 栈顶地址 pxTopOfStack →0x80bf7504

从栈底到栈顶这段空间存储了函数调用的地址，对分析函数调用关系挺有帮助。

3. 栈的起始地址 pxStack → 0x80bf74a8

![](https://jsd.cdn.zzko.cn/gh/hacperme/picx_hosting@master/images/qns/Untitled-2.3jp3qqf5xdc0.png)

可以使用trace 32查看这些地址的内存数据，比如，先来看TCB的起始地址→0x80bf78b8的数据，下图右箭头所指的地方就是TCB的起始地址，我们可以看到开始的4个字节数据是0x80bf7504，也就是存储着pxTopOfStack的地址。

![](https://jsd.cdn.zzko.cn/gh/hacperme/picx_hosting@master/images/qns/Untitled-3.3bfxtptmgl80.png)

接着再看栈的最低地址0x80bf74a8的数据，通过前面栈溢出的检测机制介绍可以知道，栈在初始化的时候会填充默认值0xA5，然后kernel在任务上下文切换的时候会去比较栈起始地址开始的16个字节是不是保持着默认值0xA5，如果初始值被篡改，则会触发vApplicationStackOverflowHook。从下图的数据来看，栈最低地址0x80bf74a8的数据已经被破坏了。

![](https://jsd.cdn.zzko.cn/gh/hacperme/picx_hosting@master/images/qns/Untitled-4.2hud1c5k2rs0.png)

我们再找一个正常的task做对比，也就是下图的IDLE task。同样的方法，先查看TCB数据，找到栈的起始地址0x80992500, 栈的结束地址0x809934f8，栈顶地址0x8099342c。

![](https://jsd.cdn.zzko.cn/gh/hacperme/picx_hosting@master/images/qns/Untitled-5.12q9ltcipo8g.png)

查看栈的起始地址0x80992500的内存数据，开始的16个字节都是0xA5，说明栈空间够，没有溢出，且还有一段未曾使用的空间。

![](https://jsd.cdn.zzko.cn/gh/hacperme/picx_hosting@master/images/qns/Untitled-6.mjuoih6h8hc.png)

再看栈顶地址0x8099342c的数据，在0x8099342c之上有一段不是0xA5的数据，说明曾经有发生过函数调用，且成功返回了。栈保存的函数调用信息在出栈之后并不会主动清除，只会在下次入栈的时候数据覆盖，由此我们也可以通过这个信息查看这个任务曾经调用过什么函数。

![](https://jsd.cdn.zzko.cn/gh/hacperme/picx_hosting@master/images/qns/Untitled-7.v4jbtnlhacw.png)

从栈顶到栈底这段空间保存在函数调用的信息，我们可以使用trace 32来解析函数调用关系。

![](https://jsd.cdn.zzko.cn/gh/hacperme/picx_hosting@master/images/qns/Untitled-8.5d2amkqv3u80.png)

在栈顶与栈底这段内存数据里，我先找到其中的函数地址，再通过设置断点的方式根据地址找到函数名。比如下图依次找到两个程序地址：0x6014B6CF 和 0x6014B651，通过设置断点查到对应的函数名prvIdleTask和xTaskResumeAll，再根据栈的增长方向可以判断是 prvIdleTask  调用→ xTaskResumeAll。

![](https://jsd.cdn.zzko.cn/gh/hacperme/picx_hosting@master/images/qns/Untitled-9.67ablvydo580.png)

把栈空间所有程序地址找出来，按照同样的方法就能找到这个任务的函数调用痕迹了：

```c
6014B795	vTaskSwitchContext\51+0x1
6014B795	vTaskSwitchContext\51+0x1
6014B795 	vTaskSwitchContext\51+0x1
6014B651	xTaskResumeAll\88+0x1
6014B6CF 	prvIdleTask\104+0x5
```

有一个规律，EC200U的程序地址都是60XXX开头的，比如6014B6CF、6014B651、6014B795等，可以按照这个规律排查是否是程序地址。当然，如果你能确定内存布局，在栈的数据里判断是否是程序地址就不会出错了。

## 总结

1. 从任务栈的结构的角度分析，便于理解栈溢出产生的原因和栈溢出导致的两类问题。
2. FreeRTOS提供两种软件栈溢出检测方式，两种方法各有优缺点，且不一定能够在任何时候都检测到所有类型的栈溢出。kernel 是在任务的上下文切换的时候进行栈溢出检查，通过软件检测栈溢出的方法，在发现问题上具有一定的“滞后性”。
3. 在memory dump中分析栈问题有几个关键的信息点需要掌握：栈的生长方向、栈的起止地址、栈顶地址，栈的保护标记（栈空间初始化设置的固定值）以及通过栈空间的数据分析函数调用关系。

## 参考资料

1. [FreeRTOS - stacks and stack overflow checking](https://www.notion.so/FreeRTOS-stacks-and-stack-overflow-checking-df93402e952b4aa79b381d28693b2dc5) 
2. [https://github.com/FreeRTOS/FreeRTOS-Kernel](https://github.com/FreeRTOS/FreeRTOS-Kernel)
3. [Zephyr使用的堆栈保护技术](https://www.notion.so/Zephyr-cf671a64822a4bd7be4e200ba9240676)