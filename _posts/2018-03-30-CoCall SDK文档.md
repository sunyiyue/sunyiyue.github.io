---
layout: post
title: CoCall SDK文档
date: 2018-03-30 13:05:00.000000000 +09:00
---




### 消息对象 CCMessage (NSObject)

|   字段    |  类型   |                       含义                       |
| :-------: | :-----: | :----------------------------------------------: |
|   messageId   |  NSNumber   |          本地存储的消息的唯一值（数据库索引唯一值）           |
|   msgId   |  NSNumber   |          服务器中消息ID，唯一，按时间顺序           |
|   mType   | NSNumber |                  消息类型                   |
|    sid    | NSString  |            会话ID，同CCSession.id             |
|   cType   | NSNumber |          会话类型，同CCSession.type           |
|  sender   | NSString  |                  发送人ID                   |
|  content  | NSString  |      消息实体内容字段，可用于直接展示       |
|  search   | NSString  | 服务器消息搜索字段,返回的数据并不含有该数据 |
|   data    | NSString  |   消息体内容，json数据串，根据消息类型返回不同消息体    |
| timestamp |  NSNumber   |               消息发送时间戳                |
|    at     | BOOL |             该条消息中是否被@到             |
|    msgStatus     | NSString |              消息发送状态 0 正在发送  -1 失败  1 成功           |

### 消息类型

| 字段 |            含义            |                            消息体                            |
| :--: | :------------------------: | :----------------------------------------------------------: |
|  1   |    本文消息    |                            String                            |
|  2   |     图片消息    |  {id:String,url:String,md5:String,length:long,extra:String}  |
|  3   | 文本图片消息| [{type:1,content:String},{type:2,d:String,url:String,md5:String,length:Long,extra:String}...] |
|  4   |    文件消息    | {id:String,url:String,md5:String,length:Long,name:String,mimeType:String} |
|  5   |    语音消息   | {id:String,url:String,md5:String,length:Long,name:String,mimeType:String} |
|  6   |     系统消息    |                             ???                              |
| ...  |                            |                                                              |
| 255+ |         自定义消息         |                            String                            |




### 会话对象 CCSession (NSObject)



|   字段    |  类型   |                       含义                       |
| :-------: | :-----: | :----------------------------------------------: |
|    id     | NSNumber |                      会话Id                      |
|   type    | NSNumber |             会话类型，见CCConversationType              |
|   name    | NSString  | 会话名称，单人会话为对方名称，讨论组为讨论组名称 |
| readMsgId |  NSNumber   |                    已读消息ID                    |
| timestamp |  NSNumber   |                  会话变更时间戳                  |
|    cnt    | NSNumber |                   未读消息数量                   |
### 上传语音、文件消息所用对象 CCUploadModel(NSObject)

|   字段    |  类型   |                       含义                       |
| :-------: | :-----: | :----------------------------------------------: |
|    name     | NSString |                      文件名字                      |
|    data     | NSData |                      数据                     |

### 已读状态返回数据对象 CCReadStat(NSObject)

|   字段    |  类型   |                       含义                       |
| :-------: | :-----: | :----------------------------------------------: |
|    statArray     | NSMutableArray |   @[CCstateModel,CCstateModel,...]                     |
|    timestamp     | long long |                      timestamp                     |

### CCstateModel(NSObject)


|   字段    |  类型   |                       含义                       |
| :-------: | :-----: | :----------------------------------------------: |
|    uid     | NSString |   会话内成员id                   |
|    msgId     | NSNumber|       会话内成员已读的消息id                                 |


### 会话类型  CCConversationType (枚举类型)

|   字段    |  对应值   |                       含义                       |
| :-------: | :-----: | :----------------------------------------------: |
|    CCSingleConversation     | 1 |   单人会话类型                  |
|    CCBroadcastConversation     | 2 |   广播会话类型                 |
|    CCGroupConversation     | 3 |   讨论组会话类型                 |

### 草稿对象  MessageDraft

|   字段    |  类型   |                       含义                       |
| :-------: | :-----: | :----------------------------------------------: |
|    text     | NSString |   草稿信息                   |
|    atList     | NSArray|       @到的人员列表                                |
|    cType     | NSNumber|      会话类型,同CCSession.type           |
|    sid     | NSString|      会话ID，同CCSession.id          |

