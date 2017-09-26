---
layout: post
title: NSURLCache缓存 Etag
date: 2017-04-20 16:05:00.000000000 +09:00
---





### Etag - 工作原理

Etag由服务器端生成，客户端通过If-Match或者说If-None-Match这个条件判断请求来验证资源是否修改。常见的是使用If-None-Match.请求一个文件的流程可能如下：

====第一次请求===
1.客户端发起 HTTP GET 请求一个文件；


2.服务器处理请求，返回文件内容和一堆Header，当然包括Etag(例如"2e681a-6-5d044840")(假设服务器支持Etag生成和已经开启了Etag).状态码200

====第二次请求===
1.客户端发起 HTTP GET 请求一个文件，注意这个时候客户端同时发送一个If-None-Match头，这个头的内容就是第一次请求时服务器返回的Etag：2e681a-6-5d044840


2.服务器判断发送过来的Etag和计算出来的Etag匹配，因此If-None-Match为False，不返回200，返回304，客户端继续使用本地缓存；

流程很简单，问题是，如果服务器又设置了Cache-Control:max-age和Expires呢，怎么办？
答案是同时使用，也就是说在完全匹配If-Modified-Since和If-None-Match即检查完修改时间和Etag之后，服务器才能返回304.

### NSURLCache
实现方法里，最烦的一件事就是怎么缓存接口内容，一种方法是自己写一套代码来实现缓存功能，SDWebImage就是这么实现缓存机制的，其实对于接口请求，iOS 系统已经帮你做好了缓存，而且非常完善简单，这个方式叫NSURLCache。这种方式只需两个步骤就能缓存网络接口返回内容：

第一步：客户端设置缓存大小

```
AppDelegate.m

NSURLCache *urlCache = [[NSURLCache alloc] initWithMemoryCapacity:4*1024*1024 diskCapacity:100*1024*1024 diskPath:nil];
NSURLCache.sharedURLCache = urlCache;

```

第二步：客户端发出 GET请求

第三步：Done
这两个步骤之后，GET请求的内容会被系统自动缓存了，无需自己去实现内容缓存，对于这种方法，Mattt大神说过“无数开发者尝试自己做一个简陋而脆弱的系统来实现网络缓存的功能，殊不知 NSURLCache 只要两行代码就能搞定且好上 100 倍。”

### Demo（NSURLSession）


```
- (void)viewDidLoad {
    [super viewDidLoad];    
    NSURLSessionConfiguration *config = [NSURLSessionConfiguration defaultSessionConfiguration];
    NSURLSession *session = [NSURLSession sessionWithConfiguration:config delegate:self delegateQueue:nil];
    
    NSMutableURLRequest *request = [NSMutableURLRequest requestWithURL:[NSURL URLWithString:@"https://api.penguinguide.cn/article/973"]
                                                           cachePolicy:NSURLRequestReloadIgnoringLocalCacheData
                                                       timeoutInterval:30.f];
    if (self.etag) {
        [request setValue:self.etag forHTTPHeaderField:@"If-None-Match"];
    }
    
    NSURLSessionDataTask *task = [session dataTaskWithRequest:request completionHandler:^(NSData * _Nullable data, NSURLResponse * _Nullable response, NSError * _Nullable error) {
        NSHTTPURLResponse *urlResponse = (NSHTTPURLResponse *)response;
        if (urlResponse.statusCode == 304) {
            NSCachedURLResponse *cachedResponse = [[NSURLCache sharedURLCache] cachedResponseForRequest:request];
            if (cachedResponse) {
                data = cachedResponse.data;
            }
        } else {
            if (urlResponse.allHeaderFields[@"Etag"]) {
                NSString *etag = urlResponse.allHeaderFields[@"Etag"];
                if (etag && etag.length > 0) {
                    [self saveEtag:etag];
                }
            }
        }
    }];
    [task resume];
}

                                    
```

### Demo（AFNetworking）

