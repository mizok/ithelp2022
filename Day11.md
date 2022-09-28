# Day11 - 「關於紋理」- Material解密(一)  

> 這裡是「Three.js學習日誌」的第10篇，本篇的主旨是要介紹紋理與材質的關係，這系列的文章假設讀者看得懂javascript，並且有Canvas 2D Context的相關知識。

其實我們在前幾天的內容有大概給大家展示過**紋理**(Texture)的使用，但沒有講得很Detail，所以今天我特別規劃了一篇文章要用來講解這個部分。


## 什麼是紋理(Texture)?什麼又是材質(Material)?

在3D建模的知識圈中，通常會有**紋理**(Texture)和**材質**(Material)這兩個專有名詞交互穿插於文獻之中。

通常**紋理**(Texture)這個詞指的是:

```
一組圖像數據，用來表示3D模型表面上的細節。
```

而**材質**(Material)這個詞則指的是:

```
一組定義光照模型如何與表面交互的係數。例如決定物體表面的反射率/漫反射率,...etc.
```

儘管概念不相似，但這兩者其實常常被混為一談，尤其是有些人已經習慣把**紋理**(Texture)稱作**材質貼圖**，導致兩者之間的界線更加模糊。

## Three.js中紋理的載入

在`Three.js`中，若想要導入**紋理**，例如我想要給一個**方塊**表面具有磚塊的圖樣，那就會需要用到`TextureLoader`。

通常我們會在專案裡面建立一個`TextureLoader`的實例。然後反覆地用`TextureLoader.load`這個方法去讀入材質。

```javascript
const tl = new TextureLoader();
const brickTexture = tl.load('../img/brick.png'); 
```
`TextureLoader.load`這個方法其實有點像是`jquery`的`$.ajax`，他也一樣可以傳入載入目標的`url`，還有onLoad/onError時的`callback`;

有些人可能會覺得好奇為什麼`TextureLoader.load`這邊不設計成回傳`Promise`。

> 畢竟大家都愛Promise，Promise讚!。

其實官方有說因為這樣改下去會牽涉到大幅度改動，所以暫時沒有規劃。

