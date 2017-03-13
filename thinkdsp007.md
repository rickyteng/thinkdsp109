
# 第六章 離散餘弦轉換 | Discrete Cosine Transform

這一章的主題是離散徐弦轉換 Discrete Cosine Transform (DCT)，這用在 MP3 與類似的壓縮音樂，JPEG 與類似的壓縮圖片，MPEG 類的影像格式。

DCT 在許多方面都與 DFT (Discrete Fourier Transform) 相似，之前我們有用來做一些分析。一旦我們學會 DCT 的原理，也就比較容易解釋 DFT。

我們依循以下步驟來解釋：

1. 我們從合成問題開始：給定一組頻率組成，與它們的振幅，我們該如何組成一個波？
2. 接下來，我們重寫合成問題的程式，改用 NumPy array。這一步會增加效能，也同時為下一步提供透視。
3. 我們會看分析問題：給定一個訊號與一組頻率，我們要如何找到每個頻率的振幅？我們會從一個概念比較簡單但比較慢的解法開始。
4. 最後，我們會用幾個線性代數的概念來找到更有效率的演算法。如果你已經熟悉線性代數，那很棒。總之我會解釋所需要的事情。

這章的程式碼在 chap06.ipynb (請參見 0.2節)，你也可以在以下網址看到。http://tinyurl.com/thinkdsp06

## 6.1 合成 | Synthesis

假設我給你一串振幅與一串頻率，然後要你組成一個訊號，它是這些頻率成份的總和。可使用 thinkdsp 模組裡的物件，這有一個簡單的方法執行這個操作，就叫做合成 synthesis：

```
def synthesize1(amps, fs, ts):
    components = [thinkdsp.CosSignal(freq, amp)
                  for amp, freq in zip(amps, fs)]
    signal = thinkdsp.SumSignal(*components)

    ys = signal.evaluate(ts)
    return ys
```

amps 是一串振幅，fs 是一串頻率，ts 是一串時間，它表明該評估訊號值的時間。

components 是一串 CosSignal 物件，每一個振幅與頻率的對應配對。SumSignal 代表這些頻率成份的總和。

最後，evaluate 計算每一個在 ts 裡的時間所對應的訊號值。

我們可以測一下這個函式：

```
    amps = np.array([0.6, 0.25, 0.1, 0.05])
    fs = [100, 200, 300, 400]
    framerate = 11025

    ts = np.linspace(0, 1, framerate)
    ys = synthesize1(amps, fs, ts)
    wave = thinkdsp.Wave(ys, framerate)
```

這個例子會組成一個訊號，它包含基頻 100Hz 與三個諧波(100Hz 是升G2)。它描繪出訊號在每秒 11025 幀的採樣下，1 秒鐘的波，且把它放到 Wave 物件裡。

## 6.2 使用陣列合成 | Synthesis with arrays

---
$ 
\begin{matrix}
  ~ & M & \begin{bmatrix}
    0.6 \\
    0.25 \\
    0.1 \\
    0.05
  \end{bmatrix} & = & amps \\
  \begin{matrix}
    \cdot \\
    \cdot \\
    t_n \\
    \cdot \\
    \cdot \\
  \end{matrix} & \begin{bmatrix}
    \cdot & \cdot & \cdot & \cdot \\
    \cdot & \cdot & \cdot & \cdot \\
    a & b & c & d \\
    \cdot & \cdot & \cdot & \cdot \\
    \cdot & \cdot & \cdot & \cdot
  \end{bmatrix}  & \begin{bmatrix}
    \cdot \\
    \cdot \\
    e \\
    \cdot \\
    \cdot 
  \end{bmatrix} & = & ys \\
  ~ & \begin{matrix}\cdot & f_k & \cdot & \cdot \end{matrix}   
\end{matrix}
$

圖6.1 用陣列合成
---

這裡有另個方式寫出 synthesize：

