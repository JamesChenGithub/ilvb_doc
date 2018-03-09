<!--AVSDK日志分析_非常用流程.md-->
# AVSDK日志查询技巧

## <a name="log_changecontrolrole">切换角色/上下麦</a>
1. 作用：用于检查用户切换角色是否生效
2. 关键字：`ChangeAVControlRole`
3. 示例日志：

```
2017/06/09 17:30:42.231| E| 28238| Client | av_room_multi_impl.cpp(1421):ChangeAVControlRole  | ChangeAVControlRole LiveGuest
2017/06/09 17:30:42.232| E| 28238| Client | av_room_multi_impl.cpp(1468):ChangeAVControlRole  | roleInfo_3
2017/06/09 17:30:42.232| E| 28238| Client | av_room_multi_impl.cpp(1469):ChangeAVControlRole  | audioScheme_1
2017/06/09 17:30:42.316| E| 28238| Client | av_room_multi_impl.cpp(1497):ChangeAVControlRoleOn| av_control_role LiveGuest
2017/06/09 17:30:42.339| E| 28238| Client | av_room_multi_impl.cpp(4046):OnChangeAVControlRole| ******OnChangeAVControlRoleOnlyComplete. eResult = 0.
```
4. 注意事项：
	* 切换至的角色信息可在<a href="AVSDK日志分析_基础流程.md/#log_getuserinfo">获取用户基本信息</a>查到；
	* 此错出错通常是：切换的角色不存在（会显示 `roleInfo_-1` 表示未找到角色），或者角色对应的角色类型没有选择；


## <a name="log_avdata">数据流变化</a>
1. 作用：用于查看直播过程中，上下行数据包的变化，进而推断相关上麦用户的行为。
2. 关键字：`AVDATA`
3. 示例日志：

