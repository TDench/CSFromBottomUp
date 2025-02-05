# 記憶體與快取

## 記憶體架構

CPU 只能直接存處理器晶片上面的快取記憶體(cache)存取指令與數據。快取的資料是從主記憶體(main system memory, 也就是 Random Access Memory, RAM)讀取出來的。但是RAM上面的資料會因為斷電而消失，所以資料必須儲存在更永久的除存裝置上，如硬碟等。

我們將這種記憶體的階層關係稱為　&#x6D;_&#x65;mory hierarchy，_&#x8A73;細見下面表格



記憶體的階層關係表格&#x20;

<table><thead><tr><th width="122.33333333333331">存取速度</th><th width="112">記憶體</th><th>描述</th></tr></thead><tbody><tr><td>很快</td><td>Cache</td><td>快去記憶體是鑲嵌在CPU晶片中ㄉㄜ記憶體，非常的快，基本上只需要一個處理器時脈的時間就可以存取。但因為是直接鑲嵌在CPU的晶片上，所以限制了快取的大小。事實上，還有快取很有分幾個階層，叫做L1,L2,L3 每層的速度都不同</td></tr><tr><td></td><td>RAM</td><td><p>所有指令跟儲存的記憶體位置都是從RAM存取的，雖然RAM已經很快了，但比起CPU的速度，還是需要等一段時間，這段時間被叫做latency。RAM的資料除存在一個分開的，獨立安裝在主機板上的</p><p>某個位置，這意味著他有比快取更大的記憶體空間。</p></td></tr><tr><td>很慢</td><td>Disk</td><td>我們在硬碟上儲存我們ㄒ需要的檔案，我們有很習慣打開檔案需要花一段時間讀取。具有選轉磁碟盤和移動磁碟盤的讀取頭這種物理的機制讓硬碟變成最慢的儲存形式。但他是目前最大的儲存空間</td></tr></tbody></table>

了解記憶體的架構，最重要的一點就是儲存大小跟速度之間的權衡問題。越快的記憶體容量越少。但，如果你可以找到一個顛覆此概念的儲存媒介，你就會成為超級有錢人。

快取這麼小，卻有用的原因是因為電腦的程式通常都具有兩種性質

1. 空間局部性(_Spatial_ locality)，一個單位(blocks)的數據很有可能會被一起存取
2. 時間局部性(_Temporal_ locality)，最近使用過的資料高機率會再被使用一次

這代表，寫程式的時候盡量將相關的訊息儲存在同一塊記憶體空間範圍，或是將常使用的程式放在相近的地方都可以增進執行的效率

## 快取細節

快取是CPU架構中重要的元素。想要寫出有效率的程式，程式工程師需要了解處理器架構的快取機制。

快取是用來複製主記憶體的資料。但快取的記憶體容量比主記憶體小很多，因為快取是處理器晶片裡面硬體結構，他跟暫存器跟處理器的邏輯放在一起。在處理器製造來說，記憶體就代表成本，所以不會實做很大的快取處理器，這個是物理上，也是經濟上的限制。處理器製造商想盡辦法在塞更多的電晶體在同一個大小的晶片上，晶片的大小變得很大，但即使這樣，最大的快取記憶體也是 MB 等級，不像RAM可以作到 GB，硬碟可以做到 TB。（按編：目前i9-13900K 快取36MB）

快取是為了要複製整個主記憶體的資料，他跟主記憶體有一些映射關係，結構上有所謂的快取行 (line)  ，資料大小通常是 32 或是 64bytes ，快取一次就是存取一個快取行的資料大小。

* 這邊可以參考「每位程式開發者都該有的記憶體知識」\
  &#x20;[https://github.com/sysprog21/cpumemory-zhtw](https://github.com/sysprog21/cpumemory-zhtw)

快取有自己的階層架構，通常稱為L1、L2和L3。L1快取最小最快，L3最大最慢。

如果L1快取分別儲存指令跟資料，那就是被稱為「哈佛架構」，因為作者叫做Harvard Mark。把快取拆成兩塊可以增進速度，因為pipeline前期都是指令，後期比較多是資料快取。除了可以減少資料與指令競爭共享資源以外，由於指令是唯讀資料，且有固定的指令大小，所以不需要處理一些讀寫相關的同步問題，或是一次讀很多快取的問題。



