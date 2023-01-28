# 小系統到大系統

根據莫爾定律，電腦的計算能力用一個很誇張的速度在增長，而且目前看起來並不會減慢。一般來說，高階系統，伺服器等電腦都不是只有一個CPU的系統。對於如何達成各個CPU的協作有很多種方式。

## 對稱多處理系統(Symmetric Multi-Processing)

對稱多處理系統(Symmetric Multi-Processing, SMP) 是一個系統裡面包含多個CPU的一個常見的配置。`symmetric` 這個字代表所有的CPU都是相同的(同樣的架構，同樣的時鐘速度)。在SMP系統中，多個處理器會共享其他人的資源(resources)，例如記憶體、硬碟空間之類的。

### 快取一致性(Cache Coherency)

大多數的情況下，系統中的 CPU獨立運作，每個人都有自己的一組暫存器、program counter等。儘管他們分別獨立的運行，有一個重要的部分需要嚴格的同步(synchronisation)。

這個東西就是 CPU 快取，怕你忘記提醒一下，快取就是一個小的記憶體位置，可以快速的存取主記憶裡面的資訊。如果一個CPU對主記憶體中的資料進行修改，然後其他的CPU還在讀自己手上本來有的快取資訊，這樣系統就不處於同步的狀態。順帶一提，這種問題只有在寫入的時候才會發生，因為如果只是讀取的話，每個CPU讀取的資料都會相同。

為了要協調所有處理器都處於快取一致性的狀態，SMP系統會使用窺探(snooping)，窺探就是處理器會去監聽所有連接到匯流排的處理器，有沒有發生快取改變的事件，如果有的話就會作相對應的快取更新。

其中一個快取一致性的協定(Protocol)是MOESI協定，這幾個英文字分別代表修改（Modified），所有權（Owner），獨佔（Exclusive），共享（Shared），無效（Invalid）。這裡每一個英文字都是代表快取行(cache line) 上處理器的狀態。當然，還有協定，但是他們都有相似的概念，下面我們會仔細觀察 MOESI的協定，讓你有基本的認識。

當處理器需要從主記憶體讀取快取的時候，處理器會先去窺探(snooping)系統中所有的處理器，看看其他處理器有沒有對於這塊記憶體的訊息(例如，這段記憶體已經被其他處理器快取了)。如果目前並沒有任何處理器有這段記憶體的資料，那使處理器就會快取這段記憶體，然後把這段標為獨佔(E)，當寫入快取這件事情發生的時候，就把狀態標記程修改(M)，下面開始就有趣了，有一些快取會馬上寫回主記憶體(這個就被稱為 write-through ，因為寫入的動作直接寫到主記憶體)，有一些快取會只先寫到快取上，除非快取被寫滿了或是逐出(evicted)才會寫入主記憶體。

另外一種情況就是當處理器窺探其他處理器的時候，發現有人標記這段記憶體為修改(M)，這個時候就把資料複製一分到自己的快取，並且標記共享(Shared)，他會傳訊息給其他處理器，告訴大家剛剛那個M改成所有者(O)。想像中，現在有第三個處理器想要使用這塊記憶體的資料的時候，他會看到一個S跟一個O。這個時候他會去O那邊複製一份資料到自己的快取。如果現在其他處理器都只是要讀取，快取行(cache line)就會維持共享(S)的狀態。但是，當其中一個處理器想要更新那段資料的時候，他需要傳送無效(invalidate)訊息給其他的處理器。有快取這段資料的處理器就要把自己的快取標記成無效(I)，代表現在這個快取上面的資料已經不是正確的資料了。當這段無效訊息除給其他處理器之後，自己的快取行標記成修改(M)，其他處理器標記程無效(I)。順帶一提，如果這個快取行是獨佔的(E)，處理器就會知道沒有其他的處理器會用到這段資料，就可以避免傳送無效訊息給其他處理器。

從現在開始，整個邏輯就開始了。所以不管是哪個處理器擁有被標記成修改(M)的快取，那個處理器就有責任在逐出(evicted)的時候，把這段資料寫回主記憶體。透過思考這個協定，你就會知道，這個協定是要確保各個處理器的快取行的資料是一致的。

當系統處理器的數量增加的時候會產生一些問題。當處理器數量不多的時候，檢查其他的處理器是否擁有這個快取行的資料，或是檢查這個快取行是不是有效的資料(這個就是一個snoop的意思)所花的時間不多，但如果很多個處理器的時候，匯流排的訊息會太忙碌。這也就是為什麼SMP系統通常最多只有八個處理器的原因。

而且將所有的處理器都放在同一個匯流排上有物理上的限制，一般物理上的電線只能用特定的距離去排列，並且有長度的限制。對於一秒鐘幾千兆赫茲(GHz)運算的處理器來說，就算是用光速傳遞資訊，也需要考慮資料在這些導線裡面傳輸的時間。

