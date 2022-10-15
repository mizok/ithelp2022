# Day29 - 跟著甜甜圈大師~ 使用three.js + blender 一起製造甜甜圈 

> 這裡是「Three.js學習日誌」的第29篇，這篇主要在講解如何把Blender建立的模型，拿到three.js專案裡面使用。這系列的文章假設讀者看得懂javascript，並且有Canvas 2D Context的相關知識。

今天就是**最後一戰**了! 我們今天主要的目標就是利用`Blender`去實作一個甜甜圈的**建模**，然後把它放到`three.js`的`scene`裡面進行渲染。

## 1. 先來看看一些blender的基本快捷鍵

我們在上一回有提到，**blender**有非常多的預設快捷鍵，這邊我們先來講講比較常用的幾個:


### 自由旋轉視角(滑鼠中鍵)

就跟`three.js`的`orbitControls`差不多，只要按下中鍵就可以旋轉視角。

如果同時按住**shift**，則可以平移視角。

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

> 按下**Ctrl+A**來**Apply**剛剛的**形變(transform)**。


### 2-3 使用Smooth shade + subdivision modifier 來做一次整體優化

這邊我們要點選右鍵，並使用一個**黑魔法**!! 「Smooth shade」直接把**Torus**的陰影修成不會這樣一格一格的。

其實這是某種shader的應用方式，原理是透過對每個面的`normal`值做線性映射來計算曲面的陰影顏色，就有點像是我們之前說過的normal map(法線貼圖)，是一種障眼法。

>在這次的賽程我自己覺得時間太趕了，而且three.js東西真的太多，所以除了開場的webgl以外，沒有怎麼帶到shader的內容QQ，畢竟那算是更進階的know-how了。

