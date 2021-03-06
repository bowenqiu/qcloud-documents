## 1. TXBoardSDK 简介

`TXBoardSDK.framework`是一个实现了基本白板功能的组件，提供了包括画笔、橡皮擦、背景图、标准图形、移动涂鸦和文档展示等功能。
除了以上的本地白板功能之外，白板SDK还提供了网络多终端互通的扩展能力。

> **注意：**TICSDK 中已经包含了白板 SDK，开发者无需单独集成，也不需要关心代理设置，协议方法，白板数据传输和同步等逻辑，这些功能已经在 TICSDK 内部实现，开发者只需要创建白板对象，并调用白板对象的相应接口来操作白板即可。

####头文件概览

先总体说明下 SDK 中暴露的公开头文件的主要功能：

类名 | 主要功能
--------- | ---------
TXBoardSDK.h | SDK 的整体管理类，暂时只有【verifySDK:】接口，用来校验sdk权限
TXBoardView.h | 白板视图类，白板操作类，包含了所有白板相关的操作接口
TXBoardCommon.h | 白板定义类，定义了一些与白板视图相关的枚举，类和代理方法
TXFileManager.h | 文件管理类，内部封装了腾讯云对象云存储 COSSDK，负责文件（PPT、wrod、Excel、pdf、图片等）的上传、下载、在线转码预览等。

> TXBoardSDK.h 中的 `verifySDK:` 接口用于校验客户是否开通了白板业务（没开通将不能使用白板的大部分功能），TICSDK内部已经调用该方法校验，集成TICSDK的开发者可以忽略该方法。

> 如果单独集成`TXBoardSDK`，则需在使用白板前自行调用 `verifySDK:` 方法，不调用则`TXBoardSDK`内部默认为未开通状态。

## 2. 白板使用方法

### 2.1 创建一块白板

`TXBoardView`的创建方法普通的 UIView 一致，创建实例后需要添加到一个 UIView 中。
为了正常使用白板功能，我们要求`TXBoardView`可以直接捕获用户的 UI 交互事件。

**创建一块白板的示例代码：**

```objc
TXBoardView *boardView = [[TXBoardView alloc] initWithFrame:frame];
[self.view addSubview:boardView];
```

>白板长宽比固定为 16：9。

### 2.2 设置白板的 delegate

> **注意：**如果使用 TICSDK，则不需要设置该代理对象，TICSDK 内部已经实现了所有代理方法

为了使白板 SDK 正常运行，开发者需要在`TXBoardView`中设置满足`TXBoardViewDelegate`声明的 delegate 对象。

**delegate声明：**

```objc
#pragma mark - 白板代理
@protocol TXBoardViewDelegate <NSObject>

- (void)sendMessage:(NSDictionary *)message;

- (uint32_t)getTimestamp;

- (TXBoardDataConfig *)getBoardDataConfig;

@end

```

**方法说明：**

接口 | 说明
---|---
sendMessage: | 向网络中其他白板终端发送协议数据，开发者可以透传 NSDictionary 的 json 字符串
getTimestamp | 获取秒级的时间戳，在多终端白板互通场景下使用服务器时间
getBoardDataConfig | 获取白板所需外部参数，包含 uid、userSig、roomID（这些值发送改变时需要重新设置）


### 2.3 选择白板的画笔状态

开发者使用白板 SDK 过程中，需要主动设置白板的画笔类型，才能实现画线、橡皮擦、缩放、添加标准图形等操作。

**相关接口：**

