# iOSCoreNFC
This sample shows how to integrate Core NFC Framework into your application to enable NFC tag reading.


---
#### About Core NFC 
##### Core NFC支持的读取数据类型：
![image](http://note.youdao.com/yws/public/resource/357231ddfcbd2ad2f747cdb296ffb119/xmlnote/WEBRESOURCEafc5a7fd5f348a07cb90574a7a113202/11238)

##### Core NFC框架特性/要求
- 目前支持NFC Tags（标签）的读取
- 不支持输出和格式设置
- 仅支持iphone 7 & iphone 7plus，且iOS11系统

---
#### 项目加入Core NFC框架使用的要求
- （必须）支持iOS11，且只有iOS11和iphone7/plus机型才可以
- （必须）像Apple pay或者Push Notification一样，需要添加一个entitlement
- （保修）在plist文件中添加Privacy - NFC Scan Usage Description。这里使用的描述信息会显示在读取界面中

###### 要求示例如图：
![image](http://note.youdao.com/yws/public/resource/357231ddfcbd2ad2f747cdb296ffb119/xmlnote/WEBRESOURCEf04efe488cba4f9f77013612e0df8824/11242)

![image](http://note.youdao.com/yws/public/resource/357231ddfcbd2ad2f747cdb296ffb119/xmlnote/WEBRESOURCE97560d5f1c9e914a956b795b3463b1a1/11240)

##### 集成Core NFC中的一些细节说明
- 设备读取标签是一个被动的过程，所以需要程序主动发起一个会话即为session去读取标签。与处理摄像头相关功能类似，所有的操作都必须建立在session基础之上
- 程序必须始终保持前台运行并且识别界面可视。如果至于后台session会自动终止，读取失败。
    -   Tips：这里我做过一个测试，实际上当Core NCF读取标签界面出现后，无法下拉通知栏中心，也无法上滑出现控制中心，如果识别过程中，点击home第一次会取消识别，不会直接进入主屏幕。这样的设计应该是为了防止在识别过程中出现误操作等情况的发生
-  读取标签被限制的60秒之内。意思就是60秒内标签必须识别完成，否则session会自动终止。如果会话过期或者未经过验证，则你的程序需要重新去建立新的会话
-  Core NFC可以设置会话读取一个标签或者多个标签。在读取单个标签的时候，读取完成后，会话自动终止。如果读取多个标签，会话会一直持续直到程序主动终止会话或者60秒后。60秒是一个最大的节点


---

#### 示例代码
##### 其实Core NFC目前放出的权限很少，只支持特定格式的NFC数据读取，不支持输出和格式设置，所以代码上并不难，可以说是傻瓜式的调用处理即可。我猜想可能是因为Apple为了保证Apple Pay的安全性，毕竟Apple Pay也是采用NFC完成支付。

###### 使用Core NFC
- @import CoreNFC 导入框架，这点没啥可说的
- 遵循 NFCNDEFReaderSessionDelegate 协议
- 创建 NFCNDEFReaderSession 实例
- 开启 NFCNDEFReaderSession 以及处理协议回调方法

###### 具体代码如下如下

```
// @import CoreNFC 导入框架
// 遵循 NFCNDEFReaderSessionDelegate 协议
#import "ViewController.h"
#include <sys/types.h>
#include <sys/sysctl.h>

@import CoreNFC;
@interface ViewController ()<NFCNDEFReaderSessionDelegate>
@end
@implementation ViewController

// 创建 NFCNDEFReaderSession 实例，开启NFCNDEFReaderSession
// Tips：开启 
// 条件：iphone7/7plus运行iOS11
if ([ViewController isiPhone7oriPhone7Plus] && [UIDevice currentDevice].systemVersion.floatValue >= 11.0) {
    // ReadingAvailable is YES if device supports NFC tag reading.
    if ([NFCNDEFReaderSession readingAvailable]) {
        // beginScanning
        // invalidateAfterFirstRead 属性表示是否需要识别多个NFC标签，如果是YES，则会话会在第一次识别成功后终止。否则会话会持续
        // 不过有一种例外情况，就是如果响应了-readerSession:didInvalidateWithError:方法，则是否为YES，会话都会被终止
        NFCNDEFReaderSession *session = [[NFCNDEFReaderSession alloc] initWithDelegate:self queue:nil invalidateAfterFirstRead:YES];
        
        [session beginSession];
    }
}

// 处理协议回调方法
#pragma mark - NFCReaderSessionDelegate
// Check invalidation reason from the returned error. A new session instance is required to read new tags.
// 识别出现Error后会话会自动终止，此时就需要程序重新开启会话
- (void)readerSession:(NFCNDEFReaderSession *)session didInvalidateWithError:(NSError *)error {
    // error明细参考NFCError.h
    NSLog(@"%@",error);
}

// Process detected NFCNDEFMessage objects
- (void)readerSession:(NFCNDEFReaderSession *)session didDetectNDEFs:(NSArray<NFCNDEFMessage *> *)messages {
    // 数组messages中是NFCNDEFMessage对象
    // NFCNDEFMessage对象中有一个records数组，这个数组中是NFCNDEFPayload对象
    // 参考NFCNDEFMessage、NFCNDEFPayload类
    // 解析数据
    for (NFCNDEFMessage *message in messages) {
        for (NFCNDEFPayload *playLoad in message.records) {
            NSLog(@"typeNameFormat : %d", playLoad.typeNameFormat);
            NSLog(@"type : %@", playLoad.type);
            NSLog(@"identifier : %@", playLoad.identifier);
            NSLog(@"playload : %@", playLoad.payload);
        }
    }
}

// 主动终止会话，调用如下方法即可。
[session invalidateSession];
```