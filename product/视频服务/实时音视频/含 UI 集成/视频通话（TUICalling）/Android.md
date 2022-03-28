## 组件介绍

TUICalling 是一个开源的音视频 UI 组件，通过在项目中集成 TUICalling 组件，您只需要编写几行代码就可以为您的 App 添加“一对一是视频通话”，“多人音视频通话”等场景，并且支持离线唤起能力。TUICalling 包含 Android、iOS、Web、小程序、Flutter、Uniapp 等平台的源代码，基本功能如下图所示：

<table class="tablestyle">
<tbody><tr>
<td><img src="https://qcloudimg.tencent-cloud.cn/raw/c792c6fa94e4c4dd3151003b0f28eab7.png" width="250"></td>
<td><img src="https://qcloudimg.tencent-cloud.cn/raw/a75f7d5f2911f2e21d3e4b3b9dfc4db5.png" width="500"></td>
<td><img src="https://qcloudimg.tencent-cloud.cn/raw/3bb1a748b590518e2be09326cc30dc0b.png" width="250"></td>
</tr>
</tbody></table>

>? 如果您有任何咨询或建议，可以加入 QQ 群 **592465424** 一起交流~

## 二. 组件集成

### 步骤一：下载并导入 TUICalling 组件

点击进入 [Github](https://github.com/tencentyun/TUICalling) ，选择克隆/下载代码，然后拷贝 tuicalling 目录到您的工程中，并完成如下导入动作：
- 在 `setting.gradle` 中完成导入，参考如下：
```
include ':tuicalling'
```

- 在 app 的 build.gradle 文件中添加对 tuicalling 的依赖：
```
api project(':tuicalling')
```

### 步骤二：配置权限及混淆规则

在 AndroidManifest.xml 中配置 App 的权限，SDK 需要以下权限（6.0以上的 Android 系统需要动态申请相机、麦克风、读取存储权限等）：

```
<uses-permission android:name="android.permission.SYSTEM_ALERT_WINDOW" />        // 使用场景：悬浮窗、应用在后台时拉起通话界面时需要此权限；
<uses-permission android:name="android.permission.INTERNET" />              
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
<uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
<uses-permission android:name="android.permission.RECORD_AUDIO" />
<uses-permission android:name="android.permission.MODIFY_AUDIO_SETTINGS" />
<uses-permission android:name="android.permission.BLUETOOTH" />                  // 使用场景：使用蓝牙耳机时需要此权限；
<uses-permission android:name="android.permission.CAMERA" />
<uses-permission android:name="android.permission.READ_PHONE_STATE" />           // 使用场景：判断是否是系统来电打断时需要此权限；
<uses-feature android:name="android.hardware.camera"/>
<uses-feature android:name="android.hardware.camera.autofocus" />
```

在 proguard-rules.pro 文件，将 SDK 相关类加入不混淆名单：

```
-keep class com.tencent.** { *; }
```

### 步骤三：创建并初始化 TUI 组件库

```java
  // 1.组件登录，
  TUILogin.init(this, "您的SDKAppId", config, new V2TIMSDKListener() {
            @Override
            public void onKickedOffline() {  // 登录被踢下线通知（示例：账号再其他设备登录）
            }
            @Override
            public void onUserSigExpired() { // userSig过期通知
            }
  });
  TUILogin.login("您的userId", "您的userSig", new V2TIMCallback() {
            @Override
            public void onError(int code, String msg) {
                Log.d(TAG, "code: " + code + " msg:" + msg);
            }
            @Override
            public void onSuccess() {
            }
  });
  
  // 2.初始化TUICalling实例
  TUICalling callingImpl = TUICallingImpl.sharedInstance(context);
  
```
#### 3.1 参数说明
- **SDKAppID**：**TRTC 应用ID**，如果您未开通腾讯云 TRTC 服务，可以点击 [这里](https://console.cloud.tencent.com/trtc/app)，进入腾讯云实时音视频控制台，创建一个新的的TRTC应用后，点击应用信息，SDKAppID信息如下图所示；
 <img src="https://liteav.sdk.qcloud.com/app/doc/app_manager_sdk_secretkey.png" width="700">
	
- **Secretkey**：**TRTC 应用密钥**，和SDKAppId对应，进入 [TRTC 应用管理](https://console.cloud.tencent.com/trtc/app) 后，SecretKey信息如上图所示；

- **userId**：当前用户的 ID，字符串类型，只允许包含英文字母（a-z 和 A-Z）、数字（0-9）、连词符（-）和下划线（_）。建议结合业务实际账号体系自行设置。

- **userSig**：根据SDKAppId、userId，Secretkey等信息计算得到的安全保护签名，您可以点击 [这里](https://console.cloud.tencent.com/trtc/usersigtool) 直接在线生成一个调试的userSig，也可以参照我们的[示例工程](https://github.com/tencentyun/TUICalling/blob/main/Android/App/src/main/java/com/tencent/liteav/demo/LoginActivity.java#L74)自行计算，更多信息见 [如何计算及使用 UserSig](https://cloud.tencent.com/document/product/647/17275)。


### 步骤四：实现音视频通话

#### 4.1. 实现1对1视频通话
```java
// 3. 发起1对1视频通话，假设userId为：1111；
callingImpl.call(["1111"], TUICalling.Type.VIDEO);
```

#### 4.2. 实现多人视频通话：
```java
// 4. 发起多人视频通话，假设userId分别为：1111、2222、3333；
callingImpl.call(["1111", "2222", "3333"], TUICalling.Type.VIDEO);
```

>? 当接收方完成步骤三后，即登录成功后，在收到通话请求后，TUICalling组件会自动启动相应的接听界面。

### 步骤五：实现离线唤醒（可选）

完成以上四个步骤，就可以实现视频通话的拨打和接通，但如果您的业务场景需要在App 的进程被杀死后，还可以正常接收到视频通话请求，就需要为 TUICalling 组件增加离线推送功能，详情见 [**Android离线推送接入指引.md**](https://github.com/tencentyun/TUICalling/blob/main/Android/Android%E7%A6%BB%E7%BA%BF%E6%8E%A8%E9%80%81%E6%8E%A5%E5%85%A5%E6%8C%87%E5%BC%95.md)。

## 三. 常见问题

**Q: TUICalling 组件支不支持悬浮窗**

A: 支持，组件初始化时调用`callingImpl.enableFloatWindow(true)`即可；

**Q: TUICalling 组件支持自定义头像和昵称吗？**

A: 支持，调用`setUserNickname/setUserAvatar`即可；

>? 更多帮助信息，详见 [TUI 场景化解决方案常见问题](https://cloud.tencent.com/developer/article/1952880)，或者点击[这里](https://cloud.tencent.com/document/product/647/70641)联系我们。