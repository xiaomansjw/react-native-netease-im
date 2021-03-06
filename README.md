
React Native的网易云信插件
欢迎加入QQ群交流`153174456`
## Demo
[react-native-chat-demo](https://github.com/reactnativecomponent/react-native-chat-demo)

#### 注意事项: 
##### 2.普通帐号不要使用5位数，因为5位数设定是系统帐号，尽量使用6位或者6位以上

## 如何安装

### 1.首先安装npm包

```bash
npm install react-native-netease-im --save 或者 yarn add react-native-netease-im
```

### 2.link
```bash
react-native link react-native-netease-im
```

#### 手动link~（如果不能够自动link）
##### ios
```
a.打开XCode's工程中, 右键点击Libraries文件夹 ➜ Add Files to <...>
b.去node_modules ➜ react-native-netease-im ➜ ios ➜ 选择 RNNeteaseIm.xcodeproj
c.在工程Build Phases ➜ Link Binary With Libraries中添加libRNNeteaseIm.a
```
##### Android
```
// file: android/settings.gradle
...

include ':react-native-netease-im'
project(':react-native-netease-im').projectDir = new File(settingsDir, '../node_modules/react-native-netease-im/android')
```

```
// file: android/app/build.gradle
...

dependencies {
    ...
    compile project(':react-native-netease-im')
}
```

`android/app/src/main/java/<你的包名>/MainActivity.java`

```
import com.netease.im.uikit.permission.MPermission;
import com.netease.im.RNNeteaseImModule;
import com.netease.im.ReceiverMsgParser;

public class MainActivity extends ReactActivity {

 ......

  @Override
     protected void onCreate(Bundle savedInstanceState) {
         super.onCreate(savedInstanceState);
         if(ReceiverMsgParser.checkOpen(getIntent())){//在后台时处理点击推送消息
             RNNeteaseImModule.launch = getIntent();
         }
     }

 @Override
 public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
        MPermission.onRequestPermissionsResult(this, requestCode, permissions, grantResults);
 }
 ```

`android/app/src/main/java/<你的包名>/MainApplication.java`中添加如下两行：

```java
...
import com.netease.im.RNNeteaseImPackage;  // 在public class MainApplication之前import
import com.netease.im.IMApplication;

public class MainApplication extends Application implements ReactApplication {

  private final ReactNativeHost mReactNativeHost = new ReactNativeHost(this) {
    @Override
    protected boolean getUseDeveloperSupport() {
      return BuildConfig.DEBUG;
    }

    @Override
    protected List<ReactPackage> getPackages() {
      return Arrays.<ReactPackage>asList(
          new RNNeteaseImPackage(), // 然后添加这一行
          new MainReactPackage()
      );
    }
  };

  @Override
  public ReactNativeHost getReactNativeHost() {
      return mReactNativeHost;
  }
   @Override
  public void onCreate() {
    //初始化方法appId以及appKey在小米开放平台获取，小米推送证书名称在网易云信后台设置
    IMApplication.setDebugAble(BuildConfig.DEBUG);
    IMApplication.init(this, MainActivity.class,R.drawable.ic_stat_notify_msg,new    IMApplication.MiPushConfig("小米推送证书名称","小米推送appId","小米推送的appKey"));
   ...
  }
}
```


### 3.工程配置
#### iOS配置
install with CocoaPods
```
pod 'NIMSDK', '6.2.0'
pod 'CocoaLumberjack', '~> 2.0.0-rc2'
```
Run `pod install`

在你工程的`AppDelegate.m`文件中添加如下代码：

```
...
#import <NIMSDK/NIMSDK.h>
#import "NTESSDKConfigDelegate.h"
@interface AppDelegate ()
@property (nonatomic,strong) NTESSDKConfigDelegate *sdkConfigDelegate;
@end
@implementation AppDelegate

- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
{
...
[self setupNIMSDK];
[self registerAPNs];
if (launchOptions) {//未启动时，点击推送消息
    NSDictionary * remoteNotification = [launchOptions objectForKey:UIApplicationLaunchOptionsRemoteNotificationKey];
    if (remoteNotification) {
      [self performSelector:@selector(clickSendObserve:) withObject:remoteNotification afterDelay:0.5];
    }
  }
...
return YES;
}
- (void)clickSendObserve:(NSDictionary *)dict{
  [[NSNotificationCenter defaultCenter]postNotificationName:@"ObservePushNotification" object:@{@"dict":dict,@"type":@"launch"}];
}
- (void)setupNIMSDK
{
//在注册 NIMSDK appKey 之前先进行配置信息的注册，如是否使用新路径,是否要忽略某些通知，是否需要多端同步未读数
self.sdkConfigDelegate = [[NTESSDKConfigDelegate alloc] init];
[[NIMSDKConfig sharedConfig] setDelegate:self.sdkConfigDelegate];
[[NIMSDKConfig sharedConfig] setShouldSyncUnreadCount:YES];
//appkey 是应用的标识，不同应用之间的数据（用户、消息、群组等）是完全隔离的。
//注册APP，请将 NIMSDKAppKey 换成您自己申请的App Key
[[NIMSDK sharedSDK] registerWithAppID:@"appkey" cerName:@"证书名称"];
}

#pragma mark - misc
- (void)registerAPNs
{
[[UIApplication sharedApplication] registerForRemoteNotifications];
UIUserNotificationType types = UIUserNotificationTypeBadge | UIUserNotificationTypeSound | UIUserNotificationTypeAlert;
UIUserNotificationSettings *settings = [UIUserNotificationSettings settingsForTypes:types categories:nil];
[[UIApplication sharedApplication] registerUserNotificationSettings:settings];
}

- (void)application:(UIApplication *)app didRegisterForRemoteNotificationsWithDeviceToken:(NSData *)deviceToken
{
  [[NIMSDK sharedSDK] updateApnsToken:deviceToken];
}
//在后台时处理点击推送消息
- (void)application:(UIApplication *)application didReceiveRemoteNotification:(NSDictionary *)userInfo{

  [[NSNotificationCenter defaultCenter]postNotificationName:@"ObservePushNotification" object:@{@"dict":userInfo,@"type":@"background"}];
}
- (void)application:(UIApplication *)app didFailToRegisterForRemoteNotificationsWithError:(NSError *)error
{
  NSLog(@"fail to get apns token :%@",error);
}
- (void)applicationDidEnterBackground:(UIApplication *)application {
  NSInteger count = [[[NIMSDK sharedSDK] conversationManager] allUnreadCount];
  [[UIApplication sharedApplication] setApplicationIconBadgeNumber:count];
}
```

#### Android配置
在`android/app/build.gradle`里，defaultConfig栏目下添加如下代码：
```
manifestPlaceholders = [
	// 如果有多项，每一项之间需要用逗号分隔
    NIM_KEY: "云信的APPID"    //在此修改云信APPID
]
```
## 如何使用
### 引入包

```
import {NimSession} from 'react-native-netease-im';
```

### API

参考[index.js](https://github.com/reactnativecomponent/react-native-netease-im/blob/master/index.js)

#### 监听会话
```
NativeAppEventEmitter.addListener("observeRecentContact",(data)=>{
  console.log(data); //返回会话列表和未读数
})；
```
#### 推送
```
//程序运行时获取的推送点击事件
NativeAppEventEmitter.addListener("observeLaunchPushEvent",(data)=>{
  console.log(data);
})；
//程序后台时获取的推送点击事件
NativeAppEventEmitter.addListener("observeBackgroundPushEvent",(data)=>{
  console.log(data); 
})；
//推送数据格式
{
    ...
    sessionBody：{
        sessionId:"",
        sessionType:"",
        sessionName:""
    }
}

```

