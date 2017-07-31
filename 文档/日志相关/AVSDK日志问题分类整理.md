
# <a name="log_basic">基本流程</a>
## <a name="log_basic_login">登录不成功</a>
1. 检查相应的登录接口参数（sdkappid, accouttype, id, sig）是否有效；
2. 询问用户的集成方式（[托管模式](https://www.qcloud.com/document/product/268/7655)，[独立模式](https://www.qcloud.com/document/product/268/7654)），确认用户在集成方式上没有问题；
3. 引导用户阅读[互动直播->开发前必看](https://www.qcloud.com/document/product/268/7540)

## <a name="log_basic_startcontext">开起上下文不成功</a>
1. 代码配置是否正常，尤其是iOS，没有配置`other link flag`为`-ObjC`时，经常会报错；
2. 询问用户，或查看IMSDK日志，查看当前用户是否登录成功；
3. 阅读<a href="AVSDK日志分析_基础流程.md?#log_startcontext">音视频上下文</a>相关排查步骤
4. 过程是是否有切换IM帐号，但是音频视上下文还是用之前的帐号去初始化；

## <a name="log_basic_enterroom">进房不成功</a>
1. 相关时间点上一次退房是否成功；
1. 根据用户反馈的错误码（[SDK相关错误码](https://www.qcloud.com/document/product/268/8423)，<a href="10000以上被转换的错误码.xls">Server相关错误码</a>），判断是否是代码逻辑问题；具体方法可查询<a href="AVSDK日志分析_基础流程.md?#log_enterroom">进房流程</a>；
2. 询问用户网络情况，是否是所有网络不成功，还是部分网络问题；如果是部分网络问题，具体引导用户，拿到<a href="AVSDK日志分析_基础流程.md?#log_enterroom_ipaddress">接口机列表</a>，使用`ping`，或者[华佗工具](http://ping.huatuo.qq.com/)初步确认用户的网络情况；
3. 如果是IPV6环境，可阅读<a href="AVSDK日志分析_基础流程.md?#log_enterroom_ipaddress_problems">接口机相关常见问题</a>；
4. 如果用户是私有化环境，可询问用户是否需要进行私有化部署，如同意可走<a href="#log_workorder_private">私有化工单流程</a>;

## <a name="log_basic_exitroom">退房不成功</a>
1. 退房不成功原因一般是：当前不在房间内，进行退房；
2. 具体可查<a href="AVSDK日志分析_基础流程.md?#log_exitroom">退房流程</a>

## <a name="log_basic_exitroom">切换房间不成功</a>
1. 切换房间前提：当前必须在音视频房间中；
2. 可看详细的错误码，引导用户正确接入：2001时，说明还停留在上一个房间，2002说明已不在作何房间，相当于退出房间了；具体可查[SDK相关错误码](https://www.qcloud.com/document/product/268/8423)

```
    QAV_ERR_NOT_TRY_NEW_ROOM        = 2001, ///< 没有尝试进入新房间，将停留在旧房间    QAV_ERR_TRY_NEW_ROOM_FAILED     = 2002, ///< 尝试进入新房间，但失败了，旧房间也将关闭
```

## <a name="log_param_changeavcontrolrole">切换角色不成功</a>
1. 通过<a name="AVSDK日志分析_非常用流程.md?#log_changecontrolrole">切换角色/上下麦</a>查到具体的错误提示信息；
2. 通过<a href="AVSDK日志分析_基础流程.md?#log_getspearinfo">拉取spear配置</a>处，查到用户的具体的spear信息，是否有对应的角色，以及角色类型配置；
3. 询问用户拉取spear配置时（一般在登录的时候）的网络状况，有可能spear配置没有拉成功；




# <a name="log_audio">音频类问题</a>

### <a name="log_noaudio">没有声音</a>
1. 拿到上行用户的ID，用[AVMonitor](http://avq.server.com/reportapp/)查看该用户的`上行音频码率`是否正常，是否全为0；或者查## <a href="AVSDK日志分析_非常用流程.md?#log_avdata">数据流变化</a>中的对应用户的音频流变化
2. 如果有日志，检查有问题用户进房间时的<a href="AVSDK日志分析_基础流程.md?#log_enterroom">进房流程</a>的`auth_bits `参数是否有上行音频的权限；
3. 建议用户检查手机是否有开启静音，或勿扰模式；或者是否有其他设备在使用mic；
4. 代码中是否有其他音频相关的库，导致与AVSDK使用上发生冲突；
5. 代码中是否有调其他音频相关系统API，导致使用上发生冲突；
6. 切换成上麦角色之后，是否有<a href="AVSDK日志分析_非常用流程.md?#log_mic">打开Mic操作</a>


### <a name="log_audio_stuck">声音质量有问题，如有滋滋声，或一卡一卡之的声音，声音不连续</a>
1. 拿到上行用户的ID，用[AVMonitor](http://avq.server.com/reportapp/)查看该用户的`上行音频码率`是否波动较大，毛刺较多，或者相关时间上，是否有丢包存在；
2. 如果有日志，检查有问题用户进房间时的<a href="AVSDK日志分析_基础流程.md?#log_enterroom">进房流程</a>的`auth_bits `参数是否有上行音频的权限；
3. 代码中是否有其他音频相关的库，在链接时出出现了冲突（能正常运行），但未使用互动直播时，其他模块就出现问题，1.8.5.122之后修复编译链接导致音频出错；
3. 代码中是否有其他音频相关的库，同时采集导致，导致上行的音频有问题；
4. 可<a name="AVSDK日志分析_基础流程.md?#log_dumpaudio">dump上行音频</a>，给音频组ahuang看下，让其定位下具体的问题；
5. 检查代码中实际使用的音频编解码格式是否有问题（互动直播下为aac格式，实时通信下用的silk格式）：多发生成<a href="AVSDK日志分析_基础流程.md?#log_enterroom">进房流程</a>未设置角色（导致使用的音频编解码器不一致导致）；
6. 咨询用户是否有集成音频透传，或变声等功能；


# <a name="log_video">视频类问题</a>

## <a name="log_video_requestviewlist">请求不到画面</a>
1. 检查有问题用户进房间时的<a href="AVSDK日志分析_基础流程.md?#log_enterroom">进房流程</a>的`video_recv_mode`, `screen_recv_mode` 的配置，是否均为半自动接收（1），以及`auth_bits `是否有下行视频，辅流的权限；
2. 查看<a name="AVSDK日志分析_基础流程.md?#log_eventid">用户事件</a>，查看<a href="AVSDK日志分析_基础流程.md?#log_requestviewlist">请求画面</a>时刻是否有参数问题；
3. 查看下[AVMonitor](http://avq.server.com/reportapp/)，查对应画面上行者在基本参数（sdkappid, 房间号）是否一致，在相应时间段内是否有获取到视频位，以及反馈的时间点上行用户是否真的有上行视频；

## <a name="log_video_stuck">视频卡住不动（表现上是黑屏或者卡在最后一帧），或不连续</a>
1. 查看<a name="AVSDK日志分析_基础流程.md?#log_eventid">用户事件</a>，对应的时间点是否有关摄像头，桌面分享，或播片等事件，或者是否有<a href="AVSDK日志分析_基础流程.md?#log_exitroom">退房流程</a>，<a href="AVSDK日志分析_非常用流程.md?#log_changecontrolrole">下麦事件</a>；
2. 用户是否有强制退出或者退后台挂起等行为，相应时间点上，看日志时间是否有中断；
3. 在[AVMonitor](http://avq.server.com/reportapp/)上查看上行用户的`上行视频码率`是否中断，或者有`视频位移除事件` ，丢包率过高，以及`应用的cpu使用率`是否在相关时间段出现过高的情况；
4. 如果是部分用户观看卡住不动，但所有数据又都正常，可询问用户的渲染处理逻辑， 看是否<a href="AVSDK日志分析_非常用流程.md?#log_avdata">数据回调</a>正常，但是业务层没有正常渲染；
5. 如果画面很卡，不连续：在[AVMonitor](http://avq.server.com/reportapp/)上查看下行用户的`下行视频码率`毛刺是否严重，或丢包过高（多为收到视频帧后解码失败，导致无法正解码出视频帧以供渲染）；
6. 业务层渲染机制是否有问题（多为自定义渲染逻辑不正确认导致），或者业务上是否有把多个画面渲染到同一个区域或渲染控件上；
7. 确认用户的网络：可以使用[华佗工具](http://ping.huatuo.qq.com/)看下用户的具体网络情况；或者其他测速工具看下用户的带宽是否符合要求；

## <a name="log_video_badframe">出现花屏，绿屏，红屏，灰屏等</a>
1. 确认业务侧是否使用的SDK推荐的渲染，以及相关的视频格式配置是否正确认，尤其是自定义采集时：此种多为灰屏情况。
2. 如果问题某些机型上必现（多为android）：可询问机型，并对比[硬件编解码名单](http://tapd.oa.com/mobile_av/markdown_wikis/#硬件编解码名单)以及spear上对应进房角色的分辨率配置（960*540分辨率经常会出问题绿条），一般特定分辨率下花屏，多为机型硬编解兼容性问题，可联系erikge适当关闭对应的机型。
3. 是否是短暂性花屏：可查看[AVMonitor](http://avq.server.com/reportapp/)上对应下行视频码率变化，以及丢包情况，并同时查看日志中是否有视频帧是否解码失败;花屏一般是数据值被破坏或丢失，从日志无法定位，主播端可咨询用户是否使用了美颜相关的功能，观看端花屏，除主播上行数据有问题之外，还可能是丢包导致视频帧不完整；
4. 确认用户添加渲染的方式：之前出现过一例，用户在子线程中添加渲染控件，导致绘出来的画面是花屏；
5. 支持过程，引导用户使用不同的机型进行对比测试，如果所有机型上者有问题，一般是上行用户侧出现问题，如果是个别手机上出现，可再咨询用户具体使用方式；
6. 确认上行画面用户侧是否使用的美颜美白等，看是否是因为美颜美白导致画面出现问题；
7. 再能重现的情况下，尽量让用户录一个视频，把提供相应的日志，以便开发进一步分析；


## <a name="log_video_rotate">视频转置相关的问题</a>
1. 具体先让用户录个视频描述清楚其需求；
2. 如果是想要画面角度的，主要是老用户，可参考转置[功能升级指南](https://github.com/zhaoyang21cn/suixinbo_doc/blob/master/doc2/AVSDK%201.8.4%E8%BD%AC%E7%BD%AE%E5%8A%9F%E8%83%BD%E5%8D%87%E7%BA%A7%E6%8C%87%E5%8D%97.md);
3. 如果想自定义控制渲染画面方向类的，可参考：[如何旋转和裁剪画面](https://www.qcloud.com/document/product/268/7647)；



# <a name="log_avsync">音视频不同步的问题</a>

1. 先确认用户的基本信息：应用场景（实时通信，互动直播），用户的网络情况（[华佗工具](http://ping.huatuo.qq.com/)），以及用户所在的地理位置，确认用户使用的版本（互动直频直播下，1.8.4之前的版本有问题）；
2. 在[AVMonitor](http://avq.server.com/reportapp/)上查看有问题用户的：丢包率，延时是否有明显的毛刺；
3. 拿到用户题用户的日志查看<a href="AVSDK日志分析_非常用流程.md?#log_net">网络包情况</a>，以及相应的音视频帧是否有大量的解码失败的日志；具体可以找mortyzhu帮看下； 







<!--# <a name="log_device">设备操作相关</a>
## <a name="log_device_mic">Mic相关问题</a>



### <a name="log_device_camera">Camera相关问题</a>

### <a name="log_device_speaker">Speaker相关问题</a>

### <a name="log_device_aux">辅路相关的问题</a>-->

<!--## <a name="log_param">参数配置类问题</a>
### <a name="log_param_sdkappid">SDKAppID</a>

### <a name="log_param_spear">Spear配置信息</a>
### <a name="log_param_enterroom">进房参数配置</a>


## <a name="log_updownmic">互动连麦类</a>
### <a name="log_updownmic_updonwparam">上麦下麦问题</a>
#### 切换角色不成功
1. 通过<a name="AVSDK日志分析_非常用流程.md?#log_changecontrolrole">切换角色/上下麦</a>
1. 检查用户的spear配置中是否有对应

### <a name="log_updownmic_video">连麦过程中视频有问题</a>
### <a name="log_updownmic_audio">连麦过程中音频有问题</a>-->



<!--

## <a name="log_audio">音频类问题</a>
### <a name="log_audio_noaudio">没有声音</a>
### <a name="log_audio_stuck">声音卡顿</a>
### <a name="log_audio_quality">声音有问题</a>

## <a name="log_video">视频相关的问题</a>
### <a name="log_video_noframe">无视频画面，黑屏</a>
### <a name="log_video_badframe">花屏，绿屏，红屏</a>
### <a name="log_video_stuck">视频画面卡顿</a>
### <a name="log_video_render">渲染相关的crash</a>


## <a name="log_avsync">音视频不同步的问题</a>



## <a name="log_performance">性能相关的问题</a>

## <a name="log_compatibility">新老版本兼容性问题</a>

## <a name="log_crash">crash相关的问题</a>

## <a name="log_debug">需要AVSDK排查的问题</a>

## <a name="log_workorder">工单相关流程</a>
### <a name="log_workorder_livecode">直播码相关</a>
### <a name="log_workorder_rotate">转置相关</a>
### <a name="log_workorder_private">私有化</a>-->







