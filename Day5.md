# Day5 - 事前健身操 - 向量與形變
> 這裡是「Three.js學習日誌」的第5篇，本篇的主旨是要介紹一些在Three.js中，一些常用的基礎操作，這系列的文章假設讀者看得懂javascript，並且有Canvas 2D Context的相關知識。

想要學習`Three.js`，首先當然要先練習一些基本的操作，這邊我們會先從`形變`，`向量` 開始。

---

## 向量

 `Three.js`的向量類一共有3種，分別是`Vector2`，`Vector3`，`Vector4`，它們各自代表著二三四維空間的向量。

> 如果你對**向量**這個名詞很不熟悉，那麼你也許可以看看我去年寫的[文章](https://ithelp.ithome.com.tw/articles/10268826)

這些**向量**物件都各自有提供各自的運算方法，這邊我們來介紹幾個比較常用的操作:

- `.add`/`sub`/`multiply`/`divide`:也就是向量的四則運算，要特別注意的一點是，這些運算方法都不是`pure function`，他們會回傳已經做過運算的自身。
- `.length`: 也就是求取向量的**絕對值**，也就是**長度**，例如假設有一個向量是(a,b)，那麼它的絕對值就是√(a^2+b^2)
- `normalize`: 單位向量化，意思就是說把向量轉變成一個方向不變，但是長度只有1的向量。`normalize`最大的用處通常在於決定`朝向`，像是如果要指定往某個方向射出一顆球，通常就是先設法取得射出方向的單位向量，接著就可以再透過`multiply`來決定速度的大小。
- `lerp`: 其實就是大家國高中都學過的`內插法`，或是叫做`線性映射`。假設今天我們用(a,b)這個向量發動`lerp`方法，那我們要傳入一個`端點向量`(c,d)，還有一個浮點數(假設是0.6)，這樣意思就是要去取得(a,b)轉變到(c,d)的過程中，處於60%變化的那個點所代表的向量(特別注意，`lerp`本身也不是`pure function`)。
- `set`: 最後介紹的是最常用的set，顧名思義就是可以直接設定`xyz`座標值。

> 四維的向量比較特別一點，這個接著會提到。

## 形變

 說到形變，那當然就是`Translate`/`Scale`/`Rotate`三劍客。常常寫`CSS`的前端工程師應該都跟它們混得很熟。

 ### 平移(translate)

 `Three.js`的**平移**，有2種比較常見的方式。

 其中一種是我們在前面的**Hello World**也用過的`Object3D.position`。

 `Object3D.position`本身的型別是`Vector3`，也就是**三維向量物件**，所以理所當然的，我們可以用上面介紹過的各種方法，比方說**四則運算**來定義物體的位置。

 第二種則是只有`Geometry`類才可以使用的`translate`方法。
 
 假設我們把`mesh`比喻成一個3D物件的"外層"，`Geometry`則是它的"內層"，當`Geometry`被下了`translate`時，它其實不會影響到`Mesh`的`position`屬性。

 關於`Geometry.translate`有一點要注意的就是它是一種`one-time-operation`，意思就是說它只能在生成`Mesh`之前被操作一次，`renderer`在render的時候並不會去偵測`Geometry.translate`的改變。
 ### 縮放(scale)

`縮放`其實跟**平移**差不多，有`Object3D.scale`也有`Geometry.scale`(`Geometry.scale`也同樣是`one-time-operation`)，這邊就不描述太多。
 ### 旋轉(rotate)

 首先跟前面一樣，`Three.js`的**旋轉**，也同樣有`Object3D`和`Geometry`的版本。

 但旋轉其實是形變中最複雜的一個部分，為什麼說它複雜?

 首先我們來看一下官方文件`Object3D.rotation`這個屬性。

 ![img](https://i.imgur.com/x8KHcPP.png)

 它的型別叫做`Euler`，這個`Euler`代表的是**歐拉角**， 所謂的**歐拉角**可以想像成一種**朝向**。

 這邊我自己畫了一張簡單的圖來解釋，圖片左邊是一個球體，它的球心剛好就落在座標軸原點(0,0,0)上。

 ![img](https://i.imgur.com/cBIxJnK.png)
 
 > 噢對了，Three.js的座標軸基本上就跟這張圖左半邊一樣，Z軸會朝向螢幕，而y軸則是朝向上方。
 
 在這個時候，我們可以把靜止的xyz三個軸當前所構成的**朝向**視為一種狀態，我們把它的數值訂為(0,0,0)，而接著當我們把這個坐標系跟著球一起旋轉，直到變成圖片右邊x'y'z'的狀態時，我們可以用一組參數(a,b,c)，來表示xyz軸各被轉動了多少度(Radian)，這組參數(a,b,c)就是**歐拉角**的數值化定義。

 > 透過坐標系跟著球一起旋轉所定義出來的這種歐拉角，我們稱之動態歐拉角。既然說到動態，那麼當然也就有靜態，不過靜態歐拉角跟Three.js比較沒有關係，所以這邊不特別提。

 這邊其實有一點要特別注意的:

```
 歐拉角旋轉其實存在順序差異，也就是先轉x軸，再轉y，再來z，還是先y再z接著x，最後的結果會有差。
```
`three.js`其實有提供一個方法可以讓使用者重新定義旋轉順序。

```javascript
Euler.reorder(string)
//可以輸入例如'XYZ','YXZ'這樣的字串值來決定旋轉的順序
```
 
 到了這邊，好像還不是很複雜? OK ~

 ![img](https://i.imgur.com/wgaZtak.jpg)

 接著我們要來講講歐拉角旋轉的一種異常狀況: **環架鎖定**(gimbal lock)

**環架鎖定**是一種只有動態歐拉角(就是坐標系跟著物體一起旋轉)才會出現的一種BUG。

>這邊我思考了很久，不過還是想不到有比影片更快更好的解釋方法，所以還是決定放上影片了。

[![Yes](https://img.youtube.com/vi/rsKy-4dbA04/0.jpg)](https://www.youtube.com/watch?v=rsKy-4dbA04)

**環架鎖定**的問題通常是發生在動畫中。例如鏡頭的旋轉，在某個時間點突然變得很不自然，但是實際排查程序卻又找不到問題(因為問題是來自於動畫補間所產生的數值變化)。

要避免**環架鎖定**問題，最好的方法就是使用**另外一種旋轉方法**，也就是**四元數**(Quaternion)。

> 於是又出現了一個看不懂的名詞，這次是真的看不懂QQ

所謂的**四元數**其實是一個數學的專有名詞，通常大量的出現在大學線性代數的課本裡面，它的意義就是用來代表**四維空間中的一個點**，但是它為什麼可以用來表示一個3D物體的旋轉呢?。

> 這可能要請真的有修過線性代數的人來解釋了，小弟我只有修過工數，實在不敢隨便誤人子弟QQ。

`Three.js` 的**四元數**類可以在`Object3D`底下找到:

```javascript
Object3D.quaternion
```
`Object3D.quaternion`最常用到的方法之一就是`quaternion.setFromAxisAngle`，這個方法可以透過輸入一個旋轉軸向量，搭配一個角度(Radian)，這樣就可以實現繞軸轉動。

## 小結

關於向量與形變的描述差不多就到這邊，接著下一部分會講到顏色/動畫循環/群組 等操作。

## 延伸閱讀

- [https://openhome.cc/Gossip/ComputerGraphics/QuaternionsRotate.htm](https://openhome.cc/Gossip/ComputerGraphics/QuaternionsRotate.htm)

- [https://read01.com/OAggzM0.html#.Yysqm3ZByw4](https://read01.com/OAggzM0.html#.Yysqm3ZByw4)

- [https://zh.wikipedia.org/zh-tw/%E7%92%B0%E6%9E%B6%E9%8E%96%E5%AE%9A](https://zh.wikipedia.org/zh-tw/%E7%92%B0%E6%9E%B6%E9%8E%96%E5%AE%9A)