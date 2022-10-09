# Day22 - 使用Webpack 5打造Three.js的Boilerplate(三)

> 這裡是「Three.js學習日誌」的第22篇，這篇的內容是要講解如何將筆者開發的Webpack模板改造成一個three.js的Boilerplate。這系列的文章假設讀者看得懂javascript，並且有Canvas 2D Context的相關知識。

今天我們要來將上一回提到的「webpack-template」改造成一個可以重複使用的`three.js boilerplate`，在開始之前我們會先來講講為什麼要製作**Boilerplate**。

## 4. 什麼是Boilerplate?為什麼要有Boilerplate?

**Boilerplate**這個詞的中文翻譯是"**樣板**"。

我們之所以要為大型專案製作**Boilerplate**，原因有:

- 減少流水帳一般的**Coding Style**，把部分的邏輯抽出並且實現橫向管理

- 統一各大專案的資料夾結構，減少歧異性

- 使用高階語言來優化程式碼開發過程

- 避免每次專案都要重新寫一次初始化的環節

也許有人有注意到，雖然我們前面每次在寫**Code**的時候，筆者小弟我都是盡量的把關鍵程式碼抽出來做介紹，但事實上如果我們把一篇文章中所有的程式碼片段都集中起來放進一個`Function`裡面去執行，整篇程式碼其實會變得相當長。

程式碼太長通常會導致幾個問題:

- 開發者需要花時間去捲動卷軸，或是使用文章內搜尋功能，才能找到特定的區塊。
- 廢棄不用的程式碼容易堆積在不常捲動到的死角，久而久之造成垃圾越來越多，寫作也一起被拖慢。

所以說適時的把程式碼分割/分類其實是很重要的一環。在這個部分`ES6`提供了我們很方便的`import `、`export`，讓我們可以很輕鬆地整理程式碼，同時搭配`typescript`做開發，我們還可以在IDE裡面看到每個`Function`、`Variable`的型別。

我們今天主要的目標就是要來使用「webpack-template」打造一個`three.js`的`Boilerplate`，接著就讓我們開始吧~

## 5. 打造一個`three.js`的`Boilerplate`

### 5-1 安裝NPM包

