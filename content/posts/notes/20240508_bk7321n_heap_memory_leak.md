---
title: "为 freertos 的 heap_4 动态内存分配方案增加 heap info 调试信息"
date: 2024-06-23T02:00:18+08:00
lastmod: 2024-06-23T02:00:18+08:00
author: ["hacper"]
tags:
    - bk7231n
    - freertos
    - c
categories:
    - 笔记
description: "" # 文章描述，与搜索优化相关
summary: "为 freertos 的 heap_4 增加 heap info 调试信息，用于排查内存泄漏和内存越界问题" # 文章简单描述，会展示在主页
# weight: # 输入1可以顶置文章，用来给文章展示排序，不填就默认按时间排序
slug: ""
draft: false # 是否为草稿
comments: true
showToc: true # 显示目录
TocOpen: true # 自动展开目录
autonumbering: true # 目录自动编号
hidemeta: false # 是否隐藏文章的元信息，如发布日期、作者等
disableShare: true # 底部不显示分享栏
searchHidden: false # 该页面可以被搜索到
showbreadcrumbs: true #顶部显示当前路径
# mermaid: true

---




## 如何为 freertos 的 heap_4 动态内存分配方案增加 heap info 调试信息？

增加调试信息的目的是为了方便排查内存泄漏和内存越界问题，为了排查内存泄漏，需要记录每个内存块的申请者信息，以下是针对内存泄漏问题调试的修改。

- 修改内存块头部的数据结构，增加字段记录内存申请者的信息和申请的内存大小。

```c
typedef struct A_BLOCK_LINK
{
    struct A_BLOCK_LINK * pxNextFreeBlock; /*<< The next free block in the list. */
    size_t xBlockSize;                     /*<< The size of the free block. */
    size_t xWantedSize;
    void *caller;
} BlockLink_t;

```
xWantedSize 和 caller 是新增的字段，分别用于记录内存分配时候的调用者信息和申请的内存大小。

- 修改 pvPortMalloc 和 pvPortRealloc 的定义，传入参数增加 caller。

```c
void * pvPortMalloc( size_t xSize , void *caller) PRIVILEGED_FUNCTION;
void * pvPortRealloc( void *p,
                     size_t xSize, void *caller) PRIVILEGED_FUNCTION;
```

- 修改 prvHeapInit pvPortMalloc 的实现，增加记录 caller，增加新接口 show_heap_info 查看 heap 的内存使用情况。

在 heap 初始化的时候增加 caller xWantedSize 的初始化。
```c
static void prvHeapInit( void ) /* PRIVILEGED_FUNCTION */
{
    BlockLink_t * pxFirstFreeBlock;
    uint8_t * pucAlignedHeap;
    size_t uxAddress;
    size_t xTotalHeapSize = configTOTAL_HEAP_SIZE;

    /* Ensure the heap starts on a correctly aligned boundary. */
    uxAddress = ( size_t ) ucHeap;

    if( ( uxAddress & portBYTE_ALIGNMENT_MASK ) != 0 )
    {
        uxAddress += ( portBYTE_ALIGNMENT - 1 );
        uxAddress &= ~( ( size_t ) portBYTE_ALIGNMENT_MASK );
        xTotalHeapSize -= uxAddress - ( size_t ) ucHeap;
    }

    pucAlignedHeap = ( uint8_t * ) uxAddress;

    /* xStart is used to hold a pointer to the first item in the list of free
     * blocks.  The void cast is used to prevent compiler warnings. */
    xStart.pxNextFreeBlock = ( void * ) pucAlignedHeap;
    xStart.xBlockSize = ( size_t ) 0;
    xStart.caller = NULL;
    xStart.xWantedSize = 0;

    /* pxEnd is used to mark the end of the list of free blocks and is inserted
     * at the end of the heap space. */
    uxAddress = ( ( size_t ) pucAlignedHeap ) + xTotalHeapSize;
    uxAddress -= xHeapStructSize;
    uxAddress &= ~( ( size_t ) portBYTE_ALIGNMENT_MASK );
    pxEnd = ( void * ) uxAddress;
    pxEnd->xBlockSize = 0;
    pxEnd->caller = NULL;
    pxEnd->pxNextFreeBlock = NULL;

    /* To start with there is a single free block that is sized to take up the
     * entire heap space, minus the space taken by pxEnd. */
    pxFirstFreeBlock = ( void * ) pucAlignedHeap;
    pxFirstFreeBlock->xBlockSize = uxAddress - ( size_t ) pxFirstFreeBlock;
    pxFirstFreeBlock->xWantedSize = 0;
    pxFirstFreeBlock->caller = NULL;
    pxFirstFreeBlock->pxNextFreeBlock = pxEnd;

    /* Only one block exists - and it covers the entire usable heap space. */
    xMinimumEverFreeBytesRemaining = pxFirstFreeBlock->xBlockSize;
    xFreeBytesRemaining = pxFirstFreeBlock->xBlockSize;
}
```


在分配内存的时候，记录caller 和 xWantedSize 到内存卡的头部。
```c
void * pvPortMalloc( size_t xWantedSize , void * caller)
{
    BlockLink_t * pxBlock, * pxPreviousBlock, * pxNewBlockLink;
    void * pvReturn = NULL;
    size_t xAdditionalRequiredSize;
    size_t _xWantedSize = xWantedSize;

    vTaskSuspendAll();
    {
        /* If this is the first call to malloc then the heap will require
         * initialisation to setup the list of free blocks. */
        if( pxEnd == NULL )
        {
            prvHeapInit();
        }
        else
        {
            mtCOVERAGE_TEST_MARKER();
        }

        if( xWantedSize > 0 )
        {
            /* The wanted size must be increased so it can contain a BlockLink_t
             * structure in addition to the requested amount of bytes. Some
             * additional increment may also be needed for alignment. */
            xAdditionalRequiredSize = xHeapStructSize  + portBYTE_ALIGNMENT - ( xWantedSize & portBYTE_ALIGNMENT_MASK );

            if( heapADD_WILL_OVERFLOW( xWantedSize, xAdditionalRequiredSize ) == 0 )
            {
                xWantedSize += xAdditionalRequiredSize;
            }
            else
            {
                xWantedSize = 0;
            }
        }
        else
        {
            mtCOVERAGE_TEST_MARKER();
        }

        /* Check the block size we are trying to allocate is not so large that the
         * top bit is set.  The top bit of the block size member of the BlockLink_t
         * structure is used to determine who owns the block - the application or
         * the kernel, so it must be free. */
        if( heapBLOCK_SIZE_IS_VALID( xWantedSize ) != 0 )
        {
            if( ( xWantedSize > 0 ) && ( xWantedSize <= xFreeBytesRemaining ) )
            {
                /* Traverse the list from the start (lowest address) block until
                 * one of adequate size is found. */
                pxPreviousBlock = &xStart;
                pxBlock = xStart.pxNextFreeBlock;

                while( ( pxBlock->xBlockSize < xWantedSize ) && ( pxBlock->pxNextFreeBlock != NULL ) )
                {
                    pxPreviousBlock = pxBlock;
                    pxBlock = pxBlock->pxNextFreeBlock;
                }

                /* If the end marker was reached then a block of adequate size
                 * was not found. */
                if( pxBlock != pxEnd )
                {
                    /* Return the memory space pointed to - jumping over the
                     * BlockLink_t structure at its start. */
                    pvReturn = ( void * ) ( ( ( uint8_t * ) pxPreviousBlock->pxNextFreeBlock ) + xHeapStructSize );

                    /* This block is being returned for use so must be taken out
                     * of the list of free blocks. */
                    pxPreviousBlock->pxNextFreeBlock = pxBlock->pxNextFreeBlock;

                    /* If the block is larger than required it can be split into
                     * two. */
                    if( ( pxBlock->xBlockSize - xWantedSize ) > heapMINIMUM_BLOCK_SIZE )
                    {
                        /* This block is to be split into two.  Create a new
                         * block following the number of bytes requested. The void
                         * cast is used to prevent byte alignment warnings from the
                         * compiler. */
                        pxNewBlockLink = ( void * ) ( ( ( uint8_t * ) pxBlock ) + xWantedSize );
                        configASSERT( ( ( ( size_t ) pxNewBlockLink ) & portBYTE_ALIGNMENT_MASK ) == 0 );

                        /* Calculate the sizes of two blocks split from the
                         * single block. */
                        pxNewBlockLink->xBlockSize = pxBlock->xBlockSize - xWantedSize;
                        pxNewBlockLink->xWantedSize = 0;
                        pxNewBlockLink->caller = NULL;
                        pxBlock->xBlockSize = xWantedSize;

                        /* Insert the new block into the list of free blocks. */
                        prvInsertBlockIntoFreeList( pxNewBlockLink );
                    }
                    else
                    {
                        mtCOVERAGE_TEST_MARKER();
                    }

                    

                    xFreeBytesRemaining -= pxBlock->xBlockSize;
                    

                    if( xFreeBytesRemaining < xMinimumEverFreeBytesRemaining )
                    {
                        xMinimumEverFreeBytesRemaining = xFreeBytesRemaining;
                    }
                    else
                    {
                        mtCOVERAGE_TEST_MARKER();
                    }

              
                    pxBlock->xWantedSize = _xWantedSize;
                    pxBlock->caller = caller;
                    /* The block is being returned - it is allocated and owned
                     * by the application and has no "next" block. */
                    heapALLOCATE_BLOCK( pxBlock );
                    pxBlock->pxNextFreeBlock = NULL;
                    xNumberOfSuccessfulAllocations++;
                }
                else
                {
                    mtCOVERAGE_TEST_MARKER();
                }
            }
            else
            {
                mtCOVERAGE_TEST_MARKER();
            }
        }
        else
        {
            mtCOVERAGE_TEST_MARKER();
        }

        traceMALLOC( pvReturn, xWantedSize );
    }
    xTaskResumeAll();

    #if ( configUSE_MALLOC_FAILED_HOOK == 1 )
    {
        if( pvReturn == NULL )
        {
            vApplicationMallocFailedHook();
        }
        else
        {
            mtCOVERAGE_TEST_MARKER();
        }
    }
    #endif /* if ( configUSE_MALLOC_FAILED_HOOK == 1 ) */

    configASSERT( ( ( ( size_t ) pvReturn ) & ( size_t ) portBYTE_ALIGNMENT_MASK ) == 0 );
    return pvReturn;
}

void *pvPortRealloc(void *p,
                    size_t xSize, void * caller)
{
    void *pv = NULL;
    uint8_t *puc = (uint8_t *)p;
    BlockLink_t *pxLink;

    if (p == NULL)
    {
        return NULL;
    }

    puc -= xHeapStructSize;
    pxLink = (void *)puc;

    configASSERT(heapBLOCK_IS_ALLOCATED(pxLink) != 0);
    configASSERT(pxLink->pxNextFreeBlock == NULL);

    pv = pvPortMalloc(xSize, caller);
    if (pv == NULL)
    {
        return NULL;
    }
    memcpy(pv, p, pxLink->xWantedSize);
    vPortFree(p);

    return pv;   
}

```

