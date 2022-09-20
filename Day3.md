# Day3 - 進入Three.js的領域

> 這裡是「Three.js學習日誌」的第3篇，本篇的主旨是透過試做一個Three.js的Hello World案例，來讓讀者對於Three.js有基本的認識，這系列的文章假設讀者看得懂javascript，並且有Canvas 2D Context的相關知識。

## 初次來到新世界

第一次使用`three.js`，尤其是那些沒有使用過3D建模軟體的人，多半會對每一個基本操作都有不小的疑問，畢竟`three.js`提供的API種類繁多，而且光看官方文件又有一大堆的專有名詞，讓人頭昏眼花。

>建議初學者可以先去玩過隨便一款3D建模軟體(首推Blender)，畢竟菜鳥如小弟我就是這麼做的。

這邊我們會先做一個簡單的Hello World，並且一步一步講解每個步驟的用意，目的是希望可以讓讀者試試味道，並且對Three.js產生基本的認識。

---

### 0. 這個Hello World的目標

這個Hello World的目標是要畫出來一個轉動的立方體(如圖)。

![轉動方塊](https://i.imgur.com/qHjLieI.gif)
> codepen連結:[https://codepen.io/mizok/pen/XWqRyPQ?editors=0010](https://codepen.io/mizok/pen/XWqRyPQ?editors=0010)

### 1. 導入npm module

為了求快速，在這個案例中我們先用`codepen`搭配`https://cdn.skypack.dev`提供的CDN來取得`three.js`的module。

```javascript
import {
  ... //這邊接著會引入必要的模組
} from "https://cdn.skypack.dev/three";
```

### 2. 建立Scene實例

Scene這個類別按字面解釋就是**景**，他有點像是**世界**的概念，當我們要去把一個`three.js`的程序render出畫面，基本上就是要去渲染這個**景**裡面所有物件構成的圖像。

```javascript
import {
  Scene
} from "https://cdn.skypack.dev/three";

function main (){
   const scene = new Scene();
}

main();
```

### 3. 建立Renderer(渲染器)實例

`three.js` 其實有提供很多種類的`renderer`，但是最常用的基本上還是`WebGLRenderer`。渲染器顧名思義當然是用來渲染用的，他本身會提供一個`render`方法，可以用來把一個`Scene`中的圖像渲染出來。

這邊我們順手設置`antialias`(反鋸齒)和`alpha`(開啟透明通道)兩個option。

`antialias`(反鋸齒)是一種特殊的演算法，他可以用來柔化圖形邊緣因為像素顆粒太明顯而形成的*鋸齒狀瑕疵*。

而之所以要開啟`alpha`(透明通道)是因為`three.js`預設是不渲染透明通道的(也就是背景會全黑)。

還記得我們在前面演示過webgl清除畫布的方法嗎? 就是藉由填入特定的clearColor來達到清除的效果，清除色是可以設定為具有透明度的顏色的，所以假如我們要讓`canvas`背景為*透明無色*，那就必須要:

- 開放alpha通道
- 清除色(ex:(0,0,0,0))的最後一個數值必須是0

> 設置完`renderer`實例之後要記得把`renderer.domElement`放到dom tree裡面(domElement其實就是canvas)

```javascript
import {
  Scene
  WebGLRenderer
} from "https://cdn.skypack.dev/three";

function main (){
   const scene = new Scene();
   const renderer = new WebGLRenderer({
    antialias: true,
    alpha: true
  });
  document.body.append(renderer.domElement);
}

main();
```

### 4. 設定畫布大小與Viewport，還有清除色

這邊應該就沒甚麼特別的，`Viewport`其實就是我們在上一篇介紹過的`gl.viewport`方法，也就是畫布範圍的映射。

畫布大小與Viewport，還有清除色的設定方法都可以從`renderer`物件底下取得，他們分別是`renderer.setSize`,`renderer.setViewport`,還有`renderer.setClearColor`。

```javascript
import {
  Scene,
  WebGLRenderer,
} from "https://cdn.skypack.dev/three";

function main() {
  const scene = new Scene();
  const renderer = new WebGLRenderer({
    antialias: true,
    alpha: true
  });

  renderer.setSize(window.innerWidth, window.innerHeight);
  renderer.setViewport(0, 0, window.innerWidth, window.innerHeight);
  renderer.setClearColor(0, 0, 0, 0);
}

main();

```

### 5. 建立Camera實例

沒有使用過3D建模軟體的人可能會不太明白`Camera`的概念。

在3D建模軟體中，`Camera`的用途就是把景物"照"下來。

我們前面有提到，`three.js`要渲染畫面的時候要透過`renderer`的`render`方法，我們除了要告訴`renderer`是要渲染哪一個**景**以外，還得告訴他是要使用那個**景**裡面的哪一台`Camera`，這樣才可以取得那台`Camera`照出來的圖像。

![img](https://i.imgur.com/k6oScT0.gif)
> 這張圖片([來源](https://www.pcmag.com/encyclopedia/term/3d-graphics))稍微有點不正確，其實成像畫面不應該呈現在Near Clip plane上面。

`Camera`(相機)有很多種類型，每一種都各有各的使用場景。

在這邊我們使用`PerspectiveCamera`這個類所生成的相機實例，所謂的"**Perspective**"就是"**透視法**"，所謂的"**透視法**"是一種繪畫的技法，用來把實際的三維空間的座標點，投射到二維平面上。

> 關於透視法，我去年有寫過一篇3維投影相關的技術文章，可以參考[這裡](https://ithelp.ithome.com.tw/articles/10281029)

`PerspectiveCamera`這個類必須要傳入4個參數，分別是`FOV`，`畫面長寬比`，`近平面`，`遠平面`。

![img](https://i.imgur.com/l1lK8Ws.png)
> 圖片來源:[computergraphics.stackexchange](https://computergraphics.stackexchange.com/questions/12609/how-to-derive-field-of-view-fov-angles-from-a-2d-projection)

- `FOV`: 也就是攝影機的水平視角(Field of View)，基本上所有的光學成像儀器都會有`FOV`的設置，以**人眼**來講，人眼的FOV大約是120度，超出視角的東西就不會在成像畫面中顯示出來。
- `畫面長寬比`:就是預期成像畫面的長寬比。
- `近平面/遠平面`:其實就是一種範圍限制，攝影機不會把比近平面還要近，或是位置超出遠平面的東西照進去畫面。

另外順帶一提，`three.js`的`PerspectiveCamera`模仿的是現實生活中的`35mm`相機，也就是預設具備`35mm`的焦距，再換句話說就是攝影機到成像平面(上圖的z-projection平面)的預設距離為35單位長。

> 初始化過`PerspectiveCamera`的實例之後，記得要`add`進去`Scene`裡面。

```javascript
import {
  Scene,
  WebGLRenderer,
  PerspectiveCamera
} from "https://cdn.skypack.dev/three";

function main() {
  ...
  
  const camera = new PerspectiveCamera(
    75,
    window.innerWidth / window.innerHeight,
    0.1,
    1000
  );
  
  scene.add(camera)
}

main();

```


