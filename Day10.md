# Day10 - 最後補充 - 幾何結構Geometry(四)  

> 這裡是「Three.js學習日誌」的第10篇，本篇的主旨是要介紹Geometry的概念，這是Geometry章節的最後一篇，主要會再補充能一些夠用來客製化Geometry的小技巧。這系列的文章假設讀者看得懂javascript，並且有Canvas 2D Context的相關知識。


在上一篇我們介紹了要怎麼從0去建立一個正立方體，並提及了要如何以Winding rule的規則去排序Triangle soup。

> 上回忘了說，Triangle soup 就是指一群三角面座標所形成的陣列。

今天主要是要收尾`Geometry`這一個章節，我們來講講最後一個段落 - 布林運算。

## 布林運算 - Union/Difference/Intersection

有用過**Adobe illustrator**的人應該都很習慣使用**路徑管理員**這個玩意。

![img](https://i.imgur.com/hCQJTr0.png)

**路徑管理員**就是一個可以用來做圖形路徑的**布林運算**，也就是**差集**（Difference），**聯集**（Union），**交集**（Intersection）...,etc.等操作。

而市面上的3D建模軟體中，大多也都會提供**布林運算**的功能，但是在`Three.js`中，官方卻<u>沒有</u>提供**布林運算**的方法。

> Yup，沒有。 當初知道這個我也有點意外。

如果想要在前端實現`Three.js`對**Geometry**的布林運算，我們必須要使用外部套件`ThreeCSG`。

> `CSG`的意思是『**構造實體幾何**(Constructive solid geometry)』，詳細的解釋可以看[這裡](https://zh.wikipedia.org/zh-tw/%E6%9E%84%E9%80%A0%E5%AE%9E%E4%BD%93%E5%87%A0%E4%BD%95)

這邊我們會用一個簡單的範例來給大家演示一下怎麼樣使用`ThreeCSG`。


### 1. 導入NPM包

這邊要注意，因為`ThreeCSG`有不只一個版本，現在`npm`線上有些版本是針對`Three.js` `r124`版本以前開發的，對於`r125`版本沒有向下相容性。

這邊的話是使用`three-csg-ts`這一個版本。

> npm頁面連結:[點我](https://www.npmjs.com/package/three-csg-ts)

```javascript
import { CSG } from "https://cdn.skypack.dev/three-csg-ts";
```

### 2. 初始化渲染環境並且選定布林運算的目標和對象

這邊我建立一個**正立方體**和一個**正四面體**的`Mesh`，我打算透過`CSG.subtract`方法，用正四面體把正立方體削去一個角。

所以除了要建立**正立方體**和**正四面體**的實例之外，我還得調整一下正四面體的位置，這樣才可以達到削去一角的目的。

這邊要注意，要調整**正四面體**的位置，應該是要用`Geometry.translate`，而不是使用`Object3D.position`。


```javascript
  // 為節省篇幅，這邊略過渲染環境的設置
  ...
  const geo = new BoxGeometry(1, 1, 1);
  const mat = new MeshStandardMaterial({
    color: 0xff0000
  });
  const mesh = new Mesh(geo, mat);

  const geo2 = new TetrahedronGeometry(1);
  const mat2 = new MeshStandardMaterial({
    color: 0x00ff00
  });
  
  geo2.translate(0.5, 0.5, 0.5);

  const mesh2 = new Mesh(geo2, mat2);
```

### 3. 發動`CSG.subtract`

CSG 一共有提供三種布林運算方法，分別是:

- `subtract`(差集)
- `union`(聯集)
- `intersect`(交集)

三個方法都可以接受2個參數，第一個參數要填入被做**布林運算**的**受體**(型別是`Mesh`)，第二個參數則是填入**客體**(型別也是`Mesh`)，而方法執行完之後會返回結果的`Mesh`，其材質會與**受體**的材質相同。

這邊我們使用`subtract`(差集)作為示範 :

```javascript
  const mesh3 = CSG.subtract(mesh, mesh2);
  scene.add(mesh3);
```
### 4. 結果

![img](https://i.imgur.com/C8ok41x.gif)

這邊我們可以把被削去一角的正立方體`Mesh`印出來看看。

![img](https://i.imgur.com/UiOcP9j.png)

可以發現`ThreeCSG`甚至還幫我們補了一個**切面**的`group`進去。

>也是蠻貼心的XD


## 小結

`Geometry`的章節先到這邊告一段落，不過老實說我之後還有規畫要講講怎麼樣用`Blender`來建立自定義模型，這部分再敬請大家期待。

明天我們會開始講`Material`的部分，這部分也敬請大家期待~

## 延伸閱讀

-[https://zh.wikipedia.org/zh-tw/%E6%9E%84%E9%80%A0%E5%AE%9E%E4%BD%93%E5%87%A0%E4%BD%95](https://zh.wikipedia.org/zh-tw/%E6%9E%84%E9%80%A0%E5%AE%9E%E4%BD%93%E5%87%A0%E4%BD%95)