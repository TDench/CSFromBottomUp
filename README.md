# 從下而上的電腦科學(Computer Science from the Bottom Up)

## 歡迎

歡迎來到從下而上的電腦科學。

* 本書翻譯自 **Ian Wienand** 的 Computer Science from the Bottom Up。[https://www.bottomupcs.com/](https://www.bottomupcs.com/)
* 翻譯的時候我基本上不是逐字翻譯，有些是為了更容易理解做的中文語句調整。有些為了解釋新增了一些自己的想法在上面，如果有翻譯錯誤或理解錯誤，麻煩各位路見不平拿patch來填。拜託QQ\
  [https://github.com/TDench/CSFromBottomUp](https://github.com/TDench/CSFromBottomUp)



## 作者的想法

簡單的來說，你接下來要閱讀的東西就是電腦科學的手把手實作課程。年輕的電腦科學學生被教導成如何去使用電腦。但是要去哪裡學習更底層的東西呢?理解作業系統不像是車子打開引擎蓋就可以一探究竟，如今Linux C核心(Kernel)內部運行著數百萬行的程式碼，再加上一些現代作業系統中需要執行的關鍵部分(編譯器、組譯器(assembler)、系統函式庫)，我們寫程式的基礎設施根本無法想像。然後就變成了，使用大學等級的作業系統課程知識，加上兩三年的C語言程式經驗，或許(也只是或許)，你才有機會知道要從哪裡開始看懂這些架構。

就好像有些學生可能還沒有搞清出二行程引擎就想要去做F1賽車的引擎，理論上應該要先了解最簡單的引擎構造，然後分解、優化、重新組合之後，有了概念再來理解F1賽車引勤的工作原理。我們不期望學生馬上就能變成F1賽車引擎工程師，但藉由這樣的學習路程，我們才會離目標越來越近

## 為什麼要從下而上學習

不是每個人都願意參加手工藝課程，從無到有的創造出東西。大多數的人只想要開車，不會想要憑空做出一台車。而一般的程式課程也考慮到了這點，所以大致上的電腦科學都是「由上而下」的教學：從應用程式，高級語言，軟體設計，開發原理，資料結構。或許可能會學一些簡單不深入的二進位，二進位邏輯，或是更底層的一些概念與知識，如暫存器(Register)、CPU指令(opcodes)。

本書的目標就是完全跟剛剛所說的方向相反的教學，從作業系統基礎講到應用程式是如何被編譯與執行

## 開源萬歲XD

多虧了開源技術([Open Source](https://www.bottomupcs.com/gloss01.html#opensource))，這本書才可能完成。在Linux 出現在這個世界之前，我們上課就像是在不能打開引擎蓋的車子上面學習車子的結構，如今我們可以打開這個引擎蓋，戳看看這個零件有什麼用途，做更棒的是，我們可以直接拆引擎下來做自己想要做的任何的事。

