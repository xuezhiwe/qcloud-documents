## 组件介绍

TUICalling 是一个开源的音视频 UI 组件，通过在项目中集成 TUICalling 组件，您只需要编写几行代码就可以为您的 App 添加“一对一音视频通话”，“多人音视频通话”等场景，并且支持离线唤起能力。TUICalling 包含 Android、iOS、Web、小程序、Flutter、UniApp 等平台的源代码，基本功能如下图所示：

<table class="tablestyle">
<tbody><tr>
<td><img src="https://qcloudimg.tencent-cloud.cn/raw/c792c6fa94e4c4dd3151003b0f28eab7.png" width="250"></td>
<td><img src="https://qcloudimg.tencent-cloud.cn/raw/a75f7d5f2911f2e21d3e4b3b9dfc4db5.png" width="500"></td>
<td><img src="https://qcloudimg.tencent-cloud.cn/raw/3bb1a748b590518e2be09326cc30dc0b.png" width="250"></td>
</tr>
</tbody></table>

## 二. 组件集成

### 步骤一：导入 TUICalling 组件

**通过 cocoapods 导入组件**，具体步骤如下：

1. 点击进入 [**Github**](https://github.com/tencentyun/TUICalling) ，选择克隆/下载代码，然后将iOS目录下的 `Source`、`Resources` 文件夹 和 `TUICalling.podspec` 文件拷贝到您的工程目录下。
2. 在您的 `Podfile` 文件中添加以下依赖。之后执行 `pod install` 命令，完成导入。

```swift\
# :path => "指向TUICalling.podspec所在目录的相对路径"
pod 'TUICalling', :path => "../", :subspecs => ["TRTC"]
```

>!  `Source`、`Resources` 文件夹 和`TUICalling.podspec`文件必需在同一目录下。

### 步骤二：配置权限

使用音视频功能，需要授权麦克风和摄像头的使用权限。在 App 的 Info.plist 中添加以下两项，分别对应麦克风和摄像头在系统弹出授权对话框时的提示信息。
- **Privacy - Microphone Usage Description**，并填入麦克风使用目的提示语。
- **Privacy - Camera Usage Description**，并填入摄像头使用目的提示语。

![](https://main.qcloudimg.com/raw/54cc6989a8225700ff57494cba819c7b.jpg)

### 步骤三：创建并初始化 TUI 组件库

```
// 1.组件登录，
TUILogin.initWithSdkAppID(SDKAPPID)
TUILogin.login(userId, userSig) {
    print("login success")
} fail: { code, errorDes in
    print("login failed, code:\(code), error: \(errorDes ?? "nil")")
}
  
// 2.初始化TUICalling实例
TUICalling.sharedInstance();
```
#### 3.1 参数说明
- **SDKAppID**：**TRTC 应用ID**，如果您未开通腾讯云 TRTC 服务，可以点击 [这里](https://console.cloud.tencent.com/trtc/app)，进入腾讯云实时音视频控制台，创建一个新的的TRTC应用后，点击应用信息，SDKAppID信息如下图所示；
 <img src="https://liteav.sdk.qcloud.com/app/doc/app_manager_sdk_secretkey.png" width="700">
	
- **Secretkey**：**TRTC 应用密钥**，和SDKAppId对应，进入 [TRTC 应用管理](https://console.cloud.tencent.com/trtc/app) 后，SecretKey信息如上图所示；

- **userId**：当前用户的 ID，字符串类型，只允许包含英文字母（a-z 和 A-Z）、数字（0-9）、连词符（-）和下划线（_）。建议结合业务实际账号体系自行设置。

- **userSig**：根据SDKAppId、userId，Secretkey等信息计算得到的安全保护签名，您可以点击 [这里](https://console.cloud.tencent.com/trtc/usersigtool) 直接在线生成一个调试的userSig，也可以参照我们的[示例工程](https://github.com/tencentyun/TUICalling/blob/main/Android/App/src/main/java/com/tencent/liteav/demo/LoginActivity.java#L74)自行计算，更多信息见 [如何计算及使用 UserSig](https://cloud.tencent.com/document/product/647/17275)。


### 步骤四：实现音视频通话

#### 4.1. 实现1对1视频通话
```
// 发起1对1视频通话，假设userId为：1111；
TUICalling.shareInstance().call(userIDs: ["1111"], type: .video)
```

#### 4.2. 实现多人视频通话：
```
// 发起多人视频通话，假设userId分别为：1111、2222、3333；
TUICalling.shareInstance().call(userIDs: ["1111", "2222", "3333"], type: .video)
```

>? 当接收方完成步骤三后，即登录成功后，再收到通话请求后，TUICalling组件会自动启动相应的接听界面。

### 步骤五：实现离线唤醒（可选）

完成以上四个步骤，就可以实现视频通话的拨打和接通，但如果您的业务场景需要`在App 的进程被杀死后`或者`APP 退到后台后`，还可以正常接收到音视频通话请求，就需要为 TUICalling 组件增加离线推送功能，详情见 [**离线推送（iOS）**](https://cloud.tencent.com/document/product/269/44517)。

### 步骤六：监听通话状态（可选）
如果您的业务需要监听通话的状态，例如通话开始、结束(忙线、超时无响应、拒绝、取消)等，您可以监听以下事件：
```Swift
TUICalling.shareInstance().setCallingListener(listener: self)

public func shouldShowOnCallView() -> Bool {
    return true
}
    
public func callStart(userIDs: [String], type: TUICallingType, role: TUICallingRole, viewController: UIViewController?) {

}
    
public func callEnd(userIDs: [String], type: TUICallingType, role: TUICallingRole, totalTime: Float) {

}
    
public func onCallEvent(event: TUICallingEvent, type: TUICallingType, role: TUICallingRole, message: String) {

}
```

### 步骤七：开启悬浮窗功能（可选）
如果您的业务需要开启悬浮窗功能，您可以在TUICalling组件初始化时调用`TUICalling.shareInstance().enableFloatWindow(enable: true)`开启该功能。

>? 目前组件仅支持应用内悬浮窗(最小化退到上一层界面)

## 三. 常见问题

**Q: TUICalling 组件支持自定义头像和昵称吗？**

A: 支持，调用`setUserNickname/setUserAvatar`即可；

**Q: TUICalling 组件支持自定义铃声吗？**

A: 支持，调用`setCallingBell`即可；

**Q: CocoaPods如何安装？**

在终端窗口中输入如下命令（需要提前在 Mac 中安装 Ruby 环境）：
```
sudo gem install cocoapods
```

**Q: TUICalling是否支持后台运行？**

支持，如需要进入后台仍然运行相关功能，可选中当前工程项目，在 **Capabilities** 下的设置  **Background Modes** 打开为 **ON**，并勾选 **Audio，AirPlay and Picture in Picture** ，如下图所示：
![](https://main.qcloudimg.com/raw/d960dfec88388936abce2d4cb77ac766.jpg)

>? 更多帮助信息，详见 [TUI 场景化解决方案常见问题](https://cloud.tencent.com/developer/article/1952880)，或者点击[这里](https://cloud.tencent.com/document/product/647/19906)联系我们。