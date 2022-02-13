+++
title = "Android App QRcodec"
date = 2019-03-02T10:06:25+08:00
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

> 偽目標：
>  1. 製作QR code 
>  2. 解析QR code
>     (1). 來自相機
>     (2). 來自網路
>     

<!--more-->

![](https://i.imgur.com/e0r6uyb.png?width=300px)  ![](https://i.imgur.com/x7SqpAb.png?width=300px)




在Android中想要使用QRcode
應該都會套用[ZXing](https://github.com/zxing/zxing)套件
```
    compile 'com.google.zxing:core:3.2.1'
    compile 'com.journeyapps:zxing-android-embedded:3.2.0@aar'
```

製作QR code
---
製作比較容易
先從製作QR code開始說起

### android:onClick
這邊套用了`android:onClick="genCodeByUserText"`可以減少一些在`onCreate`的code
接著...
> public void genCodeByUserText(==View view==)

### Unicode
==\u27a5==是Unicode，用來表示![](https://s3.amazonaws.com/static.graphemica.com/glyphs/i500s/000/005/998/original/27A5-500x500.png?1275296322?width=30px)


```xml
<Button
            android:id="@+id/btnGen"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_marginLeft="8dp"
            android:layout_marginRight="8dp"
            android:layout_marginTop="8dp"
            android:onClick="genCodeByUserText"
            android:text="\u27a5產生 Code"/>
```

### 按下按鈕送出後，關閉鍵盤
這樣做可讓使用者操作變得流暢[[ref]](http://justimchung.blogspot.com/2017/10/android-qr-code.html)
```java=209
        InputMethodManager imm = (InputMethodManager) getSystemService(Context.INPUT_METHOD_SERVICE);
        if (imm != null) {
            imm.hideSoftInputFromWindow(getWindow().getDecorView().getWindowToken(), 0);
        }
    
```

### BarcodeEncoder 
透過`hints`可以設定自我修正程度與邊框大小
BarcodeEncoder會**return** Bitmap 
這時候再放到ImageView即可
```java=213
    public void updateImageAndContent(String text, BarcodeFormat barcodeFormat)
    {//ref: http://justimchung.blogspot.com/2017/10/android-qr-code.html
        ImageView ivCode = (ImageView) findViewById(R.id.ivCode);
        BarcodeEncoder encoder = new BarcodeEncoder();
        try {
            Map hints = new EnumMap(EncodeHintType.class);
            //hints.put(EncodeHintType.ERROR_CORRECTION, ErrorCorrectionLevel.H);
            hints.put(EncodeHintType.MARGIN, 2);
            Bitmap bit = encoder.encodeBitmap(text, barcodeFormat,
                    500, 500,hints);
            ivCode.setImageBitmap(bit);
            deCodeBitmapToContent(bit);// check answer
        } catch (WriterException e) {
            e.printStackTrace();
        }
    }
```

解析QR code (下載網路圖片)
---

### deCodeBitmapToContent
從網路照片解析的話
需要使用**deCodeBitmapToContent**這個方法
傳入==Bitmap==
decode方法是`result = reader.decode(binaryBitmap);`
文字結果會放在`String text = result.getText();`
barcodeFormat會放在`BarcodeFormat barcodeFormat = result.getBarcodeFormat();`
```java=142
private void deCodeBitmapToContent(Bitmap generatedQRCode)
    {//ref: https://stackoverflow.com/questions/32120988/qr-code-decoding-images-using-zxing-android
        if (generatedQRCode==null)
        {Toast.makeText(this, "At deCode, generatedQRCode==null", Toast.LENGTH_SHORT).show();
            return;
        }
        int width = generatedQRCode.getWidth();
        int height = generatedQRCode.getHeight();
        int[] pixels = new int[width * height];
        generatedQRCode.getPixels(pixels, 0, width, 0, 0, width, height);

        RGBLuminanceSource source = new RGBLuminanceSource(width, height, pixels);

        BinaryBitmap binaryBitmap = new BinaryBitmap(new HybridBinarizer(source));

        Reader reader = new MultiFormatReader();
        Result result = null;
        try {
            result = reader.decode(binaryBitmap);
        } catch (NotFoundException e) {
            e.printStackTrace();
        } catch (ChecksumException e) {
            e.printStackTrace();
        } catch (FormatException e) {
            e.printStackTrace();
        }
        String text = result.getText();
        BarcodeFormat barcodeFormat = result.getBarcodeFormat();
        TextView textViewQRCode = (TextView) findViewById(R.id.decode_text);
        textViewQRCode.setText(" barcodeFormat: " +barcodeFormat.toString()+"\n CONTENT: " + text);

    }
```

### DownloadImageTask
把**要更新的圖片位置**與**網址字串**傳到**DownloadImageTask**[[ref]]( http://finalfrank.pixnet.net/blog/post/43369915-%5Bandroid-%E9%96%8B%E7%99%BC%5D-%E5%B0%87%E5%A4%96%E9%83%A8%E7%B6%B2%E5%9D%80%E9%A1%AF%E7%A4%BA%E6%96%BC-imageview-%E4%B8%AD)
```java=88
                EditText url = (EditText)findViewById(R.id.internet_scan_edittext);
                String url_str = url.getText().toString();
                Log.d("dddd",url_str);
                new DownloadImageTask((ImageView) findViewById(R.id.ivCode)).execute(url_str);
```


這是一個AsyncTask，可以在背景下載


```java=175
private class DownloadImageTask extends AsyncTask<String, Void, Bitmap> {
        //ref: http://finalfrank.pixnet.net/blog/post/43369915-%5Bandroid-%E9%96%8B%E7%99%BC%5D-%E5%B0%87%E5%A4%96%E9%83%A8%E7%B6%B2%E5%9D%80%E9%A1%AF%E7%A4%BA%E6%96%BC-imageview-%E4%B8%AD
        ImageView bmImage;

        public DownloadImageTask(ImageView bmImage) {
            this.bmImage = bmImage;
        }

        protected Bitmap doInBackground(String... urls) {
            String urldisplay = urls[0];
            Bitmap mIcon11 = null;
            try {
                InputStream in = new java.net.URL(urldisplay).openStream();
                mIcon11 = BitmapFactory.decodeStream(in);
            } catch (Exception e) {
                Log.e("Error", e.getMessage());
                e.printStackTrace();
            }
            return mIcon11;
        }

        protected void onPostExecute(Bitmap result) {
            bmImage.setImageBitmap(result);
            deCodeBitmapToContent(result);// check answer
        }
    }
```

解析QR code (開啟相機)
---
用一個**按鈕**發動相機
加一個**onActivityResult**收來自相機的返回資訊
結果放在
`String text = result.getContents();`
`String barcodeFormat_str = result.getFormatName();`
```java=54
        scan_btn = (Button)findViewById(R.id.scan_btn);
        final Activity activity = this;
        scan_btn.setOnClickListener(new View.OnClickListener()
        {//ref: http://dangerlover9403.pixnet.net/blog/post/200742339-%5B%E6%95%99%E5%AD%B8%5D-qrcode-scanner(android-version)
         //ref: http://oldgrayduck.blogspot.com/2018/01/android-studio-zxing-qrcode.html
            //自製掃描圖示教學ref: https://www.jianshu.com/p/ef2ef751bf5b

            @Override
            public void onClick(View view)
            {
                IntentIntegrator integrator = new IntentIntegrator(activity);
                integrator.setPrompt("掃描中...");
                integrator.setCameraId(0);//0:後  1:前鏡頭
                integrator.setBeepEnabled(false);
                integrator.setBarcodeImageEnabled(false);//该方法用于设置“被扫描的二维码图片”可以保存在本地//result.getBarcodeImagePath()
                integrator.setOrientationLocked(false);//<!--如果要直立螢幕 要加這段/ -->调整手机方向时，扫描布局也会重新布置
                integrator.setDesiredBarcodeFormats(IntentIntegrator.ALL_CODE_TYPES);//QR_CODE_TYPES
                integrator.initiateScan();

                //按下按鈕後 取消鍵盤顯示 看起來順一點
                //ref: https://blog.csdn.net/ccpat/article/details/46717573
                InputMethodManager imm = (InputMethodManager) getSystemService(Context.INPUT_METHOD_SERVICE);
                if (imm != null) {
                    imm.hideSoftInputFromWindow(getWindow().getDecorView().getWindowToken(), 0);
                }

            }
        });
```

```java=104
    @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data)
    {
        IntentResult result = IntentIntegrator.parseActivityResult(requestCode,resultCode,data);
        if (result!= null)
        {
            if (result.getContents()==null)
            {
                Toast.makeText(this, "You cancelled the scanning", Toast.LENGTH_SHORT).show();
            }
            else
            {
                Toast.makeText(this,"CONTENT已經更新到 Text view",Toast.LENGTH_SHORT).show();
                String text = result.getContents();
                String barcodeFormat_str = result.getFormatName();
                TextView textViewQRCode = (TextView) findViewById(R.id.decode_text);
                textViewQRCode.setText(" barcodeFormat: " +barcodeFormat_str+"\n CONTENT: " + text);
/*不能開來用....
                BarcodeFormat barcodeFormat;
                try {
                    barcodeFormat = BarcodeFormat.valueOf(barcodeFormat_str);
                } catch (Exception e) {
                    Log.e("Error", e.getMessage()+": this enum type has no constant with the specified name");
                    Toast.makeText(this,"this enum type has no constant with the specified name",Toast.LENGTH_SHORT).show();
                    e.printStackTrace();
                    barcodeFormat =BarcodeFormat.QR_CODE;
                }
*/
                EditText etContent = (EditText) findViewById(R.id.etContent);
                etContent.setText(text);
            }
        }
        else
        {
            super.onActivityResult(requestCode, resultCode, data);
        }
    }
```

### 為了讓相機保持直立
在AndroidManifest.xml加上
```xml=29
<activity
            android:name="com.journeyapps.barcodescanner.CaptureActivity"
            android:screenOrientation="portrait"
            tools:replace="screenOrientation" />
```

在java加上
```java=69
                integrator.setOrientationLocked(false);//<!--如果要直立螢幕 要加這段/ -->调整手机方向时，扫描布局也会重新布置
```

![](https://i.imgur.com/aTvLKTe.jpg?width=200px)

強制直立螢幕顯示
---
```xml=
<activity
android:screenOrientation="portrait"/>
```

未來可增加
---
[鍵盤改造 How to Change the Input Method Action](https://www.youtube.com/watch?v=DivBp_9ZeK0)
![](https://i.imgur.com/wY3hb5j.png?width=300px)
