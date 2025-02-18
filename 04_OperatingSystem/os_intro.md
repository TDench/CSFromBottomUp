# 作業系統簡介

## 作業系統的角色

作業系統支撐著現代電腦的運轉。

## 硬體的抽象化(Abstraction)

作業系統(Operation system, OS)最基本的功能就是將硬體功能抽象化，讓程式設計師跟使用者可以使用。作業系統提供使用界面(interfaces)讓人可以簡單的使用底層或是硬體的功能。

在一個沒有作業系統的電腦，每一個程式設計師都必須知道硬體的運作細節才可以開始執行程式。作業系統，更糟糕的是，如果硬體有些許的不同的話，軟體就必須要重寫。

## 多工（Multitasking）

我們希望現代的電腦可以同時執行很多事情，我們需要一個作業系統能用某種方式決定不同程式之間的先後順序，如何分配資源給每個程式之類的功能。作業系統的工作就是讓這些切換險的天衣無縫，不能讓使用者覺得卡頓。

作業系統業要府則管理資源(resource management)。系統中很多任務會競爭資源，所謂的資源包含處理器的計算時間，記憶體，硬碟，使用者的輸入。作業系統的工 做就是去仲裁(arbitrate)這些資源的分配，讓他們可以用時間順序取得資源。你一定也曾經體驗到作業系統分配失敗的時候，會顯示一個藍色的螢幕，俗稱藍屏(blue screen of death)。

## 標準化的介面

程式設計師會希望可以寫一個程式，而且這個程式可以在很多平台上面運行。有了作業系統提供的標稕化的界面，程式設計師就可以達成這個夢想。

例如，假如我們要在系統打開一個檔案的時候，系統A要呼叫`open()`，系統B要呼叫`open_file()` ，系統C要呼叫 `openf()` 這樣要寫程式就要記得要呼叫哪個函式，而且換個系統這個軟體也不能直接使用。

但是我們有Portable Operating System Interface (POSIX)，UNIX 系列的作業系統通用的界面。當然，微軟的Windows也有類似的標準化界面

## 安全性（Security）

很多使用者一起使用的電腦，安全性就很重要。作業系統會負責仲裁誰可以存取哪一些資源，確保有正確權限的使用者可以存取到他應該存取的資源。

例如，如果這個檔案是小明的，那其他阿貓阿狗就不應該可以打開或是閱讀小明的檔案。但如果小明想要把信分享給大家檔案，那作業系統也應該要有一套機制讓小明可以分享檔案給其他人看

作業系統是一個龐大且複雜的程式，很常會看到安全性的問題。一般來說，所謂的電腦病毒都是利用這一些安全性的漏洞來取得一些本來不應該取得的資源，例如你的檔案跟你的信用卡密碼之類的，為了要對抗這些並度，我們不需要常常安裝更新(patches)或者是更新你的作業系統。

## 效能

因為作業系統提供很多功能，他的效能就非常的重要。作業系統會頻繁的被執行，所以就算某一個程式多花費一些CPU的時間都會放大成很大的效能問題。

作業系統必須要需要完整的挖掘硬體的效能，榨乾硬體的可用價值。所以寫作業系統的人需要理解非常細微的硬體架構知識。

寫作業系統的的工作就是決定一個「政策」，通常有一好沒兩好，某個部份效率變好另外一個部份就會便慢。如何去權衡作業系統的表現就是寫作業系統的關鍵。

{% hint style="info" %}
Overhead

這個常常聽到，英文的解釋是「經常性開銷」，通常用在描述效能的場合

the regular and necessary costs, such as rent and heating, that are involved in operating a business
{% endhint %}

\


\
