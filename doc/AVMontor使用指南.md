# AVMonitor使用指南
AV_Monitor是帮助用户定位排除网络问题的实时监控工具。

其原理是：通过客户端的2秒心跳上报逻辑，将客户端的数据上报到后台，然后与接口机机统计到的数据进行汇总，最终以图表形式展现出来；

* 内部访问地址: http://avq.server.com/
* 外部客户地址: http://avq.avc.qcloud.com/monitor.html

外部客户访问的版本较内部使用版本功能有裁剪，本文以内部工具为主；

<img src="https://main.qcloudimg.com/raw/594aeb151f066c8655f7b2a9edc53208.png">

# 排查流程
向客户确认好以下必填信息信息：

* sdkappid 
* 帐号 ： 由业务侧分配，具体询问业务侧，或从客户提供的日志中解析identifier或tinyid(如果是tinyid注意沟选`is_tinyid`项);
* is_qcloud : 是否是云上环境
* 时间段 ：<font color="red">对应出问题的**北京时间**，如果用户在国外，注意时区，如果是日志时间注意结束时区偏差进行比对;  **AVM最多只能查近七天的信息**，单次查询最大时间段是1小时;</font>



以蘑菇街案例讲解: http://avq.server.com/#&sdkappid=1400006909&userid=14y6vg8&env=&from=2018-08-22%2015:00:00&to=2018-08-22%2016:00:00
<img src="https://main.qcloudimg.com/raw/99e2fb8286ad60d0769675a3d72d4443.png">


# 上行质量监控图表

## 上行总丢包率

### 信息中心分配信息
红包点

* 业务类型:开放业务
* 鉴权模式:6
* SdkAppId:1400006909
* 群组号码:10712710				： <font color="red">对应日志enterroom时传入的房间号，</font>						
* 房间号:991404
* uin:144115197437872527		： 用户对应的tinyid
* 是否代理机:否						:  否 ： DC  是 ： OC
* 网络协议:UDP			
* 传输协议:UDT						:  <font color="red">互动直播下 通常主播类型都是UDT，如果使用<a href="http://video.isd.com/ilvb/spear">运营系统</a>查到用户的是互动直播，但传输协议使用的是UDP, 说明用户对应的进房角色的网络配置`重传抗丢包`关闭了</font>
* 客户端ip:122.224.77.194		:  可用<a href="http://ip138.com/">ip138</a>确认用户区域
* 接口机ip:101.226.235.37		： 可用<a href="http://ip138.com/">ip138</a>确认分配置的接口机位于何处
* 流控ip:10.49.97.137			： 该地址为局域网地址，可用<a href="tnm2.oa.com">网管系统</a>查询是否进入到测试环境，如下图划线处显示`测试`字样，即为进入到测试环境
										 <img src="https://main.qcloudimg.com/raw/ed2b50c1fdfea7dcc01593336923e918.png">
										 
### 进房间带上的能力
流控信息区展示： 可用于确认出问时进房的配置是否与spear匹配，<font color="red">通常没有拉到配置，使用sdk默认配置会从这里看出来 (主要通过音视频参数可看出)</font> 

```
时间:Invalid Date
------------------
通用能力:			客户端的基本信息
|-CPU名称:aarch64 processor rev 2 (aarch64)|mt6750
|-CPU最大频率:1508MHZ
|-CPU核数:8
|-CPU L2:0
|-CPU L3:0
|-网络类型:WIFI
|-终端类型:andriod
|-操作系统:android 
|-SDK版本:1.9.9.1.release- 38855
------------------
音频能力:
|-版本:2
|-最多混音路数:0
------------------
视频能力:
|-屏幕宽:1280
|-屏幕高:720
|-ARQ:不支持
|-卡尔曼:不支持
|-QSTAR:不支持
|-硬件加速:不支持
------------------
特殊视频能力:
|-编码协议:VE_H264_EXTENDED
|-宽:960
|-高:540
|-帧率:25
|-码率:0kbps
|-最大QP:0
|-最小QP:0
|-GOP:5
|-FEC开关:关闭
|-最大宽:0
|-最大高:0
|-最大码率:1600kbps
|-最小码率:1400kbps
|-小视频开关:关闭
|-主播自适应开关:关闭
------------------
特殊音频能力:
|-编码协议:AAC						
|-采样率:48000
|-声道:2
|-音频包间隔:40ms
|-音频码率:24000bps
|-AEC:打开
|-AGC:关闭
|-ANS:打开
|-VAD:关闭
|-FEC:关闭
------------------
网络能力:
|-udt重传最大延时:900
------------------

```

