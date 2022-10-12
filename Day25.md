# Day25 - 打造質感系3D聊天室 - three.js + socket.io

> 這裡是「Three.js學習日誌」的第25篇，這篇是在講解使用three.js + socket.io打造3D聊天室作品。這系列的文章假設讀者看得懂javascript，並且有Canvas 2D Context的相關知識。

今天我們的目標是要來完成**時鐘面板**還有兩個地方的**聊天室面板**~

>當然還有socket.io的連線機制

話不多說~就讓我們先來看看今天的完成狀況吧~

![img](https://i.imgur.com/q21gl3B.gif)

>這邊我還順便補了一個能夠用來停止方塊旋轉的按鈕


## 1. 首先來看看停止方塊旋轉按鈕的實作

筆者昨天在實際試用的時候發覺... 

果然~還是需要有一個停止方塊運動的手段，畢竟這玩意一直自旋，想要看到某一畫面還得一直動手去轉，真的很煩XD。

> 謎:明明是你自己設計的。

所以我後來還是決定加上一個禁止自旋的按鈕~

![img](https://i.imgur.com/iQVYZO6.jpg)

不過，因為我們在程式架構上一開始的設計是讓`./src/ts/main.ts` 底下的`main`類別去`extends``base`類別，而`domCube` 和 `Cube`又是去偵測`base`底下的屬性(`touched`)來決定要不要停止旋轉的。

所以這邊的作法就必須要像下面這樣。

![img](https://i.imgur.com/xM3vq3T.jpg)

我們在`base`類別的外面宣告一個變數，用來存放**禁止方塊旋轉**的狀態。

![img](https://i.imgur.com/rfP4wFb.jpg)

然後在`base`裡面建立`get/set`取向的方法，這樣`rotationlocaked`就不會被子類別繼承到。

接著就可以在`domCube` 和 `Cube`裡面去用`base`提供的`get/set`方法來偵測**禁止方塊旋轉**的狀態。

![img](https://i.imgur.com/m2dhh8m.jpg)

> 看起來很簡單吧~


## 2. 來個可愛的Space Inavader

我們目前還有**2**個空白的方塊面板沒有拿來用，所以我打算拿其中一個來放一隻吉祥物XD。

![img](https://i.imgur.com/4cC5H7e.gif)

> 剩下一面明天再來思考要放些什麼好了XD

這邊動畫的實作我其實不是使用`canvas`。而是直接使用`CSS @keyframe`

![img](https://i.imgur.com/6FTdyem.png)

簡單來說原理就是利用`CSS @keyframe`去動態改變`background-image`，然後把所有的動畫幀都用`Photoshop`輸出成PNG圖串，每過幾毫秒就變動一次`background-image`。

> 其實就是很簡單的特效，不過其實蠻不錯的吧?

## 3. 實作時鐘的部分

因為我們之前的時鐘其實還一直是只有假字串的狀態，今天把這部分處理完了，雖然這部分其實蠻簡單的，不過還是來看一下大概是怎麼處理~

![img](https://i.imgur.com/GFW4ZSX.jpg)

> 簡單來說就是在`./src/ts/dom/clock.ts`底下建立渲染畫面的方法，還有`setInterval`的計時器~ 


## 4. 聊天室部分

最後就是聊天室的部分了~!

首先我感覺應該還是有必要來介紹一下`Socket.io`的用途，還有這部分我在架構上的規劃。

### 4-1 什麼是`socket.io`? 他跟`Websocket`有什麼關係?

`socket.io`跟`Websocket`的關係有一點點像`jquery`和`javascript`，也就是包裝庫的概念。

#### 4-1-a 那`Websocket`又是什麼呢?

`Websocket`其實是一種通訊協定，就像我們一般瀏覽網頁用的`HTTP`協定一樣。

但差別在於`HTTP`的連線機制是:

- 會在每次和伺服器互動的時候，建立「**請求**(request)」

- 伺服器確認這個請求之後，就會「**回應**(response)」對應的資料

而`Websocket`的機制則是:

- 只要在初期跟伺服器互動的時候，作出一次「**交握**(connection)」，接下來客戶端和伺服器端就可以自由的互相傳遞資料。


![img](https://i.imgur.com/8PO7hdy.png)


當我們在瀏覽器的`Console`介面打上`Websocket`這個字的時候，我們其實可以看到瀏覽器環境底下是具備這個同名的函數的。

![img](https://i.imgur.com/tupSX7z.jpg)

而這個函數的用途其實只是用來實作`Websocket`程序的**客戶端部分**。

```javascript
// Create WebSocket connection.

const socket = new WebSocket('ws://localhost:8080');

// Connection opened
socket.addEventListener('open', (event) => {
    socket.send('Hello Server!');
});

// Listen for messages
socket.addEventListener('message', (event) => {
    console.log('Message from server ', event.data);
});
```

以上面這張圖片來說，我們可以透過該函數建立客戶端的`Websocket`實例，並把該實例的伺服器位址指向`localhost:8080`。

這樣做意思就是說在`8080`上面其實已經有一台架設好的`websocket server`，而我們這邊只是引導客戶端跟這台**Server**執行「交握」

交握完畢之後，我們在客戶端就會收到`Websocket`伺服器傳來的信號。

>就有點像我們前面提到過的EventEmitter。

這樣我們就可以決定要在什麼信號發生的時候去對前端畫面做什麼事。

而當然我們也可以從客戶端去發送訊息給`websocket server`，像是下面這樣。

```javascript
socket.send(data)
```

#### 4-1-b `Websocket`伺服器看起來像什麼樣子?

所謂的`伺服器`其實就是一台電腦，上面搭載的一段程序。

所以要建立伺服器不一定要使用另外一台電腦，通常測試開發階段我們都是在本機同時`run` `websocket server` 和 `websocket client端`。

而如果要編寫`websocket server`的程序，其實有很多種電腦語言都可以辦到，但我們這邊當然就是要選用前端工程師比較常聽到的`node.js`。

#### 4-1-c 使用`socket.io` 和使用`Websocket`的差別在哪?

首先當然就是**效率**。

在正常的使用狀況下，想要跟`Websocket`伺服器互動還是有分很多的**場景**，例如多方發送/廣播這種狀況， 使用`Websocket`就要自己**重新造輪子**。

> 因為`Websocket`就只有指定對象的收跟發，如果要指定對全場成員發送，那就要自己寫出這樣的功能。

### 4-2 架構上的規劃

![img](https://i.imgur.com/OCBF4Ao.jpg)

我自己的習慣是如果專案的規模不會很大，那就在同一個`Repo`底下開一個` server`程序 的資料夾，這邊我把它命名為`chat`。

`chat`底下會是一個獨立的`module`，所以在建立的時候我們必須要先:

```
cd chat && npm init
```
接著安裝`socket.io`的server端 `npm` 包。

```
npm i socket.io
```

另外我們需要有一個主程序，所以我建立了一個`index.ts`。建立完之後其實我們已經可以用`node.js`去執行這支`index.ts`，就像下面這樣。

```
node index.ts
```

**不過**，因為像這樣的執行方法，其實不會偵測`index.ts`本身的改動，並發動重載。所以我們這邊要改用`nodemon`來執行這個主程序。

> nodemon 就有點像是給node.js程序用的live server

這邊先來安裝`nodemon`

```
npm i nodemon 
```

> 要注意到這邊為止我們都還是位於./chat底下唷。

#### 4-2-a `socket.io`伺服器主程序

接下來就是主程序`index.ts`的部分。 

>這邊我其實是參考**Jia-yun Yang** 寫的 [「用 Socket.IO 打造多人聊天室」](https://viboloveyou12.medium.com/%E7%94%A8-socket-io-%E6%89%93%E9%80%A0%E5%A4%9A%E4%BA%BA%E8%81%8A%E5%A4%A9%E5%AE%A4-%E4%B8%8A-e601f411d2a7)這篇文章，然後自己修改成`TS`版本。


`./chat/index.ts`


```typescript
import { createServer } from "http";
import { Server } from "socket.io";

const app = createServer()

//由於
const io = new Server(app, {
    cors: {
        origin: "http://192.168.1.101:8080", //這是我自己的IP
        methods: ["GET", "POST"]
    }
})
/*自訂監聽端口*/
const port = 5500;
app.listen(port);

console.log('app listen at ' + port)


/*用戶陣列*/
const users: { username: string }[] = [];

// 交握
io.on('connection', (socket) => {
    /*是否為新用戶*/
    let isNewPerson = true;
    /*當前登入用戶*/
    let username: string = null;

    //監聽登入
    socket.on('login', (data) => {
        // 遍歷檢查有沒有同名的用戶，若沒有則代表新用戶
        for (var i = 0; i < users.length; i++) {
            isNewPerson = (users[i].username === data.username) ? false : true;
        }
        // 為新用戶在陣列中建立資料
        if (isNewPerson) {
            username = data.username
            users.push({
                username: data.username
            })
            data.userCount = users.length 
            /*發送 登入成功 事件*/
            socket.emit('loginSuccess', data)
            /*向所有連接的用戶廣播 add 事件*/
            io.sockets.emit('add', data)
        } else {
            /*發送 登入失敗 事件*/
            socket.emit('loginFail', '')
        }
    })

    //監聽登出
    socket.on('logout', () => {
        /* 發送 離開成功 事件 */
        socket.emit('leaveSuccess')
        // 從陣列中剔除離開的用戶資料
        users.map((val, index) => {
            if (val.username === username) {
                users.splice(index, 1);
            }
        })
        /* 向所有連接的用戶廣播 有人登出 */
        io.sockets.emit('leave', { username: username, userCount: users.length })
    })
    // 當用戶發送訊息到聊天室
    socket.on('sendMessage', function (data) {
        /*發送receiveMessage事件*/
        io.sockets.emit('receiveMessage', data)
    })
})

```

值得一提的是在`socket.io v3`版本之後，我們會需要在伺服器端上建立`CORS`的白名單。

> 如果不懂什麼是CORS可以看[這邊](https://shubo.io/what-is-cors/)

```typescript
const io = new Server(app, {
    cors: {
        //http://192.168.1.101:8080是我自己當前的區網IP，
        //我把server架設在http://192.168.1.101:5500
        // 客戶端的port則是調整了webpack的host設定，定在了8080
        origin: "http://192.168.1.101:8080", 
        methods: ["GET", "POST"]
    }
})

```

白名單的意思也就是說「不會去排除特定網域的客戶端交握行為」。如果我們剛剛在上面沒有設定`cors`這個屬性，那在我們按下登入按鈕的時候就會出現下面這個錯誤。

![img](https://i.imgur.com/V2QoC5s.jpg)

#### 4-2-b `socket.io`客戶端

最後就是客戶端的部分了。

我目前是先把`socket.io`客戶端的程序放在 `./src/ts/main.ts`裡面，這邊我們來看看都寫了些什麼~

```typescript
import { Base } from './class/base';
import { io, Socket } from 'socket.io-client';
import { trim } from 'lodash';
class Main extends Base {
    private wrapper: Element = document.querySelector('#wrapper');
    private chatBlock: Element = document.querySelector('#chat-block');
    private chatBlockActive = false;
    private socket: Socket = io('ws://192.168.1.101:5500');
    private myName: string;
    constructor(canvas: HTMLCanvasElement, domCanvas: HTMLElement, domBundle: HTMLElement) {
        super(canvas, domCanvas, domBundle);
        this.initChatUI();
        this.initChatSocket();
    }
    // ui操作後發動的事件綁定
    private initChatUI() {
        const toggler = this.chatBlock.querySelector('#chat-block-toggler');
        const rotationLock = this.chatBlock.querySelector('#rotation-lock');
        const loginBtn = this.chatBlock.querySelector('#login-button');
        const sendBtn = this.chatBlock.querySelector('#send-message-button');
        const logoutBtn = this.chatBlock.querySelector('#logout-button');
        toggler.addEventListener('click', () => {
            if (this.chatBlockActive) {
                this.wrapper.classList.remove('wrapper--active');
            }
            else {
                this.wrapper.classList.add('wrapper--active');
            }
            this.chatBlockActive = !this.chatBlockActive;
        })

        rotationLock.addEventListener('click', () => {
            const status = this.getRotationLockStatus();
            if (status) {
                rotationLock.classList.remove('chat-block__rotation-lock--active');
            }
            else {
                rotationLock.classList.add('chat-block__rotation-lock--active');
            }

            this.toggleRotationLock(!status)

        })

        loginBtn.addEventListener('click', () => {
            this.myName = trim((this.chatBlock.querySelector('#login-name') as HTMLInputElement).value);
            if (this.myName) {
                /*發送事件*/
                this.socket.emit('login', { username: this.myName })
            } else {
                alert('Please enter a name :)')
            }
        })

        sendBtn.addEventListener('click', () => {
            this.sendMessage();
        })

        logoutBtn.addEventListener('click', () => {
            let leave = confirm('Are you sure you want to leave?')
            if (leave) {
                /*觸發 logout 事件*/
                this.socket.emit('logout', { username: this.myName });
            }
        })

        document.addEventListener('keydown', (evt: KeyboardEvent) => {
            if (evt.keyCode == 13) {
                this.sendMessage()
            }
        })


    }
    private initChatSocket() {
        /*登入成功*/
        this.socket.on('loginSuccess', (data) => {
            if (data.username === this.myName) {
                this.checkIn(data)
            } else {
                alert('Wrong username:( Please try again!')
            }
        })

        /*登入失敗*/
        this.socket.on('loginFail', () => {
            alert('Duplicate name already exists:0')
        })

        /*加入聊天室提示*/
        this.socket.on('add', (data) => {
            var html = `<p>${data.username} 加入聊天室</p>`
            // $('.chat-con').append(html);
            document.getElementById('chat-title').innerHTML = `在線人數: ${data.userCount}`
        })

        //離開成功
        this.socket.on('leaveSuccess', () => {
            this.checkOut()
        })

        //退出提示
        this.socket.on('leave', (data) => {
            if (data.username != null) {
                let html = `<p>${data.username} 退出聊天室</p>`;
                // $('.chat-con').append(html);
                // document.getElementById('chat-title').innerHTML = `在線人數: ${data.userCount}`;
            }
        })

        //收到訊息
        this.socket.on('receiveMessage', (data) => {

            this.showMessage(data)
        })


    }
    private checkIn(data: any) {
        const loginWrapper = this.chatBlock.querySelector('#login');
        const userNameEle = this.chatBlock.querySelector('#my-name');
        userNameEle.innerHTML = data.username;
        loginWrapper.classList.add('login--logined');
    }

    private checkOut() {
        const loginWrapper = this.chatBlock.querySelector('#login');
        loginWrapper.classList.remove('login--logined');
    }

    private sendMessage() {
        const inputEle = this.chatBlock.querySelector('#message-input');
        const message = (inputEle as HTMLInputElement).value;
        (inputEle as HTMLInputElement).value = ''
        if (message) {
            /*觸發 sendMessage 事件*/
            this.socket.emit('sendMessage', { username: this.myName, message: message });
        }
    }

    private showMessage(data: any) {
        let html;
        if (data.username === this.myName) {
            html = `<div class="chat-main__chat ">
                        <div class="chat-main__bubble-name">You</div>
                        <div class="chat-main__bubble">${data.message}</div>
                    </div>
                    `;
        } else {
            html = `<div class="chat-main__chat chat-main__chat--other">
                        <div class="chat-main__bubble-name">${data.username}</div>
                        <div class="chat-main__bubble">${data.message}</div>
                    </div>
                    `;
        }
        const ele = this.createElementFromHTML(html)
        this.chatBlock.querySelector('#chat-main').appendChild(ele);
        this.wrapper.querySelector('#chat-main-cube').appendChild(ele.cloneNode(true));
    }

    private createElementFromHTML(htmlString: string) {
        const div = document.createElement('div');
        div.innerHTML = htmlString.trim();

        // Change this to div.childNodes to support multiple top-level nodes.
        return div.firstChild;
    }

}

```


基本上我這邊的區塊劃分就是把「`UI`操作之後綁定的動作」，和「`socket`偵測到某特定信號之後的操作」個別分成一類。

> 筆者個人覺得把特定形式的程式碼分在同一個方法會相對地比較好整理。


## 小結

我們這個作品已經接近完成了~ 大概明天就會是這個部分的最後一篇，再麻煩各位持續追蹤~

