# NERtcCallKit-Android
为了方便开发者接入音视频2.0呼叫功能，我们将NIN的信令和NERTC的音视频能力结合，以组件的形式提供给客户，提高接入效率，降低接入成本。[Demo传送门](https://github.com/netease-im/NIM_Android_Demo/tree/dev_g2)

## 功能开通

### 1. 登录

网易云控制台，点击【应用】>【创建】创建自己的App，在【功能管理】中申请开通如下功能

1. 若仅使用呼叫功能，则开通
   1. 【信令】
   2. 【音视频通话2.0】
   3. 【非安全模式】-组件默认使用非安全模式，开启安全模式请咨询SO
2. 若还需使用话单功能，则需要开通
   1. 【IM】
   2. 【G2话单功能】-目前仅支持联系销售/SO进行开通
      - 发送邮件至hzcaojiajun@corp.netease.com
      - 抄送hzsuzhongbin@corp.netease.com、hzliuxuanlin@corp.netease.com、wangjiangwen@corp.netease.com、zhangguanglu@corp.netease.com、hzyushaohua@corp.netease.com
      - 邮件内容：appkey、功能名称：G2话单消息通知

3. 在控制台中【appkey管理】获取appkey。

注：如果曾经已有相应的应用，可在原应用上申请开通【音视频通话2.0】及【信令】功能



## 整体架构图



![img](https://netease-we.feishu.cn/space/api/box/stream/download/asynccode/?code=e9f5e1dacd02782e23f257543f4e1cc3_8f118824ce50c961_boxcnK7D7XRErHIY9habMpiZHig_khhDdQLSWfkYrJbG7wCRKayms7i2Uy6V)

### 集成方式

* Maven集成

> implementation 'com.netease.yunxin.kit:call:1.1.0'
```groovy
allprojects {
    repositories {
        //...
        mavenCentral()
       //...
    }
}
```

* 手动集成

> clone 工程代码，以library的形式导入app项目
>
> 注意更改setting文件
>
> 注意组件中的IM版本和信令版本和APP保持一致（如果APP需要IM的其他功能）
>

### 组件结构

- nertcVideoCall 文件夹：音视频管理类，包含初始化，登录，呼叫，邀请等逻辑的相关操作管理
  - bean 呼叫所需数据元，需要混淆时keep
  - model 主要功能实现
  - push 推送相关配置
  - service 呼叫相关服务，确保应用收到呼叫时换气相关接听界面
  - utils 其他

### 代码示例

#### 初始化

```java
/**
 * 初始化，需要且仅能调用一次
 *
 * @param context
 * @param appKey
 * @param option
 */

NERTCVideoCall.sharedInstance().setupAppKey(getApplicationContext(), appKey, new VideoCallOptions(null, new UIService() {
    // 点对点音频页面
    @Override
    public Class getOneToOneAudioChat() {
        return NERTCVideoCallActivity.class;
    }

    // 点对点视频视频
    @Override
    public Class getOneToOneVideoChat() {
        return NERTCVideoCallActivity.class;
    }

    // 群聊页面
    @Override
    public Class getGroupVideoChat() {
        return TeamG2Activity.class;
    }

    @Override
    public int getNotificationIcon() {
        return R.drawable.ic_logo;
    }

    @Override
    public int getNotificationSmallIcon() {
        return R.drawable.ic_logo;
    }
}, ProfileManager.getInstance()));
```

#### 登录

> **组件与IM的login方法可共用，如果IM登录成功，组件可不在调用login方法**

```java
/**
 * IM 登录,使用前需要登录，最好在初始化后登录
 *
 * @param imAccount im账号
 * @param imToken   im 密码
 * @param callback  登录回调
 */
NERTCVideoCall.sharedInstance().login(imAccount, imToken, new RequestCallback<LoginInfo>() {
    @Override
    public void onSuccess(LoginInfo param) {

    }

    @Override
    public void onFailed(int code) {

    }

    @Override
    public void onException(Throwable exception) {

    }
});
```



#### Token注入

> // 安全模式音视频房间token获取，非安全模式可在callback中返回null，callback方法回调需要在主线程进行。
```java
//注册获取token的服务
//在线上环境中，token的获取需要放到您的应用服务端完成，然后由服务器通过安全通道把token传递给客户端
//Demo中使用的URL仅仅是demoserver，不要在您的应用中使用
//详细请参考: 获取安全模式token
NERTCVideoCall.sharedInstance().setTokenService((uid, callback) -> {
    String demoServer = "业务服务器URL";
    new Thread(() -> {
        try {
            // 请求业务服务器，由业务服务器，调用云信HTTPS接口，获取到Token回传
        } catch (Exception e) {
            e.printStackTrace();
        }
```



#### 呼叫

> Demo中在 AVChatAction 调用 startAudioVideoCall 方法，传入当前的通话类型，以及用户信息，启动相应的通话，NERTCVideoCallActivity 调用 callOut 方法


```java
/**
 * C2C邀请通话，被邀请方会收到 {@link NERTCCallingDelegate#onInvited } 的回调
 *
 * @param userId              被邀请方
 * @param selfUserId          自己的用户Id
 * @param type                1-语音通话，2-视频通话
 * @param joinChannelCallBack channel 回调
 */

nertcVideoCall.call(callOutUser.imAccid, selfUserId, type, new JoinChannelCallBack() {
    @Override
    public void onJoinChannel(ChannelFullInfo channelFullInfo) {
    // ChannelFullInfo 为信令的对象，音视频的为NERtcCallback的onJoinChannel
    }

    @Override
    public void onJoinFail(String msg, int code) {

    }
});

/**
 * 多人邀请通话，被邀请方会收到 {@link NERTCCallingDelegate#onInvited } 的回调
 *
 * @param ArrayList<String> callUserIds         被邀请方
 * @param String selfUserId         自己的用户Id
 * @param ChannelType type     1-语音通话，2-视频通话
 * @param joinChannelCallBack channel 回调
 */
NERTCVideoCall.sharedInstance().groupCall(accounts, ProfileManager.getInstance().getUserModel().imAccid, ChannelType.VIDEO, new JoinChannelCallBack() {
    @Override
    public void onJoinChannel(ChannelFullInfo channelFullInfo) { 

    }

    @Override
    public void onJoinFail(String msg, int code) {   }
});

```

#### 监听

> Demo中在 NERTCVideoCallImpl 类实现 Observer<ChannelCommonEvent>，监听信令事件，在handleNIMEvent()处理invite

```java
// 接口 NERTCCallingDelegate

/**
 * 被邀请通话回调
 *
 * @param invitedEvent 邀请参数
 */
void onInvited(InvitedEvent invitedEvent);

InvitedEvent invitedEvent = (InvitedEvent) event;
if (delegateManager != null) {
    if (currentState.getStatus() != CallState.STATE_IDLE) { //占线，直接拒绝
        Log.d(LOG_TAG, "user is busy status =  " + currentState.getStatus());
        InviteParamBuilder paramBuilder = new InviteParamBuilder(invitedEvent.getChannelBaseInfo().getChannelId(),
                invitedEvent.getFromAccountId(), invitedEvent.getRequestId());
        paramBuilder.customInfo(BUSY_LINE);
        reject(paramBuilder, false, null);
        break;
    } else {
        startCount();
        delegateManager.onInvited(invitedEvent);
    }
}
setCallType(invitedEvent);
currentState.onInvited();

/**
 * 返回操作
 *
 * @param errorCode  错误码
 * @param errorMsg   错误信息
 * @param needFinish UI层是否需要退出（如果是致命错误，这里为true）
 */
void onError(int errorCode, String errorMsg, boolean needFinish);

/**
 * 被邀请通话回调
 *
 * @param invitedEvent 邀请参数
 */
void onInvited(InvitedEvent invitedEvent);

/**
 * 如果有用户同意进入通话频道，那么会收到此回调
 *
 * @param uid 进入通话的用户
 */
void onUserEnter(long uid,String accId);


/**
 * 如果有用户同意离开通话，那么会收到此回调
 *
 * @param accountId 离开通话的用户
 */
void onCallEnd(String accountId);

/**
 * 用户离开时回调
 * @param accountId
 */
void onUserLeave(String accountId);

/**
 * 用户断开连接
 * @param userId
 */
void onUserDisconnect(String userId);

/**
 * 拒绝通话
 *
 * @param userId 拒绝通话的用户
 */
void onRejectByUserId(String userId);


/**
 * 邀请方忙线
 *
 * @param userId 忙线用户
 */
void onUserBusy(String userId);

/**
 * 作为被邀请方会收到，收到该回调说明本次通话被取消了
 */
void onCancelByUserId(String userId);


/**
 * 远端用户开启/关闭了摄像头
 *
 * @param userId           远端用户ID
 * @param isVideoAvailable true:远端用户打开摄像头  false:远端用户关闭摄像头
 */
void onCameraAvailable(long userId, boolean isVideoAvailable);

/**
 * 远端用户开启/关闭了麦克风
 *
 * @param userId           远端用户ID
 * @param isAudioAvailable true:远端用户打开麦克风  false:远端用户关闭麦克风
 */
void onAudioAvailable(long userId, boolean isAudioAvailable);

/**
 * 网络状态回调
 *
 * @param stats
 */
void onUserNetworkQuality(NERtcNetworkQualityInfo[] stats);

/**
 * 通话类型改变
 *
 * @param type
 */
void onCallTypeChange(ChannelType type);

/**
 * 呼叫超时
 */
void timeOut();
```



#### 接听

```java
/**
 * 当您作为被邀请方收到 {@link NERTCCallingDelegate#onInvited } 的回调时，可以调用该函数接听来电
 *
 * @param invitedParam 邀请信息
 * @param selfAccId 自己的accid
 * @param joinChannelCallBack  加入channel的回调
 */

nertcVideoCall.accept(invitedParam, selfUserId, new JoinChannelCallBack() {
    @Override
    public void onJoinChannel(ChannelFullInfo channelFullInfo) {
        
    }

    @Override
    public void onJoinFail(String msg, int code) {

    }
});
```



#### 挂断

```java
/**
 * 当您处于通话中，可以调用该函数结束通话（离开房间并关闭房间）
 * 通话发起者拥有此权限，并可以授权给接受者是否拥有此权限
 */
nertcVideoCall.hangup(new RequestCallback<Void>() {
    @Override
    public void onSuccess(Void aVoid) { }

    @Override
    public void onFailed(int i) { }

    @Override
    public void onException(Throwable throwable) { }
});
```



#### 话单

> **注：多人通话没有封装话单，如有需求，请自行实现。**

> 话单功能需要单独开通，如有需求，请联系对应商务。



1. 正常挂断，NIMRtcCallStatus 状态为 NIMRtcCallStatusComplete

> 调用组件封装的挂断之后，服务器会正常下发正常结束的话单。

客户端会通过客户端接收消息的回调observeReceiveMessage，收到一条为 NetCallAttachment 的消息，对消息解析并抛到上层进行展示，可参考

- - 在Application中NimUIKit.registerMsgItemViewHolder(NetCallAttachment.class, MsgViewHolderNertcCall.class);

- - 实现 MsgViewHolderNertcCall 类


2. 非正常挂断

> 异常挂断，需要业务层主动调用以下方式，组件话单提供给对方，包含**超时**、**忙线**、**拒绝**的话单都是组件内部发送的

```java
// 取消通话
nertcVideoCall.cancel(new RequestCallback<Void>() {
    @Override
    public void onSuccess(Void aVoid) { }

    @Override
    public void onFailed(int i) { }

    @Override
    public void onException(Throwable throwable) { }
});
```

底层实现主要是调用SDK发送点对点消息sendMessage，通过封装信息之后，发送到对方，对方收到消息解析。可参考在 **NERTCVideoCallImpl** 调用 **makeCallOrder** 方法 ，实现话单，构建一条**createNrtcNetcallMessage**的消息进行发送

### UI相关

[Demo](https://github.com/netease-im/NIM_Android_Demo/tree/dev_g2)中UI可参考模块：

> NERTCVideoCallActivity，以及话单类 MsgViewHolderNertcCall
