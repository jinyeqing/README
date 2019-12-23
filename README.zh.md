# Agora Video SDK Android Tutorial - 1to1

## 是什么

这个开源示例项目是为了帮助开发者更好，更快捷了解，学习，集成 Agora 视频 SDK，演示了如何快速集成 Agora 视频 SDK，实现1对1视频通话。

在这个示例项目中主要包含了以下功能：

* 加入通话和离开通话；
* 静音和解除静音；
* 关闭摄像头和开启摄像头；
* 切换前置摄像头和后置摄像头；


### 功能和场景
 主要功能       | 功能描述                                                                                  | 典型适用场景                                                                                             |
|------------|---------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------|
| 伴奏混音       | 将本地或在线的音频和用户声音，同时发送并播放给频道内其他用户                                                        | 在线合唱 音乐互动课堂                                                                                        |
| 基础美颜       | 支持基础的美颜功能，包括设置美白、磨皮、祛痘、红润效果。                                                          | 视频聊天美颜                                                                                             |
| 屏幕共享       | 把屏幕内容同步展示给频道内的其他用户，支持指定共享某个屏幕或窗口，同时支持指定共享区域。                                          | 互动课堂                                                                                               |
| 修改音视频原始数据  | 可支持变声，支持获取媒体引擎的原始语音或视频数据，对原始数据进行处理                                                    | 聊天室变声 视频聊天美颜                                                                                       |
| 自定义视频源和渲染器 | 支持自定义的视频源和渲染器，可以不使用系统摄像头，使用自己构建的 camera 视频源，屏幕共享视频源，或者文件视频源等，可以更灵活地处理视频，比如添加美颜效果、滤镜等。 | 需要使用自定义的美颜库或者前处理库 开发者 App 中已经有自己的图像视频模块 开发者希望使用非 Camera 的视频源，如录屏数据 有些系统独占的视频采集设备，为了避免冲突，需灵活的设备管理策略 |


## 为什么要使用 Agora 视频 SDK
### (1)产品优势
- 超低延时,超高质量
- 实时监控让质量透明可见
- 弹性可扩展    from Zero to Hero
- 每月 10000 分钟免费

### (2)关键特性

| 特性      | Agora 视频通话指标                          |
|---------|---------------------------------------|
| SDK 包体积 | 3\.69 M \- 7\.75 M                    |
| 多人视频    | 支持 7 人视频通话（如需更多人数，可以使用 Agora 互动直播）    |
| 视频属性    | SDK 采集支持 1080p 分辨率，60 fps 帧率 自采集支持 4K |
| 音频属性    | 音频采样率：16 KHz \- 48 KHz 支持单、双声道        |
| 音频抗丢包率  | 上下行抗丢包率 70%                           |

## 如何使用

### (1)环境准备
* Android Studio 3.3+
* 真实 Android 设备 (Nexus 5X 或者其它设备)
* 部分模拟器会存在功能缺失或者性能问题，所以推荐使用真机

