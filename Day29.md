# Day29 - 跟著甜甜圈大師~ 使用three.js + blender 一起製造甜甜圈 

> 這裡是「Three.js學習日誌」的第29篇，這篇主要在講解如何把Blender建立的模型，拿到three.js專案裡面使用。這系列的文章假設讀者看得懂javascript，並且有Canvas 2D Context的相關知識。

今天就是**最後一戰**了! 我們今天主要的目標就是利用`Blender`去實作一個甜甜圈的**建模**，然後把它放到`three.js`的`scene`裡面進行渲染。

## 1. 先來看看一些blender的基本快捷鍵

我們在上一回有提到，**blender**有非常多的預設快捷鍵，這邊我們先來講講比較常用的幾個:


### 抓取(g)

之所以是g，是因為它代表的是Grab(抓取)，我們可以在**物體/編輯**(點/線/面)模式對個別對象做抓取，
而被抓取起來的物件就可以自由移動。

另外:

- 按下`g`之後馬上接著按下`x/y/z`，就可以鎖定在指定的軸向移動。
- 按下`g`之後馬上接著按下`x/y/z`，在接著指定一個數字`N`，就可以朝該軸向移動`N`單位。
- 按下`g`之後馬上接著按下`x/y/z`，然後接著在維持按住`shift`的狀況下在按一次`x/y/z`，就會可以限制在某個平面上移動(例如先按`X`再按下`y`，就會在`xy`平面上移動)

> 這邊要注意一下，blender跟three.js的坐標軸不太一樣，blender朝上的座標軸是Z，但three.js是y

### 旋轉(r)

旋轉的快捷鍵沒有甚麼意外的是`r`，它跟`抓取(g)`一樣，也可以在按下`r`之後，按下指定的軸向，這樣就可以限制在以該軸為軸心做旋轉。


### 縮放(s)

縮放的快捷鍵同樣也沒有甚麼意外的是`s`，跟`抓取(g)`一樣，也可以在按下`s`之後，按下指定的軸向，這樣就可以限制在以該軸為軸向做拉伸、縮短。


### 切換編輯/物體模式(Tab)

我們在上一回其實有講過這個快捷鍵，這邊就不多提。


### 顯示視點模式的轉盤(z)

我們在上一回其實有講過這個快捷鍵，這邊也是不多提。


### 刪除物體/點/線/面(x)

初次使用**blender**的新手通常會因為想要刪除物體，但又不知道怎麼刪除，覺得很苦惱，因為**blender**的刪除不是**delete**，而是**x**。

> 這邊說的新手就是我。。。

只要先選取物體/點/線/面然後按下`x`，就會彈出小選單問你說是不是確定要刪除。

### 新增物體(shift+a)

|![img](https://i.imgur.com/cGxozxY.jpg)|![img](https://i.imgur.com/l6oiDdb.jpg)|
|---|---|

其實這個快捷鍵也可以直接點選上面的按鈕，點選之後就會出現新增物件的選單。


### 全選(a)

全選會把整個Scene裡面的物件都選起來，其中包括**光源**、甚至是**攝影機**。


### 數字鍵盤

數字鍵盤(Numpad)的用途是用來檢視**上下左右前後**、**正交**、**主攝影機**,...etc.所看到的景象。


|![img](https://i.imgur.com/UiZciaF.jpg)|![img](https://i.imgur.com/5mVkEyN.png)|
|---|---|

> 如果你的電腦沒有Numpad，blender有提供一個設置可以打開，打開之後你就可以用鍵盤英文字母上方的數字模擬Numpad的行為(如上方右圖)。


## 2. 開始製作甜甜圈吧~

### 2-1 首先先從一個**Torus**開始。

![img](https://i.imgur.com/k5ikoPZ.jpg)

> 按下(shift+a)然後從mesh這個類別裡面找到Torus

基本上所有的物體(Object)，在被創建的時候，左下角都會出現這個視窗。

![img](https://i.imgur.com/FMT6EKE.jpg)

而且重點是，這個左下角的視窗，只要在當你一取消對該物體的**Focus(Active)**，就會直接消失。

>這時我們可以按下F9來讓他重新顯示(mac鍵盤的話要搭配fn(fn+f9))。

不過，如果你除了取消**Focus(Active)**之外，還對這個物體作了**移動/旋轉/縮放**，那就算是按下**F9**也沒辦法把它叫回來了。

>就算是Ctrl+z也沒有用哦~

這個小視窗主要是讓我們決定這個物體的初始Config設置，就有點像`three.js` `torusGeometry`的建構式，它可以決定`torus`的內、外圈、截面半徑，也可以決定物體的
`segments`, (也就是我們在`three.js`講過的**網格細分**數量)

>網格數量很重要~所以下好物件離手之前都要再三確認過設置沒問題

這邊`segments`注意先不要下的太大，因為網格面數太多會導致模型的檔案容量過大。

![img](https://i.imgur.com/eF1AdXm.jpg)

> 建議可以看一下Blender guru的設置，在ep2 的 [4:54](https://youtu.be/imdYIdv8F4w?t=294)

### 2-2 調整一下大小

這邊我們需要調整一下大小，讓Torus比較接近現實生活中甜甜圈的大小。

![img](https://i.imgur.com/XfZ9J5G.jpg)

> 畢竟如果按照我們現在設置的大小，這會是個怪物等級的甜甜圈XD，放到`three.js`的Scene裡面會超級大~

這邊我們把它縮小到大約直徑為10公分左右(按s)。

![img](https://i.imgur.com/ns48hnt.jpg)

這邊接著要注意，在blender裡面，如果有做過任何的形變，最後一定要記得「**Apply**」。

所謂的「**Apply**」就是把擴張/縮小的值實際的附加到`Geometry`頂點座標上。

我們按下**Ctrl+A**來**Apply**剛剛的**形變(transform)**。