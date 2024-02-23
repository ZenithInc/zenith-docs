# 物理层概述 

物理层主要是将数据和比特流之间相互转换,关注的是硬件的物理特性。

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/gHGHX5risfyBYEkFdbHj.png" alt="futures"/>

## 信道 {id="channels"}

信道指的是往一个方向传送信息的媒体，一条通信电路包含一个接收信道和发送信道。那么一条线路，既要接收又要发送会不会有冲突呢？这也是物理层解决的事情，我们将信道分为三种：

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/fMvkaZXRqoVG97nFkyVw.png" alt="channel_categories"/>

## 信道复用技术 {id="channel-multiplexing"}

信道复用技术是一种将有限的通信资源（如频谱、传输介质等）有效地分配给多个用户或多个通信流的方法。它通过合理地将不同的通信信号分配到不同的信道上，从而实现多路复用，提高通信系统的容量和效率。

<img src="http://file-linker.oss-cn-hangzhou.aliyuncs.com/T5m2o66G34ZisnuTRXEk.png" alt="channel multiplexing" />

### 频分复用 {id="FDM"}

FDM 将频谱划分为多个不重叠的子信道，并将每个用户的信号分配到不同的子信道上。每个用户占用不同的频带，因此可以同时进行通信，互不干扰。

### 时分复用 {id="TDM"}

TDM 将时间划分为多个时隙，每个用户在不同的时隙中传输数据。通过快速地切换时隙，不同用户的数据可以在同一个时间段内传输，实现数据的复用。

### 统计时分复用 {id="STDM"}

STDM 是一种动态的时分复用技术，根据用户的实际通信需求来分配时隙。当某个用户需要传输数据时，会被分配一个或多个连续的时隙，而其他用户则会被暂时占用的时隙空闲。

### 波分复用 {id="WDM"}

WDM 利用光纤中不同波长的光信号进行复用，将多个信号同时传输在同一根光纤上。每个信号使用不同的波长，相互之间不会干扰。