### (2)创建Agora账号并获取AppId
在编译和启动实例程序前，您需要首先获取一个可用的App ID:
1. 在[agora.io](https://dashboard.agora.io/signin/)创建一个开发者账号
2. 前往后台页面，点击左部导航栏的 **项目 > 项目列表** 菜单
3. 复制后台的 **App ID** 并备注，稍后启动应用时会用到它
4. 在项目页面生成临时 **Access Token** (24小时内有效)并备注，注意生成的Token只能适用于对应的频道名。

5. 将 AppID 填写进 "app/src/main/res/values/strings.xml"
  ```
  <string name="private_app_id"><#YOUR APP ID#></string>
  <!-- 临时Token 可以在 https://dashboard.agora.io 获取 -->
  <!-- 在正式上线生产环境前，你必须部署你自己的Token服务器 -->
  <!-- 如果你的项目没有打开安全证书，下面的值可以直接留空 -->
  <string name="agora_access_token"><#YOUR TOKEN#></string>
  ```
### (3)集成 Agora 视频 SDK
集成方式有以下两种：
  - 首选集成方式：
    - 在项目对应的模块的 `app/build.gradle` 文件的依赖属性中加入通过 JCenter 自动集成 Agora 视频 SDK 的地址：
      ```
      implementation 'io.agora.rtc:full-sdk:2.4.1'
      ```
      (如果要在自己的应用中集成 Agora 视频 SDK，添加链接地址是最重要的一步。）
    - 在 [Agora.io SDK](https://www.agora.io/cn/download/) 下载 **视频通话 + 直播 SDK**，解压后将其中的 **libs**/**include** 文件夹下的 ***.h** 复制到本项目的 **app**/**src**/**main**/**cpp**/**agora** 下。
  - 次选集成方式：
    - 在 [Agora.io SDK](https://www.agora.io/cn/download/) 下载 **视频通话 + 直播 SDK**并解压，按以下对应关系将 **libs** 目录的内容复制到项目内。
      
      SDK目录|项目目录
      ---|---
      .jar file|**/apps/libs** folder
      **arm64-v8a** folder|**/app/src/main/jniLibs** folder
      **x86** folder|**/app/src/main/jniLibs** folder
      **armeabi-v7a** folder|**/app/src/main/jniLibs** folder
      

### (4)实现音视频通话原理
本节介绍SDK是如何实现音视频通话，选取一些关键步骤的代码示例做介绍。视频通话的 API 调用时序见下图：
![API调用时序图](https://github.com/jinyeqing/README/blob/master/imag/1568254412236.png)

* 初始化 RtcEngine

在调用其他 Agora API 前，需要创建并初始化 RtcEngine 对象。调用 create 方法，传入获取到的 App ID，即可初始化 RtcEngine。你还根据场景需要，在初始化时注册想要监听的回调事件，如本地用户加入频道，及解码远端用户视频首帧等。注意不要在这些回调中进行 UI 操作。
```
private final IRtcEngineEventHandler mRtcEventHandler = new IRtcEngineEventHandler() {
    @Override
    // 注册 onJoinChannelSuccess 回调。
    // 本地用户成功加入频道时，会触发该回调。
    public void onJoinChannelSuccess(String channel, final int uid, int elapsed) {
        runOnUiThread(new Runnable() {
            @Override
            public void run() {
                Log.i("agora","Join channel success, uid: " + (uid & 0xFFFFFFFFL));
            }
        });
    }

    @Override
    // 注册 onFirstRemoteVideoDecoded 回调。
    // SDK 接收到第一帧远端视频并成功解码时，会触发该回调。
    // 可以在该回调中调用 setupRemoteVideo 方法设置远端视图。
    public void onFirstRemoteVideoDecoded(final int uid, int width, int height, int elapsed) {
        runOnUiThread(new Runnable() {
            @Override
            public void run() {
                Log.i("agora","First remote video decoded, uid: " + (uid & 0xFFFFFFFFL));
                setupRemoteVideo(uid);
            }
        });
    }

    @Override
    // 注册 onUserOffline 回调。
    // 远端用户离开频道或掉线时，会触发该回调。
    public void onUserOffline(final int uid, int reason) {
        runOnUiThread(new Runnable() {
            @Override
            public void run() {
                Log.i("agora","User offline, uid: " + (uid & 0xFFFFFFFFL));
                onRemoteUserLeft();
            }
        });
    }
};

...

// 初始化 RtcEngine 对象。
private void initializeEngine() {
    try {
        mRtcEngine = RtcEngine.create(getBaseContext(), getString(R.string.agora_app_id), mRtcEventHandler);
    } catch (Exception e) {
        Log.e(TAG, Log.getStackTraceString(e));
        throw new RuntimeException("NEED TO check rtc sdk init fatal error\n" + Log.getStackTraceString(e));
    }
}
```

* 加入频道

完成初始化和设置本地视图后（视频通话场景），你就可以调用 joinChannel 方法加入频道。更多的参数设置注意事项请参考 [joinChannel](https://docs.agora.io/cn/Video/API%20Reference/java/classio_1_1agora_1_1rtc_1_1_rtc_engine.html#a8b308c9102c08cb8dafb4672af1a3b4c) 接口中的参数描述。

```
private void joinChannel() {

    // 使用 Token 加入频道。
    mRtcEngine.joinChannel(YOUR_TOKEN, "demoChannel1", "Extra Optional Data", 0);
}
```

* 离开频道

根据场景需要，如结束通话、关闭 App 或 App 切换至后台时，调用 leaveChannel 离开当前通话频道。

```
@Override
protected void onDestroy() {
    super.onDestroy();
    if (!mCallEnd) {
        leaveChannel();
    }
    RtcEngine.destroy();
}

private void leaveChannel() {
    // 离开当前频道。
    mRtcEngine.leaveChannel();
}
```

* 更多步骤

你还可以在通话中参考[代码](https://docs.agora.io/cn/Video/start_call_android?platform=Android#%E8%BF%90%E8%A1%8C%E9%A1%B9%E7%9B%AE)实现更多功能及场景。

1.获取设备权限

2.设置本地/远端试图

3.停止发送本地音频流

4.切换前后摄像头


### (5)启动应用程序
* 用 Android Studio 打开该项目，连上设备，编译并运行。
* 也可以使用 Gradle 直接编译运行。


## 常见问题
* [听不到声音](https://docs.agora.io/cn/faq/audio_noaudio)
* [视频黑屏](https://docs.agora.io/cn/faq/video_blank)
* [如何区分媒体音量和通话音量](https://docs.agora.io/cn/faqs/system_volume)
* 其他更多常见问题可以参考 [FAQ](https://docs.agora.io/cn/Video/faq)


## 联系我们
* 产品的新手指南及完整的 [文档中心](https://docs.agora.io/cn/)
* SDK快速指南视频教程见 [视频指南](https://space.bilibili.com/237765579/video)
* 如果在集成中遇到问题, 你可以到 [开发者社区](https://dev.agora.io/cn/) 提问
* 如果需要售后技术支持, 你可以在[Agora Dashboard](https://dashboard.agora.io)提交工单
* 如果发现了示例代码的 bug, 欢迎提交[issue](https://github.com/AgoraIO/Basic-Video-Call/issues)

## 开源协议
* The MIT License (MIT)
