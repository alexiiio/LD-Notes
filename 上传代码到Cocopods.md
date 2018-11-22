Cocopods作为iOS的第三方库管理工具，非常方便易用。之前都是使用别人的三方库，自己没有上传过代码，今天就一边学习一边记录下具体操作过程。

# 准备工作
- 首先**准备要上传的代码**，这里我写了一个GCD timer的简单工具类。


- **创建github仓库。**    

![image](https://github.com/alexiiio/LD-Notes/blob/master/pics/屏幕快照%202018-11-22%20下午1.13.49.png?raw=true)

- **然后`clone`到本地，把代码上传到github仓库**。        

红色的是我们最终要上传的文件，最好放到单独一个文件夹里，相关路径后面会用到。   
![image](https://github.com/alexiiio/LD-Notes/blob/master/pics/屏幕快照%202018-11-22%20下午8.57.00.png?raw=true)


- **添加tag。** 点击release按钮->选择Create a new release。
![image](https://github.com/alexiiio/LD-Notes/blob/master/pics/屏幕快照%202018-11-22%20下午4.15.34.png?raw=true)


![image](https://github.com/alexiiio/LD-Notes/blob/master/pics/io98$2018-11-22$2.png?raw=true)

填写版本号和描述。

![iamge](https://github.com/alexiiio/LD-Notes/blob/master/pics/屏幕快照%202018-11-22%20上午11.20.17.png?raw=true)


# 创建.podspec文件
- **cd到本地工程目录**
```
cd /Users/xxx/LDGCDTimer
```
- **创建`.podspec`文件，以工程名命名。**
```
touch LDGCDTimer.podspec
```
- **用Xcode或Sublime Text打开`podspec`文件（把文件拖拽到Xcode图标上），把下面内容复制进去。**
```
Pod::Spec.new do |s|
s.name = 'LDGCDTimer'
s.version = '0.0.1'
s.license = 'MIT'
s.summary = '一个GCD timer工具。'
s.description = '一个GCD timer简单工具类。'
s.homepage = 'https://github.com/alexiiio/LDGCDTimer'
s.author = { 'alexiiio' => '450145524@qq.com' }
s.source = { :git => "https://github.com/alexiiio/LDGCDTimer.git", :tag => "0.0.1"}
s.requires_arc = true
s.ios.deployment_target = '7.0'
s.source_files = "LDGCDTimer/LDGCDTimer.h","LDGCDTimer/LDGCDTimer.m"
s.frameworks = 'UIKit'
end
```
相应内容对应的含义如下，修改成自己的项目信息。
```
s.name ： 工程名 
s.version ： 版本，要和github上的仓库tag对应
s.license ：授权，前面创建仓库选择的类型
s.summary ：简述 
s.description ： 描述 
s.homepage ： github仓库主页 
s.author ： 作者名字和联系邮箱
s.source : github仓库地址（.git），以及版本tag 
s.requires_arc ：是否是ARC 
s.ios.deployment_target ： 你支持的最低版本 
s.source_files ： 资源文件，此处是我们最重要上传的代码所在路径。起始路径跟podspec路径同级。这里很容易出错，需注意！
s.framework ：所需的framework，多个用英文逗号隔开

其他
s.dependency：依赖库，不能依赖未发布的库，可以写多个依赖库
s.social_media_url:社交网址
s.public_header_files:公开的头文件
```
- **修改好之后验证，在终端输入：**
```
pod spec lint
```
终端会提示哪些格式错误，修改之后一直报错：  
![image](https://github.com/alexiiio/LD-Notes/blob/master/pics/屏幕快照%202018-11-22%20下午7.03.17.png?raw=true)   
搞了好久，查了不少资料，最后发现工程命名错了[吐血],两个英文字母颠倒了，看了多少遍都没看出来......    
![image](https://ww1.sinaimg.cn/large/6af89bc8gw1f8nufnvwqoj206r06qmx8.jpg)   
之后重建项目，检查命名，上传github，修改tag位置。还是报`- ERROR | [iOS] file patterns: The `source_files` pattern did not match any file.`错误，最后把`s.source_files`改为：   
```
s.source_files = "LDGCDTimer/LDGCDTimer/"
```
终于验证通过了！   
![image](https://github.com/alexiiio/LD-Notes/blob/master/pics/屏幕快照%202018-11-22%20下午7.35.51.png?raw=true)   
而我之前写的是`s.source_files = "LDGCDTimer/LDGCDTimer/*.{h,m}" `,按照网上说的:

> '\*'表示匹配所有文件   
> '\*.{h,m}' 表示匹配所有以.h和.m结尾的文件   
> '**' 表示匹配所有子目录    

并不应该出现这种问题，具体是因为什么就不得而知了。    

# 上传到Cocopods
- **注册上传到CocoaPods所用的账号（trunk账号）**    
> pod trunk register 邮箱 \`用户名\` –-description=\`描述\` --verbose   
注意： 邮箱必须是你注册github的邮箱，用户名最好是你github的用户名，不是应该也没关系，我没试过，你可以试试。    
这一步会给你邮箱发一条验证邮件，点击里面的链接，如果链接不可以点击，那就复制粘贴到浏览器按回车。   

终端显示：`[!] Please verify the session by clicking the link in the verification email that has been sent to xxxxx@xx.com
`    
很快就会收到一封邮件，点击邮箱里的验证链接，显示验证成功：   
![image](https://github.com/alexiiio/LD-Notes/blob/master/pics/WechatIMG560.jpeg?raw=true)    
- **向trunk服务器查询自己的注册信息。**
```
pod trunk me
```
输出如下信息就是注册好了：
```
- Name:     -description=
- Email:    xxx@xx.com
- Since:    November 22nd, 06:05
- Pods:     None
- Sessions:
- November 22nd, 06:05 - March 30th, 2019 06:11. IP: xx.xx.xx.xx
```
- **通过trunk推送podspec**
```
pod trunk push xxx.podspec
```
成功的信息：   
![image](https://github.com/alexiiio/LD-Notes/blob/master/pics/屏幕快照%202018-11-22%20下午8.36.12.png?raw=true)    

大功告成！接下来更新一下仓库，检查能否使用。    

------

**更新仓库**
```
pod repo update
```
**搜索上传的版本**
```
pod search xxxx
```
![image](https://github.com/alexiiio/LD-Notes/blob/master/pics/屏幕快照%202018-11-22%20下午8.46.52.png?raw=true)   

----
整个流程就是这样，并不难，但也因为一些问题搞了一整天。[衰]总之也是走通了。
![欣慰](https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1542902282747&di=19bafef1992a28877666851b84277c3c&imgtype=0&src=http%3A%2F%2Fspider.nosdn.127.net%2F4884045dbfd58d7ec1f9e364f66304f5.jpeg)    
希望以后能写出高质量的代码，为开源事业做贡献！
