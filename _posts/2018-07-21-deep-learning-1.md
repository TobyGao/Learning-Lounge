---
layout: article
title: deep learning 1
key: 20180721
tags: AI
---

深度學習 I
===

## 1. 感知器(perceptron)
### 1. 何謂感知器

什麼是感知器呢？感知器可以想成是二元的線性分類器，其上有多個訊號輸入，而輸出訊號輸出0或1，在神經網路之中，它可以被看作是一個神經元。


<!--more-->

### 2. 圖解
#### 1. 基礎型
![perceptron](https://i.imgur.com/1usAAcK.png){:width="40%"}

這張圖上描述了一個感知器，其y值為：


$$
\begin{equation}
  y  = \left\{
   \begin{array}{c}
   0\ \ (\ \omega_{1}x_{1}+\omega_{2}x_{2}\leq\theta\ )  \\
   1\ \ (\ \omega_{1}x_{1}+\omega_{2}x_{2}>\theta\ )  \\
   \end{array}
  \right.
\end{equation}
$$


其中$\omega$為權重(weight)，$\theta$為臨界值(threshold)
#### 2. 導入權重與偏權值
先將前面提到的$\theta$設為$-b$，再做移項可得：

$$
\begin{equation}
  y  = \left\{
   \begin{array}{c}
   0\ \ (\ b+\omega_{1}x_{1}+\omega_{2}x_{2}\leq0 )  \\
   1\ \ (\ b+\omega_{1}x_{1}+\omega_{2}x_{2}>0 )  \\
   \end{array}
  \right.
\end{equation}
$$


其中$b$成為偏權值(bias)

### 3. 實例
以下為以感知器做出AND電路的範例

```python
# coding: utf-8                                                                 
import numpy as np


def AND(x1, x2):
    x = np.array([x1, x2])
    w = np.array([0.5, 0.5])
    b = -0.7
    tmp = np.sum(w*x) + b 
    if tmp <= 0:
        return 0
    else:
        return 1

if __name__ == '__main__':
    for xs in [(0, 0), (1, 0), (0, 1), (1, 1)]:
        y = AND(xs[0], xs[1])
        print(str(xs) + " -> " + str(y))
```

### 4.多層感知器(multi-layer perceptron)
**單一感知器的極限在於僅能處理線性的分類問題(利用直線分類的問題)**，故我們可以將感知器層層交疊（層疊）來處理非線性的分類問題，下圖可以看出XOR就是一個無法透過一條直線來分類的非線性問題。

![xor](https://i.imgur.com/GssPJRT.png){:width="60%"}


#### 1.例子
- 程式碼

```python
# coding: utf-8                                                                 
from and_gate import AND 
from or_gate import OR
from nand_gate import NAND


def XOR(x1, x2):
    s1 = NAND(x1, x2) 
    s2 = OR(x1, x2) 
    y = AND(s1, s2) 
    return y

if __name__ == '__main__':
    for xs in [(0, 0), (1, 0), (0, 1), (1, 1)]:
        y = XOR(xs[0], xs[1])
        print(str(xs) + " -> " + str(y)
```

- 圖示：

![XOR](https://i.imgur.com/5en3KO4.png){:width="60%"}

## 2. 神經網路
### 1.術語
#### 1. 層（layer）
   層依其所在位置可以分為三類：
   - 輸入層(input layer)
   - 隱藏層(hidden layer)
   - 輸出層(output layer)

   ![neural network](https://i.imgur.com/03oVI4l.png){:width="60%"}

#### 2. 活化函數(activation function)與偏權值(bias)
   - 偏權值(bias)：我們可以將前面提到的偏權值加入到神經元之中，看成是1經過b權重。
   - 活化函數(activation function)：我們可以將先前感知器中的式子作整理形成：

$$
\begin{equation}
    a\ =\ b+\omega_{1}x_{1}+\omega_{2}x_{2}\\
     y\ =\ h(a) 
\end{equation}
$$

其中$h(\ )$稱為活化函數，這個函數用來決定如何活化輸入訊號總和的功能

   ![activation function](https://i.imgur.com/MQj6J1h.png){:width="60%"}
   
#### 3. 預處理(pre-processing)
   有時為了要達成提高識別效能、提高學習速度、提高識別速度...等原因，會對資料先作處理，此處理稱為pre-processing

### 2. 活化函數(activation function)
**一般都選用非線性函數，如果選用線性函數的話，則神經網路不管有幾層，能處理的還是只有線性分類的問題**

#### 1. step function :

$$
   \begin{equation}
   h(x)  = \left\{
         \begin{array}{c}
           0\ \ (\ x\leq\theta )  \\
           1\ \ (\ x>\theta )  \\
         \end{array}
   \right.
   \end{equation}
$$


   ![step function](https://i.imgur.com/2A2Acfc.png){:width="60%"}


#### 2. sigmoid function :

$$
\begin{equation}
     h(x)\ =\frac{1}{1+e^{-x}}
\end{equation}
$$
     
![sigmoid function](https://i.imgur.com/T1blhtD.gif){:width="60%"}

#### 3. ReLU( Rectified Linear Unit ) 
   \begin{equation}
      h(x)\ =max(0,x)
    \end{equation}

   ![ReLU](https://i.imgur.com/MO6NkQG.png){:width="60%"}

更多活化函數可見[[Day 32] Deep learning -- activation function](https://ithelp.ithome.com.tw/articles/10189085)

### 3. 神經網路
#### 1. 透過矩陣運算
   在神經網路中我們可以利用矩陣這個工具來作神經網路的運算(以下省略bias和activation function)，如下例：
   
   ![](https://i.imgur.com/U42KjuT.png){:width="40%"}
   
   其運算式為：

   $\ X(3) \bullet\ W(3,2)\ \ =\ Y(2)$
   
   **需要注意矩陣的行列是否匹配運算**
   
   如果把偏權值(bias)加入則是

   $\ A(1)\ =X(2)\ \bullet\ W(2,1)\ \ +\ B(1)$

   ![activation function](https://i.imgur.com/MQj6J1h.png){:width="60%"}
   
   - 例子
  
   ```python
   def init_network():
       network = {}
       network['W1'] = np.array([[0.1,0.3,0.5],[0.2,0.4,0.6]])
       network['b1'] = np.array([0.1,0.2,0.3])
       network['W2'] = np.array([[0.1,0.4],[0.2,0.5],[0.3,0.6]])
       network['b2'] = np.array([0.1,0.2])
       network['W3'] = np.array([[0.1,0.3],[0.2,0.4]])
       network['b3'] = np.array([0.1,0.2])
       
       return network
       
   def forward(network,x):
       W1,W2,W3 = network['W1'],network['W2'],network['W3']
       b1,b2,b3 = network['b1'],network['b2'],network['b3']
       
       a1 = np.dot(x,W1) + b1
       z1 = sigmoid(a1)
       a2 = np.dot(z1,W2) + b2
       z2 = sigmoid(a2)
       a3 = np.dot(z2,W3) + b3
       y = identity_fuction(a3)
       
       return y
    
    network = init_network()
    x = np.array([1.0,0.5])
    y = forward(newtwork,x)
   ```

#### 2. 輸出層的設計
   一般而言，回歸問題使用恆等函數（identity function），分類問題使用softmax
   - 分類問題：判斷資料屬於哪種類別的問題
   - 回歸問題：從資料預測（連續性）數值的問題 ex.預測年紀

   1. **identity function**
      僅僅將輸入輸出來

      $f(x) = x$

   2. **softmax**
      softmax會針對所有的輸入訊號來將其作正規化，他會使每一輸出項都為0~1之間的實數，且每一輸出項的總和為1，因為這個性質，我們可以將其視為各個$y_k$值的出現機率。
      
      值得注意的，這個式子並不會改變元素間的大小關係，原因是$y=e^x$是**單調遞增函數**，故如果我們不需要了解落在某分類的機率，我們可以簡單的去判斷$a_k$的大小

$$
\begin{equation}
    y_k\ =\frac{e^{a_k}}    {\displaystyle\sum_{i=1}^{n}e^{a_i}}
\end{equation}
$$ 
    
**執行softmax因為指數函數的成長速度極快，很有可能出現溢位，故可對其作改良**

$$
\begin{equation}
y_k\ 
=\frac{e^{a_k}}{\displaystyle\sum_{i=1}^{n}e^{a_i}}
=\frac{Ce^{a_k}}{C\displaystyle\sum_{i=1}^{n}e^{a_i}}
=\frac{e^{a_k+logC}}{\displaystyle\sum_{i=1}^{n}e^{a_i+logC}}
=\frac{e^{a_k+C'}}{\displaystyle\sum_{i=1}^{n}e^{a_i+C'}}
\end{equation}
$$
     
故我們可以將$C'=-max(a_i)$，這樣就能使運算中不會產生溢位。

#### 3. 批次輸入
   在使用神經網路時，為了節省分類的時間，有時會採用批次的技巧，所謂批次就是將資料分成一群一群，以一群為單位來作處理。


## 3.神經網路的學習
### 1. 資料
   機器學習使用的資料集可以分成訓練資料(training data)與測試資料testing data，利用訓練資料學習，在以測試資料評估學習模型的一般化能力。
   
   
### 2. 損失函數(loss function):
   用來評斷神經網路效能好壞的指標，之所以要引入這個函數是因為，當神經網路學習時，要找出盡量縮小損失函數值的參數。為了找出最小損失函數，必須計算參數的微分，將微分值當作線索，逐漸更新參數值。
   
   **之所以不使用辨識準確度來當作指標的原因是：幾乎在所有位置，微分值都會是0，使得參數無法更新，這個的原因是當你約略調整參數時，辨識準確度通常不會出現改變，必須要作比較大幅的更動時，辨識準確度才會有變化，且其變化並非是連續變化，而是呈不連續變化。**
   
   **上述的性質就類似階梯函數的表現，所以如果我們的activation function不是選用連續函數（且上面不能有微分值為0的部份），則學習也會因為同樣的理由，無法利用微分的概念來作學習。**
   
#### 1. 均方誤差(mean squared error)

$$
\begin{equation}
      E\ =\frac{1}{2}\displaystyle\sum_{k}^{}(y_k-t_k)^2
      \end{equation}
$$

$t_k$ 為訓練資料，$y_k$ 為神經網路的輸出，k代表資料的維度。
舉例來說：如果今天有一個數字辨識，


$$
\begin{equation}
t = [0,0,1,0,0,0,0,0,0,0]\\
y = [0.1,0.05,0.6,0.0,0.05,0.1,0.0,0.1,0.0,0.0]
\end{equation}
$$


則將其代入式子可得到其mean squared error
      
#### 2. 交叉熵誤差(cross entropy error)

$$
\begin{equation}
      E\ =-\displaystyle\sum_{k}^{}t_k\ logy_k
\end{equation}
$$

log以e為底，$y_k$ 為神經網路的輸出，$t_k$ 為訓練資料


**$-logy_k$定義為資訊量，當其$yk$發生的機率越大，則代表其攜帶的資訊越少（詳見資料壓縮領域的推導）**
      
      
**entropy 則是資訊量不確定的程度，如何定義呢？這件事情發生的機率乘上資訊量（就是期望值）**
      

### 3. 小批次學習 
   當我們訓練資料很大量的時候，我們有時會採用在其中隨機選取一些資料來代表整體的訓練資料，這種學習手法稱為小批次學習
   
   - 例子：
     如果我們損失函數採用交叉熵誤差則所有訓練資料的損失函數為
     \begin{equation}
     E\ =-\frac{1}{N}\displaystyle\sum_{n}^{}\displaystyle\sum_{k}^{}t_{nk}\ logy_{nk}
     \end{equation}
     
     $y_{nk}$為第n筆資料的第k個輸出值，$t_{nk}$則是對應的訓練資料。除以N是為了作正規化。
     
     因為訓練資料可能很大量所以我們可以隨機挑選適當的$N'$來代替N筆的訓練資料
     
### 4. 學習:
   神經網路的參數學習利用了微分的計算，利用微分的變化率來決定如何修正參數
   
#### 1. 微分(differentiation)：
電腦上的微分因為存在誤差所以有兩個地方需要注意
- 我們採用的是(x+h)和(x-h)的差分，利用中間差分可以減少誤差值
      \begin{equation}
         \frac{d f(x)}{d x } =\displaystyle{\lim_{h \to 0}}\frac{ f(x+h) - f(x-h)}{2h }
      \end{equation}
- h 我們如果真的用極小的數值，會發生數值上的誤差，故我們的ｈ設為$10^{-4}$
      

#### 2. 梯度(gradient)： 
將全部變數的偏微分當作向量處理的方式稱為梯度，梯度是種向量，每一個點上的梯度代表向哪個方向移動，函數值會變化的最快(梯度指向的方向，不一定就是極值的方向)

#### 3. 梯度法(gradient method)：
梯度法是從目前位置往梯度方向，只移動一定距離，再計算梯度，再移動一定距離，重複這些步驟的方法。
      
可依據要找出最大值或最小值而分成
- 梯度上升法(gradient ascent method):找最大值
- 梯度下降法(gradient descent method):找最小值
\begin{equation}
         x_0 = x_0 - \eta\frac{\partial f}{\partial x_0}\\
         x_1 = x_1 - \eta\frac{\partial f}{\partial x_1}\\
         .\\
         .\\
         .
\end{equation}
$\eta$ 代表更新的量，稱為學習率(learning rate)，因為這個值太大或太小都可能會使神經網路的參數落在不對的地方，故在神經網路的學習中，通常會一邊調整學習率，一邊確認是否正確的學習。
$\eta$ 太大會使其往大值擴散，太小則幾乎不更新就結束。
      
**一般將學習率稱為超參數[hyperparameter](https://en.wikipedia.org/wiki/Hyperparameter_(machine_learning))**，hyperparameter指的是那些不是透過學習來學到的參數，而是開發者手動去修正的參數。
      
#### 4. 學習重點：
在神經網路學習的過程中，必須要確認是否會發生過度學習(overfitting)的情況，過度學習指的是：只有訓練資料才能正確辨識，訓練資料以外的資料就無法辨識。

為了達到神經網路可以辨識多數情況，我們會在學習的過程中，定期以訓練資料及測試資料為對象，紀錄辨識準確度。這裡依照每個循環週期(epoch)，紀律訓練資料與測試資料的辨識準確率，為什麼採用1 epoch為週期來紀錄辨識準確率，因為如果再做每一筆batch時，就做這樣的動作會花費太多的時間，而且事實上我們也不需要那麼密集的頻率。
      
**1 epoch**指的是學習中用完訓練資料的次數。假設以100個小批次來學習10000個訓練資料，重複100準確率梯度下降法，就能看完全部的訓練資料。此時，100次 = 1 epoch。
      
### 5. 範例
- 以下是個簡單神經網路（隱藏層為一層），其採用SGD(準確率梯度下降法 stochastic gradient descent），這邊的準確率是指「準確隨機挑選」的意思（因為我們使用的資料都是利用隨機挑選出來的）

```python
class TwoLayerNet:

    def __init__(self, input_size, hidden_size, output_size, weight_init_std=0.01):
        # set up the initial weight and bias
        self.params = {}
        self.params['W1'] = weight_init_std * np.random.randn(input_size, hidden_size)
        self.params['b1'] = np.zeros(hidden_size)
        self.params['W2'] = weight_init_std * np.random.randn(hidden_size, output_size)
        self.params['b2'] = np.zeros(output_size)

    def predict(self, x):
        W1, W2 = self.params['W1'], self.params['W2']
        b1, b2 = self.params['b1'], self.params['b2']
    
        a1 = np.dot(x, W1) + b1
        z1 = sigmoid(a1)
        a2 = np.dot(z1, W2) + b2
        y = softmax(a2)
        
        return y
        
    def loss(self, x, t):
        y = self.predict(x)
        
        return cross_entropy_error(y, t)
    
    def accuracy(self, x, t):
        y = self.predict(x)
        y = np.argmax(y, axis=1)
        t = np.argmax(t, axis=1)
        
        accuracy = np.sum(y == t) / float(x.shape[0])
        return accuracy
        
    def gradient(self, x, t):
        loss_W = lambda W: self.loss(x, t)
        
        grads = {}
        grads['W1'] = numerical_gradient(loss_W, self.params['W1'])
        grads['b1'] = numerical_gradient(loss_W, self.params['b1'])
        grads['W2'] = numerical_gradient(loss_W, self.params['W2'])
        grads['b2'] = numerical_gradient(loss_W, self.params['b2'])
        
        return grads
```


```python
(x_train, t_train), (x_test, t_test) = load_mnist(normalize=True, one_hot_label=True)

network = TwoLayerNet(input_size=784, hidden_size=50, output_size=10)

#hyperparameter
iters_num = 10000  
train_size = x_train.shape[0]
batch_size = 100
learning_rate = 0.1

train_loss_list = []
train_acc_list = []
test_acc_list = []

iter_per_epoch = max(train_size / batch_size, 1)

for i in range(iters_num):
    # get batch of data
    batch_mask = np.random.choice(train_size, batch_size)
    x_batch = x_train[batch_mask]
    t_batch = t_train[batch_mask]
    
    # get gradien
    grad = network.gradient(x_batch, t_batch)
    
    # renew parameter(weight and bias)
    for key in ('W1', 'b1', 'W2', 'b2'):
        network.params[key] -= learning_rate * grad[key]
    
    #record learning process
    loss = network.loss(x_batch, t_batch)
    train_loss_list.append(loss)
    
    # in every epoch，we record accuracy
    if i % iter_per_epoch == 0:
        train_acc = network.accuracy(x_train, t_train)
        test_acc = network.accuracy(x_test, t_test)
        train_acc_list.append(train_acc)
        test_acc_list.append(test_acc)
        print("train acc, test acc | " + str(train_acc) + ", " + str(test_acc))
```

## 4. 誤差反向傳遞法(backward propagation)
### 1. 計算圖(computational graph)
利用圖表程呈現計算過程就稱為計算圖，而這邊我們所會使用的圖主要由下面節點構成：
1. 加法：

    ![](https://i.imgur.com/hfi3uX5.png){:width="40%"}

2. 乘法：

    ![](https://i.imgur.com/pAe5wb5.png){:width="40%"}

計算圖擁有局部性計算的性質，局部性計算是指針對每一個node我們僅僅需要考慮這個node的輸入，對於輸入是怎麼來的我們是不需要考慮的，這個性質就類似數位電路的邏輯閘一般。

**另外，計算圖利用反向傳播，可以有效的計算微分**，詳細的部份會在之後說明。

### 2. 連鎖律(chain rule)

![backward propagation](https://i.imgur.com/UiXqgSU.png){:width="60%"}

如果想知道輸入改變，輸出會作何變化，就必須使用微分這種工具，但整個神經網路包含多層的結構，如何能得知輸出與輸入的關係呢？（輸出對輸入的微分）這時就需要利用連鎖律這個性質了，對於每一個節點我們僅需要計算節點上的輸出對輸入的微分就能算出整體的輸出對輸入的微分。


$$
\begin{equation}
         \frac{\partial z}{\partial x}=\frac{\partial z}{\partial t}\frac{\partial t}{\partial x}\\
\end{equation}
$$

例子：

$$
\begin{equation}
         z = t^2\\
         t = x+y\\
         \frac{\partial z}{\partial x}=\frac{\partial z}{\partial t}\frac{\partial t}{\partial x}=2t \times 1 = 2(x+y)
\end{equation} 
$$


### 3. 反向傳播
#### 1. 基本運算層
##### 1. 加法
1. 數學式

$$
\begin{equation}
         z = x+y\\
         \frac{\partial z}{\partial x}=1\\
         \frac{\partial z}{\partial y}=1\\
\end{equation}
$$


2. 圖示

![add](https://i.imgur.com/nCpuw4v.png){:width="50%"}

3. 程式碼

```python
class AddLayer:
    def __init__(self):
        pass

    def forward(self, x, y):
        out = x + y

        return out

    def backward(self, dout):
        dx = dout * 1
        dy = dout * 1

        return dx, dy
```



##### 2. 乘法層
1. 數學式
\begin{equation}
         z = xy\\
         \frac{\partial z}{\partial x}=y\\
         \frac{\partial z}{\partial y}=x\\
\end{equation}

2. 圖示

![multiply](https://i.imgur.com/O1oYUXj.png){:width="50%"}

3. 程式碼

```python
class MulLayer:
    def __init__(self):
        self.x = None
        self.y = None

    def forward(self, x, y):
        self.x = x
        self.y = y                
        out = x * y

        return out

    def backward(self, dout):
        dx = dout * self.y
        dy = dout * self.x

        return dx, dy
```

#### 2. 活化函數層
##### 1. ReLU 層
1. 數學式

$$
\begin{equation}
  y  = \left\{
   \begin{array}{c}
   x\ \ (\ x>0 )  \\
   0\ \ (\ x \leq 0 )  \\
   \end{array}
  \right.
\end{equation}
$$


$$
\begin{equation}
  \frac{\partial y}{\partial x}  = \left\{
   \begin{array}{c}
   1\ \ (\ x>0 )  \\
   0\ \ (\ x \leq 0 )  \\
   \end{array}
  \right.
\end{equation}
$$

2. 程式碼

```python
class Relu:
    def __init__(self):
        self.mask = None

    def forward(self, x):
        self.mask = (x <= 0)
        out = x.copy()
        out[self.mask] = 0

        return out

    def backward(self, dout):
        dout[self.mask] = 0
        dx = dout

        return dx
```



##### 2. Sigmoid 層
1. 數學式

$$
\begin{equation}
  y\ =\frac{1}{1+e^{-x}}
\end{equation}
$$


$$
\begin{equation}
\begin{aligned}
  \frac{\partial y}{\partial x} &= \frac{e^{-x}}{(1+e^{-x})^2}\\
  &= \frac{1}{(1+e^{-x})}\times\frac{e^{-x}}{(1+e^{-x})} \\
  &= y \times (1-y)
\end{aligned}
\end{equation}
$$


2. 程式碼

```python
class Sigmoid:
    def __init__(self):
        self.out = None

    def forward(self, x):
        out = sigmoid(x)
        self.out = out
        return out

    def backward(self, dout):
        dx = dout * (1.0 - self.out) * self.out

        return dx
```

#### 3. Affine層
Affine Transformation是一種混合線性二維幾何轉換，而混合線性轉換包括：位移、放大縮小、旋轉...等操作，透過混合一個個不同的操作，來完成看似複雜的動作，神經網路其實就利用了Affine Transformation。


![](https://i.imgur.com/1rIwn5d.png){:width="50%"}


1.

$$
 \begin{equation}
   \frac{\partial L}{\partial \mathbf X} =\frac{\partial L}{\partial \mathbf Y}\bullet \mathbf W^T\\
   (N,2)\ \ \ \ (N,3)\ \ (3,2)
\end{equation}
$$

2.

$$
\begin{equation}
   \frac{\partial L}{\partial \mathbf W} =\mathbf X^T \bullet \frac{\partial L}{\partial \mathbf{Y}}\\
   (2,3)\ \ \ \ (2,N)\ \ (N,3)
\end{equation}
$$


3.

$$
\begin{equation}
   \frac{\partial L}{\partial \mathbf B} = \frac{\partial L}{\partial \mathbf{Y}}\\
   (2,3)\ \ \ \ (N,3)
\end{equation}
$$

$\frac{\partial L}{\partial \mathbf B}$是由$\frac{\partial L}{\partial \mathbf{Y}}$每一個column的合構成的。

原因是因為bias是會加到每一筆的資料上，因此，當反向傳播時，各個資料的反向傳播值，會集中在偏權值的元素中。

#### 4. Softmax-with-Loss層
詳細的推導與說明請見此書：[用Python進行深度學習的基礎理論實作](http://www.books.com.tw/products/0010761759)，這邊僅列出相關結果與程式碼

1. 圖示
![Softmax-with-Loss](https://i.imgur.com/Ai39Sfe.png =x400)

其中$t_i$是訓練資料

2. 程式碼

```python
class SoftmaxWithLoss:
    def __init__(self):
        self.loss = None # loss
        self.y = None # output of softmax
        self.t = None # training data(one-hot vector)

    def forward(self, x, t):
        self.t = t
        self.y = softmax(x)
        self.loss = cross_entropy_error(self.y, self.t)
        
        return self.loss

    def backward(self, dout=1):
        batch_size = self.t.shape[0]
        if self.t.size == self.y.size: 
            dx = (self.y - self.t) / batch_size
        else:
            dx = self.y.copy()
            dx[np.arange(batch_size), self.t] -= 1
            dx = dx / batch_size
        
        return dx
```
**我們可以觀察到其反向傳播剛好會形成$(y_i-t_i)$的漂亮形式，其實這並非巧合而是特地設計出來的**，而如果我們的輸出層採用**恆等函數**，而損失函數採用**均方誤差**，則也會得到相同的漂亮形式。

### 5. 範例
這裡的神經網路與3.5的部份大同小異，主要的差異是引入了backward propagation

```python
class TwoLayerNet:

    def __init__(self, input_size, hidden_size, output_size, weight_init_std = 0.01):
        # initialize weight
        self.params = {}
        self.params['W1'] = weight_init_std * np.random.randn(input_size, hidden_size)
        self.params['b1'] = np.zeros(hidden_size)
        self.params['W2'] = weight_init_std * np.random.randn(hidden_size, output_size) 
        self.params['b2'] = np.zeros(output_size)

        # generate each layer
        self.layers = OrderedDict()
        self.layers['Affine1'] = Affine(self.params['W1'], self.params['b1'])
        self.layers['Relu1'] = Relu()
        self.layers['Affine2'] = Affine(self.params['W2'], self.params['b2'])

        self.lastLayer = SoftmaxWithLoss()
        
    def predict(self, x):
        for layer in self.layers.values():
            x = layer.forward(x)
        
        return x
        
    # x:input data, t:traing data
    def loss(self, x, t):
        y = self.predict(x)
        return self.lastLayer.forward(y, t)
    
    def accuracy(self, x, t):
        y = self.predict(x)
        y = np.argmax(y, axis=1)
        if t.ndim != 1 : t = np.argmax(t, axis=1)
        
        accuracy = np.sum(y == t) / float(x.shape[0])
        return accuracy
        
    # x:input data, t:traing data
    def numerical_gradient(self, x, t):
        loss_W = lambda W: self.loss(x, t)
        
        grads = {}
        grads['W1'] = numerical_gradient(loss_W, self.params['W1'])
        grads['b1'] = numerical_gradient(loss_W, self.params['b1'])
        grads['W2'] = numerical_gradient(loss_W, self.params['W2'])
        grads['b2'] = numerical_gradient(loss_W, self.params['b2'])
        
        return grads
        
    def gradient(self, x, t):
        # forward
        self.loss(x, t)

        # backward
        dout = 1
        dout = self.lastLayer.backward(dout)
        
        layers = list(self.layers.values())
        layers.reverse()
        for layer in layers:
            dout = layer.backward(dout)

        # setting
        grads = {}
        grads['W1'], grads['b1'] = self.layers['Affine1'].dW, self.layers['Affine1'].db
        grads['W2'], grads['b2'] = self.layers['Affine2'].dW, self.layers['Affine2'].db

        return grads
```

需要特別注意的是：第20行程式碼，我們使用了OrderedDict，他是有序字典，可以記住字典內的元素順序，因此正向傳播時，只要按順序呼叫forward()即可，而反向傳播時，則只要反序呼叫backward()。


我們在實作時，可以用數值微分（梯度法）來檢查分析微分（反向傳播）的實作是否有問題。


---
參考資料：

[斎藤康毅,"Deep Learning：用Python進行深度學習的基礎理論實作.歐萊禮出版社, 17th Aug.2017"](http://www.books.com.tw/products/0010761759)


[The revenge of Perceptron! — Learning XOR with TensorFlow.](https://medium.com/@claude.coulombe/the-revenge-of-perceptron-learning-xor-with-tensorflow-eb52cbdf6c60)

[類神經網路跟Backpropagation一點筆記](http://terrence.logdown.com/posts/1132631-neural-networks-with-backpropagation-one-notes)


[我的資料科學之路](https://ithelp.ithome.com.tw/users/20103529/ironman/1212)


[Wikipedia contributors. "Softmax function." Wikipedia, The Free Encyclopedia. Wikipedia, The Free Encyclopedia, 31 May. 2018. Web. 18 Jun. 2018.](https://en.wikipedia.org/wiki/Softmax_function)

範例來源：

[deep-learning-from-scratch](https://github.com/oreilly-japan/deep-learning-from-scratch)
