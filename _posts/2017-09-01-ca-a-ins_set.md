---
layout: article
title: 附錄.A 指令集準則
key: 20170901
tags: 計算機結構
---

### 1. 指令集的分類


<!--more-->

1.  如果依照CPU內部儲存的方式可分為下列幾種計算機架構：
    * stack architecture
    * accumulator architecture
    * register-memory architecture
    * register-register architecture
    * memory-memory architecture
    
    
    ![ four instruction set architecture](https://i.imgur.com/6EHBcE2.png)


    
2.  如果依照指令集分類可分為：
    * RISC
    * CISC

### 2. 記憶體定址（Memory addressing）
1.  解讀記憶體的方式(位元組存放順序 Endianness )：
    * 小端序 (Little Endian)
      記憶體的位置由LSB開始存
      
      ![Little Endian](https://upload.wikimedia.org/wikipedia/commons/thumb/e/ed/Little-Endian.svg/280px-Little-Endian.svg.png)

      
    * 大端序（Big Endian）
      記憶體的位置由MSB開始存
      
      ![Big Endian](https://upload.wikimedia.org/wikipedia/commons/thumb/5/54/Big-Endian.svg/420px-Big-Endian.svg.png)
      
2.  對齊
    在儲存資料時，記憶體需要對齊，這可以方便CPU取出資料
    
3.  定址模式(Addressing Mode)

    定址模式能得知哪些指令中的運算元要被執行、儲存的地方為何、處理的動作...
    
    常見的定址模式如下：
    
    ![定址模式](https://i.imgur.com/MnDPN5w.png)
    
    通常displacement 和 immediate addressing 使用機率最高

### 3. 運算元的型態與大小 (Type and Size of Operands)

1.  常見的型態：

    |-----------+-----------|
    |型態(types)|大小(sizes)|
    |---|---|
    |字元 (character)|8 bits|
    |半字 (half word)|16 bits|
    |字 (word)|32 bits|
    |整數 (integers)|32 bits|
    |單精度浮點數 (single-precision floating point)|32 bits|
    |雙精度浮點數 (double--precision floating point)|64 bits|
    |-----------+-----------|
    

2.  十進位格式(packed decimal or binary-coded decimal—)：
    用四個bits來表示一個十進位的數字，使用的原因為利用十進位格式的數字可以得到更精確的十進位數值
    
    #### **Example** 
    $(0.1)_{10} = (0.000110011\overline{0011})_{2}$
    
    
### 4.流程控制相關的指令(Instructions for Control Flow)
1.  分類：
    * Conditional branches
    * Jumps
    * Procedure calls
    * Procedure returns


2.  方法：
    常見作法為利用程式計數器(program counter - PC)作位移(displacement)來得到目標地址，此方法稱為<font color="red">**PC-relative**</font>，這個特性使得程式不管從那個位置載入都能順利執行，這種與載入位址無關的特性，我們稱之<font color="red">**position independence**</font>。
    
    在控制流程中目標位址應當是明確的，但在有些情況下，目標位址在編譯期間仍是未知的，故另外一個方法是動態的改變跳躍的位址，常用的實作方法為利用<font color="red">**暫存器間接定址模式(register indirect addressing mode)**</font>使目標位址可以變動。
    
    * 暫存器間接定址法模式對下列四種情況特別有效：
        * Case or Switch
        * Virtual functions or methods
        * High-order functions or function pointers
        * Dynamically shared library
    
3.  條件分支選項(Conditional Branch Options)
    因為大部分的控制流程所產生的改變都來自於條件分支(Conditional Branch)，故作條件分支判斷就成了很重要的問題，以下是常見的三種技術：
    
    |Name|Examples|How condition is tested Advantages|Disadvantages|
    |:---:|:---:|:---|:---|
    |Condition code (CC)| 80x86, ARM, PowerPC, SPARC, SuperH | Tests special bits set ALU operations, possibly under program control.| Sometimes condition is set for free. |CC is extra state. Condition codes constrain the ordering of instructions since they pass information from one instruction to a branch.|
    |Condition register|Alpha, MIPS|Tests arbitrary register with the result of a comparison.|Simple.|Uses up a register.|
    |Compare and branch|PA-RISC, VAX|Compare is part of the branch. Often compare is limited to subset.|One instruction rather than two for a branch.|May be too much work per instruction for pipelined execution.|

4.   程序調用選項(Procedure Invocation Options)
     程序的呼叫和返回包含了控制權轉移與狀態儲存(至少在程式呼叫上要將返回地址給儲存起來)，有些傳統的架構甚至會提供一套機制來儲存許多的暫存器，而現今的架構幾乎都是需要編譯器來產生對應的load/store來對每個暫存器作儲存/讀出。
     
     而在儲存狀態至暫存器中的方法，主要分成兩個流派：
     1. 呼叫者儲存(Caller save):
        呼叫者儲存在呼叫後仍需要的值
        
     2. 被呼叫者儲存(Callee save):
        被呼叫者儲存呼叫後仍需要的值，在程式結束前，將這些值在交還給呼叫者
        
     ![Caller save vs. Callee save](https://i.imgur.com/rwSeyMd.jpg )

     <font color="blue">*有些狀況使用Caller save較容易，像是兩個程序都能看到同一些全域變數，這樣Caller在Callee執行時也能夠看到那些變數，進一步作處理*</font>
     
### 5.編譯器規則(The Role of Compilers)
1. 現今編譯器:
   現今編譯器一般多由2~4個passes組成
   <font color="red">*一個pass:指讀且轉換整個程式的過程*</font>
   
   * 編譯器架構
   
   ![編譯器架構](https://imgur.com/eeYdNs9.png)
    

2. 優化序列排序問題(Phase-ordering problem):
   因為現今的編譯器多是一次針對一些部份作優化，故優化過程的前後序列排序就成了很重要的議題，
   優化序列排序問題主要就是決定—<font color="red">**以怎樣的順序作多少部份的優化能得到最大好處**</font>。
   
   * Example:
     在編譯器中有一種優化技術稱為：全域共同表達式精簡(global common subexpression elimination)，它利用找尋有無某些表達式含的子表達式計算後的值相同，並宣告一個變數將值儲存起來，這樣下次僅需要從變數中讀取這個值，不需要在重新計算。
     
     ```c
     a = b * c + g;
     d = b * c * e;
     ```
     
     可轉換成下列程式碼來進行優化
     
     ```c
     tmp = b * c;
     a = tmp + g;
     d = tmp * e;
     ```

     而這邊如果暫存器分配(register allocation)在這之後，且作的方式有誤，可能不會將tmp放在暫存器中，導致在讀取tmp時比原先計算所花的時間更多。
 ![](https://i.imgur.com/KF1J2E8.png)
    
   
3. 計算機架構設計者如何幫助編譯器設計者？
   * 提供有規律的指令集(Provide regularity):
     指令集中資料型態、操作和定址模式應該彼此正交，即不可特定某些暫存器僅能以某些特別的指令操作，或是指令之間有很強的相依性
   * 提供必要的，而不是複雜的指令(Provide primitives, not solutions)
   * 提供簡單的取捨判斷(Simplify trade-offs among alternatives)
   * 提供指令使某些在編譯期間已知的數值能以常數來表示(Provide instructions that bind the quantities known at compile time as con-
stants)

### 6.MIPS架構 (Microprocessor without Interlocked Pipeline Stages)

1. 特性

   * 為一 load-store 架構
   * 在實現管線化(pipeling)的部份極具效率，每個指令都是相同長度

2. 暫存器(Registers for MIPS)
   在MIPS中暫存器可分為GPRs(general-purpose register)與FPRs(floating-point registers)
   命名方式：
   * GPRs: R0, R1, R2 ... R32
   * FPRs: F0, F1, F2 ... F32
   FPR可以儲存雙精度或單精度的浮點數，如果儲存單精度浮點數的話，僅會使用一半的空間。
   
3. 資料型態(Data Types for MIPS)
    |型態(types)|大小(sizes)|
    |---|---|
    |位元組 (byte)|8 bits|
    |半字 (half word)|16 bits|
    |字 (word)|32 bits|
    |雙字 (double word)|64 bits|
    |單精度浮點數 (single-precision floating point)|32 bits|
    |雙精度浮點數 (double--precision floating point)|64 bits|


4. 資料轉移的定址模式(Addressing Modes for MIPS Data Transfers)
   MIPS所提供的資料定址僅有兩種模式
   1. 立即定址(immediate addressing)
   2. 位移定址(displacement addressing)
    
5. MIPS指令格式(MIPS Instruction Format)
   ![MIPS指令格式](https://imgur.com/Tk5ouIj.png)

6. MIPS操作(MIPS Operations)
   它含有4種類型的操作指令：
   * 存取(load-store)
   * 算術運算(ALU operations)
   * 分支與跳躍(branches and jumps)
   * 浮點數運算(floating-point operations)

7. MIPS指令解說
   * 符號說明：
     * ←的下標代表搬移幾個bit到左邊
     * 而文字下標的表示bit的區域，而A'\.\.'B代表從位置A到位置B區段的bits，在這邊MSB為0（e.g., $Regs[R3]_{56..63}$ 代表靠LSB最近的那個bytes）
     * Mem代表主記憶體
     * 上標代表複製這個值幾個bit（e.g., $0^{48}$代表有48bits的0）
     * \#\#代表連接兩個不同的區段
   * 讀存(Load-Store)：
     ![Load-Store1](https://i.imgur.com/nvtbjFo.png)
     ![Load-Store2](https://i.imgur.com/dw01fcP.png)

    * 控制流程(Control flow)：
![Control flow](https://i.imgur.com/uoZElRn.png)



### 7.常見的迷思與陷阱(Fallacies and Pitfalls)

1. 針對高階語言來設計指令集可能會導致指令集設計複雜度提昇，且有許多指令利用率低，且換了其他的語言可能因為相性不好，而使用不佳
2. 不存在有一個"**典型的**"程式，只要針對這個程式設計指令集就能對所有的程式作最佳化
3. 減少程式碼體積並不只有設計指令集一途，事實上改良編譯器也能大幅減少程式碼體積
4. 計算機架構就算有缺陷也不一定就不會成功，成功者不代表就是最好的架構(ex.x86)
5. 世界上沒有無瑕的計算機架構

### 8.現今的計算機架構發展趨勢：
* 記憶體位址兩倍(Address size doubles)
* Optimization of conditional branches via conditional execution
* Optimization of cache performance via prefetch
* Support for multimedia
* Faster floating-point operations


   


---
參考資料：
John L. Hennessy, David A. Patterson. "Computer Architecture: A Quantitative Approach, 5/e", Appendix B

http://slideplayer.com/slide/3336616/    
