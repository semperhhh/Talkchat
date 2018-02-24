# Talkchat集成文档

> 具备了iOS应用开发的基础经验,能够理解相关基础概念

### 1. SDK工作流程

![屏幕快照 2017-09-13 下午2.04.49](http://olnx80yq4.bkt.clouddn.com/%E6%8E%A8%E9%80%81%E6%B5%81%E7%A8%8B%E5%9B%BE.png)

* 如果开发者需要推送服务,请在talk99后台中设置推送的app相关资料,申请相应的talk99SDK需要的appkey.
* 如果开发者需要用到talk99服务端做推送消息,需要将app对应的apns推送证书上传到talk99后台,推送证书相关详情[苹果开发者](https://developer.apple.com/)

## 文件介绍

> talkSDK.framework中的各个头文件介绍

| .h                              | 介绍               |
| :------------------------------ | ---------------- |
| TalkChatHeader.h                | SDK中宏            |
| TalkChatManager.h               | SDK提供的初始化工具类     |
| TalkChatMessageManager.h        | SDK提供的聊天逻辑API    |
| TalkChatSaveManager.h           | SDK提供的聊天数据持久化API |
| TalkChatWebSocketManager.h      | SDK提供的网络消息API    |
| TalkChatMessageViewController.h | SDK提供的聊天界面       |
| talk99_chatBar.bundle           | SDK需要用到的图片资源     |
| face.bundle                     | SDK需要用到的表情资源     |

### 2.导入talk99_SDK

objective-c项目:导入talk99_sdk文件夹中对应的.framework静态包,拷贝在工程路径下面,子啊工程目录结构中.

swift项目:暂不支持

#### 引入依赖库

talk99_SDK的实现,依赖了一些系统框架和一些常用的第三方框架,开发者首先添加系统框架依赖,点击工程右边的工程名,然后在工程名右边依次选择 *TARGETS* -> *BuiLd Phases* -> *Link Binary With Libraries*，展开 *LinkBinary With Libraries* 后点击展开后下面的 *+* 来添加下面的依赖项:

- AVFoundation.framework

  然后添加第三方框架,开发者可以使用CocoaPods导入,也可以通过下载拷贝到工程路径中

- Masonry

- SocketRocket

- FMDB

  如果工程中已经依赖这些第三方框架,则可以忽略不计,后续会逐渐减少第三方库的使用

### 3.快速集成SDK

talk99_SDK封装了一套简单易用的聊天界面,帮助开发者快速生成聊天视图,并提供自定义接口,满足一定定制需求.

*所有的操作都必须在初始化SDK,并且talk99服务端返回可用的appId后才可以正常运行

~~~~objective-c
//在AppDelegate.m增加如下 SDK 设置
//#import <talkchatSDK/TalkChatManager.h>

- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    [TalkChatManager initWithAppId:@"talk99后台配置的appId" appSecret:@"talk99后台配置的appSecret" compId:@"talk99提供的公司id" appVersion:@"开发者app版本" appViewerId:@"开发者app内用户id" customize:nil completion:^(NSError *error) {
        
        NSLog(@"init error - %@",error);//错误码
    }];
return YES;
}

//在开发者需要调出聊天界面的位置，增加如下代码
pushviewcontroller
~~~~

deviceToken:在使用推送功能时需要开发者提供deviceToken,将下列代码添加到到 `AppDelegate.m` 中系统回调 `didRegisterForRemoteNotificationsWithDeviceToken` 中：

~~~objective-c
//注册deviceToken代码
[TalkChatManager registerDeviceToken:[@"deviceToken"dataUsingEncoding:NSUTF8StringEncoding]];
~~~

> ## 建立聊天界面

在初始化后建立聊天界面

~~~~objective-c
//建立聊天界面
#import <talkchatSDK/TalkChatManager.h>
#import <talkchatSDK/TalkChatMessageViewController.h>

    TalkChatMessageViewController *messageVC = [[TalkChatMessageViewController alloc]init];
	messageVC.imageName	//自定义访客头像
    messageVC.groupId	//指定客服分组
    messageVC.userID	//指定客服的userId
    //(groupId和userID最少传入一个,否则将接入机器人自助答疑)
    [self.navigationController pushViewController:messageVC animated:YES];
~~~~

调用拍照、选取照片和录音功能的时候,需要开发者在项目的plist文件中添加两个key,否则应用可能会crash

````
Privacy - Photo Library Additions Usage Description
Privacy - Photo Library Usage Description
Privacy - Camera Usage Description
Privacy - Microphone Usage Description
````

如果不添加这两个在用户点击时会crash

> #### 自定义UI

开发者自定义UI的情况下,可以调用SDK中提供的逻辑接口,实现用户和客服之间的消息对话,也可以实现部分自定义

~~~~objective-c
//逻辑接口内容
//1.修改客服头像.TalkChatMessageViewController的imageName可以用于修改客服头像,开发者只需要将图片放入工程中,在初始化界面时将图片名称set到imageName
TalkChatMessageViewController *messageVC = [[TalkChatMessageViewController alloc]init];
messageVC.imageName = @"test";
//2.导航栏 - 开发者可以选择push和present聊天控制器TalkChatMessageViewController,present到的聊天界面可以选择是否添加导航栏并且改变导航栏的颜色,例:
TalkChatNavigationController *navi = [[TalkChatNavigationController alloc]initWithRootViewController:messageVC navigationColor:[UIColor redColor]];
//如果在开发者的控制器push到的聊天控制器则忽略
//3.在初始化TalkChatMessageViewController时有这些可配选项
/**
 导航栏标题(可选,默认客服昵称)
 */
@property (nonatomic,strong)NSString *naviTitle;
/**
 是否在导航栏右边添加清除缓存按钮(默认不添加)
 */
@property (nonatomic,assign,getter=isClearCache)BOOL clearCache;
/**
 访客发送消息文字字体颜色
 */
@property (nonatomic,strong)UIColor *customTextColor;
~~~~

> 添加开场白

开发者可以选择在访客进入对话界面的时候是否向访客发送一条开场白(问候语)

~~~~objective-c
/**	TalkChatMessageViewController.m
 添加问候语
 */
@property (nonatomic,assign,getter=isAddGreetings)BOOL addGreetings;
/**
 问候语内容
 */
@property (nonatomic,strong)NSString *greetingsContent;
//两个需要同时配置,和自动回复不冲突(默认不添加)
~~~~



## 消息推送

> 点击项目---->`TARGET`---->`Capabilities`，将这里的`Push Notification`的开关打开

~~~~objective-c
//在appDelegate.m的- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions中添加代码:根据系统版本判断推送消息类型和app是否在前台工作判断推送
    if ([[UIDevice currentDevice].systemVersion floatValue] >= 10.0) {
        UNUserNotificationCenter *center = [UNUserNotificationCenter currentNotificationCenter];
        center.delegate = self;
        [center requestAuthorizationWithOptions:(UNAuthorizationOptionAlert | UNAuthorizationOptionBadge | UNAuthorizationOptionSound) completionHandler:^(BOOL granted, NSError * _Nullable error) {
            if (granted) {
                //点击允许
                NSLog(@"注册成功");
                [center getNotificationSettingsWithCompletionHandler:^(UNNotificationSettings * _Nonnull settings) {
                    NSLog(@"settings = %@",settings);
                }];
            }else {
                //点击不允许
                NSLog(@"注册失败");
            }
            
        }];
    }else if ([[UIDevice currentDevice].systemVersion floatValue] >=8.0) {
        //iOS8 - iOS10
        [application registerUserNotificationSettings:[UIUserNotificationSettings settingsForTypes:UIUserNotificationTypeAlert | UIUserNotificationTypeSound | UIUserNotificationTypeBadge categories:nil]];
    }else if ([[UIDevice currentDevice].systemVersion floatValue] < 8.0) {
        //iOS8系统以下
        [application registerForRemoteNotificationTypes:UIRemoteNotificationTypeBadge | UIRemoteNotificationTypeAlert | UIRemoteNotificationTypeSound];
    }
~~~~



###talk99_SDK逻辑接口介绍

#### 初始化

> 所有操作都需要在初始化SDK后进行

~~~~objective-c
//初始化方法
  /** TalkChatManager.h
 初始化,开发者在AppDelegate中调用这个方法完成talkchat_sdk的初始化操作,上传相关参数
 @param appId talk99提供的appId
 @param appSecret talk99提供的appsecret
 @param compId talk99提供的公司id
 @param appVersion 开发者app版本
 @param appViewerId 开发者app内用户id
 @param customizeDictionary 开发者自定义访客信息,所有key和value都为字符串
 @param completion 初始化回调,如果返回error 则初始化失败 可能会有网络问题引起的初始化失败
 */
+(void)initWithAppId:(NSString *)appId appSecret:(NSString *)appSecret compId:(NSString *)compId appVersion:(NSString *)appVersion appViewerId:(NSString *)appViewerId customize:(NSDictionary *)customizeDictionary completion:(void (^)(NSError *error))completion;
//如果开发者自定义UI则需要注意返回内容,返回:
 {
        id:100,//req id
        type:'resp',
        err:0, //错误码        
        data:{
            pushType:-1,//推送方式（-1：未配置或者自定义推送，0：ios）
            server:'', //特定的服务地址,用于后面的创建对话等操作的前缀
        }        
    }
~~~~

#### 心跳消息

> sdk会定时向服务器发送心跳包来保证用户的状态,建议每10秒发一次

~~~~objective-c
//发送 TalkChatMessageManager.h
/**
 发送心跳消息
 */
+(void)requestWithSocketHeartbeat;
////如果开发者自定义UI则需要注意返回内容,返回对应id的数据
 {
    id:100,
    type:'resp',      
    err:0  
}
~~~~



#### 创建对话

~~~~objective-c
/**
 创建对话  -创建和客服或自助答疑的对话,开发者可以根据返回的内容判断创建对话的对象

 @param groupId 目标分组
 @param userId 目标客服ID,目标分组和目标客服至少存在一个
 @param probeId 使用探头ID
 @param createBlock 创建状态回调 mode 客服区分 */
+ (void)createMessageWithGroupId:(int)groupId
                          UserId:(NSString *)userId
                         ProbeId:(int)probeId
                     CreateBlock:(void(^)(BOOL success, NSDictionary *messageDic,int mode))createBlock;

////如果开发者自定义UI则需要注意返回内容,返回
    {
        id:100,
        type:'resp',     
        err:0,   
        data:{          
            mode:0,//0：对话，1：自助答疑 
            // 为0有效
            userId: '' //用户名
            userName: '' //外部名字
            photo:'' //头像 
            email:'',
            mobile:'',
            status:0, //客服状态，0：离线或忙碌，1：在线
            fromType:0, 
            fromAppId:'',//仅当fromType=100时有效
            /为1有效
            robotName:'',
            robotPhoto:'',
            robotStupid:''
        }
    }
~~~~

#### 修改app用户自定义信息

~~~~objective-c
/** TalkChatManager实例类
 修改app用户自定义信息

 @param dictionary 要修改的键值字典 自定义访客信息,所有key和value都为字符串
 'name':'value',..  
 */
+(void)changeAppUserInfoWithDictionary:(NSDictionary *)dictionary completion:(void (^)(NSError *))completion;
//服务端返回对应id的消息
~~~~



#### 发送消息

> #### 文本消息

~~~~objective-c
/**TalkChatMessageManager.h实例类
 发送文字消息
 这个文字消息会根据对话创建时的(人工/自助)属性去判断具体发送对象
 如果不需要这个判断可以调用下面的方法

 @param content 文字内容
 @param completion 回调,可能含有消息发送的状态
 */
+(void)sendTextMessageWithContent:(NSString *)content completion:(void (^)(NSError *error))completion;

/**
 发送文字消息(客服对话)

 @param content 文字内容
 @param completion 回调
 */
+(void)sendDialogueTextMessageWithContent:(NSString *)content completion:(void (^)(NSError *))completion;

/**
 发送文字消息(自助)

 @param content 文字内容
 @param completion 回调
 */
+(void)sendAutoTextMessageWithContent:(NSString *)content completion:(void (^)(NSError *))completion;
~~~~

> #### 图片消息

~~~~objective-c
//发送
/**
 发送图片消息

 @param image 图片
 @param completion 回调,可能含有消息发送的状态
 */
+(void)sendImageMessageWithImage:(UIImage *)image completion:(void (^)(NSError *error))completion;
~~~~

> #### 语音消息

~~~~objective-c
//发送
/**
 发送语音消息

 @param voiceData 语音
 @param completion 回调,可能含有消息发送的状态
 */
+(void)sendVoiceMessageWithVoice:(NSData *)voiceData completion:(void (^)(NSError *error))completion;
~~~~

#### 接收消息

~~~~objective-c
//接收消息的是TalkChatWebSocketManager实例类,发送消息通知的是TalkChatMessageManager实例类
### 在合适的地方监听有新消息的广播
[[NSNotificationCenter defaultCenter]addObserver:self selector:@selector(receiveMessageWithNotification:) name:talkChatMessageReceiveNotification object:nil];

### 监听收到聊天消息的广播
-(void)receiveMessageWithNotification:(NSNotification *)tifiy {
    
    NSLog(@"userinfo = %@",tifiy.userInfo);
    NSDictionary *dictionary = tifiy.userInfo;
}

//接收到对话消息格式,开发者可以根据需要得到对话的message内容
  {
      id:100,//消息ID
      type:'chat",
      data:{
          from:'' //发送者        
          category:0//0：普通消息，1：图片（接受png,gif,jpeg,jpg），2：语音（接受mp3,amr(可为silk v3)）,3:视频（接受h264编码，mp4后缀），4:附件
          message:''
          fromType:0,//发送者身份
          time:1111111 //发送时间戳
      }                                                  
    }

////如果开发者自定义UI则需要注意返回内容,接收到自助答疑响应格式
    {
        id:100,
        type:'resp',   
        err:0,     
        data:{      
            mode:0, //0：转人工客服，1:自助回答
            //0有效      
            answers:[
                {
                    id:1,
                    question:'',
                    answer:'',
                    files:[
                        {
                            name:'',
                            url：''
                        }...                    
                    ]
                }...
            ]
            //0有效
            {
                //见创建对话回复
            }
        }
    }
~~~~

#### 消息确认

> 消息确认机制用于确定消息是否成功接收到,通过建立的消息通道发送出的消息需要确认

~~~~objective-c
/** 
 发送确认消息  TalkChatMessageManager.h

 @param idString 消息id
 */
+(void)requestWithMessageConfirmWithIdString:(NSString *)idString;
~~~~



#### 事件

事件类消息主要为对话关闭、对话转移、正在输入及访客接入对话状态

~~~~objective-c
//接收事件类消息
~~~~

#### 通知

通知类消息分为系统类通知和内部消息.系统类通知，有我们公司发送，一般为系统维护类消息，以及日报.内部消息，一般有客户公司发送，一般为订单，支付，代办等消息

~~~~objective-c
//接收通知类消息
~~~~

> 接收到的消息可以通过talk99服务端进行app消息推送,请按申请步骤在talk99后台进行配置[talk99后台配置](#peizhi)

> 消息可以在app内部展示,展示样式需要注册才可以使用

~~~~objective-c
//注册代码
~~~~

### 客户离线

> 客户离线注销,不会再收到消息和推送

````objective-c
//注销代码
````



### 消息历史记录

> SDK通过数据库表来保存消息,需要导入FMDB

~~~~objective-c
//需要用到消息保存时可以在跳转控制器时插入代码 TalkChatSaveManager.h
/**
 打开数据库(添加历史记录必要操作,在初始化TalkChatMessageController前设置)

 @param userName 新登陆用户昵称
 */
- (void)openDBWithUserName:(NSString *)userName;

//一些数据持久化的接口,这些是基于创建上面的数据表的方法
/**
 插入操作

 @param model 模型
 @param successBlock 成功回调
 */
- (void)insertChatDetailTableWithModel:(TCMessageModel *)model SuccessBlock:(void(^)(BOOL success,NSString * uid))successBlock;

/**
 更新发送状态

 @param detailUid 消息唯一id
 @param sendSuccess 消息成功状态
 @param block 成功回调
 */
- (void)updateMessageSendSuccessByDetailUid:(NSString *)detailUid SendSuccess:(NSInteger)sendSuccess SuccessBlcok:(void(^)(BOOL success))block;

/**
 更新已读状态

 @param detailUid 消息唯一id
 @param block 成功回调
 */
- (void)updateIsReadInMessageDetailTabByDetailUid:(NSString *)detailUid SuccessBlcok:(void(^)(BOOL success))block;

/**
 获取聊天记录

 @param from 消息来源
 @param loadNum 加载页数
 @param block 成功回调
 */
- (void)queryChatDetailByFrom:(NSString *)from LoadNum:(NSInteger)loadNum ResultBlock:(void(^)(NSMutableArray *))block;

/**
 删除某对话全部消息

 @param from 消息来源
 @param block 成功回调
 */
- (void)deleteAllMessageDetailByFrom:(NSString *)from SuccessBlcok:(void(^)(BOOL success))block;
~~~~



### 初始化时返回错误码说明

| 错误码  | 说明               |
| ---- | ---------------- |
| 0    | 调用完成             |
| 1001 | 参数错误             |
| 1002 | 错误的公司ID          |
| 1003 | 该公司已经停止服务        |
| 1004 | 错误的app           |
| 1005 | token校验失败        |
| 1006 | 访客被屏蔽            |
| 1007 | 发送的消息为空          |
| 1008 | 发送的消息过长          |
| 1009 | 其他错误             |
| 2000 | 留言失败，token失效     |
| 2001 | 留言失败，缺少联系方式      |
| 2002 | 留言失败，名字过长        |
| 2003 | 留言失败，电话过长或者错误的格式 |
| 2004 | 留言失败，邮件过长或者错误的格式 |
| 2005 | 留言失败，QQ过长或者错误的格式 |
| 2006 | 留言失败，weixin过长    |
| 2007 | 留言失败，内容过长        |
| 2008 | 留言失败，自定义内容过长     |

### <span id="peizhi">talk99后台配置</span>

>talk99配置地址
