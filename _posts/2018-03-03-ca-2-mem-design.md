---
layout: article
title: 章節.2 記憶體階層架構設計
key: 20180303
tags: 計算機結構
---

### 1. 概論：
<!--可見[附錄.B 記憶體架構複習(Review of Memory Hierarchy)](https://hackmd.io/s/ryfiF-80Z)-->
可見[附錄.B 記憶體架構複習(Review of Memory Hierarchy)](/2017/10/12/ca-c-mem_review.html)

### 2. 十種進階的快取記憶體效能優化：
這裡探討了幾個快取記憶體優化的方向：
* 減少命中時間：
  下述兩種技術皆能夠減少功率損耗
  * 使用小且簡單的一級快取
  * 使用way prediction
    
<!--more-->
    
* 增加快取記憶體的頻寬：
  下述三種技術都會影響功率損耗
  * 將快取記憶體流水線化
  * 將快取記憶體分成數個bank
  * 使快取記憶體為nonblocking特性
    
* 減少未中代價：
  下述兩種技術不太會影響功率損耗
  * Critial Word First & Early Restart
  * 合併寫入緩衝

* 減少未中率：
  下述技術可以改善功率損耗
  * 編譯器優化


* 透過平行化減少未中代價或未中率：
  下述兩種技術通常會增加功率損耗
  * Hardware prefetching
  * compiler prefetching

1. 優化方法詳述：
   1. 使用小且簡單的一級快取(Using Small and Simple First-level Caches)
   
      由於技術的不斷進步，On chip的記憶體可以越做越大，但是L1的大小卻因為要和處理器的時脈相匹配，而一直都沒有什麼成長。高相聯度會增加hit time的時間，同時也會增加功率的消耗，一般來說，直接映射(direct mapped)略快於2-way associative，2-way associative速度大約是4-way associative的1.2倍，而4-way associative則是8-way associative的1.4倍。
      
      在實務上，有三種原因使設計者選用教高的相聯度：
      1. 處理器讀取快取的時間大於兩個時鐘週期，因此命中時間略長沒有嚴重的影響
      2. 將TLB排除在關鍵路徑外
      3. 多線程會導致conflict misses機率增加

   2. **Way Prediction**
      在cache中添加額外的bits，用來紀錄接下來可能的存取的way或是block，這個技術可以先設定在N-way associative中多工器可能要讀那一筆資料，如果猜對的話，可以有很短的hit time，猜錯的話則增加miss penalty，根據模擬，**I-cache預測的準確度高於D-cache**。
      
      另外有種技術是利用Way Prediction所添加的bit，這種技術稱為**way selection**，這種技術利用Prediction bit，來看要讀的資料位於那一個way，它僅僅供電給要讀這個way所需要電路，故能大量的節省功率，但是當預測錯誤時，它必須從頭存取資料，故這項技術通成都用於不需要高效能運算的嵌入式處理器。**way selection這個技術最大的缺點就是，流水化時非常難以實現**。
      
   3. 流水化快取記憶體的存取
      流水化快取記憶體，可以使記憶體的頻寬變大，同時系統的時脈可以提高，但其命中時間會變長。
      
      現今的pipelining cache通常都支援兩種模式，Burst Mode和Pipelining Mode，詳細部份可見[wiki](https://en.wikipedia.org/wiki/Pipeline_burst_cache)。
      
   4. non-blocking cache
      在out-of-order的執行下，如果快取記憶體支援non-blocking，則能在發生miss時，繼續存取其他的hit的資料，而不是直接stall起來。這些支援non-blocking的快取記憶體稱為 non-blocking cache 或 lookup-free cache。
      
      分類：
      1. 僅能在最多一筆miss下繼續存取
      2. 可以有多筆miss下繼續存取(hit under multiple or miss under miss)
      在高階處理器，一般兩種non-blocking都支援。
      
   5. Multibanked Caches
      如果將快取記憶體分成數個bank，可以使整體的頻寬大幅增加，而我們觀察到當記憶體的存取平均分散到每個bank時，整體的快取記憶體的頻寬可以最大化，為此，我們的作法稱為sequential interleaving，就是將資料依序放入不同的bank中。
      ![sequential interleaving](https://i.imgur.com/0WGXFm9.png)
      Multibanked Caches也是種可以節省能耗的作法。

   6. Critial Word First & Early Restart
      微處理器在至快取存取資料時，通常只需要一個word，故當它miss時，我們可以透過以下兩種方法來減少未中代價：
      - Critial Word First：
        先至主記憶體要求所需要的那一個word，當資料到達快取後立即發給CPU，繼續執行CPU的工作，同時將快取內需要更新的block做更新。
        
      - Early Restart：
        以正常的流程到記憶體提取資料，當把所需要的那個word寫入快取後，馬上將它傳給CPU，繼續執行CPU的工作。
      
      通常在大的block size會使用這兩種技術。
      
   7. 合併寫入緩衝（Merge Write Buffer）
      當要將快取資料寫回主記憶體，在現在常見的架構中，通常會讓CPU將資料寫到寫入緩衝(Write Buffer)，再將緩衝內的資料寫到主記憶體。
      
      此項技術，當寫入資料到緩衝時，檢查緩衝內部有無位置相鄰的資料，如果有則將其排在一起，如下圖所示，下圖上面部份是不使用這個技術的情況， 而下部則是使用這個技術的狀況，可以預想，這個技術可以減少CPU因為write buffer滿了所造成的stall的次數，另外，因為記憶體的性質，一次寫入大筆資料比一次寫一筆的速度要快的多。
      
      ![](https://i.imgur.com/4SOifJg.png)

     * 因為I/O通常也會被映射成實體位置，但是相鄰的實體位置不代表他們會是同樣的I/O裝置，故I/O裝置不能使用此技術，一般會在分頁上對這些記憶體做標注。

   8. 編譯器優化
      針對快取記憶體部份，下列為兩種編譯器優化的方法：
      - loop interchange
        有時在巢狀迴圈，其迴圈的取用順序並不依照其在記憶體排列的方式，這時只要經過迴圈做順序上的調整，就能大幅減少miss rate。
        ex.
        針對row major的儲存方式做優化(C語言為row major)
        優化前
        ```c
           for (j = 0; j < 100; j = j+1)
               for (i = 0; i < 5000; i = i+1)
                   x[i][j] = 2 * x[i][j];
        ```
        
        優化後
        ```c
           for (i = 0; i < 100; i = j+1)
               for (j = 0; j < 5000; j = i+1)
                   x[i][j] = 2 * x[i][j];
        ```
        **補充：row major & column major**
        ![](https://i.imgur.com/arG4h4H.png =400x500)

        
      - blocking
        如果今天遇到的是：資料運算同時間包含多個列和多個行的運算，則loop interchange就不能取得良好的功效，故blocking是把大陣列切成許多的小塊，讓這些小塊的在讀進快取後，能在被在至置換前被使用，這個技巧主要是利用tempal locality和spatial locality的特性。範例中的B我們稱為blocking factor。
        
        blocking除了可以減低miss rate，透過適當的blocking factor，它也可以降低load/store的數量。
        
        ex.
        優化前
        ```c
        for (i = 0; i < N; i = i+1)
            for (j = 0; j < N; j = j+1){
                r = 0;
                for (k = 0; k < N; k = k + 1)
                    r = r + y[i][k]*z[k][j];
                x[i][j] = r;
            }
        ```
        
        優化後
        ```c
           for (jj = 0; jj < N; jj = jj+B)
               for (kk = 0; kk < N; kk = kk+B)
                   for (i = 0; i < N; i = i+1)
                       for (j = jj; j < min(jj+B,N); j = j+1){
                           r = 0; 
                           for(k = kk; k < min(kk+B,N); k = k + 1)
                               r = r + y[i][k]*z[k][j];
                           x[i][j] = x[i][j] + r;
                      }
        ```
   9. Hardware prefetching
      透過硬體預先提取指令或是資料，預先讀取的部份可以放入快取記憶體或是其他的外部緩衝（讀取速度必須快於主記憶體的速度，否則就沒有意義了）。
      
      通常如果預取存放的地方不在快取記憶體中，我們稱為stream buffer
      ![stream buffer](https://i.imgur.com/706eOIV.png)

      
      一般來說，作法通常是：在發生miss時，先去檢查請求快是否在預取緩衝中，如若不然，則去主記憶體讀兩個區塊的資料：被請求的區塊以及下一個相鄰的區塊，被請求的區塊放入快取中該放的位置中，而預取的區塊則放入預取緩衝中。
      

   10. compiler prefetching
       * 依照儲存位置可分為：
         * **register prefetch**: 將資料載入到暫存器中
         * **cache prefetch**: 將資料載入到快取記憶體中
         
       * 依照是否會導致例外可分為：
         * **faulting** : 會導致例外
         * **nonfaulting** ：發生例外使會把這個指令變成no-op
         所以一般的存取指令（load/store）屬於faulting指令，一般來說，預取指令是被實作成nonfaulting，因為如果發生例外事件而去處理的話，可能會花更多的存取時間，而這類不影響程式運行中記憶體與暫存器內容且不會產生例外的prefetching，另外有一個術語，稱為**nonbinding prefetching**。
       
       利用compiler prefetching，prefetching必須要是nonblocking才有意義（如果是blocking，還不如不要預取，直接等miss再來搬資料），另外，必須注意到的是compiler prefetching會增加指令的數量，必須謹慎使用。

   
       * 例題： 
         For the code below, determine which accesses are likely to cause data cache misses. Next, insert prefetch instructions to reduce misses. Finally, calculate the number of prefetch instructions executed and the misses avoided by prefetching.
         
         Let’s assume we have an 8 KB direct-mapped data cache with 16-byte blocks, and it is a write-back cache that does write allocate. The elements of a and b are 8 bytes long since they are double-precision floating-point arrays. There are 3 rows and 100 columns for a and 101 rows and 3 columns for b. Let’s also assume they are not in the cache at the start of the program.
         ```c
         for (i = 0; i < 3; i = i+1)
             for (j = 0; j < 100; j = j+1)
                 a[i][j] = b[j][0] * b[j+1][0]
         ```
         
         Ans.
         
         我們可以將a,b陣列分開來觀察，a因為讀取的方式跟儲存的方式一致（row major），故其可以得到spatial locality的優點，又因為一個block是16個byte，故a在i為偶數的時候都為miss，而在i為奇數時都為hit，故a的miss總數為：$3\times(100/2)=150$次
         
         而b則是利用temporal locality帶來的優勢，其每次讀取時都有一個值是上次讀過得，故對$j從1到100$其miss數為$100+1$（第一次要多讀一個），又因為cache總共有${8K\ byte}\over{16\ byte}$$=512$個block，故i在作切換時並不會將區塊做替換，故b總共的miss數一樣等於101次。
         
         總共為251次的miss。
         
         這邊將code改寫為下面這個版本，其中預取七次後的資料，僅是因為範例，實際上必須要透過評估才知道預取多少次以後的資料才能達到最少的miss數。
         
         ```c
         for (j = 0; j < 100; j = j+1) {
             prefetch(b[j+7][0]);
             /* b(j,0) for 7 iterations later */
             prefetch(a[0][j+7]);
             /* a(0,j) for 7 iterations later */
             a[0][j] = b[j][0] * b[j+1][0];
         }
         for (i = 1; i < 3; i = i+1)
             for (j = 0; j < 100; j = j+1) {
                 prefetch(a[i][j+7]);
                 /* a(i,j) for +7 iterations */
                 a[i][j] = b[j][0] * b[j+1][0];
         }
         ```
         
         * 第一個迴圈：
           - a: a[0][0],a[0][1]...a[0][6],共$7/2=4$次miss，因為a的排列方式以及一個block可以放兩筆雙精度資料，故在除以2
           - b: b[0][0],b[1][0]...b[6][0],共$7$次miss


         * 第二個迴圈：
           - a: a[1][0],a[1][1]...a[1][6],共$7/2=4$次miss
           - a: a[2][0],a[2][1]...a[2][6],共$7/2=4$次miss

         總共的miss數為19次。
         
   11. 總結
       "+"表示改進，"-"表示會損害
       ![summary](https://i.imgur.com/YZ2EcYs.png)


### 3. 記憶體技術與組織：
記憶體的效能測量主要有兩個指標：延遲（latency）和頻寬（bandwidth），一般來說，延遲主要是快取記憶體的考量（miss plenty），而I/O跟多核心處理器的考量主要是頻寬的大小。

記憶體延遲可以在細分成兩個部份：
- **access time**: 指的是從讀記憶體的指令發送到資料送達的時間
- **cycle time**: 指的是無關的兩次記憶體請求最短的時間

1. SRAM 技術
SRAM的s為static，代表在電壓供給的情況下，不需要像DRAM一樣要不斷的刷新資料，且不像DRAM在讀過資料之後，要在對資料做寫入（否則電壓會變低，無法判斷原先的值），因為不需要刷新資料，**SRAM的cycle time可以非常的低且省電**，以下是常見的6T架構。
      ![SRAM](https://i.imgur.com/pWeBI7s.png)

以下擷取自維基百科：

SRAM的基本單元有3種狀態：standby (電路處於閒置), reading（讀取）與writing（寫入）. SRAM的讀取或寫入模式必須分別具有"readability"（可讀）與"write stability"（寫入穩定）。

- Standby
  如果字元線沒有被選為高電位，那麼作為控制用的M5與M6兩個電晶體處於斷路，把基本單元與位元線隔離。由M1 – M4組成的兩個反相器繼續保持其狀態，只要保持與高、低電位的連接。

- Reading
  假定儲存的內容為1,即在Q處的電位為高。讀取周期開始時，兩條位元線初始值為邏輯1,隨後字元線WL提高電位，使得兩個訪問控制電晶體M5與M6通路。第二步是保存在Q的值與位元線BL的初始值相同，所以BL保持邏輯1，而Q與BL的初始值不同，使得BL經由M1與M5放電而變成邏輯0（即Q的高電位使得電晶體M1通路）。在位元線BL一側，電晶體M4與M6通路，把位元線連接到VDD所代表的邏輯1（M4作為P溝道場效電晶體，由於柵極加了Q的低電位而M4通路）。如果儲存的內容為0,相反的電路狀態將會使BL為1而BL為0.只需要BL與BL有一個很小的電位差，讀取的放大電路將會辨識出哪條位元線是1哪條是0.敏感度越高，讀取速度越快。

- Writing
  寫入周期開始時，把要寫入的狀態加載到位元線。如果要寫入0，則設置BL為1且BL為0。隨後字元線WL加載為高電位，位元線的狀態被載入SRAM的基本單元。這是透過位元線輸入驅動能力設計的比基本單元相對較弱的電晶體更為強壯，使得位元線狀態可以覆蓋基本單元交叉耦合的反相器的以前的狀態。


2. DRAM技術

因為DRAM的容量大，故傳存取記憶體地址的作法是將其地址分兩次傳，先傳row方向的地址，再傳column方向的地址，分別透過RAS（Row Access Strobe）與CAS（Column Access Strobe）兩個訊號來控制。

![DRAM timing diagram](https://i.imgur.com/hlOnpqZ.jpg)

![DRAM](https://i.imgur.com/a1XfF3e.png)

DRAM的D為dynamic，其原因:為了儲存更多資料，DRAM不像SRAM使用那麼多個電晶體來儲存一個bit，DRAM一般使用一個電晶體來儲存一個bit，故當我們讀取資料時就會破壞它原先的電位，故DRAM在使用上：每次讀取完都要對它再做重寫入，因此，**DRAM的cycle time通常大於access time**。

- 為了解決長cycle time，DRAM引入了**multiple bank**的技術，透過這個技術，可以將cycle time一定程度的藏起來。

- DRAM由於儲存的電荷會自動放電（因為是透過電容儲存電荷），所以每段時間就必須更新(**refresh**)一次資料，更新的作法是每一條row同時做更新，而DRAM會去週期性的更新每一條row的資料，**refresh會使access time和cycle time是變動的**。

- **Amdahl's law**為經驗法則：要使系統平衡，記憶體的容量必須跟CPU的運算速度呈線性關係。

- DRAM在出售時，通常都放在**DIMM**(Dual inline memory modules)上出售，DIMM通常會有4～16DRAM並會在其上添加ECC。


3. 改進DRAM效能：
   1. 加入一個訊號，使同一個row不同column的資料的access time減少
      因為DRAM存取資料時所傳的地址會分成兩次來傳輸，故需要一個緩衝區來傳指定的row上的資料，這邊我們可以用一個時間訊號，允許重複去存取同個row不同column的位址。

   2. 加入時鍾訊號來做同步：
      為了不要花時間同步記憶體的控制器，我們可以加一個時鍾訊號(clock signal)，透clock signal，傳輸資料就沒有這個overhead，有做這種優化的DRAM我們稱為**SDRAM(Synchronous DRAM)**，通常SDRAM還有一個可變動的暫存器，用來儲存要傳輸的資料數。（可用於burst mode ），burst mode可以支援Critial Word First。

   3. 增加其DRAM的寬度

   4. 在CLK上緣跟下緣都傳輸資料，這個技術被稱為DDR(double data rate)

   為了要增進interleaving的效能和增進功率的管理，SDRAMs也引入了banks。當**DRR SDRAMs**包裝於DIMM上，他們的命名方式為了不被混淆，採用每秒傳多少bits來命名，而DIMM的名稱則是每秒傳多少MB的四捨五入值。
   ![](https://i.imgur.com/0evQkzm.png)
   
5. DRR消耗功率控制
DRR的功率消耗可分為動態功率（讀寫造成）和靜態功率（等待與漏電流造成），兩者的共同點皆是電壓，故為了減少功率損耗，一般的作法就是將操作電壓往下降，下表為不同世代的DRR技術的電壓比較。

   不同世代的DRR技術的電壓比較：

   |-------+------+------+------+------+ 
   | 技術  | DDR  | DDR2 | DDR3 | DDR4 |
   | ---- | :--: | :--: | :--: | :--: |
   | 電壓  | 2.5V | 1.8V | 1.5V | 1.2  |
   |-------+------+------+------+------+

現代的DDR有支援省電模式（power down mode），可以禁用SDRAM，只留下將存在的資料做刷新的功能。


4. Graphics Data RAMs(GDRAMs)
GDRAMs或GSDRAMs(Graphics or Graphics Synchronous DRAMs)是DRAMs的一支，其基於SDRAM的設計，但其針對GPU使用，有做了些特性上的修改：
   1. 其有較大的頻寬與界面
   2. 他的最高工作頻率通常更高
   3. 一般來說是直接接在GPU上
   4. GPU的資料通常不具有locality，故burst mode比較不重要
   5. 一樣用multibanks來增大頻寬


5. 快閃記憶體（Flash memory）
快閃記憶體是EEPROM(Electronically Erasable Programmable Read-Only Memory)的一支，他有以下的特性：
1. 他在寫入前必須先把block做清除，且最小讀寫單位必須是一個block
2. 他的資料不會因為斷電而消失
3. 每個block的讀寫次數是固定的
4. 價格：SDRAM>Flash memory>硬碟
5. 速度：SDRAM>Flash memory>硬碟

因為PMD的DRAM數量有限，故一般會使用Flash memory來做主記憶體的功能。

6. 增加記憶體可靠度：
記憶體的錯誤可分為兩種：一個不因電路結構所造成的，通常是一些宇宙輻射粒子打進電路使其出現錯誤的，一種則是因為製造過程中，使電路結構永久性改變而產生的錯誤，這兩類錯誤我們稱為soft errors(dynamic errors)和hard errors。

為了增加可靠度，學者提出了許多方法，以下是一些常見的方法：
1. 加入一些多餘的cell，使其發生錯誤時，可以透過程式來做取代
2. 加入錯誤更正碼（Error Correcting code）
3. 利用chipkill技術來提高大型系統的可靠度（intel 稱此技術為SDDC）


### 4. 資安保護：虛擬記憶體與虛擬機：
現今的電腦都支援多程式執行，為了達成多個程式分享硬體資源而不互相干擾，除了靠作業系統的保護，硬體架構的支援也不可少，以下是硬體架構為了達成保護必須實現的功能：
   * 提供至少兩個模式（使用者模式<user mode/operating system mode>＆最高權限者模式<kernal mode/supervisor mode>）
   * 使使用者程式可以取用一部分的處理器狀態，但能保護其不被修改
   * 提供能從使用者模式到最高權限者模式與從最高權限者模式到使用者模式的轉換機制
   * 提供限制程序存取某些記憶體的機制

而針對記憶體的保護，目前最流行的作法是在虛擬記憶體上加入一些保護的限制，保護資訊位於每個page table entry，其上可以決定哪些process可以讀寫哪些記憶體。

然而，當硬體將這些功能全數實現時，還是不能保證系統是安全無虞的，主要是因為現今的作業系統太龐大了，導致會有許多的漏洞出現，因此，出現了另一個流行的資安保護方式：虛擬機。
1. 虛擬機(virtual machine)
   虛擬機並非新的技術，但這個技術到最近才重新盛行了起來，主要原因有下：
   * 安全性和隔離性（執行時互不干擾）日益重要
   * 作業系統的安全漏洞和可靠度缺陷
   * 資料中心與雲端系統的盛行
   * 處理器的速度大量提昇，使得VM的執行速度到了還可接受的程度

   虛擬機的優點：
   * 方便管理硬體：VM允許許多OS獨立執行，且跑在同樣的硬體上，並允許執行一半的OS移到其他的機器上執行執行
   * 方便管理軟體：提供了一個抽象的界面，並可讓VM們跑不同版本的OS，一般的作法是，一部分跑最原本版本的OS，大部分跑目前穩定版本的OS，少部份跑測試版本的OS。


2. VMM（virtual machine monitor）
   VMM是VM的核心，亦有別名：hypervisor。被VMM用來執行一個或多個虛擬機器的電腦稱為主體機器（host machine），這些虛擬機器則稱為客體機器（guest machine），hypervisor有兩種類型：

   ![](https://i.imgur.com/8hdbjfZ.png)

   1. 虛擬化代價（Cost of processor virtualization）:
      虛擬化的代價主要取決於workload的類型，如果是計算導向的程式（processor-bound programs），則虛擬化代價接近於0，因為大部分的時間都直接跑在原生的機器上，故有相當於原先機器的執行速度。如果是輸入輸出密集的程式（I/O-intensive program），就要看它I/O花的時間比例、VM的架構，ISA實現方式等，了解其虛擬化代價，其中如果是輸入輸出導向的程式（I/O bound programs,I/O時間佔大多數執行時間的程式），則虛擬化代價可因CPU本身運作的沒效率而被掩蓋掉。

   2. 管理
      VMM必須
      * 要使其上的VM執行的像跑在一般的機器上
      * 且要防止guest machine去修改實際資源的分配。
      
      為了完成目的，它必須讓VM執行在user mode，而VMM則有更高的權限來分配資源。
      
   3. 提高虛擬機效能
      有三種方法可以幫助提高虛擬機效能
      - 降低處理器虛擬化的成本
      - 降低因為虛擬化而造成的中斷成本
      - 將中斷傳送給正確的VM，而不是利用VMM來處理中斷

      
3. Virtualize hardware
如果當初在設計ISA時有考慮VM的話，能使其執行VM的效能更佳，如果一個硬體能讓VM直接跑在上面，我們稱之為**Virtualizable hardware**。

4. 虛擬機對虛擬記憶體與I/O的影響
由於每個VM都有自己的page table，為了實踐這個功能，VMM把memory系統分成三各層級：
   - virtual memory
   - machine memory(real memory): 每台虛擬的
   - physical memory

故他們的工作情形就像下圖：
![](https://i.imgur.com/S4SXpxR.jpg)

過去大多數的VMM不實做這樣的架構（因為速度考量），它們維護了一個shadow page table，這個page table直接讓VM的virtual address space對應physical address space，透過檢測VM上的page table上的修改，可以知道VM的virtual address對應那一個physical address。


### 5. 實例
1. ARM Cortex-A8 記憶體架構
![ARM Cortex-A8](https://i.imgur.com/210DFYj.png)


2. Intel Core i7 記憶體架構(可略)
- 規格
![](https://i.imgur.com/zMkgX4F.png)
![](https://i.imgur.com/XkZhuLY.png)
其中所有的cache的block大小都是64 bytes，且都是nonblocking並允許多個未完成寫入。
![](https://i.imgur.com/fk1u4jm.jpg)
![](https://i.imgur.com/J0HzCUK.png)

- 分別解說
  - 指令部份
    1. virtual page number送到TLB去得對應的physical address
    2. 將page offset 送至I Cache取得指令，需要注意的是：
       1. 因為L1 I Cache的大小為32KB，故其index所需的bit數為
          $index = log_2(\frac{Cache\ size}{Block\ size\times{Set\ associativity}})=log_2(\frac{32K}{64\times4})=7$
       2. 因為每一個block是64-byte，故需要6個bit來決定block offset。
       3. 故應該要有$7+6=13bits$，但是page offset僅有12bit，故需要跟虛擬部分借一個bit，幸運的是，這並不會有問題，因為一位的虛擬位置差異會對應來兩個不同的物理位址，但指令就算在cache的兩個位置，其也不會不同。

    3.  從I TLB找到對應的phyical address，並檢查其是否有違反存取的情況
    4.  如果I TLB miss則到L2 TLB做搜尋，如果L2 TLB miss則去page table更新TLB。
    5.  L2 cache與L3 cache都是用physical index，故需要使用9個bit來當L2 cache的index，13個bit來當L3 cache的index
  - 資料部份
    資料部分需要注意的是，Data Cache的是8-way associative，故其index bits僅需要6個bits
    
  - 其他
    在i7的記憶體架構中，L2、L3支援prefetch
    在i7中，L1的write miss的策略是No-write allocate，故當miss時就直接寫入write buffer中，其行為感覺不到miss plenty。

### 6. 常見的迷思與陷阱(Fallacies and Pitfalls)
1. 不該利用A程式來評估B程式的快取效能


2. 我們無法透過模擬足夠的指令得到完全正確的記憶體架構的效能
   - 因為實際執行的指令長度可能遠大於我們模擬
   - locality的表現不是固定的
   - locality會隨著input做變化

3. cache可以減少latency，但不一定能增加頻寬，故在需要頻寬的情形，需要考量使用高頻寬的主記憶體

4. 不建議在非virtualizable的機器上實現VMM。







### X.模擬工具
1. [CACTI](http://www.hpl.hp.com/research/cacti/)
其為惠普(HP)實驗室所發展的一套整合cache和記憶體特性的開源程式碼模型，透過其軟體，我們可以評估我們的記憶體系統的功耗和存取時間。


---
參考資料：
John L. Hennessy, David A. Patterson. "Computer Architecture: A Quantitative Approach, 5/e", Chapter 2     

[Wikipedia contributors. "Cache prefetching." Wikipedia, The Free Encyclopedia. Wikipedia, The Free Encyclopedia, 4 Dec. 2017. Web. 13 Jan. 2018.](https://en.wikipedia.org/wiki/Cache_prefetching)

[Wikipedia contributors. "Static random-access memory." Wikipedia, The Free Encyclopedia. Wikipedia, The Free Encyclopedia, 6 Jan. 2018. Web. 14 Jan. 2018.](https://en.wikipedia.org/wiki/Static_random-access_memory)


[DRAM時脈圖(來自大同高工網頁)](http://elect.taivs.tp.edu.tw/class89b/word/topic_research/no7/004.htm)


[Wikipedia contributors. "Dynamic random-access memory." Wikipedia, The Free Encyclopedia. Wikipedia, The Free Encyclopedia, 12 Jan. 2018. Web. 14 Jan. 2018.](https://en.wikipedia.org/wiki/Dynamic_random-access_memory)


[Wikipedia contributors. "DIMM." Wikipedia, The Free Encyclopedia. Wikipedia, The Free Encyclopedia, 21 Sep. 2017. Web. 15 Jan. 2018.](https://en.wikipedia.org/wiki/DIMM)
