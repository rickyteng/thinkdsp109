
# 第七章 離散傅立葉轉換 | Discrete Fourier Transform

我們已經在第一章用過離散傅立葉轉換 Dsicrete Fourier Transform (DFT)，但我還沒解釋它的原理，現在是時候了。

如果你能理解 Discrete Cosine Transform (DCT)，你也能理解 DFT。兩者唯一差別是，一個用的是 cosine 函數，一個用的是複數指數(complex exponential)函數。我會從解釋複數指數開始，然後照第六章的流程來一遍：

1. 從合成問題開始：給定一組頻率成份與它們的振幅，我們如何建構一個訊號。合成問題等同於「反DFT」
2. 我們會重寫合成問題，利用 numpy array，寫成矩陣乘法的形式。
3. 接下來，我們會突分析問題，這等同於 DFT：給定一個訊號，我們如何找到與之對應的頻率成份的振幅與相位差。
4. 最後，我們會利用線性代數找到更有效率的方式計算 DFT。

本章的程式碼在 chap07.ipynb，它的位置請看0.2節。你也可以在後方連結找到：http://tinyurl.com/thinkdsp07 。

## 7.1 複數指數 | Complex exponentials

數學其中一個好玩的是，將一個動作從一種形式推廣到另一種形式。舉例來說，階乘是操作整數的函數，它的自然定義是 n 階乘就是從 1 到 n 的整數相乘。

如果你愛異想，也許會想知道如何計算非整數階乘，例如 3.5。因為自然定義無法應用，你應該尋求其他方式來計算階乘，能計算非整數階乘的方式。

