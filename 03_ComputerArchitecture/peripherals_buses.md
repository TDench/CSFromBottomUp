# 周邊與匯流排

任何連接到電腦的外部裝置都叫做電腦周邊(Peripherals )，處理器必須要有某種跟外部裝置溝通的功能，才能使這些裝置有用。

處理器跟周邊溝通的訊息管道就叫做匯流排(_bus_)

## 概念

為了讓裝置的能溝通，使得輸入輸出有意義。以下會介紹一些常見的概念。如中斷( Interrupts) 之類的概念

## **中斷 (Interrupts)**

中斷就是允許設備停止處理器，並標記一些問題。例如，當你按下鍵盤的一個鍵時，就會產生一個中斷訊號，傳遞這個「按鍵被按下去了」的這個中斷事件給作業系統。作業系統跟BIOS共同指派一個中斷組合給每一個設備。

設備通常都會連結到一個可程式設計中斷控制器(_programmable interrupt controller_, PIC) ，這是一個主機板上獨立的晶片，他會緩衝(buffer)跟傳遞中斷訊號給處理器。每一個設備跟PIC之間都有一條物理上存在的中斷線(&#x69;_&#x6E;terrupt line_)，當設備想要發出中斷的時候，他會調整這條線的電壓。

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

電位觸發中斷的問題是，可能需要一段蠻長的時間處理中斷。在這段時間，中斷訊號線一直維持高電位，這種情況下就沒有辦法判斷有沒有其他外部的設備發出中斷訊號。這代表，在處理中斷服務時，會產生一段不可預期的延遲(latency)

使用邊緣觸發中斷，就可以注意到這些中斷，並且把中斷隊列(queue)起來。同時，其他設備可以發起電位轉換(transition)觸發中斷。但是，這個有一個新的問題，就是如果兩個設備同時發起中斷，可能會錯過其中一個中斷訊號，或是環境因素，或是訊號干擾，都會產生一個假性(_spurious_)的中斷，這些中斷都是應該被忽略的。



### 非可遮蔽**中斷(Non-maskable)**

對於系統來說，能在某些重要的時間範圍內不被中斷，也就是能遮蔽(_mask)_&#x6216;阻止中斷是非常重要的。一般來說，中斷是可以被暫停並且晚點再處理的。但是有些特別的中斷，也就是非可遮蔽中斷(NMI)，這種中斷是不能被遮蔽，需要馬上被處理的，例如重置訊號(reset)。

NMI 很適合拿來實作系統看門狗(watchdog)，每當NMI被週期性的被觸發，然後設定一些標誌(flag)給作業系統處理。如果作業系統沒有看到這些NMI設定的標誌的話，作業系統就會認為這件事情沒有進展。另外一個常見的用途就是用來剖析系統，週期性的觸發NMI，去看看現在處理器是在執行哪段程式碼，隨著時間紀錄系統每一個時刻在做什麼，就可以用來分析系統效率。

## **IO 空間(Space)**

顯而易見地，處理器需要去跟周邊設備溝通，一般是使用 IO 操作(operations)來實現設備的溝通。最常見的IO叫做記憶體對映IO(_memory mapped IO_)，也就是每一個記憶體位置會對應到一個設備。

這就代表說，你需要去跟其他設備溝通，你就是簡單的去存取特定的記憶體位置就可以了。

## DMA

由於周邊設備的速度遠遠的比處理器還慢，因此需要一些方法來避免CPU等待資料從設備傳過來。

直接記憶體存取(Direct Memory Access, DMA)，就是一種資料直接從周邊設備傳到系統記憶體的這個行為。驅動程式可以設定設備做一個DMA傳輸資料，並且要求一塊記憶體放資料。然後開始傳輸資料，讓CPU可以去做其他工作。

當設備完成這項操作之後，他會舉起中斷或是訊號，來告訴驅動程式傳完資料了。從這個時候開始，設備傳出去的資料就是可以使用的了(可以想像成硬碟裡面的檔案傳玩了才可以使用，或者是影片傳到完了才可以看這樣)



## 其他匯流排

其他匯流排連接在PIC匯流排跟其他外部設備之間

### **USB**

從作業系統的角度來看，UBS相關的設備就是一組端點(end-points)聯在一起的介面，每一個端點只能是輸入或是輸出，所以資料傳輸的方向是單向的，端點都有以下這些類型

* 控制端點(_Control_)，也就是用來設定設備的參數用的
* 中斷端點，也就是拿來傳輸少量的資料，具有高優先的特質
* &#x20;批量端點(_Bulk_ )，用來傳輸大量的資料，但不保證在時間內傳達
* 等時傳輸是高優先的即時傳輸，但當資料丟失的的時候不會重傳，用來傳輸串流資料，如影片或是聲音之類的訊號，這種資料再傳一次沒有意義。

很多端點(end-points)組成一個介面(interfaces)，很多介面組成一個設定(_configurations_)，大部分的設備都只有一個設定

<figure><img src="../.gitbook/assets/image (3).png" alt=""><figcaption><p>UHCI控制器的簡介。</p></figcaption></figure>



上圖示UHCI控制器的示意圖，這張圖大致說明  UHCI 也就是 universal host controller interface 的運作。UHCI 的資料室如何藉由軟體跟硬體的配合從系統傳出去。也就是說，軟體會設定一個資料的模板，一個特定格式的資料模板，讓控制器可以藉由USB匯流排讀取資料。

我們從左上角開始看，控制器有一個  _frame_ 暫存器、跟一個計數器(每毫秒遞增)。這組暫存器(bit 2\~31)會做為 _Frame List_ 的索引指標。_Frame List_  是一張表，這張表裡面存放著 _transfer descriptors_ 的 queue 的開頭。軟體會在記憶體中創建這張Frame list 表格，控制器會去讀取這個記憶體位置的表格(控制器是獨立的晶片，可以驅動USB匯流排)，軟體負責排程工作queue，用以達成90%的時間在處理等時的資料，10%的時間在處理中斷、控制器、跟大型資料傳輸。

從這張也可以看到，這種 linked list queue的就代表，每一個等時傳輸都對應到一個特定的一個frame。也就是說，只有一個特定的時間可以傳輸，之後這段資料就會被丟棄。但是中斷，控制訊號，跟大型資料傳輸都是排在等時資料傳完之後才傳輸，因此如果在那個時間點沒有處理完，就會在下一個frame去把它做完。

USB層是透過 URBs (USB request block) 傳輸，URB裡面會記載，資料、這筆資料要傳給哪個端點、相關的訊息、屬性(attributes )、當 URB 傳輸完成的時候要呼叫的call-back 函式。 USB驅動會以固定格式傳送URBs給USB core。USB core會跟控制器協調處理這些URBs。你的數據由USB core傳給 USB 設備，當傳輸完成的時候會呼叫 call back 函式。
