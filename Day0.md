# Day0 - 為什麼想要寫這個主題?

> 這裡是「Three.js學習日誌」的第0篇，本篇的主旨在於講述本次參賽的想法

## 心理的嚮往

四年前我剛成為前端工程師的時候，心裡就一直對**3D特效**有一定的嚮往。

我不太確定原因是什麼，也許是因為當年還是個學生的時候，在系所開的課程裡看過系友們製作的3D動畫/建模 --- 當時我的心裡感到很憧憬，但身為研究生，除去寫論文/策展的時間外，剩下能用來研究技術的時間寥寥無幾，於是只好作罷。

而在離開學校成為了前端工程師之後，我的日常開始漸漸的離**美術/3D/動畫/建模** 越來越遠，原本以為應該再也沒有機會研究3D建模，直到我發現了**three.js**這個library。看著前端圈子中各式各樣國外大神製作的3D沉浸式網頁，我似乎也重新找回了那種熱情和初衷。

## 從2D Rendering開始的旅程

由於當時了解到，`three.js`是透過`canvas`提供的渲染環境來實踐前端的3D特效，所以我就一頭栽進去了開始研究MDN上面有關`canvas`的教程。

一開始我認為，既然要研究`canvas`，那應該就是要從最基本的`2D context`開始，就像學**功夫**，那馬步就必須紮穩，於是我花了大把大把的下班時間去做了一大堆的Survey，中間學習了基本的2D Animation rendering，2D 物理模擬，2D遊戲實作，去年還用2D這個題目參加了上屆的鐵人賽。過程確實蠻辛苦的，不過我個人覺得收穫到很多的樂趣。

## 來到webGL與3D的世界

在我從有了`2D Context`基礎，想要開始研究`webGL Context`的時候，我注意到一件事，那就是兩種環境之間的開發難易度差異所形成的GAP。

假如把一般的**前端開發**(也就是套套BS，串串api)比做都市中*市井小民的生活*，那`2D Context`的開發環境大概就是讓把一個都市人帶到*觀光露營區*，給他帳篷睡袋和一些簡易的野營工具，過程也許不像都市生活這麼舒服，但至少也不至於不好受。

但`webGL Context`的開發環境就沒這麼親切了，以我自己的感想而言，研究`webGL Context`就像是都市人不小心迷失在深山野外，身上除了衣服以外什麼都沒有，水源要自己找，火也要想辦法生，而你最終的目標是要憑空建立一座*自由女神像*。

在學習`webGL Context`的初期，我認知到:「如果想要想體會學習這門技術的甜蜜點(Sweet Point)，那我大概需要先從框架或libray開始」，所以才有了研究`three.js`的計畫。

## 在這次的參賽過程中想要達成的目標

表裡一體，簡單來說就是要記錄自己學習`three.js`的過程。最大的期望就是希望自己的努力過程可以成為別人的道標，在研究`webGl` /`3D` 的路上能夠更加順利。

- 初期會從`webGL`的基礎開始，講講自己踩過的坑，犯過的誤解
- 接著開始進入`three.js`
- 簡易的hello world 範例
- 基本的api操作
- 導入外部模型/外部材質
- 搭配Blender來建模用來放在網頁呈現
- ... 基本的預期大致上就這樣