在 1730 年，Leonhard Euler 找到一個，階乘函數的一般化的方式，我們現在叫 gamma function。(參見 http://en.wikipedia.org/wiki/Gamma_function)

Euler 也找到在應用數學最有用的一般化的方式之一，也就是 complex exponential function。

exponentiation 的自然定義是重覆相乘，例如，$ \varphi^3 = \varphi \cdot \varphi \cdot \varphi$ 但這個定義無法用在非整數指數。

然後，指數也可以用 power series 來表示：

$$ e^\varphi = 1 + \varphi + \varphi^2/2! + \varphi^3/3! + \dotso $$

經過重新安排之後，我們可以證明它等同於以下樣子：

$$ e^{i \varphi} = \cos \varphi + i \sin \varphi $$

你可以在後面連結找到它的推導 http://en.wikipedia.org/wiki/Euler's_formula

這個公式暗示 $ e^{i \varphi} $ 是複數長度為 1 的複數。如果你想像它是複數平面的一個點，它會在單位圓上。如果你把它想成向量，它與正 x 軸的夾角的弧度就是它的參數 $\varphi $。

在這個例子，指數是個複數，於是得到：

$$ e^{a + i \varphi} = e^a e^{i \varphi} = A e^{i \varphi} $$

A 現在變成實數，它就是複數長度，$ e^{i \varphi} $ 是個單位複數，它指示著角度。

NumPy 提供一個 exp 的版本可以使用複數：

```
>>> phi = 1.5
>>> z = np.exp(1j * phi)
>>> z
(0.0707+0.997j)
```

python 使用 j 來代表虛數單位，而不是 i。所以數字後面接個 j 就會被認為是虛數，所以 1j 就是 i。

當傳入 np.exp 的是虛數或複數，結果也會回傳複數，明確地說，是 np.complex128，它用兩個 64-bit 浮點數來表示。在這例子的結果是 0.0707+0.997j 。

複數有兩個屬性，實部 real 與虛部 imag：

```
>>> z.real
0.0707
>>> z.imag
0.997
```

要得到複數長度，你可以使用內建函數 abs 或是 np.absolute：

```
>>> abs(z)
1.0
>>> np.absolute(z)
1.0
```

要得到弧度，你可以用 np.angle：

```
>>> np.angle(z)
1.5
```

這個例子確定 $ e^{i \varphi}$ 是個複數長度為 1，弧度為 $\varphi$ 的複數

## 7.2 複數訊號 | Complex signals

如果 $\varphi(t) $ 是時間的函數， $ e^{i \varphi(t)} $ 也會是時間的函數。明確地說，

$$ e^{i \varphi(t)} = \cos {\varphi (t)} + i \sin{\varphi (t)} $$

這個函數描述隨時間變化的數(quantity)，所以它是一個訊號(signal)。明確地說，它是一個複數指數訊號。

在一個特別的情形下，當該訊號的頻率是常數，$ \varphi (t) $ 是 $ 2 \pi f t $，所以結果是複數正弦 complex sinusoid：

$$ e^{i 2 \pi f t} = \cos {2 \pi f t} + i \sin{2 \pi f t} $$

更一般化的形式，讓訊號從 offset $ \varphi_0 $ 開始，也就是：

$$ e^{i 2 \pi f t + \varphi_0} $$

thinkdsp 模組提供這訊號的實作，`ComplexSinusoid`：

```
class ComplexSinusoid(Sinusoid):
 
   def evaluate(self, ts):
        phases = PI2 * self.freq * ts + self.offset
        ys = self.amp * np.exp(1j * phases)
        return ys
```

`ComplexSinusoid` 繼承 `Sinusoid` 的 `__init__`。它提供的 `evaluate` 幾乎與 `Sinusoid.evaluate` 相同，唯一的差別是它使用 `np.exp` 取代 `np.sin`。

用之產生的結果是裝滿複數的 NumPy array。

```
>>> signal = thinkdsp.ComplexSinusoid(freq=1, amp=0.6, offset=1)
>>> wave = signal.make_wave(duration=1, framerate=4)
>>> wave.ys
[ 0.324+0.505j -0.505+0.324j -0.324-0.505j  0.505-0.324j]
```

這個訊號的頻率是每秒 1 個 cycle，振幅是 0.6 (沒指定單位)，相位移是一個弧度(radian)。

這個範例評估了 0 到 1 秒間等分切的四個時間點，其結果為四個複數。

## 7.3 合成問題 | Ths synthesis problem

我們之前對實數正弦函數做過一次，我們現在也要對複數正弦函數做一次。我們用不同頻率的複數正弦函數加總來製造一個訊號。這樣會給我們一個複數版本的合成問題：給定頻率與振幅的數個複數成份，我們要怎麼評估出這個訊號？

最簡單的解法是，建立 `ComplexSinusoid` 物件，然後將它加總。

```
def synthesize1(amps, fs, ts):
    components = [thinkdsp.ComplexSinusoid(freq, amp)
                  for amp, freq in zip(amps, fs)]
    signal = thinkdsp.SumSignal(*components)
    ys = signal.evaluate(ts)
    return ys
```
這個函數幾乎相同於 6.1節 的 synthesize1，唯一的差別是我將 `CosSignal` 換成 `ComplexSinusoid`。

這裡來個例子：

```
amps = np.array([0.6, 0.25, 0.1, 0.05])
fs = [100, 200, 300, 400]
framerate = 11025
ts = np.linspace(0, 1, framerate)
ys = synthesize1(amps, fs, ts)
```
它的結果是：
```
[ 1.000 +0.000e+00j  0.995 +9.093e-02j  0.979 +1.803e-01j ...,
  0.979 -1.803e-01j  0.995 -9.093e-02j  1.000 -5.081e-15j]
```
在最低層級來看，複數訊號是複數組成的序列，但是我們要怎麼解譯它？我們對實數訊號有一些直接的感覺：它們代表隨時間變化的數量；例如，一個聲音訊號代表空氣壓力的變化。但是我們在世界上量不到什麼東西會產生複數。

***
![](http://greenteapress.com/thinkdsp/html/thinkdsp037.png)

圖7.1 複數正弦的實部與虛部
***

所以，什麼是複數訊號？我沒有一個令人滿意的答案，我只有兩個不甚滿意的答案：
1. 複數訊號是數學上的抽象方法，在計算與分析上有用，但它無法直接對應到真實世界。
2. 如果你願意，你可以把複數訊號想成一連串的複數，每個複數由實部與虛部所組合。

採用第二個觀點，我們可以把前面的訊號拆成實部與虛部：
```
n=500
thinkplot.plot(ts[:n], ys[:n].real, label='real')
thinkplot.plot(ts[:n], ys[:n].imag, label='imag')
```
圖7.1 顯示其結果。實部是 cosine 的合，虛部是 sine 的合。雖然波形看來不一樣，但它們在相同比例包含相同頻率成份。對我們的耳朵來說，它們聽起來一樣。(一般來說，我們聽不出相位移)。

## 7.4 用矩陣合成 | Synthesis with matrices

我們在 6.2節做過，我們可以把合成問題用矩陣乘法來表示：

```
PI2 = np.pi * 2

def synthesize2(amps, fs, ts):
    args = np.outer(ts, fs)
    M = np.exp(1j * PI2 * args)
    ys = np.dot(M, amps)
    return ys
    
```

`amps` 是 numpy array，裡面裝著一串振幅。

`fs` 是一串頻率，組成成份的頻率，`ts` 是一串時間，那是我們會評估訊號的時間。

`args` 是 `ts` 與 `fs` 的外積，`ts` 是沿著列(row)往下行進，`fs` 是循著行 (column) 往右前進。(你可以參閱圖6.1)

矩陣 M 的每一行(column) 都包含一個複數正弦函數，每個都有其頻率，將用與 `ts` 評估訊號值 。

當我們將 M 乘上振幅，結果是一個向量，它的元素是對應於 `ts`。每個元素都是數個在特定時點的複數正弦函數的和。

接下來用之前範例代進來：

```
>>> ys = synthesize2(amps, fs, ts)
>>> ys
[ 1.000 +0.000e+00j  0.995 +9.093e-02j  0.979 +1.803e-01j ...,
  0.979 -1.803e-01j  0.995 -9.093e-02j  1.000 -5.081e-15j]
```

結果是一樣的。

在這個範例中，振幅是實數，但它們可以是複數。如果是複數時有什麼影響？記得，我們可以有兩種方法來思考複數，一個是「實數與虛數的和」，$ x + i y$，或是「實數振幅與複數指數的乘積」，$ A e^{i \varphi_0 }$。使用第二個思考方法，我們來看看，當我們將複數振幅乘上複數正弦會發生什麼事。對每個頻率 $f$，我們有：

$$ A e^{i \varphi_0} \cdot e^{i 2 \pi f t} = A e^{i  2 \pi f t + \varphi_0} $$

乘上 $ A e^{i \varphi_0 }$ 會使其得到振幅 $ A $ 與加上相位移 $ \varphi_0 $

***
![](http://greenteapress.com/thinkdsp/html/thinkdsp038.png)

圖7.2 兩個不同相位的複數訊號的實數部份
***

我們可以用前個範例測一下複數振幅：

```
    phi = 1.5
    amps2 = amps * np.exp(1j * phi)
    ys2 = synthesize2(amps2, fs, ts)

    thinkplot.plot(ts[:n], ys.real[:n])
    thinkplot.plot(ts[:n], ys2.real[:n])
```

因為 `amps` 是實數的 array，乘上 `np.exp(1j * phi)` 得到複數的 array，其中帶有 `phi` 弧度的相位移，振幅等同於 `amps`。

圖7.2 顯示波形有相位移，當 $\varphi_0 = 1.5 $ 每個成份頻率均平移約四分之一個週期。但不同頻率的組成成份有不同的週期，所う每個成份的平移量在時間上是不一樣的。當我們加總這些成份，導致波形有一些不同。

現在我們對合成問題有更一般化的解法，可以處理複數振幅。現在已準備好解決分析問題。

## 分析問題 | The analysis problem

分析問題就是合成問題的相反：給定一連串的樣本，而且知道訊號組成的頻率，我們可以算出每個成份的複數振幅 $a$ 嗎？

我們在 6.3節看過，我們可以用合成矩陣 $M$ 解決合成問題，然後再解決線性方程 $ M a = y $ 來得到 $ a $。

```
def analyze1(ys, fs, ts):
    args = np.outer(ts, fs)
    M = np.exp(1j * PI2 * args)
    amps = np.linalg.solve(M, ys)
    return amps
```

`analyze1` 由一個 wave array `ys`，一串實數的頻率 `fs`，以及一串實數的時間 `ts`而產生一序列的複數振幅 `amps`。

接續前個範例，我們可確定 `analyze1` 能夠回復原來的振幅。對於線性系統要能解，$M$ 必須是正方形，所以我們必須要讓 `ys`、`fs`與`ts`有同樣長度。所以我會把`ys`與`ts`切成與`fs`一樣長度：

```
>>> n = len(fs)
>>> amps2 = analyze1(ys[:n], fs, ts[:n])
>>> amps2
[ 0.60+0.j  0.25-0.j  0.10+0.j  0.05-0.j]
```

這答案接近我們原來的振幅，雖然每個成份有小小的虛數部份，那是因為浮點誤差造成的。


## 7.6 效率分析 | Efficient analysis

不幸地，解線性方程組就是慢。對於 DCT 來說，我們可以依靠漂亮地選擇 `fs` 與 `ts` 使 `M` 成為正交然後讓解方程組的速度增加。用那種方式，`M` 的反矩陣是 `M` 的轉置矩陣，我們可以用矩陣乘法同時計算 DCT 與 反DCT。

我們也會對 DFT 做一樣的事，但有一個小改變。因為 `M` 是複數，我們需要它是 __unitary__ 而不是正交。也就是「`M` 的反矩陣就是`M`的共軛轉置矩陣」。它可用轉置一個矩陣然後對每個元素的虛部乘上負號。請參閱 http://en.wikipedia.org/wiki/Unitary_matrix

NumPy 提供方法 `conj` 與 `transpose` 滿足我們的需要。以下的程式碼計算 $N=4$ 的成份的`M`：

```
    N = 4
    ts = np.arange(N) / N
    fs = np.arange(N)
    args = np.outer(ts, fs)
    M = np.exp(1j * PI2 * args)
```

如果 $M$ 是 unitary，就代表 $ M^* M = I$，其中 $M^*$ 是 $M$ 的共軛轉置矩陣，$I$ 是單位矩陣。我們可以利用以下方法測試 $M$ 是不是 unitary：

```
MstarM = M.conj().transpose().dot(M)
```
在浮點誤差的容許下，結果是 $4I$，所以$M$是 unitary，除了多了個因素 $N$。這個跟之前 DCT 範例多了個 2 的因素類似。

我們可以用這結果寫一個快一點的 `analyze1`：

```
def analyze2(ys, fs, ts):
    args = np.outer(ts, fs)
    M = np.exp(1j * PI2 * args)
    amps = M.conj().transpose().dot(ys) / N
    return amps
```

我們測試一下它的功能，代入適合的 `fs` 與 `ts`：

```
    N = 4
    amps = np.array([0.6, 0.25, 0.1, 0.05])
    fs = np.arange(N)
    ts = np.arange(N) / N
    ys = synthesize2(amps, fs, ts)
    amps3 = analyze2(ys, fs, ts)
```

這結果在浮點誤差的容許下是正確的。
```
[ 0.60+0.j  0.25+0.j  0.10-0.j  0.05-0.j]
```

## 7.7 DFT

`analyze2` 這個函式不好用，因為它只在 `fs` 與 `ts` 選對的時候能用。我要改寫一個版本讓它能只取 `ys` 就自己計算 `freq` 與 `ts`來用。

首先，我會寫一個函式計算合成矩陣 $M$：

```
def synthesis_matrix(N):
    ts = np.arange(N) / N
    fs = np.arange(N)
    args = np.outer(ts, fs)
    M = np.exp(1j * PI2 * args)
    return M
```

然後寫一個函式以 `ys` 而得到 `amps`：

```
def analyze3(ys):
    N = len(ys)
    M = synthesis_matrix(N)
    amps = M.conj().transpose().dot(ys) / N
    return amps
```

我們快好了。`analyze3` 這計算接近 DFT，只有一個差別，慣例定義的 DFT 不會除以 `N`：

```
def dft(ys):
    N = len(ys)
    M = synthesis_matrix(N)
    amps = M.conj().transpose().dot(ys)
    return amps
```

現在我們確定一下我的版本的產出會跟 np.fft.fft 一樣：

```
>>> dft(ys)
[ 2.4+0.j  1.0+0.j  0.4-0.j  0.2-0.j]
```

這結果接近 `amps * N`，底下是 np.fft 計算的結果：

```
>>> np.fft.fft(ys)
[ 2.4+0.j  1.0+0.j  0.4-0.j  0.2-0.j]
```

兩個結果在浮點誤差之人是一樣的。

反DCT 也幾乎一樣，除了我們不用轉置與共軛 `M`，而且現在我們有除以 `N`：

```
def idft(ys):
    N = len(ys)
    M = synthesis_matrix(N)
    amps = M.dot(ys) / N
    return amps
```

最後，我們確認一下 `dft(idft(amps))` 會得到 `amps`：

```
>>> ys = idft(amps)
>>> dft(ys)
[ 0.60+0.j  0.25+0.j  0.10-0.j  0.05-0.j]
```

如果，我能回到過去，我也許會改變 DFT 的定義，讓它除以 $N$，然後 反DCT 不要除以 $N$。這樣會讓我的合成問題與分析問題的表達方式比較一致。

或者，我也可以把兩個操作都改成除以 $ \sqrt N $，這樣 DFT 與 反DFT 會比較對稱。

但是我沒辦法回到過去，所以我們現在先使用這有點奇怪的慣例，在實務上這不會是個問題。

## 7.8 DFT 是週期的 | The DFT is periodic

在這章我把 DFT 用矩陣乘法的形式展現。我們計算合成矩陣 $M$ 與分析矩陣 $M^*$。當我們把 $M^*$ 乘上 wave array $y$，其結果的每個元素是 $M^*$ 的一個列與 $y$ 乘積，我們可以用 summation 的形式寫下來：

$$ DFT(y)[k] = \sum_n {y[n] exp(-2 \pi i n k / N)} $$

$k$ 是頻率的索引，從 $0$ 到 $N-1$；$n$ 是時間的索引，從 $0$ 到 $N-1$，所以 $DFT(y)[k]$ 是 $y$ 的 DFT 的第 $k$ 個元素。

正常來說，我們計算對 $k$ 的 $N$ 個值的 summation，是從 $0$ 算到 $N-1$。我們可以對其他沒有點的 $k$ 評估它。因為它們會開始重覆，也就是 $k$ 的值會等於 $k+N$ 的值或 $k+2N$ 或 $k-N$ 的值。

我們把 $k+N$ 加進這個 summation：

$$ DFT(y)[k+N] = \sum_n {y[n] exp(-2 \pi i n (k+N) / N)} $$

因為指數裡有個加法，我們可以將它分成兩部份：

$$ DFT(y)[k+N] = \sum_n {y[n] exp(-2 \pi i n k / N)exp(-2 \pi i n N / N)} $$

第二個指數的部份，因為它都是 $2\pi$ 的整數倍，所以都會是 $1$，於是就可以拿掉它：

$$ DFT(y)[k+N] = \sum_n {y[n] exp(-2 \pi i n k / N)} $$

這個式子跟 $ DFT(y)[k] $ 一樣，所以這個 DFT 也是週期性的，週期是 $N$。你會在底下的練習之一需要這個結論，那練習會要求你實作 Fast Fourier Transform (FFT). 

把 $DFT$ 寫成 summation 的形式提供了一個視角可觀察它如何實現的。如果你回頭看 6.2節的圖，你會看到合成矩陣的每個行是隨著時間評估的訊號。分析矩陣是合成矩陣(共軛)轉置矩陣，所以每個列是隨著時間評估的訊號。

因此，每個 summation 是 $y$ 與 array 之中訊號之一的相關(correlation)(參閱 5.5節)。也就是，DFT 裡的每個元素是兩個東西的相關，一個是 wave array $y$，另一個是特定頻率的複數指數。

## 7.9 實數訊號的 DFT | DFT of real signals

***
![](http://greenteapress.com/thinkdsp/html/thinkdsp039.png)

圖7.3 DFT，鋸齒波，頻率 500 Hz，取樣率 10 kHz
***

這個 Spectrum 類別在 `thinkdsp` 是立基於 `np.fft.rfft` 這是計算實數 DFT。也就是它針對實數訊號。但是 DFT 在這章裡是更一般化的，希望它能計算複數訊號。

當我們用「full DFT」計算實數訊號會發生什麼事？來看看個範例：

```
    signal = thinkdsp.SawtoothSignal(freq=500)
    wave = signal.make_wave(duration=0.1, framerate=10000)
    hs = dft(wave.ys)
    amps = np.absolute(hs)
```

這個程式碼做出個鋸齒波，頻率 500 Hz，取樣率 10 kHz。`hs` 包含波的複數 DFT。`amps` 包含每個頻率的振幅。但哪個頻率對應哪個振幅？如果我們看 dft 的內容，會看到：

```
    fs = np.arange(N)
```

這是設想每個值都是正確的頻率。問題在於 `dft` 不知道取樣率。這個 DFT 假設波是一個時間單位，所以它設想取樣率是每單位時間 $N$ 次。為了要解譯頻率，我們必須從任意時間單位轉回秒，像是這樣：

```
    fs = np.arange(N) * framerate / N
```

隨著這樣的改變，頻率的範圍從 0 到實際的 frame rate， 10 kHz。現在我們來畫這個 spectrum：

```
    thinkplot.plot(fs, amps)
    thinkplot.config(xlabel='frequency (Hz)', 
                     ylabel='amplitude')
```

圖7.3 畫出訊號的每個頻率的振幅，頻率從 0  到 10 kHz。左半圖是我們預期的，主頻在 500 Hz，諧波下降約在 $1/f$。

但是右半圖就有點驚訝。超過 5000 Hz 之後，諧波的振幅開始成長，最高在 9500 Hz。這發生什麼事？

答案是 aliasing。記得之前 frame rate 10000 Hz 時，folding 頻率是 5000 Hz。在 2.3節時，5500 Hz 的成份與 4500 Hz 的成份是無法分辨的。當我們用 5500 Hz 評估 DFT 時，跟用 4500 Hz 會得到一樣的值。相同的，用 6000 Hz 得到的值也會跟用 4000 Hz 的值一樣。

實數訊號的 DFT 對稱於 folding 頻率。因為沒有足夠的資訊，為節省時間我們可以只評估 DFT 的前半段。這也是 `np.fft.rfft`所做的。

## 7.10 練習

Solutions to these exercises are in chap07soln.ipynb.

Exercise 1   The notebook for this chapter, chap07.ipynb, contains additional examples and explanations. Read through it and run the code.

Exercise 2   In this chapter, I showed how we can express the DFT and inverse DFT as matrix multiplications. These operations take time proportional to N2, where N is the length of the wave array. That is fast enough for many applications, but there is a faster algorithm, the Fast Fourier Transform (FFT), which takes time proportional to N logN.

The key to the FFT is the Danielson-Lanczos lemma:

DFT(y)[n] = DFT(e)[n] + exp(−2 π i n / N) DFT(o)[n] 

Where DFT(y)[n] is the nth element of the DFT of y; e is a wave array containing the even elements of y, and o contains the odd elements of y.

This lemma suggests a recursive algorithm for the DFT:

    1. Given a wave array, y, split it into its even elements, e, and its odd elements, o.
    2. Compute the DFT of e and o by making recursive calls.
    3. Compute DFT(y) for each value of n using the Danielson-Lanczos lemma.

For the base case of this recursion, you could wait until the length of y is 1. In that case, DFT(y) = y. Or if the length of y is sufficiently small, you could compute its DFT by matrix multiplication, possibly using a precomputed matrix.

Hint: I suggest you implement this algorithm incrementally by starting with a version that is not truly recursive. In Step 2, instead of making a recursive call, use dft, as defined in Section 7.7, or np.fft.fft. Get Step 3 working, and confirm that the results are consistent with the other implementations. Then add a base case and confirm that it works. Finally, replace Step 2 with recursive calls.

One more hint: Remember that the DFT is periodic; you might find np.tile useful.

You can read more about the FFT at https://en.wikipedia.org/wiki/Fast_Fourier_transform. 


```python

```
