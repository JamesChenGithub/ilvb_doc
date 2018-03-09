# XCast对接流程




##  <a name="xcast_beforestart">准备工作（optional）</a>
### <a name="xcast_account">帐号接入方式</a>
1. IMSDK试接入 : IMSDK做登录，登录成功后使用对应帐号的tinyid作帐号传入xcast;
2. 非IMSDK方式接入 : 业务层将自有自符串帐号映身成数字型帐号，然后传入xcast;  <font color="red">并做好鉴权逻辑（缺失如何鉴权 （authbuf））</font>

 <font color="red">注意此步跟后续的<a href="#xcast_auth_type">鉴权方式</a>有关；</font>

### <a name="xcast_spear">spear配置方式</a>
1. 静态配置：流览器拼spear url , 获取到spear的json串，以{$sdkappid}.conf方式存储到<font color="red">指定目录（各平台对应目录未确认）</font>;
2. 动态更新配置：业务层根据现有spear缓存逻辑进行请求并缓存到 以{$sdkappid}.conf方式存储到<font color="red">指定目录（各平台对应目录未确认）</font>;


## <a name="xcast_flow">xcast运行流程</a>

<img src="vipkid_flow.png">

## <a name="xcast_data_t"> xcast\_data\_t基础参数类介绍</a>

* xcast API基本上都是使用key（命令字）- value（命令参数）方式控制底层逻辑，具体key-value协议可查询 `xcast_define.h`（可理解成协议文件），xcast内部参数说明也在该文件中；
* 在`xcast_define.h`中我们可以看出，基本上所有的value（命令参数）均是用类似json格式描述，接口对应的类型为`xcast_variant_t` （C接口），实际使用的是其封装 `xcast_data_t`（C++接口，xcast_data_t 重载*操作符实现转换逻辑，对于开发者实际使用时只需要知道 `xcast_data_t`即可）；
* `xcast_variant_t` 转换成 `xcast_data_t`

```
// xcast_variant_t 转  xcast_data_t
xcast_variant_t *e;

//  xcast_data_t 字符串类内查看
xcast_data_t data;
...
char *datastr =  data.dump();
printf("%s\n", datastr);


```

## <a name="xcast_integrate_flow">xcast接入流程</a>

### <a name="xcast_startup">xcast\_startup</a> (等同于AVSDK StartContext)
xcast_startup接口声明如下：

```
/*
* 
*  settings example : ["identifier":uint64(67890),"test_env":false,"app_id":int32(1400046799)] 
* `identifier` : 帐号，<a href="#xcast_account">帐号接入</a>中获取得的内容；
* `test_env` :  是否是测试环境  
* `app_id` : 即对应的SDKAPPID；
* 
* return ： 0 : 成功， 其他为失败；

```


示例代码：

```
xcast_data_t settings;

rt = xcast_startup(settings);
{  
	// startup成功
	_is_startup_succ = true;
else 
{
	// startup失败
}

```

注意事项：<font color="red"> 业务层记录下是否有 xcast_startup  成功过 </font>， 如示例代码中的`_is_startup_succ`;


###  <a name="xcast_handle_event">xcast\_handle_event </a> (注册事件监听)

接口说明如下：

```
/*
* 参数说明：
* event_path ： 在`xcast_define.h`中，其以 `XC_EVENT_`开头，如 XC_EVENT_SYSTEM，不能为空
* handler : 回调函数，如果取消对应event_path的事件监听，可以传NULL。不为NULL时，监听 event_path， 为NULL时，取消监听
* user_data : 上下文信息，会在 handler中作参数返回
* 返回值：0 成功，非0 , 失败

```

注意事项：<font color="red"> user_data为上下文信息，下同 </font>;


#### <a name="xcast_handle_event_protocol">事件监听回调格式说明</a>

```
/* xcast事件列表 */
```

#### <a name="xcast_handle_event_detail">设置监听事件</a>
<font color="red">可以监听有事件说明在`xcast_define.h`中，其以 `XC_EVENT_` 开头</font>，即以下几种类型：

```


```


#### <a name="xcast_handle_event_regist">注册监听事件</a>
示例代码：

```
// 1. 声明处理回调函数；
static int32_t on_xcast_event(void *user_data, xcast_variant_t *e);


// 2. 注册回调 
rt = xcast_startup(settings);
{  
	// startup成功
	
	/* 注册事件通知回调 */
	// user_data 可以为当前上下文信息，具体情况以业务为准，可为NULL
	xcast_handle_event(XC_EVENT_STREAM, (xcast_func_pt)on_stream_event, user_data);
	
	_is_startup_succ = true;
else 
{
	// startup失败
}

```

