# XCast核心操作



## <a name="xcast_get_set_property">xcast\_get/set_property</a>

接口声明如下：

```
// xcast_get_property
/*
* prop : 命令键直，具体参考 xcast_define.h
* 返回值：即prop对应的信息
* 


// xcast_set_property
/*
* prop : 命令键直，具体参考 xcast_define.h
* val : 对应键值prop要传的参数内容
* 返回值：操作返回返回值，0 成功，非0失败


```

<font color = red>
* `xcast_get_property` ： 主要用于查询xt发送操作命令，如请求预览画面，将画面上行；xcast数据状态，如获取摄像头列表，查询流状态等等，get方法都是同步返回结果；

* `xcast_set_property` ： 主要是向更新xcast loop里面的行为，操作结果当场返回，具体是否实现，则是通过<a href="xcast_flow.md?#xcast_handle_event_regist">事件回调</a>返回的，具体后续再介绍；

* 具体如何区分，需要对照协议`xcast_define.h`中的执行；
</font>

举例如下：

1. 查旬当前房间内的流信息（即有几个用户上行），其对应协议如下，其只有`get`方法，说明只能用 `xcast_get_property`去获取，即  `xcast_get_property(XC_STREAM)`即可；

```
/*

```

2. 打开数据的上行下行 （对音视频流信息的操作）：具体协议如下，其中只有`set`方法，说明只能`xcast_set_property`去获取设置，后续会讲解如何使用

```
/*

```

3. 设置/获取音视频能流信息：里面有`set`与`get`, 即可以`xcast_set_property`与`xcast_get_property`进行设置

```

```


# <a name="xcast_device"> 设备 </a>

##  <a name="xcast_device_hotplug">设备插拔监听</a>
* 在<a href="xcast_flow.md?#xcast_handle_event_regist">注册监听</a>后，是通过<a href="xcast_flow.md?#xcast_handle_event_deviceeventcallback">XC\_EVENT_DEVICE事件</a>回调出来的。业务上只需要详细了解回调内容并解析，即可做到设备的监听以及处理；
* 设置与房间独立的，

以下是进房前设备更新插拔回调的内容：<font color="red">通过回调，业务上决策选择要打开的设备并进行记录；</font>

```
["state":int32(1),"err":int32(0),"type":int32(1),"class":int32(7),"src":"ext1"] 
```


<!--## <a name="xcast_device_deviceinfo"> 设备信息回调 </a>
## <a name="xcast_device_hotplug">设备热插拔</a>
### <a name="xcast_device_detect_camera"> 摄像头检测 </a>
### <a name="xcast_device_detect_mic"> 麦克风检测 </a>
### <a name="xcast_device_detect_speaker"> 扬声器检测 </a>-->


## <a name="xcast_device_camera">摄像头</a>

摄像头分为预览和采集；

* 不在房间时，可以多个预览（PC上，移动端只支持一个预览），无上行；
* 在房间时，可以多个预览（PC上，移动端只支持一个预览），只能指定一个摄像头上行；



### <a name="xcast_device_cameralist">摄像头列表</a>

```
/*

```
`xcast_get_property(XC_CAMERA)` 即可返回当前的摄像头列表；

### <a name="xcast_device_cameradefault">默认摄像头</a>

```
/*

```

set : 用于指定进房间后默认的摄像头;

get : 获取set方法设置的值；

注意事项：
* SDK里面默认调用了该方法，未指定摄像头采集时，则默认用该摄像头进行采集；


### <a name="xcast_device_oper_camera_preview"> 摄像头预览 </a>

```
xcast_data_t    prop;
```
设置之后，摄像头采集数据会通过<a href="#xcast_device_oper_camera_rawdata">摄像头原始数据</a>回调给上层；


### <a name="xcast_device_oper_camera_preview"> 指定/查询采集用的摄像头</a>

```
/*

```

注意事项：

* `XC_TRACK_CAPTURE` 可以指定房间内用的音视频采集设备；此处主要讲摄像头相关的；
* `"stream.%s.%s.capture"` : 第一个`%s`为 `xcast_start_stream`参数中指定的`streamid`，第二个`%s`为当前设备名，具体看<a href="xcast_flow.md?#xcast_handle_event_deviceeventcallback">`XC_EVENT_DEVICE`</a>中的`src`
* 切换摄像头进行采集时，也是用的该方法处理；

### <a name="xcast_device_oper_camera_videoout"> 摄像头上行控制 </a>

```
// 打开摄像头并上行
xcast_data_t    data, prop;
prop.format(XC_TRACK_ENABLE,"stream1", "video-out");   
const char * prostr = prop.dump();
xcast_set_property(prop, data);

```

* `stream1` 为`xcast_start_stream`时的`streamid`,  video-out以应视频上行命令字。 
*  上行命令字有 `video-out` (摄频上行) , `audio-out` （音频上行） , `sub-video-out` （辅路上行）

### <a name="xcast_device_oper_camera_rawdata"> 摄像头原始数据 </a>

示例代码详细可见 <a href="xcast_flow.md?#xcast_device_event_samplecode">设备事件回调处理中的xc_device_preview处理逻辑</a>

具体如何处理渲染数据如下：

```
int32_t ui_device_preview(xcast_data &evt, void *user_data)
    {
    	// 播片数据
```

<!--#### <a name="xcast_device_oper_camera_encodedata"> 摄像头原始编码数据 </a>-->

### <a name="xcast_device_remote_camera_preview"> 渲染远端摄像头画面 </a>

远程数据是通过<a href="xcast_flow.md?#xcast_handle_event_trackeventcallback">track事件</a>的 <a href = "xcast_flow.md?#xcast_handle_event_trackeventcallback_samplecode">`xc_track_media`</a>回调相应的视频数据的,  拿到数据如处理如下：

```
/* 媒体流轨道数据通知 */
  
  	// 远程视频数据

```

### <a name="xcast_device_requestview">请求/取消房间内下行视频流</a>

```
/*

```
* `"stream.%s.%s.enable"` : 第一个`%s`为 <a href="xcast_flow.md?#xcast_handle_event_streameventcallback">`XC_EVENT_STREAM`</a>返回的其他流的`streamid`，第二个`%s`为参数可以 video-in (视频下行), audio-in (音频下行),  sub-video-in(辅路下行)；
* `enable` : true 请求,false 不请求

<!--#### <a name=" "> 获取远程摄像头数据 </a>-->

### <a name="xcast_device_camera_state"> 摄像头状态查询 </a>

```
/*

```
`%s`为当前设备名，具体看<a href="xcast_flow.md?#xcast_handle_event_deviceeventcallback">`XC_EVENT_DEVICE`</a>中的`src`


## <a name="xcast_device_mic"> 麦克风 </a>

以下命令字中的`%s`, 均为当前要查询的麦克风名，具体看<a href="xcast_flow.md?#xcast_handle_event_deviceeventcallback">`XC_EVENT_DEVICE`</a>中的`src`

###  <a name="xcast_device_mic_list">查询麦克风列表</a>

```
/* xcast支持的麦克风属性 */

###  <a name="xcast_device_mic_default">默认麦克风</a>

```
```

`%s` 为麦克风名，来源同<a href="#xcast_device_cameradefault">默认摄像头</a>

### <a name="xcast_device_mic_loopback">LoopBack</a>

```
```


### <a name="xcast_device_mic_volume">采集音量</a>

```

```


### <a name="xcast_device_mic_audioout">控制音频上行</a>
同 <a href="#xcast_device_oper_camera_videoout"> 摄像头上行控制 </a>


### <a name="xcast_device_mic_requestaudio">请求/取消远程音频</a>
同 <a  href="xcast_device_requestview">请求/取消房间内下行视频流</a>



## <a name="xcast_device_spearker"> 扬声器 </a>

以下命令字中的`%s`, 均为当前要查询的扬声器名，具体看<a href="xcast_flow.md?#xcast_handle_event_deviceeventcallback">`XC_EVENT_DEVICE`</a>中的`src`


### <a name="xcast_device_spearker_list"> 扬声器列表 </a>

```
/* xcast支持的扬声器属性 */
```

```
```
同<a name="xcast_device_camera_default">默认报像头</a>



```

### <a name="xcast_device_spearker_mode"> 扬声器模式(外放/耳机切换) </a>

```

### <a name="xcast_device_spearker_volume"> 扬声器音量控制 </a>

```

```


## <a name="xcast_stream">流</a>

### <a name="xcast_stream_id">获取当前流id</a>

获取当前`xcast_sart_stream`设置的`streamid`参数

```
/*

### <a name="xcast_stream_state">获取流状态</a>

```

`%s`为`streamid`参数

### <a name="xcast_stream_state">获取流媒体息</a>


获取当前房间中所有上下行的信息（类似于QAVEvent中的has_video,has_audio等信息）

```

"[\"audio-out\",\"video-out\",\"sub-video-out\",\"audio-in-12345\",\"video-in-12345\",\"sub-video-in-12345\"]"	char *
```


## <a name="xcast_track">TRACK</a>

下列命令字中两个`%s`的含义：
第一个`%s` : `xcast_start_stream`中填

### <a name="xcast_track_enable">启动/停止TRACK</a>

```
/*



### <a name="xcast_track_state">查询TRACK状态</a>

```
```

```
```


<!--以下暂不加，VIPKID未用到
### <a name="xcast_device_oper_screen"> 屏幕分享操作 </a>-->


<!--## <a name="xcast_requestview">请求画面</a>-->
<!--## <a name="xcast_externalcapture">自定义采集</a>-->


<!--## <a name="xcast_outin">上下行角色权限控制</a>-->

<!--## <a name="xcast_render">画面渲染</a>

### <a name="xcast_preview">画面预览</a>-->


<!--## <a name="xcast_changerole">角色切换/上下麦</a>-->

<!--## <a name="xcast_render">美颜接入</a>-->

<!--## <a name="xcast_exception">异常处理（如后台，中断等事件）</a>


## <a name="xcast_event_detail">事件详细处理</a>


## <a name="xcast_event_detail">原QAVEndpoint事件通知</a>

## <a name="xcast_error_code">错误码整理</a>-->

## <a name="xcast_define_detail">xcast_define明细</a>

### <a name="xcast_define_format">xcast_define中需要格式化的参数</a>

```
#define XC_STREAM_STATE                     "stream.%s.state"    // xcast_start_stream中传入的streamid

```

```flow
st=>start: start:>http://www.baidu.com
op1=>operation: 操作1
cond1=>condition: YES or NO?
sub=>subroutine: 子程序
e=>end

st->op1->cond1
cond1(yes)->e
cond1(no)->sub(right)->op1  
```







