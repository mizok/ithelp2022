# Day28 - 跟著甜甜圈大師~ 使用three.js + blender 一起製造甜甜圈 

> 這裡是「Three.js學習日誌」的第28篇，這篇主要在講解如何把Blender建立的模型，拿到three.js專案裡面使用。這系列的文章假設讀者看得懂javascript，並且有Canvas 2D Context的相關知識。

因為賽程只剩下今天跟明天，所以我想了一個相對比較簡單的主題~，我們之前有提到過賽程後半段有加入使用**Blender建模**的規劃，也就是**今天**了~接著就讓我們開始吧!


## 1.前端工程師學Blender? 你確定?

![img](https://i.imgur.com/t2jTeJF.gif)
>沒錯，不要懷疑，這就是完成後的前端畫面

歡迎來到**前端工程師の內卷之旅**的第一階段~~诶不是~~(X)。

> 雖然說我覺得這沒甚麼，筆者小弟美術系~~難民~~都跑來學前端了不是嗎XD?

### 1-1 學習Blender對於前端工程師有什麼幫助?

基本上，`Blender`這套3D建模軟體目前並不是台灣環境主流前端技能樹中的一部分。

但是學習`Blender`，其實就跟定期培養習慣，去看看美展，吸收一些基本的美學素養~有異曲同工之妙。何況只要學習**一點點**的**基礎**，就可以讓你的**作品集**質感大增，與其他競爭者的履歷作品相比起來獨樹一格。

>根本是Z>B(按:利大於弊)啊!!

### 1-2 選3DsMax或C4D不好嗎?

![img](https://i.imgur.com/8sYAqt8.png)

`Blender`是一套完全開源的3D建模軟體，你甚至還可以在**Github**上面看到它的[源碼](https://github.com/blender/blender)。

相較於其他的3D建模軟體，例如標題提到的*3DsMax、C4D*，這些軟體除了**主程序**要花錢訂閱以外，它們周邊的**渲染器/外掛**也同樣都要$$。所以身為一個**省錢黨**，`Blender`絕對是你的首選!

> 不過如果你願意花錢的話~那當然是選你所愛~愛你所選 :DD

### 1-3 那麼，想學Blender應該要怎麼開始呢?


我們今明兩天主要是會介紹著名的Blender 3D建模師 **Blender Guru**的初級教程，還有把教程產出的模型放置到`three.js`場景的過程。

[![Yes](https://img.youtube.com/vi/nIoXOplUvAw/0.jpg)](https://www.youtube.com/watch?v=nIoXOplUvAw)

**Blender Guru**的教程是**3D建模師**推薦的啟蒙首選，雖然說內容是**全英語**，不過**Blender Guru**本人的咬字其實還算清晰，同時再搭配YT的自動生成字幕，基本上不難懂(至少比YT上某些印度仔教寫Code的影片好多了)。

> 筆者小弟還是3D麻瓜時，學起Blender來，體感上老實講還真的沒有比當初從零自學Javascript困難XD

> 如果真的對英聽沒自信，但還是想學Blender，其實也可以考慮**小深藍老師**在[YOTTA](https://www.yottau.com.tw/course/intro/676#intro)開的課。


## 2.讓我們開始吧~

我這邊會用階段式的截圖+註解來展示我自己的建模過程，如果碰到不確定的地方可以直接看**Blender Guru**的影片。

> 備註: 因為我們的最終目標只是要把模型放到`three.js`的`scene`裡面，所以上面的教程只需要看完前面10個Part就可以~

### 2-1 .首先第一步當然是下載Blender~

![img](https://i.imgur.com/wnGiOXK.jpg)

> 官方下載連結: [https://www.blender.org/download/](https://www.blender.org/download/)

>這邊不太需要在意版本，blender目前除了幾年前在2.8版時做過一次大幅度的改版，後來幾次的更新基本都不太會影響到剛入門的新人。

### 2-2 基本畫面介紹

在剛打開`Blender`的時候，我們可以看到的場景大概如下:

![img](https://i.imgur.com/ByQQ9Ra.jpg)

- 左邊的奇怪三角錐體就是blender的camera，基本上它是一個perspective camera。
- 右邊一個小圓圈則是點光源。
- 座標的軸向基本跟`three.js`一樣。

> 是不是讓人很有親切感XD

在`Blender`中，有兩個基本是必須要先瞭解的:

1. `Blender`的在操作時有分成**編輯/物體模式**、而平常預覽時則有4種**視點模式**

2. `Blender`有很多的快捷鍵(可以試著記下來)，而且這些快捷鍵可能會因為你的滑鼠當前在不同的位置而有不同的作用(也就是所謂的**Area Sensetive**)

#### 編輯/物體模式

我們這邊可以先點一下畫面中一開始的方塊，這時我們會發現的邊框變成了橘色的。

![img](https://i.imgur.com/2FkIXFg.jpg)

當邊框變成**亮橘色**，代表他是這個Scene當前Active的物件，一個Scene裡面不會同時存在兩個Active物件。

而我們現在之所以可以直接把這個物件透過滑鼠選起來，是因為`blender`剛打開的時候，會是處於「物體模式」(**Object mode**)

跟「物體模式」(**Object mode**)相對的模式叫做「編輯模式(**Edit mode**)」，我們可以透過按下**Tab**來切換這兩種模式。

![img](https://i.imgur.com/7Y6MPiy.jpg)

> 先選到方塊之後，接著按下Tab就會進入編輯模式

另外，「編輯模式(**Edit mode**)」底下存在著3種子模式

這3種子模式也就是:

- 點編輯
- 線編輯
- 面編輯

顧名思義就是可以選擇一個物體(Object)的點(point)、線(line)、面(face)來進行諸如**旋轉**、**移動**等操作。

![img](https://i.imgur.com/JTYmzEI.jpg)

> 我們可以在切換到編輯模式後，於畫面的左上角看到現在是處於哪一種子模式

> 點(point)、線(line)、面(face)編輯模式可以用透過鍵盤左上的123數字鍵來快速切換。


#### 4種視點(viewport)模式

>其實「視點模式」這個名詞是我自己想的，我不確定這有沒有正式的譯名

當我們按下`Z`鍵的時候，可以看到畫面出現這個轉盤，這個就是所謂的「4種視點模式」

![img](https://i.imgur.com/iq1S1Tp.jpg)

>這四種視點模式會決定當前預覽物體的樣子

- Rendered : 最接近渲染輸出完的樣子的模式

- Solid : 也就是預設的灰色物體

- Wireframe : 網格模式可以讓我們看到當前模型的**布線**狀況

- Material Preview : 這個模式其實跟`Rendered`很像，但是差別在於他是屬於動態渲染，而不是光線追蹤式(Ray-tracing)的渲染，這我們之後會講到。



## 小結 

我們今天先到基本介紹這邊為止，明天將會是重頭戲，我們會一次把建模&置入`three.js`的部分講完，敬請期待~