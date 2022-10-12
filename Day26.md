# Day25 - 打造質感系3D聊天室- 部屬Websocket專案到Fly.io - three.js + socket.io(四)

> 這裡是「Three.js學習日誌」的第26篇，這篇是在講解如何部屬`Websocket`專案到Fly.io。這系列的文章假設讀者看得懂javascript，並且有Canvas 2D Context的相關知識。

> **前情提要** 今天有點小意外~原本筆者是打算在這篇結束掉這個作品，不過因為今天時間上有點緊張，所以只好再把這個部分延長一天。


老實說筆者小弟我今天的進度並沒有對畫面進行改動，主要是因為我把**大半時間**都花費在研究要怎麼部屬`Websocket`專案到最近前端社群很紅的**Fly.io**上面。

> 有點唐突，這邊先跟讀者和評審們說聲抱歉了。


## 1. 什麼是Fly.io

開門見山地講~ **Fly.io**其實就是**Heroku**在**停運免費方案**之後的替代品。

> 不過筆者經過一整天的折騰下來，個人是覺得**Fly.io**果然還是沒有**Heroku**來的好用QQ，不過既然是當個免費仔就別嫌了吧~

如果有定期在追蹤各大前端社群的人應該都知道，**Heroku**將會在今年底以前停止對免費帳戶的支援([IT幫新聞稿](https://www.ithome.com.tw/news/152729))。

> 筆者小弟我是有特地去查了一下之後最低門檻的收費，大約落在750/月(...好貴QQ)

不過對於比較沒有在發摟前端社群的人，甚至是從沒用過或看過**Heroku**的人來講，大概也不會知道**Fly.io**大概是什麼東西，所以這邊筆者還是稍微解釋一下。

相信大多數人應該都有用過**Github Page**的經驗~

> 如果還沒有使用過**Github Page**的話，可以看看這篇[教學](https://medium.com/%E9%80%B2%E6%93%8A%E7%9A%84-git-git-git/%E5%BE%9E%E9%9B%B6%E9%96%8B%E5%A7%8B-%E7%94%A8github-pages-%E4%B8%8A%E5%82%B3%E9%9D%9C%E6%85%8B%E7%B6%B2%E7%AB%99-fa2ae83e6276)

有了**Github Page**，我們在做好前端靜態專案之後，只要使用部屬**Github Page**專用的[**CLI**](https://www.npmjs.com/package/gh-pages)，就可以在按下Enter鍵之後，泡一杯咖啡坐在電腦前看**Github Action**把事情處理好。


而像是**Heroku/Fly.io**這樣的平台，我們其實可以把它看作另外一種類型的**Github Page**服務，差別就在於這種平台是給後端主程序(例如我們的`websocket server`)部屬用的

## 2. 要怎麼使用Fly.io來部屬`websocket`專案?

筆者小弟我當初在查找資料要怎麼樣使用Fly.io來部屬`websocket`專案時，首先是找到了[這篇官方文章](https://fly.io/blog/websockets-and-fly/)，但坦白說這篇文章沒有給我太大的幫助，尤其是文章前面的段落，不知道是沒有跟著官方`CLI`版本做更新還是什麼問題，`flyctl init` 這個指令竟然是無效的操作(*汗顏*)。

筆者小弟我在這邊整理了我自己成功部屬服務到**Fly.io**的方法。


### 2-1 首先當然是安裝官方提供的`CLI`

> 官方的安裝說明文件可以看[這裡](https://fly.io/docs/hands-on/install-flyctl/)

筆者小弟我在安裝的時候是使用**Windows**電腦，所以我打開了我的PowerShell來跑下面這段指令。

> 官方有指定要用PowerShell，用一般的命令提示字元是沒效的。

```
iwr https://fly.io/install.ps1 -useb | iex
```

>要注意，PowerShell是要安裝後才能使用的，一般的Windows沒有內建，不知道怎麼裝可以看這篇[教學](https://www.kwchang0831.dev/dev-env/pwsh)

### 2-2 裝完之後要透過CLI來登入帳戶

```
flyctl auth login
```

> 假如你沒有辦理fly.io帳號，那你在這邊需要先去官網辦理一下。

![img](https://i.imgur.com/X9p9PMB.jpg)


### 2-3 在Fly.io上面建立新的APP專案


首先我們必須要先`cd`到`websocket server`的那個資料夾。

這邊要注意該資料夾必須要是一個獨立的module，有自己的`package.json`和`node_modules`，並且在`package.json`裡面的`script`欄位有寫下名為`start`的啟動服務專用方法。

除此之外，建議不要使用`ts-node`，也就是說~不要使用`ts`來編寫服務主程序，因為筆者小弟我嘗試了多次使用`nodemon`+`ts-node`執行我的`ts`主程序，結果最後都以部屬失敗收場QQ，所以我後來把`index.ts`改回`javascript`，並且用`node.js`執行他。

```javascript
"scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "start": "node index.js",//就是他
    "dev":"npx nodemon index.js"
  },
```

接著我們可以直接在**PowerShell/terminal**打:

```
flyctl launch
```
> 對，別管官方文件，flyctl init 根本沒有效...

在發動完`flyctl launch`之後，**PowerShell/terminal**就會先給你一些Option讓你選填，就像下面這樣。

![img](https://i.imgur.com/cBRT0Qb.jpg)

選填完畢之後就又會開始進入一大長串的上傳流程，筆者小弟我自己的`websocket`程序每次部屬大約也都要花上10分鐘。

>也就是說我在那邊試錯試了不知道多少個10分鐘QQ

成功部屬完之後就像下面這樣。


![img](https://i.imgur.com/e4c3aqg.jpg)

> 這邊會有一環healthy check的檢查特別的長~，要有點耐心


部屬完畢之後回到**Fly.io**的官網**Dashboard**上面就可以看到版本多了一版~

![img](https://i.imgur.com/j0z3nLD.jpg)

> 左邊的紅框就是實際的服務位址，右邊則是在部屬之後會前進一個版本



### 2-4 把`Socket.io`客戶端連到服務位址

這邊我們拿昨天演示過的範例來改~

```javascript
// Create WebSocket connection.

const socket = new WebSocket('這邊改成部屬後的位址');

// Connection opened
socket.addEventListener('open', (event) => {
    socket.send('Hello Server!');
});

// Listen for messages
socket.addEventListener('message', (event) => {
    console.log('Message from server ', event.data);
});
```

這樣就可以啦~

接著就可以看是要把前端的部分部屬到哪裡~大概就是這樣~

>筆者小弟我是直接部屬到了Github Page



## 小結

明天我們會繼續完成這個作品，還希望各位讀者能夠持續追蹤~