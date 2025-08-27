# Basic
## 基本流程概览   
> 实线表示当前项目中有实际开发任务.
> 虚线表示无实际开发任务.
```mermaid
sequenceDiagram
    participant Device as 设备
    participant AIService as AI 服务
    Device-->>Device: 设备启动
    Device->>Web: SN 进行认证请求
    Web->>Device: universal_app_api - [locale, device info, launcher app list, app layout]
    Device->>Device: 展示 launcher app with app icon list
    Device->>Device: 用户选择某个 app 开始使用
    Device->>Device: 用户退出当前 app, 返回 launcher
```
## app layout
均采用`chatter-basic` 布局.

# 智能相册
在当前智能相册已经有的功能基础上, 增加 AI 相关的功能.   
项目将采用创建一个新的 app 来实现, 应与现有智能相册功能隔离, 以下功能实现均于此新 app 中.
## 用户故事
### 会议录音纪要应用
#### 准备会议录音
```mermaid
sequenceDiagram
    participant User as User
    participant Device as 设备
    participant Service as 后台
    User-->>Device: 点击 创建会议录音
    Device->>Service: 申请 创建会议录音
    Service->>Device: 返回 接收会议纪要注册二维码 - 每15秒更新一次
    Device->>Device: UI 展现二维码, 供想接收会议纪要的用户扫码申请
    User-->>Device: 扫码申请
    User-->>Service: 申请请求 - [邮箱, wechat id]
    Service->>Device: 初审通过(可选管理员二次审核)的用户信息
    Device->>Device: UI 展现初审用户列表
    Device->>Device: 可指定管理员(负责二次审核和编辑会议纪要), 默认第一位
    Device->>Device: 展示 开始会议录音 按钮    
```
#### 开始会议录音
```mermaid
sequenceDiagram
    participant User as User
    participant Device as 设备
    participant Service as 后台
    Device->>Device: 展示时间进度条, 以及暂停录音按钮(敏感内容?)
    Device->>Device: 可选会议纪要抄送模式(二次审核模式,或者直接发送模式)
    Device->>Service: 录音开始
    Device->>Service: 持续上传
    User-->>Device: 点击 结束会议录音 按钮
    Service->>Service: 长语音识别(调用第三方平台, 耗时操作)和LLM整理
    Service->>User: web link to 管理员: 可编辑收件人列表+可编辑会议纪要(说话人1:..., 说话人2:...)+ 可编辑说话人姓名
    User-->>Service: 确认收件人列表和会议纪要
    Service->>Service: 发送最终会议纪要到确认的申请人员
```

### 点击设备侧按钮请求克隆声音
用户在设备上点击触摸屏上的`克隆声音`按钮, 弹窗出现位于**右下角**新窗口, 其中包括:    
* Title    
例如: "以正常的 音调 和 语速 朗读下面的文字"
* 指定的朗读文字    
正中位置, 内容例如: "如果能再给我一天光明, 我会让你看到我最美的一面! 唉, 可惜了."
* 确认开始按钮
按住后, 开始录音, 展示进度条计时, 松开时结束录音, 如果录音时间过长或者过短, 则提示用户重新录音.

```mermaid
sequenceDiagram
    participant Device as 设备
    participant AIService as AI 服务
    Device->>Device: 点击dynamic func btn
    Web-->>Device: universal_app_api - [popup confirm box of 欠费提示提示框, 需要充值+ 二维码]
    Device->>Device: popup window- [Title, 指定的朗读文字, 确认开始按钮]
    Device->>Device: 按住确认开始按钮 - 开始录音
    Device->>Device: 展示进度条计时
    Device->>Device: 松开确认开始按钮 - 结束录音
    Device->>Web: universal_app_api - 录音文件
    Web->>AIService: mqtt api - [set to use new cloned voice]
    Web->>AIService: inject context - 用户刚才clone了一个新声音
    Web->>Device: universal_app_api - [show notify icon btn, click behavior: "show text: 克隆声音成功"]
```
### 手机侧进行克隆声音
用户在设备上点击触摸屏上的`克隆声音`按钮, 弹窗新窗口, 其中包括:    
* TITLE   
例如: "在手机上克隆声音"
* 正文    
"请用手机扫描以下二维码, 进行克隆声音"

```mermaid
sequenceDiagram
    participant Device as 设备
    participant AIService as AI 服务
    Device->>Device: 点击dynamic func btn
    Web-->>Device: universal_app_api - [popup confirm box of 欠费提示提示框, 需要充值+ 二维码]
    Device->>Device: popup window- [Title, 在手机上克隆声音, 二维码]
    Device->>Device: 按住确认开始按钮 - 开始录音
    Device->>Device: 展示进度条计时
    Device->>Device: 松开确认开始按钮 - 结束录音
    Web->>AIService: mqtt api - [set to use new cloned voice]
    Web->>AIService: inject context - 用户刚才clone了一个新声音
    Web->>Device: universal_app_api - [show notify icon btn, click behavior: "show text: 克隆声音成功"]
```


# 导览机
## 用户故事
* app layout    
    符合 `chatter-basic`
* app layout style
    none
