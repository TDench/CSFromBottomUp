# 周邊與匯流排

任何連接到電腦的外部裝置都叫做電腦周邊(Peripherals )，處理器必須要有某種跟外部裝置溝通的功能，才能使這些裝置有用。

處理器跟周邊溝通的訊息管道就叫做匯流排(_bus_)

## 概念

為了讓裝置的能溝通，使得輸入輸出有意義。以下會介紹一些常見的概念。如中斷( Interrupts) 之類的概念

## **中斷 (Interrupts)**

中斷就是允許設備停止處理器，並標記一些問題。例如，當你按下鍵盤的一個鍵時，就會產生一個中斷訊號，傳遞這個「按鍵被按下去了」的這個中斷事件給作業系統。作業系統跟BIOS共同指派一個中斷組合給每一個設備。

設備通常都會連結到一個可程式設計中斷控制器(_programmable interrupt controller_, PIC) ，這是一個主機板上獨立的晶片，他會緩衝(buffer)跟傳遞中斷訊號給處理器。每一個設備跟PIC之間都有一條物理上存在的中斷線(i_nterrupt line_)，當設備想要發出中斷的時候，他會調整這條線的電壓。

用一個很簡單明瞭的方法去描述PIC的作用的話，他負責接收中斷訊號，然後把這個訊號翻譯成處理器能處理的訊息。雖然說整個過程跟處理器的結構有關，但一般來說作業系統會配置一個中斷描述表(_interrupt descriptor table_)，然後將每個中斷對應到一段程式碼，這樣就可以在收到中斷的時候跳到這個地方執行，如下圖所示

撰寫中斷處理(_interrupt handler_)是寫設備driver的人，跟寫作業系統的人要煩惱的。



<figure><img src="https://www.bottomupcs.com/chapter02/figures/interrupt.svg" alt=""><figcaption><p>如何處理中斷的示意圖，有一條物理的線會從設備連接到PIC，PIC感受到電壓的變化，就把這個訊息翻譯成處理器看得懂的資料，然後有一張表格可以知道，收到這種中斷訊號之後，要跳去哪一段程式碼的IDT表格，最後作業系統會找到這個既作業系統會找到這個記憶體位置，並且執行這個中這個中斷該做的事。</p></figcaption></figure>

大多數的驅動(driver)會將中斷處理分成底層跟頂層兩個部分。底層負責接收並確認中斷訊號、暫存手邊的工作、和讓處理器可以快速回到剛剛在處理的工作。上層的中斷處理接著會在CPU空閒的時候做更多密集的處理。這樣做是避免中斷占用整個 CPU

### **保存狀態(Saving state)**

由於中斷隨時都有可能會發生，所以處理完中斷訊後之後可以返回到剛剛執行的操作是非常的重要。通常，這個是作業系統的工作。他的任務就是當進入中斷處理時，保存現在所有的狀態(例如暫存器的數值)，然後等中斷處理完之後，可以恢復到原來的狀態。如此，除了會花一些時間處理中斷以外，中斷對於執行時期的process來說是沒有影響的。

### **中斷 v 陷阱，例外處理(exceptions)**

雖然說到中斷，通常是指來自於物理設備的外部事件，但中斷的這個處理機制是對於處理系統也是非常的有用。例如，如果處理器偵測到有人想要存取無效的記憶體位置，或是把整數除以零，或是一些無效指令的時候，處理器可以在內部發起一個例外(_exception_ )訊號去處理這種事件。中斷這個處理機制也是跟系統呼叫(system)一樣的機制，雖然說內部發起的例外跟外部發起的例外不同，但處理非同步中斷的機制是一樣的。\[補上虛擬記憶體的章節連結]



### **中斷的種類**

觸發中斷的判斷分兩種，一個是電位觸發(_level_ triggered)，一個是邊緣觸發(_edge_ triggered)

電位觸發定義中斷線路的電壓是高的時候，代表有中斷存在(pending)，邊緣觸發的中斷會檢查線路電壓的變化(電壓從低到高的瞬間)。所以，在邊緣觸發中斷的情況下，一個方波輸入會產生一個中斷。

剛剛講的聽起來很不知道差異在哪裡，下面這個例子可以體現他們的差異。假設你有三個設備連到同一條中斷線上，

* 在電位觸發的狀態下\
  三個設備都發出中斷訊號，才會使電位上升到中斷的有效狀態(asserted)，處理的時候也是要三個都處理完，中斷線的狀態才會成為無效狀態(un-asserted)
* 在邊緣觸發的狀態下\
  任何一個設備發出中斷訊號的瞬間，PIC都會偵測到變化然後請作業系統處理。However, if further pulses come in on the already asserted line from another device.(這句看不懂...)

電位觸發中斷的問題是，可能需要一段蠻長的時間處理中斷。在這段時間，中斷訊號線一直維持高電位，這種情況下就沒有辦法判斷有沒有其他外部的設備發出中斷訊號。這代表，在處理中斷服務時\\





电平触发中断的问题是，可能需要相当长的时间来处理设备的中断。在此期间，中断线保持较高，无法确定是否有任何其他设备在该线路上引发中断。这意味着在服务中断时可能会有相当大的不可预测的延迟。

使用边缘触发的中断，可以注意到并排队等待一个长时间运行的中断，但在这种情况发生时，其他共享线路的设备仍然可以转换(从而引发中断)。然而，这带来了新的问题；如果两个设备同时中断，可能会错过其中一个中断，环境或其他干扰可能会造成一个应该忽略的_假性_中断。



The issue with level-triggered interrupts is that it may require some considerable amount of time to handle an interrupt for a device. During this time, the interrupt line remains high and it is not possible to determine if any other device has raised an interrupt on the line. This means there can be considerable unpredictable latency in servicing interrupts.

With edge-triggered interrupts, a long-running interrupt can be noticed and queued, but other devices sharing the line can still transition (and hence raise interrupts) while this happens. However, this introduces new problems; if two devices interrupt at the same time it may be possible to miss one of the interrupts, or environmental or other interference may create a _spurious_ interrupt which should be ignored.