請注意，通常系統軟體不會參與這個過程，雖然說寫程式的人應該知道硬體底層做了什麼事情，才可以知道怎麼樣最佳化自己的程式效率。

### **SMP系統中的快取獨佔性**

我們在記憶體與快取那小結中有討論到，有一個東西叫做 inclusive v exclusive 快取，一般來說，L1快取通陳都是具有包容性(inclusive)，這個意思就是，所有在L1的資料，也會存在L2中。在多處理器的系統來說，一個包容性的L1快取代表只有L2的快取需要做窺探(snoop)這個動作來維持快取一致性。因為L2資料的改動保證是從L1來的，這樣就可以將L1從這個複雜的窺探機制內解離出來，使得L1的設計更為簡單，也使得窺探機制變得更快。

一般來說，現代的高端處理器(非嵌入式的處理器)，會有L1的 write-through 的機制，也會有其他階級的快去得 write-back 機制。這個有很多原因，例如，這種等級的處理器的 L2 快取幾乎都是獨立且位於晶片上面，而且大部分都速度很快，所以 L1 直寫的時間不是很長。除此之外，因為L1的大小很小，將來不太可能去讀L1上面的資料，去讀L1上的資料又有可能造成資料汙染，浪費L1資源(因為要維持快取一致性)，一個L1的直寫L1不需要曲可這些東西，因此可以將一致性的邏輯傳給L2去做處理(就跟剛剛說的一樣，已經在快取一致性那邊做處理)

Further, since L1 sizes are small, pools of written data unlikely to be read in the future could cause pollution of the limited L1 resource. Additionally, a write-through L1 does not have to be concerned if it has outstanding dirty data, hence can pass the extra coherency logic to the L2 (which, as we mentioned, already has a larger part to play in cache coherency).這段看不懂。。

### **超執行緒 Hyperthreading**

現代處理器大部分都時間都在等待比較慢的周邊設備，等待周邊設備把資料傳到記憶體之後處理器才會做處理。

因此，為了要保持CPU的流水線被充分的利用，產生了一種策略，這個策略就是只使用剛好夠用的少量暫存器跟指令，保持兩個組指令可以同時處理，把一個CPU看成兩個CPU的感覺。

當每個CPU都有自己的暫存器，但是他們必須共享CPU的計算核心，快取，記憶體輸入輸出的頻寬。兩個指令流(instruction streams)可以讓CPU感覺更忙碌一些，但是提升的效率沒有像真的有兩顆物理上分開的CPU還要多。基本上會提升的效率小於20%，但根據工作負載(workload)不同，效能可以會有顯著的提升，或是顯著的更爛。

### **多核心 Multi Core**

因為晶片上可以放的電晶體越來越多，在同一個晶片封裝(package)上可以放一個或是多個的處理器。最常見的就是雙核心(dual-core)，就是把兩個處理器安裝在同一個晶片上，這些CPU核心，不像是剛剛提到的超執行緒，是真的有兩個物理上分開的處理器組成的SMP系統

雖然每個處理器都擁有自己的L1快取，但是他們要共享同一個連結到主記憶體跟外部設備的匯流排。他的效率不會像真的SMP系統一樣的快，但是一定比超執行緒系統好，而且一般來說，每一個CPU核心還是會實作hyperthreading 增加自己的處理效能

多核心處理器還有一些跟性能無關的優點，就跟我們剛剛說明的，匯流排在物理上有長度上的限制，把兩個處理器放置在物理上十分相近的位置，可以解決線路不要太長的問題。而且多核心處理器所需要的能源比較小，也就意味著更少的廢熱會產生。這個對數據中心的散熱來說是很重要的問題。透過把CPU核心封裝在一起，可以實現多核心處理，像是應用在筆記型電腦。此外，一個晶片比兩個晶片便宜很多。



## 群集 Clusters

很多應用程式需要更多的處理器幫忙處理，這個數量大於SMP系統很多。有一種方法可以擴張處理器的數量，就是群集(Clusters) 的概念。

群集就是簡單的將很多的電腦連結在一起。在硬體的層級上來看，每台電腦並不知道其他電腦在做什麼，將各台電腦連接在一起的這個任務是由軟體達成。

就像是 MPI 之類的軟體可以讓程式工程師藉由編寫一些軟體把部分的電腦移除(farm out)。舉個例子，假設有一個迴圈要執行上千次的獨立指令，而且這些指令迭代不會影響其他的迭代。假設群集之中有四台電腦，就把它分配給四台電腦，每個電腦分別計算250次這樣。

電腦之間的連結方法有很多種，有可能像一般網路一樣慢，或可能跟專用線路(Infiniband)一樣快。無論連結的方式如何，記憶體的階層又會往下產生一階，會多一個比RAM更慢的階層。所以當群集裡面的電腦要讀取其他電腦的記憶體的時候，就會表現得不好，因為有這種需求的時候，軟體就需要發出一個請求，請求其他電腦把他的資料送一份過來，然後經由很慢的線路傳送過來，然後載入到到自己的記憶體，然後處理器才可以開始工作。