### 基本对话
用户拿起设备, 用对讲机或者 RTC 进行通话, 设备将用户的语音发送到 AI 服务进行处理, AI 服务将处理结果返回给设备, 设备播放内容。
```mermaid
sequenceDiagram
    participant Device as 设备
    participant AIService as AI 服务
    
    Device->>Device: 对讲机, 拍摄图片, 或者 开启 RTC 通话
    Device->>AIService: modality_upward
    AIService->>Device: modality_downward
    Device->>Device: 播放处理结果
    Device->>Device: 用户移动
    Web->>AIService: inject context - 用户位置  
    Web->>Device: universal_app_api - [modality msg, 新景点播报词]
```
### 后台推荐内容 via 用户点击
AI 服务根据用户的 context (姓名, 性格, 位置等) 推荐后台已经生成的固定内容, 用户需要在触摸屏上点击确认后播放.
```mermaid
sequenceDiagram
    participant Device as 设备
    participant AIService as AI 服务
    Web->>Device: universal_app_api - [modality msg, 音频:"听Podcast吗,请在列表上选择一个"] 
    Web->>Device: universal_app_api - [popup list box, play sound:"dingdong"]
    Device->>Device: 播放提示音和展示列表框
    Device->>Device: 用户选择了列表中的一个Podcast
    Device->>Web: universal_app_api - 用户点击了确认
    Web->>AIService: inject context - 已推荐的内容  
    Web->>Device: universal_app_api - [modality msg, podcast, music, video]
```

### 后台推荐内容 via Agent
AI 服务根据用户的 context (姓名, 性格, 位置等) 推荐后台已经生成的固定内容, 用户通过语音选择是否进行播放.

> 技术可行, 暂不实现
# 相框
* app layout    
    `chatter-basic` with dynamic func buttons: [克隆声音]
    需要线下约定`克隆声音`按钮的交互, 可见下文说明    
* style    
    请考虑: 设备屏幕尺寸11寸;设备重量轻(不方便单手指直接点击?);app常规状态下应全屏显示多媒体内容;
## 用户故事
### 基本对话

```mermaid
sequenceDiagram
    participant Device as 设备
    participant AIService as AI 服务
    Device->>Device: 对讲机, 拍摄图片, 或者 开启 RTC 通话
    Device->>AIService: modality_upward
    AIService->>Device: modality_downward
    Device->>Device: 播放处理结果
```

### 请求定闹钟
用户在设备上进行语音对话时, 说到: "请为我设置一个`n`分钟后的闹钟" 或者 "请为我设置一个`未来时刻`的闹钟".    

```mermaid
sequenceDiagram
    participant Device as 设备
    participant AIService as AI 服务
    Device->>Device: 对讲机, 拍摄图片, 或者 开启 RTC 通话
    Device->>AIService: modality_upward - 用户语音提出请求
    AIService->>AIService: agent 判定 - loop直到满足 tool 调用条件
    AIService->>Web: webapi tool 请求 - [set alarm, time]
    Web->>AIService: tool 调用结果
    AIService->>Device: modality_downward - [已经设置好了]
    Web->>Device: universal_app_api - [show notify icon btn, click behavior: "show text: 闹钟设置于11:11:11"]
    Web->>Web: 等待 n 分钟
    Web->>Device: universal_app_api - [popup confirm box, modality msg of 闹钟提示音]
    Device->>Device: 播放闹钟提示音和展示提示框
    Device->>Web: universal_app_api - [用户点击确认]
    Web->>AIService: inject context - 已经响过闹钟且用户点击了确认
```

### 请求显示下一张图片
用户在设备上进行语音对话时, 说到: "请为我显示下一张图片" 或者 "请为我显示上一张图片".    

```mermaid
sequenceDiagram
    participant Device as 设备
    participant AIService as AI 服务
    Device->>Device: 对讲机, 拍摄图片, 或者 开启 RTC 通话
    Device->>AIService: modality_upward - 用户语音提出请求
    AIService->>AIService: agent 判定 - loop直到满足 tool 调用条件
    AIService->>Web: webapi tool 请求 - [show image, index]
    Web->>AIService: tool 调用结果
    Web->>Device: universal_app_api - [modality msg of image url]
    AIService->>Device: modality_downward - "voice: 已经执行好了"
    Device->>Device: 播放处理结果
    Web->>Device: universal_app_api - [屏幕亮度调节, 最低亮度]
    Device->>Device: 调节屏幕亮度到最低
    Device->>Web: universal_app_api - 屏幕亮度调节成功
```

### 语音请求进行设备设置
调节音量为示例.
```mermaid
sequenceDiagram
    participant Device as 设备
    participant AIService as AI 服务
    Device->>Device: 对讲机, 拍摄图片, 或者 开启 RTC 通话
    Device->>AIService: modality_upward - 用户语音提出请求
    AIService->>AIService: agent 判定 - loop直到满足 tool 调用条件
    AIService->>Web: webapi tool 请求 - [set volume, level]
    Web->>Device: universal_app_api - [set volume, level]
    Device->>Device: 调节音量
    Device->>Web: universal_app_api - 音量调节成功
    Web->>AIService: tool 调用结果
    AIService->>Device: modality_downward - [已经设置好了]
```
