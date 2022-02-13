+++
title = "如何改在Andriod Studio 編輯 sqlite database"
date = 2019-05-04T16:01:28+08:00
# description = "" # Used by description meta tag, summary will be used instead if not set or empty.
featured = false
comment = false
toc = true
reward =  false
pinned = false
categories = [
  "Android"
]
tags = [
  ""
]
series = [
  ""
]
images = []
+++

<!--more-->

1. 匯出: [參考](https://stackoverflow.com/questions/17529766/view-contents-of-database-file-in-android-studio)
Android Studio versions >= 3.0:
    Open Device File Explorer via View > Tool Windows > Device File Explorer
    Go to data > data > PACKAGE_NAME > database, where PACKAGE_NAME is the name of your package (it is com.Movie in the example above)
    Right click on the database and select Save As.... Save it anywhere you want on your PC.

2. 編輯:  [參考](https://www.youtube.com/watch?v=_6p7BqjNANQ)
妥




# SQLiteDatabase db用db.update 回傳的int 看有沒有更新成功，回傳0就新增 #
![](https://i.imgur.com/vDV7JM3.png)
![](https://i.imgur.com/MVKhthS.png)

```
//ref: https://www.codota.com/code/java/methods/android.database.sqlite.SQLiteDatabase/update
private int updateName(long id, String name) {
  ContentValues values = new ContentValues();
  values.put("name", name);
  return database.update("table_name", values, "id=" + id, null);
}
```



 ### 20190414 Youtube JS 渲染後才有 觀看次數與發表時間，嘗試很久都不行，決定放棄顯示這兩個 ###
![](https://i.imgur.com/tss2ZQ8.png)

 ### 20190414 發現Thread 不能執行畫面更新工作，來研究書本17-2-2AsyncTask ###
 
 # SQL #
 sqlite> SELECT * FROM COMPANY WHERE AGE >= 25 OR SALARY >= 65000;
 sqlite> DELETE FROM COMPANY WHERE ID = 7;
 
 ## 演算法 (DONE) ##
 是否有頻道紀錄
 有--->提出紀錄點放到temp-->讀:(第一個放到紀錄點)讀到上次頻道紀錄點
 無-->讀:(第一個放到紀錄點)讀N=3個
 
 記錄點作法[sharedpreferences](https://spicyboyd.blogspot.com/2018/04/appandroid-sharedpreferences.html)
 
 
 ## 用YouTube分享功能，增加Channel List(on going) ##
 ![](https://i.imgur.com/batX1w5.png)
![](https://i.imgur.com/O9VVdoW.png)


 ![](https://i.imgur.com/J1RNBaM.png)
[設定Activity能處理ACTION為Intent.ACTION_SEND的Intent](http://brunttech.blogspot.com/2013/02/blog-post.html)