```
// 此为接收到数据流信息
// tinyId：发送数据的人  recvAudPkgCnt 接收音频包，recvVidPkgCnt 报像头视频包数量，recvSubVidPkgCnt 接收辅路视频包
2017/07/20 16:47:16.756| E| 20240| AVGSDK | AVGRoomLogic.cpp(1133):OnTimer                    | AVDATA in 10 secs. . id =  tinyId = 144115197086273818 recvAudPkgCnt = 250 recvVidPkgCnt = 695 recvSubVidPkgCnt = 314.


// 发送到数据流的信息
// sendAudPkgCnt : 发音频包数量，sendVidPkgCnt：发摄像头视频包数量
2017/07/20 16:39:23.544| E| 18578| CAVGRoo| AVGRoomLogic.cpp(1926):OnTimer                    | AVDATA in 10 secs.  sendAudPkgCnt = 250 sendVidPkgCnt = 898.

```
4. 注意事项：
 	* `AVDATA` 在日是志里面是10s才打印一次；
 	* 实际查看时，会看到一段时间内直播间内数据流的变化；如果某一用户的上行数据（send相关）变化较大，可在[AVMonitor](http://avq.server.com/reportapp/)查相应的数据数据以及网络丢包；如果是某一用户的下行数据发生变化，可拿对应的`tinyid`值在[AVMonitor](http://avq.server.com/reportapp/)查相应的数据数据以及网络丢包；
 	* 如果发现某个用户的**视频数据**没有了，可拿`tinyid`去[AVMonitor](http://avq.server.com/reportapp/)查其对应的**视频位**情况（注意此时看）；或该用户为当前日志的用户，可查其<a href="#log_eventid">用户事件</a>
 	* 如果发现某个用户的**音频数据**没有了，可拿`tinyid`去[AVMonitor](http://avq.server.com/reportapp/)查其对应的**上行音频码率**情况；或该用户为当前日志的用户，可查其<a href="#log_eventid">用户事件</a>

## <a name="log_eventid">用户事件</a>

1. 作用：用于查看直播过程中，直播间上行用户的事件通知
2. 关键字：`eventid =`
3. 示例日志：

```
2017/07/20 16:31:36.329| E| 17324| Client | av_context_impl.cpp(1730):OnEndpointsUpdateInfo   | OnEndpointsUpdateInfo. eventid = 1, identifier = 1785032017/07/20 16:31:36.539| E| 17324| Client | av_context_impl.cpp(1730):OnEndpointsUpdateInfo   | OnEndpointsUpdateInfo. eventid = 1, identifier = 6000062017/07/20 16:31:36.539| E| 17324| Client | av_context_impl.cpp(1730):OnEndpointsUpdateInfo   | OnEndpointsUpdateInfo. eventid = 7, identifier = 6000062017/07/20 16:31:36.911| E| 17324| Client | av_context_impl.cpp(1730):OnEndpointsUpdateInfo   | OnEndpointsUpdateInfo. eventid = 5, identifier = 6000062017/07/20 16:33:39.611| E| 17324| Client | av_context_impl.cpp(1730):OnEndpointsUpdateInfo   | OnEndpointsUpdateInfo. eventid = 1, identifier = 12361592017/07/20 16:34:28.844| E| 17324| Client | av_context_impl.cpp(1730):OnEndpointsUpdateInfo   | OnEndpointsUpdateInfo. eventid = 6, identifier = 6000062017/07/20 16:35:37.608| E| 17324| Client | av_context_impl.cpp(1730):OnEndpointsUpdateInfo   | OnEndpointsUpdateInfo. eventid = 2, identifier = 1236159

```
`identifier`为事件的主体者：

`eventid` 对应的值可参考如下：

```
typedef NS_ENUM(NSInteger, QAVUpdateEvent) {    QAV_EVENT_ID_NONE                      = 0, ///< 默认值，无意义。    QAV_EVENT_ID_ENDPOINT_ENTER            = 1, ///< 进入房间事件。    QAV_EVENT_ID_ENDPOINT_EXIT             = 2, ///< 退出房间事件。    QAV_EVENT_ID_ENDPOINT_HAS_CAMERA_VIDEO = 3, ///< 有发摄像头视频事件。    QAV_EVENT_ID_ENDPOINT_NO_CAMERA_VIDEO  = 4, ///< 无发摄像头视频事件。    QAV_EVENT_ID_ENDPOINT_HAS_AUDIO        = 5, ///< 有发语音事件。    QAV_EVENT_ID_ENDPOINT_NO_AUDIO         = 6, ///< 无发语音事件。    QAV_EVENT_ID_ENDPOINT_HAS_SCREEN_VIDEO = 7, ///< 有发屏幕视频事件。    QAV_EVENT_ID_ENDPOINT_NO_SCREEN_VIDEO  = 8, ///< 无发屏幕视频事件。    QAV_EVENT_ID_ENDPOINT_HAS_MEDIA_FILE_VIDEO = 9, ///< 有发文件视频事件。    QAV_EVENT_ID_ENDPOINT_NO_MEDIA_FILE_VIDEO  = 10, ///< 无发文件视频事件。};
```


4. 注意事项：
 	* `eventid` 只反应当前用户从进房到出房这段时间内用户的关键事件；
 	* 实际使用，结合用户反馈的情况，梳理对应的时候间，以分析推断用户的操作。如用户反馈画面卡住，没声明音，可查对应时间点上是否有用户的关摄像头事件(eventid = 4)或关mic事件(eventid = 6)
 	

## <a name="log_requestviewlist">请求画面</a>
1. 作用：用于查看画面请求状态，可以从日志中判断当前用户的观看画面的情况；
2. 关键字：`requestviewlist`
3. 注意事项：该项需要与<a href="AVSDK日志分析_基础流程.md#log_enterroom">进房流程查询</a处的`video_recv_mode`,`screen_recv_mode`参数配置 ，以及<a href="#log_eventid">用户事件</a>结合使用
3. 示例日志：

```
// 1.先用用户事件
2017/07/21 15:38:04.715| E| 26718| Client | av_context_impl.cpp(1855):OnEndpointsUpdateInfo   | OnEndpointsUpdateInfo. eventid = 7, identifier = adev0

// 2.根据用户事件去合理请求用户的画面
2017/07/21 15:38:04.733| I| 26718| Client | av_request_view_impl.cpp(63):RequestViewList      | ******RequestViewList. count = 1, complete_callback = 0x126826240. 2017/07/21 15:38:04.733| I| 26718| Client | av_request_view_impl.cpp(63):RequestViewList      | ******RequestViewList. count = 1, complete_callback = 0x126826240. 2017/07/21 15:38:04.733| I| 26718| Client | av_request_view_impl.cpp(161):RequestViewList     | ******RequestViewList. count = 1, id = adev0, size_type = 1, video_src_type = 2, callback = 0x126826240.2017/07/21 15:38:04.733| I| 26718| Client | av_request_view_impl.cpp(161):RequestViewList     | ******RequestViewList. count = 1, id = adev0, size_type = 1, video_src_type = 2, callback = 0x126826240.2017/07/21 15:38:04.733| E| 26718| Client | av_request_view_impl.cpp(200):RequestViewList     | screen_identifier = adev0, screen_tiny_id = 144115200359467436.2017/07/21 15:38:04.734| I| 26718| Client | av_request_view_impl.cpp(435):RequestViewList     | ******RequestViewListing...2017/07/21 15:38:04.734| I| 26718| Client | av_request_view_impl.cpp(435):RequestViewList     | ******RequestViewListing...2017/07/21 15:38:04.748| F| 26718| Client | av_request_view_impl.cpp(655):OnRequestViewListCal| result = 0, pDataReply = 0x12699dc90.2017/07/21 15:38:04.748| E| 26718| Client | av_request_view_impl.cpp(680):OnRequestViewListCal| OK. result = 0, retCode = 0.2017/07/21 15:38:04.748| E| 26718| Client | av_request_view_impl.cpp(735):OnRequestViewListCal| ret_code = 0.2017/07/21 15:38:04.748| I| 26718| Client | av_request_view_impl.cpp(961):OnRequestViewListCal| ******RequestViewList OK. identifier count = 1, id = adev0, size_type = 1, video_src_type = 2.2017/07/21 15:38:04.748| I| 26718| Client | av_request_view_impl.cpp(961):OnRequestViewListCal| ******RequestViewList OK. identifier count = 1, id = adev0, size_type = 1, video_src_type = 2.

```


4. 注意事项：
 	* 如果<a href="AVSDK日志分析_基础流程.md#log_enterroom">进房流程查询</a处的`video_recv_mode`,`screen_recv_mode`参数配置均为true, 进房成功后，server会主动下发进房时刻，房间内已上行的用户视频信息。其他的请求请依赖<a href="#log_eventid">用户事件</a>（主要是eventid = 3, 7, 9）进行合理请求；
 	* AVSDK自带的接口`- (void)requestViewList:(NSArray *)identifierList srcTypeList:(NSArray *)srcTypeList ret:(RequestViewListBlock)block;
`不会对接口请求参数作缓存，如当前房间内有A的画面，些时B上麦时，请求画面要传 A , B两个对应的参数信息；
	* ILiveSDK内部对AVSDK自带的接口进行封装，内部会有记忆功能，如当前房间内有A的画面，些时B上麦时，请求画面只要传B对应的参数信息即可；
	* 该信息可辅助查询

## <a name="log_camera">摄像头操作</a>
<a name="log_device_windows">注意：<font color=red>windows下较其他平台多一个选择摄像头麦克风输入源操作，windows下需要在进房间成功后，使用AVDeviceMgr去选择对应的设备</font></a>

1. 作用：用于看用户操作摄像头相关的操作；<a name="log_camera_case">打开摄像头，本质上要向server申请摄权限位，关闭则为清除权限位；若申请到视频位且有上行视频权限，则打开摄像头后，可上行视频，能推流成功（web侧可以观看），且后台会转发，这样其他的观众也可看到；若无视频位权限，客户端会可以上行，能推流成功，但是后台不会转发，其他使用AVSDK的看不了</a>；
2. 关键字：`EnableCamera` , `SwitchCamera`
3. 操作摄像头时机：
	* <a href="AVSDK日志分析_基础流程.md#log_enterroom_param">进房配置</a>：与进房的`auth_bits`有关；
	* 手动打开／切换/关闭摄像头；
	* 上麦（开摄像头）／下麦（关摄像头）；
	* 前后台切换：移动端切后台时，需要关掉摄像头，切前台要重新打开摄像头

4. 关键日志：

```
2017/07/21 17:47:46.279| I| 28368| Client | av_video_ctrl_impl.cpp(622):EnableCamera          | ******EnableCamera. is_enable = 1, callback = 0x17005cb30.
2017/07/21 17:47:46.279| E| 28368| Client | av_video_ctrl_impl.cpp(1111):EnableCameraInternal | AVVideoCtrlImpl::EnableCameraInternal. is_enable = 1, camera_id = 0
2017/07/21 17:47:46.282| E| 28368| Client | av_camera_device_ios.mm(819):EnableCamera         | AVCameraDeviceIOS::EnableCamera. camera_id = 0, is_select = 1
2017/07/21 17:47:46.765| I| 28368| AVGSDK | av_camera_device.cpp(203):Enable                  | EnableCamera OK.
2017/07/21 17:47:46.769| I| 28368| Client | av_video_ctrl_impl.cpp(654):EnableCamera          | ******EnableCameraing...
2017/07/21 17:47:54.423| I| 28368| Client | av_video_ctrl_impl.cpp(622):EnableCamera          | ******EnableCamera. is_enable = 0, callback = 0x17005cb30.
2017/07/21 17:47:54.424| E| 28368| Client | av_video_ctrl_impl.cpp(1111):EnableCameraInternal | AVVideoCtrlImpl::EnableCameraInternal. is_enable = 0, camera_id = 0
2017/07/21 17:47:54.425| E| 28368| Client | av_camera_device_ios.mm(819):EnableCamera         | AVCameraDeviceIOS::EnableCamera. camera_id = 0, is_select = 0
2017/07/21 17:47:54.564| I| 28368| AVGSDK | av_camera_device.cpp(203):Enable                  | EnableCamera OK.
2017/07/21 17:47:54.564| I| 28368| Client | av_video_ctrl_impl.cpp(654):EnableCamera          | ******EnableCameraing...

```

注意事项：

* 1.9.1之后的版本，支持提前预览，摄像头操作与房间概念独立，在退出房间时，记得手动关闭摄像头（1.9.1之前退房时，SDK内部会自动关闭摄像头）；
* 上行权限跟音视权限位一般情况下可以等同理解，但有以下情况下：用户直播过程中，遇到网络中断，或网络丢包严重，或退后台等，或者过多用户拥有上行权限，过长时间没有向后台发送视频数据，此时视频位会被云后台回收，则会出现<a href="#log_camera_case">上述的情况</a>;

## <a name="log_mic">Mic操作</a>
请留意：<a href="#log_device_windows">注意事项</a>

1. 作用：用于看用户操作麦克风的操作；音频相关组件在第一次使用到麦克风或扬声器时就已初始化好，`EnableMic`操作主要是向后台发送一条<a name="log_eventid">信令</a>（eventid = 5, 6）；
2. 关键字：`EnableMic`
3. Mic时机：
	* <a href="AVSDK日志分析_基础流程.md#log_enterroom_param">进房配置</a>：与进房的`auth_bits`有关，以及`QAVMultiParam`中的`enableMic`；
	* 手动打开／关闭Mic；
	* 上麦（开Mic）／下麦（关Mic）；
	* 前后台切换：具体与应用是否支持后台模式有关

4. 关键日志：

```
2017/07/21 17:47:46.143| I| 28368| Client | av_audio_ctrl_impl.cpp(446):EnableMic             | ******EnableMic. isEnable = 1
2017/07/21 17:47:46.145| I| 28368| Client | av_audio_ctrl_impl.cpp(475):EnableMic             | ******EnableMicing...
2017/07/21 17:47:47.200| A| 18570| CMultiM| CMultiMediaEngine.cpp(148):MultiSpeechEngineLog   | AudioEngineLog [TRAE] [INFO] [QTTopo.cpp:415] $ EnableMic: on
2017/07/21 17:47:47.203| A| 18570| CMultiM| CMultiMediaEngine.cpp(148):MultiSpeechEngineLog   | AudioEngineLog [TRAE] [INFO] [QTTopo.cpp:415] $ EnableMic: on
2017/07/21 17:47:47.263| A| 18570| CMultiM| CMultiMediaEngine.cpp(148):MultiSpeechEngineLog   | AudioEngineLog [TRAE] [INFO] [QTTopo.cpp:415] $ EnableMic: off
2017/07/21 17:47:47.305| A| 18570| CMultiM| CMultiMediaEngine.cpp(148):MultiSpeechEngineLog   | AudioEngineLog [TRAE] [INFO] [QTTopo.cpp:415] $ EnableMic: on

```

注意事项：

* 注意与<a href="#log_eventid">用户事件</a>结合使用

## <a name="log_audioencdec">音频编解码相关</a>

* 主播无声音查看：<a href="AVSDK主播无声音问题定位手册.docx">AVSDK主播无声音问题定位手册.docx</a>
* 观众无声音查看：<a href="AVSDK观众无声音问题定位手册.docx">AVSDK观众无声音问题定位手册.docx</a>
* 伴奏相关可查看：<a href="伴奏文档1.0.doc">伴奏文档1.0.doc</a>




## <a name="log_videoenc">视频编码相关</a>
1. 作用：查看视频上行视频帧信息变化，如果上行帧数变化大，可结合<a href="#log_net">网络情况相关</a>对比一下丢包
2. 关键字：`capfps` 、`encSendfps`， `EncodeFrame`，相关日志2s打一次
3. 示例日志：

```
2017/07/27 13:41:48.527| E| 18592| CVideoE| VideoEncSession.cpp(977):GetEncVideoStat          | capfps :199 ifec:20 pfec:20 dwPkt_S:760 encSendfps:200 encSendbitrate:505 encbit 486  0
2017/07/27 13:41:50.529| E| 18592| CVideoE| VideoEncSession.cpp(977):GetEncVideoStat          | capfps :200 ifec:20 pfec:20 dwPkt_S:821 encSendfps:200 encSendbitrate:575 encbit 548  0
2017/07/27 13:41:52.532| E| 18592| CVideoE| VideoEncSession.cpp(977):GetEncVideoStat          | capfps :199 ifec:20 pfec:20 dwPkt_S:881 encSendfps:199 encSendbitrate:568 encbit 540  0
2017/07/27 13:41:54.544| E| 18592| CVideoE| VideoEncSession.cpp(977):GetEncVideoStat          | capfps :199 ifec:20 pfec:20 dwPkt_S:881 encSendfps:199 encSendbitrate:568 encbit 540  0
2017/07/27 13:41:56.549| E| 18592| CVideoE| VideoEncSession.cpp(977):GetEncVideoStat          | capfps :200 ifec:20 pfec:20 dwPkt_S:942 encSendfps:201 encSendbitrate:517 encbit 498  0
```
```
2017/07/27 13:41:27.949| E| 18649| CVideoE| VideoEncoder.cpp(2609):EncodeFrame                | Begin encode nGopIndex:11 nFrameType:0 nFrameIndex:0 nEncodeIndex 400 
2017/07/27 13:41:29.954| E| 18649| CVideoE| VideoEncoder.cpp(2609):EncodeFrame                | Begin encode nGopIndex:12 nFrameType:0 nFrameIndex:0 nEncodeIndex 440 
2017/07/27 13:41:32.000| E| 18649| CVideoE| VideoEncoder.cpp(2609):EncodeFrame                | Begin encode nGopIndex:13 nFrameType:0 nFrameIndex:0 nEncodeIndex 480 
2017/07/27 13:41:33.959| E| 18649| CVideoE| VideoEncoder.cpp(2609):EncodeFrame                | Begin encode nGopIndex:14 nFrameType:0 nFrameIndex:0 nEncodeIndex 520 
2017/07/27 13:41:35.956| E| 18649| CVideoE| VideoEncoder.cpp(2609):EncodeFrame                | Begin encode nGopIndex:15 nFrameType:0 nFrameIndex:0 nEncodeIndex 560 
2017/07/27 13:41:37.954| E| 18649| CVideoE| VideoEncoder.cpp(2609):EncodeFrame                | Begin encode nGopIndex:16 nFrameType:0 nFrameIndex:0 nEncodeIndex 600 
2017/07/27 13:41:39.953| E| 18649| CVideoE| VideoEncoder.cpp(2609):EncodeFrame                | Begin encode nGopIndex:17 nFrameType:0 nFrameIndex:0 nEncodeIndex 640 
```


## <a name="log_videodec">视频解码相关</a>
1. 作用：查看视频上行视频帧信息变化，如果上行帧变化大，会引影响其他观看用户
2. 关键字：`decoderfps`, `DecodeFrame`
3. 示例日志：

```
2017/07/18 19:38:49.886| D| 10943| CMultiM| CMultiMediaEngine.cpp(2961):GetRecvVideoStat      | decoderfps 184 bit 761 delay 0 loss 0  hwdec = 02017/07/18 19:38:51.898| D| 10943| CMultiM| CMultiMediaEngine.cpp(2961):GetRecvVideoStat      | decoderfps 93 bit 681 delay 0 loss 0  hwdec = 02017/07/18 19:38:51.898| D| 10943| CMultiM| CMultiMediaEngine.cpp(2961):GetRecvVideoStat      | decoderfps 93 bit 681 delay 0 loss 0  hwdec = 02017/07/18 19:38:53.911| D| 10943| CMultiM| CMultiMediaEngine.cpp(2961):GetRecvVideoStat      | decoderfps 114 bit 739 delay 0 loss 0  hwdec = 0
	
```
备注： 通常`decoderfps`波动较大时，会大量伴随出现下面的日志（可搜关键字`DecodeFrame ` `can not be decode`），即出现观看视频卡问题，一般解码出错的原因是网络丢包，导致参考帧丢失，解码失败

```
2017/07/18 19:34:27.095| D| 12812| CVideoD| VideoDecoder.cpp(519):DecodeFrame                 | WARNING!!! CVideoDecoder::it can not be decode. CHN = 0 nFrameType = 3, bCanDecode = 0, m_enDataType = 1, nGOPIndex = 233, nFrmIdx 2 nRefFrameIndex = 0, m_nLastGOPIndex = 0, m_nLastFrameIndex = 0, m_nLastIFrameIndex = 0, m_nLastSPFrameIndex = 0, m_nLastGFFrameIndex = 0
```

## <a name="log_net">网络情况相关</a>
1. 作用：查看用户的直播过程中网络包丢包情况，以及重传之后的丢包情况；如果丢包较严重。注意此处与[AVMonitor](http://avq.server.com/reportapp/)丢包率的区别：如果用户侧丢包严重，有可能上传的心跳包都丢失，这样AVMonitor在绘制时，默认的处理是填0, 有可能监控上看到的丢包率是0, 但实际看到的直播效果很差；
2. 关键字：`CurLostRate`可查网络包丢包情况 , `UDTS CalcSendLoss dwNoAckNum`可查重传之后的丢包情况
3. 示例日志：

```
//CurLostRate查到的示例日志

[2017/01/04 17:13:54.455| E| 916  | AVGCongestion.cpp(534):CheckLostRate_AfterACK     ] CWnd Size:52,CurLostRate:1016,MinLostRate:419,RTT:1012,RTTD:378,Available[2],MaxCwnd:210,MinCwnd:52[2017/01/04 17:13:54.893| E| 916  | AVGCongestion.cpp(534):CheckLostRate_AfterACK     ] CWnd Size:52,CurLostRate:1105,MinLostRate:469,RTT:1281,RTTD:418,Available[0],MaxCwnd:210,MinCwnd:52[2017/01/04 17:13:55.410| E| 916  | AVGCongestion.cpp(534):CheckLostRate_AfterACK     ] CWnd Size:52,CurLostRate:1032,MinLostRate:519,RTT:1752,RTTD:69,Available[0],MaxCwnd:251,MinCwnd:52
```
备注：
	* 可结合<a href="#log_avdata">数据流变化</a>来查：如果收到视频帧数据变化不大，下行解决波动较大，则可说明下行端解码没有跟上，导致观看端视频出现卡顿；
	* 可结合中[AVMonitor](http://avq.server.com/reportapp/)看下行视频帧率是否有较大波动；

* 其中丢包率为：`CurLostRate`值，计算如：1016/100 = 10.16 ，即丢包率为10.16%；
* `RTT`为往返延时，单位ms

```
//UDTS CalcSendLoss dwNoAckNum查重传之后还有多少丢包

[2017/01/04 17:13:51.792| E| 916  | AVGUDTSend.cpp(560):InternalUDTCalcSendLoss       ] UDTS CalcSendLoss dwNoAckNum[4] dwSendTotalNum[107] dwSendLossRate[373][2017/01/04 17:13:53.820| E| 916  | AVGUDTSend.cpp(560):InternalUDTCalcSendLoss       ] UDTS CalcSendLoss dwNoAckNum[7] dwSendTotalNum[124] dwSendLossRate[564][2017/01/04 17:13:55.847| E| 916  | AVGUDTSend.cpp(560):InternalUDTCalcSendLoss       ] UDTS CalcSendLoss dwNoAckNum[4] dwSendTotalNum[141] dwSendLossRate[283][2017/01/04 17:13:59.888| E| 916  | AVGUDTSend.cpp(560):InternalUDTCalcSendLoss       ] UDTS CalcSendLoss dwNoAckNum[15] dwSendTotalNum[164] dwSendLossRate[914]
```
备注：

* `dwSendTotalNum`为总重传发包数，`dwNoAckNum`为重传丢包数，`dwSendLossRate`为重传丢包率；计算规则：`dwSendLossRate` = `dwNoAckNum` ／ `dwSendTotalNum` * 10000;

### <a name="log_net_send">UDT发包日志</a>
1. 作用：查看上行端网络发包情况2. 关键字：`OutPacketNew Subtype`UDT出包日志，通常2S一次
3. 示例日志：

```
2017/07/27 11:39:35.436| E| 10063| CmdCode| AVGUDTRecv.cpp(1807):OutPacketAudioNew            | OutPacketNew Subtype:1 Seq:15051 28106723 TimelineOut:2183188355 METimeStamp:2183188711 DataLen:441 Jitter:41 Tickout:102 FrameType:1 GOP:0 FrameIdx:0 TotalPkg:0 AudPlayDelay:357 Uin:144115199612954932 OutStamp:2183189132
2017/07/27 11:39:37.476| E| 10063| CmdCode| AVGUDTRecv.cpp(1807):OutPacketAudioNew            | OutPacketNew Subtype:1 Seq:15104 28106776 TimelineOut:2183190490 METimeStamp:2183190833 DataLen:343 Jitter:45 Tickout:10 FrameType:1 GOP:0 FrameIdx:0 TotalPkg:0 AudPlayDelay:379 Uin:144115199612954932 OutStamp:2183191172
2017/07/27 11:39:39.479| E| 10063| CmdCode| AVGUDTRecv.cpp(1807):OutPacketAudioNew            | OutPacketNew Subtype:1 Seq:15154 28106826 TimelineOut:2183192517 METimeStamp:2183192846 DataLen:329 Jitter:45 Tickout:8 FrameType:1 GOP:0 FrameIdx:0 TotalPkg:0 AudPlayDelay:384 Uin:144115199612954932 OutStamp:2183193175
```

备注：
* 如果网络稳定，`Seq`增量变化不会太大，如果波动大可结合[AVMonitor](http://avq.server.com/reportapp/)上的丢包率，以及是否有下行音视频数据看下；
* 适当看下[AVMonitor](http://avq.server.com/reportapp/)上CPU变化情况是否能与发包变时对得上，有些情况下是用户没有做压测，线上消息量大时，就会影响采集，间接影响发包数；

### <a name="log_net_recv">UDT收包日志</a>
1. 作用：查看下行端网络收包情况2. 关键字：`OnDataHandle `UDT收包日志，通常2S一次
3. 示例日志：

```
2017/07/27 11:39:34.861| E| 9081 | CmdCode| AVGUDTRecv.cpp(658):OnDataHandleNew               | OnDataHandle: SubType 1 seq 140646 74763636 FT 1 PkgIdx  0 TotalPkgCnt  0 fecN  0 FrmIdx  0 GopIdx  0 dataTS 2183188459 AudPlayDelay 382 Uin 144115199612798687
2017/07/27 11:39:36.883| E| 9081 | CmdCode| AVGUDTRecv.cpp(658):OnDataHandleNew               | OnDataHandle: SubType 1 seq 140697 74763687 FT 1 PkgIdx  0 TotalPkgCnt  0 fecN  0 FrmIdx  0 GopIdx  0 dataTS 2183190498 AudPlayDelay 388 Uin 144115199612798687
2017/07/27 11:39:38.881| E| 9081 | CmdCode| AVGUDTRecv.cpp(658):OnDataHandleNew               | OnDataHandle: SubType 1 seq 140747 74763737 FT 1 PkgIdx  0 TotalPkgCnt  0 fecN  0 FrmIdx  0 GopIdx  0 dataTS 2183192490 AudPlayDelay 391 Uin 144115199612798687
2017/07/27 11:39:40.880| E| 9081 | CmdCode| AVGUDTRecv.cpp(658):OnDataHandleNew               | OnDataHandle: SubType 1 seq 140797 74763787 FT 1 PkgIdx  0 TotalPkgCnt  0 fecN  0 FrmIdx  0 GopIdx  0 dataTS 2183194498 AudPlayDelay 369 Uin 144115199612798687
2017/07/27 11:39:42.914| E| 9081 | CmdCode| AVGUDTRecv.cpp(658):OnDataHandleNew               | OnDataHandle: SubType 1 seq 140848 74763838 FT 1 PkgIdx  0 TotalPkgCnt  0 fecN  0 FrmIdx  0 GopIdx  0 dataTS 2183196530 AudPlayDelay 371 Uin 144115199612798687
```

备注：
* 如果网络稳定，`Seq`增量变化不会太大，如果波动大可结合[AVMonitor](http://avq.server.com/reportapp/)上的丢包率，以及上行音视频数据看下；
* 适当看下[AVMonitor](http://avq.server.com/reportapp/)上CPU变化情况是否能与发包变时对得上，有些情况下是用户没有做压测，线上消息量大时，就会影响采集，间接影响收包数，但一般影响较小；



## <a name="log_dumpaudio">如何dump音频数据</a>


### <a name="log_dumpaudio_ios">iOS上dump音频数据</a>
示例：彩视2017年6月底一直反馈，iOS主播直播过程中，所以观众听到主播的音频是一卡一卡的，此种情况下dump主播相关的音频信息，以便进一步分析；
操作步骤：
1. 把这个<a href="trae_indev_capture.config">`trae_indev_capture.config`</a>放到app的document目录里面去（记得在info.plist里面打开文件共享）；若文件无法下载，可将以下内容保存成文件`trae_indev_capture.config`;

```
indev

```

2. 启动app，开始一个直播；
3. 直播完成后，如果dump成功后，document目录下面会生成一个TRAE_DBG_Dump目录，此目录下即为相关的音频文件，将音频文件拿到电脑上使用core audio进行分析即可；


<font color=red> 以下待补充
### <a name="log_dumpaudio_android">android上dump音频数据</a>
### <a name="log_dumpaudio_windows">windows上dump音频数据</a>
</font>
## <a name="log_dumpvideo">如何dump视频帧数据</a>

<font color=red> 以下待补充
### <a name="log_dumpvideo_ios">iOS上dump视频数据</a>
### <a name="log_dumpvideo_android">android上dump视频数据</a>

</font>

### <a name="log_dumpvideo_windows">windows上dump视频数据</a>
<font color=red>此方法只能拿到的视频是采集后，编码前的数据，实战上可能用不上，此处不多说明，主要作有印象即可。</font>

示例：智慧树使用182, 以及185, 直播静态图像，在大屏上观看时，画面呼吸现象严重（观看端规律性地看到画面一时清晰，一时模糊，时间间隔约在2s左右）；
原因：直播时画面呼吸现象多少会有一点儿，主要是固定刷I帧，GOP比较小（不然进房无法很快显示画面）。这种无法做到完全没有的，但是模糊的程度比较严重的话，是有问题的，轻微的是正常的；
操作步骤：

1. 把这个<a href="avsdk_config.ini">`avsdk_config.ini`</a>文件放到exe所在目录下;
2. 开始一场直播后，会在d盘根目录生成编码后的录像文件`encode.264`;
3. 将`encode.264`使用工具`Elecard StreamEye`打开（工具较专业，可找mortyzhu看下）；