#### 监听回调处理

处理回调前，可用该方法打信回调信息打印出来，然后对着协议进行解析

```

void xc_print_variant(xcast_data_t evt)
{
   
		const char             *err_msg;
	  
	  
	  	// 找印回调信息
	 	 char *evt_str = evt.dump();
	 	 if (evt_str)
	 	 	printf("%s\n", evt_str);
 	 
}
```

##### <a name="xcast_handle_event_systemeventcallback">系统事件回调处理</a>

回调格式说明

```
// 系统事件回调内容格式
* xcast支持的系统事件


/* xcast system events */

```

示例代码：

```
/* xcast系统事件通知 */
  
  
  // 找印回调信息
 xc_print_variant(evt);
 // TODO: 将错误信息转给业务上层进一步处理

```

#####  <a name="xcast_handle_event_streameventcallback">流事件回调处理</a>


```
/*
* xcast支持的媒体流事件
* "event.stream":{
*   // 事件源:媒体流名称
*   "*src":"vstring",
*   // 事件类型: 新增，更新，删除
*   "*type":[xc_stream_event],
*   // 事件状态
*   "state":[xc_stream_state],
*   // 错误代码
*   "err":"vint32",
*   // 错误信息
*   "err-msg":"vstring"
* },
*/
#define XC_EVENT_STREAM                     "event.stream"

// 流事件类型: 新增，更新，删除
/* xcast stream events */



```
示例代码：

```
/* 流状态通知： 更新UI中媒体流状态 */
  
  // 根据流事件类型处理
   	 TODO：
  }
    {
      TODO： 流更新成功
    // TODO：通知上层流关闭了

```

##### <a name="xcast_handle_event_trackeventcallback">流轨道事件回调处理</a>

```
/*

// 协议中用到的枚举类型说明
/* xcast stream track events */

/* xcast media types */

```

<a name="xcast_handle_event_trackeventcallback_samplecode">示例代码</a>

```
/* 流轨道状态通知： 更新UI中媒体流轨道状态 */
  
  
  
  // 业务侧可根据返回的  type,  direction自己做业务上的过滤
 
  	/* 移除轨道 */
  	// 收到数据？

```

##### <a name="xcast_handle_event_deviceeventcallback">设备事件回调处理</a>


回调协议说明：

```

/*


/* xcast device running state */

```

<a name = "xcast_device_event_samplecode">示例代码</a>

```
/* 设备事件通知： 更新UI中设备状态 */
    // 通过该回调可以拿到摄像头采集的数据，交由业务层上渲染
```

##### <a name="xcast_handle_event_tipseventcallback">统计信息回调处理</a>

回调格式：

```
/*

```

求例代码

```
static int32_t
    // TODO：显示到UI上，或者回调给业务层


```

### <a name="xcast_start_stream">xcast\_start_stream</a> (等同于AVSDK EnterRoom)

接口声明

```
/* 
* id : 业务层流标识，可以任意指定，但为了便于管理，建议以上行用户的id作标识；
* params 参数说明：
* role : spear上对应的角色；
* relation_id ： 房间号
* auto_recv : 自动接收，为true时，进房自动下发视频，如果有新上线的用户，也会自动去请求；为 false则不处理；
* videomaxbps ：[optional]投屏专用
* ayth_type : 参考 xc_auth_type_s
* typedef enum xc_auth_type_s {
* auth_info : 鉴权信息，注意与ayth_type结合使用
* auth_info.auth_buffer : auth_buffer ，手动鉴权情况下使用；
* auth_info.account_type ：帐号类型；
* auth_info.expire_time : 过期时间, 单位s
* auth_info.secret_key : 权限密钥
* auth_info.auth_bits : 权限位 ： 可参考AVSDK的用法，
* #define QAV_AUTH_BITS_DEFAULT 0xFFFFFFFFFFFFFFFF ///< 缺省值。拥有所有权限。
* #define QAV_AUTH_BITS_OPEN 0x000000FF            ///< 权限全开
* #define QAV_AUTH_BITS_CLOSE 0x00000000           ///< 权限全关
* #define QAV_AUTH_BITS_CREATE_ROOM 0x00000001     ///< 创建房间权限。
* #define QAV_AUTH_BITS_JOIN_ROOM 0x00000002       ///< 加入房间的权限。
* #define QAV_AUTH_BITS_SEND_AUDIO 0x00000004      ///< 发送语音的权限。
* #define QAV_AUTH_BITS_RECV_AUDIO 0x00000008      ///< 接收语音的权限。
* #define QAV_AUTH_BITS_SEND_VIDEO 0x00000010      ///< 发送视频的权限。
* #define QAV_AUTH_BITS_RECV_VIDEO 0x00000020      ///< 接收视频的权限。
* #define QAV_AUTH_BITS_SEND_SUB 0x00000040        ///< 发送辅路视频的权限。
* #define QAV_AUTH_BITS_RECV_SUB 0x00000080        ///< 接收辅路视频的权限。
* 自定义采集[optional]：
* track.ext-video-capture : 外部摄像头采集，bool
* track.ext-audio-capture : 外部音频采集, bool
* track.ext-audio-playback : 外部播放，bool
* 局域网设置[optional]：暂不添加

```

