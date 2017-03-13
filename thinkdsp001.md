
# 第零章 前言

訊號處理是我喜愛的主題之一。在科學與工程領域的許多方面都很有用，如果你了解一些基礎概念，當你在看世界的時候會多深入一些，尤其是我們聽的東西。

但除非你學電子或機械工程，不然你大概沒機會學訊號處理。這問題是，多數的書(而且多數使用它的課程)教的方法是 bottom-up 的方法，從介紹數學式子開始。而且他們傾向於理論化，所以比較少應用範例，彼此之間的相關性比較少。

這本書的前提就是，如果你知道如何寫程式，就可以用這個技術來學其他的技術，而且樂在其中。

[譯註：我自己就是用程式學了很多其他的東西。]

使用程式為底的學習方法，我可以馬上展現最重要的概念，在第一章的尾巴，你就可以分析聲音與其他的訊號，產生新的訊號。每一章介紹一個新的技術與應用，你都可以用到真的訊號去。每一個步驟，你會先學到如何使用這個技術，然後才是它怎麼辦到的。

這學習方法比較實用，我也希望你會同意，這比較有趣。

## 0.1 誰適合這本書

這本書是用 python 來寫範例與程式，所以你應該要了解 python，熟悉 object-oriented 的特色，至少會用別人定義的 object。

如果你不熟 python，你可以看我的另一本書教 python 的書，Think Python，這本是給沒學過的人看的。Mark Lutz 的 Learning Python，這本是給有寫程式經驗的人看的。

我額外用了 NumPy 與 SciPy，如果你已經熟悉它，那很好。但我仍會解釋我使用到的 function 與 data structure。

我假設讀者有基本的數學能力，包含要懂得複數。你不需要太深的微積分，如果你懂得積分與微分的概念，會有幫助。我會使用一些線性代數，但我會在碰到的時候解釋。

## 0.2 使用程式碼

這本書用到的程式碼與聲音檔放在 https://github.com/AllenDowney/ThinkDSP 。Git 是版本控制系統，讓你追蹤專案中的檔案。一組在 git 控管下的檔案叫做 repository。GitHub 是一個網路服務讓大家可以把 git repository 放在網路上並提供方便的 web 介面。

GitHub 提供了幾個方式來使用我的程式碼：

* 在 github 直接 fork，如果你有 github 帳號。
* 用 git clone 一個我的 repository，這不需要 github 帳號。
* 如果你完全不想用 git，那就按 download zip，把檔案一次下載回來。

所有的程式碼都是可在 python2，python3 執行的，不需要轉換。

我開發這本書的程式碼是用 Anaconda，由 Continuum Analytics 維護的。Anaconda 是一個免費的 python distribution。它包含了所有你會需要的套件。我發現 Anoconda 很好安裝，預設它是使用者層級的安裝，不是系統層級的安裝，所以你不需要擁有管理者權限。而且它也支援 python2 與 python3。你可以從這裡下載 Anaconda：http://continuum.io/downloads

如果你不想要使用 Anaconda，你可以自己安裝下面的套件：

* NumPy：基礎數值計算 http://www.numpy.org/
* SciPy：科學計算 http://www.scipy.org/
* matplotlib：視覺化 http://matplotlib.org/

雖然這些都是常用的套件，但它們都不包含在 python 的安裝裡，所以對於某些環境來說可能會有安裝上的難度。如果你在安裝上遇到困難，我建議用 Anaconda 來安裝，或是其他有這三個套件的 python distribution。

多數的練習使用 python script，但有些也使用 Jupyter notebook。如果你以前沒用過，我建議你參閱一下這個網站：http://jupyter.org

這裡有三個方法可讓你使用 Jupyter notebook：

* 在你的電腦上執行 Jupyter
  如果你安裝了 Anaconda，你應該已經預裝了 Jupyter。可以開啟命令列，用以下指令檢查：

    $ jupyter notebook

  如果沒有安裝的話，你可在 Anaconda 裡用這樣安裝：

    $ conda install jupyter

  當你啟動了伺服器，它會用你預設的瀏覽器開啟 jupyter 的頁面。

* 在 Binder 執行 Jupyter
  Binder 提供在虛擬機器裡執行 Jupyter 的服務。如果你進到以下連結 http://mybinder.org/repo/AllenDowney/ThinkDSP 你會進到這本書的 Jupyter 首頁與相關的資料和腳本。
  
  你可以執行其中的腳本，也執行自己修改過的腳本，但是虛擬機器是暫時的，如果你離開約一小時之後，任何你做的改變都會消失。
  
* 在 nbviewer 上看 notebooks
  稍後我們的書裡提到的 notebooks，我們會提供它們在 nbviewer 的連結，它會提供靜態的程式碼與結果的展示。你可以用這些連結閱讀 notebooks 與範例，但無法修改也無法執行這些程式碼，頁面上的互動零件也無法使用。
  
  祝好運，玩得開心！

## 貢獻列表

如果你有任何建議或是指正，請寄 email 到 downey@allendowney.com。如果我接受你的回饋而做了改戀，我會把你加到貢獻者名單裡(除非你說不要)

提出指正時請附上原句，這樣我會比較好搜尋。頁數及章節也是可以，但不是這麼好處理。感謝。

* 在我開始寫之前，我對於這書的許多想法有很多來自於我與 Boulos Harb (at Google) 與 Aurelio Ramos (之前任職於 Harmonix Music System)

* 在 2013 年秋季，Nathan Lintz 與 Ian Daniher 與我一起在一個獨立研究的專案工作，在這本書的第一版草稿幫助我。

* 在 Reddit's 的 DSP 討論區，我的 Brownian Noise 實作有錯誤，匿名的 RamjetSoundwave 幫我修正。Andodli 找到一個錯字。

* 2015 春天，我這份材料很榮幸被 Oscar Mur-Miranda 教授與 Siddhartan Govindasamy 拿來使用，有很多建議與指正。

* Giuseppe Masetti 給了很多有用的建議。

* Kim Cofer 編輯整本書，找到很多錯誤，給了很多有用的建議。

其他人們找到許多錯字與錯誤，包含 Silas Gyger 與 Abe Raher。

特別感謝技術編審 Eric Peters，Bruce Levens 與 John Vincent 給予的許多建議、澄清與修正。

也感謝 Freesound，我在書中用到許多聲音檔都來自這裡，同時也感謝提供這些檔案的人。我把聲音檔放一份在 github 裡，檔名沒有變，方便將來可以找到來源。

不幸的是，多數 Freesound 使用者沒有讓他們的真名露出，所以我只能對著他們的使用者名稱表達謝意。以下列出本書用到的聲音檔的提供者名稱： iluppai，wcfl10，thirsk，docquesting，kleeb，landup，zippi1，themusicalnomad，bcjordan，rockwehrmann，marcgascon7，jcveliz。謝謝你們。