![](https://www.bottomupcs.com/chapter02/figures/sets.svg)Figure 2.2.1 Cache Associativity

During normal operation the processor is constantly asking the cache to check if a particular address is stored in the cache, so the cache needs some way to very quickly find if it has a valid line present or not. If a given address can be cached anywhere within the cache, every cache line needs to be searched every time a reference is made to determine a hit or a miss. To keep searching fast this is done in parallel in the cache hardware, but searching every entry is generally far too expensive to implement for a reasonable sized cache. Thus the cache can be made simpler by enforcing limits on where a particular address must live. This is a trade-off; the cache is obviously much, much smaller than the system memory, so some addresses must _alias_ others. If two addresses which alias each other are being constantly updated they are said to _fight_ over the cache line. Thus we can categorise caches into three general types, illustrated in [Figure 2.2.1, Cache Associativity](https://www.bottomupcs.com/ch03s02.html#cache_associativity).

* _Direct mapped_ caches will allow a cache line to exist only in a singe entry in the cache. This is the simplest to implement in hardware, but as illustrated in [Figure 2.2.1, Cache Associativity](https://www.bottomupcs.com/ch03s02.html#cache_associativity) there is no potential to avoid aliasing because the two shaded addresses must share the same cache line.
* _Fully Associative_ caches will allow a cache line to exist in any entry of the cache. This avoids the problem with aliasing, since any entry is available for use. But it is very expensive to implement in hardware because every possible location must be looked up simultaneously to determine if a value is in the cache.
* _Set Associative_ caches are a hybrid of direct and fully associative caches, and allow a particular cache value to exist in some subset of the lines within the cache. The cache is divided into even compartments called _ways_, and a particular address could be located in any way. Thus an _n_-way set associative cache will allow a cache line to exist in any entry of a set sized total blocks mod n — [Figure 2.2.1, Cache Associativity](https://www.bottomupcs.com/ch03s02.html#cache_associativity) shows a sample 8-element, 4-way set associative cache; in this case the two addresses have four possible locations, meaning only half the cache must be searched upon lookup. The more ways, the more possible locations and the less aliasing, leading to overall better performance.

Once the cache is full the processor needs to get rid of a line to make room for a new line. There are many algorithms by which the processor can choose which line to evict; for example _least recently used_ (LRU) is an algorithm where the oldest unused line is discarded to make room for the new line.

When data is only read from the cache there is no need to ensure consistency with main memory. However, when the processor starts writing to cache lines it needs to make some decisions about how to update the underlying main memory. A _write-through_ cache will write the changes directly into the main system memory as the processor updates the cache. This is slower since the process of writing to the main memory is, as we have seen, slower. Alternatively a _write-back_ cache delays writing the changes to RAM until absolutely necessary. The obvious advantage is that less main memory access is required when cache entries are written. Cache lines that have been written but not committed to memory are referred to as _dirty_. The disadvantage is that when a cache entry is evicted, it may require two memory accesses (one to write dirty data main memory, and another to load the new data).

If an entry exists in both a higher-level and lower-level cache at the same time, we say the higher-level cache is _inclusive_. Alternatively, if the higher-level cache having a line removes the possibility of a lower level cache having that line, we say it is _exclusive_. This choice is discussed further in [Section 4.1.1.1, Cache exclusivity in SMP systems](https://www.bottomupcs.com/ch03s04.html#cache_exclusivity_in_smp).

**2.2.1 Cache Addressing**

So far we have not discussed how a cache decides if a given address resides in the cache or not. Clearly, caches must keep a directory of what data currently resides in the cache lines. The cache directory and data may co-located on the processor, but may also be separate — such as in the case of the POWER5 processor which has an on-core L3 directory, but actually accessing the data requires traversing the L3 bus to access off-core memory. An arrangement like this can facilitate quicker hit/miss processing without the other costs of keeping the entire cache on-core.

![](https://www.bottomupcs.com/chapter02/figures/tags.svg)Figure 2.2.1.1 Cache tags

To quickly decide if an address lies within the cache it is separated into three parts; the _tag_ and the _index_ and the _offset_.

The offset bits depend on the line size of the cache. For example, a 32-byte line size would use the last 5-bits (i.e. 25) of the address as the offset into the line.

The _index_ is the particular cache line that an entry may reside in. As an example, let us consider a cache with 256 entries. If this is a direct-mapped cache, we know the data may reside in only one possible line, so the next 8-bits (28) after the offset describe the line to check - between 0 and 255.

Now, consider the same 256 element cache, but divided into two ways. This means there are two groups of 128 lines, and the given address may reside in either of these groups. Consequently only 7-bits are required as an index to offset into the 128-entry ways. For a given cache size, as we increase the number of ways, we decrease the number of bits required as an index since each way gets smaller.

The cache directory still needs to check if the particular address stored in the cache is the one it is interested in. Thus the remaining bits of the address are the _tag_ bits which the cache directory checks against the incoming address tag bits to determine if there is a cache hit or not. This relationship is illustrated in [Figure 2.2.1.1, Cache tags](https://www.bottomupcs.com/ch03s02.html#cache_tags).

When there are multiple ways, this check must happen in parallel within each way, which then passes its result into a multiplexor which outputs a final _hit_ or _miss_ result. As describe above, the more associative a cache is, the less bits are required for index and the more as tag bits — to the extreme of a fully-associative cache where no bits are used as index bits. The parallel matching of tags bits is the expensive component of cache design and generally the limiting factor on how many lines (i.e, how big) a cache may grow.
