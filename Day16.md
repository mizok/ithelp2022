# Day16 - 金屬球範例試作(3) - Material解密(六)  

> 這裡是「Three.js學習日誌」的第16篇，這是Material章節的最後一篇，主旨是在講解`Three.js`所提供的其餘材質，這系列的文章假設讀者看得懂javascript，並且有Canvas 2D Context的相關知識。

在上一回我們成功的把金屬球&金屬平面做出來了~ 但由於我們之前都只有使用過`MeshStandardMaterial`和`MeshBasicMaterial`，所以我想說趁著這個機會來介紹一下`Three.js`所提供的其他材質。

### 9. 最後的最後來試試看MeshStandardMaterial以外的材質

`Three.js`所提供的`Material`類其實不少，這邊我做了一個列表來釐清各材質之間的差異。

|材質名稱|介紹|對光源反應|
|---|---|---|
|`MeshBasicMaterial`|可以支援填入顏色的最簡單材質|無|
|`MeshStandardMaterial`|最常用的標準PBR材質|有|
|`MeshMatcapMaterial`|不接受光源, 但是它具備特別的UV運算模式，可以透過傳入特殊的球狀紋理，來把模型渲染得像是有受光的樣子|無|
|`MeshLambertMaterial`|基於非物理的Lambertian模型來計算反射率，適合用來模擬一些未加工的木材或石材，除此之外，效能的消耗相較於`MeshStandardMaterial`來的低一些|有|
|`MeshPhongMaterial`|基於非物理的Blinn-Phong模型來計算反射率，具有反射光點的設計的可受光材質, 耗能比Lambertian略多，適合用來模擬鏡面材質|有|
|`MeshNormalMaterial`|透過模型各處的法向量，直接Mapping到RGB值的材質|無|
|`MeshToonMaterial`|近年常見的三渲二技術所使用的材質，可受光，但是色彩上較為不同|有|


這邊讓我們依序來試試`MeshMatcapMaterial`、`MeshNormalMaterial`和`MeshToonMaterial`。

#### 9-1 MeshMatcapMaterial 是什麼?

`Matcap`這個詞的意思其實是「**材質捕捉**」。

平常我們在`3D建模軟體`中，把模型做好之後，給它打上特定的光源，並加上一系列的材質修改係數，這些渲染到模型上的色彩是動態(Dynamic)的，它們會根據光源的位置、材質係數的不同而有不同的成色。

講到這邊大概就有人會想到:那有沒有一種可能，就是可以把這些渲染到模型上的色彩「皮」，剝下來變成一套靜態的資源呢?

當然是有的，這個手段在`3D建模`的領域中就叫做「Bake」。

透過「Bake」，我們其實就可以在無光照的環境下實現「模型看起來好像有受光」的假象，這麼一來程式對於硬體的需求度就可以降低。

>要知道實時渲染的光照/陰影是非常消耗效能的

`MeshMatcapMaterial`的原理其實就是把通過「Bake」獲得的材質球紋理，映射到模型上面。

