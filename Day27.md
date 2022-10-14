# Day26 - 打造質感系3D聊天室 - three.js + socket.io(五)

> 這裡是「Three.js學習日誌」的第26篇，這篇是「打造質感系3D聊天室」系列的最後一篇。這系列的文章假設讀者看得懂javascript，並且有Canvas 2D Context的相關知識。

今天我們要來收尾**3D聊天室**這個部份了~ 

首先我們先來看看**完成**的樣子。

![img](https://i.imgur.com/O6VX7zI.gif)

| ![img](https://i.imgur.com/BkHR2JR.jpg) | ![img](https://i.imgur.com/Kh3Xu8c.jpg)  | 
|---|---|
|![img](https://i.imgur.com/T9HbPSo.jpg)|![img](https://i.imgur.com/SUlpGA5.jpg)|


## 1. 是音樂!我加了音樂!


沒錯~! 最後一面我想來想去，最後聽了公司主管的意見，決定直接丟一個內嵌的ifrmae上去XD。

> 放的是我最喜歡的Johnny Cash 在Sound Cloud 的專輯~ 



##  2. 其他面機能的改動

我*前天*晚上**左思右想**，想到最後還是推翻掉了自己原本「**兩邊的聊天室會同時出現**」的構想。因為連自己都覺得同時有兩個聊天室很多餘...(汗顏)。

> 謎:真的很愛出爾反爾這人。

方塊上的聊天室，現在在側選單聊天室展開的時候會**自動隱藏**，並且會在每次有人**進場/退場**的時候從`websocket`伺服器取得當前房間內**遊客名單**，然後渲染成`HTML`元素植入到畫面上；而當如果使用者退出登入狀態的話則會顯示**登入提示**。

我在這邊把**遊客名單**/**登入提示**/**聊天室畫面**拆成三層來做，簡單來說就是放了三個`DIV`，然後弄成有點像`TAB`結構的形式，然後再寫一個簡單的`API`
來操作這個部分。

```typescript
//輸入特定的代號，來從三個選擇中顯示指定的畫面
showScreen(target: ShowScreenTargets) {
        const hideClass = 'chat-main__inner--hide';
        const targets = ['chatMainInner', 'loginGuide', 'guestList'];
        this[target].classList.remove(hideClass);
        targets.filter((val) => val !== target).forEach((unchosen) => {
            (this[unchosen as keyof this] as (HTMLElement | any))?.classList.add(hideClass)
        })
    }
```

## 3. CSS3DRenderer 渲染出來的元素不能觸發Click事件?

在處理**登入提示**畫面上的按鈕的時候我注意到了一個*挺詭異*的問題，那就是:「**透過CSS3DRenderer做過3D變形的元素，竟然不會觸發點擊事件(但卻可以Hover)**」。

當時發現這個問題的時候我都還沒有把音樂的`iframe`加上去~很明顯這個問題*迫在眉睫*，畢竟要是確定點擊事件都不能觸發，那肯定`iframe`也不會有效果。

>心想芭比Q了

所以我仔細的在`Google`搜索了很多遍，發現很多過去的版本都有類似的問題，但是細節上卻有微妙的差異。

- 有些人說是**更版**的時候有錯誤沒有修復，只要**退版**就可以了
- 有些人說是因為部分的元素沒有補上`pointer-events:auto`，結果被父層的元素上的`pointer-events:none`給影響到了。
- 有人說是因為`CSS3DRenderer` 同時使用多層`matrix3d`導致沒辦法正確取得點擊位置。
- 其他還有各種說法...不過大部分都不管用。

我最後則是看到了[這篇](https://discourse.threejs.org/t/onclick-is-not-working-on-css3dobject/31108)。

上面提到`CSS3DRenderer`所產生的元素必須要改成偵測`pointerdown`這個事件，而不是`click`事件。

> 但意外的是我在後來把iframe放上方塊上，iframe的點擊事件卻毫無受到影響。。

```javascript
element.addEventListener('pointerdown', () => { alert(1) })
```

> 不過老實說我目前也還沒找到為什麼`click`會被block掉，如果有人曉得問題的答案，希望能透露一下~



## 後記&小結

想當初我壓根沒想到這部分會花這麼多時間QQ，接下來的**賽程**還有兩天，希望這兩天足夠讓我做下一個創作。

關於這個聊天室作品，筆者目前是有規劃在**賽程**結束之後要繼續優化這個專案。

這邊是一些我自己的構想:
- 音樂的部分也許會把這部分實作成可以自己上傳歌單的系統~
- 想要嘗試加上音軌偵測，做一些跟音頻有關連的動態
- 聊天室的部分感覺還有很多優化空間，畢竟目前的功能還有些陽春。


Repo地址　:　[https://github.com/mizok/3d-Cube-Chat](https://github.com/mizok/3d-Cube-Chat)


Github-Page :  [https://mizok.github.io/3d-Cube-Chat/](https://mizok.github.io/3d-Cube-Chat/)



> 如果對作品內容有感興趣，或是有相關建議的歡迎聯絡我XD
