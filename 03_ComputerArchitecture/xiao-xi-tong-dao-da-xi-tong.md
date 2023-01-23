# 小系統到大系統

根據莫爾定律，電腦的計算能力用一個很誇張的速度在增長，而且目前看起來並不會減慢。一般來說，高階系統，伺服器等電腦都不是只有一個CPU的系統。對於如何達成各個CPU的協作有很多種方式。

## 對稱多處理(Symmetric Multi-Processing)

對稱多處理系統(Symmetric Multi-Processing, SMP) 是一個系統裡面包含多個CPU的一個常見的配置。`symmetric` 這個字代表所有的CPU都是相同的(同樣的架構，同樣的時鐘速度)。在SMP系統中，多個處理器會共享其他人的資源(resources)，例如記憶體、硬碟空間之類的。

## 快取一致性(Cache Coherency)

大多數的情況下，系統中的 CPU獨立運作，每個人都有自己的一組暫存器、program counter等。儘管他們分別獨立的運行，有一個重要的部分需要嚴格的同步(synchronisation)。

這個東西就是 CPU 快取，怕你忘記提醒一下，快取就是一個小的記憶體位置，可以快速的主存取記憶裡面的資訊。如果一個CPU對主記憶體中的資料進行修改，然後其他的CPU還在讀自己手上本來有的快取資訊，這樣系統就不會



这是 CPU 缓存;记住缓存是一个快速访问内存的小区域，它反映了存储在主系统内存中的值。如果一个 CPU 修改主存储器中的数据而另一个 CPU 在其高速缓存中具有该存储器的旧副本，则系统显然不会处于一致状态。请注意，问题仅在处理器写入内存时发生，因为如果仅读取值，则数据将保持一致。

为了协调在所有处理器上保持高速缓存一致性，SMP 系统使用侦听(snooping)。侦听(snooping)是处理器侦听总线的地方，所有处理器都连接到该总线以进行缓存事件，并相应地更新其缓存。

一个执行此操作的协议是 MOESI 协议;代表修改（Modified），所有者（Owner），独家（Exclusive），共享（Shared），无效（Invalid）。这些中的每一个都是高速缓存行可以位于系统中的处理器上的状态。还有其他协议可以做多少，但它们都有相似的概念。下面我们检查 MOESI，以便您了解该过程需要什

****

This is the CPU cache; remember the cache is a small area of quickly accessible memory that mirrors values stored in main system memory. If one CPU modifies data in main memory and another CPU has an old copy of that memory in its cache the system will obviously not be in a consistent state. Note that the problem only occurs when processors are writing to memory, since if a value is only read the data will be consistent.

To co-ordinate keeping the cache coherent on all processors an SMP system uses _snooping_. Snooping is where a processor listens on a bus which all processors are connected to for cache events, and updates its cache accordingly.

One protocol for doing this is the _MOESI_ protocol; standing for Modified, Owner, Exclusive, Shared, Invalid. Each of these is a state that a cache line can be in on a processor in the system. There are other protocols for doing as much, however they all share similar concepts. Below we examine MOESI so you have an idea of what the process entails.

When a processor requires reading a cache line from main memory, it firstly has to snoop all other processors in the system to see if they currently know anything about that area of memory (e.g. have it cached). If it does not exist in any other process, then the processor can load the memory into cache and mark it as _exclusive_. When it writes to the cache, it then changes state to be _modified_. Here the specific details of the cache come into play; some caches will immediately write back the modified cache to system memory (known as a _write-through_ cache, because writes go through to main memory). Others will not, and leave the modified value only in the cache until it is evicted, when the cache becomes full for example.

The other case is where the processor snoops and finds that the value is in another processors cache. If this value has already been marked as _modified_, it will copy the data into its own cache and mark it as _shared_. It will send a message for the other processor (that we got the data from) to mark its cache line as _owner_. Now imagine that a third processor in the system wants to use that memory too. It will snoop and find both a _shared_ and a _owner_ copy; it will thus take its value from the _owner_ value. While all the other processors are only reading the value, the cache line stays _shared_ in the system. However, when one processor needs to update the value it sends an _invalidate_ message through the system. Any processors with that cache line must then mark it as invalid, because it not longer reflects the "true" value. When the processor sends the invalidate message, it marks the cache line as _modified_ in its cache and all others will mark as _invalid_ (note that if the cache line is _exclusive_ the processor knows that no other processor is depending on it so can avoid sending an invalidate message).

From this point the process starts all over. Thus whichever processor has the _modified_ value has the responsibility of writing the true value back to RAM when it is evicted from the cache. By thinking through the protocol you can see that this ensures consistency of cache lines between processors.

There are several issues with this system as the number of processors starts to increase. With only a few processors, the overhead of checking if another processor has the cache line (a read snoop) or invalidating the data in every other processor (invalidate snoop) is manageable; but as the number of processors increase so does the bus traffic. This is why SMP systems usually only scale up to around 8 processors.

Having the processors all on the same bus starts to present physical problems as well. Physical properties of wires only allow them to be laid out at certain distances from each other and to only have certain lengths. With processors that run at many gigahertz the speed of light starts to become a real consideration in how long it takes messages to move around a system.

Note that system software usually has no part in this process, although programmers should be aware of what the hardware is doing underneath in response to the programs they design to maximise performance.

``