```objc

/** 白板工具类型 */
typedef NS_ENUM(NSInteger, TXBoardBrushModel)
{
    TXBoardBrushModelNone = 0,
    TXBoardBrushModelLine = 1,      //线条（默认）
    TXBoardBrushModelEraser,        //橡皮擦（点选删除某段涂鸦）
    TXBoardBrushModelTapSel,        //点击选择涂鸦
    TXBoardBrushModelRectSel,       //框选涂鸦
    TXBoardBrushModelSegment,       //直线线段
    TXBoardBrushModelOval,          //椭圆
    TXBoardBrushModelOvalFill,      //实心椭圆
    TXBoardBrushModelRound,         //正圆
    TXBoardBrushModelRoundFill,     //实心正圆
    TXBoardBrushModelRectangle,     //矩形
    TXBoardBrushModelRectangleFill, //实心矩形
    TXBoardBrushModelTransform      // 缩放(双指)/移动(单指)
};

@interface TXBoardView : UIView

- (void)setBrushModel:(TXBoardBrushModel)model;

- (TXBoardBrushModel)getBrushModel;

@end

```

当选择对应的画笔类型后，用户操作白板区域时，会展示对应的效果。

### 2.4 白板的接口列表

除了通过设置白板模式参数，调整白板功能外，开发者可以调用`TXBoardView`的其他接口，触发白板功能。
例如：创建子白板、更新背景图、选择颜色等操作。

**TXBoardView接口声明：**

```objc

@interface TXBoardView : UIView

#pragma mark - 白板配置
@property (nonatomic, weak) id<TXBoardViewDelegate> boardViewDelegate;

+ (NSString *)getVersion;

#pragma mark - 白板操作相关

- (void)setBrushColor:(UIColor *)color;
- (UIColor *)getBrushColor;

- (void)setBrushWidth:(CGFloat)width;
- (CGFloat)getBrushWidth;

- (void)setBrushModel:(TXBoardBrushModel)model;
- (TXBoardBrushModel)getBrushModel;

- (void)setEraserRadius:(CGFloat)radius;
- (CGFloat)getEraserRadius;


- (void)setBackgroundColor:(UIColor *)color;


- (void)setGlobalBackgroundColor:(UIColor *)color;

- (void)undo;

- (BOOL)canUndo;

- (void)redo;

- (BOOL)canRedo;

- (void)clear;

- (void)clearDraws;

#pragma mark - 背景图片

- (void)updateBgImageWithPath:(NSString *)imagePath mode:(TXBoardImageMode)mode succ:(TXSuccBlock)succ failed:(TXFailBlock)failed;

- (void)updateBgImageWithURL:(NSString *)imageURL mode:(TXBoardImageMode)mode succ:(TXSuccBlock)succ failed:(TXFailBlock)failed;

- (NSString *)getBGImageURL:(NSString *)BoardId;

- (void)saveToAlbumWithFinish:(void (^)(void))finishBlcok;

#pragma mark - 多白板

- (NSString *)currentBoardId;

- (NSString *)createSubBoard;

- (BOOL)switchToSubBoard:(NSString *)boardId;

- (BOOL)deleteSubBoard:(NSArray<NSString *> *)boardIds stayBoard:(NSString *)stayBoard;

- (NSArray<NSString *> *)boardList;

#pragma mark - 文档

- (NSString *)addFile:(NSArray *)urls fileName:(NSString *)fileName;

- (int)deleteFile:(NSString *)fid stayBoardID:(NSString *)stayBoardID;

- (NSArray *)getBoardIDsWithFid:(NSString *)fid;

- (NSArray *)getAllFileInfo;

#pragma mark - 数据

- (void)recvBoardViewData:(NSArray<NSDictionary *> *)data;

- (void)getBoardData:(void (^)(void))succ failed:(void (^)(void))failed;

@end

```

**白板的基本功能：**