![img](https://i.imgur.com/16HXhRY.png)

> 上面這張圖很好的解釋了Matcap的原理。

在`Three.js`中，若想要使用`MeshMatcapMaterial`，那就必須要先有`Matcap`素材。

想要自己製作`Matcap`素材，通常是需要使用3D建模軟體去「Bake」的，但這邊我們先去網路上找可以免費下載的資源。

> 例如這裡:[點我](https://www.deviantart.com/sespider/art/163-FREE-MatCaps-258893793)

這邊我們用這張素材來試試看。

![img](https://i.postimg.cc/L5gYZjVk/Genetic-View-Light-Green-Jade1a.png)

`Matcap`紋理的載入不需要額外的`Loader`，直接使用`TextureLoader`就可以。這邊我們先載入素材。

```javascript
const matcapTexture = tl.load(
  "https://i.postimg.cc/L5gYZjVk/Genetic-View-Light-Green-Jade1a.png"
);
```
接著我們把原本**金屬球/金屬面**的材質都換成`MeshMatcapMaterial`。

```javascript
const mat = new MeshMatcapMaterial({
    matcap: matcapTexture
});
```

![16-1.jpg](https://i.postimg.cc/CxMRYyqp/16-1.jpg)

> 其實感覺不錯對吧~

> codepen連結:[點我](https://codepen.io/mizok/pen/BaxxwEW)


#### 9-2 MeshNormalMaterial 是什麼?

之所以把`MeshNormalMaterial`放在第二順位介紹，是因為它其實跟`MeshMatcapMaterial`是很相像的東西。

我們剛剛有提到`MeshMatcapMaterial`是把`Matcap`素材映射到物體表面，但`MeshNormalMaterial`則是把RGB色域整個映射到物體表面。

通常這種材質只是拿來檢視3D模型每一個面的狀況，避免模型因為使用同一個顏色作為材質而分不清面的朝向。

> 通常這種材質比較少使用在以視覺/美術為主打特色的產品中。

這邊我們就簡單的示範一下。


```javascript
const mat = new MeshNormalMaterial({
});
```

![img](https://i.imgur.com/D2RVMFT.jpg)

>codepen 連結:[點我](https://codepen.io/mizok/pen/oNddoBb)


#### 9-3 MeshToonMaterial 是什麼?

近年來，**三渲二**已經逐漸成為了一種成熟的渲染風格。之所以叫做**三渲二**，是因為透過這種方法渲染出來的模型，看起來就像2D動畫片中的圖像一樣。

![img](https://i.imgur.com/CSc9ta2.jpg)


**三渲二**這種技術目前在動畫產業已經被大幅的使用，它的優點是可以減少很多人工作畫的技術門檻(例如鏡頭旋轉/透視畫面的cut)


而在`Three.js`中，我們其實可以透過`MeshToonMaterial`做出類似的效果


這邊我們馬上來試試看~

```javascript
const mat = new MeshToonMaterial({
    color:0xff0000 //隨便給個紅色
});
```

![img](https://i.postimg.cc/85Rhtjng/16-5.jpg)

> 哪尼? 阿說好的光照和陰影咧? 怎麼變得跟`MeshBasicMaterial`一樣?

當我們把`MeshToonMaterial`套到我們的**球和平面**上面時，會發現陰影整個不見了。這是因為我們的`AmbientLight`強度給的太高了。

所以這邊我們削弱`AmbientLight`的強度~

```javascript
const al = new AmbientLight(0xffffff, 0.2);
```

![img](https://i.postimg.cc/jdbZyr27/16-6.jpg)

>陰影出來了~

如果保持`MeshToonMaterial`預設的設定，物體的表面上會以2種顏色來呈現受光的程度。

> 也就是官方文件中提到的`twoTone`，其餘還有可選值

如果我們想要客製化`tone`這個部分，我們會需要傳入自定義的`gradient texture`(漸層紋理)。

所謂的`gradient texture`，其實用意就是告訴`Three.js`到底受光程度的漸變有幾個階層，每個階層又是要用多少的灰度值來呈現。

`gradient texture`不需要太大張，像這邊我使用的`threeTone`版本就只需要3px*1px尺寸即可。

![img](https://i.postimg.cc/cJzvmNSW/16-7.jpg)

這邊我們示範一下把自製的`gradient texture`使用在`MeshToonMaterial`的狀況。

```javascript
const gTexture = tl.load("https://i.imgur.com/BybHhWd.jpg");
const mat = new MeshToonMaterial({
    color: 0xff0000,
    gradientMap: gTexture
  });
```

![img](https://i.postimg.cc/zvhLP2kQ/16-8.jpg)

> 這陰影怎麼又~變的怪怪的啊

在我們把自定義的`gradient texture`作為`gradientMap`傳給`MeshToonMaterial`之後，它卻還是沒有按照我們的想像生成3種顏色，反而是變成了類似`MeshStandardMaterial`一樣有漸層的陰影。

之所以會這樣，原因是因為當使用自訂義`gradientMap`的時候，我們必須要再調整兩個設置(這一點官方文件有提到)。

![img](https://i.postimg.cc/6qZk2C9R/16-9.jpg)


```javascript
...
gTexture.minFilter = NearestFilter; //NearestFilter記得要import
gTexture.magFilter = NearestFilter;
```

![](https://i.postimg.cc/B6bs68Lc/16-10.jpg)

> 有了! 第三層陰影!

> codepen連結: [點我](https://codepen.io/mizok/pen/bGMMYLM)


## 小結

我們終於結束了`Material`章節~ 明天開始終於要來到全新的進度了，還請大家拭目以待!

## 延伸資源

- [藝術家sespider提供的免費MatCaps材質](https://www.deviantart.com/sespider/art/163-FREE-MatCaps-258893793)

- [台部落上面有關MeshNormalMaterial的介紹](https://www.twblogs.net/a/5ef2d470ef31d0621c6707a3)