> 我們上一回有提到「webpack-template」的安裝方法，如果還沒Clone，可以移步到[上一回](https://ithelp.ithome.com.tw/articles/10304920)

這邊我們除了「webpack-template」本身的依賴以外，當然還得安裝`three.js`的`npm package`.

```
npm i three;
```

安裝完畢之後，可以直接開啟「webpack-template」內建的`dev-server`，這樣就可以在瀏覽器上面即時看到開發的狀況。

```
npm run dev;
```

> 當然初始會是一片空白啦~

「webpack-template」中，其實預設有一些基本的`reset.css`(重置瀏覽器樣式的樣式表)，所以您不太需要擔心需要去補上很多的CSS。


### 5-1 補上一些初始的檔案內容

首先我們在`./src/pages/index.main.ejs`這個檔案中，加入`canvas`元素。

```html
<body>
  <%- include('../template/header.ejs') %>
  <canvas></canvas>
  <%- include('../template/footer.ejs') %>
</body>
```

接著在`./src/scss/main`這個檔案中，補上下面這段樣式

```scss
html,
body {
  height: 100%;
  >canvas{
    width: 100% !important;
    height: 100% !important;
  }
}
```

接著就是重頭戲了，我們開始來編寫`./src/ts `裡面的內容~

### 5-2 回想一下通常一個Three.js專案，`JS`的部分基本會需要什麼內容?

一個基本的`three`專案通常需要具備的最基本內容，大概如下~

- `renderer`
- `camera`
- `tick loop`
- `mesh`
- **resize機制**
- **載入的資源(紋理/模型/音效,...etc.)**

這些東西我們在到目前為止的範例，都是直接以流水帳的形式寫在`Codepen`裡面，但是從這次開始我們就是要把這些邏輯分散到不同的檔案中作為`modules`。


我自己對於上述內容的架構拆分，主要是參考**Bruno Simons**這位開發者的[開源專案](https://github.com/brunosimon/threejs-template-complex/tree/master/sources)。

> **Bruno**是一位著名的法國three.js開發者，有興趣可以自己Google看看，他超有名的~

接著讓我們一步一步講解上述的內容我是怎麼做拆分~

### 整體的架構大概長這樣

![img](https://i.imgur.com/llNA2H6.jpg)

> 一共四個資料夾 + 一隻`main.ts`

這邊`main.ts`會採用「webpack-template」的機制直接轉變成`Entry Chunk`的一環。所以我們不需要再去把這隻檔案寫到`./src/pages/index.main.ejs`的`script`標籤上面。

> 這部分相關訊息可以看[上一回](https://ithelp.ithome.com.tw/articles/10304920):【4-2-c 重點機制:「由檔案名生成Entry Chunks/HtmlWebpackPlugin Instance」】

接著我們會One by One的介紹每隻檔案在做些甚麼~

### 5-3 首先當然是入口的`main.ts`

`main.ts`

```typescript
import { Clock } from 'three';
import { Base } from './class/base';

class Main extends Base {

    constructor(canvas: HTMLCanvasElement) {
        super(canvas);
    }
}

(() => {
    const cvs = document.querySelector('canvas');
    const instance = new Main(cvs);
})()
```

大家應該可以大概看懂這個就是一個**Init Function**的入口，`Main`這個`class`上面`extends`了一個叫做`Base`的`class`，這個`Base`的用途就是用來標記`Main`，讓它成為一個操作口的`class`。

> 我們接著看看Base裡面有些什麼玩意。


### 5-4 `./class/base.ts`

```typescript
import { Env } from './env';
import { Renderer } from './renderer';
import { Camera } from './camera';
import { Sizer } from './sizer'
import { Ticker } from '../util';
import { getResources } from '../resource'
import { Scene, Clock } from 'three';
import { Playground } from './playground';

export class Base {
    sizer = new Sizer(this.canvas)
    scene = new Scene();
    ticker = new Ticker();
    playground = new Playground(this)
    camera = new Camera(this);
    renderer = new Renderer(this);
    resources: {
        [key: string]: any
    }

    constructor(public canvas: HTMLCanvasElement) {
        this.initResizeMechanic();
        this.initTickMechanic();
    }
    // 當螢幕resize的時候，會導致sizer這個物件觸發resize事件，
    //並連帶發動renderer 和camera各自的resize方法
    initResizeMechanic() {
        this.sizer.on('resize', () => {
            this.renderer.resize();
            this.camera.resize();
        })
    }
    // 當ticker這個物件每循環一次tick loop，就會觸發tick事件
    // 並連帶發動renderer 和camera各自的update方法
    // 還有frameListener這個會在main.ts被override掉的method
    initTickMechanic() {
        this.ticker.on('tick', (clock: Clock) => {
            this.renderer.update();
            this.camera.update();
            this.playground.update();
        })
    }
   
    // 非同步取得所有專案外連資源的方法
    async getResources() {
        this.resources = await getResources()
    }
}
```

> `base.ts`看起來就多了不少東西，我們一個一個來做介紹。

#### 5-4-a sizer

是一個由`sizer`這個`class`生成的實例，它會在畫面`onload`的時候初始校正一次畫布(canvas)的大小，然後在每次視窗(**window**)發生`resize`事件的時候則會觸發`resize`事件。

我們可以透過`.on`這個方法來把想要在`sizer`觸發`resize`時發動的動作繫結在一起。

#### 5-4-b ticker

其實就是我們之前寫的`tick loop`，`ticker`會在每次循環`tick loop`的時候觸發`tick`事件。

我們同樣也可以透過`.on`來把想要在`ticker`觸發`tick`時發動的動作繫結在一起。

#### 5-4-c getResources

簡單來說就是把所有需要載入的資源集中到一個地方做管理，然後等到全部都載入完畢之後再把內容送到`Base`這邊。

#### 5-4-d  scene

就是`three.js`的`scene`。


#### 5-4-e  camera
用來初始化`camera`還有`orbitControl`，來自於`camera.ts`內部的`class`。


#### 5-4-f renderer

用來初始化`renderer`，來自於`renderer.ts`內部的`class`。

#### 5-4-g playground

用來統合所有可以操作的物體，包括`env`和其他`mesh`

---

### 5-5 `./util/sizer.ts`

```typescript
import { EventEmitter } from './event-emitter'

export class Sizer extends EventEmitter {
    width: number;
    height: number;
    pixelRatio = Math.min(window.devicePixelRatio, 2);

    constructor(public canvas: HTMLCanvasElement) {
        super()
        this.initSizingMechanic();
    }
    // 綁定window resize事件
    initSizingMechanic() {
        this.sizing();
        window.addEventListener('resize', this.sizing.bind(this))
    }
    // 更新width/height 屬性，並主動觸發sizer自己的resize事件，同時再帶入事件參數
    sizing() {
        const rect = this.canvas.getBoundingClientRect();
        this.width = rect.width;
        this.height = rect.height
        this.trigger('resize', [this.width, this.height])
    }
}

```

我們在`sizer.ts`這隻檔案裡面可以看到這隻檔案其實就是在綁定**Window Resize**事件，並且觸發`sizer`自己的**Resize**事件(同時還會附帶傳遞事件參數)

### 5-6 `./util/ticker.ts`

```typescript
import { EventEmitter } from './event-emitter'
import { Clock } from 'three';

export class Ticker extends EventEmitter {
    private clock: Clock = new Clock();
    constructor() {
        super()
        window.requestAnimationFrame(() => {
            this.tick()
        })
    }
    tick() {
        this.trigger('tick', [this.clock])
        window.requestAnimationFrame(() => {
            this.tick()
        })
    }
}
```

`ticker.ts`就是我們之前寫的`tick loop`，每次`tick loop`的循環都會觸發`ticker`的**tick**事件(同時還會傳遞`Clock`的實例作為事件參數)

### 5-7 `./util/event-emiter.ts`


這隻檔案比較特別。

首先，它不是我寫的XD，而是我從**Bruno Simons**的[**Gist**](https://gist.github.com/brunosimon/120acda915e6629e3a4d497935b16bdf)裡面抄過來的。

因為它的篇幅很長，所以我沒有打算貼在這邊。

> 有寫過Angular專案開發的人應該很熟悉`event-emiter`這個名詞

所謂的`event-emiter`其實就是一種`Callback`的**註冊**與**觸發**服務的統稱。

![img](https://i.imgur.com/ZMVO401.png)

一般的`event-emiter`類別，裡面通常都會提供**註冊(綁定)**和**觸發**的方法，在**Bruno Simons**寫的這支`event-emiter`，它們分別叫做`.on`跟`.trigger`。

> 在angular中則是叫做`output`和`emit`

講到`.on`跟`.trigger`，大概就會有人聯想到`jquery`的`Jquery.on`跟`Jquery.trigger`。

而確實這邊的`event-emiter`和`jquery`的`Jquery.on`跟`Jquery.trigger`是差不多的東西。

在這邊如果想要使用`event-emiter`這個類，就只要把它`extends`到目標的**類**上面(就像前面的`sizer`和`ticker`)。然後在該類別中決定要在什麼條件**觸發**(trigger)事件，接著在有生成該類別實例的地方用`.on`註冊`Callback`即可。

### 5-8 `./resource/index.ts`

```typescript
import { textureSources } from './textures';
import { TextureLoader, CubeTextureLoader } from 'three';

interface SourceObj {
	name: string,
	content: any
}

const textureLoader = new TextureLoader();
const cubeTextureLoader = new CubeTextureLoader();

// 取得texture的load promise
const getTexture = (source: any) => {
	const prm: Promise<SourceObj> = new Promise((res, rej) => {
		textureLoader.load(
			source.path,
			(texture) => {
				res({
					name: source.name,
					content: texture
				});
			},
			null,
			rej
		);
	});
	return prm;
};

// 取得cubeTexture的load promise

const getCubeTexture = (source: any) => {
	const prm: Promise<SourceObj> = new Promise((res, rej) => {
		cubeTextureLoader.load(
			source.paths,
			(texture) => {
				res({
					name: source.name,
					content: texture
				});
			},
			null,
			rej
		);
	});
	return prm;
}


export const getResources = () => {

	const promiseArr: Promise<SourceObj>[] = [];

	for (let textureSource of textureSources) {
		switch (textureSource.type) {
			case 'cubeTexture': promiseArr.push(getCubeTexture(textureSource))
			case 'texture': promiseArr.push(getTexture(textureSource))
		}
	}

	return Promise.all(promiseArr).then((values) => {
		const result: {
			[key: string]: any
		} = {};
		values.forEach((sourceObj) => {
			result[sourceObj.name] = sourceObj.content;
		})
		return result;
	})
}

```

如果你在前面有看過我們介紹要怎麼把`textureLoader`的`.load`方法包裝成`Promise`，這一段應該就沒甚麼特別的。

> 如果忘記是哪邊有提到這段，可以[這邊](https://ithelp.ithome.com.tw/articles/10299778)請

這邊其實就是單純的去遍歷在`./resource/textures`裡面的陣列資料，然後依據這些資料去GET對應的資源，接著再利用`Promise.all`把每一個`Load Promise`合併在一起，再`return`出去。


### 5-9 `./class/camera.ts`與`./class/renderer.ts`

`./class/camera.ts`

``` javascript
import { PerspectiveCamera } from 'three';
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js';
import { Base } from './base';

export class Camera {
    instance: PerspectiveCamera;
    controls: OrbitControls;
    private sizer = this.base.sizer;
    private canvas = this.base.canvas;
    private scene = this.base.scene;

    constructor(
        private base: Base
    ) {
        this.setInstance()
        this.setControls()
    }

    //建立相機實例
    setInstance() {
        const camera = new PerspectiveCamera(35, this.sizer.width / this.sizer.height, 0.1, 100);
        camera.position.set(0, 0, 5);
        this.instance = camera;
        this.scene.add(this.instance)
    }

    //建立Orbit Control實例
    setControls() {
        this.controls = new OrbitControls(this.instance, this.canvas)
        this.controls.enableDamping = true
    }

    //Camera自己的Resize方法
    resize() {
        this.instance.aspect = this.sizer.width / this.sizer.height
        this.instance.updateProjectionMatrix()
    }

    //Camera自己的update方法
    update() {
        this.controls.update()
    }
}
```

`./class/renderer.ts`

``` javascript
import { WebGLRenderer, PCFSoftShadowMap } from 'three';
import { Base } from './base';

export class Renderer {
    instance: WebGLRenderer;
    private sizer = this.base.sizer;
    private canvas = this.base.canvas;
    private scene = this.base.scene;
    private camera = this.base.camera;
    constructor(
        private base: Base
    ) {
        this.setInstance()
    }
    // 建立`renderer`實例
    setInstance() {
        this.instance = new WebGLRenderer({
            canvas: this.canvas,
            antialias: true
        })
        this.instance.toneMappingExposure = 1.75
        this.instance.shadowMap.enabled = true
        this.instance.shadowMap.type = PCFSoftShadowMap
        this.instance.setClearColor(0xffffff)
        this.resize();
    }
     // renderer自己的resize方法
    resize() {
        this.instance.setSize(this.sizer.width, this.sizer.height);
        this.instance.setPixelRatio(this.sizer.pixelRatio);
    }
    // renderer自己的update方法
    update() {
        this.instance.render(this.scene, this.camera.instance)
    }
}
```

`./class/camera.ts`和`./class/renderer.ts`其實也沒甚麼特別的。它們分別就是用來初始化`camera`和`renderer`實例的分層。

比較值得一提的是我在`camera`和`renderer`都有把`base`傳進去，這樣就可以透過`base`拿到`base`底下的`sizer`或是`ticker`之類的東西。

除此之外`camera`和`renderer`都有自己的`resize`和`update`方法，以便可以在`base`裡面，從`event-emitter`的註冊`Callback`部分呼叫它們。

### 5-10 `./class/env.ts`

```typescript
import { Base } from './base';
import { AmbientLight, Clock, DirectionalLight } from 'three';

export class Env {
    ambientLight: AmbientLight;
    directionalLight: DirectionalLight;
    constructor(private base: Base) {
        this.setLights();
    }
    
    setLights() {
        this.setAmbientLight();
        this.setDirectionalLight();
    }
    // 設置方向光
    setDirectionalLight() {
        this.directionalLight = new DirectionalLight(0xffffff, 1);
        this.directionalLight.castShadow = true
        this.directionalLight.shadow.mapSize.set(2048, 2048)
        this.directionalLight.shadow.normalBias = 0.05
        this.directionalLight.position.set(3.5, 2, - 1.25)
        this.base.scene.add(this.directionalLight)
    }
    //設置環境光
    setAmbientLight() {
        this.ambientLight = new AmbientLight(0xffffff, 0.3);
        this.base.scene.add(this.ambientLight)
    }
    //env 自己的update方法
    update(clock: Clock) {

    }

}
```

`./class/env.ts`說穿了其實就是用來放置燈光和環境貼圖的`class`，它最終會在`Playground`這個類裡面被建立起來。

### 5-11 `./class/playground.ts`

```typescript
import { Env } from "./env";
import { Base } from "./base";
import { Box } from "../mesh";
import { Clock } from "three";

export class Playground {
    env: Env;
    box: Box;
    constructor(private base: Base) {
        this.init();
    }
    init() {
        this.base.getResources().then(() => {
            this.env = new Env(this.base);
        })
    }
    // playground自己的update方法
    update(clock: Clock) {
        this.env.update(clock);
    }
}
```


最後是`./class/playground.ts`的介紹，在這邊我們可以看到`./class/playground.ts`的`init`方法會強制在資源載入之後才作發動，這樣確保了`Env`裡面的環境貼圖/其他`mesh`的紋理不至於會產生非同步問題。


## 6. 在這個模板裡面放進去一個會隨時間旋轉的方塊看看~


雖然上述的Boilerplate看上去架構好像有點複雜，但是如果把觀念釐清之後其實不難。

這邊我們來嘗試在這個模板裡面放進去一個會隨時間旋轉的**方塊**看看~


### 6-1 建立方塊的`class`和`ts`文件

首先我們在`./ts`底下建立一個叫做`mesh`的資料夾，並且在`mesh`底下建立`box.ts`和`index.ts`。

`box.ts`
```typescript
import { map } from 'lodash';
import { BoxGeometry, Clock, Mesh, MeshStandardMaterial } from 'three';
import { Base } from '../class/base';

export class Box {
    mesh: Mesh;
    constructor(private base: Base) {
        this.setModel();
    }


    setModel() {
        const geo = new BoxGeometry(1, 1, 1);
        const mat = new MeshStandardMaterial({
            color: 0xff0000,
            map: this.base.resources.someTexture
        })
        this.mesh = new Mesh(geo, mat);
        this.base.scene.add(this.mesh);

    }
    update(clock: Clock) {
        this.mesh.rotation.x = Math.sin(clock.getElapsedTime())
    }
}
```

`index.ts`
```typescript
export * from './box' 
```

之所以要這樣BY MESH去建立`.ts`文件，是因為在專案後期，物件都會變得越來越多，像這樣去把每個`Mesh`分隔到獨立的文件，在未來才能方便查找。

而在這邊我們可以看到，`box.ts`其實基本上就是可以讓我們在`new`完它的時候，就自動建立到`Scene`裡面。

所以這邊我們在`playground.ts`裡面把`Box`給建立出來，同時再補上`Box`自身的`update`方法，讓它可以跟著`Playground`的`update`方法一起更新狀態。

`Playground.ts`

```typescript
import { Env } from "./env";
import { Base } from "./base";
import { Box } from "../mesh";
import { Clock } from "three";

export class Playground {
    env: Env;
    box: Box;
    constructor(private base: Base) {
        this.init();
    }
    init() {
        this.base.getResources().then(() => {
            this.env = new Env(this.base);
            this.box = new Box(this.base);
        })
    }

    update(clock: Clock) {
        this.env.update(clock);
        this.box.update(clock);
    }
}
```

> 搭拉~

![img](https://i.imgur.com/bQ7wmv9.gif)


## 7. 用流程圖來分析一下整體模板的運作

![img](https://i.imgur.com/njj8lGZ.jpg)

這邊我用一張簡單的流程圖講述了模板整體的運作方式。
我認為其中有幾點是比較重要的:

- `Playground`的存在，統整`env`和`mesh`，並且統一的去更新子項目的狀態。
- 使用`Singleton`的方式來實作，並且分層管理，大幅減少了流水帳的問題。


## Github Repository 地址

關於`three.js boilerplate`製作的部分差不多就到這邊先告一段落，接著我們會使用這套模板來進行專案的製作。

這邊是我們這次製作的`three.js boilerplate`的`Github Repository`地址
> https://github.com/mizok/three-ts-template

---

## 小結
今天我們終於結束了`three.js boilerplate`模板的實作，敬請各位期待接下來的作品創作!


>備註: 本文在2022.10.9有作過一部分優化調整，因為考慮到原本的模板設計有些瑕疵，修改之後增加`Playground`這個類，並且優化了一些文字描述，如對評審產生不便，敬請見諒。