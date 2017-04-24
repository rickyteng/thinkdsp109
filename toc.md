
# ThinkDSP

## 超譯前言

原文連結在此：http://greenteapress.com/thinkdsp/html/index.html

文件版本 1.0.9

Copyright 2012 Allen B. Downey

Permission is granted to copy, distribute, and/or modify this document under the terms of the Creative Commons Attribution-NonCommercial 3.0 Unported License, which is available at http://creativecommons.org/licenses/by-nc/3.0/.

這篇超譯是翻譯 Allen B. Downey 的 Think DSP: Digital Signal Processing in Python。這個人非常好心教了很多我喜愛的主題。超譯成中文給自己以及有興趣的人。本來我無法入門這個數位訊號處理的領域，有了這本免費電子書，得以進門走走看看了。

既然會說是超譯，就是不見得會完全照著翻，因為我也沒把握能完全把意思、精神、美感都翻譯出來。有時候偷懶就會少翻一點看來是閒聊的部份。

之前翻的時候是 0.10.0，有些章節原作者還未完成。2016年初時，與他聯絡時，他告訴我今年會完成。現在 html 版本已經是 1.0.9 了，於是我打算再重頭翻譯一遍。

以下開始超譯。

# 目錄

* [前言 | Preface](thinkdsp001.md)
  * 誰適合這本書
  * 程式碼的使用方法
  * 貢獻列表
* [聲音與訊號 | Sounds and Signals](thinkdsp002.md)
  * 週期訊號 | Periodic signals
  * 頻域分解 | Spectral decomposition
  * 訊號(類別) | Signals
  * 讀寫波形(檔) | Reading and writing Waves
  * 頻譜 | Spectrums
  * 波形物件 | Wave objects
  * 訊號物件 | Signal objects
  * 練習
* [諧波 | Harmonics](thinkdsp003.md)
  * 三角波 | Triangle waves
  * 方波 | Square waves
  * 混疊 | Aliasing
  * 計算頻譜 | Computing the spectrum
  * 練習
* [非週期訊號 | Non-periodic signals](thinkdsp004.md)
  * 線性唧頻 | Linear chirp
  * 指數唧頻 | Exponential chirp
  * 唧頻的頻譜 | Spectrum of a chirp
  * 時頻圖 | Spectrogram
  * Gabor 極限 | The Gabor limit
  * 洩漏 | Leakage
  * 窗 | Windowing
  * 實作時頻圖 | Implementing spectrograms
  * 練習
* [噪音 | Noise](thinkdsp005.md)
  * 不相關噪音 | Uncorrelated noise
  * 積分頻譜 | Integrated spectrum
  * 布朗噪音 | Brownian noise
  * 粉紅噪音 | Pink Noise
  * 高斯噪音 | Gaussian Noise
  * 練習
* [自相關 | Autocorrelation](thinkdsp006.md)
  * 相關 | Correlation
  * 序列相關 | Serial correlation
  * 自相關 | Autocorrelation
  * 週期訊號的自相關 | Autocorrelation of periodic signals
  * 相關視為點積 | Correlation as dot product
  * 使用 NumPy | Using NumPy
  * 練習
* [離散餘弦轉換 | Discrete cosine transform](thinkdsp007.md)
  * 合成 | Synthesis
  * 使用陣列合成 | Synthesis with arrays
  * 分析 | Analysis
  * 正交矩陣 | Orthogonal matrices
  * DCT-IV
  * 反離散餘弦轉換 | Inverse DCT
  * DCT 類別 | The Dct class
  * 練習
* [離散傅立葉轉換 | Discrete Fourier Transform](thinkdsp008.md)
  * 複數指數 | Complex exponentials
  * 複數訊號 | Complex signals
  * 合成的問題 | Ths synthesis problem
  * 用矩陣合成 | Synthesis with matrices
  * 分析的問題 | The analysis problem
  * 有效率地分析 | Efficient analysis
  * 離散傅立葉轉換 | DFT
  * DFT 是週期的 | The DFT is periodic
  * 真實訊號的 DFT | DFT of real signals
  * 練習
* [濾波器與摺積 | filtering and convolution](thinkdsp009.md)
  * 平滑化 | Smoothing
  * 卷積 | Convolution
  * 頻域 | The frequency domain
  * 卷積定理 | The Convolution Theorem
  * 高斯濾波器 | Gaussian filter
  * 有效率的摺積 | Efficient convolution
  * 有效率的自相關 | Efficient autocorrelation
  * 練習
* [微分與積分 | Differentiation and Integration](thinkdsp010.md)
  * 有限差 | Finite differences
  * 頻域 | The frequency domain
  * 微分 | Differentiation
  * 積分 | Integration
  * 累積和 | Cumulative sum
  * 積分噪音 | Integrating noise
  * 練習
* [線性非時變系統 | LTI system](thinkdsp011.md)
  * 訊號與系統 | Signals and systems
  * Windows and filter
  * 聲音響應 | Acoustic response
  * 系統與卷積 | Systems and convolution
  * 卷積定理的證明 | Proof of the Convolution Theorem
  * 練習
* [調變與取樣 | Modulation and sampling](thinkdsp012.md)
  * 與脈衝做卷積 | Convolution with impulses
  * 調幅 | Amplitude modulation
  * 取樣 | Sampling
  * 混疊 | Aliasing
  * 插值 | Interpolation
  * 總結 | Summary
  * 練習
* [索引 | index](index.md)


