+++
title = "我的第一篇用Mark down"
date = 2017-02-27T21:00:00+08:00
description = "description" # Used by description meta tag, summary will be used instead if not set or empty.
featured = false
comment = false
toc = true
reward = true
pinned = false
categories = [
  "Mark Down"
]
tags = [
  ""
]
series = [
  ""
]
images = []
+++



我覺得自己練程式沒有寫些筆記
最後可能都會忘光

為了給未來的自己
==一些索引==
<!--more-->
:::info
**我決定用hack md**
:::



---


他可以支援
![imgur](https://i.imgur.com/9cgQVqD.png?width=50px)
圖庫
也可以放程式碼
# xml #
```xml=87
 <TableRow>
            <Button
                android:id="@+id/job_button"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="Job" />
        </TableRow>
```
# java #
```java=998
    @Override
    public boolean onStartJob(JobParameters params) {
        Log.d(TAG, "onStartJob: ");
        //返回true，表示該工作耗時，需要執行結束時手動調用jobFinished()，來銷毀任務
        //返回false，代表任務處理不需要很長的時間，在resurn時已經處理結束
        return false;
    }
```

再比較其他附加功能後
這是目前最好的編輯環境了!:smile: