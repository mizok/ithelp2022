# Day7 - 「就像在玩摺紙一樣」 - 幾何結構Geometry(一)  

> 這裡是「Three.js學習日誌」的第7篇，本篇的主旨是要介紹Geometry的概念，還有一些常用的Geometry子類的使用方法。這系列的文章假設讀者看得懂javascript，並且有Canvas 2D Context的相關知識。

我們在前面有稍微介紹過**Geometry**(幾何結構)大概是個甚麼樣的概念。

除了`BoxGeometry`(**箱型幾何結構**)以外，`three.js`還內建了很多可以開箱即用的`Geometry`子類。像是我們之前就有示範過的`BoxGeometry`(方塊)，還有`ConeGeometry`(圓錐)，`SphereGeometry`(球體)。 

## BufferGeometry是什麼?

為什麼我們會說`BoxGeometry`(方塊)，`ConeGeometry`(圓錐)，`SphereGeometry`(球體)這些東西是`Geometry`子類呢?

那是因為，大多數`three.js`的內建`geometry`都是`BufferGeometry`這個類的延伸。

但`BufferGeometry`是什麼? 其實看到`Buffer`這個字眼應該就有人聯想到了，他其實就是與`webgl`中用來供給`shader attribute`取用數值的那個`Buffer`。

所以我們其實也可以像之前在`webgl hello world`範例中做的一樣，把頂點的座標傳入`BufferGeometry`，用來生成**形狀**。

---

### 這邊我們稍微來點範例

> 這個範例是打算要在Scene裡面，透過BufferGeometry繪製一系列隨機的三角形


![img](https://i.imgur.com/uJNz55v.gif)


1. 首先先建立`BufferGeometry`的實例。

```javascript
//這邊我們忽略掉Scene實例和一些初期渲染環境的設置流程，直接來到展示的重點

const geo = new BufferGeometry();

```

2. 先產生一系列隨機的點座標。

```javascript
...
const vertexNumber = 100; //假設有100個座標
let arr = new Float32Array(vertexNumber * 3); //每個座標都有xyz
arr = arr.map((ele) => {
  return Math.random() * 2 - 1;
});
```

4. 利用這些點座標去生成一個`BufferAttribute`的實例，然後把這個實例轉嫁給我們前面建立好的`BufferGeometry`實例。

```javascript
...
const ba = new BufferAttribute(arr, 3);//這邊填3是有點類似stride的概念，也就是每三個算一組
// 
geo.setAttribute("position", ba);

```


5. 上材質之後生成`Mesh`，再來丟進Scene裡面，打完收工。

```javascript
// 這次我們選用MeshBasicMaterial，這是一種不會受到光源影響的Material，可以很直接地呈現出顏色
  const mat = new MeshBasicMaterial({
    color: new Color("red"),
    transparent: true
  });
  const mesh = new Mesh(geo, mat);
  scene.add(mesh);

```

> Codepen 連結 :[點我](https://codepen.io/mizok/pen/qBYPpNz)



## BufferGeometry的幾個重要屬性:normal/position/uv

我們剛剛提到了如果要把頂點的座標資料轉嫁給`BufferGeometry`，我們會需要利用`BufferAttribute`這個類。

```javascript
const ba = new BufferAttribute(arr, 3);
geo.setAttribute("position", ba);
```

這邊我們可以發現我們是把座標資料傳進去`BufferGeometry`底下一個叫做"position"的屬性去了。

這邊我們其實可以打開剛剛範例中的`BufferGeometry.attribute`來瞧瞧，看看裡面到底都有些甚麼樣的東西~

這邊我們可以看到`BufferGeometry.attribute`裡面就只有一個"position"這樣的屬性，裡面的值也就是我們剛剛轉嫁進去的頂點座標資料。

![img](https://i.imgur.com/mN5noUv.png)

接著再讓我們來看看之前我們做的`three.js Hello World`，裡面用到的`BoxGeometry`，有沒有甚麼不一樣的結果。

![img](https://i.imgur.com/JFCxLu6.png)


我們可以發現這邊多出了兩個東西，分別是

- `normal`

- `uv`

`BoxGeometry`之所以會有這兩個`attribute`，原因是因為他本身是一種開箱即用的`Geometry`。

`normal`其實就是我們高中都學過的**平面法向量**，他的用途是在定義一個多面體的每個三角形面的**朝向**為何。`normal`是決定一個3D物件的受光狀況的重要條件之一。

`uv`這個詞，相信只要有摸過3D建模軟體的人應該都很熟悉。

相信很多人都知道**貼圖**這個概念。很多的3D物件，從外表上看起來有很多的細節，但是實際上那些都只是一些**材質貼圖**形成的假象。

![img](https://i.imgur.com/qLDc5dq.jpg)

> 這個貼了磚塊皮的方塊，只要不從側面看，要看出來是貼皮其實有點難度

而通常我們在3D軟體中，我們會把像上面這樣的材質貼圖，合併為一整張圖片。


![img](https://i.imgur.com/8meZKGH.png)


這時候`UV mapping`就是一個很重要的概念，他代表一張**材質貼圖**上面，每一塊小部位，實際是要映射到模型上的"**哪個位置**"。


而我們在上面"attribute"中看到的"uv attribute"其實就是這樣的概念，他決定了顏色/圖樣附著在模型上的Mapping。


## 小結 

`Geometry`的內容還沒有結束，我大約預計至少還要再花一天的時間才能寫完預期的內容，明天我們可以來講講一些如何客製`Geometry`的內容，今天就先到這邊結束。

## 延伸閱讀


- [https://www.twblogs.net/a/5eef515933cbe858769e6bd4](https://www.twblogs.net/a/5eef515933cbe858769e6bd4)