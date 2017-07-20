# AVSDK日志中级查询技巧

## <a name="log_changecontrolrole">1. 切换角色/上下麦</a>
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


## <a name="log_avdata">2. 数据流变化</a>

## <a name="log_eventid">3. 用户事件</a>

## <a name="log_camera">4. 摄像头操作</a>

## <a name="log_mic">5. Mic操作</a>

## <a name="log_audioencdec">6. 音频编解码相关</a>

## <a name="log_videoencdec">7. 视频编解码相关</a>

## <a name="log_net">8. 网络情况相关</a>
1. 作用：查看用户的直播过程中网络包丢包情况，以及重传之后的丢包情况；如果丢包较严重。注意此处与[AVMonitor](http://avq.server.com/reportapp/)丢包率的区别：如果用户侧丢包严重，有可能上传的心跳包都丢失，这样AVMonitor在绘制时，默认的处理是填0, 有可能监控上看到的丢包率是0, 但实际看到的直播效果很差；
2. 关键字：`CurLostRate`可查网络包丢包情况 , `UDTS CalcSendLoss dwNoAckNum`可查重传之后的丢包情况
3. 示例日志：

```
//CurLostRate查到的示例日志

Line 114424: [2017/01/04 17:13:54.455| E| 916  | AVGCongestion.cpp(534):CheckLostRate_AfterACK     ] CWnd Size:52,CurLostRate:1016,MinLostRate:419,RTT:1012,RTTD:378,Available[2],MaxCwnd:210,MinCwnd:52Line 114448: [2017/01/04 17:13:54.893| E| 916  | AVGCongestion.cpp(534):CheckLostRate_AfterACK     ] CWnd Size:52,CurLostRate:1105,MinLostRate:469,RTT:1281,RTTD:418,Available[0],MaxCwnd:210,MinCwnd:52Line 114477: [2017/01/04 17:13:55.410| E| 916  | AVGCongestion.cpp(534):CheckLostRate_AfterACK     ] CWnd Size:52,CurLostRate:1032,MinLostRate:519,RTT:1752,RTTD:69,Available[0],MaxCwnd:251,MinCwnd:52
```
备注：

* 其中丢包率为：`CurLostRate`值，计算如：1016/100 = 10.16 ，即丢包率为10.16%；
* `RTT`为往返延时，单位ms

```
//UDTS CalcSendLoss dwNoAckNum查重传之后还有多少丢包

Line 114266: [2017/01/04 17:13:51.792| E| 916  | AVGUDTSend.cpp(560):InternalUDTCalcSendLoss       ] UDTS CalcSendLoss dwNoAckNum[4] dwSendTotalNum[107] dwSendLossRate[373]Line 114381: [2017/01/04 17:13:53.820| E| 916  | AVGUDTSend.cpp(560):InternalUDTCalcSendLoss       ] UDTS CalcSendLoss dwNoAckNum[7] dwSendTotalNum[124] dwSendLossRate[564]Line 114504: [2017/01/04 17:13:55.847| E| 916  | AVGUDTSend.cpp(560):InternalUDTCalcSendLoss       ] UDTS CalcSendLoss dwNoAckNum[4] dwSendTotalNum[141] dwSendLossRate[283]Line 114816: [2017/01/04 17:13:59.888| E| 916  | AVGUDTSend.cpp(560):InternalUDTCalcSendLoss       ] UDTS CalcSendLoss dwNoAckNum[15] dwSendTotalNum[164] dwSendLossRate[914]
```
备注：

* `dwSendTotalNum`为总重传发包数，`dwNoAckNum`为重传丢包数，`dwSendLossRate`为重传丢包率；计算规则：`dwSendLossRate` = `dwNoAckNum` ／ `dwSendTotalNum` * 10000;



## <a name="log_dumpaudio">9. 如何dump音频数据</a>


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
## <a name="log_dumpvideo">10. 如何dump视频帧数据</a>

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
3. 将`encode.264`使用工具`Elecard StreamEye`打开（工具较转业，可找mortyzhu看下）；




