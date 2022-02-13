+++
title = "Android App Note 20190228 與要學的 20190403"
date = 2019-03-07T23:25:57+08:00
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
  ""
]
series = [
  ""
]
images = []
+++


計畫
1. [x]寫Qr code
2. 寫recycle view
[x]存入設定，下次開機要有
[x]禁止轉向
[x]編輯新增事項
4. 試著用fire base?!
5. SHA-256
[MessageDigest](https://developer.android.com/reference/java/security/MessageDigest)
SHA-256

<!--more-->
```java=
public static String encrypt(String s){   
  MessageDigest sha = null;
  
  try{
   sha = MessageDigest.getInstance("SHA-1");   
   sha.update(s.getBytes());   
  }catch(Exception e){
   e.printStackTrace();
   return "";
  }

  return byte2hex(sha.digest());   
  
 }

↑如果要改 SHA-256 或 MD5 那就直接改紅色的字串，把它給成想要的編碼方法。

private static String byte2hex(byte[] b){
     String hs="";
     String stmp="";
     for (int n=0;n<b.length;n++){
      stmp=(java.lang.Integer.toHexString(b[n] & 0XFF));
      if (stmp.length()==1) hs=hs+"0"+stmp;
      else hs=hs+stmp;
     }
     return hs.toUpperCase();
    }

```

Jsoup
前面說的都是抓取夾在Tag間的文字,這邊教一下抓取Tag內src或是href屬性內的網址,使用的是attr()
```
<script type="text/javascript" src="http://whdn.williamhill.com/core/ob/static/cust/js/minified/main_racing.js?ver=723fa5ea119ca4e349d4e10120810d9a"></script>
```
	
Document doc =  Jsoup.parse(url, 3000);        //連結該網址
Elements title = doc.select("script[type=text/javascript]");
String te=title.attr("src");            //用來獲取src屬性的值


---

# 需要學的20190403


https://www.google.com/search?client=firefox-b-d&q=%E6%A8%A1%E6%93%AC%E7%99%BB%E5%85%A5+jsoup
https://blog.csdn.net/dietime1943/article/details/73294442
http://www.voidcn.com/article/p-wpmsbzzm-bmr.html

## 下滑 重整
android.support.v4.widget.SwipeRefreshLayout，讓開發者可以輕鬆實作以手指下滑(拉) 螢幕以更新資料的功能的 app。


## 分享給/接受來自 其他程式
http://blog.tonycube.com/2013/01/android-app.html?m=1

https://fecbob.pixnet.net/blog/post/38061071-android中「分享」功能的實現

https://juejin.im/entry/599ae92151882524302845fb

## web view 與 解析原始碼
html 的 tag 
### HtmlCleaner
YouTube
Parsing HTML using Jsoup on Android
YouTube
How to Pull text from any website using Jsoup in Android(Easy)

https://xxs4129.pixnet.net/blog/post/165417214-android使用jsoup抓取網頁資料

http://myted1217.blogspot.com/2015/09/androidjsoup.html?m=1

必看
https://blog.csdn.net/lisonglisonglisong/article/details/43091359

codebook
https://www.open-open.com/jsoup/dom-navigation.htm

https://www.open-open.com/jsoup/example-list-links.htm

新聞 jsoup
https://www.twblogs.net/a/5b8224502b717737e032a68f/

http://www.javakcsj.com/javaal/1721.html

http://www.demodashi.com/demo/10220.html

https://blog.csdn.net/yxmaomao1991/article/details/50550887


## 讀/寫行事曆
https://blog.tonycube.com/2012/10/androidcalendar-event.html?m=1

## 讀 /寫儲存空間
https://blog.tonycube.com/2012/03/android-internal-and-external-storage.html?m=1

## Google Analytics for Android
https://blog.tonycube.com/2013/04/google-analytics-for-android-1.html?m=1
Google Analytics is natively integrated with Firebase

# 20190403目標
目標:Youtube 可以追蹤多個頻道
功能:
1. 可以自己把頻道分類
2. 可以快速inport/export 已經存的頻道
3. 可以在程式內看影片(要會自製html與渲染)

400px" height="225px\
新增頻道->初始化: (最多)抓前三個當作
更新->抓所有項目到直到與資料庫重覆

學!!!!在APP內 自製html與渲染
<iframe width="560" height="315" src="https://www.youtube.com/embed/H4bKela_Dac" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

教材:
https://www.youtube.com/watch?v=b1dkJXjuTsg

