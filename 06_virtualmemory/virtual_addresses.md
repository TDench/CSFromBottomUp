# 虛體地址（Virtual Addresses）



當我們的程式存取記憶體的時候，他不知道，也不會在乎這個地址的實體位置儲存在哪裡。因為程式知道真實的記憶體位置是由作業系統跟硬體一起去找到的，並且提供給程式他想要的資料。所以我們將程式使用的記憶體地址稱為「_虛擬地址(virtual address)_」，那這個虛擬地址什麼樣子呢？他有兩個部分，一個叫做 page 一個叫做 offset 。

## Page

因為整個地址空間都被切割成某個特定大小的 page ，所以每一個可能的地址都會是在某一張 page 裡面。所以虛擬地址中的 page 代表的就是 page table 裡面的索引，也就是第幾張 page 的意思。由於 page 是作業系統裡面最小的記憶體分配的單位，所以如果每一張 page 很小，那作業系統就必須管理一個數量很大的 page tables 。但如果每一張 page 很大，那可能會浪費記憶體空間。這個就交給大家去權衡一下。

## Offset

虛擬地址的最後幾個 bit 就被稱為偏移量(offset)。就是指你想要的地址到這張 page 開頭差了幾個 byte 。你需要足夠的 bit 數量才可以抵達任何一個 byte 。也就是説，對於一個 4KB 的 page ，你需要 12 個 bit 的 offset 才足夠描述 4KB( 2^12) 的地址。請記得，作業系統與硬體處理的最小的單位就是 page，這 4096 byte 都在同一個 page 裡頭，被當成一個物件來處理。

虛擬地址的最後一點稱為_偏移量_，即您想要的位元組地址與頁面開頭之間的位置差異。 您需要偏移量足夠的位數才能到達頁面中的任何位元組。 對於4K頁面，您需要（4K ==（4 \* 1024）= 4096 == 212 =，=）12位偏移量。 請記住，作業系統或硬體處理的最小記憶體量是一個頁面，因此這些4096位元組中的每一個都位於一個頁面中，並作為“一個”處理。

## 虛擬地址的對應關係

虛體地址對應關係，英文叫做 Virtual address translation，就是在指實際上的記憶體位置如何對應到虛擬記憶體位置的轉換。

一般來說，將虛擬記憶體轉換成實體記憶體位置的時候，我們只需要知道我們想要第幾張 page 就可以了。也就是說，這個流程就是，我們輸入我們想要第幾張 page ， 然後查看 page table 就可以知道這張 page 對應到實體記憶體的哪一個地址。然後再加上 offset ，那就會是我們想要存取的實體記憶體的位置。

由於  page table  是由作業系統負責控制。所以你想要的虛擬地址並不存在在這張 page table 的話，作業系統就會知道這個 process 嘗試想要存取沒有分配給他的記憶體位置，這個時候就會禁止他存取這段記憶體的內容。

<figure><img src="../.gitbook/assets/virtaddress.svg" alt=""><figcaption><p>我們的虛擬記憶體有兩個部分，一個是 page 一個是 offset 。 page 是我想要第幾張 page 的意思，假設是第13 張， page = 13, offset = 0x2F ，這個時候去查 page table 上看到 page 13 對應到實體記憶體 0x1000 0000 的位置。那我想要存取的位置就是在 0x1000 002F 的這個位置。</p></figcaption></figure>

我們如果依照之前線性的 page table 的邏輯來看。假設我們使用 page size = 4 KiB , 32 bit 的地址空間，我們需要大概一百萬個表格(1048576)。因此，假設我們的虛擬地址在 0x80001234 ，前面 0x80001 就是 page number ，去 page table 裡面找第 524289 (0x80001) ，發現這張 page 對應到實體記憶體的 0x1000000 的位置。那實際記憶體位置就是 0x10001234 這個位置。

那你可能會發現到線性 page table 的問題：由於每個 page 不管有沒有被使用到，我們都必須記錄他們的對應關係。這種對應表在 64bit 的地址空間下是完全不切實際的。假設每一個 page 需要 8 byte 來記錄他的位置，那你會總共有 2^52 次方個項目會在 page table 裡面。這樣需要 8\*2^52 = 32 PiB 的空間來儲存 page table ，那不就光存虛擬記憶體跟實體記憶體的對應關係就沒辦法做其他事情了？那我們後續會討論如何解決這個問題。



## 虛擬地址造成了什麼影響

剛剛那些虛擬記憶體啊，page 啊，page-table 都是現代作業系統的重要的一個基石。也是我們最常使用到的一個作業系統的特徵。他讓作業系統有自己獨立的地址空間、提供記憶體保護機制、swap 機制、共享記憶體的機制、硬碟快取的機制都跟虛體地址有關，下面會跟你說明。

### 獨立的地址空間(Individual address spaces)

