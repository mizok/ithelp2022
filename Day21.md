# Day21 - 使用Webpack 5打造Three.js的Boilerplate(二)

> 這裡是「Three.js學習日誌」的第21篇，這篇的內容是要講解Webpack的操作與Webpack config的編寫方法。這系列的文章假設讀者看得懂javascript，並且有Canvas 2D Context的相關知識。

## 3. Webpack的基本用法

首先我們要知道幾件事:

1. `Webpack`必須要在有安裝`Node.js`的環境下執行
2. `Webpack`本身需要透過`CLI`介面來執行，`CLI`必須要被全域安裝，或者是以`devdependency`的形式安裝在`node_modules`中
3. `Webpack`的`CLI`會根據專案裡面的`webpack.config.js`這個文件，來決定要怎麼打包專案
4. 我們必須手動建立並編寫`webpack.config.js`這個檔案

> 其實絕大多數網路上關於`Webpack`的問題討論，基本都是在討論`webpack.config.js`的寫法。

所以我們接下來要簡單介紹一下怎麼樣寫一份`webpack.config.js`。

### 3-1 `webpack.config.js`的基本架構

---

想要學習怎麼編寫`webpack.config.js`，

第一步就是要先記下**EOMMP**這五個英文字。

---

- **E**就是我們剛剛提到的**Entry**
- **O**代表**Output**，意思是打包出來的檔案要輸出到哪裡。
- **第一個M**代表**Mode**，意思是當前我們是在哪一種模式底下做打包。
- **第二個M**就是我們剛剛提到的**Module**

- **P**則代表**Plugin**，意思是**插件**。

下面這是一個常見的`webpack.config.js`檔案，內容看起來的樣子。

```javascript
const { resolve } = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
  //這邊是E
  entry: { 
    main:['./src/js/index.js','./src/scss/index.scss'] 
  }, 
  // 這邊是O
  output: {
    filename: 'js/[name].[chunkhash].js', //輸出的檔案名稱格式
    path: resolve(__dirname, 'build'),
  },
   // 這邊是M(1)
  mode: 'development', // 值可以是'development'或是'production'
   // 這邊是M(2)
  module: {
    rules: [
      //...
    ]
  },
  // 這邊是P
  plugins: [
    new HtmlWebpackPlugin({
     template: './index.html'
    })
  ]
}
```
---

### 3-2 `webpack.config.js`的各屬性解釋

這邊請容我再解釋一下幾個關於上面這段`webpack.config.js`的細節。

#### a. entry屬性的解釋