接口 | 说明
---|---
setBrushColor: | 设置画线、添加标准图形时线条的颜色
getBrushColor | 获取当前画笔颜色
setBrushWidth: | 设置画线、添加标准图形时线条的宽度
getBrushWidth | 获取当前画笔宽度
setEraserRadius: | 设置擦除效果的范围半径
getEraserRadius: | 获取擦除效果的范围半径
setBackgroundColor: | 设置当前白板背景颜色
setGlobalBackgroundColor: | 设置全局白板背景颜色
clear: | 清空当前白板所有数据，包含背景图片
clearDraws: | 清空当前白板涂鸦数据，不包含背景图片、PPT
undo | 撤销上一步操作
canUndo | 获取是否可以进行撤回操作
redo | 重做上一步被撤回的操作
canRedo | 获取是否可以进行重做操作

**背景图片：**

接口 | 说明
---|---
updateBgImageWithPath:mode:succ:failed: | 设置背景图片（本地图片路径）
updateBgImageWithURL:mode:succ:failed: | 设置背景图片（网络图片 URL）
getBGImageURL: | 获取BoardId对应白板当前显示的背景图片
saveToAlbumWithFinish: | 将白板当前内容截图，保存到本地相册


**多白板及文档：**

接口 | 说明
---|---
currentBoardId | 获取当前的白板实例 ID，默认的 ID 为字符串 #DEFAULT。
createSubBoard | 创建一个内部白板实例，返回标识 ID；创建白板后默认使用 #DEFAULT 为 ID 白板实例。
switchToSubBoard | 切换当前白板的展示内容，返回切换结果。
deleteSubBoard:stayBoard: | 删除一系列白板实例，指定当前白板展示内容，返回结果；SDK 不允许删除 #DEFAULT 的白板实例。
boardList | 返回所有白板 ID 数组。
addFile:fileName: |  添加文档
deleteFile:stayBoardID: |  删除文档
getBoardIDsWithFid: |  获取文档对应的所有boardID
getAllFileInfo | 获取所有文档信息(包括普通白板组)

**数据的接收与同步：**

接口 | 说明
---|---
recvBoardViewData: | 填入远端的白板操作数据，白板会同步展示远端画面。
getBoardData:failed: | 拉取白板数据（加入课堂后，从服务器拉取课堂白板数据，填充白板）。


## 3. 白板数据实时收发

不同端白板间的数据传输是建立在腾讯`IMSDK`建立的即时信道上的，该功能已经封装在 TICSDK 内部，开发者无需自行实行。

## 4. 白板数据上报备份和拉取填充

课堂中，老师对白板的操作，涂鸦、图片、PPT、撤销、清空等操作需要上报到后台，并进行存储，这样后面中途加入课堂的成员就能拉取之前的白板数据进行展示。

该过程主要分为两步，数据上报和数据拉取：

**白板数据上报：**
在每次对白板操作后，SDK 会将操作的数据上报到白板后台，目前 SDK 内部已经实现了该功能，白板后台服务也是由我们维护，开发者无需自行实现。

**白板数据拉取（同步）：**
每次进入课堂时，TICSDK 会拉取该课堂的所有历史白板消息，展示在白板上，该功能也已经在 TICSDK 内部实现，开发者无需自行实行。

## 5. 文档上传下载
文档的上传下载封装于`TXFileManager.h`类中，该类内部封装了腾讯云对象存储COS V5版本（https://cloud.tencent.com/product/cos）。COS 为 [腾讯云对象存储](https://cloud.tencent.com/document/product/436/6225)，如果您的 App 中需要用到上传图片、文档到白板上展示的功能，则需要用到COS，SDK 内部会将调用 SDK 接口上传的图片，文件上传到 COS 的存储桶中。

### 配置COS
由于，SDK内部文件管理是基于COS封装的，所以使用之前必须先配置一下COS。

开发者可以使用我们维护的公共账号（每个客户对应一个存储桶，推荐），也可以自己申请配置COS账号并自行维护。

具体接口如下：

```objc
> TXFileManager.h

/**
 @brief 初始化COS（使用COS上传文件前必须先初始化）

 @param sdkAppID 腾讯云控制台注册的应用ID
 @param config COS配置对象（传nil，表示使用腾讯云的公共账号）
 @see TXCosConfig
 @return 0 配置成功，否则配置失败(返回错误码 8021，表示参数无效)
 */
- (int)initCosWithSDKAppID:(NSString *)sdkAppID config:(TXCosConfig *)config;
```

