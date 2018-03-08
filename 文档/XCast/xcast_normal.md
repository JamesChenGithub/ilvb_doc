# XCast核心操作



## <a name="xcast_get_set_property">xcast\_get_property</a>

接口声明如下：

```
// xcast_get_property
/** get property with the given property path.
* prop : 命令键直，具体参考 xcast_define.h
* 返回值：即prop对应的信息
* */xcast_export xcast_variant_t *xcast_get_property(const char *prop);


// xcast_set_property
/** set property with the given property path.
* prop : 命令键直，具体参考 xcast_define.h
* val : 对应键值prop要传的参数内容
* 返回值：操作返回返回值，0 成功，非0失败*/xcast_export int32_t xcast_set_property(const char *prop, xcast_variant_t *val);


```

<font color = red>
* `xcast_get_property` ： 主要用于查询xt发送操作命令，如请求预览画面，将画面上行；xcast数据状态，如获取摄像头列表，查询流状态等等，get方法都是同步返回结果；

* `xcast_set_property` ： 主要是向更新xcast loop里面的行为，操作结果当场返回，具体是否实现，则是通过<a href="xcast_flow.md?#xcast_handle_event_regist">事件回调</a>返回的，具体后续再介绍；

* 具体如何区分，需要对照协议`xcast_define.h`中的执行；
</font>

举例如下：

1. 查旬当前房间内的流信息（即有几个用户上行），其对应协议如下，其只有`get`方法，说明只能用 `xcast_get_property`去获取，即  `xcast_get_property(XC_STREAM)`即可；

```
/** // 媒体流属性: 获取xcast中媒体流列表* "stream":{*   "get":{*     "return":[*       // "null"表示不存在*       null,*       // 所有媒体流名的字符串数组*       ["vstring"]*     ]*   }* },*/#define XC_STREAM                           "stream"

```

2. 打开数据的上行下行 （对音视频流信息的操作）：具体协议如下，其中只有`set`方法，说明只能`xcast_set_property`去获取设置，后续会讲解如何使用

```
/** // 媒体流属性: 启动/停止媒体流轨道* "stream.%s.%s.enable":{*   "set":{*     "params":{*       // true启动,false停止*       "*enable":"vbool",*       // 请求video下行的时候可以指定请求画面大小*       "size":["small","big"]*     }*   }* },*/#define XC_TRACK_ENABLE                     "stream.%s.%s.enable"

```

3. 设置/获取音视频能流信息：里面有`set`与`get`, 即可以`xcast_set_property`与`xcast_get_property`进行设置

```/** // 媒体流属性: 设置媒体流轨道上行音频/视频源* "stream.%s.%s.capture":{*   "get":{*     "return":[*       // 无视频源*       null,*       // 音频/视频源名(摄像头或麦克风)*       "vstring"*     ]*   },*   "set":{*     // 视频源名(摄像头名)*     "params":"vstring"*   }* },*/#define XC_TRACK_CAPTURE                    "stream.%s.%s.capture"

```


# <a name="xcast_device"> 设备 </a>

在<a href="xcast_flow.md?#xcast_handle_event_regist">注册监听</a>后，是通过<a href="xcast_flow.md?#xcast_handle_event_deviceeventcallback">XC\_EVENT_DEVICE事件</a>回调出来的。业务上只需要详细了解回调内容并解析，即可做到设备的监听以及处理；

以下是进房前设备更新插拔回调的内容：<font color="red">通过回调，业务上决策选择要打开的设备并进行记录；</font>

```
["state":int32(1),"err":int32(0),"type":int32(1),"class":int32(7),"src":"ext1"] ["state":int32(1),"err":int32(0),"type":int32(1),"class":int32(7),"src":"ext2"]["state":int32(1),"err":int32(0),"type":int32(1),"class":int32(7),"src":"ext3"] ["state":int32(1),"err":int32(0),"type":int32(1),"class":int32(7),"src":"ext4"] ["state":int32(1),"err":int32(0),"type":int32(1),"class":int32(2),"src":"screen-capture"] ["state":int32(1),"err":int32(0),"type":int32(1),"class":int32(5),"src":"扬声器 (Realtek High Definition Audio)"] ["state":int32(1),"err":int32(0),"type":int32(1),"class":int32(1),"src":"USB 视频设备"] ["state":int32(1),"err":int32(0),"type":int32(3),"class":int32(1),"src":"USB 视频设备"] ["state":int32(1),"err":int32(0),"type":int32(1),"class":int32(1),"src":"USB 视频设备"] ["state":int32(2),"err":int32(0),"type":int32(2),"class":int32(5),"src":"扬声器 (Realtek High Definition Audio)"] 
```

