
# ThinkDSP

## 超譯前言

原文連結在此：http://greenteapress.com/thinkdsp/html/index.html

文件版本 1.0.9

Copyright 2012 Allen B. Downey

Permission is granted to copy, distribute, and/or modify this document under the terms of the Creative Commons Attribution-NonCommercial 3.0 Unported License, which is available at http://creativecommons.org/licenses/by-nc/3.0/.

這篇超譯是翻譯 Allen B. Downey 的 Think DSP: Digital Signal Processing in Python。這個人非常好心教了很多我喜愛的主題。超譯成中文給自己以及有興趣的人。本來我無法入門這個數位訊號處理的領域，有了這本免費電子書，得以進門走走看看了。

既然會說是超譯，就是不見得會完全照著翻，因為我也沒把握能完全把意思、精神、美感都翻譯出來。有時候偷懶就會少翻一點看來是閒聊的部份。

之前翻的時候是 0.10.0，有些章節原作者還未完成。2016年初時，與他聯絡時，他告訴我今年會完成。現在 html 版本已經是 1.0.9 了，於是我打算再重頭翻譯一遍。

以下開始超譯

# 目錄

* [前言](thinkdsp001.md)
  * 誰適合這本書
  * 程式碼的使用方法
  * 貢獻列表
* 聲音與訊號
  * 週期訊號
  * 頻域分解
  * 訊號(類別)
  * 讀寫波形(檔)
  * 頻譜
  * 波形物件
  * 訊號物件
  * 習題
* 諧波
  * 三角波
  * 方波
  * Aliasing
  * 計算頻譜
  * 習題
* 非週期訊號
  * 線性唧頻 (Linear chirp)
  * 指數唧頻 (Exponential chirp)
  * 唧頻的頻譜
  * Spectrogram
  * Gabor 極限
  * Leakage
  * Windowing
  * 實作 spectrogram
  * 習題
* 噪音
  * 不相關噪音(Uncorrelated noise)
  * 積分頻譜(Integrated spectrum)
  * 布朗噪音(Brownian noise)
  * 粉紅噪音(Pink Noise)
  * 高斯噪音(Gaussian Noise)
  * 習題
* 自相關(Autocorrelation)
  * 相關(Correlation)
  * 序列相關(Serial correlation)
  * 自相關(Autocorrelation)
  * 週期訊號的自相關
  * 相關與點積(Correlation as dot product)
  * 使用 numpy
  * 習題
* 離散餘弦轉換(Discrete cosine transform)
  * 合成
  * 用 array 合成
  * 分析
  * 正交矩陣(Orthogonal matrices)
  * DCT-IV
  * 反離散餘弦轉換(Inverse DCT)
  * DCT 類別
  * 練習
* 離散傅立葉轉換(Discrete Fourier Transform)
  * 複數指數
  * 複數訊號
  * 合成的問題
  * 用矩陣合成
  * 分析的問題
  * 有效率地分析
  * DFT
  * DFT 是週期的 --
  * 真實訊號的 DFT --
  * 習題
* 濾波器與摺積(filtering and convolution)
  * 平滑化(smoothing)
  * 摺積(convolution)
  * 頻域(the frequency domain)
  * 摺積定理
  * 高斯濾波器(Gaussian filter)
  * 有效率的摺積
  * 有效率的自相關
  * 習題
* 微分與積分
  * 有限差(Finite differences)
  * 頻域(The frequency domain)
  * 微分
  * 積分
  * 累積和 --
  * 積分噪音 --
  * 習題
* 線性非時變(linear time-invariant)系統(縮寫 LTI system)
  * 訊號與系統
  * Windows and filter --
  * Acoustic response --
  * 系統與摺積
  * 摺積定理的證明
  * 習題
* Modulation and sampling
  * 與脈衝做摺積
  * Amplitude modulation
  * 取樣(Sampling)
  * Aliasing
  * Interpolation
  * 總結
  * 習題
* 索引


```python

```
