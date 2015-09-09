#前言

首发在 Segment Fault

（求star，follow ^_^）

之前总结分享的内容在实际 iOS 开发过程中都能起到一定的辅助，那么本篇文章就着重归类一些平时开发常用的代码模块部分，那种经常使用但容易出错或者漏掉细节的内容。除此之外，还有一些优化方法的归纳，特别是`UITableView`，使用频率相当高。

内容慢慢添加，涉及的可能会很多，写这些的目的是为了提醒自己，不要在一些细节上弄出问题。

##多线程

多线程开发在 iOS 开发中是非常强大而且能提升性能的手段，特别是在网络请求部分，结合 block 使用，不仅要考虑到线程问题，还有 block 块的循环引用，所以必须要特别强调，时刻提醒自己。

### block 的注意事项

使用`copy`修饰属性

想不循环应用，那么在 block 外面这样声明

    __weak __typeof(self)weakSelf = self;

接着，在 block 里面这样

    __strong __typeof(weakSelf)strongSelf = weakSelf;
    
这样既防止循环应用，又避免 block 内部 self 会无效的可能

`AFNetworking`里面这样使用的很多

### GCD

异步请求，我只认准 GCD | GCD异步，你值得拥有 （广告先走一波）

**网络请求放在子线程，UI 只能在主线程更新**

    dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^(void) {
        // 处理各种耗时间的事情，比如网络请求数据，天知道要什么时候才能结束
        
        dispatch_async(dispatch_get_main_queue(), ^(void) {
            
            // 好了以后，我们回到主线程进行界面刷新
        });
    });

**只执行一次**

Xcode 自带的代码块
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        
    });

**延迟执行**

Xcode 自带的代码块

    double delaySeconds = 5.0;
    dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(delaySeconds * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
        
    });
    
**自定义队列**    
    dispatch_queue_t baidu_queue = dispatch_queue_create("baidu.com", NULL);
    
    dispatch_async(baidu_queue, ^{
        
    });
    
**并行队列**

**串行队列**

### NSOperation


# UIKit

###新特性
版本更新或者第一次下载，我们都要看的页面，对 app 的初步介绍

https://github.com/nsdictionary/CoreNewFeatureVC

###侧滑菜单

https://github.com/romaonthego/RESideMenu

###图片浏览器

点击一张图片，全屏浏览，背景为黑色

https://github.com/mwaterfall/MWPhotoBrowser

###HUD

有两个非常好用

- SVProgressHUD

https://github.com/TransitApp/SVProgressHUD

- MBProgressHUD

https://github.com/jdg/MBProgressHUD

###`UILabel`

非常高能，支持各种富文本

https://github.com/TTTAttributedLabel/TTTAttributedLabel

###`UINavigationBar`

向上滑动的时候，`UINavigationBar`会跟随滚动消失，反之会显示

1. https://github.com/andreamazz/AMScrollingNavbar
2. http://old.code4app.com/ios/XHYScrollingNavBarViewController/531d5768933bf0a83e8b6ac0

可伸缩的头部视图

http://www.cocoachina.com/bbs/read.php?tid=288489

###`UITableView` 的优化

- 最基本的，cell 重用机制，如果这个都不知道的话，那可以去撞墙了。。。

- 异步加载数据，这个也是很基本的

- 自动载入更新数据，比如每次载入20条信息，然后在滚动到最后5条信息，就加载更多的信息

- 图片下载完成以后，判断 cell 如果是可见的，那么就需要更新图像


###`UIPasteboard` 粘贴板

`UITextView`，`UITextField`，`UIWebView`这三个控件自带复制功能，而`UILabel`和`UIImageView`需要手动修改，才有复制功能，这个时候用到`UIPasteboard`

相关地址：http://blog.csdn.net/chengwuli125/article/details/8588554



#数据

##数据持久化