<a name="xcast_auth_type"></a>示例代码：

```

// 参数配置

	xcast_data_t       params, auth_info, track;
	params["auto_recv"] = true;
	params["role"]="LiveGuest";
	
	
	// 监权方式： 设置鉴权方式，以下三种选其一即可：
	
	// 方式1. 无鉴权，白名单方式
	params["auth_type"] = xc_auth_none;
	auth_info["auth_bits"] = -1;
	
	// 方式2. 手动鉴权
	params["auth_type"] = xc_auth_manual;
	auth_info["auth_bits"] = -1;
	auth_info.put_bytes("auth_buffer", (const uint8_t*)"xxxx", 4);  // "xxxx" 为业务生成auth_buffer
	
	// 方式3. 自动鉴权
	params["auth_type"] = xc_auth_manual;
	auth_info["auth_bits"] = -1;
	auth_info["account_type"] = 18454;
	
	auth_info.put_bytes("secret_key", (const uint8_t *)secret_key, (uint32_t)strlen(secret_key));

	xc_print_variant(params);
   		// TODO：进房失败；
   else
   {
   		// 接口调用成功
   }
   
   // 等待stream事件回调

```

调用xcast_start_stream后，会收到两次回调（开始连接，连接成功）如下，具体可在  <a href="#xcast_handle_event_systemeventcallback">流事件回调处理</a>中查看

```
第一次 ： state ： 1(xc_stream_connecting） 准备链接 ；  回调示例 ：["state":int32(1),"err":int32(0),"type":int32(1),"src":"67890"] 

```


### <a name = "xcast_close_stream">xcast\_close_stream</a> (等同于AVSDK ExitRoom)

接口声明：

```
/* 
* id : 与xcast_start_stream时传入的相同
* 返回值 ： 0 成功，非0失败

```

调用xcast_close_stream后，会收到一次回调（断开连接）如下，具体可在  <a href="#xcast_handle_event_systemeventcallback">流事件回调处理</a>中查看

```
 state ： 3(xc_stream_closed） 断开链接 ；  回调示例 ：["state":int32(3),"err":int32(0),"type":int32(1),"src":"67890"] 

```


### <a name = "xcast_shutdown">xcast\_shutdown</a> (等同于AVSDK stopContext)
接口声明：

```
/*

```

示例代码：

```
	// 如果当前正在推流
 	if (_is_start_streaming)
 	{
 		// 停止当前流
 		xcast_close_stream(streamid);
 	}
 	
 	xcast_shutdown();


```
注意事项：
1. 调用`xcast_shutdown`，内部会注销所有回调监听；
2. 如果当前有`xcast_start_stream`, 在没有`xcast_close_stream`前提下，也可以调用`xcast_shutdown`，但是这样不会收到`xcast_close_stream`的回调，建议业务层在调用`xcast_close_stream`后再`xcast_shutdown`，或者在`xcast_close_stream`回调里面再`xcast_shutdown`；




<!--### event\_callback

#### system

#### stream

#### track

#### devices

#### statistic_tips-->


<!--## <a name ="other">其他</a>

### <a name ="xcast_get_property">xcast\_get_property</a> (kvc/kvo方式)
### <a name ="xcast_set_property">xcast\_set_property</a> (kvc/kvo方式)-->



<!--## 内部知晓
对用xcast的用户开白名单 （xcast使用的是tinyid，开白名单之后，推流录制才可以查） -->