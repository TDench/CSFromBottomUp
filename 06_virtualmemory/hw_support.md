# 硬體支援

到目前為止，我們只提到硬體與作業系統一起實作出虛擬記憶體。我們現在終於可以一窺究竟虛擬記憶體是怎麼實作的。

虛擬記憶體大多是跟硬體架構高度相關，每個架構都有自己的微妙之處。但他們有一些共同的元素，後面會介紹給你聽聽。

## 物理/虛擬 記憶體模式

所有處理器都有兩種記憶體模式的概念，要麻是物理模式，要麻虛擬記憶體模式。在物理模式下，硬體會期望地址就是真實的記憶體位置；在虛擬模式下，硬體知道需要翻譯之後才會是真實的記憶體位置。

在許多處理器中，這兩種模式被簡單地稱為物理模式和虛擬模式，例如 intel Itanium 架構。而在最常見的 x86 處理器中，因為一些歷史的包袱，這兩個模式被稱為真實（ real） 跟保護（ _protected）_模式。第一個實現保護模式的是 i386 ，直到現在最新的 x86 仍然可以執行 real mode (雖然說沒有人這樣使用)。在 真實模式下，處理器會實施一種記憶體的分割，叫做 segmentation 。

{% hint style="info" %}
關於 x86 的 real mode 的 segmentation