> webpack 官方文件: [點我](https://webpack.docschina.org/configuration/entry-context/)

`entry`可以填入專案中用用到的**entry files**。

我們會把多個**entry files**構成的集合體稱為`Chunk`。

這邊的`main:['./src/js/index.js','./src/scss/index.scss'] `換句話說就是:

「我們在這個專案中有個叫做**main**的`chunk`，它是由`index.js`和`index.scss`所構成。

#### b. output屬性的解釋

> webpack 官方文件: [點我](https://webpack.docschina.org/configuration/output/)

我們可以在上面的`output`欄位中看到`path: resolve(__dirname, 'build')`這樣的寫法。

這一段的`resolve`和`__dirname`其實是`node.js`提供的方法還有變數。

`resolve`的用途是將一系列**路徑**或**路徑段**解析為**絕對路徑**(詳細請見[文件](https://nodejs.org/api/path.html#pathresolvepaths))。

`__dirname`則是代表`webpack.config.js`所位於的資料夾名稱(詳細請見[文件](https://nodejs.org/docs/latest/api/globals.html#__dirname))。


#### c. mode屬性的解釋

> webpack 官方文件: [點我](https://webpack.docschina.org/configuration/mode/)

`mode`表示我們當前是在哪一種**模式**底下做打包。

基本上可以選擇為，**development**(開發模式) 或 **production**(生產模式)。

不同的**模式**採用的打包方式會不同，產生的結果其細節也會不一樣。

例如我們可以選擇只在**生產模式**下對打包的檔案進行**極小化**，以減少檔案容量。

#### d. module.rules 的解釋

> webpack 官方文件: [點我](https://webpack.docschina.org/configuration/module/#modulerules)

這個部分主要是去定義當`webpack`偵測到資源的時候，要怎麼去**載入**或**編譯**這個資源。

例如假設我們如果要載入`.SCSS`資源。最簡單的方式就是像下面這樣設置:

```javascript
module: {
    rules: [
      {
        test: /\.scss$/,
        use: [
          'style-loader',
          'css-loader',
          'sass-loader'
        ]
      },
    ]
},

```

我們可以注意到這邊的`use`使用了三個`Loader`，分別是`style-loader`、`css-loader`、`sass-loader`。 但實際上這三個`Loader`的執行順序並不是由上往下，而是由下往上。意思也就是說當`webpack`偵測到`SCSS`資源的時候:

- `sass-loader`會去把`SCSS`轉譯為`CSS`。
- `CSS-loader`會去偵測`CSS`內部的`url()`和`@import`轉換成`require()`，目的是為了讓`js`可以認知到`CSS`內部有使用其他資源。
- `style-loader`則是把`CSS`插入`DOM`中

上面的寫法只是一個範例，實際上我們會因為需要應對專案的各種需求去決定`module.rules`的寫法，而不同類型的資源往往也會有不同的`Loader`需要使用。

> 通常`Loader`會是由`webpack`社群的使用者一起協力開發，不過像這樣的開源插件常常有的毛病就是:*更新可能不及時*(~~或是作者人間蒸發~~)，所以通常要選用`
Loader`插件的時候，都要盡量觀察插件NPM頁面/Github Repo的更新狀況和健康程度，要升級版本時也最好都要先確認過Patch note。

#### e. plugin屬性的解釋

> webpack 官方文件: [點我](https://webpack.docschina.org/configuration/plugins/)

老實說`plugin`這個屬性並沒有一個一定的解釋。

這個欄位就是用來讓我們導入一些插件，這些插件會在`webpack`建構打包內容的時候造成影響。

這邊我們可以介紹幾個常用的插件:

- `webpack.DefinePlugin`: 這是一個`webpack`官方內建的插件，我們可以用它來定義在不同的`mode`時，產生不同的值作為環境變數。

- `MiniCssExtractPlugin`: 這是一個可以用來把`module.rules`中偵測到的`CSS`字串資源抽取出來成為獨立的`CSS`檔案。

- `CopyPlugin` : 這是一個可以把指定資料夾裡面的內容，複製到另外一個資料夾的插件，通常會使用這個插件，大多是因為有些檔案需要在不被轉譯的情況下使用，所以必須要繞過`module.rules`>`Loader`的階段，直接把檔案複製到`output`的路徑上。

- `HtmlWebpackPlugin`: 這個插件可以透過傳入`Entry Chunk`的名稱，還有一個`html`模板的路徑，來把`Entry Chunk`的內容跟該`html`模板合併輸出一個`html`檔案。(意思就是可以把`Entry Chunk`裡面的`js`和`css`塞到目標的`html`裡面，變成一個有導入樣式表和JS的`html`檔)。

> 除了上述這些以外當然還有很多其他的Plugin，狀況跟Loader一樣。


## 4. 筆者自行開發的模板「webpack-template」，其使用方法介紹

老實說 ~ 筆者其實沒有打算一步一步的去講解到底怎麼樣去寫好一個`webpack`模板。

>這邊可能讓大家失望了QQ

但畢竟競賽的主題不是`webpack`，而是`Three.js`，而且如果真的要講完「webpack-template」的開發過程，我預估大概也要講到賽程結束...

因此關於`webpack`操作的細節，我只打算點到上面為止。

但是因為在接下來幾天的範例，筆者小弟我都會使用我自己開發的`webpack`模板:「webpack-template」來進行範例的實作。

所以我打算在這邊講解一下我所開發的「webpack-template」的使用方法，還有一部分的機制與原理。


### 4-1 首先當然是下載/安裝的方法

Github Repository: https://github.com/mizok/webpack-template

---

1. 請先登入`Github`後移步至上述的`Repository`地址，然後點擊畫面中的綠色"Use this template"按鈕

![img](https://i.imgur.com/lO4pEQI.jpg)

2. 接下來就會進入建立`Repository`的頁面，`Github`將會直接使用筆者開發的`webpack-template`作為模板來在您的帳號名下建立一個`Repository`。

![img](https://i.imgur.com/w3m8dNL.jpg)

3. 建立好`Repository`後就可以`git clone`下來。

> git操作的部分如果不熟悉，可以看看[這邊](https://www.youtube.com/watch?v=CKcqniGu3tA)


### 4-2「webpack-template」介紹

這個模板主要使用的技術/語言有:

- `webpack5` (作為主打包程序)
- `typescript`
- `scss`
- `ejs` (作為主樣板語言)

> 在[Github Repo](https://github.com/mizok/webpack-template)上面，您其實可以檢閱英文的Readme文件，而若有問題，也歡迎隨時發布ISSUE或投遞PR ~
---

#### 4-2-a 基本NPM Script

- `npm run build` : 會呼叫`webpack` 的 `CLI`工具執行專案打包
- `npm run dev` : 會使用內置的`webpack-dev-server`來模擬專案的Hosting，用途類似`VScode Live Server`
- `npm run deploy` : 會使用`gh-pages`這個插件把專案打包後部署到`Github Page`

#### 4-2-b 打包/編譯流程圖

![img](https://i.imgur.com/xGsTBqX.png)


#### 4-2-c 重點機制:「由檔案名生成Entry Chunks/HtmlWebpackPlugin Instance」

在「webpack-template」中，當使用者按下Enter送出`npm run build`指令後，首先會先在`webpack.config.ts`裡面碰到一層判斷。

這層判斷會去根據當前每個`./src/pages/`底下的`EJS`檔案的命名方式，來決定要生成哪些`Entry Chunks`。

例如: 

- `index.ejs` 會使程序判斷必須要生成一個被命名為`index`的`Entry Chunks`。
- `index.main.ejs` 會使程序判斷必須要生成一個被命名為`main`的`Entry Chunks`。

而當如果專案備程序判定將生成了一個名為"xx"的`Entry Chunks`時，我們就必須要在:

- `./src/ts` 底下補上一個名為`xx.ts`的檔案(作為`ts entry`)
- `./src/scss` 底下補上一個名為`xx.scss`的檔案(作為`scss entry`)


#### 4-2-d 靜態檔案的堆放處: ./static資料夾

我們之前提到的`CopyPlugin`，會把在`./src/static`資料夾底下的所有檔案，一併複製到`./dist/static`。

通常我們會把一些`.pdf`或是`.obj`、`.mtl`、`.gltf`等外部3D模型檔案堆放在這個資料夾，以避免它們被`Loader` 加載到。


> 其他像是有關於EJS的寫作方法、環境變數的取用之類的Tips，都可以在Repo的README進行查閱~ 這邊就不多做說明。

## 小結

對於`webpack`/「webpack-template」的介紹就差不多在這先打住。
明天我們將著手從一個空白的「webpack-template」模板來改造成為我們在接下來的天數會使用到的`Three.js Boilerplate` !