```
def synthesize2(amps, fs, ts):
    args = np.outer(ts, fs)
    M = np.cos(PI2 * args)
    ys = np.dot(M, amps)
    return ys
```

這函式看來非常不同，但它做的事情一樣。來看他怎麼辦到的：

1. np.outer 計算 ts 與 fs 的外積(outer product)，結果是一個陣列，有一個橫列的對應 ts 的元素，有一個直行對應 fs 的元素。每一個元素在陣列裡就是頻率與時間的乘積，$ f t $。
2. 我們將 args 乘上 $2 \pi $ 然後做 $ \cos $ 運算，所以結果的每個元素是 $(2 \pi f t)$。因為 ts 是沿著直行下來，每個直行都包含了在某特定頻率的 cosine 訊號，其由時間演算而來。
3. np.dot 把 M 的每個橫列與 amps 相乘(以元素)，然後再加起這些乘積。以現性代數的話來說，「矩陣 M 用向量 amps 乘」。以訊號的話來說，我們計算了頻率成份的加權總和。

圖 6.1 說明計算的架構。矩陣 M 的每個橫列對應 0.0 到 1.0 秒的時間，也就是 $ t_n $ 代表第 n 個橫列的時間。每個直行對應 100 到 400Hz 的頻率，也就是 $f_k$ 代表第 k 個直行的頻率。

範例中，我標示第 n 個橫列用 a 到 d，它的值 $a$ 會是 $\cos[2 \pi (100) t_n]$。

點積(dot product)的結果是 ys，它是一個向量每橫列是一個元素對應到 M。第 n 個元素標記為 e，是乘積的合：

$$ e = 0.6 a + 0.25 b + 0.1 c + 0.05 d $$

ys 其他的元素也是如此計算。所以 y 的每個元素是四個頻率的總和，在某個時間點計算，且乘上對應的振幅。這正是我們要的。

我們可以用前一節的程式來驗證兩個版本的 synthesize 會得到一樣的結果。

```
ys1 = synthesize1(amps, fs, ts)
ys2 = synthesize2(amps, fs, ts)
max(abs(ys1 - ys2))
```

ys1 與 ys2 最大的差異是 1e-13，我們認為這個誤差來自浮點數誤差。

寫下這個用線性代數的計算讓程式碼變小也變快。線性代數提供簡潔的記號給矩陣與向量。例如，我們可以將 synthesize 寫成這樣：

$$ M = \cos ( 2 \pi t \otimes f) $$
$$ y = M a $$

a 是振幅的向量，t 是時間的向量，f 是頻率的向量，$\otimes$ 是兩個向量外積的記號。

## 6.3 分析 | Analysis

現在我們準備好來解決分析問題。假設我給你一個波，告訴你它是一組給定頻率的 cosine 的和。你該如何找到每個頻率成份的指幅？換句話說，給你 `ys`、`ts`、`fs`，你能找回 `amps` 嗎？

在線性代數的語言裡，第一步跟合成問題一樣：我們計算 $ M = cos(2 \pi t \otimes f) $。然後要找到一個 $ a $ 使得 $ y = M a $。換句話說，我們要解出一個線性系統。NumPy 提供 `linalg.solve`，它正是做這事的。

它的程式碼像這樣：

```
def analyze1(ys, fs, ts):
    args = np.outer(ts, fs)
    M = np.cos(PI2 * args)
    amps = np.linalg.solve(M, ys)
    return amps
````
前面兩行用 $ts$ 與 $fs$ 建立 matrix $M$，然後 `np.linalg.solve` 計算 `amps`。

這裡有個小技巧，一般我們只在 matrix 是正方形的時候解系統的線性方程組，也就是說方程式(rows)的數量要等同末知數(unknowns)的數量。

在這例子中，我們只有四個頻率，但我們評估這個訊號有 11025 次。所以我們的方程式遠比未知數多。

一般來說，如果`ys`少於四個元素，我們無法只有四個頻率來分析它。

但在這個例子，我們知道`ys`只有四個頻率加總，所以我們可以用任意四個 wave array 的值來還原 `amps`。

為了簡化，我會使用訊號的前面四個採樣，用前一節的`ys`，`fs`，`ts`的值，來執行程式 `analyze1`：

```
n = len(fs)
amps2 = analyze1(ys[:n], fs, ts[:n])
```
`amps2` 會是 `[0.6 0.25 0.1 0.05]