**如选择使用COS公共账号，`config` 参数传入 `nil` 即可。**

如使用自己申请的COS账号，则需配置 `config` 参数：

```objc
/**
 COS 配置类，其属性参数都可从腾讯云 COS 控制台获取到
 */
@interface TICCosConfig : NSObject

/// @brief COS服务的appId，用以标识资源
@property (nonatomic, copy) NSString *cosAppID;
/// @brief 存储桶名称
@property (strong, nonatomic) NSString *bucket;
/// @brief 服务地域名称
@property (nonatomic, copy) NSString *region;
/// @brief 开发者拥有的项目身份识别 ID，用以身份认证
@property (nonatomic, copy) NSString *secretID;
/// @brief 开发者拥有的项目身份密钥
@property (nonatomic, copy) NSString *secretKey;

@end
```

### 上传/下载文件
SDK 已经将COS相关接口封装好，上层调用对应接口即可，接口如下：

> 这里的下载接口只是为了方便开发者使用，开发者也可以上传文件之后拿到下载链接自行下载

```
> TXFileManager.h

#pragma mark - 上传下载COS默认文件夹中的文件

/**
 @brief 上传文件到COS云存储TXSDK默认文件夹（/TIC/iOS）
 @discussion 可上传图片和文件，文件类型包含：doc(x)、wps、rtf 、xls(x)、et、ppt(x)、dps、pdf、txt

 @param filePath 要上传文件的本地路径，为了确保同名文件覆盖问题，该方法上传时会在文件名前拼接一个时间戳
 @param onProgress 上传进度回调
 @param onFinish 上传完成回调（回调内容为上传文件的下载URL和error信息）
 */
- (void)uploadFile:(NSString *)filePath onProgress:(onUploadProgress)onProgress onFinish:(onUploadFinish)onFinish;


/**
 @brief 下载COS云存储TXSDK默认文件夹（/TIC/iOS）中的文件

 @param fileURL 下载文件URL(上传文件方法返回的URL)
 @param onProgress 下载进度回调
 @param onFinish 下载完成回调（回调内容为下载文件的data数据和error信息）
 */
- (void)downloadFile:(NSString *)fileURL onProgress:(onDowloadloadProgress)onProgress onFinish:(onDownLoadloadFinish)onFinish;

#pragma mark - 上传下载COS指定路径下的文件

/**
 @brief 上传文件到COS指定路径
 @discussion 可上传图片和文件，文件类型包含：doc(x)、wps、rtf 、xls(x)、et、ppt(x)、dps、pdf、txt

 @param filePath 要上传文件的本地路径
 @param cosPath 指定要上传到COS的路径（例如:/APP/0514/image.jpg）（相同的cosPath后上传的会覆盖先上传的）
 @param onProgress 上传进度回调
 @param onFinish 上传完成回调（回调内容为上传文件URL和error信息）
 */
- (void)uploadFile:(NSString *)filePath cosPath:(NSString *)cosPath onProgress:(onUploadProgress)onProgress onFinish:(onUploadFinish)onFinish;


/**
 @brief 下载COS指定路径中的文件

 @param cosPath 要下载文件的COS指定路径（例如:/APP/0514/image.jpg）
 @param onProgress 下载进度回调
 @param onFinish 下载完成回调（回调内容为下载文件的data数据和error信息）
 */
- (void)downloadFileWithCosPath:(NSString *)cosPath onProgress:(onDowloadloadProgress)onProgress onFinish:(onDownLoadloadFinish)onFinish;
```

## 6. 文档展示功能详解
### 相关概念解释
* 默认白板：白板 SDK 初始化后内部会自动生成一个白板，这个白板是第一块白板，作为白板 SDK 展示的默认白板，不允许删除。

