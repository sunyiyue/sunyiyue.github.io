---
layout: post
title: CoCall SDK文档
date: 2018-03-30 13:05:00.000000000 +09:00
---




### 消息对象 CCMessage (NSObject)

|   字段    |  类型   |                       含义                       |
| :-------: | :-----: | :----------------------------------------------: |
|   messageId   |  NSString   |          本地存储的消息的唯一值（数据库索引唯一值）           |
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
|    id     | NSString |                      会话Id                      |
|   type    | NSNumber |             会话类型，见CCConversationType              |
|   draft    | NSString  | 该会话中的草稿 |
| readMsgId |  NSNumber   |                    已读消息ID                    |
| timestamp |  NSNumber   |                  会话变更时间戳                  |
|    cnt    | NSNumber |                   未读消息数量                   |


### 会话类型  CCConversationType (枚举类型)

|   字段    |  对应值   |                       含义                       |
| :-------: | :-----: | :----------------------------------------------: |
|    CCSingleConversation     | 1 |   单人会话类型                  |
|    CCGroupConversation     | 3 |   讨论组会话类型                 |
|    CCPublicConversation     | 11 |   公众号                 |

### 草稿对象  MessageDraft

|   字段    |  类型   |                       含义                       |
| :-------: | :-----: | :----------------------------------------------: |
|    text     | NSString |   草稿信息                   |
|    cType     | NSNumber|      会话类型,同CCSession.type           |
|    sid     | NSString|      会话ID，同CCSession.id          |

### 错误信息  CCError

|   字段    |  类型   |                       含义                       |
| :-------: | :-----: | :----------------------------------------------: |
|    code     | NSInteger |   返回代码值                 |
|    message     | NSString |   请求数据失败原因                 |



###使用方法 


必须：把CCIMLib.framework和 CCData.bundle 添加到工程中


```
        [CCIMClient sharedCCIMClient].ccService = b@"172.16.11.172";
        [CCIMClient sharedCCIMClient].ccPort = @"8091";
        //系统初始化方法
        [[CCIMClient sharedCCIMClient] userRegisterWithToken: token pushId:pushId success:^(NSString *userId) {
            UIAlertView *alert = [[UIAlertView alloc] initWithTitle:@"提示" message:@"登录成功" delegate:nil cancelButtonTitle:nil otherButtonTitles:@"确定", nil];
            [alert show];
            NSLog(@"%@", userId);

        } error:^(CCError *error) {
            NSLog(@"%@", error.message);
            UIAlertView *alert = [[UIAlertView alloc] initWithTitle:@"提示" message:error.message delegate:nil cancelButtonTitle:nil otherButtonTitles:@"确定", nil];
            [alert show];
        }];
        接收到推送实现sdk中的方法 并在消息列表页和消息详情页设置delegate并实现协议（- (void)onMessageReceived;- (void)onMessageRecalled:(NSNumber *)msgId；）
        //appdelegate.m
- (void)application:(UIApplication *)application didReceiveRemoteNotification:(NSDictionary *)userInfo fetchCompletionHandler:(void (^)(UIBackgroundFetchResult))completionHandler {
    [[CCAppEngine shareEngine] application:application didReceiveRemoteNotification:userInfo fetchCompletionHandler:completionHandler];
}
// iOS 10收到通知
- (void)userNotificationCenter:(UNUserNotificationCenter *)center willPresentNotification:(UNNotification *)notification withCompletionHandler:(void (^)(UNNotificationPresentationOptions options))completionHandler API_AVAILABLE(ios(10.0)){
    
    [[CCAppEngine shareEngine] userNotificationCenter:center willPresentNotification:notification withCompletionHandler:completionHandler];
}

//发送消息
    CCTextMessage *mess = [[CCTextMessage alloc] initWithContent:@"测试消息"];
	CCMessage *message = [[CCIMClient sharedCCIMClient] sendMessage:_type targetId:_sId content:mess success:^(NSString *messageId) {
       
    } error:^(CCError *error, NSString *messageId) {
        NSLog(@"%@", error.message);
    }];
        
```