這演算法有用，但是慢。解線性系統的時間會正比於 $ n^3 $，$n$ 是 $M$ 的行數。我們可以做得更好。

## 6.4 正交矩陣 | Orthogonal matrices

一個解線性系統的方法是利用反矩陣。$M$的反矩陣寫成 $M^{-1}$。它有個性質是 $M^{-1} M = I $，$I$ 是單位矩陣，它的對角線的元素全是 1，其他元素是 0。

所以，要解 $ y = Ma $，我們可以把兩邊都乘上 $M^{-1}$ 然後得到：
$$ M^{-1} y = M^{-1} M a $$
在右邊，我們把 $M^{-1} M$ 換成 $I$：
$$ M^{-1} y = I a $$
如果 $I$ 乘上向量 $a$ 會得到 $a$，所以：
$$ M^{-1} y = a $$
這暗示我們，如果能有效率地計算 $M^{-1}$，我們就可以用簡單的矩陣乘法(用 np.dot)來找到 $a$。這時間花費正比於 $n^2$，比 $n^3$ 好。

一般找反矩陣是慢，但有些情況下比較快。一個特殊情況下，如果 $M$ 是正交(orthogonal)，$M$ 的反矩陣就是 $M$ 的轉置矩陣，寫成 $M^T$。在 NumPy 中，轉置一個 array 是個常數時間的操作。它不是真的移動 array 的元素，取而代之的是建立一個視角，改變元素取用的方式。

一個矩陣要說是正交(orthogonal)，它的轉置矩陣要等同於反矩陣，也就是 $M^T = M^{-1}$，這暗示 $M^T M = I$，所以我們可以計算 $M^T M $ 來檢查是否一個矩陣是正交。

來看在 sysnthesize2 的矩陣長怎樣。在前一個例子裡， M 有 11025 列，所以用個小一點的例子會是個好主意。

```
def test1():
    amps = np.array([0.6, 0.25, 0.1, 0.05])
    N = 4.0
    time_unit = 0.001
    ts = np.arange(N) / N * time_unit
    max_freq = N / time_unit / 2
    fs = np.arange(N) / N * max_freq
    ys = synthesize2(amps, fs, ts)
```
`amps`我們之前看過，是振幅的向量表示。因為我們有四個頻率成份，我們會在訊號中的時間中取四個點。這樣 $M$ 會是正方形。

`ts` 也是等距時間間隔的時間取樣點的向量表示，範圍是 0 到 1 的時間單位。我選擇時間單位是 1ms。這是個可隨意的選擇，我們會看到選 1 分鐘算很慢就是了。

因為 frame rate 是每時間單位 N 個採樣，Nyquist 頻率會是 $ N / time_unit / 2 $，這例子是 2000 Hz，所以 `fs` 是 0 到 2000 Hz 等距頻率的向量表示。

有了這些 `ts` 與 `fs` 的值，矩陣 $M$ 會是：
```
[[ 1.     1.     1.     1.   ]
 [ 1.     0.707  0.    -0.707]
 [ 1.     0.    -1.    -0.   ]
 [ 1.    -0.707 -0.     0.707]]
```

你會發現 0.707 是 $\sqrt 2 /2$ 的近似值，它是 $ \cos{ \pi /4 }$，你也會發現這是對稱矩陣，這表示 (j, k) 元素會跟 (k, j)元素的值一樣。這表示， $M$ 就是自己的轉置矩陣，也就是 $M^T = M$。

