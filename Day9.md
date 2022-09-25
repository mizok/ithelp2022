# Day9 - 「點點到位」 - 幾何結構Geometry(三)  

> 這裡是「Three.js學習日誌」的第9篇，本篇的主旨是要介紹Geometry的概念，還有一些常用的Geometry子類的使用方法。這系列的文章假設讀者看得懂javascript，並且有Canvas 2D Context的相關知識。

在上一篇中我們提到了要怎麼去定義一個正四面體`Geometry`的面(faces)還有`uv`，而這篇我們則是要看看:怎麼樣從只有**頂點座標**的狀態，建立一個`Geometry`。

## 以點構成空間

我們之前有展示過透過直接設定`position attribute`的方式來產生隨機三角形，而這次我要來給大家講講`BufferGeometry.setFromPoints`這個方法。

`BufferGeometry.setFromPoints`這個方法跟`setAttribute`的做法最主要的差別就只有`setAttribute`是必須傳入一整個包含`Float32Array`陣列內容的`BufferAttribute`，而`BufferGeometry.setFromPoints`則是可以直接傳入一個點座標陣列。

> 這邊用表格表示，釐清一下差異。

|   | 泛用性  |  傳入型別 | 直觀程度  | 
|---|---|---|---|
| `setAttribute`  | 較高  | `BufferAttribute`  | 較低  | 
|  `setFromPoints` |  較低 |  `Vector3[]` | 較高  | 

>`BufferGeometry.setFromPoints`在把座標傳入`BufferGeometry`之後，最終其實也是一樣會把值傳遞到`position attribute`，所以結果沒甚麼差異。

### 從零建立正立方體

這邊我們示範一下要怎麼用`BufferGeometry.setFromPoints`來排出一個正立方體。

> 這是一個中心位於(0,0,0)的立方體