进房前可以调以下代码进行摄像头的预览 : 

// TODO : 补日志

如何在进房后立即打开摄像头预览以及上行

1.xcast_


## <a name="xcast_device_deviceinfo"> 设备信息回调 </a>






## <a name="xcast_device_hotplug">设备热插拔</a>



### <a name="xcast_device_detect_camera"> 摄像头检测 </a>
### <a name="xcast_device_detect_mic"> 麦克风检测 </a>
### <a name="xcast_device_detect_speaker"> 扬声器检测 </a>

## <a name="xcast_device_oper"> 设备操作 </a>

### <a name="xcast_device_oper_camera"> 摄像头 </a>

#### <a name="xcast_device_oper_camera_preview"> 摄像头预览 </a>

```
xcast_data_t    prop;const char *src = evt["src"];		// src为摄像头id,  对应 XC_EVENT_DEVICE协议中的src字段prop.format(XC_CAMERA_PREVIEW, src);const char *str = prop.dump();int32_t ret = xcast_set_property(prop, xcast_data_t(true/false)); // 打开或关闭预览
```
设置之后，摄像头采集数据会通过<a href="#xcast_device_oper_camera_rawdata">摄像头原始数据</a>回调给上层；

#### <a name="xcast_device_oper_camera_videoout"> 摄像头上行 </a>

```
// 打开摄像头并上行
xcast_data_t    data, prop;
prop.format(XC_TRACK_ENABLE,"stream1", "video-out");   // stream1 为xcast_start_stream时的 streamid,  video-out以应视频上行命令字
const char * prostr = prop.dump();data["enable"] = true; // true上行,false不上行
xcast_set_property(prop, data);

```

#### <a name="xcast_device_oper_camera_rawdata"> 摄像头原始数据 </a>

示例代码详细可见 <a href="xcast_flow.md?#xcast_device_event_samplecode">设备事件回调处理中的xc_device_preview处理逻辑</a>

具体如何处理渲染数据如下：

```
int32_t ui_device_preview(xcast_data &evt, void *user_data){  const char       *dev = evt["src"];  if (!dev) return XCAST_ERR_INVALID_ARGUMENT;  switch ((int32_t)evt["class"]) {  case xc_device_camera:  case xc_device_screen_capture: {    /* 摄像头预览数据渲染 */    int32_t         width = evt["width"];    int32_t         height = evt["height"];    int32_t         format = evt["format"];    if (format == xc_media_argb32) 
    {      TrackVideoBuffer *buffer = GetTrackBuffer(dev, width, height);      memcpy(buffer->data, evt["data"].bytes_val(), width * height * kBytesPerPixel);      InvalidVideoView(&buffer->rcOut);    } else if (format == xc_media_layer) {
    	// 播片数据      TrackVideoBuffer *buffer = GetTrackBuffer(dev, width, height, true);      TrackBufferRefreshLayers(buffer, evt["data"]);    }    break;  }  case xc_device_mic:    break;  default:    break;  }  return XCAST_OK;}
```

<!--#### <a name="xcast_device_oper_camera_encodedata"> 摄像头原始编码数据 </a>-->

#### <a name="xcast_device_oper_remote_camera_preview"> 渲染远端摄像头画面 </a>

远程数据是通过<a href="xcast_flow.md?#xcast_handle_event_trackeventcallback">track事件</a>的 <a href = "xcast_flow.md?#xcast_handle_event_trackeventcallback_samplecode">`xc_track_media`</a>回调相应的视频数据的,  拿到数据如处理如下：