###沙盒
**访问沙盒里面几个路径**
```objective-c
//获取根目录
NSString *homePath = NSHomeDirectory();
NSLog(@"Home目录：%@",homePath);
```
    //获取Documents文件夹目录
    NSArray *docPath = NSSearchPathForDirectoriesInDomains(NSDocumentDirectory,NSUserDomainMask, YES);
    NSString *documentsPath = [docPath objectAtIndex:0];
    NSLog(@"Documents目录：%@",documentsPath);

   
    //获取Cache目录
    NSArray *cacPath = NSSearchPathForDirectoriesInDomains(NSCachesDirectory, NSUserDomainMask, YES);
    NSString *cachePath = [cacPath objectAtIndex:0];
    NSLog(@"Cache目录：%@",cachePath);   


    //获取Library目录
    NSArray *libsPath = NSSearchPathForDirectoriesInDomains(NSLibraryDirectory, NSUserDomainMask, YES);
    NSString *libPath = [libsPath objectAtIndex:0];
    NSLog(@"Library目录：%@",libPath);


    //获取temp目录
    NSString *tempPath = NSTemporaryDirectory();
    NSLog(@"temp目录：%@",tempPath);
    
**写入与读取**

    NSArray *testArray1 = @[@"1",@"2"];
    //documentsPath是前面创建的
    NSString *filePath = [documentsPath stringByAppendingPathComponent:@"testArray1.text"];
    //写入
    [testArray1 writeToFile:filePath atomically:YES];
    //读取
    NSArray *readTestArray1 = [NSArray arrayWithContentsOfFile:filePath];
    
 
 **文件管理**
    
    //获取文件管理器
    NSFileManager *fileManager = [NSFileManager defaultManager];
    
    //文件目录名
    NSString *testDirectory = [documentsPath stringByAppendingPathComponent:@"ttttest"];
    
    //执行创建
    [fileManager createDirectoryAtPath:testDirectory withIntermediateDirectories:YES attributes:nil error:nil];   
    
**在刚才创建的文件夹下继续写入内容**   
     
    //在文件目录下继续写入的路径
    NSString *tPath = [testDirectory stringByAppendingPathComponent:@"t1.txt"];
    
    //要写入内容
    NSString *contentString = @"ready...";
    
    //执行写入
    [fileManager createFileAtPath:tPath contents:[contentString dataUsingEncoding:NSUTF8StringEncoding] attributes:nil];
    
**获取一个目录下面所有的子文件**  
    //获取documents下面的所有文件，可以看到隐藏的内容
    NSArray *file = [fileManager subpathsOfDirectoryAtPath:documentsPath error:nil];
    
    NSLog(@"file%@",file);
    
    NSArray *file2 = [fileManager subpathsAtPath:documentsPath];
    
    NSLog(@"file2=%@",file2);   
  
  
**改变文件管理器所能操作的位置**
  
    //移到准备操作的目录下
    [fileManager changeCurrentDirectoryPath:[documentsPath stringByExpandingTildeInPath]];
    
    //创建文件
    NSString *fileName = @"newTest.txt";
    
    NSString *tString = @"ttttttt";
    
    [fileManager createFileAtPath:fileName contents:[tString dataUsingEncoding:NSUTF8StringEncoding] attributes:nil];
    
 **删除文件**
    //接着上面，就一句话
    [fileManager removeItemAtPath:fileName error:nil];
    

###归档与解档

遵循`NSCoding`协议，然后实现那两个协议方法

`MJExtension`可以快速实现，不用写恶心的代码

###属性列表

`NSUserDefault`

记得存的时候同步一下，因为存储不是立即的

###数据库

系统自带的 API 实在不好用，`FMDB`的封装性非常高，专注于写 sql 语句即可

https://github.com/ccgus/fmdb

（特别注意FMDB的多线程处理）

###Core Data

同样，原生的内容复杂，`Magical Record`是很好的解决方案

https://github.com/magicalpanda/MagicalRecord

（新版本过期了一些方法）

#（未完待续）

#总结
随着 iOS 不断的发展，我们总是在接触新的知识，在不断的学习过程中，有那么一些细节性的内容总是会遗漏，以前总没有好的习惯来总结，以至于会疏忽一些问题从而导致不必要的麻烦。