但是，許多應用程式並不需要這種電腦之前的互相複製，傳送資料的功能。一個大型的例子就是 SETI@Home ，這個專案裡面會有來自於望遠鏡的無線電資料，希望可以拿來找到外星生命。每一台電腦會被分配到一小段資料去做分析，然後回傳自己的分析結果就可以。所以這個專案實際上就是一個非常大的群集

另外一個運用就是拿來渲染影像(rendering)，尤其是電影裡面的特效。每一台電腦都負責一幀，分別處理火焰，紋理渲染，光暈渲染之類的特效，然後把他們組合在一起，就可以變成我們看到的驚人特效。因為每一幀的畫面都是靜態的，電腦有了這個輸入之後，就不需要其他的資訊了。所以可以分配給很多電腦計算之後再回傳即可。例如魔戒(Lord of the Rings)就是用Linux的巨大群集做出來的特效。



## 非統一記憶體存取架構 (Non-uniform memory access)

非統一記憶體存取架構(Non-uniform memory access, NUMA)，幾乎和上面提到的群集的概念相反。在群集系統中就像是連結了很多節點，且連結成本非常的特殊(昂貴)，每個節點的硬體互相不知道對方的訊息。在NUMA架構中，軟體沒有太多對於系統的認識，大部分都是由硬體將結點連接在一起。

這個NUMA這個專有名詞來自於 CPU 跟 RAM 在不同的物理位置，所以資料傳輸需要從一個有點距離的地方傳過來，顯然需要再多花一些時間。跟單個處理器或SMP系統不同，單處理器或SMP系統的記憶體是直連到RAM，而且只需要花一個定量(constant)的時間存取

### **NUMA 系統架構**

由於系統中有很多節點要互相溝通，所以把這些節點之間的距離最小化就是很重要的議題了。很明顯的，最簡單直接的方法就是把每個節點都連在一起，就可以最大程度的減少一個節點到另外一個節點的距離。但當數量變成幾百幾千的時候，這種解法的就顯得不太實際了。如果你高中數學還沒有忘記的話，n個點任意連接在一起的組合有 `n!/2*(n-2)!` 這樣。

為了要解決這個指數成長的問題]，我們會採用另外一種布局，權衡距離跟所需要連結的節點數量。現代常見的布局就是超立方體(hypercube)架構。

這個超立方架構有嚴謹的數學定義(超過我們討論的範圍)，但是簡單來說立方體就是一個3維的立體構造，超立方體就是大概是4維的感覺。

<figure><img src="../.gitbook/assets/image (2).png" alt=""><figcaption><p>超立方體的示意圖</p></figcaption></figure>

我們可以看到上面這張圖，感將像是立方體

Above we can see the outer cube contains four 8 nodes. The maximum number of paths required for any node to talk to another node is 3. When another cube is placed inside this cube, we now have double the number of processors but the maximum path cost has only increased to 4. This means as the number of processors grow by 2n the maximum path cost grows only linearly.

上面我们可以看到外部多维数据集包含四个 8 个节点。 任何节点与另一个节点通信所需的最大路径数为 3.当另一个多维数据集放置在此多维数据集内时，我们现在有两倍的处理器数，但最大路径成本仅增加到 4.这意味着数量 处理器增长 2n，最大路径成本只

### **Cache Coherency**

Cache coherency can still be maintained in a NUMA system (this is referred to as a cache-coherent NUMA system, or ccNUMA). As we mentioned, the broadcast based scheme used to keep the processor caches coherent in an SMP system does not scale to hundreds or even thousands of processors in a large NUMA system. One common scheme for cache coherency in a NUMA system is referred to as a _directory based model_. In this model processors in the system communicate to special cache directory hardware. The directory hardware maintains a consistent picture to each processor; this abstraction hides the working of the NUMA system from the processor.

The Censier and Feautrier directory based scheme maintains a central directory where each memory block has a flag bit known as the _valid bit_ for each processor and a single _dirty_ bit. When a processor reads the memory into its cache, the directory sets the valid bit for that processor.

When a processor wishes to write to the cache line the directory needs to set the dirty bit for the memory block. This involves sending an invalidate message to those processors who are using the cache line (and only those processors whose flag are set; avoiding broadcast traffic).

After this should any other processor try to read the memory block the directory will find the dirty bit set. The directory will need to get the updated cache line from the processor with the valid bit currently set, write the dirty data back to main memory and then provide that data back to the requesting processor, setting the valid bit for the requesting processor in the process. Note that this is transparent to the requesting processor and the directory may need to get that data from somewhere very close or somewhere very far away.

Obviously having thousands of processors communicating to a single directory does also not scale well. Extensions to the scheme involve having a hierarchy of directories that communicate between each other using a separate protocol. The directories can use a more general purpose communications network to talk between each other, rather than a CPU bus, allowing scaling to much larger systems.
