# Day19 - Three.js與滑鼠互動操作(三)

> 這裡是「Three.js學習日誌」的第19篇，這篇的內容是要來講解Raycasting的概念。這系列的文章假設讀者看得懂javascript，並且有Canvas 2D Context的相關知識。

今天我們來講講`Three.js`中與滑鼠**懸停(Hover)**/**點擊(Click)**息息相關的技巧:**Raycasting(光線投射)**


##  什麼是Raycasting(光線投射)?

當我們在**Google**上面搜索**Raycasting**這個詞的時候，其實可以找到兩種方面的應用。

一種是關於3D圖像體積繪製的演算方法，另外一種則是遊戲中對於3D物件的命中檢測/點擊偵測,...等。

> 其實兩者背後的原理差不多，只是應用的面向不同而已。

|3D圖像體積繪製演算|命中檢測/點擊偵測|
|---|---|
|![img](https://i.imgur.com/hFBIcS2.png)|![img](https://i.imgur.com/3wzAfes.jpg)|

而我們今天要講的內容跟後者比較有關。

如果要簡單解釋Raycasting這種技術在點擊偵測上的應用原理，我們可以想像當我們點擊螢幕的時候，螢幕會在**滑鼠**點擊的位置，沿著螢幕的**法向量**(也就是**攝影機**的朝向)，去延伸出去一條<u>無限長的射線</u>。

而若這條射線與任何一個3D物體相交的話，該物體就會被判定為「**被點擊**」。

![img](https://i.imgur.com/MWfI8ez.jpg)


## Three.js 中的Raycasting實作

在`Three.js`中，如果想要做到`mesh`的**懸停/點擊**偵測，那就要使用`Raycaster`。

```javascript
import {raycaster} from "https://cdn.skypack.dev/three";

const raycaster = new Raycaster();

// 從相機的朝向發出射線
raycaster.setFromCamera( pointer, camera );
// 把一整個scene的mesh通通檢測過一遍，看有沒有相交
const intersects = raycaster.intersectObjects( scene.children );
```

> 說實話，其實只要弄懂了`Raycasting`的概念，那這段程式應該也不難理解才對。

除了**物件點擊偵測**之外，`Three.js`的`Raycaster`也可以用來偵測到底是點擊到物件的哪一個**面**，甚至是**面**上的哪一個**座標位置**。

> 

接著我們會用一個簡單的範例來演練如何使用`Raycaster`。

## Raycaster 範例演練


### 從一個空白的Scene開始

> 老樣子，初始化的過程就跳過了~

首先我們先在這個空白的`Scene`裡面，填入大量的`Box Mesh`，然後讓這些`Box Mesh`以隨機的方式分布。

> 這邊要特別注意`Material`一定要By Loop去重新生成，不要讓所有Meshe共用用同一個材質實例。

```javascript
...
const randomness = 10;//亂度係數
for (let i = 0; i < 300; i++) {
  const geo = new BoxGeometry(1, 1, 1, 10, 10, 10);
  const mat = new MeshStandardMaterial({
      color: new Color("rgba(255,0,0,0.1)"),
      transparent: true
  });
  const mesh = new Mesh(geo, mat);
  mesh.position.set(
    (Math.random() - 0.5) * 2 * randomness, 
    // 之所以要先減去0.5再乘以2，
    //是因為要讓方塊落在-randomness到+randomness的範圍
    (Math.random() - 0.5) * 2 * randomness,
    (Math.random() - 0.5) * 2 * randomness
  );
  scene.add(mesh);
}
```

### 抓取滑鼠的位置

接著我們需要像上一回一樣，透過`addEventListener`去抓取滑鼠座標，並把座標以`Vector2`的形式記錄下來。

```javascript
const cursor = new Vector2();
// 這裡綁定的事件如果改成click就會變成點擊偵測了~
renderer.domElement.addEventListener("mousemove", (ev) => {
  const rect = renderer.domElement.getBoundingClientRect();
  // 透過把滑鼠的位置除以canvas寬度，來映射到垂直/水平 -1 ~ 1 的區間
  cursor.x = ((ev.clientX - rect.left) / rect.width - 0.5) * 2;
  cursor.y = -((ev.clientY - rect.top) / rect.height - 0.5) * 2;
  // 因為clientY和Three.js的坐標軸方向相反，所以記得要乘以-1
});
```


### 在每一個Tick Loop的循環都對所有的Mesh做檢測

我們接著要對所有的方塊做滑鼠懸停的偵測，假如滑鼠移到某個**方塊**上，那我們就把那個**方塊**變成**綠色**的。

```javascript
const rc = new Raycaster();

const loop = (time) => {
  rc.setFromCamera(cursor, camera);
  const intersects = rc.intersectObjects(scene.children);
  // 這邊我們必須要先刷新一次Scene裡面所有方塊的顏色，把它洗回去初始狀態
  scene.children.forEach((o) => {
    if (o instanceof Mesh) {
      o.material.color.set(0xff0000);
    }
  });
  // 接著再去把有跟射線相交的方塊洗成綠色
  intersects.forEach((o) => {
    o.object.material.color.set(0x00ff00);
  });
  controls.update();
  renderer.render(scene, camera);
  requestAnimationFrame((time) => {
    loop(time);
  });
};

loop();
```
> 搭拉~

![img](https://i.imgur.com/inJHwJ0.gif)

> codepen連結:[點我](https://codepen.io/mizok/pen/zYjLRrZ)


## 小結

我們今天大致上介紹過了`Raycasting`還有`Raster`的用法~，而且今天也是**Three.js與滑鼠互動操作**章節的最後一篇了。但我們之後還有機會用到這個技術，大家可以再期待一下XD。