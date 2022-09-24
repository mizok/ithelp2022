# Day6 - 事前健身操 - 顏色/動畫循環/群組

> 這裡是「Three.js學習日誌」的第6篇，本篇的主旨是要介紹一些在Three.js中，一些常用的基礎操作，這系列的文章假設讀者看得懂javascript，並且有Canvas 2D Context的相關知識。


相對於前幾天的內容來講，今天的內容應該算偏簡單。

今天預計要來寫完剩下的一部分基本操作，那麼就讓我們開始吧~ 


## 顏色

```javascript
const color = 0xff0000;
// 猜猜我是誰?
```

我猜大概也有人跟小弟我一樣，第一次看到`three.js`中顏色的寫法時感覺很問號。

`0xff0000`，這個到底是甚麼格式的顏色表示方法? Hex Color不都應該是**字串**並且是"#"開頭的嗎?

這種用`0x`作為前綴的格式，其實就是**十六進制計數法**，也就是跟`CSS`的`#ff0000`是一樣的東西。

> 至於為什麼十六進制是用0x當前綴，可以看看這篇[Stackoverflow](https://stackoverflow.com/questions/2670639/why-are-hexadecimal-numbers-prefixed-with-0x)

當然，`Three.js`要表示一個顏色，除了可以用**十六進制計數法**以外，也可以用`Color`這一個類。

>畢竟不是每個人都可以手算十六進位然後再腦袋裡面假想顏色XD

`Color`可以接受普通的**色彩名**，**RGB**，**HSL**，當然也可以用我們熟悉的**Hex**。

> 疑? 但卻沒有支援RGBA嗎?

對的，實際上這一點在官方的`Repo`上面也有提出[討論](https://github.com/mrdoob/three.js/issues/6014)。官方的回應是說，`three.js`在早期階段其實是有支援`rgba`的，但是後來因為開發人員認為，讓每個頂點都儲存ALPHA數據資料實在浪費太多GPU內存空間，所以從某一版本之後就會自動ignore掉`Color`的alpha數值。

>如果不小心在`Color`裡面使用`rgba`，`three.js`還會特別提醒你alpha被drop掉了XD

![img](https://i.imgur.com/Y2LxZMB.png)


但接著可能就會有人有疑問: **那這樣我要怎麼讓`three.js`產生帶有透明度的顏色呢?**

這邊我們有兩種辦法:

- 直接在Material上面把`transparent`這個屬性設置為`true`，接著就可以透過調節`opacity`這個浮點數屬性來控制透明度。

- 使用`Alpha Map`，`Alpha Map`是一種圖片材質的統稱，用來表示一張材質透明的區塊，這個在後面會再提到。

```javascript
const mat = new MeshStandardMaterial({
  color:new Color('red'),
  transparent:true,
  opacity:0.5
 // 這樣就會拿到一個半透明紅色的材質
})
```

另外，`Color`作為一種長得很像Vector的型別，它其實也有提供`.lerp`方法，也就是顏色的內插。


``` javascript
Color.lerp(Color,float)
```

我們可以指定另外一個顏色作為**端點**，然後再指定一個浮點數，例如:

``` javascript
const red = new Color('red');
const yellow = new Color('yellow');
const orange = yellow.clone().lerp(red,0.5);
// 這樣就可以get橘色

```
這其實是一個很方便的功能，我們這樣就可以透過改變傳入的浮點數值來作出顏色的漸變。

不過要特別注意，`.lerp`不是一種`pure function`，所以如果要回傳一個全新的Color物件，請務必要`Clone`(如果你不想像小弟我一樣花上整整一個小時抓bug 的話XD)。

## 動畫循環

我們前面有提到過，`three.js`的動畫機制其實就跟`2D Context`差不多。

```javascript
let time = 0;
const loop = (time) => {
  renderer.render(scene, camera);
  requestAnimationFrame((time) => {
    loop(time);
  });
};

loop();
```

不過我這邊其實是想要介紹一下`Clock`這一個類。

`Clock`顧名思義就是一個計時器，他的底層是用`performance.now()`來實現的。

`Clock`把我們常常需要自己寫的"取得總經過時間"和"取得幀間時差"包裝成了兩個方法：

- `getElapsedTime`
- `getDelta`

所以我們其實可以把上面的`requestAnimationFrame`範例改成像這樣:

```javascript
const clock = new Clock();
const loop = () => {
  const time = clock.getElapsedTime();
  renderer.render(scene, camera);
  requestAnimationFrame(() => {
    loop();
  });
};

loop();
```

> 不過老實說其實兩種寫法沒太大差異就是了 XD


## 群組

**群組**也就是**Group**，它的用途就是可以讓你把一組`Object3D`型別的物件包起來，變成一個**Group**。 順帶一提，**Group**本身的型別也是`Object3D`(所以可以Group包Group)。



```javascript
const cubeA = new THREE.Mesh( geometry, material );
cubeA.position.set( 100, 100, 0 );

const cubeB = new THREE.Mesh( geometry, material );
cubeB.position.set( -100, -100, 0 );

//create a group and add the two cubes
//These cubes can now be rotated / scaled etc as a group
const group = new THREE.Group();
group.add( cubeA );
group.add( cubeB );
```

之所以使用**Group**的好處就是，被包在子層的物件，他的形變會受到父層級的影響，例如假設子層級的歐拉角旋轉為(0,0,Pi)，這時候我們把它包在一個具有歐拉角旋轉為(0,Pi,0)的父層Group裡面，這個子層級的物件，最後看上去就會像是實際旋轉了(0,Pi,Pi)的角度。

運用這樣的特性，我們其實就可以做到「**把物體的旋轉軸分離在不同層級物件屬性上**」，這樣在做多物件形成的部件時，就能夠更加直觀的操作形變。

> 雖然這邊只有文字的描述，但是我們在之後進到實際的範例演練後，大家應該就更能理解我的意思。


## 小結

基本操作的部分差不多到這邊先告一段落，接下來我們會來進入比較重點的`Geometry`，`Material`的介紹，中間也會穿插一些案例的演練，敬請期待 :D


## 延伸閱讀

- [https://github.com/mrdoob/three.js/issues/6014](https://github.com/mrdoob/three.js/issues/6014)

- [https://stackoverflow.com/questions/2670639/why-are-hexadecimal-numbers-prefixed-with-0x](https://stackoverflow.com/questions/2670639/why-are-hexadecimal-numbers-prefixed-with-0x)