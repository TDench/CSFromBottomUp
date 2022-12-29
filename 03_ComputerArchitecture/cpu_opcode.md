# CPU與指令

## CPU簡介

<figure><img src="../.gitbook/assets/image (1).png" alt=""><figcaption><p>CPU 的操作簡介。右手邊這個是CPU的指令，他的意思是，R1暫存器儲存100這個數值，R2暫存器讀取記憶體中0x100這個記憶體位置的數值(也就是10)，然後R3暫存器會是R1,R2這兩個數值加起來，最後再把R3暫存器的數值存到記憶體中0x110 的位置，理論上數值會是110。</p></figcaption></figure>

超級簡化的來說，一台電腦包含一個處理器(central processing unit, CPU)連著一些記憶體(memory)，上面這張圖解釋了一般CPU的操作

1. 從記憶體讀取(Load)數值，把它放到暫存器(register)和從暫存器把數值儲存(save)到記憶體
2. CPU只會操作在暫存器的資料。例如，在兩個暫存器之中做加減乘除、執行位元操作(bitwise and, or,之類的操作)、或是其他數學計算(平方根, sin, cos, tan 之類的)

因此，在這張圖中，描述我們CPU簡單的存取記憶體中的數字，加100之後再儲存在記憶體之中。



## 分支(Branching)

除了存取資料以外，CPU還有一種重要的操作就是**Branching**，CPU內部會保存「下一個指令的位置」，這個資訊存放在_`instruction pointer`_裡面。通常，指令指標會遞增指向下一個要執行的CPU的指令，分支指令通常會檢查特定的暫存器是否為零或是有特殊標記，如果有的話，就會修改指令指標到一個不同的記憶體位置，因此，下一個要執行的指令就會跳到程式的其他位置，這個就是一般迴圈跟if else的在　CPU指令執行的樣子。

舉個例子，如果我們寫 if(x==0)，這樣子的程式，我們需要兩個暫存器，R1放x的數值，R2放0，如果這兩個暫存器比較的結果是零(用 R1 or R2 == 0? 或 R1 | R2 == 0 ?)，就代表x 這個數值是0，這個條件成立，所以執行接下來的程式碼，如果沒有成立的話，就把指針只到另外一個地方(else的地方)



## CPU時派(Cycles)

我們都知道電腦的速度用 Mhz 跟 GHz 表示(每秒百萬次，或每秒十億次)，我們稱這個速度叫做時脈(clock speed)，因為他就是電腦內部時鐘脈衝的速度。

處利器內部使用脈衝來保持同步，每一次脈衝，就可以開始一個新的操作。可以把他想像成，每敲一次鼓，船上的人就划一次槳，的這種感覺，他們會保持同步。

## 提取( Fetch), 解碼(Decode),執行( Execute),(儲存) Store

執行一條指令包含以下這些時間：拿取＋解碼＋執行＋儲存資料。

也就是說，在CPU上面要執行 add 的指令，必須

1. 提取：將指令從記憶體輸入到處理器中
2. 解碼：CPU 解碼這個指令的意思(在這個範例之中，CPU 會知道要 `add`)
3. 執行：從暫存器拿取數值，把數字加在一起
4. 儲存：把結果儲存到另外一個暫存器，這個步驟叫做 _**retiring**_** ** 指令

### **看看 CPU的內部構造**

在CPU裡面有很多不同的部分在執行剛剛說到的每一個步驟，通常這些步驟都可以被獨立的執行。這就好像是一條工廠生產線一樣，有很多站點。每一個步驟都有一個特定的任務要執行，當這個任務執行完畢的時候，就會把結果傳遞給下一個站點，然後接收上一站點的新的工作。

<figure><img src="../.gitbook/assets/image.png" alt=""><figcaption><p>這張圖是CPU的內部構造，這張圖簡介了現代處理器裡面一些主要的原件，裡面有解碼(decode)，算術運算(ALC)、浮點數運算(FPU)、快取記憶體(cache)等，CPU可以直接存取記憶體(RAM)，並且接受指令的輸入。</p></figcaption></figure>



