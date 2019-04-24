
**app的生命周期：**

![applifecycle](https://raw.githubusercontent.com/alexiiio/LD-Notes/master/pics/applifecycle.png)

**app的5种状态：**

1. **Not running**      
应用还没有启动，或者应用正在运行但是途中被系统停止

2. **Inactive**         
当前应用正在前台运行，但是并不接收事件（当前或许正在执行其它代码）。一般每当应用要从一个状态切换到另一个不同的状态时，中途过渡会短暂停留在此状态。唯一在此状态停留时间比较长的情况是：当用户锁屏时，或者系统提示用户去响应某些（诸如电话来电、有未读短信等）事件的时候。
3. **Active**          
当前应用正在前台运行，并且接收事件。这是应用正在前台运行时所处的正常状态。
4. **Background**       
应用处在后台，并且还在执行代码。大多数将 要进入Suspended状态的应用，会先短暂进入此状态。然而，对于请求需要额外的执行时间的应用，会在此状态保持更长一段时间。另外，如果一个应用要求启动时直接进入后台运行，这样的应用会直接从Notrunning状态进入Background状态，中途不会经过Inactive状态。比如没有界面的应用。注此处并不特指没有界面的应用，其实也可以是有界面的应用，只是如果要直接进入background状态的话，该应用界面不会被显示。
5. **Suspended**         
应用处在后台，并且已停止执行代码。系统自动的将应用移入此状态，且在此举之前不会对应用做任何通知。当处在此状态时，应用依然驻留内存但不执行任何程序代码。当系统发生低内存告警时，系统将会将处于Suspended状态的应用清除出内存以为正在前台运行的应用提供足够的内存。


![appfiveStatus](https://raw.githubusercontent.com/alexiiio/LD-Notes/master/pics/appfiveStatus.png)

**UIViewController的生命周期：**
```
2017-03-01 18:03:41.577 ViewControllerLifeCircle[32254:401790] -[ViewController initWithCoder:] 
2017-03-01 18:03:41.579 ViewControllerLifeCircle[32254:401790] -[ViewController awakeFromNib] 
2017-03-01 18:03:41.581 ViewControllerLifeCircle[32254:401790] -[ViewController loadView] 
2017-03-01 18:03:46.485 ViewControllerLifeCircle[32254:401790] -[ViewController viewDidLoad] 
2017-03-01 18:03:46.486 ViewControllerLifeCircle[32254:401790] -[ViewController viewWillAppear:] 
2017-03-01 18:03:46.487 ViewControllerLifeCircle[32254:401790] -[ViewController viewWillLayoutSubviews] 
2017-03-01 18:03:46.488 ViewControllerLifeCircle[32254:401790] -[ViewController viewDidLayoutSubviews] 
2017-03-01 18:03:46.488 ViewControllerLifeCircle[32254:401790] -[ViewController viewWillLayoutSubviews] 
2017-03-01 18:03:46.488 ViewControllerLifeCircle[32254:401790] -[ViewController viewDidLayoutSubviews] 
2017-03-01 18:03:46.490 ViewControllerLifeCircle[32254:401790] -[ViewController viewDidAppear:] 
2017-03-01 19:03:13.308 ViewControllerLifeCircle[32611:427962] -[ViewController viewWillDisappear:] 
2017-03-01 19:03:14.683 ViewControllerLifeCircle[32611:427962] -[ViewController viewDidDisappear:] 
2017-03-01 19:03:14.683 ViewControllerLifeCircle[32611:427962] -[ViewController dealloc]
2017-03-01 19:12:05.927 ViewControllerLifeCircle[32611:427962] -[ViewController didReceiveMemoryWarning]
```