### 错误信息  CCError

|   字段    |  类型   |                       含义                       |
| :-------: | :-----: | :----------------------------------------------: |
|    code     | NSInteger |   返回代码值                 |
|    message     | NSString |   请求数据失败原因                 |


### 初始化、消息相关方法入口 CCIMClient
```
/**
@brief 单例
@discussion 获取该类单例进行操作
@return 返回类实例
*/
+ (CCIMClient*)sharedCCIMClient;
/**
*处理网络异常
*　token失效处理
*/
@property (nonatomic, copy) CCTokenInvalidBlock ccTokenInvalidBlock;
/**
* 处理网络异常
*/
@property (nonatomic, copy) CCNetworkErrorBlock ccNetworkErrorBlock;

/**
初始化方法

@param token 用户的token
*/
- (void)initToken:(NSString *)token;

/**
系统初始化方法

@param token 用户的token
@param successBlock 用户ID
@param errorBlock 错误信息
*/
- (void)connectWithToken:(NSString *)token
success:(void (^)(NSString *userId))successBlock
error:(void(^)(CCError *error))errorBlock;


/**
发送消息 (文字 和 语音 消息)
必填
@param conversationType 会话类型             是
@param targetId 联系人ID/讨论组ID             是
@param messageContent 消息的内容              是
param successBlock 消息ID
@param errorBlock 错误信息
@return 发送的消息实体
*/
- (CCMessage *)sendMessage:(CCConversationType)conversationType
targetId:(NSString *)targetId
content:(CCMessageContent *)messageContent
success:(void (^)(NSNumber *messageId))successBlock
error:(void(^)(CCError *error,NSNumber *messageId))errorBlock;

/**
发送媒体消息 （包括 图片 和 文件消息）
必填
@param conversationType 会话类型             是
@param targetId 联系人ID/讨论组ID             是
@param messageContent 消息的内容              是
@param progressBlock 上传进度
param successBlock 消息ID
@param errorBlock 错误信息
@return 发送的消息实体
*/
- (CCMessage *)sendMediaMessage:(CCConversationType)conversationType
targetId:(NSString *)targetId
content:(CCMessageContent *)messageContent
progress:(void (^)(int progress, NSNumber *messageId))progressBlock
success:(void (^)(NSNumber *messageId))successBlock
error:(void(^)(CCError *error,NSNumber *messageId))errorBlock;

/**
发送自定义消息
必填
@param conversationType 会话类型             是
@param targetId 联系人ID/讨论组ID             是
@param mType 自定义消息类型(使用255以上的数字)   是
@param json 自定义消息体内容，json数据串        是
@param search 服务器用于搜索的字段             是
@param atList @到的人员列表                   否
@param successBlock 消息ID
@param errorBlock 错误信息
*/
- (void)sendCustomMessage:(CCConversationType)conversationType
targetId:(NSString *)targetId
mType:(NSNumber *)mType
data:(NSString *)json
search:(NSString *)search
atList:(NSArray *)atList
success:(void (^)(NSNumber *messageId))successBlock
error:(void(^)(CCError *error,NSNumber *messageId))errorBlock;
/**
获取最近联系人/讨论组(全量/增量)

@param size 获取最新会话列表最大条数，默认200
@param timestamp 最后更新时间(可选，空为全量)
@param successBlock 联系人/讨论组信息组成的数组
@param errorBlock 错误信息
*/
- (void)getRecentContact:(NSInteger)size
timestamp:(NSNumber *)timestamp
success:(void (^)(NSArray *contactArray))successBlock
error:(void(^)(CCError *error))errorBlock;

/**
获取某条消息ID之后的消息 重复调用该接口可以获取该消息ID之后的所有消息 获取消息列表按时间逆序返回

@param sid 会话ID
@param conversationType 会话类型
@param size 获取消息列表长度，默认50
@param msgId 消息ID，获取该消息之后的消息
@param successBlock more为本次获得数据后续是否还存在数据 msgs内为获取到的消息列表
@param errorBlock 错误信息
*/
- (void)getNextRecord:(NSString *)sid
type:(CCConversationType)conversationType
size:(NSInteger)size
msgId:(NSNumber *)msgId
success:(void (^)(BOOL more,NSArray *recordArray))successBlock
error:(void(^)(CCError *error))errorBlock;

/**
获取某条消息ID之前更多的消息记录 如果没有消息ID，则表明获取最新的消息记录 消息数据按时间倒序返回

@param sid 会话ID
@param conversationType 会话类型
@param size 获取消息列表长度，默认50
@param msgId 消息ID，获取该消息之后的消息
@param successBlock more为本次获得数据后续是否还存在数据 msgs内为获取到的消息列表
@param errorBlock 错误信息
*/
- (void)getMoreRecord:(NSString *)sid
type:(CCConversationType)conversationType
size:(NSInteger *)size
msgId:(NSNumber *)msgId
success:(void (^)(BOOL more,NSArray *recordArray))successBlock
error:(void(^)(CCError *error))errorBlock;

/**
获取会话中成员的已读状态

@param sid 会话ID
@param conversationType 会话类型
@param timestamp 最后更新时间
@param successBlock CCReadStat
@param errorBlock 错误信息
*/
- (void)getReadStat:(NSString *)sid
type:(CCConversationType)conversationType
timestamp:(NSNumber *)timestamp
success:(void (^)(CCReadStat *readStat))successBlock
error:(void(^)(CCError *error))errorBlock;

/**
撤回自己发送的消息

@param sid 会话ID
@param conversationType 会话类型
@param msgId 待撤回的消息ID
@param successBlock 撤回成功的回调 [messageId:撤回的消息ID，该消息已经变更为新的消息]
@param errorBlock 错误信息
*/
- (void)chatRecall:(NSString *)sid
type:(CCConversationType)conversationType
msgId:(NSNumber *)msgId
success:(void (^)(NSNumber *messageId))successBlock
error:(void(^)(CCError *error))errorBlock;
/**
将消息置为已读状态

@param sid 会话ID
@param conversationType 会话类型列表，见SessionType
@param msgId 已读的消息ID（会话中最新的一条消息id）
@param successBlock 置已读成功success返回true
@param errorBlock 错误信息
*/
- (void)chatRead:(NSString *)sid
type:(CCConversationType)conversationType
msgId:(NSNumber *)msgId
success:(void (^)(BOOL success))successBlock
error:(void(^)(CCError *error))errorBlock;
/**
批量置已读

@param ids 会话ID列表
@param types conversationType组成的数组 会话类型列表，见SessionType
@param msgIds 已读的消息ID列表
@param successBlock 置已读成功success返回true
@param errorBlock 错误信息
*/
- (void)chatsRead:(NSArray *)ids
type:(NSArray *)types
msgIds:(NSArray *)msgIds
success:(void (^)(BOOL success,NSString *message))successBlock
error:(void(^)(CCError *error))errorBlock;

/**
会话消息输入，目前只支持1V1

@param sid 会话ID
@param conversationType 会话类型
@param successBlock 成功时success为true 失败时success为false,返回错误信息msg
@param errorBlock 错误信息
*/
- (void)chatTyping:(NSString *)sid
type:(CCConversationType)conversationType
success:(void (^)(BOOL success,NSString *message))successBlock
error:(void(^)(CCError *error))errorBlock;

/**
搜索指定会话中包含关键字的消息记录

@param sid 会话ID
@param conversationType 会话类型
@param content 搜索关键字
@param size 要获取的消息数量
@param page 页数(可选,从1开始)
@param successBlock 消息实体组成的数组
@param errorBlock 错误信息
*/
- (void)chatSearch:(NSString *)sid
type:(CCConversationType)conversationType
content:(NSString *)content
size:(NSInteger)size
page:(NSInteger)page
success:(void (^)(NSArray *recordArray))successBlock
error:(void(^)(CCError *error))errorBlock;

/**
搜索全部会话中包含关键字的消息记录

@param content 搜索关键字
@param size 要获取的消息数量
@param page 页数(可选,从1开始)
@param successBlock 消息实体组成的数组
@param errorBlock 错误信息
*/
- (void)chatSearchAll:(NSString *)content
size:(NSInteger)size
page:(NSInteger)page
success:(void (^)(NSArray *recordArray))successBlock
error:(void(^)(CCError *error))errorBlock;

/**
创建讨论组

@param discussionId 讨论组ID
@param userIdList 讨论组成员列表
@param successBlock 返回 true 成功 false失败
@param errorBlock 错误信息
*/
- (void)createDiscussion:(NSString *)discussionId
userIdList:(NSArray *)userIdList
success:(void (^)(BOOL success,NSString *message))successBlock
error:(void (^)(CCError *error))errorBlock;

/**
讨论组添加成员

@param discussionId 讨论组ID
@param userIdList 新添讨论组成员
@param successBlock 返回 讨论组成员列表
@param errorBlock 错误信息

备注

members内为添加成员后讨论组成员列表，如果讨论组成员之前就存在讨论组中，也会返回成功
*/
- (void)addMemberToDiscussion:(NSString *)discussionId
userIdList:(NSArray *)userIdList
success:(void (^)(NSArray *memberList))successBlock
error:(void (^)(CCError *error))errorBlock;

/**
获取讨论组成员列表

@param discussionId 讨论组ID
@param successBlock 返回 讨论组成员列表
@param errorBlock 错误信息
*/
- (void)getDiscussion:(NSString *)discussionId
success:(void (^)(NSArray *memberList))successBlock
error:(void (^)(CCError *error))errorBlock;
/**
将用户移出讨论组

@param discussionId 讨论组ID
@param userIds 移除讨论组成员
@param successBlock 返回 讨论组成员列表
@param errorBlock 错误信息

备注

members内为移除成员后讨论组成员列表，如果讨论组成员之前不存在，也会返回成功
*/
- (void)removeMemberFromDiscussion:(NSString *)discussionId
userIds:(NSArray *)userIds
success:(void (^)(NSArray *memberList))successBlock
error:(void (^)(CCError *error))errorBlock;


```
//获取本地消息

