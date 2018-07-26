---
layout: article
title: 附錄.C 流水線：基本與進階概念
key: 20171215
tags: 計算機結構
---

* 當指令被保證完成時，我們稱之為committed

### 1. 概述
#### 1. 基本RISC的指令集概念 
   在RISC的指令集中有幾個重要的特性:
   * 所有的操作都是操作於暫存器
   * 唯二是對記憶體中的資料作操作的指令為load和store
   * 指令格式很少，絕大部分都是相同長度

<!--more-->
   RISC指令基本上分成三大類：
   * ALU 指令
   * Load 和 Store 
   * Branches 和 Jumps

#### 2. RISC指令集的基本實現
   1. Instruction fetch cycle(IF):
      1. Instruction Register = Mem[PC]
      2. NPC = PC + 4 


   2. Instruction decode/register fetch cycle(ID):
      1. 將Instruction作切割，並由切割結果到暫存器集取得對應的暫存器內的值
         A = Regs[rs]
         B = Regs[rt]
         
      2. 將Instruction內的immediate field作sign-extension
         Imm = sign-extended immediate field

   3. Execution/effective address cycle(EX):
      其根據op code 決定要作什麼動作：
      * 記憶體相關：
        ALUOutput = A + Imm 
        
      * Register-Register ALU 指令
        ALUOutput = A op B

      * Register-Immediate ALU 指令
        ALUOutput = A op Imm

      * Branch:
        這邊我們僅考慮BEQZ
        ALUOutput = NPC + (Imm << 2)
        Cond = (A == 0)

   4. Memory access(MEM):
      * 記憶體相關：
        LMD = Mem[ALUOutput] or 
        Mem[ALUOutput] = B

      * Branch:
        if (cond) PC = ALUOutput

   5. Write-back cycle(WB):
      * Register-Register ALU 指令
        Regs[rd] = ALUOutput

      * Register-Immediate ALU 指令
        Regs[rt] = ALUOutput
        
      * Load
        Regs[rt] = LMD
        
   ![RISC implementation](https://i.imgur.com/MAeVMve.png)
   ![Events on every pipe stage of the MIPS pipeline](https://i.imgur.com/JhVLwZ0.png)

   


     
     
#### 3. RISC指令集經典Pipeline 架構
   基於RISC指令集的基本實現的分割，在不同級間插入pipeline registers，讓每一級的pipe stage 可以同時間運作。
   但<font color="blue">**需要注意**：
   * 我們分離指令記憶體跟資料記憶體，避免結構衝突，使得流水線效率變差
   * 在 ID 和 WB 都會讀或存取暫存器，故為了避免衝突，我們將前半週期來作WB的寫入，後半週期來作ID的讀取
   * 因為PC每個週期都需要增加並儲存，且必須在 IF 就完成(因為要接著取下一個指令)，故我們在ID中加入一個加法器來避免結構衝突。
   </font>
    
![RISC pipeline architecture](https://i.imgur.com/UIgTWBE.png)
![pipeline cycles](https://i.imgur.com/Se23gUV.png)


#### 4. 基本效能討論
   * 名詞介紹
     * 吞吐量(throughput)
       throughput 指的是每一單位時間，會有幾個指令被完成

     * 延遲(latency)
       latency 指的是一個指令需要花多久的時間才會被完成
   
   流水線所影響的並不是延遲，而是吞吐量，流水線會因為加入切割的pipeline register使得延遲時間增加。如果時脈週期小於clock slew和latch的overhead，流水線化沒有意義。

### 2. 流水線衝突(Pipeline Hazards)
流水線衝突依照其不同的發生原因，可以分為下列三種：
   * 結構衝突(Structural Hazard)：由於資源有限的原因而產生的衝突
   * 資料衝突(Data Hazard)：因為所需資料與前面的指令有關而產生的衝突
   * 控制衝突(Control Hazard)：跟控制 PC 有關的指令( branchs or jumps )所產生的衝突

Pipeline Hazards最簡單的處理方式就是將流水線中會導致衝突的指令的停頓(stall)下來，直到前面的指令作到不會發生衝突的程度，再繼續流水線運作，停頓（stall）我們通常又稱它為pipeline bubble或bubble。

#### 1. 結構衝突(Structural Hazard):
   Structural Hazard最常發生的情形是functional unit並沒有很好的管線化，使得某些functional unit被不同的流水線區塊共用

#### 2. 資料衝突(Data Hazard):
   資料衝突也可以透過停頓來解決，不過停頓並非最好的方式，對現今的技術來講，一般採用<font color="red">**轉發( forwarding/bypassing/short-circuiting )**</font>的技術來解決，forwarding是將已經運算好但未到達WB的data拉線到前面的pipeline stage，使得前面的pipeline stage可以使用到正確的值。

   根據觀察，我們可以根據下列兩點來實作RISC經典的pipeline架構forwarding：
   1. ALU的運算如果仰賴EX/MEM或MEM/WB上的pipeline stage，則我們就將它轉發到ALU的輸入。


   2. 我們會在架構中加入一轉傳硬體( forwarding hardware )來幫助資料作forwarding，這一硬體會檢查是否有在ALU運算的暫存器，在先前的指令中有作更改，如果有的話，則作forwarding
   
   這邊需要注意，forwarding並不總能解決Data Hazard，事實上，還是有些Data Hazard會需要透過stall來解決，例如：
   
   ![Data Hazards Requiring Stalls](https://i.imgur.com/EzZD5UH.png)
   
   在此例 DSUB 並無法利用forwarding來解決，必須透過stall的方式解決，因此，我們必須在架構中加入一硬體，這個硬體稱為流水線內部鎖（pipeline interlock)，這個硬體會偵測是否發生衝突且指令是否需要停頓，如果需要停頓，則它會停頓後面的流水線直到狀況排除。


#### 3. 控制衝突(Control Hazard):
   控制衝突在MIPS流水線所造成的效能損耗遠比前面兩種Hazard嚴重(因為如果預測錯誤的話，流水線必須清空)，一般，MIPS pipeline架構在ID檢查出這個指令是branch，會將pipeline後面stall一個cycle(因為接著讀進來的指令通常是無用)。
   
   1. 減少pipeline branch penalties
      為了減少pipeline branch的代價，有幾種方式來處理pipeline中的branch：
      * 暫停或是清空流水線：
        優點：容易實現
        缺點：代價高

      * 總是猜是untaken:
        當猜錯時要將因為猜錯而讀進來的指令變成no-op，且必須將流水線上的狀態回復。

      * 總是猜是taken:
        當猜錯時要將因為猜錯而讀進來的指令變成no-op，且必須將流水線上的狀態回復。MIPS通常不使用，因為MIPS知道taken而跳躍的位址需要經過較多級的pipeline stage。
        
       總是猜是untaken/taken，兩者都能透過編譯器優化使其猜對的機率增加不少。
       
      * 延遲分支(delayed branch):
        延遲分支的概念是，因為branch到確定會不會taken會有一段時間（branch delay slot），我們利用compiler在那邊插上不管branch會不會taken必定會執行的指令。
        
        ![delayed branch](https://i.imgur.com/Apt3itC.png)

      a 是最好的選擇，如果沒有a這個情形，再考慮b或c。當branch大部分都是taken時，則考慮b，否則考慮c。

   2. Pipeline speedup:
      $\begin{split}Pipeline\ speedup &={\frac{pipeline\  depth}{1+Pipeline\ Stall\ cycles\ from\ branches}}\\
      &={\frac{pipeline\ depth}{1+Branch\ frequency\times Branch\ penalty}}
      \end{split}$
  
   
   3. 分支預測（Branch Prediction）
      當流水線越切越深，僅用上述的方法明顯是不足的，故則必須要加入分支預測來改善效能。
      
      * 靜態分支預測（static Branch Prediction）
        將要跑的程式先拿來作分析，得到其taken/untaken的比例，來作預測。
        
      * 動態分支預測（Dynamic Branch Prediction）
        基本的動態分支預測是利用branch-prediction-buffer(或稱ㄔbranch history table)，branch-prediction-buffer 是塊小塊的記憶體，它利用branch部份位址來作索引，其中儲存了下一個的分支預測，分支預測主要是透過n-bit prediction。
        常見的prediction是2-bit prediction，之所以不使用1-bit的原因，如果在大部份都是taken/un-taken的情況時，只要有一個不同，就會導致兩次的預測錯誤，故透過2-bit prediction來解決這個問題。
        ![2-bit prediction](https://i.imgur.com/tR4I3O5.png)
       




### 3. 實現管線流MIPS
#### 1. 實現控制 
   1. 檢測load interlocks:
      load與其後面指令的交互關係如下：
      
       ![](https://i.imgur.com/lO8DD8t.png)
       
      我們可以從中找到其發生load interlocks的邏輯：
      
       ![](https://i.imgur.com/18YQ124.png)


   2. 分支控制：
      ![](https://i.imgur.com/vyH9eok.png)

#### 2. 轉發架構
   ![](https://i.imgur.com/tr5VRvk.png)


#### 3. 分支設計
   在之前的架構中，因為branch是否執行，與其位置必須要在EX之後才能得到，故當如過預測錯誤時，會使下一個指令慢三個單位時間執行(IF,ID,EX)。
   
   ![RISC pipeline architecture](https://i.imgur.com/UIgTWBE.png)
   故多加一個加法器並移動其位置，使其架構如下:
   ![](https://i.imgur.com/eviQyb3.png)
   
   在這個架構下，如果預測錯誤，其正確指令僅會慢一個單位時間執行(IF)。
   故其增加的邏輯規則如下:
   
   

### 4. 造成管線流難實現的原因:
#### 1. 例外處理(Dealing with Exceptions):
   例外在這個領域的術語並不同一，一般來說是用exception這個字，但有些人也會利用interupt、fault，來表示相似的概念， 不過還是有一些常見用法，一般常見的說法如下:
   
   * I/O interrupt
   * page fault
   * floating point exception
    
   ![Exception event](https://i.imgur.com/pLwK7Qb.png)

   1. 例外的種類:
      1. 同步 v.s 非同步 (Synchronous versus asynchronous):
         如果每次例外事件都發生在同個地方則將之歸於Synchronous，如果是例外事件是因為外部給予的刺激，則事件不一定每次都發生在同一個地方，這類的例外事件我們將之歸於asynchronous

      2. 使用者請求 v.s 系統強制 (User requested versus coerced):
         如果例外事件是使用者直接要求而產生的則屬於User requested，如果不然則歸類於coerced

      3. 可遮蔽 v.s 不可遮蔽 (User maskable versus user nonmaskable):
         如果事件是可以由使用者屏蔽的則稱為User maskable，如若不然則歸於nonmaskable

      4. 指令中 v.s 指令間 (Within versus between instructions):
         如果事件發生於指令間則歸於between instructions， 如若不然則歸於Within instructions，通常Within instructions是Synchronous，因為是由該指令觸發例外事件，一般來說Within instructions比between instructions難實現，因為導致例外事件的指令必須停止且重新執行。

      5. 恢復 v.s 終結 (Resume versus terminate):
         如果事件發生會導致程式終結則我們稱之為terminating event，而若事件執行結束還能繼續執行則稱為resuming event

      ![category of Exceptions](https://i.imgur.com/HkRUfSi.png)


   2. 暫停與重新執行:
      <font color="red">**Synchronous且coerced的例外事件且發生在指令內部(with in instruction)且這個是必須恢復執行(resume)的指令是最難實現的。**</font>

      with in instruction 且 resume 指令是難實現的指令，以下我們探討將如何處理之。當發生例外時，如果該指令非分支指令，則我們依然以正常的情況去取接續的指令，如果是分支指令的話，則重新計算分支結果，並由結果去取接續的指令
      
      如果當例外發生時，流水線控制可以透過下列步驟來儲存好流水線的狀態:
      1. 插入一個陷阱指令(trap instruction)到下一個IF
      2. 將發生例外指令後的指令都改為null，使其不改動記憶體或暫存器，接著盡可能執行完前面的指令。
      3. 儲存需要重新執行的指令的PC
      4. 呼叫系統程式來接受這個例外

      但是delay slot的存在使得這個問題變得更難，當例外發生在delay slot，restart PC僅存入delay slot的位置是不夠的，我們還需要存入前面那個分支指令的位址。
      
      ![delay slot makes problem harder](https://i.imgur.com/KsEchgc.png)
      
   3. 精確中斷(Precise Interrupts):
      精確中斷必須滿足:
      * 流水線必須能夠暫停，且處理器狀態必須能夠儲存。
      * 在錯誤指令前執行的指令必須能夠被完成。
      * 在錯誤指令後的指令必須能夠重新啟動。

      支援Precise Interrupts的CPU一般會比不支援的要慢上許多，有些CPU會同時擁有兩種模式。
      
      
   4. MIPS 中的例外(Exceptions in MIPS) 
      
      ![Exceptions in MIPS pipeline stage](https://i.imgur.com/Q2JLull.png)


      現實中的例外可能會亂序發生，可能會有一條指令先行產生例外之後，排在前面的指令才發生錯誤，在這個情況中，雖然在流水線中後面的指令先發生例外，但實際上，在非流水線的處理器中，還是前面的流水線先發生例外。

      為了實現Precise exception，MIPS在發生例外時，並不直接處理他，而是將指令產生的所有例外都存在一個 **狀態向量(status vactor)** 中，這個status vactor會隨著這個指令往下傳遞，一旦這個向量中設定了例外，則關閉任何可能的數據值寫入，最後，在其進入WB時檢查其狀態向量，並依照其在非流水線的CPU的處理順序來做處理。



#### 2. 指令集的複雜程度(Instruction Set Complications):
   某些指令集較為複雜(CISC)，使得其流水線難以設計，而例外情形又更難處理，故現在常見的方式是透過實現將指令在分解成微指令(microinstructions)，再針對這些micro instructions來做流水線，透過較為簡單的流水線架構，例外情形能夠較好的處理。

### 5. 擴充浮點數運算功能至MIPS流水線
在這邊，我們添加下列四個functional unit至我們的MIPS實現中:
* Integer unit to handle Load/Store, Integer ALU, and Branch
* FP & Integer multiplier
* FP adder for FP add, subtract
* FP & Integer divider

![Extended MIPS Pipeline](https://i.imgur.com/JTqzcft.png)

我們在其中定義了兩個術語: 延遲(latency)、啟動間隔/重複間隔(initiation interval/repeat interval)，這邊**latencey的定義是，生成結果的指令與使用結果的指令間相隔的週期數**，舉例來說，在上圖的FP adder在進到計算的那個區段時要經過四個週期數，故下一個要使用FP adder結果的指令要跟這個指令至少再隔三個指令，故其latency為3。而initiation interval則是同類型的必須相隔的週期數，值得注意的是除法器得部份因為沒有做步驟切割，故其initiation interval為25

![](https://i.imgur.com/29Xb1Ff.png)

#### 1. Hazards 與 Forwarding
   擴充浮點數運算功能的版本有下列所述的問題:
   * Structural hazard: 除法部分並無真正的流水線化
   * 不同的流水線長度會導致:
      * 在同個時間可能寫入多餘一筆資料到暫存器，故可能要有額外的寫入埠
      * 指令不再依序到達WB，這使得**WAW harzard**變得可能，**但WAR是不可能發生的，因為讀總是在ID中完成**
   * 不相同的流水線長度與不同的functional units使得:
      * 亂序完成(out of order completion)變得可能
      * 例外的處理變得更加的複雜
   * 長延遲:
      **RAW** 所造成的stall頻率增高
      
   1. Structural hazard解決方式:
      由於實際上一個以上的資料寫入暫存區的機會並不大，故一般並不會增加寫入埠，所以一般的方式是透過以下兩種方法來處理:
      
      1. 在ID中追蹤指令對寫入埠的使用情形，當需要interlock時，則在指令要issue前進行stall，直到Structural hazard發生機會消除。

      2. 當會造成衝突指令進入MEM或是WB時(這個方式可能比較容易檢查)，停頓他們

   2. WAW解決方式:
      ![](https://i.imgur.com/BgDjoXj.png)
      
      1. 在ID中偵測 hazard，若會發生WAW，則在指令要issue前進行stall，直到WAW發生機會消除。


      2. 偵測到hazard，將前面指令無效化，而後面的指令正常執行，需要偵測指令的前後順序。


   3. 綜合之下的解決之道
      在ID時就檢測所有的harzard，當所有harzard發生的可能性消除了才發射(issue)指令。
      Forwarding依之前討論的那般。
      
#### 2. 維持精確例外:
   在亂序執行中，維持精確例外變成一個重要的課題，一般來說針對亂序執行的精確例外有下面三種方法:
   1. 忽略精確例外問題，這通常出現在早期的電腦或是超級電腦中
   2. buffer住指令的結果，直到在較早issue的指令都完成，缺點是需要大量的比較器和大的多工器
       * History file: 持續追蹤原先暫存器的值，當例外發生時，利用這個檔案回復其中的狀態
       * future file: 紀錄暫存器新的值，當所有在前面的指令都完成，將暫存器的狀態透過future file更新
   3. Stall 指令的issue直到先前的指令都被完成且沒有導致任何的例外
   
      
### 6. MIPS R4000 流水線      
因為快取存取的時間並不固定，故可以將memory access的步驟切得更細。這類更深的
pipelining 有時稱作superpipeling。

![](https://i.imgur.com/8Vb0IM4.png)

* IF—First half of instruction fetch; PC selection actually happens here,together with initiation of instruction cache access.
* IS—Second half of instruction fetch, complete instruction cache access.
* RF—Instruction decode and register fetch, hazard checking, and instruction cache hit detection.
* EX—Execution, which includes effective address calculation, ALU operation, and branch-target computation and condition evaluation.
* DF—Data fetch, first half of data cache access.
* DS—Second half of data fetch, completion of data cache access.
* TC—Tag check, to determine whether the data cache access hit.
* WB—Write-back for loads and register-register operations.

指令在IS可以讀取，但確認tag卻是在RF才完成，但因為不是做寫入，故不需要特定有一級來確認tag，而在MEM的部分，因為它是作寫入，所以一定要在tag被確認才能做寫入，故其有TC這一級，必須注意的是:其實可以在DS結束就Forward資料，但如果在TC這級發現tag是錯的，則必須將pipeline倒退一個週期，使其使用正確的資料，另外，由這架構我們可以發現到load delay變為兩個周期。




![](https://i.imgur.com/agDO7RX.png)

值得注意的是，基本的分支延遲變為三個周期，因為條件判斷是在EX這級，在R4000使用predicted-not-taken來減少兩個周期的分支延遲
      
![](https://i.imgur.com/cxA1cgv.png)


#### 1. 浮點數運算部分:
   其內部的functional unit:
   ![](https://i.imgur.com/yycdxan.png)
   
   各浮點數運算的pipe stages:
   ![](https://i.imgur.com/bvsR9xw.png)

   下列是常見的運算情形:
   ![](https://i.imgur.com/xo1elsl.png)
   ![](https://i.imgur.com/A7oGoig.png)
   ![](https://i.imgur.com/jsaAGNj.png)
   ![](https://i.imgur.com/3PPQGSt.png)





### 7. Scoreboard
指令的排程會極大地影響程式執行的效率，故一般都會對指令做些順序上的調動，依類型可分為:

* 靜態排程(static scheduling):也可稱為編譯器排程(compiler scheduling)

* 動態排程(dynamic scheduling):利用硬體支援，使他一邊執行一邊改變指令執行順序，其發展的原因如下:
    * 因為在編譯時期，不一定能得到指令真正相依性
    * 可以使編譯器的複雜度降低
    * 不會產生因為編譯器對某一台機器優化的程式碼，帶到另一台機器效率就變差的情況


這邊將會解說一套動態排程(dynamic scheduling)的方法，透過這各方法可以讓指令改變執行的順序，進一步減少停頓(stall)的情形發生，這套方法稱為Scoreboard。

![](https://i.imgur.com/6tUxAE6.png)



CDC 6600是第一個利用Scoreboard來完成out of order execution的機器，Scoreboard能使指令依序地issue(in-oder issue)，亂序地執行/commit。

1. 控制stage
   1. Issue:
      解碼指令，並檢查其上的結構跟WAW的hazard
   
   2. Read operands:
      如果沒有data harzard，則讀入operands

   3. Execution:
      依據指令來作執行

   4. Write result:
      檢查WAR，並完成執行

   ![](https://i.imgur.com/e4YWWty.png)

2. 範例:
   * Functional unit status:
     Busy—Indicates whether the unit is busy or not
     Op—Operation to perform in the unit (e.g., + or –)
     Fi—Destination register
     Fj, Fk—Source-register numbers
     Qj, Qk—Functional units producing source registers Fj, Fk
     Rj, Rk—Flags indicating when Fj, Fk are ready
     
     [範例可參照此投影片](https://drive.google.com/file/d/1BA-ouuacIZFNdxAYLB1iwSEgMmgKNqvC/view?usp=sharing)
   
### 8.常見的迷思與陷阱(Fallacies and Pitfalls)  
1. 意料外的執行序列可能導致意料外的hazards
2. 全面的流水化，可能會影響設計的其他方面，降低整體系統的性價比
3. 不該依據未經優化的程式碼來評估動態或靜態排程:因為實際上使用者在使用程式之前，一定會對程式做某種程度的優化。

---

參考資料：
John L. Hennessy, David A. Patterson. "Computer Architecture: A Quantitative Approach, 5/e", Appendix B

許雅三教授, "高等計算機結構課程投影片", 國立清華大學
