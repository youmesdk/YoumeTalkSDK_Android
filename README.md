# Talk SDK for Android 使用指南

## 适用范围
本文档适用于游密实时语音引擎（Talk SDK）Android Studio开发环境下接入。

## SDK目录概述
语音SDK中有lib文件夹，该文件夹下又包含几种cpu系统架构的动态库文件以及youme_voice_engine.jar包。

## Android Studio开发环境集成

1. 将SDK内的lib文件夹的所有文件移至Android工程libs文件夹下（可视实际情况自行放置），如下图所示：

  ![](https://youme.im/doc/images/android_libs_view.png)

  ![](https://youme.im/doc/images/talk_sdk_android_guide_2.png)

  ![](https://youme.im/doc/images/talk_sdk_android_guide_3.png)
 
2. AndroidManifest.xml配置
##### 添加录音和网络相关权限
  ``` xml
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
    <uses-permission android:name="android.permission.CHANGE_WIFI_STATE" />
    <uses-permission android:name="android.permission.CHANGE_NETWORK_STATE" />
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
    <uses-permission android:name="android.permission.RECORD_AUDIO" />
    <uses-permission android:name="android.permission.MODIFY_AUDIO_SETTINGS" />
    <uses-permission android:name="android.permission.READ_PHONE_STATE" />
    <uses-permission android:name="android.permission.BLUETOOTH" />
  ```


 反混淆配置
##### YOUMESDK不需要混淆,如果你的工程有混淆  请在proguard.cfg文件中添加如下代码：

``` shell
    -keep class com.youme.**
    -keep class com.youme.**{*;}
    -keepattributes Signature
```
	


3. 设置加载so库以及启动相关服务
  在入口Activity类先导入package:   

``` java
  	import android.content.Intent;
  	import android.os.Bundle;
  	import com.youme.voiceengine.mgr.YouMeManager;
```


  	然后在onCreate方法里添加如下代码:

``` java
  	YouMeManager.Init(this);
    
  	super.onCreate(savedInstanceState);
```

4. 使用talkSDK实现语音通话


##### 接口调用基本流程 
`设置回调（参考api手册里面的实现回调）-> 初始化（init）->收到初始化成功回调通知（YOUME_EVENT_INIT_OK）`->`加入语音单频道（joinChannelSingleMode）->收到加入频道成功回调通知（YOUME _EVENT_JOIN_OK）`->`打开麦克风（SetMicrophoneMute（false））-> 收到麦克风已打开回调通知(YOUME_EVENT_LOCAL_MIC_ON)->打开扬声器（SetSpeakerMute（false））-> 收到扬声器已打开回调通知（YOUME_EVENT_LOCAL_SPEAKER_ON）`->`设置音量（SetVolume（70）（该音量建议70））->(到了前面一步已经可以和当前进入同一频道的人进行实时通话了)`

4.1 导入了sdk后,先implements  YouMeCallBackInterface
``` java
public class MainActivity extends AppCompatActivity implements YouMeCallBackInterface
```

然后重写里面的方法，其中里面的onEvent会得到关于所有Talk相关的回调


``` java
 	@Override
    public void onEvent(int eventType, int errorCode, String channelID, Object param) {

        log("OnEvent:event "+eventType + ",error " + errorCode + ",channel " + channelID + ",param_" + param.toString());
        Message msg = new Message();
        switch( eventType ){
            case YouMeEvent.YOUME_EVENT_INIT_OK: //YOUME_EVENT_INIT_OK:
                log("Talk 初始化成功");
                break;
            case YouMeEvent.YOUME_EVENT_INIT_FAILED://YOUME_EVENT_INIT_FAILED:
                log("Talk 初始化失败");

                break;
            case YouMeEvent.YOUME_EVENT_JOIN_OK://YOUME_EVENT_JOIN_OK:
                log("Talk 进入频道成功，频道："+channelID+" 用户id:"+param);

                break;
            case YouMeEvent.YOUME_EVENT_JOIN_FAILED://YOUME_EVENT_JOIN_FAILED:
                log("Talk 进入频道:"+channelID+"失败,code:"+errorCode);

                break;
            case YouMeEvent.YOUME_EVENT_LEAVED_ONE://YOUME_EVENT_LEAVED_ONE:
                log("Talk 离开单个频道:"+channelID);

                break;
            case YouMeEvent.YOUME_EVENT_LEAVED_ALL://YOUME_EVENT_LEAVED_ALL:
                log("Talk 离开所有频道，这个回调channel参数为空字符串");

                break;
            case YouMeEvent.YOUME_EVENT_PAUSED://YOUME_EVENT_PAUSED:
                log("Talk 暂停");
                break;
            case YouMeEvent.YOUME_EVENT_RESUMED://YOUME_EVENT_RESUMED:
                log("Talk 恢复");
                break;
            case YouMeEvent.YOUME_EVENT_SPEAK_SUCCESS://YOUME_EVENT_SPEAK_SUCCESS:///< 切换对指定频道讲话成功（适用于多频道模式）
                break;
            case YouMeEvent.YOUME_EVENT_SPEAK_FAILED://YOUME_EVENT_SPEAK_FAILED:///< 切换对指定频道讲话失败（适用于多频道模式）
                break;
            case YouMeEvent.YOUME_EVENT_RECONNECTIN://YOUME_EVENT_RECONNECTING:///< 断网了，正在重连
                log("Talk 正在重连");
                break;
            case YouMeEvent.YOUME_EVENT_RECONNECTED://YOUME_EVENT_RECONNECTED:///< 断网重连成功
                log("Talk 重连成功");
                break;
            case YouMeEvent.YOUME_EVENT_REC_PERMISSION_STATUS://YOUME_EVENT_REC_FAILED:///< 通知录音启动失败（此时不管麦克风mute状态如何，都没有声音输出）
                log("录音启动失败，code："+errorCode);
                break;
            case YouMeEvent.YOUME_EVENT_BGM_STOPPED://YOUME_EVENT_BGM_STOPPED:///< 通知背景音乐播放结束
                log("背景音乐播放结束,path："+param);
                break;
            case YouMeEvent.YOUME_EVENT_BGM_FAILED://YOUME_EVENT_BGM_FAILED:///< 通知背景音乐播放失败
                log("背景音乐播放失败,code："+errorCode);
                break;
            case YouMeEvent.YOUME_EVENT_OTHERS_MIC_ON://YOUME_EVENT_OTHERS_MIC_ON:///< 其他用户麦克风打开
                log("其他用户麦克风打开,userid:"+param);
                break;
            case YouMeEvent.YOUME_EVENT_OTHERS_MIC_OFF://YOUME_EVENT_OTHERS_MIC_OFF:///< 其他用户麦克风关闭
                log("其他用户麦克风关闭,userid:"+param);
                break;
            case YouMeEvent.YOUME_EVENT_OTHERS_SPEAKER_ON://YOUME_EVENT_OTHERS_SPEAKER_ON:///< 其他用户扬声器打开
                log("其他用户扬声器打开,userid:"+param);
                break;
            case YouMeEvent.YOUME_EVENT_OTHERS_SPEAKER_OFF://YOUME_EVENT_OTHERS_SPEAKER_OFF: ///< 其他用户扬声器关闭
                log("其他用户扬声器关闭,userid:"+param);
                break;
            case YouMeEvent.YOUME_EVENT_OTHERS_VOICE_ON://YOUME_EVENT_OTHERS_VOICE_ON: ///< 其他用户进入讲话状态
                log("开始讲话,userid:"+param);
                break;
            case YouMeEvent.YOUME_EVENT_OTHERS_VOICE_OFF://YOUME_EVENT_OTHERS_SPEAKER_OFF: ///< 其他用户停止讲话
                log("停止讲话userid:"+param);
                break;
            case YouMeEvent.YOUME_EVENT_MY_MIC_LEVEL://YOUME_EVENT_MY_MIC_LEVEL: ///< 自己的麦克风的语音音量级别
                log("我当前讲话的音量级别是,数值："+errorCode);
                break;
            case YouMeEvent.YOUME_EVENT_MIC_CTR_ON://YOUME_EVENT_MIC_CTR_ON: ///< 自己的麦克风被其他用户打开
                log("自己的麦克风被其他用户打开，userid："+param);
                break;
            case YouMeEvent.YOUME_EVENT_MIC_CTR_OFF://YOUME_EVENT_MIC_CTR_OFF: ///< 自己的麦克风被其他用户关闭
                log("自己的麦克风被其他用户关闭，userid："+param);
                break;
            case YouMeEvent.YOUME_EVENT_SPEAKER_CTR_ON://YOUME_EVENT_SPEAKER_CTR_ON: ///< 自己的扬声器被其他用户打开
                log("自己的扬声器被其他用户打开，userid："+param);
                break;
            case YouMeEvent.YOUME_EVENT_SPEAKER_CTR_OFF://YOUME_EVENT_SPEAKER_CTR_OFF: ///< 自己的扬声器被其他用户关闭
                log("自己的扬声器被其他用户关闭，userid："+param);
                break;
            case YouMeEvent.YOUME_EVENT_LISTEN_OTHER_ON://YOUME_EVENT_LISTEN_OTHER_ON: ///< 取消屏蔽某人语音
                log("取消屏蔽某人语音，userid："+param);
                break;
            case YouMeEvent.YOUME_EVENT_LISTEN_OTHER_OFF://YOUME_EVENT_LISTEN_OTHER_OFF: ///< 屏蔽某人语音
                log("屏蔽某人语音,userid："+param);
                break;
            default:
                break;
        }
```

更多的事件状态码可以参考官网的错误码定义以及5.YouMeEvent类


4.2 调用初始化INIT 和SetCallback



	@Override
    protected void onCreate(Bundle savedInstanceState) {
	//设置回调监听对象,需要import com.youme.voiceengine.*; 以及implements YouMeCallBackInterface
		api.SetCallback(this);
        YouMeManager.Init(this);
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
	}

当onEvent返回一个YouMeEvent.YOUME_EVENT_INIT_OK的时候带包初始化成功


4.3 初始化成功调用joinChannelSingleMode方法进入语音频道

``` java
  	public  void joinroom(){
       	/**
         
          第一个参数是用户id  全局唯一的用户标识，全局指在当前应用程序的范围内
          第二个参数是房间id  全局唯一的频道标识，全局指在当前应用程序的范围内
          第三个参数，角色身份说明
        YOUME_USER_NONE = 0,         ///< 非法用户，调用API时不能传此参数
        YOUME_USER_TALKER_FREE = 1,      ///< 自由讲话者，适用于小组通话（建议小组成员数最多10个），每个人都可以随时讲话, 同一个时刻只能在一个语音频道里面
        YOUME_USER_TALKER_ON_DEMAND = 2, ///< 需要通过抢麦等请求麦克风权限之后才可以讲话，适用于较大的组或工会等（比如几十个人），同一个时刻只能有一个或几个人能讲话, 同一个时刻只能在一个语音频道里面
        YOUME_USER_LISTENER = 3,         ///< 听众，主播/指挥/嘉宾的听众，同一个时刻只能在一个语音频道里面，只听不讲
        YOUME_USER_COMMANDER = 4,        ///< 指挥，国家/帮派等的指挥官，同一个时刻只能在一个语音频道里面，可以随时讲话，可以播放背景音乐，戴耳机情况下可以监听自己语音
        YOUME_USER_HOST = 5,             ///< 主播，广播型语音频道的主持人，同一个时刻只能在一个语音频道里面，可以随时讲话，可以播放背景音乐，戴耳机情况下可以监听自己语音
        YOUME_USER_GUSET = 6,            ///< 嘉宾，主播或指挥邀请的连麦嘉宾，同一个时刻只能在一个语音频道里面， 可以随时讲话
        */
        api.joinChannelSingleMode("Userid","RoomID",5);
    }
```

当onEvent返回 `YouMeEvent.YOUME_EVENT_JOIN_OK` 的时候表示进入频道成功

4.4 进入频道成功后，调用接口打开扬声器setSpeakerMute(false)和麦克风setMicrophoneMute（false）以及音量设置setVolume(int max)

``` java
	//麦克风开启
        api.setMicrophoneMute(false);
	//扬声器开启
        api.setSpeakerMute(false);
	//音量设为100
        api.setVolume(70);
```

当onEvent返回相应的回调 `YouMeEvent.YOUME_EVENT_LOCAL_MIC_ON`（自己麦克风能被打开），`YouMeEvent.YOUME_EVENT_LOCAL_SPEAKER_ON`（自己的扬声器打开），表示当前已经打开了麦克风和扬声器（注意需要给权限）

到了这一步已经可以和另外的在当前频道的人进行对话了。


4.5  离开频道leaveChannelAll


`api.leaveChannelAll()` ,当调用leaveChannelAll();方法后  onEvent收到事件通知YouMeEvent.YOUME_EVENT_LEAVED_ALL后表示已经退出语音频道


5. YouMeEvent事件通知详细

``` java
public  class YouMeEvent {
    public  static final  int    YOUME_EVENT_INIT_OK=0;           // SDK初始化成功
    public static final  int    YOUME_EVENT_INIT_FAILED=1;           //SDK初始化失败
    public static final  int    YOUME_EVENT_JOIN_OK=2;               // 进入语音频道成功
    public static final  int   YOUME_EVENT_JOIN_FAILED=3;  ///< 进入语音频道失败
    public static final  int   YOUME_EVENT_LEAVED_ONE=4;  ///< 退出单个语音频道完成
    public static final  int   YOUME_EVENT_LEAVED_ALL=5;  ///< 退出所有语音频道完成
    public static final  int   YOUME_EVENT_PAUSED=6;  ///< 暂停语音频道完成
    public static final  int   YOUME_EVENT_RESUMED=7;  ///< 恢复语音频道完成
    public static final  int    YOUME_EVENT_SPEAK_SUCCESS=8;  ///< 切换对指定频道讲话成功（适用于多频道模式）
    public static final  int   YOUME_EVENT_SPEAK_FAILED=9 ;  ///< 切换对指定频道讲话失败（适用于多频道模式）
    public static final  int   YOUME_EVENT_RECONNECTIN=10; ///< 断网了，正在重连
    public static final  int    YOUME_EVENT_RECONNECTED=11; ///< 断网重连成功
    public static final  int   YOUME_EVENT_REC_PERMISSION_STATUS= 12; ///< 通知录音权限状态，成功获取权限时错误码为YOUME_SUCCESS，获取失败为YOUME_ERROR_REC_NO_PERMISSION（此时不管麦克风mute状态如何，都没有声音输出）
    public static final  int   YOUME_EVENT_BGM_STOPPED          = 13; ///< 通知背景音乐播放结束
    public static final  int   YOUME_EVENT_BGM_FAILED           = 14; ///< 通知背景音乐播放失败
    //YOUME_EVENT_MEMBER_CHANGE        = 15, ///< 频道成员变化
    public static final  int    YOUME_EVENT_OTHERS_MIC_ON        = 16; ///< 其他用户麦克风打开
    public static final  int   YOUME_EVENT_OTHERS_MIC_OFF       = 17; ///< 其他用户麦克风关闭
    public static final  int   YOUME_EVENT_OTHERS_SPEAKER_ON    = 18; ///< 其他用户扬声器打开
    public static final  int   YOUME_EVENT_OTHERS_SPEAKER_OFF   = 19; ///< 其他用户扬声器关闭
    public static final  int   YOUME_EVENT_OTHERS_VOICE_ON      = 20; ///< 其他用户进入讲话状态
    public static final  int   YOUME_EVENT_OTHERS_VOICE_OFF     = 21; ///< 其他用户进入静默状态
    public static final  int  YOUME_EVENT_MY_MIC_LEVEL         = 22;///< 麦克风的语音级别
    public static final  int   YOUME_EVENT_MIC_CTR_ON           = 23; ///< 麦克风被其他用户打开
    public static final  int   YOUME_EVENT_MIC_CTR_OFF          = 24; ///< 麦克风被其他用户关闭
    public static final  int  YOUME_EVENT_SPEAKER_CTR_ON       = 25; ///< 扬声器被其他用户打开
    public static final  int   YOUME_EVENT_SPEAKER_CTR_OFF      = 26; ///< 扬声器被其他用户关闭
    public static final  int  YOUME_EVENT_LISTEN_OTHER_ON      = 27; ///< 取消屏蔽某人语音
    public static final  int  YOUME_EVENT_LISTEN_OTHER_OFF     = 28; ///< 屏蔽某人语音
    ///
    public static final  int   YOUME_EVENT_LOCAL_MIC_ON = 29; ///< 自己的麦克风打开
    public static final  int   YOUME_EVENT_LOCAL_MIC_OFF = 30; ///< 自己的麦克风关闭
    public static final  int   YOUME_EVENT_LOCAL_SPEAKER_ON = 31; ///< 自己的扬声器打开
    public static final  int   YOUME_EVENT_LOCAL_SPEAKER_OFF = 32; ///< 自己的扬声器关闭

    public static final  int   YOUME_EVENT_GRABMIC_START_OK = 33; ///< 发起抢麦活动成功
    public static final  int   YOUME_EVENT_GRABMIC_START_FAILED = 34; ///< 发起抢麦活动失败
    public static final  int   YOUME_EVENT_GRABMIC_STOP_OK = 35; ///< 停止抢麦活动成功
    public static final  int  YOUME_EVENT_GRABMIC_STOP_FAILED = 36; ///< 停止抢麦活动失败
    public static final  int   YOUME_EVENT_GRABMIC_REQUEST_OK = 37; ///< 抢麦成功（可以说话）
    public static final  int  YOUME_EVENT_GRABMIC_REQUEST_FAILED = 38; ///< 抢麦失败
    public static final  int   YOUME_EVENT_GRABMIC_REQUEST_WAIT = 39; ///< 进入抢麦等待队列（仅权重模式下会回调此事件）
    public static final  int YOUME_EVENT_GRABMIC_RELEASE_OK = 40; ///< 释放麦成功
    public static final  int  YOUME_EVENT_GRABMIC_RELEASE_FAILED = 41; ///< 释放麦失败
    public static final  int  YOUME_EVENT_GRABMIC_ENDMIC = 42; ///< 不再占用麦（到麦使用时间或者其他原因）

    public static final  int  YOUME_EVENT_GRABMIC_NOTIFY_START = 43; ///< [通知]抢麦活动开始
    public static final  int  YOUME_EVENT_GRABMIC_NOTIFY_STOP = 44; ///< [通知]抢麦活动结束
    public static final  int   YOUME_EVENT_GRABMIC_NOTIFY_HASMIC = 45; ///< [通知]有麦可以抢
    public static final  int   YOUME_EVENT_GRABMIC_NOTIFY_NOMIC = 46; ///< [通知]没有麦可以抢

    public static final  int   YOUME_EVENT_INVITEMIC_SETOPT_OK = 47; ///< 连麦设置成功
    public static final  int   YOUME_EVENT_INVITEMIC_SETOPT_FAILED = 48; ///< 连麦设置失败
    public static final  int   YOUME_EVENT_INVITEMIC_REQUEST_OK = 49; ///< 请求连麦成功（连上了，需等待对方回应）
    public static final  int   YOUME_EVENT_INVITEMIC_REQUEST_FAILED = 50; ///< 请求连麦失败
    public static final  int  YOUME_EVENT_INVITEMIC_RESPONSE_OK = 51; ///< 响应连麦成功（被叫方无论同意/拒绝都会收到此事件，错误码是YOUME_ERROR_INVITEMIC_REJECT表示拒绝）
    public static final  int   YOUME_EVENT_INVITEMIC_RESPONSE_FAILED = 52; ///< 响应连麦失败
    public static final  int   YOUME_EVENT_INVITEMIC_STOP_OK = 53; ///< 停止连麦成功
    public static final  int  YOUME_EVENT_INVITEMIC_STOP_FAILED = 54; ///< 停止连麦失败

    public static final  int   YOUME_EVENT_INVITEMIC_CAN_TALK = 55; ///< 双方可以通话了（响应方已经同意）
    public static final  int    YOUME_EVENT_INVITEMIC_CANNOT_TALK = 56; ///< 双方不可以再通话了（有一方结束了连麦或者连麦时间到）

    public static final  int   YOUME_EVENT_INVITEMIC_NOTIFY_CALL = 57; ///< [通知]有人请求与你连麦
    public static final  int   YOUME_EVENT_INVITEMIC_NOTIFY_ANSWER = 58; ///< [通知]对方对你的连麦请求作出了响应（同意/拒绝/超时，同意的话双方就可以通话了）
    public static final  int  YOUME_EVENT_INVITEMIC_NOTIFY_CANCEL = 59;///< [通知]连麦过程中，对方结束了连麦或者连麦时间到
    ///
    public static final  int   YOUME_EVENT_SEND_MESSAGE_RESULT         = 60; ///< sendMessage成功与否的通知，param为回传的requestID
    public static final  int   YOUME_EVENT_MESSAGE_NOTIFY              = 61; ///< 收到Message, param为message内容

    public static final  int   YOUME_EVENT_SET_WHITE_USER_LIST_OK      = 62; ///< 对指定频道设置白名单成功，但可能有异常用户
    public static final  int   YOUME_EVENT_SET_WHITE_USER_LIST_FAILED  = 63; ///< 对指定频道设置白名单失败

    public static final  int   YOUME_EVENT_KICK_RESULT                 = 64;   ///< 踢人的应答
    public static final  int   YOUME_EVENT_KICK_NOTIFY                 = 65;   ///< 被踢通知   ,param: （踢人者ID，被踢原因，被禁时间）


    public static final  int  YOUME_EVENT_EOF = 1000;

    }

```

### 备注：
[详细接口介绍可查看“Talk SDK for Android API接口手册.md”文档](https://github.com/youmesdk/YoumeTalkSDK_Android/blob/master/Talk%20SDK%20for%20Android%20API%E6%8E%A5%E5%8F%A3%E6%89%8B%E5%86%8C.md)

Talk SDK常见问题->[TALK FAQ](https://github.com/youmesdk/wiki/blob/master/YoumeTalk_FAQ.md)

实际Demo可点击此处下载->[Talk Demo for Android](https://github.com/youmesdk/YoumeTalkDemo_Android)