遍历 heap 所有的内存块，打印出其调用者、申请内存大小、是已分配的内存块还是空闲的内存块，利用这些信息排查内存泄漏。

```c
void show_heap_info(void)
{
    BlockLink_t * pxLink = NULL;
    uint8_t * pucAlignedHeap;
    size_t uxAddress;
    size_t xTotalHeapSize = configTOTAL_HEAP_SIZE;

    vTaskSuspendAll();
    uxAddress = ( size_t ) ucHeap;

    if( ( uxAddress & portBYTE_ALIGNMENT_MASK ) != 0 )
    {
        uxAddress += ( portBYTE_ALIGNMENT - 1 );
        uxAddress &= ~( ( size_t ) portBYTE_ALIGNMENT_MASK );
        xTotalHeapSize -= uxAddress - ( size_t ) ucHeap;
    }

    pucAlignedHeap = ( uint8_t * ) uxAddress;

    pxLink = (BlockLink_t *)pucAlignedHeap;

    printf("\r\naddress: 0x%p\r\nsize: %d\r\navail: %d\r\npool_start: 0x%p\r\npool_end: 0x%p\r\n",
           ucHeap, xTotalHeapSize, xFreeBytesRemaining, pucAlignedHeap, pxEnd);

    printf("state,block_addr,user_addr,caller,blocksize,wanted_size\r\n");
 
    while (pxLink != pxEnd)
    {
		
        printf("%s, 0x%p, 0x%p, 0x%p,%d,%d\r\n", ((pxLink->xBlockSize & heapBLOCK_ALLOCATED_BITMASK)!=0)? "U":"F", pxLink, (uint8_t *)pxLink+xHeapStructSize, pxLink->caller, (( pxLink->xBlockSize ) & ~heapBLOCK_ALLOCATED_BITMASK), pxLink->xWantedSize);
        pxLink =(BlockLink_t *) (( uint8_t * )pxLink + (( pxLink->xBlockSize ) & ~heapBLOCK_ALLOCATED_BITMASK));
    }

    xTaskResumeAll();
}
```

- 利用库打桩机制，替换标准库中的 malloc free realloc 等内存分配函数。

```c

/************** wrap C library functions **************/
void * __wrap_malloc (size_t size)
{
    void *caller = __builtin_return_address (0);
	return pvPortMalloc(size, caller);
}

void * __wrap__malloc_r (void *p, size_t size)
{
	void *caller = __builtin_return_address (0);
	return pvPortMalloc(size, caller);
}

void __wrap_free (void *pv)
{
	vPortFree(pv);
}

void * __wrap_calloc (size_t a, size_t b)
{
	void *pvReturn;
    void *caller = __builtin_return_address (0);
    pvReturn = pvPortMalloc( a*b, caller);
    if (pvReturn)
    {
        memset(pvReturn, 0, a*b);
    }

    return pvReturn;
}

void * __wrap_realloc (void* pv, size_t size)
{
    void *caller = __builtin_return_address (0);
	return pvPortRealloc(pv, size, caller);
}

void __wrap__free_r (void *p, void *x)
{
  __wrap_free(x);
}

void* __wrap__realloc_r (void *p, void* x, size_t sz)
{
    void *caller = __builtin_return_address (0);
	return pvPortRealloc(x, sz, caller);
}

/*-----------------------------------------------------------*/
```
gcc 的链接器支持用 --wrap f 参数进行链接时打桩，这个参数告诉链接器，把对符号 f 的引用解析成 __wrap_f。通过这个机制，我们调用malloc的时候，便会替换成我们加了调试信息的__wrap_malloc。另外，再使用 __builtin_return_address 接口获取函数的返回地址并记录，这样就知道是谁申请了这块内存。

最后再添加连接器参数：
```
sdk_add_link_options(
        -Wl,--wrap=malloc
        -Wl,--wrap=_malloc_r
        -Wl,--wrap=free
        -Wl,--wrap=calloc
        -Wl,--wrap=realloc
        -Wl,--wrap=_free_r
        -Wl,--wrap=_realloc_r
)
```

对于内存越界问题，可以在每个内存块的末尾增加冗余长度，并填充特定数据，在内存释放的时候检查填充的数据有没有被篡改，以此判断是否存在内存被破坏问题。

- pvPortMalloc 的修改

在 xWantedSize 的基础上增加一个字节，确保内存块的尾部够空间填充一个特殊数据。然后在将要分配出去的内存块中末尾未使用的数据全部填充为0xfd，填充的长度为 pxBlock->xBlockSize-_xWantedSize-xHeapStructSize。 

```c
void * pvPortMalloc( size_t xWantedSize , void * caller)
{
    BlockLink_t * pxBlock, * pxPreviousBlock, * pxNewBlockLink;
    void * pvReturn = NULL;
    size_t xAdditionalRequiredSize;
    size_t _xWantedSize = xWantedSize;

    vTaskSuspendAll();
    {
        /* If this is the first call to malloc then the heap will require
         * initialisation to setup the list of free blocks. */
        if( pxEnd == NULL )
        {
            prvHeapInit();
        }
        else
        {
            mtCOVERAGE_TEST_MARKER();
        }

        if( xWantedSize > 0 )
        {
            /* The wanted size must be increased so it can contain a BlockLink_t
             * structure in addition to the requested amount of bytes. Some
             * additional increment may also be needed for alignment. */
            xWantedSize += 1;
            xAdditionalRequiredSize = xHeapStructSize  + portBYTE_ALIGNMENT - ( xWantedSize & portBYTE_ALIGNMENT_MASK );

            if( heapADD_WILL_OVERFLOW( xWantedSize, xAdditionalRequiredSize ) == 0 )
            {
                xWantedSize += xAdditionalRequiredSize;
            }
            else
            {
                xWantedSize = 0;
            }
        }
        else
        {
            mtCOVERAGE_TEST_MARKER();
        }

        /* Check the block size we are trying to allocate is not so large that the
         * top bit is set.  The top bit of the block size member of the BlockLink_t
         * structure is used to determine who owns the block - the application or
         * the kernel, so it must be free. */
        if( heapBLOCK_SIZE_IS_VALID( xWantedSize ) != 0 )
        {
            if( ( xWantedSize > 0 ) && ( xWantedSize <= xFreeBytesRemaining ) )
            {
                /* Traverse the list from the start (lowest address) block until
                 * one of adequate size is found. */
                pxPreviousBlock = &xStart;
                pxBlock = xStart.pxNextFreeBlock;

                while( ( pxBlock->xBlockSize < xWantedSize ) && ( pxBlock->pxNextFreeBlock != NULL ) )
                {
                    pxPreviousBlock = pxBlock;
                    pxBlock = pxBlock->pxNextFreeBlock;
                }

                /* If the end marker was reached then a block of adequate size
                 * was not found. */
                if( pxBlock != pxEnd )
                {
                    /* Return the memory space pointed to - jumping over the
                     * BlockLink_t structure at its start. */
                    pvReturn = ( void * ) ( ( ( uint8_t * ) pxPreviousBlock->pxNextFreeBlock ) + xHeapStructSize );

                    /* This block is being returned for use so must be taken out
                     * of the list of free blocks. */
                    pxPreviousBlock->pxNextFreeBlock = pxBlock->pxNextFreeBlock;

                    /* If the block is larger than required it can be split into
                     * two. */
                    if( ( pxBlock->xBlockSize - xWantedSize ) > heapMINIMUM_BLOCK_SIZE )
                    {
                        /* This block is to be split into two.  Create a new
                         * block following the number of bytes requested. The void
                         * cast is used to prevent byte alignment warnings from the
                         * compiler. */
                        pxNewBlockLink = ( void * ) ( ( ( uint8_t * ) pxBlock ) + xWantedSize );
                        configASSERT( ( ( ( size_t ) pxNewBlockLink ) & portBYTE_ALIGNMENT_MASK ) == 0 );

                        /* Calculate the sizes of two blocks split from the
                         * single block. */
                        pxNewBlockLink->xBlockSize = pxBlock->xBlockSize - xWantedSize;
                        pxNewBlockLink->xWantedSize = 0;
                        pxNewBlockLink->caller = NULL;
                        pxBlock->xBlockSize = xWantedSize;

                        /* Insert the new block into the list of free blocks. */
                        prvInsertBlockIntoFreeList( pxNewBlockLink );
                    }
                    else
                    {
                        mtCOVERAGE_TEST_MARKER();
                    }

                    

                    xFreeBytesRemaining -= pxBlock->xBlockSize;
                    

                    if( xFreeBytesRemaining < xMinimumEverFreeBytesRemaining )
                    {
                        xMinimumEverFreeBytesRemaining = xFreeBytesRemaining;
                    }
                    else
                    {
                        mtCOVERAGE_TEST_MARKER();
                    }

              
                    pxBlock->xWantedSize = _xWantedSize;
                    pxBlock->caller = caller;
                    int _len = pxBlock->xBlockSize-_xWantedSize-xHeapStructSize;
					uint8_t *_p = (uint8_t *)pxBlock+_xWantedSize+xHeapStructSize;
					while (_len > 0)
					{
						*_p = 0xfd;
						_p++;
						_len--;
					}

                    /* The block is being returned - it is allocated and owned
                     * by the application and has no "next" block. */
                    heapALLOCATE_BLOCK( pxBlock );
                    pxBlock->pxNextFreeBlock = NULL;
                    xNumberOfSuccessfulAllocations++;
                }
                else
                {
                    mtCOVERAGE_TEST_MARKER();
                }
            }
            else
            {
                mtCOVERAGE_TEST_MARKER();
            }
        }
        else
        {
            mtCOVERAGE_TEST_MARKER();
        }

        traceMALLOC( pvReturn, xWantedSize );
    }
    xTaskResumeAll();

    #if ( configUSE_MALLOC_FAILED_HOOK == 1 )
    {
        if( pvReturn == NULL )
        {
            vApplicationMallocFailedHook();
        }
        else
        {
            mtCOVERAGE_TEST_MARKER();
        }
    }
    #endif /* if ( configUSE_MALLOC_FAILED_HOOK == 1 ) */

    configASSERT( ( ( ( size_t ) pvReturn ) & ( size_t ) portBYTE_ALIGNMENT_MASK ) == 0 );
    return pvReturn;
}
```

在释放内存的时候检查内存块的填充数据是否有被篡改，如果有被篡改则说明出现了内存越界问题，可以打印当前内存块的 caller 信息和 dump 整个 block 数据，然后立再触发 ASSERT。
除了在内存释放的时候检查heap是否有越界，也可以在任务上下文切换的时候检查，不过这样会增加任务调度的开销，影响任务切换的效率。

