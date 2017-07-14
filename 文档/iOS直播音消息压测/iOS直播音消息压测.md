# iOS直播间消息压测

## 消息压测的意义
1. 通常开发或测试环境下，很难模拟真实直播场景中，IM消息暴发的情况，即无法较好地提前预知直播中存在的问题，避免线上出现问题；
2. 根据以往的经验：业务侧经常对消息的渲染，动画的展现上的性能问题关注不够，当IM消息量大的时候，性能问题就突现出来，对主播端的影响更明显。另外消息的展现这块又一般是在主线程中进行，这样经常会导致主线程抢占摄像头采集线程的CPU，导致直播中没有上行视频数据（主播端停留在最后一帧或黑屏），即导致所有观看端的“卡顿”（实际停留在最后一帧）。

## 如何模拟线上消息压测
### 消息压测原理
消息压测，即单步使用测试代码，向直播IM群内 （前提是要有IM群），频繁发送文本，点赞，礼物等IM消息；另外可以调整测试代码，指定发送频繁，以及不同消息的随机度；

### 压测前的准备功作

以下操作均可以新建测试工程（导入IMSDK即可），或者现在工程上添加测试入口；

1. 准备sdkappid、accounttype、 测试帐号userid、sig，用于登录 (如果在忆有App上集成，多已有登录，可跳过该步) 

```
    // InitSDK
    TIMManager *manager = [TIMManager sharedInstance];

    [manager setEnv:cfg.environment];
    [manager initLogSettings:YES logPath:[manager getLogPath]];
    [manager setLogLevel:TIM_LOG_DEBUG];
    [manager disableAutoReport];
    
    [manager initSdk:sdkappid accountType:accounttype];
    
    // 登录演示代码
    TIMLoginParam *param = [[TIMLoginParam alloc] init];
    param.appidAt3rd = sdkappid;
    param.sdkAppId = sdkappid;
    param.accountType = accounttype;
    param.identifier = userid;
    param.userSig = sig;
    
    [[TIMManager sharedInstance] login:param succ:succ fail:^(int code, NSString *msg) {
        
        DebugLog(@"TIMLogin Failed: code=%d err=%@", code, msg);
        if (code == kEachKickErrorCode)
        {
            //TODO：互踢重联，重新再登录一次
        }
        else
        {
            NSLog(@"login errcode = %d, errmsg = %@", code, msg);
        }
    }];


```

2. 直播IM群号groupId；

```
	// 当前测试号以观众身份加入到群
    [[TIMGroupManager sharedInstance] JoinGroup:groupId msg:nil succ:^{
        // 加群成功
    } fail:^(int code, NSString *string) {
        
        if (code == 10013)
        {
            // 已在直播群
        }
        else
        {
            NSLog(@"----->>>>>观众加入直播聊天室失败");
        }
        
    }];
    
    // 加入成功后，获取到群组会话
    _conversation = [[TIMManager sharedInstance] getConversation:TIM_GROUP receiver:groupId];

```

3. 准备测试代码：可以此方便中添加测试逻辑，以便控制发消息的随机性

```

- (void)onSendMsg
{
	// TODO:根据自己业务逻辑添加可被app解格的消息
	
	int bs = arc4random() % 2;
	if (bs)
	{
	 	// TODO: 添加文本消息txtmsg
		//    [_conversation sendMessage: txtmsg succ:^{
		//        NSLog(@"发送 %@ 成功", msg);
		//    } fail:^(int code, NSString *msg) {
		//        NSLog(@"发送消息失败");
		//    }];
	}
	
	
	bs = arc4random() % 2;
	if (bs)
	{
	 	// TODO: 添加点赞消息praisemsg
		//    [_conversation sendMessage: praisemsg succ:^{
		//        NSLog(@"发送 %@ 成功", msg);
		//    } fail:^(int code, NSString *msg) {
		//        NSLog(@"发送消息失败");
		//    }];
	}
	
	bs = arc4random() % 2;
	if (bs)
	{
	 	// TODO: 添加礼物消息giftmsg
		//    [_conversation sendMessage: giftmsg succ:^{
		//        NSLog(@"发送 %@ 成功", msg);
		//    } fail:^(int code, NSString *msg) {
		//        NSLog(@"发送消息失败");
		//    }];
	}
	
	// TODO:按业务测试要求添加要的消息类型
}

```



3. 启动定时器，指定发送频率fps(如20条/s)，向直播群内发消息:

```
	_sendTimer = [NSTimer scheduledTimerWithTimeInterval:1/fps target:self selector:@selector(onSendMsg) userInfo:nil repeats:YES];
	
```



## 直播间IM消息展现调优的一些建议

一般地，性能问题跟业务逻辑关联性很大，业务在上线之前要做好自己的性能问题，关于IM消息性能调优的一些建议
* 1.减少IM消息显示频率：将IM消息进行缓存以固定频率进行刷新（不要即时显示），低优先级消息可以丢弃；
* 2.同类型IM消息合并，减少不重要的消息的显示：比如进群消息并不重要，则可将连续收到的若干条进群消息进行合并，减少消息展现刷新的频繁；* 3.减少界面上的动画，增加动画缓存：测试过程中发现，频繁的动画（如直播中常见的飘星动画）的绘制渲染，都是在主线程，另外飘星动画本身的创建与销毁，也是在主线程中。所以尽量减少动画在显示频繁，可参考1,2两条，以及创建的次数，即缓存好动画资源；* 4.在AVSDK回调中进行刷新：减少消息或帧绘制在主线程切换的次数，减少交互UI卡顿；* 5.IM消息列表的计算以及渲染分开，将消息的计算放到子线程中，在只主线程中做渲染：这个属于基本的性能调优，可以百度谷歌相关说明* 6.界面上使用性能更好的代码：目前demo里面都是使用UITableView显示，业务侧可思考使用其他的方式进行显示；
* 7.多用Instrument查看消息压测时CPU以及内存的消耗，逐一排查相关消耗点；