```
/*!
获取某个会话中指定数量的最新消息实体

@param conversationType     会话类型
@param sid                目标会话ID
@param count               需要获取的消息数量
@return                    消息实体CCMessage对象列表

@discussion 此方法会获取该会话中指定数量的最新消息实体，返回的消息实体按照时间从新到旧排列。
如果会话中的消息数量小于参数count的值，会将该会话中的所有消息返回。
*/
- (NSArray *)getLatestMessages:(CCConversationType )conversationType
sid:(NSString *)sid
count:(int)count;
/*!
获取会话中，从指定消息之前、指定数量的最新消息实体

@param conversationType     会话类型
@param sid            目标会话ID
@param oldestMessageId     截止的消息ID
@param count               需要获取的消息数量
@return                    消息实体CCMessage对象列表

@discussion 此方法会获取该会话中，oldestMessageId之前的、指定数量的最新消息实体，返回的消息实体按照时间从新到旧排列。
返回的消息中不包含oldestMessageId对应那条消息，如果会话中的消息数量小于参数count的值，会将该会话中的所有消息返回。
如：
oldestMessageId为10，count为2，会返回messageId为9和8的CCMessage对象列表。
*/
- (NSArray *)getHistoryMessages:(CCConversationType )conversationType
sid:(NSString *)sid
oldestMessageId:(NSNumber *)oldestMessageId
count:(int)count;
/*!
通过messageId获取消息实体

@param messageId   消息ID（数据库索引唯一值）
@return            通过消息ID获取到的消息实体，当获取失败的时候，会返回nil。
*/
- (CCMessage *)getMessage:(NSNumber *)messageId;
/**
删除本地消息

@param messageId 消息ID（数据库索引唯一值）
@return ture 删除成功  false 删除失败
*/
- (BOOL)deleteMessage:(NSNumber *)messageId;
/*!
获取会话中的草稿信息

@param conversationType     会话类型
@param targetId            会话目标ID
@return                    该会话中的草稿
*/
- (MessageDraft *)getTextMessageDraft:(CCConversationType )conversationType
targetId:(NSString *)targetId;

/*!
保存草稿信息

@param messageDraft        草稿对象
@return                    是否保存成功
*/
- (BOOL)saveTextMessageDraft:(MessageDraft *)messageDraft;

/*!
删除会话中的草稿信息

@param cType     会话类型，同Session.type
1    单人会话
3    讨论组会话
@param targetId            会话目标ID
@return                    是否删除成功
*/
- (BOOL)clearTextMessageDraft:(NSNumber *)cType
targetId:(NSString *)targetId;

```

  
