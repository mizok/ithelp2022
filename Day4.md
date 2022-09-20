> 這裡是「Three.js學習日誌」的第4篇，本篇的主旨是透過試做一個Three.js的Hello World案例，來讓讀者對於Three.js有基本的認識，這系列的文章假設讀者看得懂javascript，並且有Canvas 2D Context的相關知識。

> **前情提要**: 昨天因為出了一點小意外，原本該一天處理完的Three.js Hello World篇，寫著寫著就不小心寫到了半夜兩點...，所以這邊為了要符合賽程規定，我決定把昨天超出12點之後撰寫的內容，轉移至今天的篇章，如造成評審不便，深感抱歉!

---

> 第5步前的內容可至[這裡](https://ithelp.ithome.com.tw/articles/10294300)觀看。

### 6. 生成方塊的Mesh(網格)，並加入Scene中

接著馬上又是一個看不懂的專有名詞**Mesh**。
所謂的`Mesh`其實就是**3D模型**，一個正常的`Mesh`會由`Geometry`(幾何結構)和`Material`(材質)組成。

- `Geometry`內部紀錄的是構成3D物體的每個表面頂點的**座標**。
- `Material`則是3D物體表面的圖樣。

> 其實這邊大概就可以聯想到跟`vertex shader`和`fragment shader`有關了。

跟前面一樣，`Geometry`和`Material`其實都有多種類型，這邊因為我們要生成一個方塊，所以選用`BoxGeometry`，`BoxGeometry.constructor`的前三個參數也就是`長`，`寬`，`高`，而後三個則是**xyz軸網格細分數量**，**網格細分數量**會影響一個**Mesh**到底有多細緻，**網格細分數量**越大。一個面上的頂點(vertex)就會越多，同時也會吃越多效能。

`Material`的部分我們選用`MeshStandardMaterial`，這是一種最基本的**PBR**材質，他可以接收的到光照的變化，而產生不同的顏色，順帶一提，如果環境中沒有光，則會呈現黑色。

> 延伸閱讀: [什麼是PBR?](https://www.twblogs.net/a/5ed85c944eab53eb1c5bc9aa)

最後我們把`Geometry`和`Material`傳進去`Mesh`生成實例，再把`Mesh`實例加進去Scene裡面。

```javascript
import {
  Scene,
  WebGLRenderer,
  BoxGeometry,
  PerspectiveCamera,
  MeshStandardMaterial,
  mesh
} from "https://cdn.skypack.dev/three";

function main() {
  ...

  const geo = new BoxGeometry(1, 1, 1, 10, 10);
  const mat = new MeshStandardMaterial({
    //這裡可以設置方塊的顏色
    color: 0xff0000
  });
  const mesh  = new Mesh(geo,mat);

  scene.add(mesh)
  
}

main();

```

### 7. 在Scene裡面加入光源，並且調整一下攝影機的位置

因為我們在前一步使用的是`MeshStandardMaterial`，所以環境中必須要有光，不然攝影機只會照出黑色的方塊。

在這邊我們給Scene加上一盞**點光源**(PointLight)。

```javascript
const pl = new PointLight(0xffffff, 0.3);
  pl.position.set(3, 3, 3);
  scene.add(pl);
```

接著我們再調整一下攝影機的位置，因為現在攝影機的位置是0，這樣方塊會因為不在**近平面/遠平面**構成的空間中，而渲染不出來，所以我們稍微拉遠一點攝影機的距離。

```javascript
camera.position.z = 5;
```

### 8. 發動render方法試試看

```javascript
renderer.render(scene, camera);
```

燈燈~ 我們的方塊就成功渲染出來啦。

![img](https://i.imgur.com/z0asRGJ.png)



### 9. 要怎麼讓方塊動起來?

其實就跟`2D context`差不多，只要在每一個`requestAnimationFrame`的循環重新發動render方法，並且同時改變`Mesh`的`rotation`屬性就可以。

```javascript
  let time = 0;
    const loop = (time) => {
      mesh.rotation.y = time / 1000;
      renderer.render(scene, camera);
      requestAnimationFrame((time) => {
        loop(time);
      });
    };

    loop();
```

## 小結

在這一篇中，我們只是先用一個最基本的案例來解釋`three.js`的渲染流程，但其實`Camera`，`Geometry`，`Material`,...etc. 都還有很多細節需要補充，所以我預計要在後面的天數一點一點地把他們補齊。

## 延伸閱讀

- [什麼是PBR?](https://www.twblogs.net/a/5ed85c944eab53eb1c5bc9aa)
- [三維透視投影](https://ithelp.ithome.com.tw/articles/10281029)
