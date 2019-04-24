
每一个APP都有一个独立的存储空间，就是沙盒。沙盒之间是相互隔离的，iOS应用不允许访问其他应用的沙盒路径。

沙盒路径包含四个文件夹： Documents、Library、tmp、SystemData和一个.app
包。

**沙盒文件目录**：

![sandboxfilecatalog](https://raw.githubusercontent.com/alexiiio/LD-Notes/master/pics/sandboxfilecatalog.png)

获取沙盒主目录路径    
```
NSString *homePaht = NSHomeDirectory();
```



一、**Documents**

用于存储用户数据。该路径可通过配置实现iTunes共享文件，可被iTunes备份。保存应用运行时生成的需要持久化的数据，苹果建议将程序中建立的或在程序中浏览到的文件数据保存在该目录下。


```
// 获取Documents目录路径
NSString *documentPath = NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES).firstObject;
```
二、**Library**

可创建子文件夹。该路径下的文件夹，除Caches以外，都会被iTunes备份。

Library目录下有两个子目录：

**Library/Caches**：存放缓存文件，iTunes不会备份此目录，此目录下文件不会在应用退出删除。一般存放体积比较大，不是特别重要的资源。

**Library/Preferences**：包含应用程序的偏好设置文件，您不应该直接创建偏好设置文件，而是应该使用NSUserDefaults类来取得和设置应用程序的偏好。存放NSUserDefaults保存的.plist文件，iTunes会自动备份该目录。


```
//  Libaray目录路径
NSString *LibarayPath = NSSearchPathForDirectoriesInDomains(NSLibraryDirectory, NSUserDomainMask, YES).firstObject;
// Library/Caches目录路径
NSString *cachePath = NSSearchPathForDirectoriesInDomains(NSCachesDirectory, NSUserDomainMask, YES).firstObject;
```

三、**tmp**

存放临时文件，iTunes不会备份和恢复此目录，此目录下文件可能会在应用退出后删除。应用没有运行时，系统也有可能会清除该目录下的文件。iPhone在重启时，会丢弃所有的tmp文件。
```
//  tmp目录路径
NSString *tmpPaht = NSTemporaryDirectory();
```

四、**SystemData**

新加入的一个文件夹, 存放系统的一些东西

五、**App bundle路径**

这是应用程序的程序包目录，存放应用程序的源文件，包括资源文件和可执行文件。由于应用程序必须经过签名。所以在运行程序时，是不可以对这个目录进行内容修改的，否则会造成应用无法启动。

```
//  获取AppName.app 目录路径
NSString *path = [[NSBundle mainBundle] bundlePath];
```