# Day15 - 金屬球範例試作(2) - Material解密(五)  

> 這裡是「Three.js學習日誌」的第15篇，本篇的主旨是要透過一個簡單的範例操作，來一步一步介紹材質與渲染的關係，這系列的文章假設讀者看得懂javascript，並且有Canvas 2D Context的相關知識。

> **前情提要**: 昨天時間略趕，一時的疏忽不小心漏掉了一條需要在環境中補上**環境光**的描述，如果有人照著步驟走，可能會有亮度不夠的問題，所以這邊重新補上，對各位造成不便，敬請見諒。
```javascript
const al = new AmbientLight(0xffffff, 1);
...
scene.add(al);
```

今天我們要來完成昨天沒完成的範例。

![img](https://i.imgur.com/fRpRSUN.jpg)

### 7. 補上環境貼圖

我們之前在講解`texture`的時候其實沒有講到這種類型的`texture`。

所謂的環境貼圖指的是:

```
周圍景色透過物體表面反射到觀察者眼中的圖樣。
```

>有些人也會把這種概念稱為Skybox(天空盒)


在`Three.js`中，**環境貼圖**需要靠`CubeTextureLoader`來載入，它的形式會比較特別一點，當我們要載入一個完整的**環境貼圖**，我們需要依序傳入6張不同的周圍景色圖片。

```javascript
import {CubeTextureLoader} from 'https://cdn.skypack.dev/three';
...
const cubeTextureLoader = new CubeTextureLoader();
const envMap = cubeTextureLoader.load([
    envPx, //+X位置的環境貼圖
    envNx, //-X位置的環境貼圖
    envPy, //+Y位置的環境貼圖
    envNy, //-Y位置的環境貼圖
    envPz, //+Z位置的環境貼圖
    envNz  //-Z位置的環境貼圖
])
```

這邊我們會把6張環境貼圖分成+X/-X/+Y/-Y/+Z/-Z，之所以會這樣分是因為通常**環境貼圖**是從**HDRI圖像**(High Dynamic Range Image，高動態範圍圖像)去拆解出來的。

> 關於**HDRI圖像**的介紹可以看[這邊](https://keyshot.mairuan.com/rumen/hdri-xuanran.html)

![img](https://i.imgur.com/vG4jEXz.jpg)


|一般的HDRI圖片|拆解後|
|---|---|
|![img](https://i.imgur.com/Kc8cpgm.jpg)|![img](https://i.imgur.com/pVOggHQ.jpg)|

一般來說，如果想要自己製造**HDRI素材**，你會需要有一台360度相機，但如果沒有，其實網路上有很多免費下載的資源可以用。

>例如這裡:[Polyhaven](https://polyhaven.com/hdris)

而若手邊已經有一張合適的**HDRI素材**，可以用[這個](https://matheowis.github.io/HDRI-to-CubeMap/)免費的線上服務來做拆解。

>這邊我把拆解完畢的圖檔上傳到imgur備用


```javascript
const cubeTextureLoader = new CubeTextureLoader();
const envMap = cubeTextureLoader.load([
    'https://i.imgur.com/9wJp0Zy.png', //+X位置的環境貼圖
    'https://i.imgur.com/damIWcE.png', //-X位置的環境貼圖
    'https://i.imgur.com/mfqMr3m.png', //+Y位置的環境貼圖
    'https://i.imgur.com/0dpZmDE.png', //-Y位置的環境貼圖
    'https://i.imgur.com/Nj6WcOI.png', //+Z位置的環境貼圖
    'https://i.imgur.com/wwLgHqa.png'  //-Z位置的環境貼圖
])
...
```

接著我們把`envMap` 寫入球和平面的材質建構參數中，並且微調一下**反射率/漫反射率**。

```javascript
...
const mat1 = new MeshStandardMaterial({
    color: new Color("#eee"),
    metalness: 0.8,
    roughness: 0.3,
    envMap: envMap
  });
  const mat2 = new MeshStandardMaterial({
    color: new Color("#eee"),
    metalness: 0.8,
    roughness: 0.8,
    envMap: envMap
  });
```

![img](https://i.imgur.com/99ZXpJQ.jpg)

> 嘿~我們辦到了。

>  codepen 連結: [點我](https://codepen.io/mizok/pen/qBYoJMe)


### 8. 來試試看補上別的材質貼圖吧

雖然這邊我們已經達成製作**金屬球+金屬平面**的目標了，但我們接著其實可以來玩些別的玩意。

例如試著把**金屬材質**變成**木頭材質**。


#### 8-1. 首先來找張具有木頭紋理的材質貼圖，並用來生成其他材質貼圖

![img](https://i.imgur.com/jzo9PZI.jpg)

然後可以透過這個[免費服務](https://cpetry.github.io/NormalMap-Online/)產生基於這個木質紋理的:

- `Normal Map`
- `AO Map`
- `Height Map`

![img](https://i.imgur.com/X1UrGS9.jpg)


#### 8-2. 載入材質貼圖並填入球和平面的材質建構參數中

```javascript
const tl = new TextureLoader();
const woodTexture = tl.load('https://i.imgur.com/jzo9PZI.jpg');
const woodTextureNormal = tl.load('https://i.imgur.com/y60NQGm.png');
const woodTextureAO = tl.load('https://i.imgur.com/jDR50UI.png');
const woodTextureHeight = tl.load('https://i.imgur.com/3fGth9V.png');
...

const mat1 = new MeshStandardMaterial({
    map:woodTexture,
    normalMap:woodTextureNormal,
    aoMap:woodTextureAO,
    displacementMap:woodTextureHeight,
    color: new Color("#eee"),
    metalness: 0.8,
    roughness: 0.3,
    envMap: envMap
  });
  const mat2 = new MeshStandardMaterial({
    map:woodTexture,
    normalMap:woodTextureNormal,
    aoMap:woodTextureAO,
    displacementMap:woodTextureHeight,
    color: new Color("#eee"),
    metalness: 0.8,
    roughness: 0.8,
    envMap: envMap
  });
```

![img](https://i.imgur.com/eWJ7Hq6.jpg)

> 疑? 怎麼變得看起來很詭異XD

#### 8-3. 調整各貼圖參數

我們在上一個步驟會發現幾個詭異的現象，例如球卡進地板裡面/紋路上有看起來很詭異的光澤。

這是因為我們還維持著原本**金屬球**的**反射率**等參數，而且也沒有針對傳進來的材質做參數調整，所以我們這邊要著手進行優化。

首先球之所以會卡進去地板裡面，是因為地板也同樣被附加了`Height Map`，所以整體的`Geometry`頂點都被提高，這邊我們可以透過修改`displacementBias`來做微調。

除此之外，這邊我把兩個材質的`metalness`設置都拿掉，並且藉由提高`roughness`來提升漫反射率，這樣球體表面就不會再反射詭異的光澤。

最後我們把**球**的材質加上`displacementScale`的修正，目的是為了避免球出現模型破裂的狀況。


```javascript
const mat1 = new MeshStandardMaterial({
    map: woodTexture,
    normalMap: woodTextureNormal,
    aoMap: woodTextureAO,
    displacementMap: woodTextureHeight,
    color: new Color("#eee"),
    roughness: 0.3,
    displacementScale: 0.1,
    envMap: envMap
  });
  const mat2 = new MeshStandardMaterial({
    map: woodTexture,
    normalMap: woodTextureNormal,
    aoMap: woodTextureAO,
    displacementMap: woodTextureHeight,
    color: new Color("#eee"),
    roughness: 0.3,
    displacementBias: -0.5,
    envMap: envMap
  });
```

![img](https://i.imgur.com/ZYHV6Hx.jpg)

> 是不是好多了呢?

> codepen連結:[點我](https://codepen.io/mizok/pen/LYmmZWG)



## 小結

今天我們成功的把金屬球做出來了~同時還示範過要怎麼把金屬球換成木頭質感。明天將會是Material章節的最後一篇，我們會介紹一些除了MeshStandardMaterial以外的材質類。

## 延伸資源


- [材質產生器](https://cpetry.github.io/NormalMap-Online/)

- [免費HDRI材質提供 - Polyhaven](https://polyhaven.com/hdris)

- [免費3D紋理提供 ](https://free-3dtextureshd.com/)