```c

void vPortFree( void * pv )
{
    uint8_t * puc = ( uint8_t * ) pv;
    BlockLink_t * pxLink;

    if( pv != NULL )
    {
        /* The memory being freed will have an BlockLink_t structure immediately
         * before it. */
        puc -= xHeapStructSize;

        /* This casting is to keep the compiler from issuing warnings. */
        pxLink = ( void * ) puc;

        configASSERT( heapBLOCK_IS_ALLOCATED( pxLink ) != 0 );
        configASSERT( pxLink->pxNextFreeBlock == NULL );

        if( heapBLOCK_IS_ALLOCATED( pxLink ) != 0 )
        {
            if( pxLink->pxNextFreeBlock == NULL )
            {
                /* The block is being returned to the heap - it is no longer
                 * allocated. */
                heapFREE_BLOCK( pxLink );
                #if ( configHEAP_CLEAR_MEMORY_ON_FREE == 1 )
                {
                    ( void ) memset( puc + xHeapStructSize, 0, pxLink->xBlockSize - xHeapStructSize );
                }
                #endif

                vTaskSuspendAll();
                int _len = pxLink->xBlockSize - pxLink->xWantedSize - xHeapStructSize;
				uint8_t *_p = (uint8_t *)pxLink + pxLink->xWantedSize + xHeapStructSize;
				while (_len > 0)
				{
					if(*_p != 0xfd)
					{
						printf("mem crash,block:0x%p,caller: 0x%p, 0x%02x\r\n", pxLink, pxLink->caller, *_p);
					}
					configASSERT( (*_p == 0xfd ) );
					_p++;
					_len--;
				}

                {
                    /* Add this block to the list of free blocks. */
                    xFreeBytesRemaining += pxLink->xBlockSize;
                    traceFREE( pv, pxLink->xBlockSize );
                    pxLink->xWantedSize = 0;
                    prvInsertBlockIntoFreeList( ( ( BlockLink_t * ) pxLink ) );
                    xNumberOfSuccessfulFrees++;
                }
                /*( void ) */xTaskResumeAll();
            }
            else
            {
                mtCOVERAGE_TEST_MARKER();
            }
        }
        else
        {
            mtCOVERAGE_TEST_MARKER();
        }
    }
}
```

增加 heap info 信息会增加内存分配的开销，导致实际可以使用的 heap 内存变小，在内存较小的机器，应在正式软件上关闭 heap info 信息，以节省内存。

## heap info 调试案例

### 内存泄漏问题案例


打开 mqtt 之前的 heap info：

