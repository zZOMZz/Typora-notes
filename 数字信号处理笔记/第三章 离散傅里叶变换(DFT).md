# 第三章 离散傅里叶变换(DFT)

- 离散傅里叶变化的定义
- 离散傅里叶变化的基本性质
- 频率域采样
- 离散傅里叶变换的应用举例(快速卷积, 谱分析)

## 3.1 离散傅里叶变换的定义

### 1. 定义

DFT变换的实质：<u>有限长</u>序列的傅里叶变换的<u>有限点</u>离散采样(时域和频域都是离散化的有限点长的序列

设x(n) 是一个长度为M的有限长序列, 则定义x(n)的N点离散傅里叶变换为:

![image-20221207172144687](C:\Users\zZOMZz\AppData\Roaming\Typora\typora-user-images\image-20221207172144687.png)

![image-20221207172351625](C:\Users\zZOMZz\AppData\Roaming\Typora\typora-user-images\image-20221207172351625.png)

逆变换是唯一的

**$$W_n = e^{\frac{-2j{\pi}}{N}}$$**

### 2. 物理意义:

![image-20221207183118481](C:\Users\zZOMZz\AppData\Roaming\Typora\typora-user-images\image-20221207183118481.png)



### 3. 周期性

![image-20221207183147513](C:\Users\zZOMZz\AppData\Roaming\Typora\typora-user-images\image-20221207183147513.png)



#### 4. 与其他变换的关系

![image-20221207184519444](C:\Users\zZOMZz\AppData\Roaming\Typora\typora-user-images\image-20221207184519444.png)





## 3.2 离散傅里叶变换的基本性质

### 1. 线性性质

$$y(n) = ax_1(n) + bx_2(n)$$, 取 N = max[N1, N2],则: 

$$Y(k) = DFT[y(n)] = aX_1(k) + bX_2(k) , 0 <= k <= N -1$$

### 2. 循环移位性质



### 3. 循环卷积定理

#### 3.1 时域循环卷积定理

如果$$X_1(k) = DFT[x_1(n)],  X_2(k) = DFT[x_2(n)], X(K) = X_1(k)X_2(k)$$, N = max[N1,N2], 长度不足的部分补零. 则 $$x(n) = x_1(n) * x_2(n)$$

#### 3.2 频域卷积定理

如果$$X(n) = x_1(n)x_2(n)$$, 则$$X(k) = \frac{1}{N}X_1(k)*X_2(k)$$



### 4. 有限长实序列DFT的共轭对称性



## 3.3 频率域采样函数

### 1. 频域采样定理

设x(n)的长度为M, 频域采样点数为M

- 若M > N ,则时域混叠
- 若M <= N, 则时域无混叠

![image-20221207212752591](C:\Users\zZOMZz\AppData\Roaming\Typora\typora-user-images\image-20221207212752591.png)



## 3.4 DFT的应用举例

### 1. 用DFT计算线性卷积

- 时域直接卷积法 : N*N次乘法，N(N-1)次加法

- 频域间接法计算： DFT有FFT算法，速度快