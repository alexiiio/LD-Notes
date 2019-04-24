无意中通过同事手机看到了星巴克app里的一个有意思的动画。在星巴克每消费一定金额，会获取一个星星，星星装在杯子里，随着手机的旋转，星星会在杯子里翻滚移动，仿佛受重力吸引一样。出于兴趣，做了简单的模仿。大概是这个样子：

![星巴克集星](https://raw.githubusercontent.com/alexiiio/LD-Notes/master/pics/Starbucks.jpeg)

##用到的知识点
SpriteKit、物理引擎和CoreMotion框架。
这里用到的实际是游戏方面的知识，每个星星作为一个精灵(sprite)对象，有自己的质量和碰撞体积，在它们所处的物理世界内（杯子）受到重力的影响，星星碰到杯子或其他星星会反弹。

首先来创建游戏项目，当然也可以在普通项目里写，但是直接用游戏项目更简单，如图：

![新建工程](https://raw.githubusercontent.com/alexiiio/LD-Notes/master/pics/createGameProject.png)

把工程初始化时添加的不必要的代码删掉。代码写在GameScene里就好了

![工程文件列表](https://raw.githubusercontent.com/alexiiio/LD-Notes/master/pics/GameProjectFiles.png)

###创建星星
这里`SKSpriteNode`是一个游戏体系下的图形元素，可以用图片或颜色来初始化。在`Sprite Kit` 中坐标系的原点在左下角，同等长度下分辨率（点数）是iOS原生的2倍。
```
-(void)createStar{
    SKSpriteNode *node = [SKSpriteNode spriteNodeWithImageNamed:@"star"];
    node.position = CGPointMake(0, self.size.height/2);
    [self addChild:node];
    //创建五角星的路径
    CGMutablePathRef path = CGPathCreateMutable();
    CGPathMoveToPoint(path,NULL,0, 50); //1  星星的角
    CGPathAddLineToPoint(path, NULL, 20,20);
    
    CGPathAddLineToPoint(path, NULL, 50,10); // 2
    CGPathAddLineToPoint(path, NULL, 20,-20);
    
    CGPathAddLineToPoint(path, NULL, 30,-50);// 3
    CGPathAddLineToPoint(path, NULL, -20,-20);
    
    CGPathAddLineToPoint(path, NULL, -35,-50); // 4
    CGPathAddLineToPoint(path, NULL, -20,20);
    
    CGPathAddLineToPoint(path, NULL, -50,10); // 5
    
    CGPathCloseSubpath(path);
    node.physicsBody = [SKPhysicsBody bodyWithPolygonFromPath:path]; // 生成碰撞体积
}
```
通过绘制并生成了物理体积的星星就是一个有棱有角的物体了，不再是普通的方块。
可以通过`physicsBody`的相关属性来设置星星的一些物理特性，比如restitution属性来让星星更Q弹。

```
    node.physicsBody.restitution = 0.8;
```
运行工程，星星受重力影响，直接落到屏幕外。我们需要把星星限制在杯子里。
把整个屏幕作为杯子：

```
    self.physicsBody = [SKPhysicsBody bodyWithEdgeLoopFromRect:self.frame];
```
这样星星只会在杯子内弹来弹去，不会飞到屏幕外。
##重力感应
现在无论怎么旋转手机，星星只会往手机屏幕下方落。而我们需要在旋转手机的时候，让星星始终落在地球重力的方向。我的想法是，在旋转手机时，改变瓶子内的重力方向，使其始终指向地球所在方向。

这就要用到CoreMotion框架，CoreMotion框架专门处理设备的动作相关的事件，其中包含了两个部分加速度计和陀螺仪，在iOS4之前加速度计是由UIAccelerometer类来负责采集数据，现在弃用了。

导入CoreMotion框架并引入


```
#import <CoreMotion/CoreMotion.h>
```
创建CMMotionManager，开启运动检测，在手机旋转的时候更新杯子内的重力方向。
```
@implementation GameScene {
    CMMotionManager *cmManager;
}
```
```
    cmManager = [[CMMotionManager alloc]init];
    cmManager.deviceMotionUpdateInterval = 1/60.0; // 设备运动状态的更新间隔
    [cmManager startDeviceMotionUpdatesToQueue:[[NSOperationQueue alloc]init] withHandler:^(CMDeviceMotion * _Nullable motion, NSError * _Nullable error) {
        self.physicsWorld.gravity = CGVectorMake(motion.gravity.x*9.8*2, motion.gravity.y*9.8*2);
    }];
```
这样效果就实现了。实际效果可自行运行demo查看。

![实际效果.gif](https://raw.githubusercontent.com/alexiiio/LD-Notes/master/pics/starAnimation.gif)

也可以使用加速度计的方法，该方法在不旋转屏幕的前提下快速把手机往一个方向移动，星星也会向该方向移动。
```
//    [cmManager startAccelerometerUpdatesToQueue:[[NSOperationQueue alloc] init] withHandler:^(CMAccelerometerData * _Nullable accelerometerData, NSError * _Nullable error) {
//        self.physicsWorld.gravity = CGVectorMake(accelerometerData.acceleration.x*9.8*2, accelerometerData.acceleration.y*9.8*2);
//    }];
```
CoreMotion框架还有很多有趣的方法，感兴趣的可以自己尝试下。

[Github代码地址](https://github.com/alexiiio/CollectStar)