```bash
address: 0x40aae8
size: 152856
avail: 45288
pool_start: 0x40aae8
pool_end: 0x42fff0
state,block_addr,user_addr,caller,blocksize,wanted_size
U,0x40aae8,0x40abe8,0x1b695,4112,4096
U,0x40baf8,0x40bbf8,0x1b6b5,40,20
U,0x40bb20,0x40bc20,0x1b695,528,512
U,0x40bd30,0x40be30,0x1b6b5,40,20
U,0x40bd58,0x40be58,0x1b695,4112,4096
U,0x40cd68,0x40ce68,0x1b6b5,40,20
U,0x40cd90,0x40ce90,0x1b695,528,512
U,0x40cfa0,0x40d0a0,0x1b6b5,40,20
U,0x40cfc8,0x40d0c8,0x7eab8,96,80
U,0x40d028,0x40d128,0x7e8ac,96,80
U,0x40d088,0x40d188,0x28319,112,92
U,0x40d0f8,0x40d1f8,0x28441,96,80
U,0x40d158,0x40d258,0x7eab8,96,80
U,0x40d1b8,0x40d2b8,0x7eab8,96,80
U,0x40d218,0x40d318,0x28319,112,92
U,0x40d288,0x40d388,0x28441,96,80
U,0x40d2e8,0x40d3e8,0x7eab8,96,80
U,0x40d348,0x40d448,0x7eab8,96,80
U,0x40d3a8,0x40d4a8,0x28319,112,92
U,0x40d418,0x40d518,0x28441,96,80
U,0x40d478,0x40d578,0x7eab8,96,80
U,0x40d4d8,0x40d5d8,0x7eab8,96,80
U,0x40d538,0x40d638,0x803f4,112,92
U,0x40d5a8,0x40d6a8,0xa4be5,56,36
U,0x40d5e0,0x40d6e0,0x9fb53,48,28
U,0x40d610,0x40d710,0xb22f3,760,744
U,0x40d908,0x40da08,0x7f51c,1040,1024
U,0x40dd18,0x40de18,0x7f51c,112,92
U,0x40dd88,0x40de88,0x7fe1c,288,272
U,0x40dea8,0x40dfa8,0x7fefc,3088,3072
U,0x40eab8,0x40ebb8,0x7fefc,112,92
U,0x40eb28,0x40ec28,0x15769,32,16
U,0x40eb48,0x40ec48,0x15789,128,108
U,0x40ebc8,0x40ecc8,0x157a9,136,114
U,0x40ec50,0x40ed50,0x80808,120,100
U,0x40ecc8,0x40edc8,0x803f4,1040,1024
U,0x40f0d8,0x40f1d8,0x803f4,112,92
U,0x40f148,0x40f248,0x7e8ac,96,80
U,0x40f1a8,0x40f2a8,0x803f4,4112,4096
U,0x4101b8,0x4102b8,0x803f4,112,92
U,0x410228,0x410328,0x28441,96,80
U,0x410288,0x410388,0x1405f,64,40
U,0x4102c8,0x4103c8,0xa01ef,40,22
U,0x4102f0,0x4103f0,0x80c7c,72,44
U,0x410338,0x410438,0x83c1b,24,8
U,0x410350,0x410450,0x83c1b,40,4
U,0x410378,0x410478,0xa1b0d,64,32
U,0x4103b8,0x4104b8,0x34f21,1040,1024
U,0x4107c8,0x4108c8,0x9f79d,40,9
U,0x4107f0,0x4108f0,0x9a079,192,172
U,0x4108b0,0x4109b0,0x9bf95,40,22
U,0x4108d8,0x4109d8,0x99f8f,224,208
U,0x4109b8,0x410ab8,0x9c46f,64,48
U,0x4109f8,0x410af8,0x9dfef,40,22
U,0x410a20,0x410b20,0x6e705,32,9
U,0x410a40,0x410b40,0xa01ef,56,22
U,0x410a78,0x410b78,0x7eab8,96,80
U,0x410ad8,0x410bd8,0x7eab8,96,80
U,0x410b38,0x410c38,0x28319,224,208
U,0x410c18,0x410d18,0x7eab8,96,80
U,0x410c78,0x410d78,0x285a9,2064,2048
U,0x411488,0x411588,0x285a9,112,92
U,0x4114f8,0x4115f8,0x80808,1120,1104
U,0x411958,0x411a58,0x803f4,4112,4096
U,0x412968,0x412a68,0x803f4,112,92
U,0x4129d8,0x412ad8,0x9d659,72,52
U,0x412a20,0x412b20,0x7e8ac,96,80
U,0x412a80,0x412b80,0x80808,1632,1616
U,0x4130e0,0x4131e0,0x803f4,2064,2048
U,0x4138f0,0x4139f0,0x803f4,112,92
U,0x413960,0x413a60,0x80808,256,240
U,0x413a60,0x413b60,0x803f4,1168,1152
U,0x413ef0,0x413ff0,0xa46c5,952,936
U,0x4142a8,0x4143a8,0xb1ff7,56,4
U,0x4142e0,0x4143e0,0x80808,176,160
U,0x414390,0x414490,0x803f4,4112,4096
U,0x4153a0,0x4154a0,0x803f4,112,92
U,0x415410,0x415510,0x7e8ac,96,80
U,0x415470,0x415570,0x803f4,112,92
U,0x4154e0,0x4155e0,0x80808,1296,1280
U,0x4159f0,0x415af0,0x803f4,112,92
U,0x415a60,0x415b60,0x7eab8,96,80
U,0x415ac0,0x415bc0,0x7e8ac,96,80
U,0x415b20,0x415c20,0x803f4,136,92
U,0x415ba8,0x415ca8,0x1405f,56,40
U,0x415be0,0x415ce0,0x1405f,56,40
U,0x415c18,0x415d18,0x13ed5,24,5
U,0x415c30,0x415d30,0x1405f,56,40
U,0x415c68,0x415d68,0x13ed5,24,8
U,0x415c80,0x415d80,0x1405f,56,40
U,0x415cb8,0x415db8,0x13ed5,24,7
U,0x415cd0,0x415dd0,0x1405f,56,40
U,0x415d08,0x415e08,0x13ed5,24,2
U,0x415d20,0x415e20,0x1405f,56,40
U,0x415d58,0x415e58,0x13ed5,24,7
U,0x415d70,0x415e70,0x1405f,56,40
U,0x415da8,0x415ea8,0x13ed5,32,12
U,0x415dc8,0x415ec8,0x1405f,56,40
U,0x415e00,0x415f00,0x13ed5,24,8
U,0x415e18,0x415f18,0x1405f,56,40
U,0x415e50,0x415f50,0x13ed5,32,11
U,0x415e70,0x415f70,0x13ed5,32,12
U,0x415e90,0x415f90,0x1405f,56,40
U,0x415ec8,0x415fc8,0x13ed5,32,13
U,0x415ee8,0x415fe8,0x13ed5,56,33
U,0x415f20,0x416020,0x1405f,56,40
U,0x415f58,0x416058,0x13ed5,32,11
U,0x415f78,0x416078,0x13ed5,24,8
U,0x415f90,0x416090,0x1405f,56,40
U,0x415fc8,0x4160c8,0x13ed5,24,7
U,0x415fe0,0x4160e0,0x1405f,56,40
U,0x416018,0x416118,0x13ed5,24,4
U,0x416030,0x416130,0x1405f,56,40
U,0x416068,0x416168,0x13ed5,24,2
U,0x416080,0x416180,0x13ed5,32,12
U,0x4160a0,0x4161a0,0x1405f,56,40
U,0x4160d8,0x4161d8,0x13ed5,24,2
U,0x4160f0,0x4161f0,0x13ed5,32,13
U,0x416110,0x416210,0x1405f,56,40
U,0x416148,0x416248,0x13ed5,24,4
U,0x416160,0x416260,0x1405f,56,40
U,0x416198,0x416298,0x13ed5,24,2
U,0x4161b0,0x4162b0,0x13ed5,32,12
U,0x4161d0,0x4162d0,0x1405f,56,40
U,0x416208,0x416308,0x13ed5,24,2
U,0x416220,0x416320,0x13ed5,32,13
U,0x416240,0x416340,0x1405f,56,40
U,0x416278,0x416378,0x13ed5,24,4
U,0x416290,0x416390,0x1405f,56,40
U,0x4162c8,0x4163c8,0x13ed5,24,2
U,0x4162e0,0x4163e0,0x1405f,56,40
U,0x416318,0x416418,0x13ed5,24,8
U,0x416330,0x416430,0x1405f,56,40
U,0x416368,0x416468,0x13ed5,32,11
U,0x416388,0x416488,0x13ed5,32,12
U,0x4163a8,0x4164a8,0x1405f,56,40
U,0x4163e0,0x4164e0,0x13ed5,32,13
U,0x416400,0x416500,0x13ed5,56,33
U,0x416438,0x416538,0x1405f,56,40
U,0x416470,0x416570,0x13ed5,32,11
U,0x416490,0x416590,0x13ed5,24,8
U,0x4164a8,0x4165a8,0x1405f,56,40
U,0x4164e0,0x4165e0,0x13ed5,24,7
U,0x4164f8,0x4165f8,0x1405f,56,40
U,0x416530,0x416630,0x13ed5,32,12
U,0x416550,0x416650,0x1405f,56,40
U,0x416588,0x416688,0x13ed5,24,7
U,0x4165a0,0x4166a0,0x1405f,56,40
U,0x4165d8,0x4166d8,0x13ed5,24,4
U,0x4165f0,0x4166f0,0x1405f,56,40
U,0x416628,0x416728,0x13ed5,24,2
U,0x416640,0x416740,0x13ed5,40,19
U,0x416668,0x416768,0x1405f,56,40
U,0x4166a0,0x4167a0,0x13ed5,24,2
U,0x4166b8,0x4167b8,0x13ed5,32,12
U,0x4166d8,0x4167d8,0x1405f,56,40
U,0x416710,0x416810,0x13ed5,24,4
U,0x416728,0x416828,0x1405f,56,40
U,0x416760,0x416860,0x13ed5,24,2
U,0x416778,0x416878,0x13ed5,40,19
U,0x4167a0,0x4168a0,0x1405f,56,40
U,0x4167d8,0x4168d8,0x13ed5,24,2
U,0x4167f0,0x4168f0,0x13ed5,32,12
U,0x416810,0x416910,0x1405f,56,40
U,0x416848,0x416948,0x13ed5,24,4
U,0x416860,0x416960,0x1405f,56,40
U,0x416898,0x416998,0x13ed5,24,2
U,0x4168b0,0x4169b0,0x1405f,56,40
U,0x4168e8,0x4169e8,0x13ed5,24,8
U,0x416900,0x416a00,0x1405f,56,40
U,0x416938,0x416a38,0x13ed5,32,11
U,0x416958,0x416a58,0x13ed5,32,12
U,0x416978,0x416a78,0x1405f,56,40
U,0x4169b0,0x416ab0,0x13ed5,32,13
U,0x4169d0,0x416ad0,0x13ed5,56,33
U,0x416a08,0x416b08,0x1405f,56,40
U,0x416a40,0x416b40,0x13ed5,32,11
U,0x416a60,0x416b60,0x13ed5,24,8
U,0x416a78,0x416b78,0x1405f,56,40
U,0x416ab0,0x416bb0,0x13ed5,24,7
U,0x416ac8,0x416bc8,0x1405f,56,40
U,0x416b00,0x416c00,0x13ed5,32,12
U,0x416b20,0x416c20,0x1405f,56,40
U,0x416b58,0x416c58,0x13ed5,24,7
U,0x416b70,0x416c70,0x1405f,56,40
U,0x416ba8,0x416ca8,0x13ed5,24,4
U,0x416bc0,0x416cc0,0x1405f,56,40
U,0x416bf8,0x416cf8,0x13ed5,24,2
U,0x416c10,0x416d10,0x13ed5,40,19
U,0x416c38,0x416d38,0x1405f,56,40
U,0x416c70,0x416d70,0x13ed5,24,2
U,0x416c88,0x416d88,0x13ed5,40,20
U,0x416cb0,0x416db0,0x1405f,56,40
U,0x416ce8,0x416de8,0x13ed5,24,4
U,0x416d00,0x416e00,0x1405f,56,40
U,0x416d38,0x416e38,0x13ed5,24,2
U,0x416d50,0x416e50,0x13ed5,40,19
U,0x416d78,0x416e78,0x1405f,56,40
U,0x416db0,0x416eb0,0x13ed5,24,2
U,0x416dc8,0x416ec8,0x13ed5,40,20
U,0x416df0,0x416ef0,0x1405f,56,40
U,0x416e28,0x416f28,0x13ed5,24,4
U,0x416e40,0x416f40,0x1405f,56,40
U,0x416e78,0x416f78,0x13ed5,24,2
U,0x416e90,0x416f90,0x1405f,56,40
U,0x416ec8,0x416fc8,0x13ed5,24,8
U,0x416ee0,0x416fe0,0x1405f,56,40
U,0x416f18,0x417018,0x13ed5,32,11
U,0x416f38,0x417038,0x13ed5,32,12
U,0x416f58,0x417058,0x1405f,56,40
U,0x416f90,0x417090,0x13ed5,32,13
U,0x416fb0,0x4170b0,0x13ed5,56,33
U,0x416fe8,0x4170e8,0x1405f,56,40
U,0x417020,0x417120,0x13ed5,32,11
U,0x417040,0x417140,0x13ed5,32,13
U,0x417060,0x417160,0x1405f,56,40
U,0x417098,0x417198,0x13ed5,24,7
U,0x4170b0,0x4171b0,0x1405f,56,40
U,0x4170e8,0x4171e8,0x13ed5,32,12
U,0x417108,0x417208,0x1405f,56,40
U,0x417140,0x417240,0x13ed5,24,7
U,0x417158,0x417258,0x1405f,56,40
U,0x417190,0x417290,0x13ed5,24,4
U,0x4171a8,0x4172a8,0x1405f,56,40
U,0x4171e0,0x4172e0,0x13ed5,24,2
U,0x4171f8,0x4172f8,0x13ed5,40,21
U,0x417220,0x417320,0x1405f,56,40
U,0x417258,0x417358,0x13ed5,24,2
U,0x417270,0x417370,0x13ed5,40,22
U,0x417298,0x417398,0x1405f,56,40
U,0x4172d0,0x4173d0,0x13ed5,24,4
U,0x4172e8,0x4173e8,0x1405f,56,40
U,0x417320,0x417420,0x13ed5,24,2
U,0x417338,0x417438,0x13ed5,40,21
U,0x417360,0x417460,0x1405f,56,40
U,0x417398,0x417498,0x13ed5,24,2
U,0x4173b0,0x4174b0,0x13ed5,40,22
U,0x4173d8,0x4174d8,0x1405f,56,40
U,0x417410,0x417510,0x13ed5,24,4
U,0x417428,0x417528,0x1405f,56,40
U,0x417460,0x417560,0x13ed5,24,2
U,0x417478,0x417578,0x1405f,56,40
U,0x4174b0,0x4175b0,0x13ed5,24,8
U,0x4174c8,0x4175c8,0x1405f,56,40
U,0x417500,0x417600,0x13ed5,32,11
U,0x417520,0x417620,0x13ed5,32,12
U,0x417540,0x417640,0x1405f,56,40
U,0x417578,0x417678,0x13ed5,32,13
U,0x417598,0x417698,0x13ed5,56,33
U,0x4175d0,0x4176d0,0x1405f,56,40
U,0x417608,0x417708,0x13ed5,32,11
U,0x417628,0x417728,0x13ed5,32,13
U,0x417648,0x417748,0x1405f,56,40
U,0x417680,0x417780,0x13ed5,24,7
U,0x417698,0x417798,0x1405f,56,40
U,0x4176d0,0x4177d0,0x13ed5,32,12
U,0x4176f0,0x4177f0,0x1405f,56,40
U,0x417728,0x417828,0x13ed5,24,7
U,0x417740,0x417840,0x1405f,56,40
U,0x417778,0x417878,0x13ed5,24,4
U,0x417790,0x417890,0x1405f,56,40
U,0x4177c8,0x4178c8,0x13ed5,24,2
U,0x4177e0,0x4178e0,0x13ed5,40,21
U,0x417808,0x417908,0x1405f,56,40
U,0x417840,0x417940,0x13ed5,24,2
U,0x417858,0x417958,0x13ed5,40,22
U,0x417880,0x417980,0x1405f,56,40
U,0x4178b8,0x4179b8,0x13ed5,24,4
U,0x4178d0,0x4179d0,0x1405f,56,40
U,0x417908,0x417a08,0x13ed5,24,2
U,0x417920,0x417a20,0x13ed5,40,21
U,0x417948,0x417a48,0x1405f,56,40
U,0x417980,0x417a80,0x13ed5,24,2
U,0x417998,0x417a98,0x13ed5,40,22
U,0x4179c0,0x417ac0,0x1405f,56,40
U,0x4179f8,0x417af8,0x13ed5,24,4
U,0x417a10,0x417b10,0x1405f,56,40
U,0x417a48,0x417b48,0x13ed5,24,8
U,0x417a60,0x417b60,0x1405f,56,40
U,0x417a98,0x417b98,0x1405f,56,40
U,0x417ad0,0x417bd0,0x13ed5,24,8
U,0x417ae8,0x417be8,0x1405f,56,40
U,0x417b20,0x417c20,0x13ed5,32,9
U,0x417b40,0x417c40,0x1405f,56,40
U,0x417b78,0x417c78,0x13ed5,32,9
U,0x417b98,0x417c98,0x1405f,56,40
U,0x417bd0,0x417cd0,0x13ed5,24,6
U,0x417be8,0x417ce8,0x1405f,56,40
U,0x417c20,0x417d20,0x1405f,56,40
U,0x417c58,0x417d58,0x13ed5,24,8
U,0x417c70,0x417d70,0x1405f,56,40
U,0x417ca8,0x417da8,0x13ed5,32,9
U,0x417cc8,0x417dc8,0x1405f,56,40
U,0x417d00,0x417e00,0x13ed5,32,9
U,0x417d20,0x417e20,0x1405f,56,40
U,0x417d58,0x417e58,0x13ed5,24,6
U,0x417d70,0x417e70,0x1405f,56,40
U,0x417da8,0x417ea8,0x1405f,56,40
U,0x417de0,0x417ee0,0x13ed5,24,8
U,0x417df8,0x417ef8,0x1405f,56,40
U,0x417e30,0x417f30,0x13ed5,32,9
U,0x417e50,0x417f50,0x1405f,56,40
U,0x417e88,0x417f88,0x13ed5,32,9
U,0x417ea8,0x417fa8,0x1405f,56,40
U,0x417ee0,0x417fe0,0x13ed5,24,6
U,0x417ef8,0x417ff8,0x1405f,56,40
U,0x417f30,0x418030,0x1405f,56,40
U,0x417f68,0x418068,0x13ed5,24,8
U,0x417f80,0x418080,0x1405f,56,40
U,0x417fb8,0x4180b8,0x13ed5,32,9
U,0x417fd8,0x4180d8,0x1405f,56,40
U,0x418010,0x418110,0x13ed5,32,9
U,0x418030,0x418130,0x1405f,56,40
U,0x418068,0x418168,0x13ed5,24,6
U,0x418080,0x418180,0x1405f,56,40
U,0x4180b8,0x4181b8,0x1405f,56,40
U,0x4180f0,0x4181f0,0x13ed5,24,8
U,0x418108,0x418208,0x1405f,56,40
U,0x418140,0x418240,0x13ed5,32,9
U,0x418160,0x418260,0x1405f,56,40
U,0x418198,0x418298,0x13ed5,32,9
F,0x4181b8,0x4182b8,0x9594d,80,63
U,0x418208,0x418308,0x1405f,56,40
U,0x418240,0x418340,0x13ed5,32,10
U,0x418260,0x418360,0x1405f,56,40
U,0x418298,0x418398,0x13ed5,24,2
U,0x4182b0,0x4183b0,0x1405f,56,40
U,0x4182e8,0x4183e8,0x13ed5,32,9
U,0x418308,0x418408,0x1405f,56,40
U,0x418340,0x418440,0x13ed5,24,5
U,0x418358,0x418458,0x1405f,56,40
U,0x418390,0x418490,0x13ed5,24,8
U,0x4183a8,0x4184a8,0x1405f,56,40
U,0x4183e0,0x4184e0,0x13ed5,24,5
U,0x4183f8,0x4184f8,0x1405f,56,40
U,0x418430,0x418530,0x13ed5,24,5
U,0x418448,0x418548,0x13ed5,24,1
U,0x418460,0x418560,0x1405f,56,40
U,0x418498,0x418598,0x13ed5,24,7
U,0x4184b0,0x4185b0,0x1405f,56,40
U,0x4184e8,0x4185e8,0x13ed5,24,2
U,0x418500,0x418600,0x1405f,56,40
U,0x418538,0x418638,0x13ed5,32,9
U,0x418558,0x418658,0x1405f,56,40
U,0x418590,0x418690,0x13ed5,24,5
U,0x4185a8,0x4186a8,0x1405f,56,40
U,0x4185e0,0x4186e0,0x13ed5,24,8
U,0x4185f8,0x4186f8,0x1405f,56,40
U,0x418630,0x418730,0x13ed5,24,5
U,0x418648,0x418748,0x1405f,56,40
U,0x418680,0x418780,0x13ed5,24,5
U,0x418698,0x418798,0x13ed5,24,1
U,0x4186b0,0x4187b0,0x1405f,56,40
U,0x4186e8,0x4187e8,0x13ed5,24,7
U,0x418700,0x418800,0x1405f,56,40
U,0x418738,0x418838,0x13ed5,24,2
U,0x418750,0x418850,0x1405f,56,40
U,0x418788,0x418888,0x13ed5,32,9
U,0x4187a8,0x4188a8,0x1405f,56,40
U,0x4187e0,0x4188e0,0x13ed5,24,5
U,0x4187f8,0x4188f8,0x1405f,56,40
U,0x418830,0x418930,0x13ed5,24,8
U,0x418848,0x418948,0x1405f,56,40
U,0x418880,0x418980,0x13ed5,24,5
U,0x418898,0x418998,0x1405f,56,40
U,0x4188d0,0x4189d0,0x13ed5,24,5
U,0x4188e8,0x4189e8,0x13ed5,24,1
U,0x418900,0x418a00,0x1405f,56,40
U,0x418938,0x418a38,0x13ed5,24,7
U,0x418950,0x418a50,0x3dc13,2720,2704
U,0x4193f0,0x4194f0,0x803f4,3088,3072
U,0x41a000,0x41a100,0x803f4,3088,3072
U,0x41ac10,0x41ad10,0x80808,896,880
U,0x41af90,0x41b090,0x803f4,1040,1024
U,0x41b3a0,0x41b4a0,0x803f4,12304,12288
U,0x41e3b0,0x41e4b0,0x803f4,112,92
U,0x41e420,0x41e520,0x80808,1296,1280
U,0x41e930,0x41ea30,0x803f4,5136,5120
U,0x41fd40,0x41fe40,0x803f4,112,92
U,0x41fdb0,0x41feb0,0x803f4,4112,4096
U,0x420dc0,0x420ec0,0x803f4,112,92
F,0x420e30,0x420f30,0x956db,64,36
U,0x420e70,0x420f70,0x42d97,40,20
U,0x420e98,0x420f98,0x7eab8,96,80
U,0x420ef8,0x420ff8,0x42d97,40,20
U,0x420f20,0x421020,0x7eab8,96,80
U,0x420f80,0x421080,0x42d97,40,20
U,0x420fa8,0x4210a8,0x7eab8,96,80
U,0x421008,0x421108,0x42d97,40,20
U,0x421030,0x421130,0x7eab8,96,80
U,0x421090,0x421190,0x42d97,40,20
U,0x4210b8,0x4211b8,0x7eab8,96,80
U,0x421118,0x421218,0x42d97,40,20
U,0x421140,0x421240,0x7eab8,96,80
U,0x4211a0,0x4212a0,0x7eab8,96,80
U,0x421200,0x421300,0x80c7c,64,44
U,0x421240,0x421340,0x7e8ac,96,80
U,0x4212a0,0x4213a0,0x7eab8,96,80
U,0x421300,0x421400,0x7e8ac,96,80
U,0x421360,0x421460,0x7e8ac,96,80
U,0x4213c0,0x4214c0,0x7e8ac,96,80
U,0x421420,0x421520,0x803f4,1040,1024
U,0x421830,0x421930,0x803f4,112,92
U,0x4218a0,0x4219a0,0x803f4,1040,1024
U,0x421cb0,0x421db0,0x803f4,112,92
U,0x421d20,0x421e20,0x7e8ac,96,80
U,0x421d80,0x421e80,0x7eab8,96,80
U,0x421de0,0x421ee0,0x961bb,24,4
U,0x421df8,0x421ef8,0x97b51,376,360
U,0x421f70,0x422070,0x6e705,32,11
U,0x421f90,0x422090,0x9642b,328,312
U,0x4220d8,0x4221d8,0x96435,472,456
U,0x4222b0,0x4223b0,0x96455,24,4
U,0x4222c8,0x4223c8,0x6e705,32,9
U,0x4222e8,0x4223e8,0x97b9d,24,4
U,0x422300,0x422400,0x97aa1,288,268
U,0x422420,0x422520,0x97e8b,24,4
U,0x422438,0x422538,0xa4c29,24,4
U,0x422450,0x422550,0x9ec37,88,72
U,0x4224a8,0x4225a8,0x95a9f,48,28
U,0x4224d8,0x4225d8,0x95a9f,48,28
U,0x422508,0x422608,0x9ef85,56,40
U,0x422540,0x422640,0x95a9f,48,28
F,0x422570,0x422670,0x95943,32,16
U,0x422590,0x422690,0xaf3ed,48,12
U,0x4225c0,0x4226c0,0x9d865,72,56
U,0x422608,0x422708,0x9d88b,1472,1456
U,0x422bc8,0x422cc8,0x9d893,64,48
U,0x422c08,0x422d08,0x986b3,40,20
U,0x422c30,0x422d30,0x986e7,112,96
U,0x422ca0,0x422da0,0x96559,104,88
U,0x422d08,0x422e08,0xb1e05,360,342
U,0x422e70,0x422f70,0x83c1b,48,26
F,0x422ea0,0x422fa0,0x9594d,216,130
U,0x422f78,0x423078,0x140f3,40,6
U,0x422fa0,0x4230a0,0x9fe83,96,80
F,0x423000,0x423100,0x8c255,176,130
U,0x4230b0,0x4231b0,0x83c1b,24,1
U,0x4230c8,0x4231c8,0x83c1b,40,20
F,0x4230f0,0x4231f0,0x812b7,40,20
U,0x423118,0x423218,0xb25ff,496,476
U,0x423308,0x423408,0x9e865,88,72
U,0x423360,0x423460,0x95a9f,48,28
U,0x423390,0x423490,0x95a9f,48,28
U,0x4233c0,0x4234c0,0xa4883,56,40
U,0x4233f8,0x4234f8,0xa350b,112,96
U,0x423468,0x423568,0xb0193,1048,1028
U,0x423880,0x423980,0xaf9f7,40,20
U,0x4238a8,0x4239a8,0x9d865,72,56
U,0x4238f0,0x4239f0,0x9d88b,1472,1456
U,0x423eb0,0x423fb0,0x9d893,64,48
U,0x423ef0,0x423ff0,0x9ef85,56,40
U,0x423f28,0x424028,0x95a9f,48,28
U,0x423f58,0x424058,0x9fa4f,64,48
U,0x423f98,0x424098,0x9f79d,32,9
U,0x423fb8,0x4240b8,0x6e705,32,9
U,0x423fd8,0x4240d8,0xb1fa5,144,128
F,0x424068,0x424168,0x9e9a1,3200,512
U,0x424ce8,0x424de8,0x803f4,2064,2048
U,0x4254f8,0x4255f8,0x803f4,112,92
F,0x425568,0x425668,0x95b35,32,16
U,0x425588,0x425688,0x803f4,2064,2048
U,0x425d98,0x425e98,0x803f4,112,92
F,0x425e08,0x425f08,0x14d11,41448,1932
```

