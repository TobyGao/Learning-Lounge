---
layout: article
title: 附錄.B 記憶體架構複習
key: 20171012
tags: 計算機結構
---

### 1. 記憶體架構
在使用上，我們希望記憶體讀取的速度越快越好，但實際上，迫於成本的考量，我們無法所有的儲存元件都用最快記憶單元來實現，故利用<font color="red">**快取記憶體(cache)**</font>加速這個方法就隨之而生，透過將記憶體階層化，我們可以將可能會用的資料先放置到速度較快的且靠CPU較近的快取記憶體上，而比較不常用的資料則放在較慢的主記憶體或是硬碟中，透過階層化的架構，我們可以在存取上節省較多的時間。

<!--more-->

1. 常見的記憶體階層：
   ![記憶體階層](https://i.imgur.com/gbMKd6V.png)

2. 快取記憶體：
   通常離CPU較近、儲存空間較小、速度較快，其上儲存了部份的資料供CPU讀寫，當CPU讀寫資料時，如果剛好需要的資料在快取記憶體上，我們稱為<font color="red">**快取命中(cache hit)**</font>，如果不在快取記憶體上，我們稱之為<font color="red">**快取未中(cache miss)**</font>，快取記憶體主要是基於程式碼的兩種特性:
   1. **時序局部性(Temporal locality)**:我們所使用的資料可能會在很近的時間內又被使用。
   2. **空間局部性(Spatial locality)**:我們所使用的資料附近的資料有很高的機率被使用。

3. 快取記憶體架構：
   快取記憶體架構除了定義它的整體大小外，我們還會設計他的組數(number of sets)跟每組裡面能儲存多少塊數(number of blocks)，依照不同的取捨，我們主要可以分成三大類：
   1. <font color="blue">**直接映射快取(direct mapped)**</font>: 將快取記憶體內每個記憶體區塊都視為一個set，當主記憶體的記憶體區塊要存入快存時，會存入對應的單一區塊。
   2. <font color="blue">**完全關聯(fully associative)**</font>:將快取記憶體內所有的記憶體區塊視為單一個set，當主記憶體的記憶體區塊要存入快存時，可以存在這個set中的任意一個區塊。
   3. <font color="blue">**集合關聯(set associative)**</font>:將快取記憶體內N個記憶體區塊視為一個set，當主記憶體的記憶體區塊要存入快存時，會存入對應set中的任意一個區塊，也被稱作<font color="blue">**N-way set associative**</font>。


   ![快取記憶體架構](https://i.imgur.com/J1gl7Gp.png)

4. 快取記憶體搜尋方式：
   我們通常會在快取記憶體中的每個block加入一個<font color="red">有效位元（valid bit）</font>用來得知block是否內含有效資料。
   
   為了在快取記憶體中搜尋我們要的記憶體區塊，我們會將記憶體位址作切割，一般切割如下：
   ![記憶體位址切割](https://i.imgur.com/rjAPlNL.png)
   * 塊位址(Block Address)：
     塊位址可以近一步切成：
     
     * 標籤欄(tag field)：
       標籤欄則是用來作比較，是否是有命中(hit)
       
     * 索引欄(index field)：
       索引欄用來決定是那一個要放在那一個set
     
   * 實體塊偏置(Block Offset)：
     用來選擇哪些是我們要從block讀寫的資料

5. 取代策略：
   當快取未中(cache miss)時，CPU會去搬我們要的資料放入快取之中，這時，哪個區塊(block)要被替換掉就是由設定的取代策略來決定，以下是常見的三種取代策略：
   * 隨機(Random):實現上最簡單。
  
   * 最近最少使用(Least recently used ,LRU)：理想上的方法，但極難實現，不過經常用模擬來比較不同的取代策略的優劣（先紀錄記憶體使用的情況，再來實現）。
   
   * 先進先出(First in, first out ,FIFO)：因為LRU極難實現，所以這個接近LRU概念的策略是將最舊的block取代。
   
   ![取代策略比較](https://i.imgur.com/CwJjYgf.png)

   
6. 寫入方式：
   * 當寫入命中(write hit)時，寫入方式依修改快取時有沒有同時寫進主記憶體分為兩種：
   
     * 完全寫入(Write-through)：每次快取記憶體內的區塊作修改時都會修改主記憶體，CPU等待這個動作的時間，稱為write stall，為了減少這段時間的浪費，通常會在加入寫入緩衝)write buffer)，來優化這段時間。

   
     * 回寫(Write-back)：快取記憶體內的區塊只有在要被取代時，才回寫到主記憶體。通常我們會在其中加入一個污染位元（dirty bit），用來紀錄這個區塊是否被修改過(dirty)或是從未被修改過(clean)。


     |特性|完全寫入(Write-through)|回寫(Write-back)|
     |:---:|:---:|:---:|
     |功耗|高|低|
     |頻寬需求|高|低|
     |實現難度|低|高|
     |使用區域|低層的記憶體層級（資料的一致性好維護）|處理器快取（因為所需的頻寬較低，避免記憶體讀寫的壅塞）|

   * 當寫入未中(write miss)發生時，寫入方式依寫入方式不同分為兩種：
   
     * write allocate：先將主記憶體中需要的那塊搬到快取，再做寫入。
     * No-write allocate：直接寫在主記憶體，不對快取作修改。
    
   寫入命中的兩種方法與寫入未中的兩種方法可以任意搭配，**<font color="red">通常Write-back搭配write allocate，Write-through搭配No-write allocate**</font>
   

---

### 2. 快取記憶體效能 (Cache performance)
1. 公式
   1. CPU執行時間：

      $$
      \begin{split}CPU\ execution\ time&=(CPU\ clock\ cycles + Memory\ stall\ cycles)\times{Clock\ cycle\ time}\\  
      &=IC\times(CPI_{exection}+{\frac{Misses}{Instruction}}\times{Miss\ penalty})\times{Clock\ cycle\ time}\\
      &=IC\times(CPI_{exection}+{\frac{Memory\ stall\ clock\ cycles}{Instruction}})\times{Clock\ cycle\ time}\\
      &=IC\times(CPI_{exection}+Miss\ rate\times{\frac{Memory\ accesses}{Instruction}}\times{Miss\ penalty})\times{Clock\ cycle\ time}\\
      \end{split}
      $$


      $$
      \begin{split}Memory\ stall\ cycles&=Number\ of\ misses\times{Miss\ penalty}\\
      &= IC \times{\frac{Misses}{Instruction}}\times{Miss\ penalty}\\
      &= IC \times{\frac{Memory\ accesses}{Instruction}}\times{Miss\ rate}\times{Miss\ penalty}
      \end{split}$$

      ${\frac{Memory\ stall\ cycles}{Instruction}}={\frac{Misses_{L1}}{Instruction}}\times{Hit\  time_{L2}}+{\frac{Misses_{L2}}{Instruction}}\times{Miss\ penalty_{L2}}$
 
   
      $IC = instruction\ count$
      $Memory\ stall\ cycles$指CPU暫停等待記憶體作存取的時間
   
   2. 平均記憶體存取時間：
      * one level cache
        $Average\ memory\ access\ time=Hit\ time+Miss\ rate\times{Miss\ penalty}$
      
      * two level cache
        $Average\ memory\ access\ time=Hit\ time_{L1}+Miss\ rate_{L1}\times{(Hit\ time_{L2}+Miss\ rate_{L2}\times{Miss\ penalty_{L2}})}$


      
   3. out-of-order CPU與記憶體：
      ${\frac{Memory\ stall\ cycles}{Instruction}}={\frac{Misses}{Instruction}}\times{(Total\ miss\ latency-overlapped\ miss\ latency)}$      

---

      
### 3. 六種基本的快取記憶體優化 (Six Basic Cache Optimizations)
1. 優化分類：
   快取記憶體的優化基本上可以分成下面三類：
   * 減少未中率(Reducing the miss rate)：
     * 增加block的大小
     * 增加cache的大小
     * 用較高的相連度associativity


   * 減少未中代價(Reducing the miss penalty)：
     * 使用多層的快取
     * 讓 read 有比 write 更高的優先度


   * 減少快取命中的時間(Reducing the time to hit in the cache)：
     * 避免在cache中索引時進行位址的轉譯


2. 未中分類(4C)：
   * 強制未中(compulsory):在CPU第一次用到記憶體的某個區塊時，不可能會放在快取記憶體中，這種未中也稱為cold-start misses 或是first-reference misses
     
   * 容量未中(capacity):如果快取的容量無法容納在程序執行期間所需要的所有區塊，則在使用上必須將某個儲存的區塊放棄，在另行提取。

   * 衝突未中(conflict):如果太多塊映射到同一個set中，在使用中可能會有一個set內部的塊都滿的情況，這時就需要將某個區塊放棄，

   * 一致性未中(coherency):因為在多個處理器的架構中，必須要保持多個cache內的資料一致所造成的未中

**<font color="blue">compulsory misses與cache的大小無關，capacity misses則隨著cache的大小增加而降低發生機率，conflict misses則是隨著相連度(associativity)增加而減少發生機率。**</font>

就經驗法則來說，小於128KB的cache，大小為N的direct mapped的miss rate 大約等於N/2的 two-way set-associative的miss rate

如果上一級的記憶體遠小於程序所需要的容量，則會有一大部分的時間用於在兩級之間移動資料，我們稱這種記憶體架構為thrash(會擺動的記憶體架構)

3. 優化方法詳述
   1. 增加block的大小來降低miss rate
      較大的block能利用Spatial locality的優勢，但同時也會增加miss penalty，而且還可能會增加conflict misses的機率。
      
      block的大小選擇有賴於低級記憶體(lower-level)的延遲(latency)和頻寬(bandwidth)
      

      | 特性 | 選擇 | 選擇原因 |
      | :--------: | :-------: | :--------: |
      | 高latency、大bandwidth | 大的 block    |  每次miss時，都能取得較多的資料且延遲的增加量沒那麼多  |
      | 低latency、小bandwidth | 小的 block    |  如果選擇大塊的block，每次miss時並不節省多少延遲時間，而且還可能會增加conflict misses |


   2. 增加cache的大小來降低miss rate:
      主要的缺點是可能會延長命中時間、增加成本跟功耗。
      
   3. 提高相聯度來降低miss rate
      * 經驗法則：
        * 對於特定大小的cache，從實際降低miss rate的功效來說，8-way set associative 和 directed mapped是一樣有效的
        * **2:1快取經驗法(2:1 cache rule of thumb)**：大小為$N$的直接映射跟 $N/2$ 的two-way set-associative的miss rate大致相同
      * 缺點：
        * 提高相聯度會提高cache設計的複雜度，進而使miss penalty增加

   4. 使用多層的快取來降低miss penalty
      * 以兩層的cache為例，記憶體的平均訪問時間如下：
          $Average\ memory\ access\ time=Hit\ time_{L1}+Miss\ rate_{L1}\times{(Hit\ time_{L2}+Miss\ rate_{L2}\times{Miss\ penalty_{L2}})}$
      * 名詞：
        為了避免混淆，在這邊將miss rate分為兩種：
        * 局部未中率(Local miss rate)：這個值代表這一級記憶體的miss rate，以第二級記憶體來說等於：$Miss\ penalty_{L2}$
        
        * 全局未中率(Global miss rate)：這個值代表從最高級記憶體一路這到這一級記憶體的miss rate，以第二級記憶體來說等於：${{Miss\ penalty_{L1}}\times{Miss\ penalty_{L2}}}$


        ![Miss rates vs cache size for multilevel caches](https://i.imgur.com/XoW0w2v.png)
        
        ![Relative execution time by second-level cache size](https://i.imgur.com/cmdGgeM.png)

      * 觀察：
        * 如果第二層的快取稍大於第一層的快取，則其局部未中率會很高(因為第一層的內容第二層都包含)，故一般設計傾向利用大的第二級快取
        * 如果第二層的快取遠大於第一層的快取，則全局未中率會接近於第二層的局部未中率
        * 第二層局部緩衝率，因為沒考慮第一層局部緩衝率所以並非好的評估指標，較好的評估指標是利用全局緩衝率

      * 第二層快取的儲存策略可依照第二層塊快取是否儲存第一層快取的內容可分為下列兩種
        * **(多層包含)multilevel inclusion**:第二級快取包含第一級快取的內容，較直覺，但是缺點是，如果兩級所使用block大小不同，在第二級快取miss 時，必須要作更多的工作(將與第二級踢掉的block有關的第一級block踢掉)，為了避免出現這個問題，一般我們都讓兩級的block大小相同。
          
        * **(多層互斥)multilevel exclusion**:第二級快取不包含第一級快取的內容，如果設計者僅能使用稍大於第一級快取的第二級快取，可以採用這種策略，這種策略會在第一級快取miss時且能在第二級找到所需的block，會將第一級跟第二級的block作互換，可以防止第二級的快取遭到浪費。

   5. 讓 read 有比 write 更高的優先度來降低miss penalty
      採用write-through時，我們通常會加入一個write buffer來改進其缺點，但是，這個write buffer反而會因為其可能包含read miss所需的更新值，而導致情況變得複雜。
      一個最簡單的解決方法就是，等到當read miss發生時等到write buffer清空才繼續動作，另一種較好的方法是：**當read miss發生時檢查write buffer是否含有所需的資料，直接讓read去讀buffer內的資料**，這種方法現在變成主流，也就是讓 read 有比 write 更高的優先度。
      在write-back中採取這個方式，也能夠獲得好處，因為一般程式的讀，緊接下來都是要對資料作操作，而寫通常都是操作完的最後一步，故如果讀優先於寫，能使處理器不需要在那邊等待。

   6. 避免在cache中索引時進行位址的轉譯，以縮短hit time：
      虛擬快取(vitual cache):利用虛擬位址的快取
      實體快取(physical cache):利用實體位址的快取
      
      完全的虛擬快取，其index和tag都是虛擬位址，可以減少要轉譯的時間，但為何人們不都使用完全的虛擬快取，其原因有三：
      1. **安全性**：虛擬位址要轉換成實體位址，必須檢查上面的保護資訊(以防程序修改別的程序的資料)。
         常見解決方法：在miss時，從TLB將保護資訊添加在cache中，在每次存取虛擬快取都進行確認。
         
      2. 映射問題I:**每次交換程序時，虛擬位址所對應的實體位址可能不同，所以必須刷新整個快取**。
         常見解決方法：多紀錄一個PID（process-identifie)，交換程序指刷掉PID是那些需要被取代的
         
      3. 映射問題II:**不同的虛擬位址可能對應到同一個實體位址，這些重複的位址稱為synonyms或aliases**。
         常見解決方法可以硬體或是軟體實現：
         * 硬體：稱為antialiasing，保證每一個block都有獨一無二的實體位址
         * 軟體：作法較為簡單，先決定虛擬位址和實體位址最低位元要有幾個完全相同，舉例來說，如果限制10個位元完全相同，則在$2^{10}$bytes cache中的每一個儲存單元絕不會對應到同一個位置的主記憶體，這項技術稱為page coloring。(可以想成分成1024種顏色，實體位址跟虛擬位址都是1024個一循環)
 
        現今的作法，主要是結合虛擬快取與實體快取的優點，其利用<font color="blue">**虛擬索引，實體標籤(virtually indexed, physically tagged)**</font>，利用虛擬的索引可以先至cache找尋所要entry，同時間可以利用virtual page number去TLB查詢實體的位址，接著，將實體位置當作標籤去確認cache entry是不是hit，這項技術主要是利用page的offset來當cache的index（因為這部份虛擬和實體都是完全相同的），可以減少轉譯的時間。缺點呢,如果cache是direct-mapped,cache的size會有限制,不能超過page offset(即page size的大小)的bit數。
        
        
        ![virtually indexed, physically tagged](https://i.imgur.com/dT4sxqA.png)

4. 結論 
        ![summary](https://i.imgur.com/FI2tdyF.png)


### 4. 虛擬記憶體 (Virtually Memory)
虛擬記憶體利用將實體記憶體切割成一塊塊的受保護的區塊(只有權限夠的程序才能讀取)，讓他們分別可以由不同的程序(process)所使用，在過去，沒有虛擬記憶體時，設計者必須自己將程式作一塊塊的分割(overlay)，執行時將一塊塊程式就載入主記憶體來進行執行，而現今，利用這項技術，系統可以自行管理主記憶體和次要記憶體間的內容。

![Virtually Memory](https://i.imgur.com/rqHo45p.png)

另外，因為虛擬記憶體將虛擬位置與記憶裝置作對應，使程式在執行時可以儲存在任何的記憶裝置中，不必一定要將程式存在特定的位址。(這現象稱為再定位(relocation))

事實上，虛擬記憶體的架構與記憶體架構類似，很多名詞甚至只是換個說法而已，舉例來說分頁(page)或分段(segment)就對應記憶體的block，page fault 和 address fault就對應miss。

在虛擬記憶體的技術中，CPU內處理的位址不再是實體位址，而是虛擬位址，正因為是虛擬位址，必須要有一個轉譯的步驟，才能將虛擬位址換成實體位址，近一步從對應的位址得到需要的資料，這個轉譯的步驟，我們稱為memory mapping 或是 address translation。

現今，電腦系統中，虛擬記憶體處理主要是DRAM和磁碟。

1. 差異


   |  | 快取記憶體(cache) | 虛擬記憶體(virtual memory) |
   | :--------: | :--------: | :--------: |
   | miss時的取代由何控制?     | 硬體     | 作業系統     |
   | 區塊的名稱?   |塊(block)| 分頁(page)或分段(segment)|
   | 大小怎決定?   |  設計者依設計考量決定    | CPU可以處理的位元地址，決定其大小     |



2. 分類：
   虛擬記憶體依照其區塊大小是否固定可以分成兩類：
   

   |  | 分頁(page) | 分段(segment) |
   | :-------- | :--------: | :--------: |
   | 大小     | 固定     | 變動     |
   | 替換區塊實現難度?     | 簡單     | 困難     |
   | 有效率的磁碟傳輸?  |是|不一定|
   | 應用程式設計者是否可見?     |   不可見   | 可能可見     |
   | 容許分頁框內部的碎片(internal fragmentation) 存在? | 允許   | 不允許     |
   |容許外部的碎片 容許外部的碎片(external fragmentation) 存在?|不允許|允許|
   |需要幾個字來紀錄|1 |2(一個紀錄段號一個紀錄偏移)|


   ![page&segment](https://i.imgur.com/C34QCWJ.png)
   
   由於分段的替換太難以實現，現今純的分段虛擬記憶體已經不多見了，有皆電腦系統則是結合兩者，提出一種paged segment的虛擬記憶體，其提供一組特定的page大小來獲得兩各的優點。


3. 虛擬記憶體特性：
   1. 如何將block放置主記憶體：
      因為虛擬記憶體如果miss會牽扯到極大的miss penalty(因為會去硬碟抓需要的資料)，故在資料區塊放置上主記憶體主要是採用完全關聯(fully associative)，來降低miss rate。
      
   2. 搜尋方式
      虛擬記憶體不論page或segment，在索引時都仰賴page number或segment number，不同的是，兩者的offset作用不同，segment的offset是拿來得到這個區段的最後位置的實體位址，而page的offset則僅僅是用來接在page number對應出來的實體位址之後(使位址完整)。
      ![virtual address to physical address](https://i.imgur.com/byvKLv6.png)
      分頁表(page table)：用來將虛擬位址換成實體位址的table
      
      Given a 32-bit virtual address,4 KB pages, and 4 bytes per page table entry (PTE), the size of the page table would be $(2^{32}/2^{12}) × 2^2 = 2^{22}$ or 4 MB.
      
      為了減少分頁表的大小，有些設計是利用是利用hash function來實現這個部份，有的甚至只需要實體分頁的數量，這種分頁表稱為inverted page table。


   3. 取代策略:
      最理想是以LRU來作取代，但實際上並不容易，故通常會加入use bit 和 reference bit 來得到趨近於LRU的效果，作法是：作業系統定時會清空這些bit而每次使用這些區塊時會對該區塊對應的bit作操作，故要取代時，則將這段時間最少使用的block取代掉。

   4. 寫入方式：
      因為下層是磁碟，故僅有一種方式：write-back且通常都有dirty bit的設計。



4. 快速位址轉譯 (Fast Address Translation)
有些系統允許分頁表被交換到次級記憶體，那麼虛實位址轉換可能要花非常長的時間，故有人提出了Translation Lookaside Buffer(TLB)或稱Translation Buffer(TB)用於快取一部分分頁表條目。
![TLB](https://i.imgur.com/SdzK8yb.png)

5. 分頁大小 (page size)
   * 傾向大的原因：
     * 可以有比較小的分頁表
     * 可以有比較低的miss rate
     * 傳輸的效率較好
     * 提高TLB的效率


   * 傾向小的原因：
     * 節省記憶空間

5. 總結 
![Virtually Memory whole map](https://imgur.com/DZFVe1a.png)

### 5. 虛擬記憶體的保護 (Protection of Virtually Memory)
在現今，程式多利用分時多工執行的背景下，要防止不同程序彼此間不干擾且維持正確的運作，需要作業系統設計者和架構設計者對此下些功夫。
作業系統設計者需要確保這些程序彼此間不互相干擾，而架構設計者則需要確保CPU可以儲存或載入程序的狀態。

保護程序不受其他程序干擾的最簡單方法就是—將程序的當前資訊存入硬碟之中，但其所耗的時間甚多。一個解決方法是將main memory作切割，讓多個程序能在主記憶體同時擁有空間儲存自己狀態，但這個解決方法需要架構設計者協助，使程序不能更改別的程序的狀態。

1. 程序保護(Protecting Process):
   程序可以分成不同的權限，依使用層級的不同來控管程序對記憶體的操作。
   
### 6.常見的迷思與陷阱(Fallacies and Pitfalls)     
1. 設計的記憶體空間太小：
   在設計架構時，最嚴重的設計問題莫過是設計的記憶體空間太小，太小的記憶體空間完全沒有辦法補救
   
2. 不要忽略作業系統對記憶體架構效能的影響
   


---
參考資料：
John L. Hennessy, David A. Patterson. "Computer Architecture: A Quantitative Approach, 5/e", Appendix B

https://www.ptt.cc/bbs/Grad-ProbAsk/M.1297706545.A.44B.html 
