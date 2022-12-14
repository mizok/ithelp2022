# Day1 - 從webGL的基礎開始?(一)

> 這裡是「Three.js學習日誌」的第1篇，本篇的主旨是藉由描述一些簡單的webGL基礎，來做為引導three.js學習的鋪墊

## 學習three.js到底需不需要完備的webGL知識?

學習three.js到底需不需要完備的webGL知識? 這個其實是我當初開始自學`three.js`時非常懷疑的一點，畢竟自己完全是個菜鳥，很自然地就會想要複製之前學習其他領域知識時的習慣(也就是想要把基礎/馬步都先紮好了再出發)。

但是到了我開始深入webgl相關領域的知識時，才發現它的深度跟廣度真的超乎我的想像，如果要完全的學習webgl，可能要理解非常多的東西才能夠達到能夠把這門技術運用在專案面的程度。而為了不讓自己的熱情與時間消耗在一些無謂的造輪子過程，我選擇只先理解一部分的`webgl`基礎，接著就直接銜接到`three.js`的部分。

這邊提到的**基礎**大略概括下面幾項:

- `canvas`元素的一些基本常識(例如`canvas`動畫的實作原理，`canvas` width 與像素密度的關係,...etc)
- 能成功地寫出來一個`webgl`渲染的hello world
- 了解`vertex shader` 跟 `fragment shader` 的差異
- `three.js` ，`webgl`，`3D`的相互關係

我個人認為，有了上面這幾項條件，至少可以避免在學習過程中像*鴨子聽雷*一樣，根本不明白自己到底學了些什麼。

## 先來了解 three.js/webgl/3D的相互關係

第一次學習webGL的人多半會有一個誤解就是:「*webGL就是用來繪製3D圖像的渲染環境，要作3D繪圖就得靠webGL*」

但其實不是這樣的。

電腦上的3D圖形其實只是一種假象，他是透過透視投影的手段去把三維空間的座標數據投影在二維平面上，不管是`2D Context`還是`webGL context`，其實都是一樣的手法。

> 延伸閱讀: [Day 27 - 3D繪圖篇 - 2D圖片上面的3D物件是怎麼產生的? I - 成為Canvas Ninja ～ 理解2D渲染的精髓](https://ithelp.ithome.com.tw/articles/10281029)


那接著大概就會有人提到一個疑問點:

「**所以你的意思是說其實就算是用2D渲染環境也一樣可以營造出3D效果? 那為什麼要使用webGL渲染環境?**」

使用2D的渲染環境確實可以營造出3D效果，[這裡](http://demo.jb51.net/js/2014/html5-3d-cloth-move-codes/)就有個經典案例。

而我們之所以要使用`webGL Context`，原因是為了**效能**。

比起`2D Context`，`webGL Context`能接受更加複雜的運算，但伴隨而來的也是更高的學習門檻和更冗長的編程體驗。

舉個簡單的例子:

假如我們要在`2D Context`的環境下畫一條弧線，我們就只要發動`ctx.arcTo`這個方法，然後再指定線段開始與結束的座標，還有弧線的曲率半徑，最後再選個顏色就可以劃出一條完美的弧線。

但換到`webGL Context`，他並沒有像是`ctx.arcTo`這樣這麼方便的api，它的原理比較像是去**定義**一張canvas上，到底是哪些像素構成了圖像，接著**賦予**這些像素指定的顏色，用這樣的手法來形成特定的圖形。

換句話說，`webGL Context`其實是一種更加接近電腦圖像底層渲染機制的東西。要探討`webGL`的渲染機制，往往會需要牽涉到GPU/CPU層面的資訊。

而說了這麼多，那`three.js`又是處於什麼樣的定位呢? 其實`three.js`就是把一些原本很複雜的`webGL`操作，例如繪製平面/立體形狀,...etc.包裝成可以直接使用的api，這樣一來就簡化了很多繁雜的過程，同時也可以做到前端的3D圖形渲染。

> 其實說白一點three.js之於webGL其實就像是jquery之於js

![img](https://i.imgur.com/OsIjm6n.png)