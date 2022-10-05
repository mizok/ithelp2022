# Day17 - Three.js與滑鼠互動操作(一)

> 這裡是「Three.js學習日誌」的第17篇，本篇主旨是在講解要怎麼讓`Three.js`與滑鼠的動作相聯繫，例如點擊、懸停、或是移動。這系列的文章假設讀者看得懂javascript，並且有Canvas 2D Context的相關知識。

由於`Three.js`畢竟還是前端的一環，專案上常常會需要根據使用者的操作狀況去渲染出對應的圖像，所以我打算接著來講講`Three.js`與滑鼠互動操作的實作方法。

`Three.js`與滑鼠互動操作的實作方法有很多，舉凡我們在前一個章節提到的利用`Orbit Control`來操作環形攝影機軌道，又或者是偵測物件點擊的`Raycasting`，接著我們會透過實作幾個範例，來學習這些技巧。

##  來自製一個陽春版本的OrbitControl吧

我們在前面一個章節有先偷偷地~~超車~~使用過了`OrbitControl`，但是實際上如果想要透過移動滑鼠來旋轉視角，方法可不只有`OrbitControl`一種。


我思考了一下，其實這個範例還蠻值得介紹的。我們可以在這個範例中學習到:

- `OrbitControl`的大略原理
- 如何自己客製化一個視角轉動特效

> 那就讓我們開始吧!
### 建立基本物件

我們這次先從一個只有一個`Torus mesh`(甜甜圈)的`Scene`開始，這邊還請容許我跳過初始化渲染環境的步驟XD。

![img](https://i.imgur.com/UfVi1CI.jpg)

```javascript
const geo = new TorusGeometry(1, 0.5, 50, 50);
//TorusGeometry 的參數依序是 外半徑/內半徑/半徑軸Segment/切面軸Segment
const mat = new MeshStandardMaterial({
  color: new Color("rgba(255,0,0,0.1)"),
  transparent: true
});
const mesh = new Mesh(geo, mat);
scene.add(mesh);

```

> codepen連結:[點我](https://codepen.io/mizok/pen/LYmrVWg)

### 使用`Helper`

其實`Three.js`有提供很多種輔助用的`Class`，`Helper`系的函數就是其中之一，`Helper`通常都是用來協助我們釐清畫面上物體/燈光/攝影機當前的位置。

這邊我們要使用的是`AxisHelper`，它可以幫助我們看清現在坐標軸的狀態。

這邊**綠軸**代表的是`Y`，**紅軸**則是`X`，**藍軸**因為正對我們所以看不到，它代表`Z`

![img](https://i.imgur.com/Rp6ynjP.jpg)

```javascript
const axis = new AxesHelper(5); // 參數代表的是XYZ軸的長度多少，這邊我給5單位
scene.add(axis)
```
### 設定攝影機與滑鼠的相對關係

我們要做的事情其實就是根據滑鼠在螢幕上:

- 水平移動的百分比
- 垂直移動的百分比

來決定要讓攝影機移動到**水平環形軌道**的哪一個角度上，還有**垂直方向軸**的哪一個位置上。

我們會把畫布的最左/下方定義為-1，最右/上則為+1。而當我們知道當前滑鼠是位於**水平/垂直環形軌道**上的哪一個位置，例如(+0.3,-0.5)，那我們就把它映射到-180~180度的範圍中，這邊如果用(+0.3,-0.5)來舉例，那就是讓攝影機位於水平軌道54度,垂直方向為距離坐標軸中心-0.5單位的地方。

![img](https://i.imgur.com/B7NTIW8.jpg)

這邊我們先利用`addEventListener`來動態把滑鼠座標記錄到變數中。
```javascript
const cursor = new Vector2(); // 建立一個Vector2來記錄滑鼠位置。
//把滑鼠移動事件綁在用來渲染畫面的canvas上面
renderer.domElement.addEventListener('mousemove', (ev) => {
  const rect = renderer.domElement.getBoundingClientRect();
  // 透過把滑鼠的位置除以canvas寬度，來映射到垂直/水平 -1 ~ 1 的區間
  cursor.x = ((ev.clientX - rect.left) / rect.width - 0.5) * 2;
  cursor.y = -((ev.clientY - rect.top) / rect.height - 0.5) * 2; 
  // 因為clientY和Three.js的坐標軸方向相反，所以記得要乘以-1
})
```
接著我們要做的事情就是在每一圈`tick loop`(`RequestAnimationFrame`的循環)中:

1. 把攝影機根據`cursor`的數值移動到**水平/垂直環形**軌道上對應的位置
2. 強制攝影機`LookAt`坐標軸原點`(0,0,0)`

```javascript
const railRadius = 5; //假設攝影機軌道半徑為5單位

const tick = () => {
    camera.position.x = Math.sin(cursor.x * Math.PI ) * railRadius;
    camera.position.z = Math.cos(cursor.x * Math.PI ) * railRadius;
    camera.position.y = Math.sin(cursor.y * Math.PI ) * railRadius;
    camera.lookAt(new Vector3(0,0,0));
    renderer.render(scene, camera);
    requestAnimationFrame(tick);
}
tick();
```

![img](https://i.imgur.com/9YDZ4yk.gif)


> 其實蠻簡單的對吧~?


## 小結

今天我其實不打算一下就跳太深，所以先點到這邊為止，明天開始會有稍微進階的一些攝影機操作~ 敬請各位期待。


