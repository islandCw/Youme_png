# 播放SDK使用指南 for Android

## 1 概述
YouMePlayer 是一个适用于 Android 平台的音视频播放器 SDK，可高度定制化和二次开发，为 Android 开发者提供了简单、快捷的接口，帮助开发者在 Android 平台上快速开发播放器应用。

### 1.1 功能特性
| 功能 | 描述 |
|---|---|
| YMVideoView | 类似 Android VideoView，基于 SurfaceView 的播放控件 | 
| 全架构支持 | 包括 arm64-v8a, armeabi-v7a, armeabi 与 x86 | 
| 后台播放 | URL 格式：protocol://ip/path?domain=xxxx.com | 
| IP 地址播放 | 更好的听觉体验 |
| 设置播放封面 | 在播放开始前显示封面 view |
| 软硬解自动切换 | 优先硬解，硬解失败自动切换到软解 |
| 自动直播延迟优化 | 播放直播流时可以通过自动变速播放来优化延迟 |
| H.265 软解 | 软解播放 H.265 视频流 |
| 变速播放 | 支持设置播放速度 |
| MP4 离线缓存 | 支持播放过程中缓存 MP4 文件到本地 |
| 解码数据回调 | 回调解码后的音视频数据，可以外部渲染 |
| 自定义 DNS 服务器 | 支持自定义 DNS 服务器与设置预解析域名 |
| 视频截图 | 支持视频截图 |
| 区域播放 | 支持播放视频画面的部分区域 |
| 音量增强 | 支持将播放音量增强到大于原始音量 |
| 快开模式 | 极大加快相同协议与格式的视频流的打开速度 |


## 2 开发准备
### 2.1 设备以及系统要求
系统要求：Android 4.1 (API 16) 及其以上

## 3. 快速接入
### 3.1 开发环境