在這張圖之中，我們可以看到指令(instructions)進入CPU之後，會先被解碼。CPU裡面有兩種暫存器。一種是給整數運算用的，另外一個是給浮點數運算用的。浮點數是一種以二進位的形式來表達小數位數的方法(可以看ieee的754 ，跟我想的都不一樣XD)，浮點數的運算在每種不同的CPU中處理的方式都有所不同。MMX(multimedia extension)、 _SSE_ (Streaming Single Instruction Multiple Data)、_Altivec_ 暫存器這些都是用來處理浮點數的暫存器。

所謂的 _register file_ 是CPU內部暫存器的統稱。下面會介紹CPU各個部份的工作內容。

剛剛有說到，處理就要不就把數值讀取到暫存器中，要不就把暫存器的數值存到記憶體中、要不就是對暫存器的數值進行一些運算操作，所以

**算數邏輯單元(**_**Arithmetic Logic Unit,**_** ALU)** 就是CPU的的核心，他讀取暫存器的數值，執行各種CPU運算操作。現代的處理器有很多個ALU，每個都可以獨立工作。例如 intel Pentium  系列的處理器就同時存在快的ALU跟慢的ALU，快的比較小(可以塞更多顆、執行簡單常見任務)，慢得比較大顆，但是可以執行其他所有的CPU操作。

位置生成單元(_Address Generation Unit,_ AGU)，處理快取(cache)跟主要記憶體的溝通。可以讓CPU讀取到正確的記憶體位置的那個數值，並且把計算之後的結果返回指定的記憶體位置。

浮點數暫存器的概念大概都差不多，但在不同的製造商之中有不同的專業術語跟結構





### **Pipelining**

就跟我們剛剛說的一樣，當 ALU 將站存器的數值們加起來，跟 AGU 將算出來的數值存回記憶體，這兩件事情完全獨立，所以 CPU 沒有理由不能同時處理這兩件事情。更何況，我們還有很多個獨立的 ALU ，每一個 ALU 都可以處理不同的指令。當然， CP依可以執行浮點數運算的時候同時執行整數運算。這種互相獨立的任務處理我們叫做 _pipelining_ 。可以這樣處理的CPU我們叫他 _superscalar architecture_ 所有的現代處理器都是這種架構。

我們可以把 pipeline 想像成水管裡面有很多彈珠，彈珠就是 CPU指令。理想的情況下，你會把你的彈珠塞到其中一端，一個接著一個塞，照著時脈塞進去。一旦整個水管滿了，你推入一個新的彈珠(指令)就會把所有的彈珠(指令)往前推一格，然後會有一個彈珠掉出來(答案掉出來)。

但是分支指令會破壞這個運作模式，因為他們可能會，也可能不會跳著執行指令。如果你是一條水管，你必須要猜猜看接下來會往哪個分支走，

但是，分支指令会对这个模型造成破坏，因为它们可能会或不会导致执行从不同的地方开始。如果您是管道，您将必须基本上猜测哪个分支将走向，所以您知道哪些指令要引入到管道中。相反，如果处理器预测错误，则浪费大量时间，必须清除管道，然后重新启动。

这一过程通常被称为_管道冲洗_,停止以及从您的软管中清空您的所有弹珠！

.

Branch instruction play havoc with this model however, since they may or may not cause execution to start from a different place. If you are pipelining, you will have to basically guess which way the branch will go, so you know which instructions to bring into the pipeline. If the CPU has predicted correctly, everything goes fine\![2](https://www.bottomupcs.com/ch03.html#the\_cpu\_s3\_s2\_para3\_footnote1-fnote) Conversely, if the processor has predicted incorrectly it has wasted a lot of time and has to clear the pipeline and start again.

This process is usually referred to as a _pipeline flush_ and is analogous to having to stop and empty out all your marbles from your hose!

**1.3.2.1 Branch Prediction**

pipeline flush, predict taken, predict not taken, branch delay slots

**1.3.3 Reordering**

In fact, if the CPU is the hose, it is free to reorder the marbles within the hose, as long as they pop out the end in the same order you put them in. We call this _program order_ since this is the order that instructions are given in the computer program.



Consider an instruction stream such as that shown in [Figure 1.3.3.1, Reorder buffer example](https://www.bottomupcs.com/ch03.html#reorder\_buffer) Instruction 2 needs to wait for instruction 1 to complete fully before it can start. This means that the pipeline has to _stall_ as it waits for the value to be calculated. Similarly instructions 3 and 4 have a dependency on _r7_. However, instructions 2 and 3 have no _dependency_ on each other at all; this means they operate on completely separate registers. If we swap instructions 2 and 3 we can get a much better ordering for the pipeline since the processor can be doing useful work rather than waiting for the pipeline to complete to get the result of a previous instruction.

However, when writing very low level code some instructions may require some security about how operations are ordered. We call this requirement _memory semantics_. If you require _acquire_ semantics this means that for this instruction you must ensure that the results of all previous instructions have been completed. If you require _release_ semantics you are saying that all instructions after this one must see the current result. Another even stricter semantic is a _memory barrier_ or _memory fence_ which requires that operations have been committed to memory before continuing.

On some architectures these semantics are guaranteed for you by the processor, whilst on others you must specify them explicitly. Most programmers do not need to worry directly about them, although you may see the terms.

#### 1.4 CISC v RISC

A common way to divide computer architectures is into _Complex Instruction Set Computer_ (CISC) and _Reduced Instruction Set Computer_ (RISC).

Note in the first example, we have explicitly loaded values into registers, performed an addition and stored the result value held in another register back to memory. This is an example of a RISC approach to computing -- only performing operations on values in registers and explicitly loading and storing values to and from memory.

A CISC approach may be only a single instruction taking values from memory, performing the addition internally and writing the result back. This means the instruction may take many cycles, but ultimately both approaches achieve the same goal.

All modern architectures would be considered RISC architectures[3](https://www.bottomupcs.com/ch03.html#the\_cpu\_s4\_para4\_footnote1-fnote).

There are a number of reasons for this

* Whilst RISC makes assembly programming becomes more complex, since virtually all programmers use high level languages and leave the hard work of producing assembly code to the compiler, so the other advantages outweigh this disadvantage.
* Because the instructions in a RISC processor are much more simple, there is more space inside the chip for registers. As we know from the memory hierarchy, registers are the fastest type of memory and ultimately all instructions must be performed on values held in registers, so all other things being equal more registers leads to higher performance.
* Since all instructions execute in the same time, pipelining is possible. We know pipelining requires streams of instructions being constantly fed into the processor, so if some instructions take a very long time and others do not, the pipeline becomes far to complex to be effective.

**1.4.1 EPIC**

The Itanium processor, which is used in many example through this book, is an example of a modified architecture called Explicitly Parallel Instruction Computing.

We have discussed how superscaler processors have pipelines that have many instructions in flight at the same time in different parts of the processor. Obviously for this to work as well as possible instructions should be given the processor in an order that can make best use of the available elements of the CPU.

Traditionally organising the incoming instruction stream has been the job of the hardware. Instructions are issued by the program in a sequential manner; the processor must look ahead and try to make decisions about how to organise the incoming instructions.

The theory behind EPIC is that there is more information available at higher levels which can make these decisions better than the processor. Analysing a stream of assembly language instructions, as current processors do, loses a lot of information that the programmer may have provided in the original source code. Think of it as the difference between studying a Shakespeare play and reading the Cliff's Notes version of the same. Both give you the same result, but the original has all sorts of extra information that sets the scene and gives you insight into the characters.

Thus the logic of ordering instructions can be moved from the processor to the compiler. This means that compiler writers need to be smarter to try and find the best ordering of code for the processor. The processor is also significantly simplified, since a lot of its work has been moved to the compiler.