* 文件白板：即通过 `addFile` 接口生成的白板，一块文件白板对应文件的一页（底层实现为普通白板，设置其背景图为文件预览URL）。

* 普通白板：即非文件白板，调用 `createSubBoard` 方法创建的白板，可涂鸦，可设置/清除白板，默认白板属于普通白板。

* fid：文件唯一标识，由于一个文件对应一组白板，所以fid也可以理解为一个白板组的标识。
* boardID：白板 ID，白板唯一标识。

* boardID 生成规则：
    - 默认白板的 boardID 为固定值 #DEFAULT。
    - 其他白板的 boardID 为 xxxxxxx_fid （fid 为该白板所属文件(白板组)的唯一标识）。

* fid生成规则（fid都以#号开头）
    - 普通白板的 fid 为固定值 #DEFAULT。
    - 文件白板的 fid 为 #xxxx。
    
>**注意：**文件在白板中的实现其实就是一组白板（文件的每一页对应一块白板），他们用一个 fid 来标识，这组白板中每个白板的 boardID 都会带上其所属文件（白板组）的 fid 当做后缀。

### 添加文档（以 PPT 为例）
1. 开发者首先调用 TICSDK 中的文件管理方法将 PPT 文件上传到腾讯云对象存储 COS，获取到 PPT 每一页转码后的图片 UR L链接。

2. 然后调用`addFile`接口，将上一步获取到的 URL 数组，和文件名当做参数传入，SDK 内部会根据 URL 数量生成对应数量的白板，并生成一个该文件对应的文件唯一标识 **fid** 返回。

* 代码实例：

```objc
NSString *fid = [self.boardView addFile:urls fileName:fileName];
```

### 删除文档
1. 调用`deleteFile`接口，传入文件对应的 fid 和删除后要跳转到的白板 ID，deleteFile 内部会将该文件对应的白板全部删除。
> **注意：**普通白板组不能删除。

* 代码实例：

```objc
[self.boardView deleteFile:deleteFid stayBoardID:stayBoardID];
```


### 切换白板
1. 调用 `switchToSubBoard` 方法，传入 boardID 即可切换到对应白板。

* 代码实例：

```objc
[self.boardView switchToSubBoard:targetBoardID];
```

### 切换 PPT
1. 切换 PPT 其实也是切换白板

> **注意：**用户需要自己维护哪个 PPT 当前显示的是哪一页。

### 获取当前显示的白板
1. 调用 `currentBoardId` 接口，即可获得当前显示的白板 ID。

* 代码实例：

```objc
NSString *currentBoardId = self.boardView.currentBoardId;
```

### 根据 fid 获取文件对应的所有白板 ID
1. 调用`getBoardIDsWithFid`接口，传入 fid 即可

* 代码实例：

```objc
NSArray *boardIds = [self.boardView getBoardIDsWithFid:fid];
```

### 获取课堂内所有文件信息（PPT）
1. 调用`getAllFileInfo`方法即可获得当前课堂内所有的文件信息（调用 deleteFile 接口删除的文件不会返回）。
2. 该接口返回格式如下，业务层根据该接口返回的信息结合 fid 即可确定白板属于哪个 PPT 和该 PPT 的title：

```json
    [
        {
        "fid" : "",      // 文件唯一标识
        "title" : "",   // 文件名称
        }
        ...
    ]
```

### 推荐实现方案
1. 进房或者调用`addFile`添加文件后，调用 `getAllFileInfo` 可获得课堂内所有文件列表，用于展示。
2. 翻页时，先调用 `currentBoardId` 接口获得当前显示的白板 ID，再调用`getBoardIDsWithFid`接口获得当前文件所有白板 ID，即可得出当前显示的是哪一页，然后根据具体的翻页指令，调用 `switchToSubBoard` 调到对应的白板即可。




