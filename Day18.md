# Day18 - Three.js與滑鼠互動操作(一)

> 這裡是「Three.js學習日誌」的第18篇，這一篇主要是接續上一篇的簡易OrbitControl實作，並加入一點進階的內容。這系列的文章假設讀者看得懂javascript，並且有Canvas 2D Context的相關知識。


##  來自製一個陽春版本的OrbitControl吧 - 進階篇 

![img](https://i.imgur.com/9YDZ4yk.gif)

在上一回我們透過定義**水平環形軌道**&**垂直軸**的攝影機移動方式來實作了簡易版本的`OrbitControls`。

但我猜應該多少有人注意到像這樣的作法，實際成品好像和`Three.js`所提供的`OrbitControls`不太一樣。

尤其是讓`Torus mesh`沿著`X`軸旋轉的時候，差異的狀況最為明顯。

|原版`OrbitControls`|自製`OrbitControls`|
|---|---|
|![img](https://i.imgur.com/jJhjObq.gif)|![img](https://i.imgur.com/5DeVdvm.gif)|

> 我們在上一回自製的版本除了水平旋轉的方向相反以外，垂直翻轉似乎轉不太動(?)

這個原因就是在於我們是用**卡式座標**(Cartesian)的概念去計算攝影機位置。

![img](https://i.imgur.com/FaZfIGY.jpg)

>我們之前的計算方式實際是讓攝影機在一個環形牆面上移動。

如果想要達成像是原版`OrbitControls`的特效的話，那我們得把計算座標的方式改成**極座標運算**才行。

![img](https://i.imgur.com/sszFOOu.jpg)

>不熟極座標運算或是已經還給高中老師的人可以看[這邊](https://www.youtube.com/watch?v=Ex_g2w4E5lQ)~

在`Three.js`中，官方其實是有提供極座標運算的方法的，也就是`Spherical`。

![img](https://i.imgur.com/g7kstzo.jpg)

`Spherical`本身並沒有提供可以轉化成`Vector3`的方法，但我們可以用`Vector3`底下的`setFromSpherical`來做這件事。

我們主要需要改寫的部分是`tick loop`的函數。

> 原本的`tick loop`長這樣

```javascript
const tick = () => {
    camera.position.x = Math.sin(cursor.x * Math.PI) * railRadius;
    camera.position.z = Math.cos(cursor.x * Math.PI) * railRadius;
    camera.position.y = cursor.y;
    camera.lookAt(new Vector3(0, 0, 0));
    renderer.render(scene, camera);
    requestAnimationFrame(tick);
  };
  tick();
```

而我們需要改寫的方向就是把`cursor`的X/Y數值改為映射到`φ`和`θ`上面。

因為我們希望極座標的起始位置會從90度開始，這樣才可以看到正面的`Torus`，所以`φ`要加上 `Math.PI / 2`做為Offset。

除此之外，我們還希望讓水平旋轉的方向與原版`OrbitControls`相同，所以`θ`必須要乘以-1。

最後建立起來的`Spherical`還要記得發動`makeSafe`方法。

`makeSafe`方法主要是用來把`φ`的變化限制在`0~PI`之間。因為大於等於`PI`的`φ`角可能會造成座標軸方向的突然變換。

```javascript
 const tick = () => {
    const sp = new Spherical(
      railRadius,
      // 因為我們希望極座標的起始位置會從90度開始，這樣才可以看到正面的Torus，所以這邊要加上 Math.PI / 2
      Math.PI * cursor.y + Math.PI / 2, 
      -Math.PI * cursor.x // 讓水平旋轉方向相反
    ).makeSafe();
    const location = new Vector3().setFromSpherical(sp);

    camera.position.x = location.x;
    camera.position.y = location.y;
    camera.position.z = location.z;
    camera.lookAt(new Vector3(0, 0, 0));
    renderer.render(scene, camera);
    requestAnimationFrame(tick);
  };
  tick();
```

![img](https://i.imgur.com/l1vEibB.gif)

> Perfect !

> codepen連結:[點我](https://codepen.io/mizok/pen/wvjXzYb)


## 最後的最後再來玩個有趣的東西 - 實做Damping

在`Three.js`原版所提供的`OrbitControls`是有`controls.enableDamping`這個**Option**的。

`Damping`指的是**遞減**的意思，它可以讓使用者在操作的時候`OrbitControls`的時候，會有一個**等減速度**的運動，這樣可以優化操作上的體感。

而我們這邊就是要來示範一下，要怎麼為我們自製的`OrbitControls`加上`Damping`。

> 老實說其實不難。

我們實做`OrbitControls`的原理是透過換算每一個`Frame`上`cursor`的數值，而現在的`cursor`數值則是會根據滑鼠當前的位置做更新，也就是說，如果要實現`Damping`，那就是要在`mousemove`傳遞數值給`cursor`的過程中做手腳。

這邊我們使用`GSAP`套件的`.to`方法來達成這件事。

```javascript
renderer.domElement.addEventListener("mousemove", (ev) => {
    const rect = renderer.domElement.getBoundingClientRect();
    gsap
      .to(cursor, {
        x: ((ev.clientX - rect.left) / rect.width - 0.5) * 2,
        y: -((ev.clientY - rect.top) / rect.height - 0.5) * 2,
        duration: 1, // 用Tween的方式刻意的讓傳遞數值的動作產生delay
        paused: true
      })
      .play();
  });
```

> ba-da-bean~ ba-da-boon~

![img](https://i.imgur.com/kc3kL8R.gif)

> 這邊光看圖片其實看不太出來，可以直接在codepen上試試~

> codepen 連結:[點我](https://codepen.io/mizok/pen/xxjzeqO)


## 小結


我們這次學習了要怎麼使用`Three.js`提供的**極座標運算**方法，並且成功實做了一個附帶`Damping`效果的簡易`OrbitControls`，如果有興趣的話其實可以接續著研究要怎麼實踐原版`OrbitControls`的機能XD。

明天我們預計要來講講與**滑鼠懸停/點擊**息息相關的`Raycasting`，敬請期待!

