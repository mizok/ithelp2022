# Day22 - 打造質感系3D聊天室 - three.js + socket.io

> 這裡是「Three.js學習日誌」的第23篇，這篇是在講解使用three.js + socket.io打造3D聊天室作品。這系列的文章假設讀者看得懂javascript，並且有Canvas 2D Context的相關知識。

今天我們要開始本次賽程的第一個創作!話不多說，就讓我們馬上開始吧!

這邊先來看一下今天我們預計要達成的進度畫面。

![img](https://i.imgur.com/X1fSlkG.gif)
## 前情提要

我們在上一回製作了`three.js`的`boilerplate`，也就是**專案模板**!

而這次我們當然就是要直接拿來使用了，不過在用之前我們先來看圖回憶一下整體模板的流程架構。


>備註:由於筆者在2022/10/9有對[上一回](https://ithelp.ithome.com.tw/articles/10306127)的文章做過一些優化更改，部分的內容可能跟賽事官方保留的快照有些微差異(主要是補上`Playground`類別的描述)，在這邊特此註明，對評審造成不便，敬請見諒。

![img](https://i.imgur.com/njj8lGZ.jpg)

`Github Repository`:
> https://github.com/mizok/three-ts-template

## 1. Clone、安裝，初始化

首先就是要先從上面的`Github Repository`先`clone`一份下來(當然也可以直接選擇**Use this template**)。

接著:

- `npm i`

- `npm run dev`

> 先跟大家說一下，這邊我其實已經先在上面的**REPO**裝好`gsap`，以備後續使用。

## 2. 先修改`./src/pages/index.main.ejs`

我們先來稍微修改一下`index.main.ejs`的架構，以符合之後UI上的需求。

```html
<!DOCTYPE html>
<html lang="en">

<%- include('../template/head.ejs',{title:'Index'})%>

<body>
  <div class="wrapper" id="wrapper">
    <div class="wrapper__inner">
      <canvas class="wrapper__canvas"></canvas>
      <div class="wrapper__chat-block chat-block " id="chat-block">
        <button class="chat-block__toggler" id="chat-block-toggler"></button>
      </div>
    </div>
  </div>
</body>

</html>

```

## 3. 調整一下`./src/ts/util/sizer.ts`

這邊因為我們在調整過`index.main.ejs`之後，`canvas`的`resize`機制會變得有點異常，所以得要修正一下`sizer`的`sizing`方法。

```typescript
sizing() {
        this.canvas.width = null;
        this.canvas.height = null;
        this.canvas.setAttribute('style', '')
        const rect = this.canvas.getBoundingClientRect();
        this.width = rect.width;
        this.height = rect.height
        this.trigger('resize', [this.width, this.height])
    }
```

## 4. 當然樣式也要順便處理一下

調整完結構接下來當然就是調整`./src/scss/main.scss`的樣式。

不過老實說筆者自己也覺得在這邊把一拖拉庫的`scss`搬上來照貼實在沒太大意義。

>畢竟切版也不是這次的主題

所以我打算用圖片說明一下我在UI方面的規劃。

![img](https://i.imgur.com/09T2zdg.jpg)


這邊我會把**UI**分成`canvas3D聊天室`區和`實體文本聊天室`區。

>給讀者的題外話: 對 ~ 兩邊都會顯示聊天室的內容，因為筆者小弟我覺得這樣很Cool~，麻煩別吐槽我實用性或意義的問題，那樣很沒幽默感唷 ^.<*~

- `canvas3D聊天室`: 就是中間的方塊

- `實體文本聊天室`: 右下角的按鈕按下之後整個畫面會往左推，接著在右側顯示出來`實體文本聊天室`的畫面。

> 如果真的想要確認樣式的部分，我們在這個作品創作的部分結束後會再提供這個專案的REPO地址。


## 5. 在畫面上實做帶有導角的方塊

在我們製作的`three.js boilerplate`中，如果想要創建新的物件，規範上是要開一個新的文件放在`./src/ts/mesh/`底下的

>並且要在`./src/ts/mesh/index.ts` Export 出去。

所以這邊我們先創立一個`cube.ts`在`./src/ts/mesh/`底下

`./src/ts/mesh/cube.ts`

```typescript
import { Clock, ExtrudeGeometry, Group, Mesh, MeshMatcapMaterial, Shape } from "three";
import { Base } from "../class/base";
import { MeshType } from "../interface";

export class Cube implements MeshType {
    mesh: Mesh;
    group: Group;
    ready = false;
    constructor(private base: Base) {
        this.setModel();
    }

    // 創建帶有導角的方塊的Geometry
    createRoundedBoxGeo(width: number, height: number, depth: number, radius0: number, smoothness: number) {
        let shape = new Shape();
        let eps = 0.00001;
        let radius = radius0 - eps;
        let faceRadius = 0.25;

        // 開始繪製Shape路徑，absarc是用來繪製橢圓曲線用的
        shape.absarc(eps, eps, faceRadius, -Math.PI / 2, -Math.PI, true);
        shape.absarc(eps, height - radius * 2, faceRadius, Math.PI, Math.PI / 2, true);
        shape.absarc(width - radius * 2, height - radius * 2, faceRadius, Math.PI / 2, 0, true);
        shape.absarc(width - radius * 2, eps, faceRadius, 0, -Math.PI / 2, true);
        // 把shape傳進去ExtrudeGeometry做extrude 和bevel
        let geometry = new ExtrudeGeometry(shape, {
            depth: depth - radius0,
            bevelEnabled: true,
            bevelSegments: smoothness * 2,
            steps: 1,
            bevelSize: radius,
            bevelThickness: radius0,
            curveSegments: smoothness
        });
        //以3D物件的包圍盒作為基準，相3D物件置中
        //不這麼做的話ExtrudeGeometry會以左下角為基準
        geometry.center();

        return geometry;
    }

    update(clock: Clock) {
        
    }
```

在這邊我們實際上是用`ExtrudeGeometry`來實作導角方塊。

所謂的`Extrude`(突出)其實是`3D`建模的一種術語，意思是把一或多個**平面**沿著它自己的法向量突出，並形成一個全新的體積。

![img](https://i.imgur.com/EFLTUSO.jpg)

> extrude前和extrude後

而大多數`3D`建模軟體在`Extrude`之後都有搭配一個機能，叫做`Bevel`，這個機能就是用來讓我們創造**導角**用的。

![img](https://i.imgur.com/rgMxBmG.jpg)

> blender的bevel看起來就像這樣

而`three.js`的`ExtrudeGeometry`用法是:

- 首先要先傳入一個 `Shape`的實例，`Shape`就有點像我們在`2D Context`上繪製的路徑

- 把shape傳進去ExtrudeGeometry做`extrude` 和`bevel`。

- 最後要記得發動`geometry`的`center`方法，不然他會以左下角為原點置中

## 6. 生成Mesh並讓方塊旋轉

我的構想是這個方塊在`onload`的時候會有一個比較大幅度的旋轉動畫，接著會開始慢慢自旋。

所以這邊我們這樣寫:


`./src/ts/mesh/cube.ts`
```typescript
    ...
    setModel() {
        const geo = this.createRoundedBoxGeo(3, 3, 3, 0.4, 20);
        //使用Matcap紋理，這樣可以大幅減少實時光照的效能問題
        const mat = new MeshMatcapMaterial({
            matcap: this.base.resources.cubeMatcap
        })
        // 這邊我們先建立一個group
        this.group = new Group();

        this.mesh = new Mesh(geo, mat);

        //開場的時候先把方塊縮小到看不見
        this.mesh.scale.set(0, 0, 0);
        //並且XYZ軸都旋轉60度
        this.mesh.rotation.set(Math.PI / 3, Math.PI / 3, Math.PI / 3);
        //把mesh放到group裡面
        this.group.add(this.mesh);
        //再把group放到scene裡面
        this.base.scene.add(this.group);
        

        this.doAnimation();
    }

    // 開場的時候使用gsap.to快速旋轉方塊mesh
    doAnimation() {
        gsap.to(this.mesh.rotation, {
            x: 0,
            y: 0,
            z: 0,
            duration: 1, // 用Tween的方式刻意的讓傳遞數值的動作產生delay
            paused: true
        }).play()
        gsap.to(this.mesh.scale, {
            x: 1,
            y: 1,
            z: 1,
            duration: 2, // 用Tween的方式刻意的讓傳遞數值的動作產生delay
            paused: true
        }).play()
    }

    // 每次tick loop都旋轉group
    update(clock: Clock) {
        this.group.rotation.y = clock.getElapsedTime() / 3;
    }
}
```


這邊比較值得一提的就是旋轉動畫的實作。

這邊我選擇去把`mesh`放入一個`group`中，再把`group`加入場景。

- `mesh`會作為**內層**，開場的時候會被旋轉。
- `group`會作為**外層**，每次`tick loop`的時候會被旋轉。



像這樣去實作方塊的動畫，也就是我們之前有提到過的「**把旋轉軸分離在不同層級**」，這樣就可以同時達成兩種旋轉方向的總和，而且也比較直觀。

> 如果忘記是哪裡有提到，可以看[這邊](https://ithelp.ithome.com.tw/articles/10296362)


## 小結

今天我們主要是作完整體外觀的一部分，明天將會繼續這部分的製作，希望各位能夠繼續追蹤~


