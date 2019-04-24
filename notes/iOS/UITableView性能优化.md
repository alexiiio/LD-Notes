

UITableView最核心的思想就是UITableViewCelI的重用机制。简单的理解就是: UITableView只会创建一屏幕(或一屏幕多一点)的UITableViewCell,其他都是从中取出来重用的。每当Cell滑出屏幕时，就会放入到一个集合(或数组) 中(这里就相当于一个重用池)，当要显示某一位置的Cel时，会先去集合(或数组)中取，如果有，就直接拿来显示;如果没有，才会创建。这样做的好处可想而知，极大的减少了内存的开销。

UITableView是继承自UIScrolView的，需要先确定它contentSize及每个Cell的位置，然后才会把重用的Cell放置到对应的位置。所以事实上，UITableView的回调顺序是先多次调用`tableView:heightForRowAtIndexPath:`以确定contentSize及Cel的位置，然后才会调用`tableView:cellForRowAtIndexPath:`,从而来显示在当前屏幕的Cell。

优化思路是把赋值和计算布局分离。这样让`tableView:cellForRowAtIndexPath:`方法只负责赋值。

优化方法：

1. 提前计算并缓存cell高度，根据数据源计算出对应的布局，并缓存到数据源中
2. 使用cell重用机制，正确使用reuseldentifier来重用Cells
3. 按需填充数据，可以在UITableView的delegate方法`tableView:willDisplayCell:forRowAtIndexPath:`中进行数据的填充
4. 不要动态创建子视图，所有的子视图都预先创建，并添加到添加到contentView上，如果不需要显示可以设置hidden
5. 尽量不用或少用透明的图层，所有的子视图都指定颜色，且颜色不要用alpha
6. 如果Cell内显示的内容来自web，使用异步加载，缓存请求结果（SDWeblmage已经实现异步加载并缓存）
7. 占位图片使用小尺寸，使用贝塞尔曲线对图片或视图绘制圆角
8. 使用[Texture](https://github.com/texturegroup/texture)。可以保持复杂用户界面的流畅和响应。Texture的node是线程安全的，通过将图像解码、文本绘制和渲染等操作从主线程中迁移，从而保证主线程的响应。