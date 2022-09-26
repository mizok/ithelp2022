# Day10 - 「多多益善」 - 幾何結構Geometry(四)  

> 這裡是「Three.js學習日誌」的第10篇，本篇的主旨是要介紹Geometry的概念，這是Geometry章節的最後一篇，主要會再補充能一些夠用來客製化Geometry的小技巧。這系列的文章假設讀者看得懂javascript，並且有Canvas 2D Context的相關知識。


在上一篇我們介紹了要怎麼從0去建立一個正立方體，並提及了要如何以Winding rule的規則去排序Triangle soup。

> 上回忘了說，Triangle soup 就是指一群三角面座標所形成的陣列。

今天我們要來講些客製化Geometry還可以用的兩個小技巧。

## 布林運算 - Union/Difference/Intersection

有用過**Adobe illustrator**的人應該都很習慣使用**路徑管理員**這個玩意。

**路徑管理員**就是一個可以用來做圖形路徑的布林運算，也就是**差集**（Difference），**聯集**（Union），**交集**（Intersection）...,etc.等操作。

而在`Three.js`中，官方其實<u>沒有</u>提供布林運算的方法。

> 是的，沒有。 當初知道這個我也有點意外

所以這部分我們必須要使用外部套件`ThreeCSG`。

`CSG`的意思是『**構造實體幾何**(Constructive solid geometry)』



