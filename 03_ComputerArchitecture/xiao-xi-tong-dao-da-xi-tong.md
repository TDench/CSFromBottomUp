# 小系統到大系統

根據莫爾定律，電腦的計算能力用一個很誇張的速度在增長，而且目前看起來並不會減慢。一般來說，高階系統，伺服器等電腦都不是只有一個CPU的系統。對於如何達成各個CPU的協作有很多種方式。

## 對稱多處理(Symmetric Multi-Processing)

對稱多處理系統(Symmetric Multi-Processing, SMP) 是一個系統裡面包含多個CPU的一個常見的配置。`symmetric` 這個字代表所有的CPU都是相同的(同樣的架構，同樣的時鐘速度)。在SMP系統中，多個處理器會共享其他人的資源(resources)，例如記憶體、硬碟空間之類的。

## 快取一致性(Cache Coherency)

大多數的情況下，系統中的 CPU獨立運作，每個人都有自己的一組暫存器、program counter等。儘管他們分別獨立的運行，有一個重要的部分需要嚴格的同步(synchronisation)。

這個東西就是 CPU 快取，怕你忘記提醒一下，快取就是一個小的記憶體位置，可以快速的存取主記憶裡面的資訊。如果一個CPU對主記憶體中的資料進行修改，然後其他的CPU還在讀自己手上本來有的快取資訊，這樣系統就不處於同步的狀態。順帶一提，這種問題只有在寫入的時候才會發生，因為如果只是讀取的話，每個CPU讀取的資料都會相同。

為了要協調所有處理器都處於快取一致性的狀態，SMP系統會使用窺探(snooping)，窺探就是處理器會去監聽所有連接到匯流排的處理器，有沒有發生快取改變的事件，如果有的話就會作相對應的快取更新。

其中一個快取一致性的協定(Protocol)是MOESI協定，這幾個英文字分別代表修改（Modified），所有權（Owner），獨佔（Exclusive），共享（Shared），無效（Invalid）。這裡每一個英文字都是代表快取行(cache line) 上處理器的狀態。當然，還有協定，但是他們都有相似的概念，下面我們會仔細觀察 MOESI的協定，讓你有基本的認識。

當處理器需要從主記憶體讀取快取的時候，處理器會先去窺探(snooping)系統中所有的處理器，看看其他處理器有沒有對於這塊記憶體的訊息(例如，這段記憶體已經被其他處理器快取了)。如果目前並沒有任何處理器有這段記憶體的資料，那使處理器就會快取這段記憶體，然後把這段標為獨佔(E)，當寫入快取這件事情發生的時候，就把狀態標記程修改(M)，下面開始就有趣了，有一些快取會馬上寫回主記憶體(這個就被稱為 write-through ，因為寫入的動作直接寫到主記憶體)，有一些快取會只先寫到快取上，除非快取被寫滿了或是逐出(evicted)才會寫入主記憶體。

另外一種情況就是當處理器窺探其他處理器的時候，發現有人標記這段記憶體為修改(M)，這個時候就把資料複製一分到自己的快取，並且標記程共享(Shared)，他會傳訊息給其他處理器，告訴大家剛剛那個M改成所有者(O)。想像中，現在有第三個處理器想要使用這塊記憶體的資料的時候，他會看到一個S跟一個O。這個時候他會去O那邊複製一份資料到自己的快取。如果現在其他處理器都只是要讀取，快取行(cache line)就會維持共享(S)的狀態。但是，當其中一個處理器想要更新段資料的時候，他需要傳送無效(invalidate)訊息給其他的處理器。有快取這段資料的處理器就要把自己的快取標記成無效(I)，因為現在這個快取上面的資料已經不是正確的資料了。當這段無效訊息除給其他處理器之後，自己的快取

The other case is where the processor snoops and finds that the value is in another processors cache. If this value has already been marked as _modified_, it will copy the data into its own cache and mark it as _shared_. It will send a message for the other processor (that we got the data from) to mark its cache line as _owner_. Now imagine that a third processor in the system wants to use that memory too. It will snoop and find both a _shared_ and a _owner_ copy; it will thus take its value from the _owner_ value. While all the other processors are only reading the value, the cache line stays _shared_ in the system. However, when one processor needs to update the value it sends an _invalidate_ message through the system. Any processors with that cache line must then mark it as invalid, because it not longer reflects the "true" value. When the processor sends the invalidate message, it marks the cache line as _modified_ in its cache and all others will mark as _invalid_ (note that if the cache line is _exclusive_ the processor knows that no other processor is depending on it so can avoid sending an invalidate message).

另一种情况是处理器听(snooping)并发现该值在另一个处理器缓存中。如果此值已标记为已修改，则会将数据复制到其自己的缓存中并将其标记为已共享。它将为另一个处理器（我们从中获取数据）发送一条消息，将其缓存行标记为所有者。现在假设系统中的第三个处理器也想使用该内存。它将窥探并找到共享和所有者副本;因此，它将从所有者价值中获取其价值。虽然所有其他处理器仅读取该值，但缓存行仍保持在系统中共享。但是，当一个处理器需要更新该值时，它会通过系统发送无效消息。具有该缓存行的任何处理器必须将其标记为无效，因为它不再反映“真实”值。当处理器发送 invalidate 消息时，它会在其缓存中将缓存行标记为已修改，而其他所有缓存行都将标记为无效（请注意，如果缓存行是独占的，则处理器知道没有其他处理器依赖它，因此可以避免发送消息无效）。

从这一点开始，整个过程就开始了。因此，无论哪个处理器具有修改后的值，都有责任在从缓存中逐出时将真值写回 RAM。通过思考协议，您可以看到这确保了处理器之间缓存线的一致性。

随着处理器数量的增加，该系统存在一些问题。只有少数处理器，检查另一个处理器是否具有高速缓存行（读取监听）或使每个其他处理器中的数据无效（使监听无效）的开销是可管理的;但随着处理器数量的增加，总线流量也会增加。这就是 SMP 系统通常只能扩展到大约 8 个处理器的原因。

将处理器全部放在同一总线上也开始出现物理问题。导线的物理特性仅允许它们彼此以一定距离布置并且仅具有一定长度。对于运行在许多千兆赫兹的处理器，光速开始成为消息在系统中移动需要多长时间的真正考虑因素。

请注意，系统软件通常不参与此过程，尽管程序员应该了解硬件在其下面做了什么，以响应他们设计的程序以最大限度地提高性能。

From this point the process starts all over. Thus whichever processor has the _modified_ value has the responsibility of writing the true value back to RAM when it is evicted from the cache. By thinking through the protocol you can see that this ensures consistency of cache lines between processors.

There are several issues with this system as the number of processors starts to increase. With only a few processors, the overhead of checking if another processor has the cache line (a read snoop) or invalidating the data in every other processor (invalidate snoop) is manageable; but as the number of processors increase so does the bus traffic. This is why SMP systems usually only scale up to around 8 processors.

Having the processors all on the same bus starts to present physical problems as well. Physical properties of wires only allow them to be laid out at certain distances from each other and to only have certain lengths. With processors that run at many gigahertz the speed of light starts to become a real consideration in how long it takes messages to move around a system.

Note that system software usually has no part in this process, although programmers should be aware of what the hardware is doing underneath in response to the programs they design to maximise performance.

``