但是，$M$不是正交，如果我們計算 $M^T M $ 會得到：
```
[[ 4.  1. -0.  1.]
 [ 1.  2.  1. -0.]
 [-0.  1.  2.  1.]
 [ 1. -0.  1.  2.]]
```
它不是單位矩陣。

## 6.5 DCT-IV

但是，如果我們小心選擇 `ts` 與 `fs`，我們可以讓 $M$ 成為正交。因為存在好幾個方法，所以也就有好幾個版本的 離散餘弦轉換 Discrete Cosine Transform (DCT)。

一個最簡單的版本是移動 `ts` 與 `fs` 半個單位，這個版本叫做 DCT-IV，IV 是羅馬數字，代表它是 DCT 八個版本的第四個。

現在來看改版過後的 test1：

```
def test2():
    amps = np.array([0.6, 0.25, 0.1, 0.05])
    N = 4.0
    ts = (0.5 + np.arange(N)) / N
    fs = (0.5 + np.arange(N)) / 2
    ys = synthesize2(amps, fs, ts)
```

如果你與前個版本比較，你會發現有兩個改變。第一個是 ts 與 fs 都加了 0.5。第二，我拿掉 time_units，如此簡化了 fs 的表示。

這樣會得到的 M 是

```
[[ 0.981  0.831  0.556  0.195]
 [ 0.831 -0.195 -0.981 -0.556]
 [ 0.556 -0.981  0.195  0.831]
 [ 0.195 -0.556  0.831 -0.981]]
```

然後 $ M^T M $ 是
```
[[ 2.  0.  0.  0.]
 [ 0.  2. -0.  0.]
 [ 0. -0.  2. -0.]
 [ 0.  0. -0.  2.]]
```

有些對角線以外的元素表示為 -0，這是浮點數表達式為很小的負數。這個矩陣接近為 $2I$，也就說 $M$ 幾乎是正交，只是他乘了個 2。對我們的要求來說，已經足夠好了。

因為 $M$ 是對稱，而且(幾乎)是正交，$M$ 的反矩陣就是 $ M/2 $。現在我們可以寫更有效庠的 `analyze`：

```
def analyze2(ys, fs, ts):
    args = np.outer(ts, fs)
    M = np.cos(PI2 * args)
    amps = np.dot(M, ys) / 2
    return amps
```

我們不使用 `np.linalg.solve`，只改用乘上 $M/2$。

把 `test2` 與 `analyze2` 合併，我們可以寫出 DCT-IV 的實作：

```
def dct_iv(ys):
    N = len(ys)
    ts = (0.5 + np.arange(N)) / N
    fs = (0.5 + np.arange(N)) / 2
    args = np.outer(ts, fs)
    M = np.cos(PI2 * args)
    amps = np.dot(M, ys) / 2
    return amps
```
ys 是 wave array。我們不需要傳入 ts 與 fs 當參數。dct_iv 可以依據 N (也就是 ys 的長度)來找出它們。

如果我們搞定它，那這個函式可以解決分析問題。也就是，給定一個 ys，它應該可以回復 amps。我們可以如此測試：

```
amps = np.array([0.6, 0.25, 0.1, 0.05])
N = 4.0
ts = (0.5 + np.arange(N)) / N
fs = (0.5 + np.arange(N)) / 2
ys = synthesize2(amps, fs, ts)
amps2 = dct_iv(ys)
max(abs(amps - amps2))
```

從 amps 開始，我們合成一個 wave array，然後使用 dct_iv 計算出 amps2。從 amps 與 amps2 中最大的差異約是 1e-16，我們可預期是因為浮點數誤差。

## 6.6 反DCT | Inverse DCT

最後要注意，analyze2 與 synthesize2 幾乎相同，其中差異只有 analyze2 有把結果除以 2。我們可以用這觀察來計算「反離散餘弦轉換 | inverse DCT」：

