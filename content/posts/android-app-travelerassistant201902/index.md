+++
title = "TravelerAssistant201902"
date = 2019-02-27T23:04:50+08:00
# description = "" # Used by description meta tag, summary will be used instead if not set or empty.
featured = false
comment = false
toc = true
reward = false
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


* 偽目標: 想要寫一個旅遊可能可以用到的東西
* 想學的功能
   - [x] BottomNavigationView
   - [x] Fragment
   - [x] GPS/測速 功能
   - [x] 計步器
   - [x] 建立通知
<!--more-->


BottomNavigationView
---
![](https://i.imgur.com/JGhMLto.png)
他其實一般的menu
但透過其他顯示方式
並且附上圖示(可以由內建圖庫製作)
==需要搭配Fragment==
Fragment弄起來頗煩
他只是Activity的附屬
只提供view
很多service 需要由Activity拿到

採用new 的方式
之後==再用.==

```java=
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_traveler_assistant201902);

        gpsFragment = new GPSFragment();
        pedometerFragment = new PedometerFragment();
        notificationFragment =new NotificationFragment();
        aboutFragment =new AboutFragment();

        BottomNavigationView navigation = (BottomNavigationView) findViewById(R.id.bottom_navigation);
        navigation.setOnNavigationItemSelectedListener(mOnNavigationItemSelectedListener);
        navigation.setSelectedItemId(R.id.navigation_pedometer);//set default
        setFagment(pedometerFragment);//set default
    }
```
其中`//set default`
是用來讓第一次落在非第一位




GPS/測速 功能
---
![](https://i.imgur.com/svOZCyC.png?width=200px) 

上圖是模擬器的結果
很多感測器都是==虛構的==
真實的手機不見得都有
:::info
有趣的是，可以去調模擬器的GPS
還可以模擬通話
改電量......
![](https://i.imgur.com/40hcLLr.png?width=500px)

:::

```java=
 @Override
    public void onLocationChanged(Location location) {
        // 讀取目前位置的經、緯度
        double latitude = location.getLatitude();
        double longitude = location.getLongitude();
        double now_time = location.getTime();
        float[] results = new float[1];
        Location.distanceBetween(last_latitude, last_longitude, latitude, longitude, results);
        double speedin_m_s = (results[0]/((now_time-last_time)/1000));
        last_time = now_time;
        last_latitude =latitude;
        last_longitude =longitude;
        // 設定經、緯度畫面元件
        gpsFragment.longitude_text.setText(String.format("%.6f", longitude));
        gpsFragment.latitude_text.setText(String.format("%.6f", latitude));
        gpsFragment.accuracy_text.setText(String.format("%.6f", location.getAccuracy()));
        gpsFragment.altitude_text.setText(String.format("%.6f", location.getAltitude()));
         gpsFragment.time_text.setText(String.format("%d", location.getTime()));// 	Return the UTC time of this fix, in milliseconds since January 1, 1970.
        gpsFragment.speed_text.setText(String.format("%.6f", location.getSpeed()));
        gpsFragment.bearing_text.setText(String.format("%.6f", location.getBearing()));
        gpsFragment.distance_text.setText(String.format("%.6f 公尺", results[0]));
        gpsFragment.speed_text1.setText(String.format("%f m/s ",speedin_m_s));
        gpsFragment.speed_text2.setText(String.format("%f km/h", (speedin_m_s*3.6)));

    }
```
為了避免耗電
切到其他Fragment時最好關掉GPS需求
```Java
// 如果已經連線到 Google Play services 用戶端服務
                    if (googleApiClient.isConnected()) {
                        // 中斷 Google Play services 用戶端連線
                        googleApiClient.disconnect();
                    }
```
計步器
---
![](https://i.imgur.com/e1LN61c.png?width=200px)

### TYPE_STEP_COUNTER
原本看網路上說，要透過三個軸向的加速度峰值，算出腳步
實測結果根本亂算，發狂的跳
後來發現用內建==TYPE_STEP_COUNTER==的比較準確(目測準度99%)
```java=
 @Override
    public void onSensorChanged(SensorEvent event) {
        double range = 1;   //设定一个精度范围
        float[] value = event.values;
        if (event.sensor.getType()==Sensor.TYPE_ACCELEROMETER)
        {
            curValue = magnitude(value[0], value[1], value[2]);   
            if (pedometerFragment.isVisible())
            {
                pedometerFragment.value0_text.setText(String.format("%f (m/s^2) ",value[0]));    
                pedometerFragment.value1_text.setText(String.format("%f (m/s^2) ",value[1]));    
                pedometerFragment.value2_text.setText(String.format("%f (m/s^2) ",value[2]));    
            }
            

        }
        else if (event.sensor.getType()==Sensor.TYPE_STEP_COUNTER)
        {
            if (pedometerFragment.isVisible())
            {
                pedometerFragment.step_text2.setText(value[0] +"");
            }
            Log.d("event.sensor.getType()=", "Sensor.TYPE_STEP_COUNTER" +event.sensor.getType());
        }
        else if (event.sensor.getType()==Sensor.TYPE_LIGHT)
        {   if (pedometerFragment.isVisible())
            { pedometerFragment.light_text.setText(value[0]+" lux");}
        }
        else if (event.sensor.getType()==Sensor.TYPE_PRESSURE)
        {   if (pedometerFragment.isVisible())
            { pedometerFragment.pressure_text.setText(value[0]+" hPa");}
        }
        else if (event.sensor.getType()==Sensor.TYPE_PROXIMITY)
        {   if (pedometerFragment.isVisible())
            { pedometerFragment.proximity_text.setText(value[0]+" cm");}
        }
        else if (event.sensor.getType()==Sensor.TYPE_RELATIVE_HUMIDITY)
        {   if (pedometerFragment.isVisible())
            { pedometerFragment.humidity_text.setText(value[0]+" %");}
        }
        else if (event.sensor.getType()==Sensor.TYPE_AMBIENT_TEMPERATURE)
        {   if (pedometerFragment.isVisible())
            { pedometerFragment.temperature_text.setText(value[0]+" degree Celsius");}
        }
        else
        {
            new AlertDialog.Builder(TravelerAssistant201902.this)
                    .setTitle("感應器未設定")
                    .setMessage("這是甚麼?")
                    .show();
        }
    }
```


建立通知
---
![](https://i.imgur.com/JYZjgDA.png)
如圖，為了弄出這個效果(head up )
明明已經加了`.setPriority(Notification.PRIORITY_HIGH)`
還是沒有反應
查了好久，最後發現是==要刪除再裝才會有==
### NOTIFICATION_SERVICE
```java=1
set_notification_button.setOnClickListener(new Button.OnClickListener(){
            @Override
            public void onClick(View v) {
                //the getSystemService() method that provides access to system services comes from Context . An Activity extends Context , a Fragment does not. Hence, you first need to get a reference to the Activity in which the Fragment is contained and then magically retrieve the system service you want.
                NotificationManager mNotificationManager =
                        (NotificationManager) getActivity().getSystemService(Context.NOTIFICATION_SERVICE);

                // 建立與設定Notify channel
                // 加入裝置版本的判斷，應用程式就不用把最低版本設定為API level 26
                if (Build.VERSION.SDK_INT < Build.VERSION_CODES.O) {
                }
                else
                {
                    // 建立channel物件，參數依序為channel代碼、名稱與等級
                    NotificationChannel nChannel = new NotificationChannel(
                            channelIdBasic, "Basiccc", NotificationManager.IMPORTANCE_HIGH);
                    // 設定channel物件
                    mNotificationManager.createNotificationChannel(nChannel);
                }

                // 建立NotificationCompat.Builder物件
                NotificationCompat.Builder builder = new NotificationCompat.Builder(getActivity(), channelIdBasic);

                // 設定圖示、時間、標題、訊息和額外資訊
                builder.setContentTitle("Basic Notification")
                        .setWhen(Calendar.getInstance().getTimeInMillis()+60000 )
                        //.setWhen(System.currentTimeMillis())
                        //.setContentInfo("3")
                        //.setFullScreenIntent(pi, true)
                        //builder.setColor(Color.BLUE)
                        .setDefaults(Notification.DEFAULT_ALL) // must requires VIBRATE permission
                        .setPriority(Notification.PRIORITY_HIGH) //must give priority to High, Max which will considered as heads-up notification
                        .setSmallIcon(R.drawable.ic_smile)
                        .setContentIntent(PendingIntent.getActivity(getActivity(), 0, new Intent(), 0))  // 設置Intent
                        .setAutoCancel(true)    // 可以注意的是若要用setAutoCancel()的話，就算只是要點擊以後就讓Notification消失而不開啟任何Activity，還是要在setContentIntent()裡放一個PendingIntent喔，單單設置setAutoCancel()是沒有效用的，此時可將PendingIntent的第三個參數改成new Intent()，即不開啟任何Activiy
                        .setContentText(adding_id+": "+info_edittext.getText().toString());

                // 建立通知物件
                Notification notification = builder.build();

                // 使用BASIC_ID為編號發出通知
                //如果ID一樣，可能導致head up notification 無法出現，必要時可以刪除app(或強制關閉)
                mNotificationManager.notify(adding_id++, notification); // 可用 Calendar.getInstance().getTimeInMillis()
            }
        });
```

line:43 這邊的id是用來送編號，標號若一樣將會覆蓋，所以我用++

### ALARM_SERVICE
設定時間按鈕點下去後會出現
![](https://i.imgur.com/EFhRkZB.png)
選完可以在預期時間發送
![](https://i.imgur.com/GzbCOSm.png)

原理是透過AlertReceiver.class中的==BroadcastReceiver==
```java=9
public class AlertReceiver extends BroadcastReceiver {
    @Override
    public void onReceive(Context context, Intent intent) {
        if(intent.getAction().equals("Hello"))
        {
            NotificationHelper notificationHelper = new NotificationHelper(context);
            NotificationCompat.Builder nb = notificationHelper.getChannelNotification(intent.getStringExtra("information"));
            notificationHelper.getManager().notify(1, nb.build());
        }
    }
}
```
> [name=小記]需要到AndroidManifest.xml增加<receiver android:name=".AlertReceiver" />

```xml=
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.user.travelerassistant201902">
    <!-- 讀取高精確度位置資訊授權 -->
    <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
    <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION"/>
    <uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED" />
    <uses-permission android:name="android.permission.VIBRATE" />
    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/AppTheme">
        <activity
            android:name=".TravelerAssistant201902"
            android:label="@string/app_name">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
        <service android:name=".JobSchedulerService"
            android:permission="android.permission.BIND_JOB_SERVICE"/>
        <receiver android:name=".AlertReceiver" />
    </application>
</manifest>
```

以及NotificationHelper.class
```java=11
public class NotificationHelper extends ContextWrapper {
    public static final String channelID = "channelID";
    public static final String channelName = "Channel Name";

    private NotificationManager mManager;

    public NotificationHelper(Context base) {
        super(base);
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
            createChannel();
        }
    }

    @TargetApi(Build.VERSION_CODES.O)
    private void createChannel() {
        NotificationChannel channel = new NotificationChannel(channelID, channelName, NotificationManager.IMPORTANCE_HIGH);

        getManager().createNotificationChannel(channel);
    }

    public NotificationManager getManager() {
        if (mManager == null) {
            mManager = (NotificationManager) getSystemService(Context.NOTIFICATION_SERVICE);
        }

        return mManager;
    }

    public NotificationCompat.Builder getChannelNotification(String contentText) {
        return new NotificationCompat.Builder(getApplicationContext(), channelID)
                .setContentTitle("Alarm!")
                .setContentText(contentText)
                .setSmallIcon(R.drawable.ic_smile);
    }
}
```


參考
---
網路與Android App程式開發剖析 第三版（適用Android 8 Oreo與Android Studio 3）

後記
---
:::success
本文寫於程式開發後兩周
令我最在意的是==通知排程==
不知道排下去會不會因為外力消失 :tada:
:::



### Abc
```abc
X:1
T:Speed the Plough
M:9988/1000
C:Trad.
K:G
|:GABc dedB|dedB ddddddddd|eadB|c2ec B2dB|c2A2 A2BA|
GABc dedB|dedB dedB|c2ec B2dB|A2F2 G4:|
|:g2gf gdBd|g2f2 e2d2|c2ec B2dB|c2A2 A2df|
g2gf g2Bd|g2f2 e2d2|c2ec B2dB|A2F2 G4:|
```