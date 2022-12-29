# CPU

### CPU簡介

<figure><img src="../.gitbook/assets/image.png" alt=""><figcaption><p>CPU 的操作簡介。右手邊這個是CPU的指令，他的意思是，R1暫存器儲存100這個數值，R2暫存器讀取記憶體中0x100這個記憶體位置的數值(也就是10)，然後R3暫存器會是R1,R2這兩個數值加起來，最後再把R3暫存器的數值存到記憶體中0x110 的位置，理論上數值會是110。</p></figcaption></figure>

超級簡化的來說，一台電腦包含一個處理器(central processing unit, CPU)連著一些記憶體(memory)，上面這張圖解釋了一般CPU的操作

1. 從記憶體讀取(Load)數值，把它放到暫存器(register)和從暫存器把數值儲存(save)到記憶體
2. CPU只會操作在暫存器的資料。例如，在兩個暫存器之中做加減乘除、執行位元操作(bitwise and, or,之類的操作)、或是其他數學計算(平方根, sin, cos, tan 之類的)

因此，在這張圖中，描述我們CPU簡單的存取記憶體中的數字，加100之後再儲存在記憶體之中。



### 分支(Branching)

除了存取資料以外，CPU還有一種重要的操作就是Branching，CPU內部會保存「下一個指令的位置」，這個資訊存放在_`instruction pointer`_裡面。通常，指令指標會遞增指向下一個要執行的CPU的指令，分支指令通常會檢查特定的暫存器是否為零或是有特殊標記，如果有的話，就會修改指令指標到一個不同的記憶體位置，因此，下一個要執行的指令就會跳到程式的其他位置，這個就是一般迴圈跟if else的在　CPU指令執行的樣子。

舉個例子，如果我們寫 if(x==0)，這樣子的程式，我們需要兩個暫存器，R1放x的數值，R2放0，如果這兩個暫存器比較的結果是零(用 R1 or R2 == 0? 或 R1 | R2 == 0 ?)，就代表x 這個數值是0，這個條件成立，所以執行接下來的程式碼，如果沒有成立的話，就把指針只到另外一個地方(else的地方)



### 1.2 Cycles

We are all familiar with the speed of the computer, given in Megahertz or Gigahertz (millions or thousands of millions cycles per second). This is called the _clock speed_ since it is the speed that an internal clock within the computer pulses.

The pulses are used within the processor to keep it internally synchronised. On each tick or pulse another operation can be started; think of the clock like the person beating the drum to keep the rower's oars in sync.

### 1.3 Fetch, Decode, Execute, Store

Executing a single instruction consists of a particular cycle of events; fetching, decoding, executing and storing.

For example, to do the `add` instruction above the CPU must

1. Fetch : get the instruction from memory into the processor.
2. Decode : internally decode what it has to do (in this case add).
3. Execute : take the values from the registers, actually add them together
4. Store : store the result back into another register. You might also see the term _retiring_ the instruction.
