# Day14 - 金屬球範例試作(1) - Material解密(四)  

> 這裡是「Three.js學習日誌」的第14篇，本篇的主旨是要透過一個簡單的範例操作，來一步一步介紹材質與渲染的關係，這系列的文章假設讀者看得懂javascript，並且有Canvas 2D Context的相關知識。

到目前為止的系列文中，我們基本上只有使用過`MeshStandardMaterial`還有`MeshBasicMaterial`。所以接下來的2天我打算透過試做一個簡單的小場景，來學習材質的運用。

## 試著做一個放置在金屬面上的金屬球

這次我們先從一個只有`camera`的`Scene`開始。

> 為了節省篇幅，初始化的環節也是先略過了。

### 1. 先把球&平面都置入Scene當中

``` javascript
const geo = new SphereGeometry(1, 100, 100);
const planeGeo = new PlaneGeometry(20, 20, 20, 20);
const mat1 = new MeshStandardMaterial({color:new Color('#eee')});
const mat2 = new MeshStandardMaterial({color:new Color('#eee')});
const mesh = new Mesh(geo, mat1);
const planeMesh = new Mesh(planeGeo, mat2);
scene.add(mesh, planeMesh);
```
> 當然這時候還是黑一片，因為沒有光源

### 2. 來點光

```javascript
const pl = new PointLight(0xffffff, 1);
pl.position.set(3, 3, 3);
scene.add(pl);
```
![img](https://i.imgur.com/6aR4Boq.jpg)


### 3. 旋轉平面，調整球的位置

因為`PlaneGeometry`在一開始建立的時候會是直立的(而不是橫躺)，所以這邊我們必須要來點旋轉。

這邊我們可以直接使用`Object3D.lookAt`，這個方法其實就跟字面上的意思一樣，可以讓`Object3D`物件「看」向某個目標座標。這邊我們讓平面朝向(0,1,0)，也就是說面會朝上。

然後再稍微調整一下球的位置到(0,1,0)，讓它剛好落在平面上(半徑是1)

```javascript
planeMesh.lookAt(0,1,0);
mesh.position.set(0,1,0);
```

![img](https://i.imgur.com/KQyVknf.jpg)


### 4. 使用 OrbitControl

這邊我們又要來超前一下進度了。

因為現在平面是完全垂直於我們的螢幕，所以看起來像一條線。

但這樣實在太無趣，所以我們要給他來點**可控性**。

`OrbitControl`就是`環形攝影機軌道`的意思，`Three.js`有提供這樣的機能讓我們可以用滑鼠直接操作`camera`，讓我們可以從不同的角度去觀察我們渲染的Scene。

作法很簡單，首先我們要先建立`OrbitControl`的實例，接著把`camera`和`renderer.domElement`當作參數傳入。

這邊因為`cdn.skypack.dev`的`Three.js` Package沒有提供`OrbitControl`的Module，所以我們必須要引用別的Package。

> 正常來講`Three.js` 的npm package底下是可以找到`OrbitControl`的，我們之後會再提到。


``` javascript
import threeTsOrbitControls from 'https://cdn.skypack.dev/@three-ts/orbit-controls';

const controls = new threeTsOrbitControls.OrbitControls(camera, renderer.domElement)

```

接著我們得在tick loop(也就是`requestAnimationFrame`的迴圈)裡面呼叫`Controls.update`

```javascript
 let time = 0;
  const loop = (time) => {
    mesh.rotation.y = time / 1000;
    controls.update();
    renderer.render(scene, camera);
    requestAnimationFrame((time) => {
      loop(time);
    });
  };

  loop();
```

>這樣就可以動手旋轉畫面了。

![img](https://i.imgur.com/Qfi50nK.gif)

最後我們可以調整一下`camera`的位置，讓畫面不要看起來這麼無趣。

這邊其實有個小撇步，假如我在利用`OrbitControls`旋轉角度，感覺有某個角度特別中意的話，我們其實可以把`camera`暴露為全域物件。

``` javascript
 window.cm = camera;
```

這樣我們就可以先使用`OrbitControls`把畫面轉到我們喜歡的位置，然後用開發者工具印出來`camera`目前的位置，接著再把這些數據紀錄到程式裡面。

![img](https://i.imgur.com/TJU2EhP.jpg)

```javascript
camera.position.set(0.8182387794884614,3.0688847649716244,3.8616617665282886)
```

> 這樣攝影機就會以我們中意的位置作為初始狀態來呈現了~


### 5. 調整金屬化/粗糙度參數，然後調整一下光照

馬上來試玩一下~上一篇介紹的**金屬化/粗糙度**，這兩個屬性可以讓我們調整`Material`的反射率/漫反射率。

``` javascript
const mat1 = new MeshStandardMaterial({
    color: new Color("#eee"),
    metalness: 0.5,
    roughness: 0.3
});
const mat2 = new MeshStandardMaterial({
    color: new Color("#eee"),
    metalness: 0.2,
    roughness: 0.8
});
const mesh = new Mesh(geo, mat1);
const planeMesh = new Mesh(planeGeo, mat2);
...
const al = new AmbientLight(0xffffff, 1); //補一個環境光
const pl = new PointLight(0xffffff, 0.3); //減弱點光源的強度
scene.add(al);
```

![img](https://i.imgur.com/AI510uQ.jpg)


### 6. 動態渲染陰影

接著又是要來超前進度啦~

做到這邊為止雖然好像有越來越像一回事(?)

但各位大概會發現球有時感覺像是飄在空中的。

這是因為我們缺乏了**陰影**


在`Three.js`中，如果我們想要透過現有的光源來運算物體實時的陰影，那首先我們會需要做兩件事。

- **光源**&**會造成陰影的物體**必須要設定`castShadow`

- 上面會產生陰影的物體材質必須要設定`receiveShadow`

所以這邊我們必須給`PointLight`和**球**設定`castShadow`，並且給**平面**設定`receiveShadow`。

```javascript
mesh.castShadow = true;
...
planeMesh.receiveShadow = true;
...
pl.castShadow = true;
```

設定完`castShadow`/`receiveShadow`之後，接著則是要打開`renderer`的`shadowMap`設置，並且把`shadow type` 改為`PCFSoftShadowMap`

```javascript
renderer.shadowMap.enabled = true;
renderer.shadowMap.type = PCFSoftShadowMap // PCFSoftShadowMap記得要從module import
```

![img](https://i.imgur.com/fvYPBcT.jpg)

> 這樣就有影子了~

但是在這邊我們可以注意到，陰影的邊緣似乎有點瑕疵。

這個問題發生的原因其實是因為燈光的`shadow map`解析度不夠，所以我們要再調整一下。

```javascript
pl.shadow.mapSize.width = 2048;
pl.shadow.mapSize.height = 2048;
```

![img](https://i.imgur.com/pTkCulT.jpg)

> Perfect Shadow !



# 小結 

到這邊我們已經做出來一個簡單的小場景，明天我們會繼續用這個範例來介紹**環境貼圖**和一些我們目前尚未使用過的**材質**。