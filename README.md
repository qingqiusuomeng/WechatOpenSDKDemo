我们在集成sdk前需要到微信开放平台注册自己的应用，然后拿到对应的key，其实这个一般都是后台来做，只要问后台拿到key就行

微信支付集成SDK介绍cocopods集成SDK和手动集成SDK

一、cocopods集成SDK

1.需要安装cocopods（安装及使用方法参照点击打开链接）

2、导入
pod 'WechatOpenSDK'
3.配置一下scheme（申请回来的key）

3.在AppDelegate.h中导入头文件，以及实现代理
#import <WechatOpenSDK/WXApi.h>//微信支付

@interface AppDelegate : UIResponder <UIApplicationDelegate,WXApiDelegate>
4.在AppDelegate.m中实现一下方法
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    // Override point for customization after application launch.
    [WXApi registerApp:@"wxd930ea5d5a258f4f"];//wxd930ea5d5a258f4f这串key是在微信开放平注册该应用后获得的，注册一般是后台去做的，叫后台把key给你即可
    return YES;
}

#pragma mark -- 重写AppDelegate的handleOpenURL和openURL方法：--

//9.0前的方法，为了适配低版本 保留
- (BOOL)application:(UIApplication *)application handleOpenURL:(NSURL *)url{
    return [WXApi handleOpenURL:url delegate:self];
}

- (BOOL)application:(UIApplication *)application openURL:(NSURL *)url sourceApplication:(NSString *)sourceApplication annotation:(id)annotation{
    return [WXApi handleOpenURL:url delegate:self];
}

//9.0后的方法
- (BOOL)application:(UIApplication *)app openURL:(NSURL *)url options:(NSDictionary<NSString *,id> *)options{
    //这里判断是否发起的请求为微信支付，如果是的话，用WXApi的方法调起微信客户端的支付页面（://pay 之前的那串字符串就是你的APPID，）
    return  [WXApi handleOpenURL:url delegate:self];
}

#pragma mark --你的程序要实现和微信终端交互的具体请求与回应，因此需要实现WXApiDelegate协议的两个方法：--
-(void) onReq:(BaseReq*)req{
    //onReq是微信终端向第三方程序发起请求，要求第三方程序响应。第三方程序响应完后必须调用sendRsp返回。在调用sendRsp返回时，会切回到微信终端程序界面。
    
}
-(void) onResp:(BaseResp*)resp{
    //如果第三方程序向微信发送了sendReq的请求，那么onResp会被回调。sendReq请求调用后，会切到微信终端程序界面。
    
    if([resp isKindOfClass:[PayResp class]])
    {
        PayResp*response=(PayResp*)resp;
        NSString *strMsg = @"支付失败";
        switch(response.errCode){
            case WXSuccess:
            {
                //服务器端查询支付通知或查询API返回的结果再提示成功
                NSLog(@"支付成功");
                strMsg = @"支付成功";
                break;
            }
            case WXErrCodeUserCancel:
            {
                NSLog(@"用户点击取消");
                strMsg = @"用户点击取消";
            }
                break;
            default:
                NSLog(@"支付失败，retcode=%d",resp.errCode);
                break;
        }
        
    }
}
5.在需要调用的地方进行调用

- (void)weixinPay{
    PayReq *req = [[PayReq alloc] init];
    //实际项目中这些参数都是通过网络请求后台得到的，详情见以下注释，测试的时候可以让后台将价格改为1分钱
    req.openID = @"appid";//微信开放平台审核通过的AppID
    req.partnerId = @"partnerid";//微信支付分配的商户ID
    req.prepayId = @"prepayid";// 预支付交易会话ID
    req.nonceStr =@"noncestr";//随机字符串
   // req.timeStamp = @"timestamp";//当前时间
    req.package = @"package";//固定值
    req.sign =@"sign";//签名，除了sign，剩下6个组合的再次签名字符串
    
    if ([WXApi isWXAppInstalled] == YES) {
        //此处会调用微信支付界面
        BOOL sss =   [WXApi sendReq:req];
        if (!sss ) {
           // [MBManager showMessage:@"微信sdk错误" inView:weakself.view afterDelayTime:2];
        }
    }else {
        //微信未安装
       // [MBManager showMessage:@"您没有安装微信" inView:weakself.view afterDelayTime:2];
    }
}


二、手动集成SDK

请到微信开放平台下载SDK。


下载微信SDK
建议把iOS头文件和支付示例都下载下来

1.导入上面那个iOS头文件和库下载下载出来的SDK包的就行，然后需要链接上依赖库，在Target —> BuildPhases —> Link Binary With Libraries— 点击+号 -> 搜索你需要的系统库。

SystemConfiguration.framework
libz.tbd
libsqlite3.0.tbd
CoreTelephony.framework
QuartzCore.framework

2.剩下的步骤参照cocopods第三步


