# Day2 - 從webGL的基礎開始?(二)

> 這裡是「Three.js學習日誌」的第2篇，本篇的主旨是藉由描述一些簡單的webGL基礎，來做為引導three.js學習的鋪墊


## 來寫一個webGL的hello world吧!

這邊我們要藉由用`webGL`畫一個三角形，來完成「寫一個webGL的hello world」這項任務~


### 1. 取得渲染環境

首先當然是要先建立canvas元素並取得`webGL`的渲染環境

`index.html`
```html
<!-- 這邊給的這些 height:100%;width:100% 目的是要讓canvas跟視窗永遠等高等寬-->
<html style="height:100%">
    <body style="height:100%">
        <canvas style="height:100%;width:100%"></canvas>
    </body>
</html>
```

`index.js`
```javascript
    const cvs = document.querySelector('canvas');
    const gl =  cvs.getContext('webgl');
    ...
```

### 2. 設定canvas的長寬與正確的螢幕解析度

接著我們要設定canvas的長寬，讓他可以以正確的螢幕解析度來顯示圖像。

`index.js`
```javascript
...
//由於我們在html已經設定讓canvas跟視窗永遠等高等寬了，
//所以這邊就算強制的讓width和height以螢幕像素比的比率倍增，也不會導致canvas超出螢幕畫面
//而是形成一種把像素強制壓縮的效果，可以藉由這樣來生成正常的螢幕解析度
gl.canvas.width = window.devicePixelRatio * gl.canvas.clientWidth;
gl.canvas.height = window.devicePixelRatio * gl.canvas.clientHeight;
...
```

### 3. 設定webgl的座標映射

`gl.viewport`是一個webgl的特有方法，在`webgl`中，座標的分布比較特別，相較於我們常見的`x軸``y軸`都有正無限大到負無限大，webgl則是用+1~-1來表示，而`gl.viewport`的用意則是把+1~-1映射到視窗上指定的範圍。

```javascript
...
// 這邊我們把整個視窗大小設定為映射範圍
// gl.viewport前兩個參數是映射範圍的x,y座標，後兩個則是映射範圍的長寬
gl.viewport(0, 0, gl.canvas.width, gl.canvas.height);
...
```

### 4. 建立空白Shader腳本

雖然大多數的坊間翻譯都是把 `Shader` 翻譯為 「著色器」，但我其實一直覺得這個*譯名翻得很不到位*。<br><br> `Shader`主要指的是**定義圖形渲染流水線規則的算法腳本** 。<br><br>(就是我們上一篇提到過的「**定義**構成了圖像的頂點座標，接著**賦予**這些頂點指定的顏色」)<br><br>一般會分成`Vertex Shader`(頂點著色器) 和 `Fragment Shader`(片段著色器)，這兩者分別的職責就是「**定義構成圖像的頂點其座標**」& 「**賦予這些頂點顏色**」。<br><br>而這裡的`gl.createShader` 的用意則是要建立空白的`shader`腳本，並且可以傳入`gl.VERTEX_SHADER` 或 `gl.FRAGMENT_SHADER`來決定到底是要產生哪一種空白腳本。

```javascript
...
const fragmentShader = gl.createShader(gl.FRAGMENT_SHADER);
const vertexShader = gl.createShader(gl.VERTEX_SHADER);
...

```

