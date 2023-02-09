# 虛擬記憶體

## 你所不知道的虛擬記憶體(Virtual memory)

虛擬記憶體通常被天真的認為是用硬碟的空間當作你額外的記憶體，這個記憶體是額外的，速度比較慢的電腦記憶體。也就是說，當我們電腦使用完記憶體之後，就會使用到硬碟上的虛擬記憶體。

在現代的電腦上，這個空間被稱為 swap space，因為這個機制是將沒有用到的記憶體內容儲存到硬碟之中，釋放出主記憶體(請記得，程式只能在主記憶體中執行)。

確實，把記憶體的內容交換到硬碟上面是一個重要的功能，但是你等等就會知道這個並不是虛擬記憶體的主要目的，這個特色只是虛擬記憶體意想不到的副作用。

## 虛擬記憶體是什麼？

虛擬記憶體就是使用地址空間的概念(address space)。

地址空間就是指處理器所有可以存取到的記憶體的可能範圍。這個地址空間受限於處理器的暫存器的大小限制，因為處理器的載入指令(load)需要輸入地址。也就是說 32bit 的處理器可以使用 `0x00000000`到`0xFFFFFFF`範圍內的地址，也就是大概4GB(2^32)的大小，所以 32bit 處理器可以載入儲存最大 4GB 的記憶體。

### 64 bit 處理器

現代新的處理器大概都是 64bit 的處理器，顧名思義，其暫存器有 64bit 。這個空間很大，大概有16EB。

> 1EB=1024PB
>
> 1PB=1024TB
>
> 1TB = 1024GB

64 bit 處理器是權衡之下的結果，因為64bit 處理器需要 8-byte pointers，程式的大小跟資料的大小的會因此增加，影響指令跟資料的快取表現。但是 64bit 處理器會有更多的暫存器，紓解了編譯器在暫存器不夠用的時候，會去存一些臨時的變數因應的情況。

### **Canonical Addresses**

雖然 64bit 處理器有 64bit 寬的暫存器，但是電腦不會實作這麼大的地址空間，因為現實中也沒有這麼大的實體記憶體可以給處理器存取。

所以大部分的處理器架構都會定義一個未實做(_unimplemented_)的地址空間，在x86-64 跟 Itanium 這兩個架構之下，



Thus most architectures define an _unimplemented_ region of the address space which the processor will consider invalid for use. x86-64 and Itanium both define the most-significant valid bit of an address, which must then be sign-extended (see[Section 2.3.1.3.1, Sign-extension](https://www.bottomupcs.com/ch02s02.html#sign\_extension)) to create a valid address. The result of this is that the total address space is effectively divided into two parts, an upper and a lower portion, with the addresses in-between considered invalid. This is illustrated in [Figure 2.1.1.1, Illustration of canonical addresses](https://www.bottomupcs.com/ch06s02.html#canonical\_address). Valid addresses are termed _canonical addresses_ (invalid addresses being _non_-canonical).