* Android Studio 开发工具，官方[下载地址](http://developer.android.com/intl/zh-cn/sdk/index.html)
* Android 官方开发 SDK，官方[下载地址](https://developer.android.com/intl/zh-cn/sdk/index.html#Other)。

### 3.2 导入SDK
播放SDK 主要包含 demo 代码、sdk jar 包，以及 sdk 依赖的动态库文件。

其中，Sdk目录下需要拷贝到您的 Android 工程的所有文件，列表如下：

![](part.png)

- 将 YouMePlayer.jar 包拷贝到您的工程的 libs 目录下
- YouMePlayer SDK 支持 armv5、armv7、arm64 和 x86 多种 CPU 架构，目前市场上主流机型的 CPU 都采用的是 armv7 架构。您可以根据兼容性的需要，将 release 目录下的动态库，拷贝到您的工程对应的目录下，例如：armeabi-v7a 目录下的 so 则拷贝到工程的 jniLibs/armeabi-v7a 目录下。

由于 Android 7.0 使用 BoringSSL 替换了 OpenSSL，一些依赖系统内建 OpenSSL 的程序在一些 7.0+ 的 ROM 里可能会崩溃。如果您的应用 targetSdkVersion >= 24，那么强烈推荐将 libopenssl.so 加入至 jniLibs 目录。

具体可以参考 SDK 包含的 demo 工程,集成后的工程示例如下:
![](all.png)

### 3.3 修改 build.gradle
双击打开您的工程目录下的 `build.gradle`，确保已经添加了如下依赖，如下所示：

```
dependencies {
    compile files('libs/YouMePlayer.jar')
}
```

### 3.4 添加相关权限
在 app/src/main 目录中的 AndroidManifest.xml 中与<application>标签同级，增加如下 `uses-permission` 声明

```xml
<!-- 访问网络状态 -->
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
<uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
<uses-permission android:name="android.permission.CHANGE_WIFI_STATE" />
        
<!-- 系统相关 -->
<uses-permission android:name="android.permission.WAKE_LOCK" />
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
```

## 4. 开发步骤
### 4.1 使用 YMVideoView 实现媒体播放功能
YouMePlayer SDK 提供的 YMVideoView 类可以快速实现带界面的播放器功能，它们的接口与 Android 官方的 VideoView 类基本保持一致，其内部封装了 YMMediaPlayer 类所提供的播放功能。

YMVideoView 类使用了 SurfaceView 来完成视频画面的渲染。

#### 4.2 布局
在 XML 文件中，配置播放的视图布局，如下所示：

```xml
<com.youme.ymstreaming.YMVideoView
        android:id="@+id/VideoView"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:layout_gravity="center" />
```

#### 4.3 初始化
在 activity 文件中，对 YMVideoView 初始化，方法如下所示：

```java
YMVideoView mVideoView = (YMVideoView) findViewById(R.id.VideoView);
```

如果是多人连麦，可以添加多个窗口。

#### 4.4 设置加载动画
YMVideoView 提供了设置加载动画的接口，在播放器进入缓冲状态时，自动显示加载界面，缓冲结束后，自动隐藏加载界面，设置方法如下：

```java
View mLoadingView = findViewById(R.id.LoadingView);
mVideoView.setBufferingIndicator(mLoadingView);
```

此 mLoadingView 可以是任意的 Android View 视图对象。

#### 4.5 设置接收视频帧数据回调

- 示例：

```java
YMAVOptions options = new YMAVOptions();
//配置接收视频帧数据回调，1-接收，0-不接收
options.setInteger(YMAVOptions.KEY_VIDEO_DATA_CALLBACK,1);
mVideoView.setAVOptions(options);
```

#### 4.6 设置播放状态监听
YMVideoView 的 YMVideoListener 接口提供了丰富的播放状态消息回调。

- 设置播放状态回调方法：YMVideoView.setYMVideoListener

``` java
   /**
     * 设置播放状态回调
     * @param pYouMeListen 播放状态回调接口
     */
    public void setYMVideoListener(YMVideoListener pYouMeListen) {
        ...
    }
```

- YMVideoListener接口定义：

```java
public interface YMVideoListener {
    /**
     * prepare回调
     *     
     * @param errorcode 错误码
     */
    public void onPrepared(int i);
    
    /**
     * 播放错误回调
     *     
     * @param errorcode 错误码
     */
    public boolean onError(int errorcode);

    /**
     * 监听播放结束的回调
     *     
     * @param errorcode 错误码
     */
    public void onCompletion();

    /**
     * 播放器的状态消息回调
     *     
     * @param what 错误码
     * @param extra 错误码
     */
    public boolean onInfo(int what, int extra);
    
    /**
     * 视频帧数据回调
     *     
     * @param data 视频帧数据
     * @param size 视频帧数据大小
     * @param width 视频帧的宽
     * @param height 视频帧的高
     * @param format 视频帧的格式，0-YUV420P,1-JPEG,2-SEI
     * @param ts 时间戳，单位：毫秒     
     */
    public void onVideoFrameAvailable(byte[] data, int size, int width, int height, int format, long ts);
}
```

- 示例：

```java
//需先实现YMVideoListener,再设置监听
mVideoView.setYMVideoListener(this);
...

@Override
    public boolean onError(int errorcode) {
        Log.e(TAG, "Error happened, errorCode = " + errorcode);
        switch (errorcode) {
            case YMVideoView.ERROR_CODE_IO_ERROR:
                /**
                 * SDK will do reconnecting automatically
                 */
                Log.e(TAG, "IO Error!");
                return false;
            case YMVideoView.ERROR_CODE_OPEN_FAILED:
                Log.e(TAG, "failed to open player !");
                break;
            case YMVideoView.ERROR_CODE_SEEK_FAILED:
                Log.e(TAG, "failed to seek !");
                break;
            default:
                Log.e(TAG, "unknown error !");
                break;
        }
        return true;
    }

    @Override
    public void onCompletion() {
        Log.i(TAG, "Play Completed !");
    }

    @Override
    public boolean onInfo(int what, int extra) {
        Log.i(TAG, "OnInfo, what = " + what + ", extra = " + extra);
        switch (what) {
            case YMVideoView.MEDIA_INFO_BUFFERING_START:
                break;
            case YMVideoView.MEDIA_INFO_BUFFERING_END:
                break;
            case YMVideoView.MEDIA_INFO_VIDEO_RENDERING_START:
                Log.e(TAG, "first video render time: " + extra + "ms");
                break;
            case YMVideoView.MEDIA_INFO_AUDIO_RENDERING_START:
                break;
            case YMVideoView.MEDIA_INFO_VIDEO_FRAME_RENDERING:
                Log.i(TAG, "video frame rendering, ts = " + extra);
                break;
            case YMVideoView.MEDIA_INFO_AUDIO_FRAME_RENDERING:
                Log.i(TAG, "audio frame rendering, ts = " + extra);
                break;
            case YMVideoView.MEDIA_INFO_VIDEO_GOP_TIME:
                Log.i(TAG, "Gop Time: " + extra);
                break;
            case YMVideoView.MEDIA_INFO_SWITCHING_SW_DECODE:
                Log.i(TAG, "Hardware decoding failure, switching software decoding!");
                break;
            case YMVideoView.MEDIA_INFO_METADATA:
                Log.i(TAG, mVideoView.getMetadata().toString());
                break;
            case YMVideoView.MEDIA_INFO_VIDEO_BITRATE:
            case YMVideoView.MEDIA_INFO_VIDEO_FPS:
                break;
            case YMVideoView.MEDIA_INFO_CONNECTED:
                Log.i(TAG, "Connected !");
                break;
            default:
                break;
        }
        return true;
    }
    
    @Override
    public void onVideoFrameAvailable(byte[] data, int size, int width, int height, int format, long ts) {
       if (format == 2)
        Log.e(TAG,"onVideoFrameAvailable，ts:" + ts);
    }
```

#### 4.7 设置画面预览模式
YMVideoView 提供了各种画面预览模式，包括：原始尺寸、适应屏幕、全屏铺满、16:9、4:3 等，设置方法如下：

```java
mVideoView.setDisplayAspectRatio(YMVideoView.ASPECT_RATIO_ORIGIN);
mVideoView.setDisplayAspectRatio(YMVideoView.ASPECT_RATIO_FIT_PARENT);
mVideoView.setDisplayAspectRatio(YMVideoView.ASPECT_RATIO_PAVED_PARENT);
mVideoView.setDisplayAspectRatio(YMVideoView.ASPECT_RATIO_16_9);
mVideoView.setDisplayAspectRatio(YMVideoView.ASPECT_RATIO_4_3);
```

#### 4.8 设置播放地址
这是最重要的环节，必须在调用播放控制接口前设置播放地址，以 activity 文件为例，在 activity 中的onCreate方法里先设置好播放地址。

传入播放地址，可以是 /path/to/local.mp4 本地文件绝对路径，或 HLS URL，或 Http URL，或 RTMP URL。

- 备注：

对于直播的播放地址获取如下：
客户端向业务服务器请求播放地址，业务服务器通过游密RestAPI向游密服务器请求播放地址。
RestAPI RTMP播放地址请求URL示例:
https://api.youme.im/v2/video/get_rtmp_play_url?appkey=123456789&identifier=admin&curtime=123456789&checksum=123456789abcdefg
详见RestAPI

示例如下：

```java
private String mVideoPath = null;
...
protected void onCreate(Bundle savedInstanceState) {
   ...
   
   mVideoView.setVideoPath(mVideoPath);
   
   ...
```

#### 4.9 播放控制
可以通过 YMVideoView 提供的接口自行进行播放过程的控制，包括：暂停、继续、停止等，相关函数如下：

```java
mVideoView.pause();
mVideoView.start();
mVideoView.stopPlayback();
```

## 5. 功能使用
当你要深入理解 SDK 的一些参数及有定制化需求时，可以从高级功能部分中查询阅读，以下小节无前后依赖。

### 5.1 关联播放控制器
将播放控制器控件关联到 PLVideoView，示例如下：

``` java
MediaController mMediaController = new MediaController(this);
mVideoView.setMediaController(mMediaController);
```

### 5.2 获取 RTMP Message Timestamp

可以通过 `getRtmpAudioTimestamp` 与 `getRtmpVideoTimestamp` 方法获取时间戳

``` java
/**
 * Get video timestamp in RTMP message
 * @return timestamp
 */
 
public long getRtmpVideoTimestamp();
/**
 * Get audio timestamp in RTMP message
 * @return timestamp
 */
public long getRtmpAudioTimestamp();
```

### 5.3 视频截图
可以通过 captureImage 方法进行视频截图，截图数据将会在 PLOnImageCapturedListener 中回调

``` java
/**
 * Capture video image
 * @param delayTimeMs 截取调用此方法后相应毫秒后的视频画面，仅对点播流生效
 */
public void captureImage(long delayTimeMs);
```

### 5.4 设置播放区域
可以通过 setVideoArea 方法，播放视频的一部分区域。若所有参数均为 0，则不裁剪画面。

``` java
/**
 * Set video area
 * @param topLeftX top left x
 * @param topLeftY top left y
 * @param bottomRightX bottom right x
 * @param bottomRightY bottom right y
 */
 public void setVideoArea(int topLeftX, int topLeftY, int bottomRightX, int bottomRightY);

```

### 5.5 设置 ZOrder 属性
可以像系统的 SurfaceView 一样设置 PLVideoView 的 ZOrder 属性，方法如下：

``` java
public void setZOrderOnTop(boolean onTop);
public void setZOrderMediaOverlay(boolean overlay);
```

### 5.6 暂停/恢复播放器的预缓冲
在播放点播流时，可以根据具体需求动态暂停/恢复播放器的预缓冲，方法如下：

``` java
/**
 * Set buffering enabled
 * @param enabled enable or not
 */
public void setBufferingEnabled(boolean enabled);

```

### 5.7 获取已经缓冲的长度
在播放 http 点播流时，可以获取播放器已经缓冲的 byte 数，方法如下：

``` java
/**
 * Get buffered length
 * @return length
 */
public BigInteger getHttpBufferSize();

```

### 5.8 播放参数配置
YouMePlayer SDK 提供的 YMAVOptions 类，可以用来配置播放参数，例如：

``` java
YMAVOptions options = new YMAVOptions();
public final static int MEDIA_CODEC_SW_DECODE = 0;
public final static int MEDIA_CODEC_HW_DECODE = 1;
public final static int MEDIA_CODEC_AUTO = 2;

// DNS 服务器设置
// 若不设置此项，则默认使用 DNSPod 的 httpdns 服务
// 若设置为 127.0.0.1，则会使用系统的 DNS 服务器
// 若设置为其他 DNS 服务器地址，则会使用设置的服务器
options.setString(YMAVOptions.KEY_DNS_SERVER, server);

// DNS 缓存设置
// 若不设置此项，则每次播放未缓存的域名时都会进行 DNS 解析，并将结果缓存
// 参数为 String[]，包含了要缓存 DNS 结果的域名列表
// SDK 在初始化时会解析列表中的域名，并将结果缓存
options.setStringArray(YMAVOptions.KEY_DOMAIN_LIST, domainList);

// 解码方式:
// codec＝YMAVOptions.MEDIA_CODEC_HW_DECODE，硬解
// codec=YMAVOptions.MEDIA_CODEC_SW_DECODE, 软解
// codec=YMAVOptions.MEDIA_CODEC_AUTO, 硬解优先，失败后自动切换到软解
// 默认值是：MEDIA_CODEC_SW_DECODE
options.setInteger(AVOptions.KEY_MEDIACODEC, codec);

// 若设置为 1，则底层会进行一些针对直播流的优化
options.setInteger(YMAVOptions.KEY_LIVE_STREAMING, 1);

// 快开模式，启用后会加快该播放器实例再次打开相同协议的视频流的速度
options.setInteger(YMAVOptions.KEY_FAST_OPEN, 1);

// 打开重试次数，设置后若打开流地址失败，则会进行重试
options.setInteger(YMAVOptions.KEY_OPEN_RETRY_TIMES, 5);

// 预设置 SDK 的 log 等级， 0-4 分别为 v/d/i/w/e
options.setInteger(YMAVOptions.KEY_LOG_LEVEL, 2);

// 打开视频时单次 http 请求的超时时间，一次打开过程最多尝试五次
// 单位为 ms
options.setInteger(YMAVOptions.KEY_PREPARE_TIMEOUT, 10 * 1000);

// 默认的缓存大小，单位是 ms
// 默认值是：500
options.setInteger(YMAVOptions.KEY_CACHE_BUFFER_DURATION, 500);

// 最大的缓存大小，单位是 ms
// 默认值是：2000，若设置值小于 KEY_CACHE_BUFFER_DURATION 则不会生效
options.setInteger(YMAVOptions.KEY_MAX_CACHE_BUFFER_DURATION, 4000);

// 是否开启直播优化，1 为开启，0 为关闭。若开启，视频暂停后再次开始播放时会触发追帧机制
// 默认为 0
options.setInteger(YMAVOptions.KEY_LIVE_STREAMING);

// 设置拖动模式，1 位精准模式，即会拖动到时间戳的那一秒；0 为普通模式，会拖动到时间戳最近的关键帧。默认为 0
options.setInteger(YMAVOptions.KEY_SEEK_MODE);

// 设置 DRM 密钥
byte[] key = {xxx, xxx, xxx, xxx, xxx ……};
options.setByteArray(YMAVOptions.KEY_DRM_KEY, key);

// 设置偏好的视频格式，设置后会加快对应格式视频流的打开速度，但播放其他格式会出错
// m3u8 = 1, mp4 = 2, flv = 3
options.setInteger(YMAVOptions.KEY_PREFER_FORMAT, 1);

// 开启解码后的视频数据回调
// 默认值为 0，设置为 1 则开启
options.setInteger(YMAVOptions.KEY_VIDEO_DATA_CALLBACK, 1);

// 开启解码后的音频数据回调
// 默认值为 0，设置为 1 则开启
options.setInteger(YMAVOptions.KEY_VIDEO_DATA_CALLBACK, 1);

// 设置开始播放位置
// 默认不开启，单位为 ms
options.setInteger(YMAVOptions.KEY_START_POSITION, 10 * 1000);

// 请在开始播放之前配置
mVideoView.setAVOptions(options);
```

### 5.9 播放状态回调
#### 5.9.1 onPrepared

``` java
public interface YMVideoListener {
    /**
     * prepare回调
     *     
     * @param errorcode 错误码
     */
    public void onPrepared(int errorcode);
    
    ...
}    
```

该回调监听播放器的 prepare 过程，该过程主要包括：创建资源、建立连接、请求码流等等，当 prepare 完成后，SDK 会回调该对象的 onPrepared 接口，下一步则可以调用播放器的 start() 启动播放。

#### 5.9.2 onInfo 

``` java
public interface YMVideoListener {
    /**
     * @param what 消息类型
     * @param extra 附加参数 
     */
    void onInfo(int what, int extra);
    
    ...
}    
```

| what | value | 描述 |
|---|---|---|
| MEDIA_INFO_UNKNOWN | 1 | 未知消息 |
| MEDIA_INFO_VIDEO_RENDERING_START | 3 | 第一帧视频已成功渲染 |
| MEDIA_INFO_CONNECTED | 200 | 连接成功 |
| MEDIA_INFO_METADATA | 340 | 读取到 metadata 信息 |
| MEDIA_INFO_BUFFERING_START | 701 | 开始缓冲 |
| MEDIA_INFO_BUFFERING_END | 702 | 停止缓冲 |
| MEDIA_INFO_SWITCHING_SW_DECODE | 802 | 硬解失败，自动切换软解 |
| MEDIA_INFO_LOOP_DONE | 8088 | loop 中的一次播放完成 |
| MEDIA_INFO_VIDEO_ROTATION_CHANGED | 10001 | 获取到视频的播放角度 |
| MEDIA_INFO_AUDIO_RENDERING_START | 10002 | 第一帧音频已成功播放 |
| MEDIA_INFO_VIDEO_GOP_TIME | 10003 | 获取视频的I帧间隔 |
| MEDIA_INFO_VIDEO_BITRATE | 20001 | 视频的码率统计结果 |
| MEDIA_INFO_VIDEO_FPS | 20002 | 视频的帧率统计结果 |
| MEDIA_INFO_AUDIO_BITRATE | 20003 | 音频的帧率统计结果 |
| MEDIA_INFO_AUDIO_FPS | 20004 | 音频的帧率统计结果 |
| MEDIA_INFO_VIDEO_FRAME_RENDERING | 10004 | 视频帧的时间戳 |
| MEDIA_INFO_AUDIO_FRAME_RENDERING | 10005 | 音频帧的时间戳 |
| MEDIA_INFO_CACHED_COMPLETE | 1345 | 离线缓存的部分播放完成 |
| MEDIA_INFO_IS_SEEKING | 564 | 上一次 seekTo 操作尚未完成 |

#### 5.9.3 onCompletion

``` java
public interface YMVideoListener {
    
    void onCompletion();    
    ...
}    
```

该回调监听播放结束的消息，关于该回调的时机，有如下定义：

如果是播放文件，则是播放到文件结束后产生回调
如果是在线视频，则会在读取到码流的EOF信息后产生回调，回调前会先播放完已缓冲的数据
如果播放过程中产生onError，并且没有处理的话，最后也会回调本接口
如果播放前设置了 setLooping(true)，则播放结束后会自动重新开始，不会回调本接口
如果同时将 YMAVOptions.KEY_FAST_OPEN 与 YMAVOptions.KEY_SEEK_MODE 设置为 1，并且希望在收到本接口后播放同一个视频，需要在 start 后手动调用 seekTo(0)

#### 5.9.4 onError

``` java
public interface YMVideoListener {
    
    boolean onError(int errorCode);    
    ...
}    
```

| errorCode | value | 描述 |
|---|---|---|
| MEDIA_ERROR_UNKNOWN | -1 | 未知错误 |
| ERROR_CODE_OPEN_FAILED | -2 | 播放器打开失败 |
| ERROR_CODE_IO_ERROR | -3 | 网络异常 |
| ERROR_CODE_SEEK_FAILED | -4 | 拖动失败 |
| ERROR_CODE_HW_DECODE_FAILURE | -2003 | 硬解失败 |
| ERROR_CODE_PLAYER_DESTROYED | -2008 | 播放器已被销毁，需要再次 setVideoURL 或 prepareAsync |
| ERROR_CODE_PLAYER_VERSION_NOT_MATCH | -9527 | so 库版本不匹配，需要升级 |
| ERROR_CODE_PLAYER_CREATE_AUDIO_FAILED | -4410 | AudioTrack 初始化失败，可能无法播放音频 |

#### 5.9.5 onBufferingUpdate

``` java
public interface YMVideoListener {
    
    void onBufferingUpdate(int percent);    
    ...
}    
```

该回调用于监听当前播放器已经缓冲的数据量占整个视频时长的百分比，在播放直播流中无效，仅在播放文件和回放时才有效。

#### 5.9.6 onVideoSizeChanged

``` java
public interface YMVideoListener {
    
    void onVideoSizeChanged(int width, int height);   
    ...
}    
```

该回调用于监听当前播放的视频流的尺寸信息，在 SDK 解析出视频的尺寸信息后，会触发该回调，开发者可以在该回调中调整 UI 的视图尺寸。

#### 5.9.7 onSeekComplete

``` java
public interface YMVideoListener {
    
    void onSeekComplete();   
    ...
}    
```

该回调用于监听 seek 完成的消息，当调用的播放器的 seekTo 方法后，SDK 会在 seek 成功后触发该回调。

#### 5.9.8 onImageCaptured

``` java
public interface YMVideoListener {
    
    /**
     * On image captured
     * @param data 编码后的 jpeg 截图数据
     */
    void onImageCaptured(byte[] data);   
    ...
}    
```

该回调用于监听 captureImage 完成的消息，当调用播放器的 captureImage 方法后，SDK 会在相应内容截取完成后触发该回调。data 参数为编码后的 jpeg 图像数据，可以直接保存为 jpeg 文件。

### 5.10 连接状态处理
#### 5.11 如何判断直播结束
对于直播应用而言，播放器本身是无法判断直播是否结束，这需要通过业务服务器来告知。当主播端停止推流后，播放器会因为读取不到新的数据而产生超时，从而触发 ERROR_CODE_IO_ERROR 回调。

建议的处理方式是：在 ERROR_CODE_IO_ERROR 回调后，查询业务服务器，获知直播是否结束，如果已经结束，则关闭播放器，清理资源；如果直播没有结束，则等待 SDK 内部自动做重连。

建议收到 ERROR_CODE_IO_ERROR 之后，使用如下方法判断一下网络的联通性，如果用户的网络已断，则可以退出播放。

```java
public static boolean isNetworkAvailable(Context context) {
        ConnectivityManager cm = (ConnectivityManager) context.getSystemService(Context.CONNECTIVITY_SERVICE);
        NetworkInfo netInfo = cm.getActiveNetworkInfo();
        return netInfo != null && netInfo.isConnectedOrConnecting();
    }
```

注意：使用网络要添加如下权限，

```xml
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
```

#### 5.12 网络异常的处理
如果申请的直播并没有在推流，或者直播过程中发生网络错误（比如：WiFi 断开），播放器在请求超时或者播放完当前缓冲区中的数据后，会触发 onError 回调，errorCode 通常是 ERROR_CODE_IO_ERROR，触发 ERROR_CODE_IO_ERROR 消息后，SDK 内部会自动重连，这种情况下，App 层面通常可以做如下判断来考虑是否需要退出播放：

查询业务服务器，获知直播是否结束，如果已经结束，则可以退出播放；
判断网络是否断开，如果已经断网，则可以提出播放。

### 5.11 播放器音量调节
如果期望调节播放器的音量，接口如下所示：

```java
public void setVolume(float leftVolume, float rightVolume);
```

音量参数的取值范围是：0.0～1.0，使用如下代码可以达到静音的效果：

```java
mVideoView.setVolume(0.0f, 0.0f);
```

### 5.12 设置播放封面
在开始播放前，YMVideoView 可以显示封面图片，设置接口如下：

``` java
public void setCoverView(View coverView);
```

### 5.13 获取播放器当前状态
在播放过程中，用户可以调用接口，获取播放器当前状态，接口如下：

``` java
public PlayerState getPlayerState();
```

### 5.14 获取 METADATA 信息
在播放过程中，用户可以调用接口，获取当前播放流的 METADATA 信息，接口如下：

``` java
public HashMap<String, String> getMetadata();

```

### 5.15 获取实时统计信息

``` java
public int getVideoFps();
public long getVideoBitrate();
public int getAudioBitrate();
public int getAudioFps();

``` 

### 5.16 倍数播放

``` java
/**
 * 设置倍数播放
 * @param speed 倍数值，16 进制表示，高 4 位代表分子，低 4 位代表分母
 * 例如：0X00010002 表示 0.5 倍数，0X00040001 表示 4 倍数
 * 范围：0.1～32 倍数
 */
public boolean setPlaySpeed(int speed);
/**
 * 设置倍数播放
 * @param speed 倍数值，0.1 - 32
 */
public boolean setPlaySpeed(float speed);

```

### 5.17 本地缓存功能
本地缓存功能，目前只支持 HTTP(s)-mp4 文件点播，开启本地缓存功能，只需要在播放前，配置缓存目录即可

``` java
// 设置本地缓存目录
// 目前只支持 mp4 点播
// 默认值是：无
options.setString(AVOptions.KEY_CACHE_DIR, dir);

// 设置本地缓存文件的后缀名
// 只有在设置了缓存目录后才会生效
// 一个流地址在设置了自定义后缀名后，再次播放前必须设置相同的后缀名，否则无法打开
// 默认值是 mp4
options.setString(AVOptions.KEY_CACHE_EXT, ext);
```

### 5.18 音视频数据回调  
播放支持将解码后的音视频数据回调出来，相关接口如下所述：

``` java
public interface YMVideoListener {
   /**
    * 回调一帧视频帧数据
    *
    * @param data   视频帧数据
    * @param size   数据大小
    * @param width  视频帧的宽
    * @param height 视频帧的高
    * @param format 视频帧的格式，0代表 YUV420P，1 代表 JPEG， 2 代表 SEI
    * @param ts     时间戳，单位是毫秒
    */
    void onVideoFrameAvailable(byte[] data, int size, int width, int height, int format, long ts);

   /**
    * 回调一帧音频帧数据
    *
    * @param data   音频帧数据
    * @param size   数据大小
    * @param samplerate 采样率
    * @param channels 通道数
    * @param datawidth 位宽，目前默认转换为了16bit位宽
    * @param ts     时间戳，单位是毫秒
    */
    void onAudioFrameAvailable(byte[] data, int size, int samplerate, int channels, int datawidth, long ts);
    
    ...
    
}    

```

*** 如何开启音视频数据回调 ？***

- 配置 YMAVOption，开启音视频数据回调

``` java
// 开启解码后的视频数据回调
// 默认值为 0，设置为 1 则开启
options.setInteger(YMAVOptions.KEY_VIDEO_DATA_CALLBACK, 1);

// 开启解码后的音频数据回调
// 默认值为 0，设置为 1 则开启
options.setInteger(YMAVOptions.KEY_VIDEO_DATA_CALLBACK, 1);
```

- 设置音视频数据回调的监听对象

``` java

mVideoView.setYMVideoListener(new YMVideoListener() {
    @Override
    public void onVideoFrameAvailable(byte[] data, int size, int width, int height, int format, long ts) {
        Log.i(TAG, "onVideoFrameAvailable: " + size + ", " + width + " x " + height + ", i420, " + ts);
    }
    
    @Override
    public void onAudioFrameAvailable(byte[] data, int size, int samplerate, int channels, int datawidth, long ts) {
        Log.i(TAG, "onAudioFrameAvailable: " + size + ", " + samplerate + ", " + channels + ", " + datawidth + ", " + ts);
    }
    
    ...

});

```

### 5.19 自定义音视频渲染
在开启 ##5.18 音视频数据回调 的基础上，再开启如下配置，即可关闭 SDK 内部的视频渲染和音频播放

``` java
// 开启自定义视频数据渲染
// 默认值是：0，由 SDK 内部渲染视频
options.setInteger(YMAVOptions.KEY_VIDEO_RENDER_EXTERNAL, 1);

// 开启自定义音频数据播放
// 默认值是：0，由 SDK 内部播放音频
options.setInteger(YMAVOptions.KEY_VIDEO_RENDER_EXTERNAL, 1);

```

### 5.20 设置日志等级
可以在创建播放器时通过 YMAVOptions.KEY_LOG_LEVEL 设置，参数 0-4 分别为 v/d/i/w/e，若希望完全去除日志打印，则可设置大于 4 的整数。