打开 mqtt，然后再关闭 mqtt 之后的 heap info:

```bash
address: 0x40aae8
size: 152856
avail: 43120
pool_start: 0x40aae8
pool_end: 0x42fff0
state,block_addr,user_addr,caller,blocksize,wanted_size
U,0x40aae8,0x40abe8,0x1b695,4112,4096
U,0x40baf8,0x40bbf8,0x1b6b5,40,20
U,0x40bb20,0x40bc20,0x1b695,528,512
U,0x40bd30,0x40be30,0x1b6b5,40,20
U,0x40bd58,0x40be58,0x1b695,4112,4096
U,0x40cd68,0x40ce68,0x1b6b5,40,20
U,0x40cd90,0x40ce90,0x1b695,528,512
U,0x40cfa0,0x40d0a0,0x1b6b5,40,20
U,0x40cfc8,0x40d0c8,0x7eab8,96,80
U,0x40d028,0x40d128,0x7e8ac,96,80
U,0x40d088,0x40d188,0x28319,112,92
U,0x40d0f8,0x40d1f8,0x28441,96,80
U,0x40d158,0x40d258,0x7eab8,96,80
U,0x40d1b8,0x40d2b8,0x7eab8,96,80
U,0x40d218,0x40d318,0x28319,112,92
U,0x40d288,0x40d388,0x28441,96,80
U,0x40d2e8,0x40d3e8,0x7eab8,96,80
U,0x40d348,0x40d448,0x7eab8,96,80
U,0x40d3a8,0x40d4a8,0x28319,112,92
U,0x40d418,0x40d518,0x28441,96,80
U,0x40d478,0x40d578,0x7eab8,96,80
U,0x40d4d8,0x40d5d8,0x7eab8,96,80
U,0x40d538,0x40d638,0x803f4,112,92
U,0x40d5a8,0x40d6a8,0xa4be5,56,36
U,0x40d5e0,0x40d6e0,0x9fb53,48,28
U,0x40d610,0x40d710,0xb22f3,760,744
U,0x40d908,0x40da08,0x7f51c,1040,1024
U,0x40dd18,0x40de18,0x7f51c,112,92
U,0x40dd88,0x40de88,0x7fe1c,288,272
U,0x40dea8,0x40dfa8,0x7fefc,3088,3072
U,0x40eab8,0x40ebb8,0x7fefc,112,92
U,0x40eb28,0x40ec28,0x15769,32,16
U,0x40eb48,0x40ec48,0x15789,128,108
U,0x40ebc8,0x40ecc8,0x157a9,136,114
U,0x40ec50,0x40ed50,0x80808,120,100
U,0x40ecc8,0x40edc8,0x803f4,1040,1024
U,0x40f0d8,0x40f1d8,0x803f4,112,92
U,0x40f148,0x40f248,0x7e8ac,96,80
U,0x40f1a8,0x40f2a8,0x803f4,4112,4096
U,0x4101b8,0x4102b8,0x803f4,112,92
U,0x410228,0x410328,0x28441,96,80
U,0x410288,0x410388,0x1405f,64,40
U,0x4102c8,0x4103c8,0xa01ef,40,22
U,0x4102f0,0x4103f0,0x80c7c,72,44
U,0x410338,0x410438,0x83c1b,24,8
U,0x410350,0x410450,0x83c1b,40,4
U,0x410378,0x410478,0xa1b0d,64,32
U,0x4103b8,0x4104b8,0x34f21,1040,1024
U,0x4107c8,0x4108c8,0x9f79d,40,9
U,0x4107f0,0x4108f0,0x9a079,192,172
U,0x4108b0,0x4109b0,0x9bf95,40,22
U,0x4108d8,0x4109d8,0x99f8f,224,208
U,0x4109b8,0x410ab8,0x9c46f,64,48
U,0x4109f8,0x410af8,0x9dfef,40,22
U,0x410a20,0x410b20,0x6e705,32,9
U,0x410a40,0x410b40,0xa01ef,56,22
U,0x410a78,0x410b78,0x7eab8,96,80
U,0x410ad8,0x410bd8,0x7eab8,96,80
U,0x410b38,0x410c38,0x28319,224,208
U,0x410c18,0x410d18,0x7eab8,96,80
U,0x410c78,0x410d78,0x285a9,2064,2048
U,0x411488,0x411588,0x285a9,112,92
U,0x4114f8,0x4115f8,0x80808,1120,1104
U,0x411958,0x411a58,0x803f4,4112,4096
U,0x412968,0x412a68,0x803f4,112,92
U,0x4129d8,0x412ad8,0x9d659,72,52
U,0x412a20,0x412b20,0x7e8ac,96,80
U,0x412a80,0x412b80,0x80808,1632,1616
U,0x4130e0,0x4131e0,0x803f4,2064,2048
U,0x4138f0,0x4139f0,0x803f4,112,92
U,0x413960,0x413a60,0x80808,256,240
U,0x413a60,0x413b60,0x803f4,1168,1152
U,0x413ef0,0x413ff0,0xa46c5,952,936
U,0x4142a8,0x4143a8,0xb1ff7,56,4
U,0x4142e0,0x4143e0,0x80808,176,160
U,0x414390,0x414490,0x803f4,4112,4096
U,0x4153a0,0x4154a0,0x803f4,112,92
U,0x415410,0x415510,0x7e8ac,96,80
U,0x415470,0x415570,0x803f4,112,92
U,0x4154e0,0x4155e0,0x80808,1296,1280
U,0x4159f0,0x415af0,0x803f4,112,92
U,0x415a60,0x415b60,0x7eab8,96,80
U,0x415ac0,0x415bc0,0x7e8ac,96,80
U,0x415b20,0x415c20,0x803f4,136,92
U,0x415ba8,0x415ca8,0x1405f,56,40
U,0x415be0,0x415ce0,0x1405f,56,40
U,0x415c18,0x415d18,0x13ed5,24,5
U,0x415c30,0x415d30,0x1405f,56,40
U,0x415c68,0x415d68,0x13ed5,24,8
U,0x415c80,0x415d80,0x1405f,56,40
U,0x415cb8,0x415db8,0x13ed5,24,7
U,0x415cd0,0x415dd0,0x1405f,56,40
U,0x415d08,0x415e08,0x13ed5,24,2
U,0x415d20,0x415e20,0x1405f,56,40
U,0x415d58,0x415e58,0x13ed5,24,7
U,0x415d70,0x415e70,0x1405f,56,40
U,0x415da8,0x415ea8,0x13ed5,32,12
U,0x415dc8,0x415ec8,0x1405f,56,40
U,0x415e00,0x415f00,0x13ed5,24,8
U,0x415e18,0x415f18,0x1405f,56,40
U,0x415e50,0x415f50,0x13ed5,32,11
U,0x415e70,0x415f70,0x13ed5,32,12
U,0x415e90,0x415f90,0x1405f,56,40
U,0x415ec8,0x415fc8,0x13ed5,32,13
U,0x415ee8,0x415fe8,0x13ed5,56,33
U,0x415f20,0x416020,0x1405f,56,40
U,0x415f58,0x416058,0x13ed5,32,11
U,0x415f78,0x416078,0x13ed5,24,8
U,0x415f90,0x416090,0x1405f,56,40
U,0x415fc8,0x4160c8,0x13ed5,24,7
U,0x415fe0,0x4160e0,0x1405f,56,40
U,0x416018,0x416118,0x13ed5,24,4
U,0x416030,0x416130,0x1405f,56,40
U,0x416068,0x416168,0x13ed5,24,2
U,0x416080,0x416180,0x13ed5,32,12
U,0x4160a0,0x4161a0,0x1405f,56,40
U,0x4160d8,0x4161d8,0x13ed5,24,2
U,0x4160f0,0x4161f0,0x13ed5,32,13
U,0x416110,0x416210,0x1405f,56,40
U,0x416148,0x416248,0x13ed5,24,4
U,0x416160,0x416260,0x1405f,56,40
U,0x416198,0x416298,0x13ed5,24,2
U,0x4161b0,0x4162b0,0x13ed5,32,12
U,0x4161d0,0x4162d0,0x1405f,56,40
U,0x416208,0x416308,0x13ed5,24,2
U,0x416220,0x416320,0x13ed5,32,13
U,0x416240,0x416340,0x1405f,56,40
U,0x416278,0x416378,0x13ed5,24,4
U,0x416290,0x416390,0x1405f,56,40
U,0x4162c8,0x4163c8,0x13ed5,24,2
U,0x4162e0,0x4163e0,0x1405f,56,40
U,0x416318,0x416418,0x13ed5,24,8
U,0x416330,0x416430,0x1405f,56,40
U,0x416368,0x416468,0x13ed5,32,11
U,0x416388,0x416488,0x13ed5,32,12
U,0x4163a8,0x4164a8,0x1405f,56,40
U,0x4163e0,0x4164e0,0x13ed5,32,13
U,0x416400,0x416500,0x13ed5,56,33
U,0x416438,0x416538,0x1405f,56,40
U,0x416470,0x416570,0x13ed5,32,11
U,0x416490,0x416590,0x13ed5,24,8
U,0x4164a8,0x4165a8,0x1405f,56,40
U,0x4164e0,0x4165e0,0x13ed5,24,7
U,0x4164f8,0x4165f8,0x1405f,56,40
U,0x416530,0x416630,0x13ed5,32,12
U,0x416550,0x416650,0x1405f,56,40
U,0x416588,0x416688,0x13ed5,24,7
U,0x4165a0,0x4166a0,0x1405f,56,40
U,0x4165d8,0x4166d8,0x13ed5,24,4
U,0x4165f0,0x4166f0,0x1405f,56,40
U,0x416628,0x416728,0x13ed5,24,2
U,0x416640,0x416740,0x13ed5,40,19
U,0x416668,0x416768,0x1405f,56,40
U,0x4166a0,0x4167a0,0x13ed5,24,2
U,0x4166b8,0x4167b8,0x13ed5,32,12
U,0x4166d8,0x4167d8,0x1405f,56,40
U,0x416710,0x416810,0x13ed5,24,4
U,0x416728,0x416828,0x1405f,56,40
U,0x416760,0x416860,0x13ed5,24,2
U,0x416778,0x416878,0x13ed5,40,19
U,0x4167a0,0x4168a0,0x1405f,56,40
U,0x4167d8,0x4168d8,0x13ed5,24,2
U,0x4167f0,0x4168f0,0x13ed5,32,12
U,0x416810,0x416910,0x1405f,56,40
U,0x416848,0x416948,0x13ed5,24,4
U,0x416860,0x416960,0x1405f,56,40
U,0x416898,0x416998,0x13ed5,24,2
U,0x4168b0,0x4169b0,0x1405f,56,40
U,0x4168e8,0x4169e8,0x13ed5,24,8
U,0x416900,0x416a00,0x1405f,56,40
U,0x416938,0x416a38,0x13ed5,32,11
U,0x416958,0x416a58,0x13ed5,32,12
U,0x416978,0x416a78,0x1405f,56,40
U,0x4169b0,0x416ab0,0x13ed5,32,13
U,0x4169d0,0x416ad0,0x13ed5,56,33
U,0x416a08,0x416b08,0x1405f,56,40
U,0x416a40,0x416b40,0x13ed5,32,11
U,0x416a60,0x416b60,0x13ed5,24,8
U,0x416a78,0x416b78,0x1405f,56,40
U,0x416ab0,0x416bb0,0x13ed5,24,7
U,0x416ac8,0x416bc8,0x1405f,56,40
U,0x416b00,0x416c00,0x13ed5,32,12
U,0x416b20,0x416c20,0x1405f,56,40
U,0x416b58,0x416c58,0x13ed5,24,7
U,0x416b70,0x416c70,0x1405f,56,40
U,0x416ba8,0x416ca8,0x13ed5,24,4
U,0x416bc0,0x416cc0,0x1405f,56,40
U,0x416bf8,0x416cf8,0x13ed5,24,2
U,0x416c10,0x416d10,0x13ed5,40,19
U,0x416c38,0x416d38,0x1405f,56,40
U,0x416c70,0x416d70,0x13ed5,24,2
U,0x416c88,0x416d88,0x13ed5,40,20
U,0x416cb0,0x416db0,0x1405f,56,40
U,0x416ce8,0x416de8,0x13ed5,24,4
U,0x416d00,0x416e00,0x1405f,56,40
U,0x416d38,0x416e38,0x13ed5,24,2
U,0x416d50,0x416e50,0x13ed5,40,19
U,0x416d78,0x416e78,0x1405f,56,40
U,0x416db0,0x416eb0,0x13ed5,24,2
U,0x416dc8,0x416ec8,0x13ed5,40,20
U,0x416df0,0x416ef0,0x1405f,56,40
U,0x416e28,0x416f28,0x13ed5,24,4
U,0x416e40,0x416f40,0x1405f,56,40
U,0x416e78,0x416f78,0x13ed5,24,2
U,0x416e90,0x416f90,0x1405f,56,40
U,0x416ec8,0x416fc8,0x13ed5,24,8
U,0x416ee0,0x416fe0,0x1405f,56,40
U,0x416f18,0x417018,0x13ed5,32,11
U,0x416f38,0x417038,0x13ed5,32,12
U,0x416f58,0x417058,0x1405f,56,40
U,0x416f90,0x417090,0x13ed5,32,13
U,0x416fb0,0x4170b0,0x13ed5,56,33
U,0x416fe8,0x4170e8,0x1405f,56,40
U,0x417020,0x417120,0x13ed5,32,11
U,0x417040,0x417140,0x13ed5,32,13
U,0x417060,0x417160,0x1405f,56,40
U,0x417098,0x417198,0x13ed5,24,7
U,0x4170b0,0x4171b0,0x1405f,56,40
U,0x4170e8,0x4171e8,0x13ed5,32,12
U,0x417108,0x417208,0x1405f,56,40
U,0x417140,0x417240,0x13ed5,24,7
U,0x417158,0x417258,0x1405f,56,40
U,0x417190,0x417290,0x13ed5,24,4
U,0x4171a8,0x4172a8,0x1405f,56,40
U,0x4171e0,0x4172e0,0x13ed5,24,2
U,0x4171f8,0x4172f8,0x13ed5,40,21
U,0x417220,0x417320,0x1405f,56,40
U,0x417258,0x417358,0x13ed5,24,2
U,0x417270,0x417370,0x13ed5,40,22
U,0x417298,0x417398,0x1405f,56,40
U,0x4172d0,0x4173d0,0x13ed5,24,4
U,0x4172e8,0x4173e8,0x1405f,56,40
U,0x417320,0x417420,0x13ed5,24,2
U,0x417338,0x417438,0x13ed5,40,21
U,0x417360,0x417460,0x1405f,56,40
U,0x417398,0x417498,0x13ed5,24,2
U,0x4173b0,0x4174b0,0x13ed5,40,22
U,0x4173d8,0x4174d8,0x1405f,56,40
U,0x417410,0x417510,0x13ed5,24,4
U,0x417428,0x417528,0x1405f,56,40
U,0x417460,0x417560,0x13ed5,24,2
U,0x417478,0x417578,0x1405f,56,40
U,0x4174b0,0x4175b0,0x13ed5,24,8
U,0x4174c8,0x4175c8,0x1405f,56,40
U,0x417500,0x417600,0x13ed5,32,11
U,0x417520,0x417620,0x13ed5,32,12
U,0x417540,0x417640,0x1405f,56,40
U,0x417578,0x417678,0x13ed5,32,13
U,0x417598,0x417698,0x13ed5,56,33
U,0x4175d0,0x4176d0,0x1405f,56,40
U,0x417608,0x417708,0x13ed5,32,11
U,0x417628,0x417728,0x13ed5,32,13
U,0x417648,0x417748,0x1405f,56,40
U,0x417680,0x417780,0x13ed5,24,7
U,0x417698,0x417798,0x1405f,56,40
U,0x4176d0,0x4177d0,0x13ed5,32,12
U,0x4176f0,0x4177f0,0x1405f,56,40
U,0x417728,0x417828,0x13ed5,24,7
U,0x417740,0x417840,0x1405f,56,40
U,0x417778,0x417878,0x13ed5,24,4
U,0x417790,0x417890,0x1405f,56,40
U,0x4177c8,0x4178c8,0x13ed5,24,2
U,0x4177e0,0x4178e0,0x13ed5,40,21
U,0x417808,0x417908,0x1405f,56,40
U,0x417840,0x417940,0x13ed5,24,2
U,0x417858,0x417958,0x13ed5,40,22
U,0x417880,0x417980,0x1405f,56,40
U,0x4178b8,0x4179b8,0x13ed5,24,4
U,0x4178d0,0x4179d0,0x1405f,56,40
U,0x417908,0x417a08,0x13ed5,24,2
U,0x417920,0x417a20,0x13ed5,40,21
U,0x417948,0x417a48,0x1405f,56,40
U,0x417980,0x417a80,0x13ed5,24,2
U,0x417998,0x417a98,0x13ed5,40,22
U,0x4179c0,0x417ac0,0x1405f,56,40
U,0x4179f8,0x417af8,0x13ed5,24,4
U,0x417a10,0x417b10,0x1405f,56,40
U,0x417a48,0x417b48,0x13ed5,24,8
U,0x417a60,0x417b60,0x1405f,56,40
U,0x417a98,0x417b98,0x1405f,56,40
U,0x417ad0,0x417bd0,0x13ed5,24,8
U,0x417ae8,0x417be8,0x1405f,56,40
U,0x417b20,0x417c20,0x13ed5,32,9
U,0x417b40,0x417c40,0x1405f,56,40
U,0x417b78,0x417c78,0x13ed5,32,9
U,0x417b98,0x417c98,0x1405f,56,40
U,0x417bd0,0x417cd0,0x13ed5,24,6
U,0x417be8,0x417ce8,0x1405f,56,40
U,0x417c20,0x417d20,0x1405f,56,40
U,0x417c58,0x417d58,0x13ed5,24,8
U,0x417c70,0x417d70,0x1405f,56,40
U,0x417ca8,0x417da8,0x13ed5,32,9
U,0x417cc8,0x417dc8,0x1405f,56,40
U,0x417d00,0x417e00,0x13ed5,32,9
U,0x417d20,0x417e20,0x1405f,56,40
U,0x417d58,0x417e58,0x13ed5,24,6
U,0x417d70,0x417e70,0x1405f,56,40
U,0x417da8,0x417ea8,0x1405f,56,40
U,0x417de0,0x417ee0,0x13ed5,24,8
U,0x417df8,0x417ef8,0x1405f,56,40
U,0x417e30,0x417f30,0x13ed5,32,9
U,0x417e50,0x417f50,0x1405f,56,40
U,0x417e88,0x417f88,0x13ed5,32,9
U,0x417ea8,0x417fa8,0x1405f,56,40
U,0x417ee0,0x417fe0,0x13ed5,24,6
U,0x417ef8,0x417ff8,0x1405f,56,40
U,0x417f30,0x418030,0x1405f,56,40
U,0x417f68,0x418068,0x13ed5,24,8
U,0x417f80,0x418080,0x1405f,56,40
U,0x417fb8,0x4180b8,0x13ed5,32,9
U,0x417fd8,0x4180d8,0x1405f,56,40
U,0x418010,0x418110,0x13ed5,32,9
U,0x418030,0x418130,0x1405f,56,40
U,0x418068,0x418168,0x13ed5,24,6
U,0x418080,0x418180,0x1405f,56,40
U,0x4180b8,0x4181b8,0x1405f,56,40
U,0x4180f0,0x4181f0,0x13ed5,24,8
U,0x418108,0x418208,0x1405f,56,40
U,0x418140,0x418240,0x13ed5,32,9
U,0x418160,0x418260,0x1405f,56,40
U,0x418198,0x418298,0x13ed5,32,9
F,0x4181b8,0x4182b8,0x812b7,80,20
U,0x418208,0x418308,0x1405f,56,40
U,0x418240,0x418340,0x13ed5,32,10
U,0x418260,0x418360,0x1405f,56,40
U,0x418298,0x418398,0x13ed5,24,2
U,0x4182b0,0x4183b0,0x1405f,56,40
U,0x4182e8,0x4183e8,0x13ed5,32,9
U,0x418308,0x418408,0x1405f,56,40
U,0x418340,0x418440,0x13ed5,24,5
U,0x418358,0x418458,0x1405f,56,40
U,0x418390,0x418490,0x13ed5,24,8
U,0x4183a8,0x4184a8,0x1405f,56,40
U,0x4183e0,0x4184e0,0x13ed5,24,5
U,0x4183f8,0x4184f8,0x1405f,56,40
U,0x418430,0x418530,0x13ed5,24,5
U,0x418448,0x418548,0x13ed5,24,1
U,0x418460,0x418560,0x1405f,56,40
U,0x418498,0x418598,0x13ed5,24,7
U,0x4184b0,0x4185b0,0x1405f,56,40
U,0x4184e8,0x4185e8,0x13ed5,24,2
U,0x418500,0x418600,0x1405f,56,40
U,0x418538,0x418638,0x13ed5,32,9
U,0x418558,0x418658,0x1405f,56,40
U,0x418590,0x418690,0x13ed5,24,5
U,0x4185a8,0x4186a8,0x1405f,56,40
U,0x4185e0,0x4186e0,0x13ed5,24,8
U,0x4185f8,0x4186f8,0x1405f,56,40
U,0x418630,0x418730,0x13ed5,24,5
U,0x418648,0x418748,0x1405f,56,40
U,0x418680,0x418780,0x13ed5,24,5
U,0x418698,0x418798,0x13ed5,24,1
U,0x4186b0,0x4187b0,0x1405f,56,40
U,0x4186e8,0x4187e8,0x13ed5,24,7
U,0x418700,0x418800,0x1405f,56,40
U,0x418738,0x418838,0x13ed5,24,2
U,0x418750,0x418850,0x1405f,56,40
U,0x418788,0x418888,0x13ed5,32,9
U,0x4187a8,0x4188a8,0x1405f,56,40
U,0x4187e0,0x4188e0,0x13ed5,24,5
U,0x4187f8,0x4188f8,0x1405f,56,40
U,0x418830,0x418930,0x13ed5,24,8
U,0x418848,0x418948,0x1405f,56,40
U,0x418880,0x418980,0x13ed5,24,5
U,0x418898,0x418998,0x1405f,56,40
U,0x4188d0,0x4189d0,0x13ed5,24,5
U,0x4188e8,0x4189e8,0x13ed5,24,1
U,0x418900,0x418a00,0x1405f,56,40
U,0x418938,0x418a38,0x13ed5,24,7
U,0x418950,0x418a50,0x3dc13,2720,2704
U,0x4193f0,0x4194f0,0x803f4,3088,3072
U,0x41a000,0x41a100,0x803f4,3088,3072
U,0x41ac10,0x41ad10,0x80808,896,880
U,0x41af90,0x41b090,0x803f4,1040,1024
U,0x41b3a0,0x41b4a0,0x803f4,12304,12288
U,0x41e3b0,0x41e4b0,0x803f4,112,92
U,0x41e420,0x41e520,0x80808,1296,1280
U,0x41e930,0x41ea30,0x803f4,5136,5120
U,0x41fd40,0x41fe40,0x803f4,112,92
U,0x41fdb0,0x41feb0,0x803f4,4112,4096
U,0x420dc0,0x420ec0,0x803f4,112,92
F,0x420e30,0x420f30,0x956db,64,36
U,0x420e70,0x420f70,0x42d97,40,20
U,0x420e98,0x420f98,0x7eab8,96,80
U,0x420ef8,0x420ff8,0x42d97,40,20
U,0x420f20,0x421020,0x7eab8,96,80
U,0x420f80,0x421080,0x42d97,40,20
U,0x420fa8,0x4210a8,0x7eab8,96,80
U,0x421008,0x421108,0x42d97,40,20
U,0x421030,0x421130,0x7eab8,96,80
U,0x421090,0x421190,0x42d97,40,20
U,0x4210b8,0x4211b8,0x7eab8,96,80
U,0x421118,0x421218,0x42d97,40,20
U,0x421140,0x421240,0x7eab8,96,80
U,0x4211a0,0x4212a0,0x7eab8,96,80
U,0x421200,0x421300,0x80c7c,64,44
U,0x421240,0x421340,0x7e8ac,96,80
U,0x4212a0,0x4213a0,0x7eab8,96,80
U,0x421300,0x421400,0x7e8ac,96,80
U,0x421360,0x421460,0x7e8ac,96,80
U,0x4213c0,0x4214c0,0x7e8ac,96,80
U,0x421420,0x421520,0x803f4,1040,1024
U,0x421830,0x421930,0x803f4,112,92
U,0x4218a0,0x4219a0,0x803f4,1040,1024
U,0x421cb0,0x421db0,0x803f4,112,92
U,0x421d20,0x421e20,0x7e8ac,96,80
U,0x421d80,0x421e80,0x7eab8,96,80
U,0x421de0,0x421ee0,0x961bb,24,4
U,0x421df8,0x421ef8,0x97b51,376,360
U,0x421f70,0x422070,0x6e705,32,11
U,0x421f90,0x422090,0x9642b,328,312
U,0x4220d8,0x4221d8,0x96435,472,456
U,0x4222b0,0x4223b0,0x96455,24,4
U,0x4222c8,0x4223c8,0x6e705,32,9
U,0x4222e8,0x4223e8,0x97b9d,24,4
U,0x422300,0x422400,0x97aa1,288,268
U,0x422420,0x422520,0x97e8b,24,4
U,0x422438,0x422538,0xa4c29,24,4
U,0x422450,0x422550,0x9ec37,88,72
U,0x4224a8,0x4225a8,0x95a9f,48,28
U,0x4224d8,0x4225d8,0x95a9f,48,28
U,0x422508,0x422608,0x9ef85,56,40
U,0x422540,0x422640,0x95a9f,48,28
F,0x422570,0x422670,0x363c5,32,8
U,0x422590,0x422690,0xaf3ed,48,12
U,0x4225c0,0x4226c0,0x9d865,72,56
U,0x422608,0x422708,0x9d88b,1472,1456
U,0x422bc8,0x422cc8,0x9d893,64,48
U,0x422c08,0x422d08,0x986b3,40,20
U,0x422c30,0x422d30,0x986e7,112,96
U,0x422ca0,0x422da0,0x96559,104,88
U,0x422d08,0x422e08,0xb1e05,360,342
U,0x422e70,0x422f70,0x83c1b,48,26
F,0x422ea0,0x422fa0,0x9594d,256,88
U,0x422fa0,0x4230a0,0x9fe83,96,80
F,0x423000,0x423100,0x7e8ac,176,80
U,0x4230b0,0x4231b0,0x83c1b,24,1
U,0x4230c8,0x4231c8,0x83c1b,40,20
F,0x4230f0,0x4231f0,0x812b7,40,20
U,0x423118,0x423218,0xb25ff,496,476
U,0x423308,0x423408,0x9e865,88,72
U,0x423360,0x423460,0x95a9f,48,28
U,0x423390,0x423490,0x95a9f,48,28
U,0x4233c0,0x4234c0,0xa4883,56,40
U,0x4233f8,0x4234f8,0xa350b,112,96
U,0x423468,0x423568,0xb0193,1048,1028
U,0x423880,0x423980,0xaf9f7,40,20
U,0x4238a8,0x4239a8,0x9d865,72,56
U,0x4238f0,0x4239f0,0x9d88b,1472,1456
U,0x423eb0,0x423fb0,0x9d893,64,48
U,0x423ef0,0x423ff0,0x9ef85,56,40
U,0x423f28,0x424028,0x95a9f,48,28
U,0x423f58,0x424058,0x9fa4f,64,48
U,0x423f98,0x424098,0x9f79d,32,9
U,0x423fb8,0x4240b8,0x6e705,32,9
U,0x423fd8,0x4240d8,0xb1fa5,144,128
F,0x424068,0x424168,0x9e9a1,3200,512
U,0x424ce8,0x424de8,0x803f4,2064,2048
U,0x4254f8,0x4255f8,0x803f4,112,92
F,0x425568,0x425668,0x9e083,32,12
U,0x425588,0x425688,0x803f4,2064,2048
U,0x425d98,0x425e98,0x803f4,112,92
U,0x425e08,0x425f08,0x140f3,32,6
U,0x425e28,0x425f28,0x803f4,2064,2048
U,0x426638,0x426738,0x803f4,112,92
F,0x4266a8,0x4267a8,0x14d11,39240,1932

```