![img](https://i.imgur.com/rbXBgXq.jpg)


接著，因為目前雖然**Torus**表面的陰影看起來已經優化，但我們若從**Torus**的邊緣看過去，仍然可以看到邊緣的**亮橘色**框線有一格一格的感覺。

所以我們這邊要來使用**blender**的**modiefier**(修改器)。

> 右邊側選單有一個扳手的小圖示~

假設我們把物體剛生成出來，並設定好參數(寬、高、網格細分等)這個階段稱作「先天」，那麼**modiefier**(修改器)則可以看成是一種「後天」對物體網格進行操作的手段。

這邊我們要使用的是**Subdivision Surface**這一種**modiefier**(修改器)，它的用途是可以「後天」的為物體做進階的**網格細分**。

> 這邊如果在修改器裡面登打數字時，會莫名的出現「按一下結果出現兩個字」的狀況的話，可以切換一下輸入語言，換成英文

![img](https://i.imgur.com/S6bIB2U.jpg)

這邊順帶一提，假如你在按下**Tab**這邊切換回去編輯模式，你會發現網格面數好像並沒有增加(但是面確實變平整了)，那是因為**modiefier**(修改器)其實也類似**形變(transform)**，它也必須要經過「**Apply**」才能夠附加到`Geometry`的頂點座標上面。

這邊如果要執行「**Apply**」的話，必須要先切換回去**物體模式(Object mode)**，然後點**modiefier**右上角的小箭頭。

![img](https://i.imgur.com/g8POO20.jpg)

不過我們這邊先不做「**Apply**」，畢竟之後還有可能會再次調整這個細分的數值。
### 2-4 搭配比例調整(Propotional Editing)來調整Vertex

接下來我們要稍微的調整這個**Torus**網格的形狀，把它變得更像現實中的甜甜圈。

> 先切換到點編輯模式。

然後**點亮**這個按鈕。

![img](https://i.imgur.com/v9yR4SR.jpg)

這個按鈕的功能是**比例調整**(Propotional Editing)，意思就是說當我們**拖動**其中一個頂點的時候，周遭的頂點也會根據與被拖動頂點距離的遠近，來產生線性的變化。

>記得先按下g來拖動其中一個頂點。

這邊我們可以使用滑鼠滾輪來決定對周遭頂點影響的範圍大小(因為我們剛剛有調整過**Scale**，所以這邊可能得**多滾幾下**才可以看到**調整影響範圍的提示圈圈**)

![img](https://i.imgur.com/cxb8mTt.jpg)

> 這邊如果想要看更Detail的操作，可以看ep2的[18:50](https://youtu.be/imdYIdv8F4w?t=1130)


### 2-5 創造糖衣(Icing)

這邊我們要來開始製作甜甜圈上層的糖衣(Icing)。

>原理是透過把`Geometry`的上半部複製出來一份，並且覆蓋在原本的`Geometry`上面

首先使用**Numpad**的**1**，把視角轉到正視圖，接著開啟**X-Ray**模式，並且框選出上半部的`Geometry`。

> 之所以開啟**X-Ray**模式是因為不開的話選不到背面的vertices

![img](https://i.imgur.com/sGyOlZ5.jpg)


接下來按下(**shift+d**)來**複製**這些框選起來的網格，這邊要注意，**複製** **vertice**並不會直接為我們創造一個新的**物件**。(**blender**反而會認為這些複製出來的頂點仍然屬於原本物件)

所以我們這邊在做完複製之後，還必須要按下`p`，並選擇`Selection`。


![img](https://i.imgur.com/IdrOZUG.jpg)

>這邊我們可以看到我們選擇的上半部頂點群被獨立出來成為了**Torus.001**

> 這邊如果想看細節操作，可以看ep3的[3:49](https://youtu.be/7wKnPclzYY8?t=229)

### 2-6 實體化修改器

這邊因為我們剛複製出來的糖衣只是一層薄薄的`Geometry`，然而真實的甜甜圈糖衣看起來應該更要有一種**厚度感**，所以我們在這邊要給糖衣套上另外一種**修改器**「**Solidify**(實體化)」，這是一種可以讓`geometry`的每個頂點，沿著法向量向外延伸，進而增加模型厚度的一種修改器。

![img](https://i.imgur.com/8fW68Us.jpg)

>這邊可以注意到，透過複製Torus本體而創造出來的糖衣同樣會繼承本體的Subdivision修改器。

這邊我們可調整**Subdivision**修改器和**Solidify**修改器的順序，讓糖衣變成「先增厚再細分」。

![img](https://i.imgur.com/7kVhs7J.jpg)

> 這邊的調整修改器所造成的差異主要會體現在糖衣邊緣的部分。


### 2-7 捏出糖衣滴落變形的部分

這邊我們先開啟**磁性吸附(sanp)**，這個功能可以讓我們在拖動頂點的時候，對拖動的方式進行輔助。這邊除了點亮**磁性吸附(sanp)**的磁鐵按鈕以外，右邊的吸附模式要記得選擇**面模式**，並且開啟「project indivisual element」這個選項。

![img](https://i.imgur.com/fhTYbrE.jpg)

> 這麼一來我們就可以沿著鄰近物件(也就是Torus)的面去拖動糖衣上面的vertex

> 這邊的操作細節可以看ep4 的[4:30](https://youtu.be/R1isb0x4zYw?t=270)

調整完這部分的細節之後先「Apply」糖衣的**Subdivision**修改器，然後再重新賦予另外一個**Subdivision**修改器

> 選定**Solidify**修改器後按下Ctrl+A也可以直接「Apply」


![img](https://i.imgur.com/itDZO5y.jpg)

> 這邊我們也可以先選擇某幾個邊緣上的點，再搭配**Extrude**(按鍵E)這個功能，用來拉出來比較誇張的滴落形狀

> 這邊的細節操作可以看ep4的[10:15](https://youtu.be/R1isb0x4zYw?t=615)


### 2-8 捏出甜甜圈中間的凹陷處

接著我們要來做出甜甜圈中間的凹陷處，首先選出**Torus**上面位於垂直方向偏中間的點，然後按下(Alt+滑鼠左鍵)選取一整圈的頂點，接著使用Scale(s)來把整圈頂點做縮放以營造出甜甜圈中間的凹陷。

![img](https://i.imgur.com/hObBqLF.jpg)

> 如果按一下沒有選到水平中線，那就多按幾下。


這時因為**Torus**的中間被我們往內收縮了，所以糖衣跟**Torus**中間出現了一部分的空洞。

在這個情況下，我們可以利用**Shrink wrap**修改器。**Shrink wrap**修改器可以藉由選擇一個**對象**(Target)，來讓被覆加上這個修改器的**物體**「貼合」這個對象。

![img](https://i.imgur.com/LOopFHQ.jpg)

> 附加上**Shrink wrap**修改器之後記得要把**Shrink wrap**修改器的順序提高到第一順位。

> 這邊的細節操作可以看ep4的[15:36](https://youtu.be/R1isb0x4zYw?t=936)

### 2-9 進入雕刻模式開始實作細節

首先在開始這一步之前先把糖衣和**Torus**的所有**modifier**(修改器)都「Apply」掉。

然後再點擊上方的**Sculpting**，就會進入**雕刻模式**。

![img](https://i.imgur.com/mvqr2dG.jpg)

**雕刻模式**其實就像是小畫家，他在左側會有多種的**雕刻筆刷**可以使用(這部分也可以搭配有感壓機能的**繪圖板**來操作，效果會比用**滑鼠**來的更好)

筆者比較常使用的工具有:

- Draw
- inflate
- Blob
- Smooth (尤其推Smooth，根本是救場神器QQ)

除此之外，這些筆刷都可以透過按下`f`來放大/`shift+f`來縮小筆刷大小。

> 這邊其實就是按照個人的感覺去選擇工具來使用就好，沒有什麼特別的步驟。

> 這邊的操作細節可以看[ep5](https://www.youtube.com/watch?v=G_OrMDOK-Og)
### 2-10 開始處理渲染環節囉~

在開始渲染之前，**第一步**我們先來把攝影機調整到適當的位置。

當然在這邊我們可以直接手動調整攝影機的位置~ 但Blender guru這邊有提供一個比較簡單的方法，也就是「Camera to View」。

首先我們先按`Ctrl+alt+Numpad0`，這樣就可以直接把攝影機移動到你當前檢視景物的視角，接著再在右邊的選單裡面勾選「Camera to View」，這樣你就可以把操作視角移動的機能鎖定在當前的View裡面。

![img](https://i.imgur.com/AlH0tqS.jpg)

接下來，我們要來講講`blender`提供的兩種渲染引擎: 「**eevee**」和「**cycles**」。

首先這兩種引擎都像我們平常在`three.js`裡面做的事一樣，透過計算物體模型的面法向量、位置、景深、質地、顏色,...etc. 來運算出來在某個視角下物體所呈現的樣子。

但是它們的差異在於「處理光反射的邏輯」

#### 「**eevee**」

> 其實我不確定「**eevee**」這個詞是不是來源於寶可夢的依布(eevee) XD。

「**eevee**」是一種**實時渲染**(realtime-render)的**渲染引擎**。

所謂的**實時渲染**(realtime-render)重視的是**效率**和**速度**，這種引擎在渲染的過程中會犧牲掉一部分的細節，用來換取更高的效能。

「**eevee**」適合的場景主要就是像網頁前端的模型，還有一些比較偏卡通風格渲染Style。

#### 「**cycles**」

「**cycles**」則是一種**靜態光線追蹤**(raytracing)的**渲染引擎**。

所謂的「光線追蹤(raytracing)」，是一種會計算光源在物體之間反射的渲染演算方式。

> 近年來「光追」這個名詞應該有被越來越多人聽到。

我們可以想像，假設有一個點光源，他會向外去散發出無數條的**光線**，假設其中有一條光線先打到了一個**紅色的物體A**上，那麼那個**紅色的物體A**，按照自然規律，就應該會反射出一定量的**紅色光**到鄰近的**物體B**上，接著**鄰近的物體B**會又因為接收到射過來的**紅光**，再根據自己的**顏色、質地**去決定要反射出什麼顏色的反射光到下一個物體，或是**觀察者**的眼中。

而像上面這樣一連串的光線路徑投射過程的運算，就是**光追**的本質之一。

光線追蹤式的渲染模式追求的是細膩度與自然度，所以它拋棄了效率和速度，渲染起來相對的需要花時間。

「**cycles**」渲染引擎通常適用在可以預先渲染的動畫，或是高仿真的照片。

![img](https://i.imgur.com/i5fDlFI.png)

> 上圖左是光追的運算結果，之所以龍的部分身體有較多的紅色像素堆積，是因為光線在龍的身體各部位反射(比方說頭反射的光線接著打到前胸，前胸又反射到某處)所計算出來的結果。

另外，「**cycles**」在渲染的時候通常會出現下圖的狀況，也就是圖像是一點一點地被算出來，跟我們習慣的`three.js`實時渲染不同~

>「**cycles**」的渲染速度會和顯卡/CPU的等級呈正相關，所以做3D動畫渲染一般會需要較好的顯卡。


![img](https://i.imgur.com/MFNfgKh.jpg)


> 接下來我們會先focus在「**eevee**」的使用，畢竟我們的目標是要把甜甜圈給丟到網頁前端做渲染。

科普講解差不多先到這邊~

接下來我們在場景裡面加入一個平面(Plane)。

![img](https://i.imgur.com/5rcvjS4.jpg)

> 這邊我在開場的時候不小心忘記先把Torus往上拉一點點XD，所以這裡甜甜圈會卡在平面上。

我們把**Torus**和糖衣一起往上拉。

> 可以搭配Numpad0使用，這樣比較好確認是不是剛好拖到平面上，當然拖曳形變做完要記的「**Apply**」!

> 假如發現拖不動的話，記得檢查一下是不是你的**磁性吸附**(snap)還開著沒關XD

接著我先把光源拉的離**甜甜圈&平面**近一點，且把光源強度下修到100W，這樣就可以看到陰影出現在平面上。

![img](https://i.imgur.com/McWcgrh.jpg)

![img](https://i.imgur.com/CgrvF4q.jpg)

再來我調整了一下相機的**焦距參數**。

![img](https://i.imgur.com/sRRD5Rz.jpg)

接著我們可以看到因為現在陰影看起來破破的(Ragged)，原因其實就跟我們之前在`three.js`裡面發生過的一樣。也就是「**shadowmap**」的解析度不夠。

這邊我們可以調整**eevee**的**shadow**相關設置。

![img](https://i.imgur.com/R2z9nMK.jpg)

再調整一下光源的**Shadow**屬性。

![img](https://i.imgur.com/rrimgbK.jpg)

>算是好多了。

然後我們可以按下`F12`來看一下渲染的結果。

![img](https://i.imgur.com/2sCOR5D.jpg)

> 有興趣的話也可以試試切換到cycles來看看渲染的狀況唷~

> 這邊的操作細節可以看ep6的[10:25](https://youtu.be/_WRUW_fs1g8?t=625)。

### 2-11. 上材質囉~

![img](https://i.imgur.com/k3MCtNT.jpg)

首先我們先分別點選**糖衣**、**Torus**、**平面**，然後再點選右側邊欄的**材質頁籤**(Material Properties)，來給這三個部分填上不同的**材質**。

|![img](https://i.imgur.com/Ys5RraO.jpg)|![img](https://i.imgur.com/a9q200n.jpg)|
|---|---|

> 通常材質的選色會比較吃建模師對顏色的敏感度，平常可以多練習這部分的直覺。

blender的材質有各式各樣的參數，其中當然也有我們熟悉的**Metallic**、**Roughness**，這邊我們可以稍微調整一下**糖衣**的**Metallic**、**Roughness**，讓它變得比較能夠反射光源。

![img](https://i.imgur.com/E1HOqFt.jpg)


這邊我們再介紹一個有趣的屬性: **Subsurface**，這是一個可以改變**材質透光率**的屬性，將這個數值提高可以帶來類似**果凍**的質感，這邊我們大概給個`0.01`~`0.02`就有明顯的不同了。


![img](https://i.imgur.com/FJV4DXe.jpg)


接下來我們要往下一個階段邁進了~

> 在這個部分Blender guru會講比較多關於Cycles渲染的細節，大部分我是略過了，如果有興趣的話可以從ep6的[15:51](https://youtu.be/_WRUW_fs1g8?t=951)看起。


### 2-12. 在blender裡面生成紋理貼圖

大部分的甜甜圈，在中間的凹槽，顏色是比較白的。

|![img](https://i.imgur.com/LFxF1Ml.jpg) |![img](https://i.imgur.com/jKBhBX2.jpg)|
|---|---|

但在我們的甜甜圈上面卻沒有這一圈**白色**的痕跡，而且本體顏色看起來也有點太乾淨。所以我們這邊其實需要一種方法來實作**比較寫實的甜甜圈**。

首先我們先隱藏**糖衣**和**平面**(點選之後按下**h**)，或是也可以點選該物件的**眼睛**圖標。

![img](https://i.imgur.com/q8AQ3nz.jpg)

![img](https://i.imgur.com/WHNufk0.jpg)

接下來我們打開**Shading**這個功能頁面。

![img](https://i.imgur.com/6Zt4oT7.jpg)

> 視角的縮放可能會需要調整一下。


**Shading**功能頁面其實就是讓使用者可以用**節點式**的操作方法，來自定義一個**shader**材質的Feature。

其實這個功能就類似於`three.js`的`ShaderMaterial`。

但很可惜我們在這次的賽程沒有時間可以講到`ShaderMaterial`，所謂的`ShaderMaterial`是一種運用`Shader`腳本，直接生成材質貼圖的方法，它的用法非常廣泛，裡面牽涉到的Know-how也非常的複雜。

> 如果鐵人賽可以開一個組別是能夠自由決定能夠寫幾天的就好了QQ，30天For three.js真的略短。

> 延伸閱讀: [three.js官方文件上ShaderMaterial的頁面](https://threejs.org/docs/#api/en/materials/ShaderMaterial)


接著我們先來看看**Shading**功能頁面都有些甚麼東西。


![img](https://i.imgur.com/ZQe8h5f.jpg)

- 上半部就是即時顯示紋理生成在模型上的狀況
- 下半部我們把它稱為**node-editor**，也就是**節點式的紋理編輯器**

我們接著會在這邊透過實作出由噪聲所生成的材質貼圖。

### 2-13. 使用**node-editor**來生成基於噪聲(Noise)的材質貼圖

首先這邊先介紹一下**node-editor**的使用方式。

在**node-editor**中，我們可以看到很多的面板，這些面板也就是我們所謂的**節點**。


![img](https://i.imgur.com/RxpHGP3.jpg)


- **節點**和**節點**之間的連線，可以透過滑鼠左鍵去**點擊=>拖拉=>連接**。

- 按下**Ctrl + 滑鼠右鍵**拖曳，就可以切斷路徑上的節點連線。

- 按下滑鼠中鍵，可以平移拖曳畫面

- 按下**shift+A**，可以決定要加入什麼樣的**節點**。

- 這些連線和節點會決定最後輸出的紋理結果。

> 筆者個人認為節點式渲染的原理應該是有點類似函數的Input/Output，把函數的Output傳給某個函數作為Input的概念，不過目前還沒有驗證過這個推論是不是正確的。

![img](https://i.imgur.com/ErekE85.jpg)

在這邊筆者是採用上述的節點連結還有參數，即可生成如下的紋理。

![img](https://i.imgur.com/ddnutNZ.jpg)

> 這邊節點的運用建議可以多看看[ep7](https://youtu.be/CmrAv8TSAao)影片中的介紹，不過筆者自己認為這種節點式的運算方法如果要達到可以自由運用，可能要經過多次練習，並且要熟悉每個節點的使用情境。


### 2-13. 繪製基底紋理

我們其實可以發現，在前面的**node-editor**中我們做到的，其實有點類似`three.js`中**Height map**、**color**、**Normal map**等多種屬性疊加起來的綜合結果。

而在`three.js`中，我們平常還可以加上`map`這一個屬性，也就是作為基底的**紋理**。

這邊我們要展示的就是要如何使用**blender**來生成**基底紋理**。

首先我們得要先把剛剛的**Shading**功能頁面中的**color ramp**這個**節點**刪除。並新增一個**image texture**的節點。

![img](https://i.imgur.com/mOuAeiw.jpg)

並點擊**image texture**上面的**New**按鈕，新增一張512*512大小，底色與原本**Torus**底色相近的的基底紋理，並取名為**base map**。

接著紋理的預覽畫面應該會變成像這樣。

![img](https://i.imgur.com/3PD22Rc.jpg)


接著再切換到 **Texture Paint**這個功能頁面。

![img](https://i.imgur.com/syercwd.jpg)


**Texture Paint**顧名思義就是**紋理繪製**，它除了可以繪製基底紋理以外，當然也是可以繪製例如**Metallic map**、**Height map**等其他類型的紋理。

> 只要你能畫得出來的話。


這邊我們可以直接把白色的區塊塗在模型中間凹槽的地方。

> 如果發現突然不能直接在模型上塗色，切回去shading再切回來texture painting就可以了。

![img](https://i.imgur.com/sGg0qN9.jpg)

接著畫完要記得存檔。

>點擊畫面左上角有寫著一個小"image"的地方，並選擇"save as"，blender會直接以你剛剛在image-texture節點上命名的名字來為這個材質命名。

![img](https://i.imgur.com/Bqz5Pto.jpg)


接著我們再回到**shading**功能面板，會發現紋理預覽上面已經出現我們剛剛繪製的**基底紋理**了~

![img](https://i.imgur.com/X4IRRd7.jpg)


這邊我們再稍微調整一下**node editor**的參數，並加入一個「mixRGB」的節點，然後把這個節點上提供的混和模式選項改為Overlay。

![img](https://i.imgur.com/1BBPdr7.jpg)

最後回到**modeling**功能頁面，解除**糖衣**和**平面**的隱藏~就得到了下圖的結果

![img](https://i.imgur.com/EFkG7DS.jpg)

>有沒有變得越來越像一回事了XD~?

> 這邊的詳細操作可以看 [ep8](https://youtu.be/_LeTDpNrdbg)

### 2-14. 使用Geometry node 給糖衣撒上糖粒(Sprinkles)

接著我們要為這糖衣的部分撒上糖粒。

要使用**Geometry nodes**這個功能。

**Geometry nodes** 顧名思義也是一種**節點編輯器**，但是他和我們剛剛使用的**shading**功能面板不同，是可以直接生成**Geometry的**。


這邊首先先打開**Geometry nodes**面板，然後**選擇糖衣**，接著按一下下圖中紅圈的**NEW**

![img](https://i.imgur.com/nuIcRRL.jpg)

所謂的**Geometry nodes**，其實基本上算是一種**modifier**(修改器)，我們透過這種**modifier**可以給原有的**Geometry**生成新的內容，而且這些內容可以透過組合不同的節點來決定。

通常**Geometry nodes**會被應用在需要產生具有規律的群體物件上(ex:大樓群、課桌椅)，又或者是透過隨機seed來產生隨機分布的**Geometry**(有點像**粒子系統**的概念)。

這邊我們透過剛剛的操作，已經為**糖衣**的部分掛上了一個**Geometry nodes**的**修改器**。

![img](https://i.imgur.com/pLqHPi1.jpg)


> 接下來的部份其實強烈建議要看過Blender guru本人的操作，在[ep9](https://youtu.be/4WAxMI1QJMQ)


接著我們給節點編輯器加上

- **Join geometry**

- **Distribute Points on Face** 

- **instance on points**

這三個**節點**，

![img](https://i.imgur.com/t60nHdy.jpg)

然後建立一個**Cylinder**(圓柱狀)的**物體**(記得寬高要先設定好)，然後把這個**Cylinder**，從物件列表**拖曳**進去**節點編輯器**。

![img](https://i.imgur.com/NGTmQyM.jpg)
![img](https://i.imgur.com/zC0gfuC.jpg)
![img](https://i.imgur.com/eIXiyC7.jpg)

我們接著把拖進來的**Cylinder**節點，和其餘三個節點像下圖一樣去做連結。

![img](https://i.imgur.com/ulCEdWx.jpg)

接著就會發現在**糖衣**的上面出現了兩根一樣的**Cylinder**。

![img](https://i.imgur.com/pgjQ0gY.jpg)

> 其實做到這邊應該差不多可以看懂接下來大概要幹甚麼了。

接下來我們點選剛剛的**Cylinder**物件(不是**節點**哦~)，然後輸入「rx90」讓他以**X軸**旋轉**90度**，並且把**Distribute Points on Face** 的**density**屬性提高。

> 形變完要記得「Apply」不然**Geometry nodes**不會有效果。

![img](https://i.imgur.com/6W2cqwj.jpg)

再來我們把**Distribute Points on Face** 的**Rotation**屬性和**instance on points**的**Rotation**屬性對連