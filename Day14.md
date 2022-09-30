# Day13 - 「紋理&種類」- Material解密(三)  

> 這裡是「Three.js學習日誌」的第13篇，本篇的主旨是要介紹紋理的種類，這系列的文章假設讀者看得懂javascript，並且有Canvas 2D Context的相關知識。

今天主要是要來把昨天沒有介紹完的紋理補完~事不宜遲，就讓我們開始吧!

## **Ambient Occulusion Map** (環境光遮蔽紋理)

### 1. 什麼是環境光?

在介紹這種紋理之前，其實應該要先理解什麼叫做**環境光**(Ambient Light)。

在3D渲染的領域中，光源其實有很多種，而**環境光**就是其中之一。

**環境光**顧名思義就是從環境<u>四面八方均勻射在</u>物體上的光。

在正常狀況下我們假設不管觀察者在哪裡，從物體上面接收到的環境光反射都是一樣的。

這邊我們用一個簡單的**Demo**來解釋一下。

![img](https://i.imgur.com/keCffwh.gif)

> codepen連結:[點我](https://codepen.io/mizok/pen/jOxzOyd)


這個Scene裡面放置了:

- 一個**環境光**(Ambient Light)實例。
- 一個由`BoxGeometry`搭配**紅色**`MeshStandardMaterial`建立出來的方塊`Mesh`。
- 一個`Camera`

我讓**方塊**隨著時間旋轉，且讓**環境光**的強度隨時間忽高忽低的波動。

我們可以注意到，這個**方塊**不管轉到什麼角度，還有**環境光**的強度如何，方塊的面上都是呈現同一個顏色。

這個就是我們剛剛提到的:

```
不管觀察者在哪裡，從物體上面接收到的環境光反射都是一樣的。
```

我們其實可以把**環境光**想像成把物體浸泡在一個光子構成的海洋，沒有任何一處會特別的暗或是特別的亮。


> 當然這個前提是建立在方塊的六個面都是一樣的材質，具有同等的反射率/漫反射率等等。


### 2. 環境光遮蔽又是什麼?

從上面的介紹看來，**環境光**(Ambient Light)其實就是均勻的把光打在物體的任何一個面上。但在真實環境中，「環境光會均勻的射到物體上」這件事其實有點過於理想化。

舉個簡單的例子，像是一個中空的圓管。如果我們按照剛剛的解釋，圓管內部應該是要能夠呈現跟外部一樣顏色的。

但現實就不是這麼一回事，大多數狀況圓管內部應該是較暗才對。

也許你會覺得疑問: 但3D模型不是有`normal attribute`和`normal map`嗎?它們不就是用來決定面與光互動的狀況用的嗎?

對，但是就跟我們前面講的一樣，**環境光**不會受到角度，也就是`normal`的影響。

所以在這種情況下，我們會需要有另外一種**手段**，來<u>限制</u>物體上每一處「接收環境光的能力」。

而這就是**Ambient Occulusion Map** (環境光遮蔽紋理)的用途。

![img](https://i.imgur.com/QHhxERT.jpg)

>AOMap on v.s off 的差異


### three.js 的環境光遮蔽紋理

> 老樣子，作法大同小異

```javascript
  const tl = new TextureLoader();
  const aoTexture = tl.load('../img/ao.png'); 
  

  const mat  = new MeshStandardMaterial({
     aoMap:aoTexture
  })
```

## **Roughness Map** (粗糙紋理)

**Roughness Map** (粗糙紋理)其實和**Metallic Map** (金屬化紋理)是很像的東西，但是**金屬化紋理**主要影響的是表面的**反射率**，而**粗糙紋理**則是決定表面的**漫反射率**。

在`Three.js`的`MeshStandardMaterial`底下，其實有一個屬性叫做`roughness`，這邊我們可以看看下圖來了解`roughness`的用途。

![img](https://i.imgur.com/J11hyz8.jpg)

> 左邊是`roughness`為0的狀況，中間則是具有高`roughness`的情形

看過上圖其實大概就可以了解，`roughness`會使物體表面光源漫射率增加，接連導致表面的**霧化**。

而**Roughness Map** (粗糙紋理)的用途，其實就是去標記物體表面漫射率高的地方(顏色越深漫射率越高)。

### three.js 的粗糙紋理

```javascript
  const tl = new TextureLoader();
  const roughnessTexture = tl.load('../img/roughness.png'); 
  
  const mat  = new MeshStandardMaterial({
     roughnessMap:roughnessTexture
  })
```


## **Alpha Map** (透明紋理)

最後一個是*Alpha Map** (透明紋理)。

其實我們之前講到透明度的時候有稍微提到一下，**Alpha Map** (透明紋理)其實顧名思義就是紋理透明區塊的**Mapping**

![img](https://i.imgur.com/VUZ6IDb.png)

> 透明紋理會讓光線直接穿越過去，而不會反射。

![img](https://i.imgur.com/BtbSRDN.jpg)

> 白色的部分會保持不透明，黑色的部分會挖空


### three.js 的透明紋理

```javascript
  const tl = new TextureLoader();
  const alphaTexture = tl.load('../img/alpha.png'); 
  
  const mat  = new MeshStandardMaterial({
     alphaMap:alphaTexture
  })
```


## 小結

今天我們講完了**PBR**材質的6種常見紋理，接著我們即將要進入`Material`的部分，再敬請各位期待~

## 延伸閱讀

-[https://zh.wikipedia.org/zh-tw/%E7%8E%AF%E5%A2%83%E5%85%89%E9%81%AE%E8%94%BD](https://zh.wikipedia.org/zh-tw/%E7%8E%AF%E5%A2%83%E5%85%89%E9%81%AE%E8%94%BD)