对比之下，比较明显地增加了几条新 heap 使用信息

```bash
U,0x425e08,0x425f08,0x140f3,32,6
U,0x425e28,0x425f28,0x803f4,2064,2048
U,0x426638,0x426738,0x803f4,112,92
```

再用 address2line 工具定位这几块内存是哪里申请的

```bash
toolchain/gcc-arm-none-eabi-5_4-2016q3/bin 
❯ ./arm-none-eabi-addr2line -e ~/FC41D/beken_freertos_sdk_release-SDK_3.0.21/out/beken7231_bsp.elf -f 0x140f3 0x803f4 0x803f4
cJSON_strdup
/home/x/FC41D/beken_freertos_sdk_release-SDK_3.0.21/demos/common/json/cJSON.c:67
rtos_create_thread
/home/x/FC41D/beken_freertos_sdk_release-SDK_3.0.21/beken378/os/FreeRTOSv9.0.0/rtos_pub.c:98
rtos_create_thread
/home/x/FC41D/beken_freertos_sdk_release-SDK_3.0.21/beken378/os/FreeRTOSv9.0.0/rtos_pub.c:98
```
/home/x/FC41D/beken_freertos_sdk_release-SDK_3.0.21/beken378/os/FreeRTOSv9.0.0/rtos_pub.c:98 对应的是创建线程的函数 rtos_create_thread