### 初始化、消息相关方法入口 CCIMClient
```


/**
 系统初始化方法
 
 @param pushId  推送使用的deviceToken
 @param successBlock 用户ID
 @param errorBlock 错误信息
 */

- (void)userRegisterWithToken:(NSString *)token
                       pushId:(NSString *)pushId
                      success:(void (^)(NSString *userId))successBlock
                        error:(void(^)(CCError *error))errorBlock;

/**
 退出登录，sdk将不会接受到服务器的消息推送

 @param successBlock 成功消息
 @param errorBlock 错误信息
 */
- (void)userUnregisterSuccess:(void (^)(BOOL success,NSString *message))successBlock
                        error:(void(^)(CCError *error))errorBlock;
#pragma mark - 消息接收提醒
/**
 设置消息接收监听器
 
 @param delegate 消息接收监听器
 */
- (void)setReceiveMessageDelegate:(id<CCReceiveMessageDelegate>)delegate;

#pragma mark - 输入状态提醒

/*!
 设置输入状态的监听器
 
 @param delegate 输入状态的的监听器
 
 @warning 目前仅支持单聊。
 */
- (void)setCCTypingStatusDelegate:(id<CCTypingStatusDelegate>)delegate;

#pragma mark - 消息相关
/**
 发送文字消息
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
                   success:(void (^)(NSString *messageId))successBlock
                     error:(void(^)(CCError *error,NSString *messageId))errorBlock;


/**
 发送媒体消息(上传图片、语音或文件媒体信息到指定的服务器)

 @param conversationType conversationType 会话类型
 @param targetId 联系人ID/讨论组ID
 @param messageContent 消息的内容
 @param uploadPrepareBlock 媒体文件上传进度更新的UI监听
 @param progressBlock 消息发送进度更新的回调
 @param successBlock 消息发送成功的回调 [messageId:消息的ID]
 @param errorBlock 消息发送失败的回调
 @return 发送的消息实体
 */
- (CCMessage *)sendMediaMessage:(CCConversationType)conversationType
                  targetId:(NSString *)targetId
                   content:(CCMessageContent *)messageContent
             uploadPrepare:(void (^)(CCUploadMediaStatusListener *uploadListener))uploadPrepareBlock
                  progress:(void (^)(int progress, NSString *messageId))progressBlock
                   success:(void (^)(NSString *messageId))successBlock
                     error:(void(^)(CCError *error,NSString *messageId))errorBlock;


/**
 发送自定义消息
                                            必填
 @param conversationType 会话类型             是
 @param targetId 联系人ID/讨论组ID             是
 @param messageContent 消息的内容              是
 @param successBlock 消息ID
 @param errorBlock 错误信息
  */
- (CCMessage *)sendCustomMessage:(CCConversationType)conversationType
                        targetId:(NSString *)targetId
                         content:(CCMessageContent *)messageContent
                         success:(void (^)(NSString *messageId))successBlock
                           error:(void(^)(CCError *error,NSString *messageId))errorBlock;


/**
 获取最近联系人/讨论组
 
 @param size 获取最新会话列表最大条数，默认200
 @param successBlock 联系人/讨论组信息组成的数组
 @param errorBlock 错误信息 从本地数据读取的最新联系人数据
 */
- (void)getRecentContact:(NSInteger)size
                 success:(void (^)(NSArray *contactArray))successBlock
                   error:(void(^)(CCError *error,NSArray *contactArray))errorBlock;


/**
 获取某条消息ID之后的消息 重复调用该接口可以获取该消息ID之后的所有消息 获取消息列表按时间逆序返回
 
 @param sid 会话ID
 @param conversationType 会话类型
 @param size 获取消息列表长度，默认50
 @param messageId 本地存储的消息ID，获取该消息之后的消息, 如果messageId为空会获取当前会话最新size条数据
 @param successBlock recordArray msgs内为获取到的消息列表
 @param errorBlock 错误信息
 */
- (void)getNextRecord:(NSString *)sid
                 type:(CCConversationType)conversationType
                 size:(NSInteger)size
            messageId:(NSString *)messageId
              success:(void (^)(NSArray *recordArray))successBlock
                error:(void(^)(CCError *error))errorBlock;

/**
 获取某条消息ID之前更多的消息记录 如果没有消息ID，则表明获取最新的消息记录 消息数据按时间倒序返回
 
 @param sid 会话ID
 @param conversationType 会话类型
 @param size 获取消息列表长度，默认50
 @param messageId 本地存储的消息ID，获取该消息之后的消息  *****不存在则获取最新size条消息*****
 @param successBlock more为本次获得数据后续是否还存在数据 msgs内为获取到的消息列表
 @param errorBlock 错误信息
 */
- (void)getMoreRecord:(NSString *)sid
                 type:(CCConversationType)conversationType
                 size:(NSInteger)size
            messageId:(NSString *)messageId
              success:(void (^)(NSArray *recordArray))successBlock
                error:(void(^)(CCError *error))errorBlock;

/**
 获取会话中成员的已读状态
 
 @param sid 会话ID
 @param conversationType 会话类型
 @param timestamp 最后更新时间
 使用 setReceiveMessageDelegate 方法设置代理   并实现此方法 onMessageReceiptResponse: 就会接收到该会话下成员的已读信息
 */

- (void)getReadStat:(NSString *)sid
               type:(CCConversationType)conversationType
          timestamp:(long)timestamp;

/**
 撤回自己发送的消息
 
 @param sid 会话ID
 @param conversationType 会话类型
 @param msgId 待撤回的消息ID
 @param successBlock 置已读成功success返回true
 @param errorBlock 错误信息
 */
- (void)chatRecall:(NSString *)sid
              type:(CCConversationType)conversationType
             msgId:(NSNumber *)msgId
           success:(void (^)(BOOL success))successBlock
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
 注意：会话列表 会话类型 已读消息id的顺序要严格对应
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
 接收到对方正在输入的推送会调用次方法
 设置 setCCTypingStatusDelegate 代理并实现协议（- (void)onTypingStatusChanged:(CCConversationType)conversationType sid:(NSString *)sid）的页面会接收到数据
 @param cType 会话类型 @warning 目前仅支持单聊
 @param sid 会话id
 */
- (void)receiveTypingPushWithCType:(int)cType
                               sid:(NSString *)sid;
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
 删除本地消息
 
 @param msgId 服务器中消息ID
 @return ture 删除成功  false 删除失败
 */
- (BOOL)deleteMessageByMsgId:(NSNumber *)msgId;

/**
 通过msgId获取消息实体
 
 @param msgId 服务器中消息ID
 @return ture 删除成功  false 删除失败
 */
- (CCMessage *)getMessageByMsgId:(NSNumber *)msgId;
/*!
 通过messageId获取消息实体
 
 @param messageId   消息ID（数据库索引唯一值）
 @return            通过消息ID获取到的消息实体，当获取失败的时候，会返回nil。
 */
- (CCMessage *)getMessageByMessageId:(NSString *)messageId;
/**
 获取某一会话中本地已存储的最新的一条消息
 @param conversationType 会话类型
 @return 通过消息ID获取到的消息实体，当获取失败的时候，会返回nil。
 */
- (CCMessage *)getSessionNewMessageBySid:(NSString *)sid
                                    type:(CCConversationType)conversationType;

/**
 删除本地消息
 
 @param messageId 消息ID（数据库索引唯一值）
 @return ture 删除成功  false 删除失败
 */
- (BOOL)deleteMessageByMessageId:(NSString *)messageId;
#pragma mark - 群组相关
/**
 获取时间戳之后变化的讨论组信息
 登录初始化之后使用 以及 收到群组相关消息推送后调用
 @param successBlock 更新成功 或 失败
 */
- (void)queryDiscussionSuccess:(void (^)(BOOL success))successBlock;
/**
 创建讨论组
 
 @param userIdList 讨论组成员列表
 @param successBlock 返回 true 成功 false失败
 @param errorBlock 错误信息
 */
- (void)createDiscussionByUserIdList:(NSArray *)userIdList
                             success:(void (^)(BOOL success,CCGroupInfo *group,NSString *message))successBlock
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
 获取讨论组信息 （成员列表 和 免打扰设置）


 @param discussionId 讨论组ID
 @return 讨论组信息
 */
- (CCGroupInfo *)getDiscussionInfoByDiscussionId:(NSString *)discussionId;

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

/**
 设置讨论组免打扰功能

 @param discussionId 讨论组ID
 @param muted 讨论组是否免打扰
 @param successBlock YES  NO
 @param errorBlock 错误信息
 */
- (void)muteDiscussionById:(NSString *)discussionId
                     muted:(BOOL)muted
                   success:(void (^)(BOOL success))successBlock
                     error:(void (^)(CCError *error))errorBlock;

#pragma mark - 草稿相关
/*!
 保存草稿信息
 
 @param messageDraft        草稿对象
 @return                    是否保存成功
 */
- (BOOL)saveTextMessageDraft:(MessageDraft *)messageDraft;
/*!
 获取会话中的草稿信息
 
 @param conversationType     会话类型
 @param targetId            会话目标ID
 @return                    该会话中的草稿
 */
- (MessageDraft *)getTextMessageDraft:(CCConversationType )conversationType
                             targetId:(NSString *)targetId;
- (NSInteger)getBadgeNumber;
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

  
