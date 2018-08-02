---
layout: article
title: 正規表示法
key: 20180801
tags: 自動機理論
---

正規表示法(regular expressions)是透過代數來描述語言的一種表達方式，
是在電腦科學被廣為使用的一個概念，此處將會對其作說明

<!--more-->

## 1. 定義
### 1. 基本定義
* 空集合$$\{  \}$$是正規語言
* 只包含一個空字串$$\{\epsilon\}$$的語言是正規語言
* 對任意$$a\in\Sigma$$(字母表集合)，$$\{a\}$$是正規語言
* 若A, B是正規語言，則再經過[正規語言的運算](#3-正規語言的運算)後依然是正規語言
* 除此之外都非正規語言


### 2. 符號定義
通常以一個英文字母來代表一個正規表示式，例如a，
而L()就表示一個正規表示式能表達出來的語言(language)，
故L(a)代表a這個正規表示式所能表示的語言。


### 3. 正規語言的運算
正規語言的運算分為
- 聯集(Union)
   Union (U): 就是把兩個language做聯集．
   舉例來說： 

   $$ \left \{1, 11, 111\right \} \cup \left \{0, 11, 000\right \} = \left \{1, 11, 111, 000\right \} $$


- 串接(Concatenation)
   把兩個language做連接，這邊的連接不是單純的將兩個正規語言所代表的集合串在一起，而是將元素中的元素作串接。
   舉例來說：
   語言L跟語言M作串接

   $$ L = \left \{ 10, 01 \right \} $$

   $$ M = \left \{ 11, 00 \right \} $$

   則

   $$ LM = \left \{ 1011, 1000, 0111, 0100 \right \} $$



- 克萊尼星號(Kleene Star)
   Kleene Star (*): * 表達空集合或是一個L以上的集合聯集的任何子集合。 

   $$ L* = \left \{ \epsilon \right \} \cup L \cup LL \cup LLL ... $$

   舉例來說：

   $$ L = \left \{01, 11\right \} $$
   $$ L*= \left \{\right \} or \left \{01, 11\right \} or \left \{0101, 0111, 1101, 1111\right \} .... $$

## 2. 引入自動機討論
### 1. 用$\epsilon-$NFA表示正規表示法
1. 基本定義
   * 邊線(arc)代表一個symbol，用以連接狀態(state)
   * 節點(node)代表automata的狀態


     我們可以透過下面將任意的正規表示法轉換成$\epsilon-NFA$


     ![re2eNFA](/Learning-Lounge/pic/automata/regularExpressions/RE2eNFA.png){:width="70%"}

2. 延伸
   - 聯集(Union)：
     利用symbol為空字串的arc來造成分支

     ![union](/Learning-Lounge/pic/automata/regularExpressions/union.png){:width="70%"}


   - 串接(Concatenation)：
     狀態以一symbol為空字串的arc連接

     ![concatenation](/Learning-Lounge/pic/automata/regularExpressions/concatenation.png){:width="70%"}



   - 克萊尼星號(Kleene Star)：
     形成迴圈，造成封閉(closure)

     ![Kleene Star](/Learning-Lounge/pic/automata/regularExpressions/KleeneStar.png){:width="70%"}


### 2. 用DFA表示正規表示法
1. 基本定義:
   * 邊線(arc)代表一個symbol\label，用以連接狀態(state)
   * 節點(node)代表automata的狀態
   * DFA所能接受的路線(path)就是我們的語言(language)

2. k-path:
   * 定義:

     k-path指的是在DFA上不經過狀態編號大於k的路徑，其中終點並不設限，可以是任意的狀態。

   * 例子:

     ![kpath example](/Learning-Lounge/pic/automata/regularExpressions/kpath_example.png){:width="70%"}
     

3. 歸納推導
   令$R_{ij}^k$是從狀態$i$到狀態$j$的k-paths的正規表示法，其初始條件如下：
   * $k=0$ 時，

    $$
    \begin{equation}
      R_{ij}^0  = \left\{
       \begin{array}{c}
       \sum labels_{arc_{ij }}\ \ (\ \exists arc_{ij},i \neq j )  \\
       \varnothing \ \ (\ \exists! arc_{ij} )  \\
       \epsilon \ \ (\ i=j )  \\
       \end{array}
      \right.
    \end{equation}
    $$

   接著可以由此推導出
   $R_{ij}^k = R_{ij}^{k-1} + R_{ik}^{k-1}(R_{kk}^{k-1})^*R_{kj}^{k-1}$
   
   $R_{ij}^{k-1}$: 從i到j不經過大於k狀態的path


   $R_{ik}^{k-1}$: 從i到k不經過大於k狀態的path


   $(R_{kk}^{k-1})^*$: 從k到k不經過大於k狀態的path(零到多次)


   $R_{kj}^{k-1}$: 從k到j不經過大於k狀態的path

4. 圖解

   ![Illustration Of Induction](/Learning-Lounge/pic/automata/regularExpressions/IllustrationOfInduction.png){:width="70%"}


---
參考資料：
[Wikipedia contributors. "Regular language." Wikipedia, The Free Encyclopedia. Wikipedia, The Free Encyclopedia, 22 Jun. 2018. Web. 1 Aug. 2018.](https://en.wikipedia.org/wiki/Regular_language)

[Wikipedia contributors. "Regular expression." Wikipedia, The Free Encyclopedia. Wikipedia, The Free Encyclopedia, 31 Jul. 2018. Web. 1 Aug. 2018.](https://en.wikipedia.org/wiki/Regular_expression)

[Stanford Online Course: Automata Theory](https://lagunita.stanford.edu/courses/course-v1:ComputerScience+Automata+SelfPaced/about)

[自動機理論-Automata筆記-第二週: Regular Expression](http://www.evanlin.com/moocs-coursera-automata-note2/)


圖片來源：

[Introduction to Automata and Complexity Theory](http://infolab.stanford.edu/~ullman/ialc/spr10/spr10.html#LECTURE%20NOTES)