```c
int ali_iot_close(custom_func_ch_id_e func_id, void *hdl)
{
    ali_iot_mqtt_client_t *mqtt_client = hdl;

    if (mqtt_client == NULL)
    {
        return -1;
    }
    rtos_lock_mutex(&mqtt_client->lock);

    rtos_delete_thread(mqtt_client->task);
    MQTTDisconnect(&mqtt_client->client);
	NetworkDisconnect(&mqtt_client->network);
    mqtt_client->ev_cb(mqtt_client, QL_MQTT_CLOSE, (QL_MQTT_OK << 16)|QL_MQTT_CLOSE_BY_DISCONNECT, NULL, (void *)mqtt_client->func_id);
    if(mqtt_client->sign_mqtt)
    {
        free(mqtt_client->sign_mqtt);
    }
    if(mqtt_client->sub_topic[0])
    {
        free(mqtt_client->sub_topic[0]);
    }
    if(mqtt_client->sub_topic[1])
    {
        free(mqtt_client->sub_topic[1]);
    }
    rtos_unlock_mutex(&mqtt_client->lock);

    rtos_deinit_mutex(&mqtt_client->lock);
    free(mqtt_client);

    return 0;
}
```
分析代码之后，找到原因是关闭mqtt的时候，没有正确删除线程导致的内存泄漏。 rtos_delete_thread(mqtt_client->task); 传参不对，修改为 rtos_delete_thread(&mqtt_client->task);之后正常。