> 對這個話題有興趣的人可以看[這篇](https://github.com/mrdoob/three.js/issues/19060)文章

---

如果想要讓紋理載入可以採`Promise`機制，這邊會需要自己做包裝。

```javascript
import { TextureLoader } from "https://cdn.skypack.dev/three";

const tl = new TextureLoader();

const getTexture = (url) => {
  return new Promise((res, rej) => {
    tl.load(
      url,
      (texture) => {
        res(texture);
      },
      null //因為TextureLoader目前不支援OnProgress，但是卻又留了一個空的參數欄位，所以必須給null
      ,
      rej
    );
  });
};

async function main() {
  const someTexture = await getTexture("https://picsum.photos/id/237/200/300");
  console.log(someTexture);

  //接著就可以在這邊取用someTexture
}

main();

```

接著我們順便介紹另外一個`Class`，他的名字叫做`LoadingManager`。

我們可以把`LoadingManager`的實例傳進去`TextureLoader.constructor`裡面。

而每當被植入這個`LoadingManager`實例的Loader系函數進入`onLoad`/`onProgress`/`OnError`階段的時候，`LoadingManager`就會通報我們階段的發生。

> 這個功能通常適用在網頁剛載入，但資源還沒有完全載入，因此需要顯示一個載入條UI的狀況。


我們把上面的`Promise`包裝範例稍作改造，用來示範如何使用`LoadingManager`

```javascript
import { TextureLoader, LoadingManager } from "https://cdn.skypack.dev/three";

const lm = new LoadingManager();
lm.onStart = (url, itemsLoaded, itemsTotal) => {
  //onStart會在每一項資源開始載入的時候被執行
  //可以選擇用console.log的方式去顯示url, itemsLoaded, itemsTotal的狀況
  //url是該項資源的位址
  //itemsLoaded是目前一共已經有多少資源完成載入
  //itemsTotal是當前所有需要載入的資源數量
  console.log(
    "開始載入: " +
      url +
      ".\nLoaded " +
      itemsLoaded +
      " of " +
      itemsTotal +
      " files."
  );
};
lm.onProgress = (url, itemsLoaded, itemsTotal) => {
  console.log( '已載入: ' + url + '.\nLoaded ' + itemsLoaded + ' of ' + itemsTotal + ' files.' );
};
lm.onError = (url) => {
  //onError會在每一項資源載入失敗的時候被執行
};

const tl = new TextureLoader(lm);

const getTexture = (url) => {
  return new Promise((res, rej) => {
    tl.load(
      url,
      (texture) => {
        res(texture);
      },
      null,
      rej
    );
  });
};

//以上部分其實可以包裝成es6 module，這樣用起來就更優雅了。

async function main() {
  const someTexture1 = await getTexture("https://picsum.photos/id/237/200/300");
  const someTexture2 = await getTexture("https://picsum.photos/id/238/200/300");

}

main();


```

> codepen連結:[點我](https://codepen.io/mizok/pen/MWGrGjv)

![img](https://i.imgur.com/oZzgqY9.png)


## 如何使用紋理?

其實我們前幾天已經有示範過了，這邊就再讓我示範一次~
最簡單的方式其實就是把紋理填入`Material`的`map`屬性。

> 假設我們要用的Material是MeshStandardMaterial

```javascript
async function main() {
  const someTexture1 = await getTexture("https://picsum.photos/id/237/200/300");

  const mat  = new MeshStandardMaterial({
     map:someTexture1
  })
} 

```

> 就這麼簡單。

## 紋理的平移/旋轉/縮放/重複

常常有種狀況就是，當我們把**紋理**貼到模型上面的時候，才發現紋理貼圖的位置好像對不上模型。 又或者是有時候我們會需要去Repeat一個紋理，就像`css`的`background-repeat`一樣。

這種時候就需要調整`Texture`本身的屬性。

### 平移

平移可以透過`Texture.offset`這個屬性來完成。

> `Texture.offset`的型別會是`Vector2`。

```javascript
texture.offset.x = 0.5;
texture.offset.y = 0.5;
```

### 旋轉

旋轉的話就是`Texture.rotation`。他的型別是`number`，所以我們這邊必須要帶入`Radian`的數值。

預設情況下他是以**材質貼圖**的左下角做為**旋轉中心**，我們可以透過改變`Texture.center`這個`Vector2`的X/Y值來改變旋轉中心。

```javascript
texture.rotation = 0.5 * Math.PI;
```

### 重複

我們之前有講解過`uv`是什麼，但是卻沒有解釋`uv`這個名稱的由來。

`uv`其實代表的是**材質貼圖**的**X軸**(又稱**U軸**)，和**Y軸**(又稱**V軸**)，之所以這樣命名是因為要避免跟**3D物體**的**XY軸**混淆。

這邊如果我們想要讓材質沿著**U軸**重複，首先我們必須要先把`Texture.wrapS`設置為`RepeatWrapping`，而如果是要沿著**V軸**重複材質，則是要先把`Texture.wrapT`設置為`RepeatWrapping`。

> 這邊要注意`RepeatWrapping`是一個常數，必須要從`Three.js`的module中引入。

設置完`wrapS`/`wrapT`之後，接著就是設置`Texture.repeat`，他也一樣是一個`Vector2`。


```javascript
texture.wrapS = RepeatWrapping;
texture.wrapT = RepeatWrapping;

texture.repeat.x = 2;
texture.repeat.y = 3;
```

### 縮放

其實`Three.js`並沒有提供原生的**材質貼圖**縮放功能。
但我們可以透過調整`Texture.repeat`的值來實現這件事。

> 這邊記得不要去設定`Texture.wrapS`/`Texture.wrapT`，除非你除了放大縮小，還想要有重複

```javascript
texture.repeat.x = 0.5; //小於1的值會造成放大
texture.repeat.y = 2; //大於1的值會造成縮小
```


## 什麼是Mipmapping?

通常我們在實作大量的**紋理**重複時，我們會碰到一個狀況。這邊我用圖片展示給大家看一下。

![img](https://i.imgur.com/jSu3Gnm.jpg)

這張圖的左右兩邊圖樣都是透過大量重複同樣的紋理才形成的。

而當我們把攝影機往下移動。

![img](https://i.imgur.com/BnHoycV.jpg)

可以發現右半邊的圖形，好像看起來有種不自然的抖動感。

> 其實這是一種GPI成像的特性，有興趣的人可以看看[這篇文](https://www.jianshu.com/p/9272a4f0447a)

而如果想要消除這種不自然的感覺，讓右邊的圖變得跟左邊一樣，那就需要使用**Mipmapping**。

在`Three.js`中，與**Mipmapping**相關的設置是`Texture.minFilter`。在預設之下，`Texture.minFilter`的值是開放**Mipmapping**的，但如果碰到不需要使用**Mipmapping**的材質(例如材質沒有像上圖一樣有大量重複的紋樣)，我們可以把它換成別的。

例如:

```javascript
texture.minFilter = LinearFilter; //注意LinearFilter也是個需要引入的常數
```

另外，因為在`Three.js`中，**Mipmapping**的原理是透過預先渲染出一系列低畫素模糊版本的材質貼圖，才達成的。

>就像這樣。

![img](https://i.imgur.com/4aP8zNN.jpg)

所以如果真的要關閉**Mipmapping**，我們還需要停止預渲染上圖材質的動作。

```javascript
texture.generateMipmaps = false;
```

## 小結

今天我們講解了`Texture`的一些基本操作，但這還只是學習`Material`的第一步而已。

接著我預期至少也還要2天以上的時間才可以結束掉這個章節。

還請大家繼續追蹤~


## 延伸閱讀


- [https://zhuanlan.zhihu.com/p/475440200](https://zhuanlan.zhihu.com/p/475440200)

- [https://github.com/mrdoob/three.js/issues/19060](https://github.com/mrdoob/three.js/issues/19060)