### 推流信息
深蓝色点：如果业务当前有推流，可通过推流地址，确认用户具体现象（主要有花屏,卡顿,变型等现象）, 注意相应的事件类类型通知：`uint32_oper`

```
uint32_oper: 1
uint32_live_code: 6
uint32_sdk_type: 1
str_channel_name: ""
str_channel_describe: ""
str_player_pwd: ""
uint32_push_data_type: 0
uint32_tape_flag: 1
uint32_push_duration: 0
uint32_watermark_flag: 0
uint32_watermark_id: 0
uint32_result: 0
msg_live_url {
  uint32_type: 5
  string_play_url: "rtmp://2345.liveplay.myqcloud.com/live/2345_5adeb4b0a5db11e892905cb9018cf0d4"
  uint32_rate_type: 0
}
msg_live_url {
  uint32_type: 2
  string_play_url: "http://2345.liveplay.myqcloud.com/live/2345_5adeb4b0a5db11e892905cb9018cf0d4.flv"
  uint32_rate_type: 0
}
msg_live_url {
  uint32_type: 1
  string_play_url: "http://2345.liveplay.myqcloud.com/live/2345_5adeb4b0a5db11e892905cb9018cf0d4.m3u8"
  uint32_rate_type: 0
}
str_errorinfo: ""
uint64_rsp_channel_id: 10905947996257141754
uint32_tape_task_id: 0

```

### 录制信息

青色点：可用于确认业务的录制情况，以及回调等信息, 如下为录制失败, 注意相应的事件类类型通知：`uint32_oper`

```
uint32_oper: 2
string_file_name: ""
uint32_classid: 0
uint32_is_trans_code: 0
uint32_is_screen_shot: 0
uint32_is_water_mark: 0
uint32_sdk_type: 1
uint32_record_data_type: 0
uint32_record_appid: 0
uint32_result: 30000412
str_errorinfo: "record already stoped!"

```

### 退出房间信息

蓝色点： 鼠标停在上面时，会提示相应的是正常退出，还是超时退出(对应可查是否长时间没有心跳包);

### 视频位改变
粉色点：代码用户获取到视频可以上行

### 下发调控

<img src="https://main.qcloudimg.com/raw/e6e4141fd1f04ea5fa0bfa977be19bc4.png">

```
时间:2018/8/22 下午3:21:25
------------------
音频调控:
|-采样率:48000
|-编码协议:AAC
|-声道:双声道
|-码率:24000bps
|-组包时间间隔:40ms
|-综合丢包率:0
|-FEC M:0
|-FEC N:4
|-音频包MTU:0
|-是否开启AEC:true
|-是否开启AGC:false
|-是否开启NS:true
|-是否开启DTX:true
|-是否开启VAD:false
|-卡尔曼阀值:80
------------------
视频调控:
主路大画面调控:
|-编码协议:VE_H264_EXTENDED
|-编码宽:960
|-编码高:540
|-编码帧率:25
|-编码码率:1400kbps
|-编码GOP类型:VGT_I_P_P_P
|-编码GOP帧数:20
|-I帧FEC百分比:45
|-SP帧FEC百分比:0
|-视频包MTU:1100
|-硬件加速开关:开启
------------------
网络调控:
|-udt重传最大延时:900
```

流控主要是预测客户端网络变化时针对性地调优;

正常情况下（网络无丢包无抖动时）, 下发流控较少(一般主要集中进房后);
如果网络较差时，会经常性出现下发流控；

流控主要看以下指标：
I帧FEC : 值越大，画面就越糊；正常时，值是20
视频分辨率: 主要反馈上行过程中，上行端编码情况。如用户反馈的视频过程中画面分辨率经常改变（尤其是使用混流的用户最敏感）
视频码率/视频帧率 : 可通过其他图表项展现，此处不作详解；


### 客户端IP变更
黄色点 : 如果进房上行过程中，出现黄色点，说明用户有切换过网络;

### 2s上报详细信息
	
<img src="https://main.qcloudimg.com/raw/666088357ca1f92dff4e7f759137217b.png">