内存泄漏的判断方法：
1. 功能打开关闭之后，对比前后的 heap info 信息，看看有哪些新增加的内存分配信息，按照这个线索去排查。
2. 挂机压测一段时间之后，打印 heap info 信息，按照 caller 出现的次数排序，出现次数较多的前几个 caller，或者内存申请较多的caller，是该怀疑存在内存泄漏的对象。

### heap 内存写越界案例

free 的时候检测到内存块被破坏

```bash
mem corrupt,block:0x422ef8, blocksize:24, xWantedSize:4, caller:0x34747
dump block:
00000000:  00 00 00 00 18 00 00 00  ........
00000008:  04 00 00 00 47 47 03 00  ....GG..
00000010:  48 54 54 50 2F 31 2E 30  HTTP/1.0


```

调用者申请了4字节内存空间，分配的内存块大小是24字节，减去头部16字节，后面填充了4字节的0xfd,从dump的内存数据来看，填充的数据都被覆盖完了, 通过 caller:0x34747 查找代码位置

```bash
FC41D/beken_freertos_sdk_release-SDK_3.0.21 on  Iotbranch [!?⇡] 
❯ ./toolchain/gcc-arm-none-eabi-5_4-2016q3/bin/arm-none-eabi-addr2line -e Release/Debug/beken7231_bsp.elf  -f 0x34747
fs_open_custom
/home/x/FC41D/beken_freertos_sdk_release-SDK_3.0.21/beken378/func/lwip_intf/lwip-2.0.2/src/apps/httpd/custom_fsdata.c:239

```

对应代码：

```c
int fs_open_custom(struct fs_file *file, const char *name)
{
    char *wifi_data="{\"aps\":[{\"ssid\":\"123\"},{\"ssid\":\"test234\"}]}";
    int file_data_len = 0;
    
     /* this example only provides one file */
     if (!strcmp(name, "/wifilist.html"))
     {
         /* initialize fs_file correctly */
         memset(file, 0, sizeof(struct fs_file));
         file_data_len = strlen((char *)CUSTOM_FSDATA_WIFILIST_HTML_HEADER) + strlen(wifi_data) +2;
         file->pextension = malloc(sizeof(file_data_len));
         if (file->pextension != NULL)
         {
            memset(file->pextension, 0, file_data_len);
            file_data_len = snprintf(file->pextension, file_data_len - 1, "%s%s", (char *)CUSTOM_FSDATA_WIFILIST_HTML_HEADER, wifi_data);
        
            file->data = (const char *)file->pextension;
            file->len = file_data_len; /* don't send the trailing 0 */
            file->index = file->len;
            /* allow persisteng connections */
            file->flags = FS_FILE_FLAGS_HEADER_INCLUDED;
            return 1;
         }
     }
     else if(!strcmp(name, "/index.html"))
     {
         char *sta_state_data = NULL;
         uint32 uart_band = 9600;
         uint8 ssid[33] = {0};
         uint8 psk[64] = {0};

         int apon = 0;
         char *ap_state_data = NULL;

         if (custom_fsdata_sta_state() == 1)
         {
             sta_state_data = CUSTOM_FSDATA_STA_CONNECTED;
         }
         else
         {
             sta_state_data = CUSTOM_FSDATA_STA_DISCONNECTED;
         }

         custom_sw_ap_cfg_get(apssid, appass, &apon);

         if (apon == 1)
         {
             ap_state_data = "Up";
         }
         else
         {
             ap_state_data = "Down";
         }

         custom_config_uart_braud_get(&uart_band);

         custom_sta_cfg_get(ssid, psk);
         /* initialize fs_file correctly */
         memset(file, 0, sizeof(struct fs_file));
         file_data_len = strlen((char *)CUSTOM_FSDATA_INDEX_HTML_HEADER) + strlen((char *)CUSTOM_FSDATA_INDEX_HTML_FILE_FMT) + 10 /* for uart band */ + strlen((char *)ssid) + strlen(ap_state_data) + strlen(sta_state_data) + strlen((char *)apssid) + 1 /* \0 */;
         file->pextension = malloc(sizeof(file_data_len));
         if (file->pextension != NULL)
         {
            memset(file->pextension, 0, file_data_len);
            file_data_len = snprintf(file->pextension, file_data_len - 1, (char *)CUSTOM_FSDATA_INDEX_HTML_FILE_FMT, (char *)CUSTOM_FSDATA_INDEX_HTML_HEADER, (char *)ssid, sta_state_data, (char *)apssid, ap_state_data, uart_band);
        
            file->data = (const char *)file->pextension;
            file->len = file_data_len; /* don't send the trailing 0 */
            file->index = file->len;
            /* allow persisteng connections */
            file->flags = FS_FILE_FLAGS_HEADER_INCLUDED;
            return 1;
         }
     }
     return 0;
}
```

file->pextension = malloc(sizeof(file_data_len)); 传的参数传错了，sizeof(file_data_len) 是4字节，实际需要申请 file_data_len 字节，导致后面内存写越界了。

