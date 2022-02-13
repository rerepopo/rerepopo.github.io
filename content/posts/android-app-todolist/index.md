+++
title = "Android App Todolist"
date = 2019-03-03T22:34:22+08:00
# description = "" # Used by description meta tag, summary will be used instead if not set or empty.
featured = false
comment = false
toc = true
reward = true
pinned = false
categories = [
  "Android"
]
tags = [
  "Todolist"
]
series = [
  ""
]
images = []
+++


tags: `List` `RecyclerView` `Firebase` `AlertDialog` `Snackbar`
![](https://i.imgur.com/Oyy2ziK.gif?width=370px)

:::success
功能:
 :o:可以新增
 :o:可以編輯
 :o:可以長按移動
 :o:可以滑動刪除
 :o:可以上傳雲端
 :o:可以下載至本地
:::danger
 :x:即時同步
:::
<!--more-->

:::info
應用項目:
- List
- RecyclerView
- CardView
- LinearLayoutManager
   >[網傳]這步驟忘記了也不會顯示錯誤，但就是不能動
- ItemTouchHelper
   >一個interface，用來控長按移動，左右滑動刪除
- Firebase
- AlertDialog and LayoutInflater
   >新增或編輯: 用彈出Dialog少去用另一個Activity :smile:
- Snackbar
   >回復刪除動作
:::

心得
---
> - RecyclerView很好玩，以後應該會常常用到，可惜的是程式碼架構並沒有很明白，目前的理解程度只堪應用樣板。
>- Firebase弄了一整天，最後只做簡化版
>  - 可以一鍵下載/一鍵上傳備份
>  - 還喬不定雲端與本地同步(任意一邊增加/刪除/修改都要反應到另一邊)
>  - 可能跟我是List class 有關，不然應該會更好處理
>  - BUG 發生在下列情況
>     1. 監聽雲端增加的時候-->本地也增加
>     2. 監聽本地增加的時候-->雲端也增加
>     3. 本地使用ADD button的時候 1. 和 2. 發生乒乓效應
>     
> [name=Backspace][time=Sun, Mar 3, 2019 10:18 PM] [color=#907bf7]





主要參考文獻
---
- [RecyclerView主程式: Android App程式開發剖析 第三版 CH12-4](https://github.com/macdidi5/Zahamena/tree/master/examples/ch12/Material02)
- [用Snackbar回復刪除動作](https://www.androidhive.info/2017/09/android-recyclerview-swipe-delete-undo-using-itemtouchhelper/)