## 集成ILiveSDK情况下，文本消息性能调优的建议

<font color=red>以下代码只是告诉对于IM文本消息的优化处理是如何实现，具体实现看业务逻辑，此处只作参考。因动画的实现方式差异较大，业务侧可以实现以下方式后，可参考上述的3, 7两条自己行调试</font>

1. 将IM消息的计算丢入到子线程（<font color=red>消息处理子线程优先级不要太高</font>）中进行，并按业务逻辑，进行缓存，合并，舍弃等:

* 如果只接入了ILiveSDK, 可以IMSDK的`TIMMessageListener`实现处中，作如下处理：

```
- (void)onNewMessage:(NSArray *)msgs
{
    [self performSelector:@selector(onHandleNewMessage:) onThread:消息处理的子线程 withObject:msgs waitUntilDone:NO];
}

- (void)onHandleNewMessage:(NSArray *)msgs
{
// TODO
// 1.将消息解析
// 2.将消息分类，按优先级进行缓存，合并，或舍弃
// 3.对于要在界面中显示的消息，如文本消息，计算其高度

}

```

* 如果接入了ILiveSDK,TILLiveSDK，则可在`ILVLiveIMListener`实现处中，作如下操作

```
/**
 * 文本消息回调
 * @param  msg  文本消息类
 */
- (void)onTextMessage:(ILVLiveTextMessage *)msg
{
   [self performSelector:@selector(onHandleNewMessage:) onThread:消息处理的子线程 withObject:msgs waitUntilDone:NO];
}
/**
 * 自定义消息回调
 * @param  msg 自定义消息类
 */
- (void)onCustomMessage:(ILVLiveCustomMessage *)msg
{
   [self performSelector:@selector(onHandleNewMessage:) onThread:消息处理的子线程 withObject:msgs waitUntilDone:NO];
}
/**
 * 其他消息回调
 * @param  msg TIMMessage
 */
- (void)onOtherMessage:(TIMMessage *)msg
{
   [self performSelector:@selector(onHandleNewMessage:) onThread:消息处理的子线程 withObject:msgs waitUntilDone:NO];
}

- (void)onHandleNewMessage:(NSArray *)msgs
{
// TODO
// 1.将消息解析
// 2.将消息分类，按优先级进行缓存，合并，或舍弃
// 3.对于要在界面中显示的消息，如文本消息，计算其高度

}

```

<font color=red>注意消息的计算已改到子线程，在主线程上直接渲染即可，无需再重计算</font>


2. 根据业务需要，主动调用`ILiveRoomManager`中下面的方法，将视频帧的渲染与IM消息的更新关联起来，减少消息频繁的显示

```
/**
 设置远程画面代理

 @param delegate 远程画面代理（主要用于预处理）
 */
- (void)setRemoteVideoDelegate:(id<QAVRemoteVideoDelegate>)delegate;

/**
 设置本地画面代理

 @param delegate 本地画面代理
 */
- (void)setLocalVideoDelegate:(id<QAVLocalVideoDelegate>)delegate;

/**
 设置屏幕画面代理

 @param delegate 屏幕画面代理
 */
- (void)setScreenVideoDelegate:(id<ILiveScreenVideoDelegate>)delegate;

/**
 设置播片画面回调，预留接口，暂时不建议使用

 @param delegate 播片代理
 */
- (void)setMediaVideoDelegate:(id<ILiveMediaVideoDelegate>)delegate;

```

并在相关的实现回调中作调下下面的方式，<font color=red>并渲染第1条中的处理后的消息</font>，具体举例如下：

```
- (void)OnLocalVideoPreview:(QAVVideoFrame*)frameData
{
	// 具体视频画面的渲染ILiveSDK内部已实现，此处可以直接用来更新界面上其他处的渲染
   [self  renderUIByAVSDK];
}

// TODO：为下一步更进一步控制频繁作准备
static BOOL kCanRenderUINow = NO;
- (void)renderUIByAVSDK
{
	// TODO : 可通过此处的控制显示的频率
    if (kCanRenderUINow/*TODO：加上自身的判断条件*/)
    {
        // TODO：根据前面的建议作渲染逻辑
        [self refreshIMMsgs];
    }
}

- (void)refreshIMMsgs
{
// TODO：业务侧根据自身逻辑
// 渲染第1条中的处理后的消息
}

```

3. 配置定时器频定，进行指定频繁刷新IM消息：当然业务侧也可以根据单位时间内的消息量来开启定时器，也可以进入直播间后立即开启，具体看业务逻辑

```

- (void)startRenderTimer
{
    if (!_renderTimer)
    {
        _renderTimer = [NSTimer scheduledTimerWithTimeInterval:2 target:self selector:@selector(onRefreshUI) userInfo:nil repeats:YES];
        [[NSRunLoop currentRunLoop] addTimer:_renderTimer forMode:NSRunLoopCommonModes];
    }
}

- (void)onRefreshUI
{
    if (kCanRenderUINow)
    {
        // 防止画面不显示时，界面无法刷新消息
        [self renderUIByAVSDK];
        kCanRenderUINow = NO;
    }
    else
    {
        kCanRenderUINow = YES;
    }
}

- (void)stopRenderTimer
{
    [_renderTimer invalidate];
    _renderTimer = nil;
}

```

4. 完成以上步骤后，根据业务逻辑，并结合压侧以及Instrument工具，检查渲染性能，逐一进行调优，此处多为业务逻辑，不再举例；