[https://en.wikipedia.org/wiki/X86\_memory\_segmentation#Real\_mode](https://en.wikipedia.org/wiki/X86\_memory\_segmentation#Real\_mode)
{% endhint %}

### &#x20;**Segmentation 的問題？**

這個「**Segmentation**」就是一個有趣的歷史故事而已，因為現在已經有虛擬記憶體的概念了。segmentation 有一些缺點，他會使沒有什麼經驗的程式設計師感到非常困惑，所以才發明了虛體記憶體的這個機制。

在 segmentation 中，會有許多的暫存器儲存「一段記憶體 segment 的開頭地址」，如果你想知道地址的話，你就必須知道你要找的資料在哪一個 segment ，找到相對應的暫存器，然後加上 offset 才會是你想要的地址。segment 的大小你可以決定，但基本上取決於暫存器中可用的 bit 數量有關。在 x86 架構中，最大的大小是 64 KB(16bit)，如果你想要存取大於 64 KB 的地址的話，就會造成系統大崩潰。但是現代的記憶體都是 MB 等級、GB 等級的記憶體。現在已經不是小小的不方便而已



<figure><img src="../.gitbook/assets/segmentation.svg" alt=""><figcaption><p>這邊在講說，在 segment 的模式下，CPU的暫存器會存放每一個 segment 的開頭位置，每一個大小最大可能是 64 KB，要讀取記憶體地址的話，就是先找到那個 segment 的開頭，然後加上一個 offset。啊如果你超過每個 segment 的大小，也就是你想要讀這個 segment 然後 offset = 128KB 系統就會大爆炸。</p></figcaption></figure>



在上面這張圖中，有三個暫存器，都指向 segment ，受到可用 bit 數影響的最大 offset用斜線表示。如果程式想要存取超過此範圍的地址，必須要重新配置 segment 暫存器。這個就變成一個很大的煩惱。另一方面，虛擬記憶體允許程式可以指定任何的地址，然後作業系統跟硬體可以努力的翻譯虛擬地址到真實的記憶體位置。&#x20;

## 什麼是 TLB？

轉譯後備緩衝區（The _Translation Lookaside Buffer, TLB）_ 是處理器主要負責虛擬記憶體的元件。他是虛擬 page 對應到真實記憶體 frame 的表格的快取，作業系統跟硬體一起管理這個 TLB

### **Page Faults**

當硬體收到一個虛擬地址的時候，假設現在是透過 `load` 指令想要獲得一些資料，處理器會先看看 TLB 有沒有這段虛擬地址的對應關係，如果有的話，那他就可以直接依據這個資訊，跟 offset ，直接找到物理社的位置，然後直接完成 load 的動作。

但是，如果處理器在 TLB 中找不到對應關係，那處理器就會發起一個 「_page fault_」，這個類似一個作業系統必須處理的中斷（跟前面討論的一樣）然後作業系統就必須處理這件事情。

當作業系統收到 _page fault_ 的時候，他需要透過 page table 去找到正確的 page 跟 frame 的對應關係，然後把這個關係插入 TLB 裡面。

如果作業系統在 page table 中找不到對應關係，或者作業系統檢查到這個 process 無權訪問這張 page 的記憶體內容，那作業系統就必須停止(kill) 這個 process。如果你曾經看過「segmentation fault (segfault)」 ，這就是作業系統停止一個 process 越界存取記憶體的意思。

如果找到對應關係，但是 TLB 已經滿了，這個時候在插入這個對應關係之前，要先刪掉另外一組。通常我們會刪除等等用不太到的那組，因為如果的等就要用到我們還要花時間去查  page table 。所以我們會用一些演算法，例如最近最久未使用演算法（_Least Recently Used,_ LRU），把其中最古老的，最少用到的那組給踢出，取而代之的是最新的翻譯。

然後這個過程就會重複的出現，如果事情順利的話，那後來會在 TLB 找到虛擬記憶體跟實際記憶體的對應關係。

{% hint style="info" %}
最近最久未使用演算法，就是把最不常用的到刪掉，大部分的人翻譯成最近最少使用法，感覺比較像是英文直接翻譯，沒有翻到意義。

[https://zhuanlan.zhihu.com/p/31997704](https://zhuanlan.zhihu.com/p/31997704)
{% endhint %}

#### **怎麼找到 page table？**

當我們說，作業系統會從 page table 中找到虛擬記憶體跟實體記憶體的對應關係的時候，我們應該問問作業系統是如何找到 page table 的？ page table 在記憶體的哪裡？

page table 的地址被儲存在每個有關連的 process 的暫存器裡面。通常這個暫存器會被叫做「page-table base-register 」，利用這個暫存器跟第幾張 page 這兩個資訊，就可以找到正確的 page table&#x20;

#### **其他跟 page 相關的錯誤**

TLB 通常還有兩種常見的壞掉的方法，

TLB通常可以產生另外兩個重要故障，這些故障有助於控制訪問和髒頁面。 每個頁面通常包含一個以單個位為形式的屬性，該屬性標記是否已訪問或髒。

There are two other important faults that the TLB can generally generate which help to mange accessed and dirty pages. Each page generally contains an attribute in the form of a single bit which flags if the page has been accessed or is dirty.

訪問的頁面只是任何已被訪問的頁面。 當頁面翻譯最初載入到TLB時，頁面可以標記為已訪問（您為什麼要載入它？[ 2）](https://www.bottomupcs.com/ch06s08.html#the\_tlb\_s2\_para2\_footnote1-fnote)

作業系統可以定期瀏覽_所有_頁面並清除訪問位，以瞭解當前使用的頁面。 當系統記憶體滿，作業系統選擇要交換到磁碟的頁面時，顯然那些訪問位尚未重置的頁面是刪除的最佳候選頁面，因為它們的使用時間不長。

髒頁面是寫入資料的頁面，因此與磁碟上已經存在的任何資料不匹配。 例如，如果頁面從交換中載入，然後由程序寫入，則在將其移出交換之前，它需要更新其磁碟副本。 乾淨的頁面沒有變化，因此我們不需要將頁面複製回磁碟的開銷。

兩者都很相似，因為它們有助於作業系統管理頁面。 一般的概念是，一個頁面有兩個額外的位：髒位和訪問位。 當頁面放入TLB時，這些位被設定為指示CPU應該引發故障。

當程序嘗試引用記憶體時，硬體會執行通常的翻譯過程。 然而，它還會進行額外的檢查，看看是否_沒有_設定訪問的標誌。 如果是這樣，它會給作業系統帶來故障，作業系統應該設定位並允許程序繼續。 同樣，如果硬體檢測到它寫入沒有髒位集的頁面，操作系統將頁面標記為髒位會引發故障。

An accessed page is simply any page that has been accessed. When a page translation is initially loaded into the TLB the page can be marked as having been accessed (else why were you loading it in?[2](https://www.bottomupcs.com/ch06s08.html#the\_tlb\_s2\_para2\_footnote1-fnote))

The operating system can periodically go through _all_ the pages and clear the accessed bit to get an idea of what pages are currently in use. When system memory becomes full and it comes time for the operating system to choose pages to be swapped out to disk, obviously those pages whose accessed bit has not been reset are the best candidates for removal, because they have not been used the longest.

A dirty page is one that has data written to it, and so does not match any data already on disk. For example, if a page is loaded in from swap and then written to by a process, before it can be moved out of swap it needs to have its on disk copy updated. A page that is clean has had no changes, so we do not need the overhead of copying the page back to disk.

Both are similar in that they help the operating system to manage pages. The general concept is that a page has two extra bits; the dirty bit and the accessed bit. When the page is put into the TLB, these bits are set to indicate that the CPU should raise a fault .

When a process tries to reference memory, the hardware does the usual translation process. However, it also does an extra check to see if the accessed flag is _not_ set. If so, it raises a fault to the operating system, which should set the bit and allow the process to continue. Similarly if the hardware detects that it is writing to a page that does not have the dirty bit set, it will raise a fault for the operating system to mark the page as dirty.

#### 8.3 TLB Management

We can say that the TLB used by the hardware but managed by software. It is up to the operating system to load the TLB with correct entries and remove old entries.

**8.3.1 Flushing the TLB**

The process of removing entries from the TLB is called _flushing_. Updating the TLB is a crucial part of maintaining separate address spaces for processes; since each process can be using the same virtual address not updating the TLB would mean a process might end up overwriting another processes memory (conversely, in the case of _threads_ sharing the address-space is what you want, thus the TLB is _not_ flushed when switching between threads in the same process).&#x20;

On some processors, every time there is a context switch the entire TLB is flushed. This can be quite expensive, since this means the new process will have to go through the whole process of taking a page fault, finding the page in the page tables and inserting the translation.

Other processors implement an extra _address space ID_ (ASID) which is added to each TLB translation to make it unique. This means each address space (usually each process, but remember threads want to share the same address space) gets its own ID which is stored along with any translations in the TLB. Thus on a context switch the TLB does _not_ need to be flushed, since the next process will have a different address space ID and even if it asks for the same virtual address, the address space ID will differ and so the translation to physical page will be different. This scheme reduces flushing and increases overall system performance, but requires more TLB hardware to hold the ASID bits.

Generally, this is implemented by having an additional register as part of the process state that includes the ASID. When performing a virtual-to-physical translation, the TLB consults this register and will only match those entries that have the same ASID as the currently running process. Of course the width of this register determines the number of ASID's available and thus has performance implications. For an example of ASID's in a processor architecture see [Section 10.2.1, Address spaces](https://www.bottomupcs.com/ch06s10.html#itanium\_address\_spaces).

**8.3.2 Hardware v Software loaded TLB**

While the control of what ends up in the TLB is the domain of the operating system; it is not the whole story. The process described in [Section 8.2.1, Page Faults](https://www.bottomupcs.com/ch06s08.html#page\_faults) describes a page-fault being raised to the operating system, which traverses the page-table to find the virtual-to-physical translation and installs it in the TLB. This would be termed a _software-loaded TLB_ — but there is another alternative; the _hardware-loaded TLB_.

In a hardware loaded TLB, the processor architecture defines a particular layout of page-table information ([Section 5, Pages + Frames = Page Tables](https://www.bottomupcs.com/ch06s05.html) which must be followed for virtual address translation to proceed. In response to access to a virtual-address that is not present in the TLB, the processor will automatically walk the page-tables to load the correct translation entry. Only if the translation entry does not exist will the processor raise an exception to be handled by the operating system.

Implementing the page-table traversal in specialised hardware gives speed advantages when finding translations, but removes flexibility from operating-systems implementors who might like to implement alternative schemes for page-tables.

All architectures can be broadly categorised into these two methodologies. Later, we will examine some common architectures and their virtual-memory support.

\






\


