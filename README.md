# FBMC physical layer: a primer
# 滤波器组多载波传输技术物理层：入门指南

## 概要：
FBMC技术引导了一种相对传统通信网络加强了的物理层，这是一种促成新概念，尤其是认知无线电(Cognitive Radio)的使能技术。
本文档的目的是提供一个FBMC的综述，并强调一些影响通信网络的特性。阅读本文的唯一前提要求是有关数字信号处理的基本知识，尤其是采样定理和快速傅里叶变换（FFT）以及有限长单位冲激响应(FIR)滤波器。更多关于已提到的技术的一些进展和与其相关的替代品以及更多复杂的方法都可以在该网站获取<http://www.ict-phydyas.org>。

本文以FFT在多载波通信的直接应用来展开，指出这种最简单化的途径的局限性，尤其是频谱泄露。然后，将展示FFT途径可以演进为一种能直接设计和实现滤波器组的途径。对每个数据块而言，时间窗被扩展到多个载波符号周期之外，并在时域上出现符号重叠。此时重叠是基于传统有效的单载波调制，其中的码间串扰是可以避免的，如果信道滤波器满足奈奎斯特准则的话。这个基本原理将可以应用到多载波传输上。至于应用方面，滤波器组途径仅仅是直接FFT途径的一个延伸，并且它可以被理解为一个扩展FFT。另一种方案有着更少的计算需求，就是所谓的”多相网络-快速傅里叶变换"(PPN-FFT)技术，该技术保持着FFT的点数但添加了一系列数字滤波器。
与必须保证所有载波的正交性的正交频分复用(OFDM)不同的是，FBMC只需要相邻子信道正交即可。事实上，OFDM是在给定的频率带宽上进行开拓，而FBMC则是用这给定的带宽将传输信道分解成一定数目的子信道。为了充分利用子信道带宽，子信道的调制器必须配合相邻正交性的约束，为此使用了偏移正交幅度调制(OQAM)。
滤波器组和OQAM调制的结合可得到最大的比特率，且不需要用于OFDM的保护时间(guard time)和循环前缀(cyclic prefix)。

传输信道的效应在子信道层级被补偿。子信道均衡器可以处理载波频偏、时偏以及相位和幅度的畸变，因此可以接纳异步用户。当FBMC工作突发传输状态时，突变的长度会被扩展以允许由滤波器脉冲响应引起的始末转变。如果允许一些临时频率泄漏的话，这些转变可能被削减，例如无论何时频隙(frequency gap)都会在相邻用户间出现。作为一种多载波方案，FBMC可得益于多天线系统，于是MIMO技术得以应用。而且既然借助于OQAM调制，那么在当前的多元化环境下为一些MIMO途径所作的配合也是必要的。

FBMC系统可能和OFDM系统共存。由于FBMC是OFDM的演化，所以一定程度的兼容是可以预见的。事实上，初始相位对双方可能是共同的，而且有效的双工模式应用也将被意识到。

在多用户环境下，一旦空闲子信道出现在两者之间，分配给用户的子信道或子信道组会被尽快分离。因此用户不必被异步化即可获取传输系统的通道。这是一个针对上行链路的关键设备——对由基站管理的网络或者未来的“机会通信”(opportunistic communications)都是如此。而在认知无线电领域，FBMC技术提供了这样一种可能：用相同的设备协同实现频率感测(spectrum sensing)和信息传输的功能。更重要的是，用户将享受到一种有可靠保证的频谱保护。

### *Contents:*
**Summary**

* 1)  作为多载波调制器的FFT
* 2)  FFT的滤波效应
* 3)  原型滤波器的设计——奈奎斯特准则
* 4)  扩展FFT来实现滤波器组
* 5)  用PPN-FFT降低计算复杂度
* 6)  OQAM调制
* 7)  传输通道效应
* 8)  子信道均衡器
* 9)  FBMC的突发传输
* 10) MIMO和FBMC结合
* 11) 与OFDM的兼容性
* 12) 网络中的FBMC

### 2.FFT的滤波效应
让我们假设**FFT**在串行传输样本的速率运行。
考察图**Fig.1**，可得**FFT**的输入端与输出端在k=0处的关系如下：
$$y_0(n)=\frac{1}{M} [x(n-M)+...+x(n-1)]=\frac{1}{M} \sum_{i=1}^M x(n-i)$$
这是一个低通线性相位FIR滤波器的方程，其M系数等于$1/M$。忽略常量延迟，则频率响应为
$$I(f)=\dfrac{sin\pi fM}{Msin\pi f}$$
如图**Fig.3**所示，频率坐标轴上的单元为$1/M$。
在相同条件下，**FFT**第k处的输出可表示为
$$y_k(n)=\frac{1}{M} \sum_{i=0}^{M-1} x(n-M+i) e^{\frac{-j2 \pi ki}{M}}$$

切换变量以$M-i$代换$i$,可得另一种表达式
$$y_k(n)=\frac{1}{M} \sum_{i=1}^M x(n-i) e^{\frac{j2 \pi ki}{M}}$$

![ Fig.3. FFT filter frequency response and coefficients in the frequency domain ]()

滤波器系数与$e^{\frac{j2 \pi ki}{M}}$相乘，可得关于频率响应中的$\frac{k}{M}$的一个频率偏移。当考虑到所有**FFT**的输出时，可得一组有M个滤波器的滤波器组，如图**Fig.4**所示，图中频率坐标轴的单元为$1/M$，即子载波间距。其正交条件通过零点表现出来：在整数倍于$1/M$的频率上，仅有一个滤波器的频率响应呈非零。
![ Fig.4. The FFT filter bank (frequency unit: sub-carrier spacing) ]()
一个FIR滤波器可由时域系数或频域系数来定义。这两组系数是等价的，并以离散傅里叶变换(**DFT**)相关联。回过去看滤波器组的第一个滤波器，如图**Fig.3**所示，它的冲激响应的DFT由一个单脉冲构成。事实上，这些频率系数是频率响应$I(f)$的样本，根据采样原理可知，$I(f)$可由这些样本通过插值公式来得到。

按滤波器组的术语，其组内第一个滤波器，即与零频率载波相关联的滤波器，被称作原型滤波器，因为其他滤波器是由它通过频率偏移推演得到的。显然在图**Fig.4**中$I(f)$是原型滤波器的频率响应，该原型滤波器的表现受到限制，尤其是带外抑制。为了减少带外纹波，有必要在时域增加系数的数量，在频域同样如此。那么，在时域，滤波器冲激响应长度超过了多载波符号周期$T$。在频域，额外的系数被插入到现有的系数之间，从而可以对滤波器频率响应进行更好地控制。

原型滤波器由重叠因子$K$表征，该因子是滤波器冲激响应长度$\Theta$与多载波符号周期$T$的比率。因子$K$也指在时域上重叠的多载波符号的数目。一般地，$K$是一个整数，而且在频域，它指被引入到**FFT**滤波器系数之间的频率系数的数目。

现在的问题是如何设计原型滤波器使得数据传输能以即使存在重叠也不产生码间串扰的方式进行。
