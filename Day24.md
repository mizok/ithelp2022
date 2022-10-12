# Day24 - 打造質感系3D聊天室 - three.js + socket.io

> 這裡是「Three.js學習日誌」的第24篇，這篇是在講解使用three.js + socket.io打造3D聊天室作品。這系列的文章假設讀者看得懂javascript，並且有Canvas 2D Context的相關知識。

今天我們要來接上昨天的進度!

按照預定今天是要把`實體文本聊天室`的UI、還有`canvas3D聊天室`的**畫面**都作出來~

首先還是先來展示一下今天的進度狀況~

![img](https://i.imgur.com/ZLId0uM.gif)

## 前情提要

![img](https://i.imgur.com/09T2zdg.jpg)

我們在上一回製作了專案的大致外觀，不過`實體文本聊天室`的UI、還有`canvas3D聊天室`的部分都還沒有畫面。

所以我們今天的目標就是要把這兩樣東西做到至少有畫面~


## 1. 首先是實體文本聊天室(的畫面)

我們今天只是要做**畫面**而已，所以這部分基本上只會改動`html`、`scss`。

> scss的部分因為太佔篇幅，而且又不是本系列的重點，所以我們還是在這篇作品結束之後再公開

這部分其實沒什麼太大的障礙，就是單純的**切版**，這邊我們只擷取右側聊天室部分的`html`，避免整篇貼上來太佔篇幅。

> 偷偷宣言一下，筆者小弟我在這部分比較支持老派的`BEM`CSS命名規則，`BEM`讚!

```html
 <div class="wrapper__chat-block chat-block " id="chat-block">
        <button class="chat-block__toggler" id="chat-block-toggler"></button>
        <div class="chat-block__header user">
          <div class="user__avatar">
            <img src="~@img/avatar.png" alt="">
          </div>
          <div class="user__name">
            Mizok
          </div>
        </div>
        <div class="chat-block__body">
          <div class="chat-block__body-inner chat-main">
            <div class="chat-main__chat ">
              <div class="chat-main__bubble">Hello There!</div>
            </div>
            <div class="chat-main__chat chat-main__chat--other">
              <div class="chat-main__bubble">Hello There!</div>
            </div>
            
          </div>
        </div>
        <div class="chat-block__footer">
          <div class="chat-block__input input-block">
            <input type="text" id="txtInput" class="input-block__input"  placeholder="Type something...">
            <button  class="input-block__button"  id="sendTxt">
            </button>
          </div>
          <div class="chat-block__author author">
            <div class="author__former">
              &copy Mizok.H
            </div>
            <div class="author__latter">
              <a href="#" class="author__link">
                <img src="~@img/github.svg" alt="">
              </a>
              <a href="#" class="author__link">
                <img src="~@img/twitter.svg" alt="">
              </a>
            </div>
          </div>
        </div>
      </div>
```

## 2. 接著是canvas3D聊天室(的畫面)

這邊筆者先來講講我自己的規劃~

因為我們的`Cube`(中間的方塊) 一共有**6**個面。

假設忽略掉上下兩面，剩下每一面都放上聊天室的畫面，這樣感覺會非常的**單調**...

所以我的規劃是剩下的**4**個面分別放置不同的機能。

今天我們會先做出其中**兩面**，明天再做剩下的**兩面**。

我們今天要做的**兩面**，也就是:

- 聊天室

- 時鐘

這裡應該會是今天最**困難**的內容。

首先，我們的**方塊上面版**的部分其實是用`three.js`的`CSS3DRenderer`渲染出來的。

所謂的`CSS3DRenderer`其實就是`three.js`透過動態地去更改`HTML`元素的`transform`屬性，讓這個`HTML`元素產生`3D(透視)效果`。

![img](https://i.imgur.com/hajFeEp.jpg)

`CSS3DRenderer`會根據選用的`camera`類型，來決定`transform` `HTML`元素的邏輯。

>例如有沒有做圖像透視運算。

同時因為我們在這個專案上已經有使用了`WebGLRenderer`，並且已經有了一個**Scene**實例，所以為了避免`CSS3DRenderer`也跑去渲染原本的**Scene**，這邊我們的做法會像是這樣:


![img](https://i.imgur.com/FuuQ4u0.jpg)

- 我們會另外新增一個**Scene**(Let's call it **Scene2**)
- 我們會把`WebGLRenderer`用來渲染的`canvas`，還有`CSS3DRenderer`用來`transform`的`HTML`元素，用`CSS` `position:absolute`的方式疊合在一起。

為了達成上面這個架構，我把資料夾的目錄做了一點修改，變成這樣:

![img](https://i.imgur.com/v6SSNcW.png)


另外在這幾隻檔案做了修改:

- `./src/ts/class/base.ts`
- `./src/ts/class/cube.ts`
- `./src/ts/class/renderer.ts`
- `./src/ts/class/playground.ts`

並新增了
- `./src/ts/class/dom-cube.ts`
- `./src/ts/dom/chat.ts`
- `./src/ts/dom/clock.ts`


這邊我們先來講講**修改**的部分，接著再講**新增**的部分

### 2-1 `./src/ts/class/base.ts`

```typescript
export class Base {
    sizer = new Sizer(this.canvas)
    scene = new Scene();
    scene2 = new Scene(); // 加入了Scene2
    ticker = new Ticker();
    camera = new Camera(this);
    renderer = new Renderer(this);
    playground = new Playground(this);
    touched = false;
    touchedReactDelay = 1000;
    resources: {
        [key: string]: any
    }

    // 這邊新增傳入了兩個HTMLElement，分別是domCanvas和domBundle，之後會提到
    constructor(public canvas: HTMLCanvasElement, public domCanvas: HTMLElement, public domBundle: HTMLElement) {
        this.initResizeMechanic();
        this.initTickMechanic();
        this.initTouchMechanic();
    }

    //...
    //...
    //... 中間部分因為沒有變更，跟之前一樣，所以省略

     initTickMechanic() {
        this.ticker.on('tick', (clock: Clock) => {
            //這邊我原本是把clock整個直接傳給playground作為參數
            //現在改成只傳送幀間時差
            const delta = clock.getDelta();
            this.renderer.update();
            this.camera.update();
            this.playground.update(delta);
        })
    }

    
    // 這邊我順手實作了當滑鼠點擊的時候暫時停止方塊旋轉的邏輯，
    // 不然使用者主動旋轉方塊時還讓方塊一直自旋的話UX體驗會很糟糕XD
    initTouchMechanic() {
        let startLocation = new Vector2();
        let endLocation = new Vector2();
        let timeout: any;
        const cbStart = (e: MouseEvent) => {
            startLocation.x = e.clientX;
            startLocation.y = e.clientY;
            this.touched = true;
        }
        const cbEnd = (e: MouseEvent) => {
            endLocation.x = e.clientX;
            endLocation.y = e.clientY;
            //如果按下跟提起的座標相距不遠，那就不暫停旋轉，反之則暫停一秒
            const delay = startLocation.distanceTo(endLocation) > 10 ? this.touchedReactDelay : 0;
            clearTimeout(timeout)
            timeout = setTimeout(() => {
                this.touched = false;
            }, delay)
        }
        this.domCanvas.addEventListener('mousedown', cbStart)
        this.domCanvas.addEventListener('touchstart', cbStart)
        this.domCanvas.addEventListener('mouseup', cbEnd)
        this.domCanvas.addEventListener('touchend', cbEnd)
        this.domCanvas.addEventListener('mouseleave', cbEnd)
    }

    //...
}
```

在`./src/ts/class/base.ts`中，主要調整的地方大概如下:

#### 新增傳入domCanvas和domBundle這兩個Html元素

`domCanvas`其實是一個`DIV`，它是後面我們用來傳給`CSS3DRenderer`作為參數用的，而`domBundle`也是一個`DIV`，它裡面放置了四個方塊面板的`Html`元素，我們之後會再講到怎麼使用它。

#### 實作「當滑鼠點擊的時候暫時停止方塊旋轉」的邏輯

除了新增了剛剛提到的**Scene2**之外，還補上了一段「當滑鼠點擊的時候暫時停止方塊旋轉」的邏輯，主要是因為我想到使用者主動旋轉方塊時還讓方塊一直自旋的話，可能體感不是很好XD

#### 調整了`initTickMechanic`的傳參
這邊原本我們是把`clock`整個直接傳給`playground`作為`update`方法的參數，而現在我改成只傳送**幀間時差**(delta)。

原因主要是因為:

現在我們有「當滑鼠點擊的時候暫時停止方塊旋轉」的邏輯，而如果這邊我們保持用原本的`getElapsedTime`去計算旋轉方塊的角度，當方塊旋轉被暫停的時候，`getElapsedTime`回傳的值還是會繼續增加，導致方塊在結束暫停的時候會直接瞬間跳轉到奇怪的角度。

所以這個地方我改成傳送**幀間時差**(delta)，讓方塊旋轉的邏輯變成是每一帧去增加角度(+=delta/5)，這樣就不會出現角度跳轉的問題。


### 2-2 `./src/ts/class/cube.ts`

```typescript
export class Cube implements MeshType {
    mesh: Mesh;
    group: Group;
    ready = false;
    constructor(private base: Base) {
        this.setModel();
    }

     //...
    //...
    //... 中間部分因為沒有變更，跟之前一樣，所以省略

    doAnimation() {
        gsap.to(this.mesh.rotation, {
            x: 0,
            y: -Math.PI / 2, 
            //這邊我稍微調整了一下旋轉的角度，
            //主要是希望可以開場看到聊天室畫面
            z: 0,
            duration: 1, 
            paused: true
        }).play()
        gsap.to(this.mesh.scale, {
            x: 1,
            y: 1,
            z: 1,
            duration: 2, 
            paused: true,
            onComplete: () => {
                this.ready = true;
            }
        }).play()
    }

    update(delta: number) {
        //這邊就是我們剛剛提到的改成以delta來計算旋轉
        if (!this.base.touched) {
            this.group.rotation.y += delta / 5;
        }
    }
}
```

基本上`./src/ts/class/cube.ts`的部分沒什麼特別的，簡單來說就是我們剛剛提到過的把旋轉的計算方式改成用「+=delta/5」的方式來實作。


### 2-3 `./src/ts/class/renderer.ts`

```typescript
export class Renderer {
    instance: WebGLRenderer;
    instance2: CSS3DRenderer;//加入了CSS3DRenderer作為第二實例
    private sizer = this.base.sizer;
    private canvas = this.base.canvas;
    private domCanvas = this.base.domCanvas;
    private scene = this.base.scene;
    private scene2 = this.base.scene2;//加入了Scene2
    private camera = this.base.camera;
    constructor(
        private base: Base
    ) {
        this.setInstances()
    }

    setInstances() {
        //instance
        this.instance = new WebGLRenderer({
            canvas: this.canvas,
            antialias: true
        })
        this.instance.physicallyCorrectLights = true
        this.instance.toneMappingExposure = 1.75
        this.instance.shadowMap.enabled = true
        this.instance.shadowMap.type = PCFSoftShadowMap
        this.instance.setClearColor(0xffffff)
        //instance2
        this.instance2 = new CSS3DRenderer({
            //我們這邊把剛剛提到的domCanvas傳進去
            //這樣等下我們add進來的物件都會被放置在這個domCanvas底下
            element: this.domCanvas  
        });
        this.sizing();
    }

    resize() {
        this.sizing();
    }

    sizing() {
        //instance
        this.instance.setSize(this.sizer.width, this.sizer.height);
        this.instance.setPixelRatio(this.sizer.pixelRatio);
        //instance2
        // 發動CSS3DRenderer的setSize方法
        this.instance2.setSize(this.sizer.width, this.sizer.height);
    }

    update() {
        this.instance.render(this.scene, this.camera.instance);
         // 發動CSS3DRenderer的render方法
        this.instance2.render(this.scene2, this.camera.instance);
    }
}
```

在`./src/ts/class/renderer.ts`中我修改的部分就是生成`CSS3DRenderer`的實例，並且統一發動`CSS3DRenderer`和`WebGLRenderer`的`render`/`setSize`方法。


### 2-4 `./src/ts/class/playground.ts`

```typescript
export class Playground {
    env: Env;
    cube: Cube;
    domCube: DomCube;
    ready = false;
    constructor(private base: Base) {
        this.init();
    }
    init() {
        this.base.getResources().then(() => {
            this.env = new Env(this.base);
            this.cube = new Cube(this.base);
            this.domCube = new DomCube(this.base);
            this.ready = true;
        })
    }

    update(delta: number) {

        if (this.ready) {
            this.env.update(delta);
            this.cube.update(delta);
            this.domCube.update(delta);
        }

    }
}
```
`./src/ts/class/playground.ts`其實也沒啥特別的，就是把`DomCube`的實例加進去，並且同步發動`env`/`cube`/`domCube`的`update`方法。


### 2-5 `./src/ts/class/dom-cube.ts`

```typescript
import { Group } from "three";
import { Base } from "./base";
import { Chat } from '../dom';
import gsap from "gsap";
import { Clock } from "../dom/clock";

export class DomCube {
    chat: Chat;
    clock: Clock;
    groupOuter = new Group();
    groupInner = new Group();
    ready = false;
    constructor(private base: Base) {
        this.init();
    }

    init() {
        this.chat = new Chat(this.base);
        this.clock = new Clock(this.base);
        this.groupInner.scale.set(0, 0, 0);
        this.groupInner.rotation.set(Math.PI / 3, Math.PI / 3, Math.PI / 3);
        this.groupInner.add(this.chat.object); //加入chat面板
        this.groupInner.add(this.clock.object); //加入clock面板
        this.groupOuter.add(this.groupInner); 
        //利用兩層group來模仿Cube的旋轉動畫
        this.base.scene2.add(this.groupOuter);
        this.doAnimation();
    }

    doAnimation() {
        gsap.to(this.groupInner.rotation, {
            x: 0,
            y: -Math.PI / 2, //這邊參數都跟cube 一樣
            z: 0,
            duration: 1, 
            paused: true
        }).play()
        gsap.to(this.groupInner.scale, {
            x: 1,
            y: 1,
            z: 1,
            duration: 2, 
            paused: true,
            onComplete: () => {
                this.ready = true;
            }
        }).play()
    }

    update(delta: number) {
        if (!this.base.touched) {
            this.groupOuter.rotation.y += delta / 5;
        }
        this.chat.update();
        this.clock.update();
    }
}
```

這邊`DomCube`的概念有點像是用`Group`假造一個虛擬的cube，它也同樣有**外層旋轉**和**內層旋轉**的作動方式，讓`DomCube`看起來好像是黏在`Cube`上面一起旋轉。

但實際上`DomCube`是`HTML`元素，而`Cube`則是`Canvas`。

![img](https://i.imgur.com/Eyn4fjl.png)

> 老實說這邊應該可以改成讓domCube extends Cube，畢竟方法有重複的，不過我後來想想決定還是先等後續優化階段再說吧 :P

### 2-6 `./src/ts/dom/chat.ts`

```typescript
import { Object3D, Vector3 } from "three";
import { CSS3DObject } from "three/examples/jsm/renderers/CSS3DRenderer";
import { Base } from "../class/base";

export class Chat {
    object: Object3D
    element: HTMLElement
    private offset = 1.7; // offset 其實就是cube的邊長/2
    constructor(private base: Base) {
        this.setElement();
    }
    setElement() {
        this.element = this.base.domBundle.querySelector('#chat-main');
        this.object = new CSS3DObject(this.element);
        this.object.position.set(0, 0, this.offset);
        this.object.scale.set(1 / 160, 1 / 160, 1); 
        //160的縮放比例是憑感覺抓的
    }
    // 這邊這個算法其實蠻重要的，主要是為了要在面板轉到背面的時候隱藏它
    update() {
        const bias = -this.offset / 10;
        //取得面板的朝向
        const objectToward = this.object.getWorldDirection(new Vector3(0, 0, 0)).normalize();
        //取得攝影機的朝向
        const cameraToward = this.base.camera.instance.getWorldDirection(new Vector3(0, 0, 0)).normalize();
        //取得上述兩個朝向的點積
        const dp = objectToward.dot(cameraToward);
        //如果點積值大於方塊邊長/2 那就隱藏，若否，那就顯示
        if (dp > bias) {
            this.element.style.opacity = '0';
        }
        else {
            this.element.style.opacity = '1';
        }
    }
}
```

在目前這個階段，`./src/ts/dom/chat.ts` 和等下要介紹的`./src/ts/dom/clock.ts`其實很像。

這邊的邏輯其實就是把`HTML`元素，利用`CSS3DObject`這個類，包裝成`CSS3DObject`物件，這樣我們就可以加進去位於`dom-cube`的`group`裡面，讓所有的面板一起被渲染。

在`./src/ts/dom/chat.ts`還有一個重要的段落就是`update`方法，這邊我還實作了「當面板轉到背面的時候將該元素透明度訂為0」。

![img](https://i.imgur.com/zPhQfnj.gif)
> 這邊如果沒有偵測背面並隱藏的話就會有這種問題

「當面板轉到背面的時候將該元素透明度訂為0」主要是利用內積去確認當前**面**和**攝影機**的正交狀況。


### 2-7 `./src/ts/dom/clock.ts`

```typescript
import { Object3D, Vector3 } from "three";
import { CSS3DObject } from "three/examples/jsm/renderers/CSS3DRenderer";
import { Base } from "../class/base";

export class Clock {
    object: Object3D
    element: HTMLElement
    private offset = 1.7;
    constructor(private base: Base) {
        this.setElement();
    }
    setElement() {
        this.element = this.base.domBundle.querySelector('#clock');
        this.object = new CSS3DObject(this.element);
        this.object.position.set(-this.offset, 0, 0);
        this.object.rotation.y = - Math.PI / 2;
        this.object.scale.set(1 / 160, 1 / 160, 1 / 160);
    }

    update() {
        const bias = - this.offset / 10;
        const objectToward = this.object.getWorldDirection(new Vector3(0, 0, 0));
        const cameraToward = this.base.camera.instance.getWorldDirection(new Vector3(0, 0, 0));
        const dp = objectToward.dot(cameraToward);
        if (dp > bias) {
            this.element.style.opacity = '0';
        }
        else {
            this.element.style.opacity = '1';
        }
    }
}
```

`./src/ts/dom/clock.ts`在現階段其實就跟`./src/ts/dom/chat.ts`差不多，所以我們就不特別介紹~


## 小結

今天實作完了大部分的外觀(還有一小部分沒處理)，明天我們除了會處理剩下的外觀，還會開始著手處理`Socket.io`連線的部分，希望各位可以繼續追蹤~