```
/* 媒体流轨道数据通知 */int32_t ui_track_media(xcast_data &evt, void *user_data){  TrackVideoBuffer *buffer;  const char        *src = evt["src"];  int32_t           type = evt["type"];  int32_t           clazz = evt["class"];  int32_t           direction = evt["direction"];  if (clazz == xc_track_audio) {    if (direction == xc_track_in) {      /* 在这里播放音频数据 */      const char* pcm_data = evt["data"];      uint32_t     pcm_size = evt["size"];      uint32_t     sample_rate = evt["sample-rate"];      uint32_t     channel = evt["channel"];      uint32_t     bits = evt["bits"];      uint64_t     uin = evt["uin"];      printf("ui_track_media: pcm_size(%u) sample_rate(%u) channel(%u) bits(%u) uin(%llu)\n",            pcm_size, sample_rate, channel, bits, uin);    }  } else if (clazz == xc_track_video) {
  
  	// 远程视频数据    int32_t fmt = (int32_t)evt["format"];    if (fmt == xc_media_argb32) {      buffer = GetTrackBuffer(evt["src"], evt["width"], evt["height"]);      memcpy(buffer->data, (const uint8_t *)evt["data"], (uint32_t)evt["size"]);      InvalidVideoView(&buffer->rcOut);    } else if (fmt == xc_media_layer) {      TrackVideoBuffer *buffer = GetTrackBuffer(evt["src"], evt["width"], evt["height"], true);      TrackBufferRefreshLayers(buffer, evt["data"]);    }  }  return XCAST_OK;}

```

<!--#### <a name="xcast_device_oper_remote_camera_capturedata"> 获取远程摄像头数据 </a>-->



### <a name="xcast_device_oper_mic"> 麦克风操作 </a>
### <a name="xcast_device_oper_spearker"> 扬声器操作 </a>
### <a name="xcast_device_oper_screen"> 屏幕分享操作 </a>


## <a name="xcast_requestview">请求画面</a>


## <a name="xcast_externalcapture">自定义采集</a>


## <a name="xcast_outin">上下行角色权限控制</a>

## <a name="xcast_render">画面渲染</a>

### <a name="xcast_preview">画面预览</a>


## <a name="xcast_changerole">角色切换/上下麦</a>

## <a name="xcast_render">美颜接入</a>

## <a name="xcast_exception">异常处理（如后台，中断等事件）</a>


## <a name="xcast_event_detail">事件详细处理</a>


## <a name="xcast_event_detail">原QAVEndpoint事件通知</a>

## <a name="xcast_error_code">错误码整理</a>

## <a name="xcast_define_detail">xcast_define明细</a>

### <a name="xcast_define_format">xcast_define中需要格式化的参数</a>

```
#define XC_STREAM_STATE                     "stream.%s.state"    // xcast_start_stream中传入的streamid#define XC_STREAM_TRACK                     "stream.%s.track"    // xcast_start_stream中传入的streamid#define XC_TRACK_ENABLE                     "stream.%s.%s.enable"  // 第一个为，streamid，第二个为 [上行：video-out,audio-out,sub-video-out, 下行：video-in,audio-in,sub-video-in]#define XC_TRACK_ENABLE                     "stream.%s.%s.enable"#define XC_TRACK_STATE                      "stream.%s.%s.state"#define XC_TRACK_STATE                      "stream.%s.%s.state"#define XC_TRACK_CAPTURE                    "stream.%s.%s.capture"#define XC_TRACK_CAPTURE                    "stream.%s.%s.capture"#define XC_CAMERA_PREVIEW                   "device.camera.%s.preview"#define XC_CAMERA_STATE                     "device.camera.%s.state"#define XC_CAMERA_PREPROCESS                "device.camera.%s.preprocess"#define XC_MIC_STATE                        "device.mic.%s.state"#define XC_MIC_LOOPBACK                     "device.mic.%s.loopback"#define XC_MIC_VOLUME                       "device.mic.%s.volume"#define XC_SPEAKER_ENABLE                   "device.speaker.%s.enable"#define XC_SPEAKER_EARPHONE_MODE            "device.speaker.%s.earphone-mode"#define XC_SPEAKER_VOLUME                   "device.speaker.%s.volume"#define XC_DEVICE_EXTERNAL_INPUT            "device.external.%s.input"#define XC_DEVICE_EXTERNAL_TYPE            "device.external.%s.type"#define XC_DEVICE_EXTERNAL_STATE            "device.external.%s.state"#define XC_MIC_DYNAMIC_VOLUME               "device.mic.%s.dynamic-volume"#define XC_MIC_MIX                          "device.mic.%s.mix"#define XC_MIC_MIX_SYNC                     "device.mic.%s.mix-sync"

```








