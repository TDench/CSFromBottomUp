---
description: 這邊會說明一個行程(process) 裡面記錄了什麼資訊
---

# 行程裡面有什麼

<figure><img src="../.gitbook/assets/image (5).png" alt=""><figcaption><p>這張圖描述 process 裡面什麼東西 (1) Process ID (2) 記憶體 (3) 檔案 </p></figcaption></figure>



## Process ID

每一個行程都有自己獨特的ID，我們稱為Process ID (PID)

## 記憶體 (Memory)

我們接下來會說明每一個行程是怎麼樣獲取這些記憶體的。 了解作業系統如何分配記憶體給每一個行程的話， 基本上我們就對作業系統有非常基礎的了解，但現在只要知道每一個行程都有自己的記憶體就可以了

記憶體裡面會儲存程式碼、變數、 這一些其他的東西

有部分的記憶體是可以在行程之間共享的， 我們稱他共享記憶體(Shared memory) 你經常會看到它被稱為「系統五共享記憶體」（System Five Shared Memory,或 SysV SHM），這是因為它最初是由舊版操作系統實現的。

另外一個概念就是行程會使用 「mmap」  記憶體映射的概念。 這個是什麼意思呢？也就是我們開個檔案的時候我們不會用`read()` 或 `write()` 我們就直接把檔案看作是一種記憶體的感覺， 這些被 mmaped 的地方會有讀、寫、執行的權利， 作業系統應該要記住這一些權利的分配， 檢查每一個行程是否有寫資料到唯讀的區域， 如果有的話要返回錯誤的訊息

{% hint style="info" %}
[https://hackmd.io/@sysprog/linux-shared-memory](https://hackmd.io/@sysprog/linux-shared-memory)

共享記憶的概念，是從古老的系統傳承下來的
{% endhint %}

### 程式碼與資料

一個行程可以被分為兩個部分， 程式碼跟資料這兩個部分。 因為這兩個部分應該是有不同的權限， 換句話說程式碼應該只能讀跟執行不能被寫入， 資料的部分應該是可以讀取、 寫入但不應該被拿來執行

### 堆疊(Stack)

另外一個重要的部分就是被稱為堆疊的記憶體， 他算是資料

Stack是一個數據結構， 他運行的方式就跟堆盤子是一樣的：放(push) 盤子的時候一定會放在最上面， 拿(pop)時候也是從最上面的盤子開始拿。

函式呼叫就是應用stack這個數據結構， 當函數被呼叫的時候他就會拿到一個 stack frame 這是一個記憶體空間， 裡面最重要的是要記錄做完函式之後要返回的地址、 函式的輸入(arguments)、變數(variables)如下圖所示

<figure><img src="../.gitbook/assets/image (6).png" alt=""><figcaption><p>這個是一個stack 一般來說他會從比較高的地址開始逐漸往低的地方堆疊 stack裡面會放返回的地址， 函數的輸入， 本地變數</p></figcaption></figure>

這樣子的數據結構可以讓我們看到一般函式的功能

* 每一個函式都擁有自己的一份輸入參數(arguments)
* 因為函式都有自己的 stack 所以其他函式沒有辦法看到別的函式內部定義的參數
* 全域變數是大家都可以看到的變數，顯然是不會放在 stack 裡面
* 遞迴也是這樣實現的，因為函式可以自己呼叫自己









