WiredTiger



```
#define WRITE_BARRIER() do{ __asm__ volatile("sfence" ::: "memory");}while(0)
```

```
typedef struct
{
        WT_CURSOR *cursor; //cursor里面塞了一个session
        WT_SESSION *session;	
}db_info_t;
每个session里塞了一个connection

```

```
/*获得session对应的connection*/
#define	S2C(session)	  ((WT_CONNECTION_IMPL *)(session)->iface.connection)

/* Get the btree for a session */
#define	S2BT(session)	   ((WT_BTREE *)(session)->dhandle->handle)
```



每个page对应的page_modify内部有page_lock_index  存在每个session中的connection中的page_lock里



blockmgr->block->slist，filehandle {slist}， block ckpt, extlist

extlist ext skiplist last,skiplist off[],skiplist sz[]

​	size* ext*有关系



btree ->  data_handle{rw_lock, data_source, slist{l,hash}, dsrc_stats{stats}}

​		ckpt

​		collator?

​		block mgr flush lock 

​		ref*	evict

​		ref root?

evict entry {btree * btree,ref * ref}

evict worker{thread,session*}

cache



ref一个page间接层 	-> page ,page state, page del ->page_modify(比较复杂)

​	-> home aka parent page root page

crusor btree, cursor ,btree* ref*,insert_head{skiplist array *2} insert(skiplist_node)



```
/*内存屏障宏定义*/

/*编译屏障,防止编译器乱序优化指令*/
#define WT_BARRIER()						__asm__ volatile("" ::: "memory")

/*用汇编指令PAUSE来防止过度消耗CPU,尤其是在空循环体中，加入PAUSE指令能防止CPU退出循环体时太多的异常指令消耗*/
#define WT_PAUSE()							__asm__ volatile("pause\n" ::: "memory")

/*定义CPU屏障*/
#if defined(x86_64) || defined(__x86_64__) /*64位CPU*/
#define WT_FULL_BARRIER() do{ __asm__ volatile("mfence" ::: "memory");}while(0)
#define WT_READ_BARRIER() do{ __asm__ volatile("lfence" ::: "memory");}while(0)
#define WT_WRITE_BARRIER() do{ __asm__ volatile("sfence" ::: "memory");}while(0)
#elif defined(i386) || defined(__i386__) /*32为CPU*/
#define WT_FULL_BARRIER() do{__asm__ volatile ("lock; addl $0, 0(%%esp)" ::: "memory");}while(0)
#define WT_READ_BARRIER() WT_FULL_BARRIER()
#define WT_WRITE_BARRIER() WT_FULL_BARRIER()
#else /*其他类CPU，直接提示编译错误*/
#error "No write barrier implementation for this hardware"
#endif
```

