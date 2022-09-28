# Day11 - 「紋理&種類」- Material解密(二)  

> 這裡是「Three.js學習日誌」的第11篇，本篇的主旨是要介紹紋理的種類，這系列的文章假設讀者看得懂javascript，並且有Canvas 2D Context的相關知識。

我們在上一回提到了如何在`Three.js`使用`Texture`(紋理)，而我們今天是要來講講`Texture`(紋理)的種類。

## 紋理的種類

```javascript
async function main() {
  const someTexture1 = await getTexture("https://picsum.photos/id/237/200/300");

  const mat  = new MeshStandardMaterial({
     map:someTexture1
  })
} 

```

我們已經知道，如果想要把模型表面填上特定的圖樣，那就是要給予`Material`的`map`這個屬性對應的`Texture`。

但是若打開`Three.js`官方文件關於`MeshStandardMaterial`的[頁面](https://threejs.org/docs/#api/en/materials/MeshStandardMaterial)，我們可以發現`MeshStandardMaterial`除了`map`以外，還有`aoMap`，`displacementMap`，`normalMap`，`alphaMap`...等相似的屬性。

這些屬性其實就是我們今天要提的**紋理的種類**。

一般而言，以一個**PBR材質**的構成來看，常見的紋理通常會有:

- **Normal Map** (法線紋理)
- **Height Map** (凹凸紋理)
- **Metallic Map** (金屬化紋理)
- **Ambient Occulusion Map** (環境光遮蔽紋理)
- **Roughness Map** (粗糙紋理)
- **Alpha Map** (透明度紋理)
- **Color Map** (色彩紋理)

除了**Color Map** (色彩貼圖)就是我們之前用來賦值給`map`屬性的紋理之外，其餘的6種我們會在下面一併作介紹。

## **Normal Map** (法線紋理)

其實看到`Normal`這個詞應該就不難猜到: 這是一種用來幹麻的東西。

**Normal Map**其實就跟我們之前提到的`Normal attribute`是類似的概念。差別就在於:

- `Normal attribute`是透過計算**頂點法向量**去取得每個**三角面**的**法向量**。

- `Normal map`則是能用來<u>模擬</u>任何凹凸處光照效果的紋理，就算模型表面沒有實際的凹凸高低變化也可以，說白了它比較像是一種障眼法。

考慮到對於第一次聽到這個名詞的人來說，可能光看上面的敘述還是不太能理解`Normal map`的用途，所以這邊我準備了2張圖片。

|  ![img](https://i.imgur.com/yj6Lx7Z.gif) |  ![img](https://i.imgur.com/bsDTZwc.png) |
|---|---|


上面左圖是一個普通的方塊，但是四面都貼上了同一張**法線紋理**。

我們可以發現，在某些角度，方塊的表面看起來好像並不如想像中一樣有高低起伏的波紋，反而看起來像一個平面。

> 這其實就是一種障眼法，透過把特定的區塊加上高亮/陰影，來營造出好像平面上存在著高低差的這種錯覺。

### Three.js 中的法線紋理

在`Three.js`中要怎麼給3D模型加上法線紋理?

>其實就跟`Color Map`差不多

```javascript
  const tl = new TextureLoader();
  const normalTexture = tl.load('../img/normal.png'); 
  

  const mat  = new MeshStandardMaterial({
     normalMap:normalTexture
  })
```

### 法線紋理的圖片格式

法線紋理的圖片格式最好不要採用`jpg`，因為法線紋理的圖片通常有大量的純色/漸層色區塊，使用`jpg`可能會導致破壞性壓縮的問題，而使得貼圖看起來有瑕疵。


|以`PNG`儲存的Normal Map|以`JPG`儲存的Normal Map|
|---|---|
|  ![img](https://i.imgur.com/SAAavI1.png) | ![img](https://i.imgur.com/b5p3POz.jpg) |


## **Height Map** (凹凸紋理)

**Height Map** (凹凸紋理)其實和**Normal Map**是很像的東西，但差別就在於**Height Map**是真的會去改變一個平面上面，頂點的位置。

| **Height Map**範例 | **Height Map**會實際提高/降低`Geometry`的頂點位置 |
|---|---|
|  ![img](https://i.imgur.com/phjDpCv.png) | ![img](https://i.imgur.com/iREpFic.gif) |

### 凹凸紋理的原理

我們前面其實有提到過，當我們在初始化一個`BoxGeometry`的實例時，其實可以傳入第`4`/`5`/`6`個參數。

```javascript
const geo = new BoxGeometry(1,1,1,100,100,100)
```

這邊第`4`/`5`/`6`個參數代表的是**Segments**，意思也就是把平面分割成多少部分，分割線和分割線的交會處會形成新的頂點，而**Height Map**就是透過變更這些**Segments**頂點，來達到`Geometry`的變形。

> 也就是說，Segments數量越高，**Height Map**所帶來的細節就越清楚。

![img](https://i.imgur.com/1gHOdM8.png)

### Three.js 中的凹凸紋理

使用方法其實和**Normal Map**大同小異。

> 要注意在three.js中height map的屬性是被命名為`displacementMap`，而不是`heightMap`


```javascript
  const tl = new TextureLoader();
  const heightTexture = tl.load('../img/height.png'); 
  

  const mat  = new MeshStandardMaterial({
     displacementMap:heightTexture
  })
```


## **Metallic Map** (金屬化紋理)

**Metallic Map**金屬化紋理主要是用來標記一個平面上具有金屬性質的部位。

![img](https://i.imgur.com/UktQ0bC.jpg)

以上面這張圖來講，左邊是完全沒有加上**金屬化紋理**的樣子，右邊是**金屬化紋理**的圖片素材，中間則是把左右兩者合併到一起時的樣子。

### 金屬化紋理的原理

**Metallic Map**金屬化紋理的原理其實就在於提升模型指定的區塊的**反射率**，讓該區域具備能夠<u>大量反射環境光</u>的能力，使得區域自身的顏色變得不那麼明顯。

> 所以可以注意到像上面的頭盔圖片，在模型特定曲率的邊角，顏色特別接近白色。



### Three.js 中的金屬化紋理

注意不是`metallicMap`，而是`metalnessMap`。

```javascript
  const tl = new TextureLoader();
  const metallicTexture = tl.load('../img/metallic.png'); 
  
  const mat  = new MeshStandardMaterial({
    
     metalnessMap:metallicTexture
  })
```


## 小結

今天我們對於紋理種類的介紹就先到**Metallic Map** (金屬化紋理)告一段落。

明天我們會繼續補完剩下的紋理種類。


## 延伸閱讀


-[https://www.twblogs.net/a/5c339be9bd9eee35b21cea09](https://www.twblogs.net/a/5c339be9bd9eee35b21cea09)

