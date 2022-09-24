# Day8 - 「面面俱到」 - 幾何結構Geometry(二)  

> 這裡是「Three.js學習日誌」的第8篇，本篇的主旨是要介紹Geometry的概念，還有一些常用的Geometry子類的使用方法。這系列的文章假設讀者看得懂javascript，並且有Canvas 2D Context的相關知識。

我們昨天提到了一些關於`BufferGeometry`的基礎面知識，也介紹了一些`BufferGeometry`底下的屬性用途，而接著主要是想講講在客製理想幾何結構時可以用的一些方法，或是一些必要的知識。

## 面(faces)

「**假如說，我想要做一個正四面體，而這個正四面體，四個面要可以填充不同的Matatrial，要怎麼做?**」

其實這是我年初的時候在寫一個**Side Project**時碰到的狀況。

我當時一開始想得很簡單，因為平常使用`BoxGeometry`的時候，我們其實可以在生成`Mesh`的時候，傳入一系列的`Material`(要以陣列的方式傳入)，這樣就可以讓方塊的6個面個別使用不同的**材質**，像這樣:

![img](https://i.imgur.com/u0F1K3d.gif)
```javascript
const geo = new BoxGeometry(1,1,1,10,10,10);
const mats = [
    new MeshBasicMaterial({color:0xff0000}),
    new MeshBasicMaterial({color:0x00ff00}),
    new MeshBasicMaterial({color:0x0000ff}),
    new MeshBasicMaterial({color:0xf0f0f0}),
    new MeshBasicMaterial({color:0x0f0f0f}),
    new MeshBasicMaterial({color:0xfff000})
]
const mesh = new Mesh(geo,mats);
Scene.add(mesh)
```

> codepen連結:[點我](https://codepen.io/mizok/pen/BaxwXdW)


有些比較早期的`Three.js`版本(*其實也沒很早，大概是一年多前的`r124`版本*)，在`Geometry`底下是有`faces`這個屬性的。

> faces是一個陣列，裡面會有一個多面體每個面的實例

不過後來因為`r125`做出了一個重大改變，官方決定要改革**幾何結構**的底層邏輯，最後導致了很多開箱即用的`Geometry`子類連帶受到影響，有興趣的可以看看[這篇文](https://discourse.threejs.org/t/three-geometry-will-be-removed-from-core-with-r125/22401/13)。

為了避免偏離主題太多，講古就先講到這邊~ 讓我們重新回到剛剛的正四面體。

我一開始的想法想得很簡單，畢竟既然要做四個面有不同材質的正四面體，那當然首先就是先建立一個`TetrahedronGeometry`實例吧~ 然後接著應該就是如法炮製，傳入4個面的材質就好。

> TetrahedronGeometry就是three.js提供的正四面體(開箱即用的)幾何結構

所以我就這樣做:

```javascript
// TetrahedronGeometry的第一個參數是他的外接球半徑
const geo = new TetrahedronGeometry(1);
const mats = [
    new MeshBasicMaterial({color:0xff0000}),
    new MeshBasicMaterial({color:0x00ff00}),
    new MeshBasicMaterial({color:0x0000ff}),
    new MeshBasicMaterial({color:0xf0f0f0})
]
const mesh = new Mesh(geo,mats);
Scene.add(mesh)
```

> 結果是和我預期的完全不一樣，什麼都沒有長出來 = = 

我當時看了半天看不明白為什麼啥都沒跑出來，於是就先把**多重材質**取消，改成引入**單一材質**

```javascript
// TetrahedronGeometry的第一個參數是他的外接球半徑
const geo = new TetrahedronGeometry(1);
const mat = new MeshBasicMaterial({color:0xff0000}),
    
const mesh = new Mesh(geo,mat);
Scene.add(mesh)
```

> [這次](https://codepen.io/mizok/pen/MWGENNx)倒是很老實地跑出來了

![img](https://i.imgur.com/J8P8m53.gif)


在了解問題出在**多重材質**之後，我就開始查找資料，上網發問。最後我在比較`BoxGeometry`和`TetrahedronGeometry`兩者物件結構時發現了`groups`這個屬性。

`BoxGeometry`

![img](https://i.imgur.com/JH2gB8F.png)

`TetrahedronGeometry`

![img](https://i.imgur.com/yFGaFDv.png)

所以我就直接找到官方文件上面關於`bufferGeometry.groups`的[解釋](https://threejs.org/docs/?q=boxgeo#api/en/core/BufferGeometry.groups)。

![img](https://i.imgur.com/4jdJhUk.png)

這裡就有提到了，`groups`就是一個`geometry`底下的分組，每一組會構成一個獨立draw call，而且同時會附帶一個`Material`的slot。

這時我才理解到，原來`TetrahedronGeometry`沒有辦法填入多重材質是因為它底下並沒有去定義`groups`(陣列是空的)。

所以這邊如果我們要讓`TetrahedronGeometry`能夠填入四種材質，那就是得用`addGroup`去給內部頂點做分組。

``` javascript
const geo = new TetrahedronGeometry(1);
//這邊addGroup 第一個參數是代表從哪一個頂點起算，第二個參數則是該組一共多少頂點，接著最後參數則是給定一個數字做為該組材質的編號
geo.addGroup(0, 3, 0);
geo.addGroup(3, 3, 1);
geo.addGroup(6, 3, 2);
geo.addGroup(9, 3, 3);
const mats = [
    new MeshBasicMaterial({color:0xff0000}),
    new MeshBasicMaterial({color:0x00ff00}),
    new MeshBasicMaterial({color:0x0000ff}),
    new MeshBasicMaterial({color:0xf0f0f0})
]
const mesh = new Mesh(geo,mats);
Scene.add(mesh)
```

![img](https://i.imgur.com/9OwYWYA.gif)

> codepen 連結:[點我](https://codepen.io/mizok/pen/MWGOgPw)


終於弄出來四個面不同材質了!菜鳥小弟我當時感覺很開心，於是就想說那就來試著把`MeshBasicMaterial`加上圖片紋理看看好了~

> 這邊我們先稍微超前一下進度。

`Three.js`的圖片紋理基本上要用`TextureLoader` - 也就是**紋理載入器**，來讀取，我們可以使用`.load`這個方法，透過填入不同的`url`，這樣就可以返回對應的`texture`，然後再把他賦值給`MeshBasicMaterial`的`map`這一屬性。

> 把每個面都填入一張300*300的圖片。

```javascript
const tl = new TextureLoader();
const mats = [
  new MeshBasicMaterial({map: tl.load( 'https://picsum.photos/seed/123/picsum/300/300')}),
  new MeshBasicMaterial({map: tl.load( 'https://picsum.photos/seed/456/picsum/300/300')}),
  new MeshBasicMaterial({map: tl.load( 'https://picsum.photos/seed/789/picsum/300/300')}),
  new MeshBasicMaterial({map: tl.load( 'https://picsum.photos/seed/012/picsum/300/300')})
  
]
```

>codepen連結:[點我](https://codepen.io/mizok/pen/jOxaOrb)

結果。

![img](https://i.imgur.com/siiUw1m.gif)


圖片確實有跑出來了，但我們可以看到其中有幾個**面**，圖片的*定位*好像被放得*不太對*，而且好像有被拉長的狀況，這是為什麼呢?

其實這個就是跟我們前面提到的`uv`有關係，也就是`TetrahedronGeometry`預設提供的`uv`跟我們理想的不一樣。

這邊再複習一次，`uv`的定義就是:

```
代表一張材質貼圖上面，每一塊小部位，實際是要映射到模型上的「哪個位置」
```

所以這邊如果我們想要弄出理想中的正三角形，我們必須要重新定義`TetrahedronGeometry`的`uv`。

```javascript

geo.setAttribute("uv", new Float32BufferAttribute([ 
    //這些代表每個頂點實際上是映射到該面圖片紋理的哪一個位置
    //第一個頂點映射到原點
    0,0, 
    1, 0, // 第二個頂點映射到(1,0)，也就是圖片的右下角
    0.5, 1, // 第二個頂點映射到(0.5,1)，也就是圖片的上緣中點
    //
    1, 0,
    0.5, 1, 
    0,0,
    //
    1, 0,
    0.5, 1,
    0,0,
    //
    0,0,
    1, 0,
    0.5, 1
  ], 2)); // 這邊的2也是類似stride的意思，代表每個頂點持有兩組數值
```
> codepen連結:[點我](https://codepen.io/mizok/pen/LYmOYOB)

> 是不是看起來好些了呢? ![/images/emoticon/emoticon72.gif](/images/emoticon/emoticon72.gif)

![img](https://i.imgur.com/WH1WQWV.gif)

## 小結

今天我們講了關於`Geometry`的面的相關知識，包括怎麼樣把`BufferGeometry`所產生的模型來分面，還有如何調整`Geometry`的`uv`，有興趣的讀者不妨自己拿其他的內建`Geometry`來試著客製化作為練習。

`Geometry`的部分依然還沒有結束，敬請各位讀者期待~

## 延伸閱讀

- [https://www.twblogs.net/a/5eef515933cbe858769e6bd4](https://www.twblogs.net/a/5eef515933cbe858769e6bd4)

- [https://stackoverflow.com/questions/71547975/how-to-render-a-tetrahedron-with-different-texture-on-each-face-using-three-js/71548718?noredirect=1#comment126456642_71548718](https://stackoverflow.com/questions/71547975/how-to-render-a-tetrahedron-with-different-texture-on-each-face-using-three-js/71548718?noredirect=1#comment126456642_71548718)

- [https://discourse.threejs.org/t/three-geometry-will-be-removed-from-core-with-r125/22401/13](https://discourse.threejs.org/t/three-geometry-will-be-removed-from-core-with-r125/22401/13)