![img](https://i.imgur.com/jmbA6dU.png)

我們先在程式裡面把所有的頂點列出來。

```javascript
//頂點集
const pts = [
    new Vector3(-0.5, 0.5, -0.5),//A
    new Vector3(0.5, 0.5, -0.5),//B
    new Vector3(-0.5, 0.5, 0.5),//C
    new Vector3(0.5, 0.5, 0.5),//D
    new Vector3(-0.5, -0.5, -0.5),//E
    new Vector3(0.5, -0.5, -0.5),//F
    new Vector3(-0.5, -0.5,0.5),//G
    new Vector3(0.5, -0.5, 0.5),//H
  ];
```

接著，因為我們之前在`webgl hello world`也有講過，`webgl`<u>沒有</u>提供四邊形的Primitive([參考連結](https://ithelp.ithome.com.tw/articles/10293500))，所以每個面我們都需要用2個三角形去組合出來。


在這邊組合三角形成為一個平面就是一個**重點**的地方，因為三角形的頂點排列順序會決定:「這個三角形面是**正面**，還是**反面**」。

`Three.js`預設是不會渲染一個**平面**的**反面**的，不過這個其實也可以在`Material`的設置中改為**雙面渲染**(DoubleSide)。

```javascript
const mat = new MeshBasicMaterial({
  color:0xff0000,
  side:DoubleSide //要注意DoubleSide是一個常數，而不是字串，必須從three.js的module中引入
})
```

上述**雙面渲染**(DoubleSide)的例子只是先提供給大家參考，這邊我們還是維持只渲染**正面**(FrontSide)的操作。

所以我們必須要先理解，怎麼樣的三角形頂點排列**規則**，才能使該**三角面**被`Three.js`定義為**正面**。

這邊我們要講的規則就是所謂的**Winding Rule**(纏繞規則)。

![img](https://i.imgur.com/Gn8XOEj.png)

> 延伸閱讀: [OpenGL上關於winding rule的解釋](https://www.khronos.org/opengl/wiki/Face_Culling)

這邊我畫了一張簡單的圖(下圖)來讓讀者理解，假設現在Scene裡面有一個平面，他是由[ABC]和[BCD]這兩個三角形構成的，[ABC]的纏繞順序是A>B>C，而[BCD]的纏繞順序則是B>C>D，以`Three.js`的定義來講，被螢幕前的觀者定義為**逆時鐘**的纏繞順序會判定為**正面**，反之則是**反面**。

> 所以ABC在這邊算反面，按照預設他是不會渲染出來的，除非觀者旋轉這個平面，從平面的後面看。

![img](https://i.imgur.com/D2hVOm6.png)


所以在這邊如果我們要做出一個會正確顯示所有**面**的正立方體，並按照我們剛剛的**頂點集**規劃，我們必須要像這樣去排列。

>所有當前看的到的面，都應該要是逆時鐘的纏繞順序，看不見的面則反之。

![img](https://i.imgur.com/GMnnQYR.png)

```javascript
const points = [
    pts[0].clone(), //A
    pts[2].clone(), //C
    pts[1].clone(), //B
    //
    pts[1].clone(), //B
    pts[2].clone(), //C
    pts[3].clone(), //D
    //ABC+BCD這樣算一個面
    pts[0].clone(), //A
    pts[1].clone(), //B
    pts[4].clone(), //E
    //
    pts[1].clone(), //B
    pts[5].clone(), //F
    pts[4].clone(), //E
    //ABE+BFE
    pts[4].clone(), //E
    pts[5].clone(), //F
    pts[6].clone(), //G
    //
    pts[5].clone(), //F
    pts[7].clone(), //H
    pts[6].clone(), //G
    //EFG+FHG
    pts[2].clone(), //C
    pts[6].clone(), //G
    pts[3].clone(), //D
    //
    pts[3].clone(), //D
    pts[6].clone(), //G
    pts[7].clone(), //H
    //EFG+FGH
    pts[0].clone(), //A
    pts[6].clone(), //G
    pts[2].clone(), //C
    //
    pts[0].clone(), //A
    pts[4].clone(), //E
    pts[6].clone(), //G
    //ACG+AEG
    pts[1].clone(), //B
    pts[3].clone(), //D
    pts[7].clone(), //H
    //
    pts[1].clone(), //B
    pts[7].clone(), //H
    pts[5].clone() //F
    
  ];

  geo.setFromPoints(points);
```

![img](https://i.imgur.com/BEuRwUg.gif)

> codepen連結:[點我](https://codepen.io/mizok/pen/MWGObax)

### 為正立方體做面分組

接著也許你會想像我們上一篇做的一樣，把這個正立方體六個面都填上不同的材質。

那你就會碰到上一篇也碰過的`group`問題，畢竟這個方塊是從0建立出來的，裡面的頂點沒有做任何的**面**分組，而且`uv`也沒有給，同樣也沒有`normal`，所以等於是剩下的東西都要自己建立出來。

所以接著我們先來完成`group`的部分。


```javascript
geo.addGroup(0,6,0)
geo.addGroup(6,6,1)
geo.addGroup(12,6,2)
geo.addGroup(18,6,3)
geo.addGroup(24,6,4)
geo.addGroup(30,6,5)
```

然後把多重材質套上去。


```javascript
const mats = [
    new MeshBasicMaterial({ color: new Color("red") }),
    new MeshBasicMaterial({ color: new Color("yellow") }),
    new MeshBasicMaterial({ color: new Color("orange") }),
    new MeshBasicMaterial({ color: new Color("brown") }),
    new MeshBasicMaterial({ color: new Color("blue") }),
    new MeshBasicMaterial({ color: new Color("purple") })
  ];

  const mesh = new Mesh(geo, mats);
```

![img](https://i.imgur.com/cZ5DVCU.gif)


### 定義uv映射


再來是`uv`，其實也沒甚麼特別的，就是照著剛剛`setFromPoints`的順序去決定`uv`映射的狀況。

```javascript
geo.setAttribute(
    "uv",
    new Float32BufferAttribute(
      [
        0,1, //A
        0,0, //C
        1,1, //B
        1,1, //B
        0,0, //C
        1,0, //D
        //
        1,1, //A
        0,1, //B
        1,0, //E
        0,1, //B
        0,0, //F
        1,0, //E
        //
        1,1, //E
        0,1, //F
        1,0, //G
        0,1, //F
        0,0, //H
        1,0, //G
        //
        0,1, //C
        0,0, //G
        1,1, //D
        1,1, //D
        0,0, //G
        1,0, //H
        //
        0,1, //A
        1,0, //G
        1,1, //C
        0,1, //A
        0,0, //E
        1,0, //G
        //
        1,1, //B
        0,1, //D
        0,0, //H
        1,1, //B
        0,0, //H
        1,0 //F
      ],
      2
    )
  );

 //把材質換成圖片材質，這樣才可以看出來uv有沒有錯誤狀況
  const tl = new TextureLoader();
  const mats = [
    new MeshBasicMaterial({
      map: tl.load("https://picsum.photos/seed/123/picsum/300/300")
    }),
    new MeshBasicMaterial({
      map: tl.load("https://picsum.photos/seed/456/300/300")
    }),
    new MeshBasicMaterial({
      map: tl.load("https://picsum.photos/seed/789/300/300")
    }),
    new MeshBasicMaterial({
      map: tl.load("https://picsum.photos/seed/012/300/300")
    }),
    new MeshBasicMaterial({
      map: tl.load("https://picsum.photos/seed/345/300/300")
    }),
    new MeshBasicMaterial({
      map: tl.load("https://picsum.photos/seed/678/300/300")
    })
  ];
```

![img](https://i.imgur.com/u6sdhW8.gif)

>codepen 連結: [點我](https://codepen.io/mizok/pen/YzLEQVO)


### 定義各頂點法向量

最後是`normal`。

我們在上一回沒有實作到`normal`的部分，我們在這邊先介紹一下。

在`three.js`中，因為`attribute`的值都是by頂點去儲存的，所以沒有辦法直接定義某個面的法向量，反而是必須要取該面所有頂點的**法向量**平均。

>計算法向量平均這一部分Three.js會自己完成，我們只需要給定每一個頂點的法向量就好~

![img](https://i.imgur.com/EZTENlS.png)

而我們因為在初期已經把正立方體的中心定在`(0,0,0)`了，而且我們要作的模型是一個正立方體。

所以這邊我們其實可以把`(0,0,0)`到每個頂點座標所形成的向量，先轉變成**單位向量**之後，把這些**單位向量**當作頂點法向量儲存到`normal attribute`中。

除此之外， 記得還要在Scene裡面補上一盞光源，並且把`MeshBasicMaterial`換成`MeshStandardMaterial`，這樣我們才能看到**材質**對**光源**產生反應。

```javascript
const nPoints = points.map((o) => {
    return o.clone().normalize();
  });

  const normalArr = [];

  nPoints.forEach((o) => {
    normalArr.push(o.x);
    normalArr.push(o.y);
    normalArr.push(o.z);
  });

  geo.setAttribute("normal", new Float32BufferAttribute(normalArr, 3));

  ...

  const pl = new PointLight(0xffffff, 1);
  pl.position.set(2, 2, 2);

  scene.add(mesh, pl);

```

![img](https://i.imgur.com/73pV6ZD.gif)

>codepen連結:[點我](https://codepen.io/mizok/pen/dyeZzZb)

## 小結

今天我們提到了如何從只有**點座標**到建立完全的**3D模型**，大家從過程中應該就可以理解到像這樣徒手建立一個新的`Geometry`其實非常的花時間。

在正常狀況下，大多數的建模都是透過3D建模軟體直接操作，不會像這樣一個座標一個座標慢慢處理。

不過個人是覺得能有像這樣自己動手作的經驗還蠻不錯的XD，希望大家喜歡今天的介紹。

## 延伸閱讀

-[https://zh.m.wikipedia.org/zh-tw/%E9%A0%82%E9%BB%9E%E6%B3%95%E5%90%91%E9%87%8F](https://zh.m.wikipedia.org/zh-tw/%E9%A0%82%E9%BB%9E%E6%B3%95%E5%90%91%E9%87%8F)

-[https://www.khronos.org/opengl/wiki/Face_Culling](https://www.khronos.org/opengl/wiki/Face_Culling)

