---
title: Android总结1
date: 2020-05-15 20:36:25
tags: Android
---



Android开发App踩坑记录

<!--more-->

[toc]

## 音频

### 录音

```java
// 录音部分，以tag的01标记是否正在录音
if (audio.getTag().toString().equals("0")) {
    // 开始录音
    audio.setImageResource(R.mipmap.stop);
    audio.setTag("1");
    time = (String) DateFormat.format("yyyyMMdd_HHmmss", Calendar.getInstance(Locale.CHINA));

    String dir_path = Note.this.getFilesDir().getPath() + "/recordings/";
    Log.d(TAG, "onClick: dir: " + dir_path);
    File dir = new File(dir_path);
    if(!dir.exists()){
        dir.mkdirs();
    }

    String file_name = dir_path + "/" + time + ".aac";
    recordAudioFile = new File(file_name);
    try {
        if (!recordAudioFile.createNewFile()) {
            Log.d(TAG, "onClick: file create failed.");
        } else {
            Log.d(TAG, "onClick: file create done.");
        }
    } catch (IOException e) {
        e.printStackTrace();
    }

    // recordAudioFile = File.createTempFile(file_name,".aac");
    // recordAudioFile = new File(file_name, ".aac");
    // 录音设置
    mediaRecorder = new MediaRecorder();
    mediaRecorder.setAudioSource(MediaRecorder.AudioSource.MIC);
    mediaRecorder.setOutputFormat(MediaRecorder.OutputFormat.DEFAULT);
    mediaRecorder.setAudioEncoder(MediaRecorder.AudioEncoder.AAC_ELD);
    mediaRecorder.setOutputFile(recordAudioFile.getAbsolutePath());
    // /data/user/0/com.example.curriculum/cache/
    try {
        mediaRecorder.prepare();
    } catch (IOException e) {
        Log.d(TAG, "onClick: prepare failed.");
    }
    mediaRecorder.start();

} else {
    // 停止录音
    audio.setImageResource(R.mipmap.audio);
    audio.setTag("0");

    if (mediaRecorder != null){
        Log.d(TAG, "onClick: record stop: " + recordAudioFile.getPath());
        mediaRecorder.stop();
        mediaRecorder.release();
        mediaRecorder = null;
        if (getCurrentFocus() != null) {
            // 当前项下插入一个singlerecord
            SingleNoteLayout child = (SingleNoteLayout) getCurrentFocus().getParent().getParent();
            create_record(linearLayout.indexOfChild(child), recordAudioFile.getAbsolutePath());
        } else {
            // 若无焦点，添加至末尾
            create_record(linearLayout.getChildCount()-1, recordAudioFile.getAbsolutePath());
        }

    }
}
```

### 播音：资源文件

```java
// 播放器需设置为类成员，以保证不会回收，否则播放可能会中断
private MediaPlayer mediaPlayer;

// MediaPlayer.create方法会自动进行设置+prepare
mediaPlayer = MediaPlayer.create(this, resid);
mediaPlayer.start();	
```

### 播音：本地文件

```java
private MediaPlayer mediaPlayer;

// new MediaPlayer()的方法不会自动设置及prepare
mediaPlayer = new MediaPlayer();
mediaPlayer.setDataSource(path);
mediaPlayer.prepare();	
mediaPlayer.setOnCompletionListener(new MediaPlayer.OnCompletionListener() {
    @Override
    public void onCompletion(MediaPlayer mp) {
        mp.stop();
        mp.release();
        mp = null;
        Log.d(TAG, "play: complete");
    }
});
mediaPlayer.start();
} catch (Exception e) {
    e.printStackTrace();
    Log.d(TAG, "play: file_path is wrong.");
}
// 最后勿加stop release等
```

<br/>

## LinearLayout的动态增删控件

增：addView时需要index参数，getChildCount、getChildAt、indexOfChild是很常用的方法。

删：removeView、removeViewAt、removeViewAll

<br/>

## 控件嵌套的parent

控件嵌套，同时只能点击到某个子控件，要通过子控件获取父级控件甚至更高级控件，可以使用onClick的参数view，一路getParent获取，同时使用getClass判断类型。多层嵌套时可能要使用多次getParent。

<br/>

## 权限框架

easypermissions

www.jianshu.com/p/41b093d213fb

<br/>

## ImageView 上下空白区域

图片像素大于屏幕像素时，显示有问题。尝试以下设置：android:adjustViewBounds="true" 。

Glide注入图片，与androidx/android10可能兼容性较差，出现了奇怪错误。

<br/>

## createNewFile()

要在手动建立的文件夹下，才可create

```java
// 建立文件夹
String dir_path = Note.this.getFilesDir().getPath() + "/new_dir/";
Log.d(TAG, "onClick: dir: " + dir_path);
File dir = new File(dir_path);
if(!dir.exists()){
    dir.mkdirs();
}
// 在该文件夹下建文件
String filename = dir_path + "/hi.txt";
File des = new File(filename);
try {
    if (!des.createNewFile()) {
        Log.d(TAG, "onClick: file already exists");
    } else {
        Log.d(TAG, "onClick: test file create done.");
    }
} catch (IOException e) {
    e.printStackTrace();
}
```

<br/>

## 数据库升级

```java
dbHelper = new MyDBHelper(this, "Course.db", null, 2);
db = dbHelper.getWritableDatabase();
```

首行参数指定新的version_code，DBHelper.class的onUpgrade中，针对不同版本的升级做出判断与操作。

<br/>

## Uri.fromFile(new File(path))

报错：java.io.FileNotFoundException: open failed: EACCES (Permission denied)

解决：`android:requestLegacyExternalStorage="true"` application

<br/>

## imageView.setImageURI(uri)

从库中读取string，setImageURI(Uri.pairse(string))会报错：

java.lang.SecurityException: Permission Denial: opening provider ... that is not exported from UID 10096

无合适解决方法，改用setImageURI(Uri.fromFile(new File(path)))。

但是该方法获得的Uri，调用图库查看图片时又会引起FileUriExposedException异常 :sweat:

<br/>

## intent向前传递的顺序问题

子活动返回result，即setResult()是在被finish()之前。

子活动setResult()之后，主活动的onActivityResult()就会启动。这就会导致**onActivityResult()先于子活动的onDestroy()**。

同时，onPause()， onStop()， onDestroy()中调用setResult()也是不安全的，这些步骤可能会在finish()之后。

<br/>

## 录音设置

编码器、输出文件、文件后缀之间的协调：最终采用文件指定后缀、输出文件Default、编码器与文件后缀匹配



![](Android总结1/zelda.png)