* <font color="red">心跳包间隔2秒一个，如果网络出现抖动或丢包，心跳曲线会变动厉害，同时其他项曲线也会出现跳变到0的情况；</font>
* <font color="red">心跳包绿色空心点表示软编解，红色实心点表示硬编解；</font>
* <font color="red">如果心跳包大面积出现间隔在1秒左右，很有可能是有两个终端在同时登录，那此时速个监控的信息是不可靠的</font>

### UDT上行总丢包率

<img src="https://main.qcloudimg.com/raw/a7ecf743d00e6ce263793bc7020cd40c.png">

* UDT丢包率只在实时通信以及互动直播下有，实时通信下不可用


### SDK统计的上行视频丢包率
<img src="https://main.qcloudimg.com/raw/daf00ebba8c2b099427cad5bca750d92.png">

* 只在进房内有效，有应收包数据/实收包数才可以用于参考;
* 真实反应用户网络的丢包情况；
* 如果是规律性的丢包，可以查下流控信息，以及让用户确认下用户自己的路由是否有做限频;

### 接口机统计到的丢包率
<img src="https://main.qcloudimg.com/raw/c241a87c084f9d59c03d64f42cba9281.png">

* 只在进房内有效，有应收包数据/实收包数才可以用于参考;
* 反应接口机收包情况（主要是运营商到接口机间的情况？？）；
* 如果出现严重的丢包，可以使用<a href="http://avq.server.com/reportapp/totalmonitor/">接口机监控</a>查到对应时间段接口机的情况; 如果同一时候段其他业务或者同一业务下其他用户的情况，对于对比分析;

## 上行总码率
<img src="https://main.qcloudimg.com/raw/ffc747fc03be223e7b76af4d674ab98b.png">

如果该值波动较大，不平滑，说明用户的网络有波动；可以查 `下发调控` `SDK统计的丢包率` `延时进行`核对

## 时延
* 国内延时基本在 50ms以内(3G/4G延时相对要高些)，国外的一般在200ms；
* 一般互动直播的延时buf是 250--900ms，如果延时较大（>900ms???）可理解延时较大

## 上行大画面帧率
* 主播的编码帧率过低时，可以理解画面出现卡顿；
* 帧率与`应用的CPU使用率` `设备的CPU使用率`相关，可以关联一起看；

## 上行大画面码率
* 上行大画面码率，即为摄像头上行码率；
* 该项只能反馈有数据上行，越平滑越好，如果波动大时，可结合丢包，以及流控查看；
* 上行画面是否正常（如用户反馈观众看言播画面画屏），可根据推流信息直接查看旁路效果，或使用录制信息进运营系统查看成的录制文件；

## 上行辅路帧率
辅路一般是屏幕分享或都播片上行的流，帧率一般都较低（15fps以内）;

## 上行辅路码率
辅路画面一般都是静态画面居多（静态画面编出来的码率小），码率一般不超过800kbps(SDK内部屏幕分享最在上行);

## 上行音频码率
1. 声音码率波动越大，声音效果越不好；
2. 声音码率波动大，也在可看下发流控的音频FEC (正常情况上FC),以及上行丢包;


# 下行质量监控图表

## 下行总丢包率
此处丢包与上行丢包信息是相同的，AVM为方便查看，在些片同样也显示了，其用法同上行丢包率

## 下行视频总码率 & 下行大画面总帧率 & 下行总码率
下行可能有多路，如果有多路，此几项可简单判定络端是否有下行。建议以接下来几项做参考会较准确；

## 下行大画面码率

## 下行辅路总帧率
当用户反馈听到下行画面有卡，可查看下行帧率是多少，同时找到对应上行用户的帧率两边对比：1. 如果上行本身就有问题，即是上行端的问题；2.如果上行端没有问题，再确认本地丢包；
## 下行辅路总码率

## 下行音频码率

# 其他监控图表

## 房间在线用户线  &  房间最高在线用户数
可根据该两项，确认房间中是否有其他人，然后向用户拿到其他用户的信息，对于对比；




## 应用的CPU使用率 & 设备的CPU使用率
1. 有些设备能禁止获取cpu信息权限，这样在监控上显示就会为0
2. CPU使用率只是其中一核的数据；
2. 对于一些卡顿问题，可在进房时拿到其设备信息，然后结合期spear配置，以及cpu信息一起看；
3. 之前有例case: PC上CPU到40%就会反过来制约编码；
