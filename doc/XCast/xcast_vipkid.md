
# VIPKID集成XCast流程

## XCast基础流程接入

1. 老师学生接入xcast主流程详细参考 <a href = "xcast_flow.md">XCast关键流程接入</a>；
2. 注意监听<a name="xcast_flow.md?#xcast_handle_event_deviceeventcallback">设备事件</a>；
3. 进行以下集成时，请优先阅读 <a href="xcast_normal.md?#xcast_get_set_property">xcast\_get/set_property</a> (主要讲述下列步骤如何使用操作)

## 老师进房
1.  <a name="teacher_audio_out">音频上麦</a> ：<a name="xcast_flow.md?#xcast_handle_event_streameventcallback">处理进房成功回调(stream事件，xc\_stream\_updated)</a>之后，默认打开麦克风并且上行 , 参考<a href="xcast_normal.md?#xcast_device_oper_camera_videoout"> 麦克风上行控制 </a>
2. <a name="teacher_video_out">视频上麦</a> : 手动打开摄像头并且上行，画面渲染 , 参考<a href="xcast_normal.md?#xcast_device_oper_camera_videoout"> 摄像头上行控制 </a>，自己的画面渲染，拿到<a name="xcast_normal.md?#xcast_device_oper_camera_rawdata"> 摄像头原始数据 </a>自行渲染
3. <a name="render_student_video">渲染学生画面</a> ：参考<a name="xcast_normal.md?#xcast_device_remote_camera_preview"> 渲染远端摄像头画面 </a>

## 老师设置
### 摄像头
1. 摄像头列表；参考 <a name="xcast_normal.md?#xcast_device_cameralist">摄像头列表</a>，并配合处理好<a name="xcast_flow.md?#xcast_handle_event_deviceeventcallback">设备事件</a>的处理;
2. 摄像头分开预览； 参考： <a name="xcast_normal.md?#xcast_device_oper_camera_preview"> 摄像头预览 </a>
3. 切换摄像头； 参考： <a name="xcast_normal.md?#xcast_device_oper_camera_preview"> 指定/查询采集用的摄像头</a>

### 麦克风
1. 麦克风列表 ：参考<a name="xcast_normal.md?#xcast_device_mic_list">查询麦克风列表</a>
2. mic音量 ：参考<a name="xcast_normal.md?#xcast_device_mic_volume">采集音量</a>
3. LoopBack : 参考<a name="xcast_normal.md?#xcast_device_mic_loopback">LoopBack</a>
4. 回音消除； <font color='red'> 待补充 </font>
5. 麦克风切换 : 参考 <a name="xcast_normal.md?#xcast_device_oper_camera_preview"> 指定/查询采集用的麦克风</a>

### 扬声器
1. 声音检测 ： 业务播放一个声音文件即可；


## 3. 学生
1. 渲染老师画面：学生进房，自动请求播放音频；只需要将老师画面显示并渲染即可，此处与<a href="#render_student_video">老师渲染学生画面相同</a>;
2. 上麦/下麦，以及自身画面: 同<a href="#teacher_video_out">老师的视频上麦</a>, <a href="#teacher_audio_out">音频上麦</a>;

## 5. 统计数据 
1. 音视频上下行码率 : 
2. RTT延时
3. 丢包率