```
AFHTTPSessionManager *sessionManager = [[AFHTTPSessionManager alloc] initWithBaseURL:[NSURL URLWithString:baseURL] sessionConfiguration:config];
sessionManager.requestSerializer = [CustomJSONRequestSerializer serializer];
sessionManager.responseSerializer = [CustomJSONResponseSerializer serializer];

 ```
  
 ```
@interface CustomJSONRequestSerializer : AFJSONRequestSerializer
@end

@implementation CustomJSONRequestSerializer

- (NSMutableURLRequest *)requestWithMethod:(NSString *)method URLString:(NSString *)URLString parameters:(id)parameters error:(NSError *__autoreleasing  _Nullable *)error
{
    NSMutableURLRequest *request = [super requestWithMethod:method URLString:URLString parameters:parameters error:error];
   
    // add request If-None-Match header
    NSString *absoluteUrl = request.URL.absoluteString;
    NSString *directory = [NSSearchPathForDirectoriesInDomains(NSCachesDirectory, NSUserDomainMask, YES).firstObject stringByAppendingPathComponent:@"etags"];
    NSString *fileName = [directory stringByAppendingPathComponent:[self cachedFileNameForKey:absoluteUrl]];
    if ([[NSFileManager defaultManager] fileExistsAtPath:fileName]) {
        NSString *etag = [NSKeyedUnarchiver unarchiveObjectWithFile:fileName];
        if (etag && etag.length > 0) {
            [request addValue:etag forHTTPHeaderField:@"If-None-Match"];
        }
    }
    
    return request;
}

@end
 ```

 ```
@interface CustomJSONResponseSerializer : AFJSONResponseSerializer
@end

@implementation CustomJSONResponseSerializer

- (nullable id)responseObjectForResponse:(NSURLResponse *)response data:(NSData *)data error:(NSError *__autoreleasing  _Nullable *)error
{
    NSUInteger responseStatusCode = 200;
    if ([response isKindOfClass:[NSHTTPURLResponse class]]) {
        responseStatusCode = (NSUInteger)[(NSHTTPURLResponse *)response statusCode];
    }
    
    id responseObject;
    
    if (responseStatusCode == 304) {
        // fetch cached contents
        NSCachedURLResponse *cachedResponse = [[NSURLCache sharedURLCache] cachedResponseForRequest:task.currentRequest];
        
        if (cachedResponse) {
            responseObject = [super responseObjectForResponse:cachedResponse.response data:cachedResponse.data error:error];
        } else {
            // cached response was cleared, need to clear cached etag
            NSString *directory = [NSSearchPathForDirectoriesInDomains(NSCachesDirectory, NSUserDomainMask, YES).firstObject stringByAppendingPathComponent:@"etags"];
            NSString *fileName = [self cachedFileNameForKey:response.URL.absoluteString];
            NSString *cachedFileName = [directory stringByAppendingPathComponent:fileName];
            [[NSFileManager defaultManager] removeItemAtPath:cachedFileName error:nil];
        }
        
        return responseObject;
    } else if (responseStatusCode >= 200 && responseStatusCode <= 299) {
        if ([response isKindOfClass:[NSHTTPURLResponse class]]) {
            NSHTTPURLResponse *httpResponse = (NSHTTPURLResponse *)response;
            if (httpResponse.allHeaderFields[@"Etag"]) {
                NSString *etag = httpResponse.allHeaderFields[@"Etag"];
                if (etag && etag.length > 0) {
                    NSString *directory = [NSSearchPathForDirectoriesInDomains(NSCachesDirectory, NSUserDomainMask, YES).firstObject stringByAppendingPathComponent:@"etags"];
                    if (![[NSFileManager defaultManager] fileExistsAtPath:directory]) {
                        [[NSFileManager defaultManager] createDirectoryAtPath:directory withIntermediateDirectories:NO attributes:nil error:nil];
                    }
                    NSString *fileName = [self cachedFileNameForKey:response.URL.absoluteString];
                    NSString *cachedFileName = [directory stringByAppendingPathComponent:fileName];
                    BOOL success = [NSKeyedArchiver archiveRootObject:etag toFile:cachedFileName];
                    NSLog(@"");
                }
            }
        }
        responseObject = [super responseObjectForResponse:response data:data error:error];
        
        return responseObject;
    } else {
        id responseObject = [super responseObjectForResponse:response data:data error:error];
        
        if ([responseObject isKindOfClass:[NSDictionary class]]) {
            *error = [NSError errorWithDomain:NSURLErrorDomain code:responseStatusCode userInfo:responseObject];
        } else {
            *error = [NSError errorWithDomain:NSURLErrorDomain code:responseStatusCode userInfo:nil];
        }
        return nil;
    }
}

@end
 ```
  