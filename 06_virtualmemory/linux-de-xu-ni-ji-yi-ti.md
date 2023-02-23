---
description: Linux 的虛擬記憶體架構
---

# Linux 的虛擬記憶體

雖然說虛擬記憶體的概念都差不多，但是實作上還是強烈的跟作業系統跟硬體相關。

## 地址空間的佈局

Linux 會將可用的地址空間切成兩個部分，一個部分是核心使用、一個部分是私有使用者空間。這個就代表，核心的地址空間對於每個 process 來說都是映射到同一個物理記憶體的位置，但是使用者空間的地址是每個 process 讀自擁有的。在 Linux 中，核心的地址空間在非常上面的位置，一般 32bit x86 系統，大概就會分在 3GB 那那個位置，因為 32 bit 最多就是 4GB 記憶體，核心保留 1 GB 作為核心共享的記憶體空間。

{% hint style="info" %}
這個是一個過度簡化的說明，因為會有 process 需要超過 4GB以上的記憶體空間，這個你就自己在調查一下要怎麼使用特殊的 extensions 支援 > 4GB 的記憶體空間。
{% endhint %}



<figure><img src="../.gitbook/assets/linux-layout.svg" alt=""><figcaption><p>這張圖在說明(1) 所有 process 都共用一個核心的記憶體空間，(2) 核心的記憶體空間對應到的位置都是同一個物理上的記憶體位置，(3) 每個 process 都有自己的記憶體空間，然後對應到不同的物理記憶體位置。 </p></figcaption></figure>

## 三層 Page Table

作業系統有許多不同的方法來組織 page table，但 linux 選擇使用分層系統( _hierarchical_ system )

由於 page table 分了三層的原因，Linux 這個方案也常常被稱為 「_three level page table_」。這是一個很強健的方案，雖然說也是有一些批評跟缺點存在。每一種處理器的實現虛擬記憶體的細節都不相同，但是 linux 要是 portable 並且相對通用的選擇

三層 page table 的概念不是很難。我們已經知道虛擬地址用 page number 跟 offset  來表達實體記憶體的地址。在三層的 page table 中，這些虛擬地址被拆分成更多的 level。

每一個 level 本身就是一個 page table。 level 1 會直接對應到leve2 這張 page table 的物理位置， level 3 會對應到另外一個物理位置，

<figure><img src="../.gitbook/assets/threelevel-2.svg" alt=""><figcaption></figcaption></figure>

所以一個簡單的虛擬記憶體對應到實體的位置，就會變成先找 level 1 的某一個物理記憶體的位置，然後再找下一層的記憶體位置，重複這個步驟這樣。

直接看這個架構感覺就是沒有必要的複雜。這個架構的考量是記憶體大小。想像一下，如果只有一個 page table 要對應整個記憶體空間，這張 page table 要很大一張。就是之前有提到的， page table 就是給你查表記憶體位置的，基本上是連續的。如果你可以用的記憶體空間很大，那這張表也會很大。

所以在三層系統中， level 1 只是一個實體記憶體的 frame ，對應到下一個 level 的 page table ，這樣。所以每一張 page table 的大小就可以縮小了。

缺點也是很明顯，本來查表一次就可以找到實體記憶體的位置，現在要查很多次，這個可能會很耗費時間。 Linux 也知道這件事情，很多處理器可能不適合這個架構，所以有些架構會簡化 page table 變成兩層(例如 x86 就是用兩層的 page table )。

\