```
def inverse_dct_iv(amps):
    return dct_iv(amps) * 2
```
`inverse_dct_iv` 解決合成問題，它使用振幅的向量，然後回傳 wave array，也就是 ys。要測試它可用 amps 開始，使用 `inverse_dct_iv` 與 `dct_iv`，試試得到的與一開始的差別。

```
amps = [0.6, 0.25, 0.1, 0.05]
ys = inverse_dct_iv(amps)
amps2 = dct_iv(ys)
max(abs(amps - amps2))
```
一樣，得到最大差異約為 1e-16。

## DCT 類別 | The Dct class

---
![](http://greenteapress.com/thinkdsp/html/thinkdsp036.png)
--圖6.2 400 Hz 的三角訊號，取樣率 10 kHz

---

thinkdsp 提供 Dct 類別，它封裝 DCT，就跟 Spectrum 類別封裝 FFT 一樣。要得到 Dct 物件，可以用 Wave 呼叫 make_dct。

```
signal = thinkdsp.TriangleSignal(freq=400)
wave = signal.make_wave(duration=1.0, framerate=10000)
dct = wave.make_dct()
dct.plot()
```

這個三角波的 DCT 的結果，顯示在圖6.2。DCT 的值可以是正的或負的。負的 DCT 對應負 cosine。也等同於正 cosine 平移 180 度。

make_dct 使用 DCT-II，這是最常用的 DCT 形式，由 scipy.fftpack 提供。

```
import scipy.fftpack

# class Wave:
    def make_dct(self):
        N = len(self.ys)
        hs = scipy.fftpack.dct(self.ys, type=2)
        fs = (0.5 + np.arange(N)) / 2
        return Dct(hs, fs, self.framerate)
```

dct 計算完的結果放在 hs。對應的頻率如同 6.5節計算方式則放在 fs。然後兩者用來初始化 Dct 物件。

Dct 提供 make_wave，它進行「反DCT」操作。我們可以這樣試試：

```
wave2 = dct.make_wave()
max(abs(wave.ys-wave2.ys))
```
ys1 與 ys2 最大差異約 1e-16，我們預期這是浮點數誤差。

make_wave 使用 scipy.fftpack.idct：

```
# class Dct
    def make_wave(self):
        n = len(self.hs)
        ys = scipy.fftpack.idct(self.hs, type=2) / 2 / n
        return Wave(ys, framerate=self.framerate) 
```

預設「反DCT」沒有將結果正規化，所以我們必須將它除以 $2N$

## 6.8 練習

For the following exercises, I provide some starter code in chap06starter.ipynb. Solutions are in chap06soln.ipynb.

Exercise 1   In this chapter I claim that analyze1 takes time proportional to n3 and analyze2 takes time proportional to n2. To see if that’s true, run them on a range of input sizes and time them. In Jupyter, you can use the “magic command” %timeit.

If you plot run time versus input size on a log-log scale, you should get a straight line with slope 3 for analyze1 and slope 2 for analyze2.

You also might want to test dct_iv and scipy.fftpack.dct.

Exercise 2   One of the major applications of the DCT is compression for both sound and images. In its simplest form, DCT-based compression works like this:

1. Break a long signal into segments.
2. Compute the DCT of each segment.
3. Identify frequency components with amplitudes so low they are inaudible, and remove them. Store only the frequencies and amplitudes that remain.
4. To play back the signal, load the frequencies and amplitudes for each segment and apply the inverse DCT.

Implement a version of this algorithm and apply it to a recording of music or speech. How many components can you eliminate before the difference is perceptible?

In order to make this method practical, you need some way to store a sparse array; that is, an array where most of the elements are zero. NumPy provides several implementations of sparse arrays, which you can read about at http://docs.scipy.org/doc/scipy/reference/sparse.html.

Exercise 3   In the repository for this book you will find a Jupyter notebook called phase.ipynb that explores the effect of phase on sound perception. Read through this notebook and run the examples. Choose another segment of sound and run the same experiments. Can you find any general relationships between the phase structure of a sound and how we perceive it?


```python

```
