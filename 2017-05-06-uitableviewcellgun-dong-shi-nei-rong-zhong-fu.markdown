---
layout: post
title: "UITableViewCell滚动时内容重复"
date: 2015-11-04 22:53:13 +0800
comments: true
categories: Dev
tags: [IOS]
---

<!--more-->
在使用UITableView表格布局时，出现了表格Cell在滚动之后，内容被覆盖的现象。如图

<a href="http://zwq.bingyan.net/wp-content/uploads/2015/11/屏幕快照-2015-11-01-15.58.31.png"><img class="alignnone size-medium wp-image-118" src="http://zwq.bingyan.net/wp-content/uploads/2015/11/屏幕快照-2015-11-01-15.58.31-163x300.png" alt="屏幕快照 2015-11-01 15.58.31" width="163" height="300" /></a>

代码如下:
```Objective-C
- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath{

if (indexPath.section == 0) {
static NSString *cellIdentify = @"DefaultSitesCell";
UITableViewCell *cell = [tableView dequeueReusableCellWithIdentifier:cellIdentify];
if (cell == nil) {
cell = [[UITableViewCell alloc] initWithStyle:UITableViewCellStyleDefault reuseIdentifier:cellIdentify];
}
NSInteger row = indexPath.row;
NSDictionary *sites = [self.listData objectAtIndex:row];

UILabel *siteName = [[UILabel alloc] init];
siteName.frame = CGRectMake(50, 0, 300, 44);
siteName.text = [sites objectForKey:@"name"];
[cell.contentView addSubview:siteName];

UIImageView *imageV = [[UIImageView alloc] init];
imageV.frame = CGRectMake(10, 7, 30, 30);
imageV.layer.cornerRadius = 15;
[imageV sd_setImageWithURL:[NSURL URLWithString:[sites objectForKey:@"image"]]
placeholderImage:[UIImage imageNamed:@"placeholder_icon.png"]];
[cell.contentView addSubview:imageV];
return cell;
}else {
static NSString *addedCellIdentify = @"AddedSitesCell";
UITableViewCell *cell = [tableView dequeueReusableCellWithIdentifier:addedCellIdentify];
if (cell == nil) {
cell = [[UITableViewCell alloc] initWithStyle:UITableViewCellStyleDefault reuseIdentifier:addedCellIdentify];
}
cell.textLabel.text = @"+ 订阅更多站点";
cell.textLabel.frame = cell.frame;
cell.textLabel.textAlignment = NSTextAlignmentCenter;
return cell;
}
}
```
出现这种情况源于`UITableViewCell`的重用机制.
借用网上的解释: [重用实现分析](http://www.cocoachina.com/bbs/read.php?tid-144611.html)

* 查看`UITableView`头文件，会找到`NSArray* visiableCells`，和`NSMutableArray* reusableTableCells`两个结构。`visiableCells`内保存当前显示的`cells`，`reusableTableCells`保存可重用的`cells`。

* `TableView`显示之初，`reusableTableCells`为空，那么`tableView dequeueReusableCellWithIdentifier:CellIdentifier`返回`nil`。开始的`cell`都是通过`[[UITableViewCell alloc] initWithStyle:UITableViewCellStyleDefault reuseIdentifier:CellIdentifier]`来创建，而且`cellForRowAtIndexPath`只是调用最大显示`cell`数的次数。

比如：有100条数据，iPhone一屏最多显示10个`cell`。程序最开始显示`TableView`的情况是：

* 用`[[UITableViewCell alloc] initWithStyle:UITableViewCellStyleDefault reuseIdentifier:CellIdentifier]`创建10次`cell`，并给`cell`指定同样的重用标识(当然，可以为不同显示类型的cell指定不同的标识)。并且10个`cell`全部都加入到`visiableCells`数组，`reusableTableCells`为空。

* 向下拖动`tableView`，当`cell1`完全移出屏幕，并且`cell11`(它也是`alloc`出来的，原因同上)完全显示出来的时候。`cell11`加入到`visiableCells`，`cell1`移出`visiableCells`，`cell1`加入到`reusableTableCells`。

* 接着向下拖动`tableView`，因为`reusableTableCells`中已经有值，所以，当需要显示新的`cell`，`cellForRowAtIndexPath`再次被调用的时候，`tableView dequeueReusableCellWithIdentifier:CellIdentifier`，返回`cell1`。`cell1`加入到`visiableCells`，`cell1`移出`reusableTableCells`；`cell2`移出`visiableCells`，`cell2`加入到`reusableTableCells`。之后再需要显示的`Cell`就可以正常重用了。

理解了原理，解决这个问题的方法就有很多了。
#### 去掉重用机制
每次都新建一个新的`UITableViewCel`·, 这样可以解决问题，但是对于内存和性能要求高的应用就不适合了。

#### 自定义`UITableViewCell`
这个方法是非常好的，推荐使用。但是我的`Cell`非常简单，自定义的话，感觉还是有点麻烦了。

#### 去掉之前的内容，添加新的内容
因为每次从`reusableTableCells`中取出的`Cell`中`contentView`都不为空，所以继续添加，就会出现内容覆盖的现象。
添加这一行, 删除之前cell中的内容
```
[cell.contentView.subviews makeObjectsPerformSelector:@selector(removeFromSuperview)];
```
或者
```
for (UIView *view in cell.contentView.subviews) {
[view removeFromSuperview];
}
```

#### 使用不同的重用标识符
```
NSString *CellIdentifier = [NSString stringWithFormat:@"%d,%d",indexPath.section,indexPath.row];
```
这种方法也值得争议,因为也没有完全利用重用机制。