由於每個 process 都有自己的 page table 。所以每一個 process 都可以假裝他可以存取到整個地址空間的任何一個位置。兩個 process 如果使用的相同的地址就不重要了。因為每個 process 的 page table 都會對應到不同的實體記憶體的位置。現代的作業系統都會為每個 process 提供自己的地址空間。

但隨著時間的推移，物理記憶體會變得支離破碎(_fragmented_)，也就是物理記憶體會存在一些「洞」，可以被分配的記憶體會變得很零碎。這個對程式設計師來說很困擾。假設，我想要配置 8 KiB 的記憶體空間，也就是兩個 4KiB 的空間，而且還希望他在物理上是連續的（不然我的 array 很難設計）。但使用虛擬記憶體的話這個就不是個問題，page 可以是連續的但是 frame 不連續也跟我沒關係。程式設計師就把這個問題留給作業系統就可以了。

### 記憶體保護

我們之前有提到 i386 處理器的虛擬模式就是保護模式。這個名字就源自於虛擬記憶體保護執行中的 process 的關係。

在沒有虛擬記憶體的系統之中，所有的 process 都可以存取所有的記憶體位置，這就代表說沒有人可阻止一個 process 覆寫掉別人的資料，造成當機，或是更糟糕的是回傳一個錯誤的數值。

為提供這個保護就是因為作業系統被當作是 process 跟記憶體之間的抽象層。如果 process 想要存取的虛擬記憶體沒有在 page table 之中，作業系統就會知道這個 process 是發生了問題，然後通知這個 process 他已經存取越界了(作業系統就話發中斷說 page fault)。

另外，由於每個 page 都有自己的屬性，有些 page 只可以讀、只可以寫、或者是一些其他有趣的屬性。當 process 存取這些 page 的時候，作業系統可以檢查它是否具有足夠的權限，如果沒有，就停止執行動作（例如在唯讀的 page 上寫資料，就拒絕這項操作。）

只用虛擬記憶體的系統本質上比較穩定，因為理想上來說，一個 process 當機不會影響到其他 process。但只是理想上，作業系統也是人寫的，總會有一些 bug 導致整台電腦死當。

### 記憶體 Swap



We can also now see how the swap memory is implemented. If instead of pointing to an area of system memory the page pointer can be changed to point to a location on a disk.

When this page is referenced, the operating system needs to move it from the disk back into system memory (remember, program code can only execute from system memory). If system memory is full, then _another_ page needs to be kicked out of system memory and put into the swap disk before the required page can be put in memory. If another process wants that page that was just kicked out back again, the process repeats.

This can be a major issue for swap memory. Loading from the hard disk is very slow (compared to operations done in memory) and most people will be familiar with sitting in front of the computer whilst the hard disk churns and churns whilst the system remains unresponsive.

**7.3.1 mmap**

A different but related process is the memory map, or `mmap` (from the system call name). If instead of the page table pointing to physical memory or swap the page table points to a file, on disk, we say the file is `mmap`ed.

Normally, you need to `open` a file on disk to obtain a file descriptor, and then `read` and `write` it in a sequential form. When a file is mmaped it can be accessed just like system RAM.

#### 7.4 Sharing memory

Usually, each process gets its own page table, so any address it uses is mapped to a unique frame in physical memory. But what if the operating system points two page table-entries to the same frame? This means that this frame will be shared; and any changes that one process makes will be visible to the other.

You can see now how threads are implemented. In [Section 4.3.1, `clone` ](https://www.bottomupcs.com/ch05s04.html#linux\_clone)we said that the Linux `clone()` function could share as much or as little of a new process with the old process as it required. If a process calls `clone()` to create a new process, but requests that the two processes share the same page table, then you effectively have a _thread_ as both processes see the same underlying physical memory.

You can also see now how copy on write is done. If you set the permissions of a page to be read-only, when a process tries to write to the page the operating system will be notified. If it knows that this page is a copy-on-write page, then it needs to make a new copy of the page in system memory and point the page in the page table to this new page. This can then have its attributes updated to have write permissions and the process has its own unique copy of the page.

#### 7.5 Disk Cache

In a modern system, it is often the case that rather than having too little memory and having to swap memory out, there is more memory available than the system is currently using.

The memory hierarchy tells us that disk access is much slower than memory access, so it makes sense to move as much data from disk into system memory if possible.

Linux, and many other systems, will copy data from files on disk into memory when they are used. Even if a program only initially requests a small part of the file, it is highly likely that as it continues processing it will want to access the rest of file. When the operating system has to read or write to a file, it first checks if the file is in its memory cache.

These pages should be the first to be removed as memory pressure in the system increases.

**7.5.1 Page Cache**

A term you might hear when discussing the kernel is the _page cache_.

The _page cache_ refers to a list of pages the kernel keeps that refer to files on disk. From above, swap page, mmaped pages and disk cache pages all fall into this category. The kernel keeps this list because it needs to be able to look them up quickly in response to read and write requests XXX: this bit doesn't file?

\