> 延伸閱讀: [著色器介紹 - 逍遙文工作室](https://cg2010studio.com/2011/06/29/shader/)

### 5. 定義Shader腳本內容來源

產生完空白腳本之後則是要定義腳本內容所指向的來源，也就是腳本內容要放些什麼。<br><br>順帶一提，這邊`shaderSource`的第二個參數是要以字串型別傳入整個`Shader`的腳本內容。<br><br>一般狀況下，如果`Shader`的內容不複雜，我們可以用`ES6`的`Template literals` (就是反引號字串) <br>來撰寫腳本內容，但如果`Shader`的內容偏長，也可以額外寫成一份`html`格式的文件，然後用`ajax`的方式引入。<br><br>或是如果有用`webpack`，也可以搭配[shader-loader](https://www.npmjs.com/package/shader-loader) 把腳本包裝成`module`。<br>(這種方法甚至還可以吃的到`VSCode`的語言highlight) <br><br> 這邊我們先用最簡單的`Template literals`來導入`Shader`腳本內容。

>延伸閱讀:[使用webpack導入shader的實際範例](https://github.com/mizok/generative-art-playground/blob/master/src/examples/dream/ts/index.ts)

```javascript
...
const vertexShaderScript = `
//這個是shader宣告變數的方式，意思近似聲明有一個attribute類型的變數存在，並且他是一組四元數
//attribute是shader與外界(也就是js)溝通的一個橋樑，js可以透過attribute把值傳進來給shader運用
//四元數指的則是vec4，有點像js的陣列，但長度固定為4

attribute vec4 a_position;
 
// 所有著色器都有一個main方法

void main() {
 
  // gl_Position 是一個頂點著色器固有的變數(就像js在瀏覽器中也會有Math這樣的固有物件)
  // 這邊我們把他的值指向我們前面宣告的a_position

  gl_Position = a_position;
}
`
;
const fragmentShaderScript = `

// 這邊mediump是用來定義GPU計算浮點數時的精確度
// 片段著色器没有預設精確度，所以我們需要額外作設定
// 通常精確度會有三種值可以選(highp/mediump/lowp)，但highp在某些系統下會有不支援的狀況，所以一般來說會選用mediump，代表“medium precision”（中等精度）

precision mediump float;
 
void main() {
  // gl_FragColor 是一個片段著色器固有的變數

  gl_FragColor = vec4(1, 0, 0.5, 1); 
  
  // 把所有的頂點都賦予“红紫色”的值，之所以是紅紫色，是因為前面三個數值(1,0,0.5) 換算成rgb,rgb通道都各乘以255，就會是 (255, 0, 127)
}
`;
gl.shaderSource(vertexShader, vertexShaderScript);
gl.shaderSource(fragmentShader, fragmentShaderScript);
...
```

### 6. 編譯Shader腳本

把兩種`Shader`腳本各自編譯成Binary Data，因為如果不這麼做，我們將沒有辦法在接下來的流程使用我們寫好的`Shader`

```javascript
...
gl.compileShader(vertexShader);
// 這邊是防呆用，假如Shader有寫錯，那就回報錯誤狀況
if (!gl.getShaderParameter(vertexShader, gl.COMPILE_STATUS)) {
    console.warn(`vertex shader error!`, gl.getShaderInfoLog(vertexShader));
}

gl.compileShader(fragmentShader);
// 同上防呆用
if (!gl.getShaderParameter(fragmentShader, gl.COMPILE_STATUS)) {
  console.warn(`fragment shader error!`, gl.getShaderInfoLog(fragmentShader));
}
...
```

### 7. 建立WebGLProgram

我們可以把`WebGLProgram`視為前面提到的兩種`Shader`整併起來的*完全體*，他的定位就是**著色程序**。

```javascript
...
function createProgram(gl, vertexShader, fragmentShader) {
  //建立空白的Program
  const program = gl.createProgram();
   //空白的Program連結上編譯好的shader
  gl.attachShader(program, vertexShader);
  gl.attachShader(program, fragmentShader);
  // 把program連接上webgl渲染環境
  gl.linkProgram(program);
  //防呆用，跟上一步驟類似
  gl.validateProgram(program);
  if (!gl.getProgramParameter(program, gl.VALIDATE_STATUS)) {
    console.warn(`validate program failed`, gl.getProgramInfoLog(program));
    gl.deleteProgram(program);
    return;
  }

  return program;
}

const program = createProgram(gl, vertexShader, fragmentShader);

...
```

### 8. 建立緩衝區，並把他綁定到webgl context 上

我猜應該也會有人跟我一樣，第一次看到**緩衝區**(Buffer)這個概念都會覺得很疑惑。<br><br>

這邊可以想像成在`webgl Context` 底下本來就固有`gl.ARRAY_BUFFER` 和 `gl.ELEMENT_ARRAY_BUFFER` 這兩個空槽位。<br>而我們需要指定一個新建立的`buffer`物件，把它放置到空槽位上。<br>
(`buffer`有點類似一個陣列，用來儲存**頂點座標**和**色彩**,...etc.的資料)<br><br>

之所以需要有`buffer`，是因為我們接著需要一次性的向`webgl Context`填充大量的數據。

```javascript
...
const positionBuffer = gl.createBuffer();
gl.bindBuffer(gl.ARRAY_BUFFER, positionBuffer);
...
```

![緩衝區示意](https://i.imgur.com/71zT7ur.jpg)
> 圖片來源: [https://www.jianshu.com/p/f81deec335f1](https://www.jianshu.com/p/f81deec335f1)

### 9. 向buffer一次性填入所有頂點座標資料

這邊的`bufferData`的第一個參數是指定我們前面提到的`webgl Context`底下固有的空槽位之一 --- `gl.ARRAY_BUFFER`，而不是去指向我們剛剛建立的`Buffer`。<br><br>
第二個參數是把一組頂點座標資料以`Float32Array`的格式傳進來。<br><br>
最後第三個參數比較特別，他表示程序將如何使用儲存在`Buffer`中的數據，有三種值可選<br>

- `gl.STATIC_DRAW`:只會向緩衝區寫入一次數據
- `gl.STREAM_DRAW`:只會向緩衝區寫入一次數據,然後繪製若干次
- `gl.DYNAMIC_DRAW`:會向緩衝區多次寫入數據,並繪製多次

這一步的操作，最終導致了`position`這個陣列的資料被傳遞到了與`gl.ARRAY_BUFFER`綁定在一起的`positionBuffer`上。

```javascript
...
const positions = [
  0, 0,
  0, 1.0,
  1.0, 0,
];
gl.bufferData(gl.ARRAY_BUFFER, new Float32Array(positions), gl.STATIC_DRAW);
...
```

### 10. 獲取Shader中的attribute變數儲存位置

這邊可以稍微注意一下，`getAttribLocation`必須要在已經執行過`Buffer`綁定的情況下才可以發動。

```javascript
const positionAttributeLocation = gl.getAttribLocation(program, "a_position");
```
這邊所謂的**位置**有點抽象，但這邊我們其實可以透過我們前面寫到的`Shader`來做一個*小實驗*。

在正常情形下，下面的`Shader`會導致`gl.getAttribLocation(program, "a_position")`返回`0`這個值。

```glsl
attribute vec4 a_position;

void main() {
  gl_Position = a_position;
}
```
但在這個情況下，`gl.getAttribLocation(program, "a_position")`一樣返回`0`這個值，<br>
但`gl.getAttribLocation(program, "b_position")`卻會返回-1。

```glsl
attribute vec4 a_position;
attribute vec4 b_position;

void main() {
  gl_Position = a_position;
}
```

接著再嘗試一種寫法，`gl.getAttribLocation(program, "a_position")`仍然會返回`0`這個值，<br>
但`gl.getAttribLocation(program, "b_position")`這次卻會返回1。

```glsl
attribute vec4 a_position;
attribute vec4 b_position;

void main() {
  gl_Position = a_position;
  gl_Position = b_position;
}
```

所以這裡可以推測，`getAttribLocation`返回的值會跟`Shader`中宣告`attribute`的順序，還有**有沒有在main方法中調用**有關。

有點像是`Shader`中每宣告一個變數，就會產生一個附帶序號的**位置**欄位，這樣的感覺。

### 11. 指定從Buffer中讀取數據的方式

我們在前面有提到，`Buffer`就像一個類陣列的物件，裡面儲存**頂點座標**，**色彩**等資訊。
這邊要特別注意一點，`Buffer`儲存資料的方式其實是把所有資料統統混在一起的。
如果今天同時有**頂點座標**和**色彩**儲存在`Buffer`裡面，內容就會有點像這樣:

```javascript
// 偽code
//這邊只是以js的方式來說明，並不是真的要重新宣告一個positionBuffer
const positionBuffer = [
    '頂點一x座標',
    '頂點一y座標',
    '頂點一色彩r通道值',
    '頂點一色彩g通道值',
    '頂點一色彩b通道值',
    '頂點二x座標',
    '頂點二y座標',
    '頂點二色彩r通道值',
    '頂點二色彩g通道值',
    '頂點二色彩b通道值',
    ...
]
```

所以我們這邊需要去定義，每一個在`Shader`中宣告的`attribute`到底是要怎麼取用buffer中的資料。
但是因為我們這個hello world並沒有把每一點要設置的顏色對外開放，而是在`Shader`中把所有的`gl_FragColor`都設定為紅紫色，所以其實`Buffer`中並不會有色彩的資訊，也就是像下面這樣。

```javascript
// 偽code
//這邊只是以js的方式來說明，並不是真的要重新宣告一個positionBuffer
const positionBuffer = [
    '頂點一x座標',
    '頂點一y座標',
    '頂點二x座標',
    '頂點二y座標',
    ...
]
```

所以接著我們要使用`gl.vertexAttribPointer`來定義儲存在`positionAttributeLocation`的這個位置上的attribute，其從`Buffer`中取用資料的Pattern。

```javascript
// 告訴屬性怎麼從positionBuffer中讀取數據 (ARRAY_BUFFER)
let size = 2;          // 每兩個單位數據算一組頂點座標
let type = gl.FLOAT;   // 每個單位的數據類型是32位浮點型
let normalize = false; // 不需要歸一化數據
let stride = 0;        // stride代表的是一個定點一共會需要多少位的數據
// 如果是有色彩數據參雜的形況，一個頂點就只會有座標資料+色彩資料，也就是5位數據，那就給5
// 但是如果沒有座標以外的數據參雜，也就是每組頂點都是只含有座標的2位數據，則應該給0(這種情形被稱為tightly packed)
let offset = 0;        // 從Buffer的哪一個index作為起始點開始讀取
gl.vertexAttribPointer(
    positionAttributeLocation, size, type, normalize, stride, offset)
```

### 12. 開放attribute為可被取用，並指定著色程序

開放定義儲存在`positionAttributeLocation`的這個位置上的attribute為可被取用。
並且指定webgl Context使用前面建立的著色程序(Program)。

```javascript
...
gl.enableVertexAttribArray(positionAttribLocation);
gl.useProgram(program);
...
```

### 13. 設定清除色，清除過一次畫布之後，就開始根據Buffer上的資料做繪製

這邊的`gl.clearColor`就有點像`2D Context`在做動畫的時候，每一幀都要清除畫布，不過這邊的做法是以特定的顏色填滿畫布，以達到清除的效果。

因為這個hello world要繪製的是三角形，所以`gl.drawArrays`的第一個參數得給`gl.TRIANGLES`。
其餘可選值有:

- `gl.POINTS`: 繪製多個點。
- `gl.LINES`: 繪製一系列的線段。
- `gl.LINE_STRIP`: 繪製一系列連接的線段。
- `gl.LINE_LOOP`: 繪製多節線段，並且把最後一個線段連結回去繪製的原點，形成封閉線段。
- `gl.TRIANGLES`: 繪製一系列的三角形。
- `gl.TRIANGLE_STRIP`: 繪製一系列連接成帶狀的三角形。
- `gl.TRIANGLE_FAN`: 繪製一系列連接成扇狀的三角形。

`gl.drawArrays`的第二個參數是`offset`，也就是要略過多少個**頂點**，注意是多少個**頂點**，而不是多少**位數**。
`gl.drawArrays`的最後一個參數則是頂點數量，因為是三角形所以給3。


```javascript
...
gl.clearColor(1, 1, 1, 1);
gl.clear(gl.COLOR_BUFFER_BIT | gl.DEPTH_BUFFER_BIT);
gl.drawArrays(gl.TRIANGLES, 0, 3 );
```

### 14. 搭拉，恭喜你完成了一個有史以來最長的hello world~

![webgl hello world](https://i.imgur.com/iSkdA58.png)

> Codepen傳送門:[https://codepen.io/mizok/pen/KKRmdyB?editors=0010](https://codepen.io/mizok/pen/KKRmdyB?editors=0010)


寫到這邊大概各位也可以理解webgl的開發其實相當的*hardcore*，我們僅僅是畫了一個三角形，就寫下了約略100行的code，相較於`2D Context`的寫作流程，根本就是大巫見小巫。

到這邊關於webgl的介紹也差不多該打住了，我們已經成功的理解到: 

- 能成功地寫出來一個`webgl`渲染的hello world
- 了解`vertex shader` 跟 `fragment shader` 的差異
- `three.js` ，`webgl`，`3D`的相互關係

接下來就要正式進入`three.js`的環節了。

### 如果還想要繼續研究webgl有沒有推薦的資源?

目前最完整且適合前端開發人員的`webgl`教程只有[這個](https://webglfundamentals.org/webgl/lessons/zh_cn/webgl-fundamentals.html)

不過個人其實覺得這篇教程還是多少對菜鳥不太友善，比方說一些很細微的地方缺乏完善的講解(比方說`vertexAttritubPointer`的 tightly packed狀況)，所以還得多搭配自行google的能力。

其餘的話其實Stackoverflow上面就有很多有相關的討論，大陸的[簡書](https://www.jianshu.com/p/64a53830c53b)上面也有不錯的教程(不過有點不完整)

這邊是只有列出我自己知道的。如果有研究的同好有其他的資源也歡迎提供QQ。


## 延伸閱讀
- [webglfundamentals](https://webglfundamentals.org/webgl/lessons/zh_cn/webgl-fundamentals.html)
- [webgl 缓冲区](https://www.jianshu.com/p/f81deec335f1)
- [webgl 入门(二)](https://www.jianshu.com/p/0db8c54af5c0)
- [webgl 基本图形和基本变换](https://www.jianshu.com/p/09e26d8b6171)
- [MDN 上關於webgl program的介紹](https://developer.mozilla.org/en-US/docs/Web/API/WebGLProgram)
- [MDN 上關於gl.createBuffer的介紹](https://developer.mozilla.org/en-US/docs/Web/API/WebGLRenderingContext/createBuffer)
- [MDN 上關於gl.drawArrays的介紹](https://developer.mozilla.org/en-US/docs/Web/API/WebGLRenderingContext/drawArrays)
- [著色器介紹 - 逍遙文工作室](https://cg2010studio.com/2011/06/29/shader/)
- [使用webpack導入shader的實際範例](https://github.com/mizok/generative-art-playground/blob/master/src/examples/dream/ts/index.ts)
