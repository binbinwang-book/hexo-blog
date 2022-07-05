---
title: tableView评论半屏重构 - 记录一次被自己叫停的重构流程
date: 2021-07-26 02:34
tags: [重构,tableView]
categories: [iOS]
---

# tableView评论半屏重构 - 记录一次被自己叫停的重构流程

最近在和同事做一个关于和`tableView`相关的曝光上报逻辑，无意中发现前人写的代码对我们做曝光上报的逻辑非常不方便，所以打算进行一波重构，但这波重构工作最终还是被我自己叫停了。

## 一、重构前存在的问题

![](https://tva1.sinaimg.cn/large/008i3skNgy1gstq4rf0yjj30n80istac.jpg)

既有逻辑如上，前人是通过控制 tableView 的 contentInset 来实现和 headerView 保持间距。

这种架构非常不方便做曝光上报，常规意义下的曝光上报是走 `willDisplayCell ` 进行上报。

```
- (void)tableView:(UITableView *)tableView willDisplayCell:(UITableViewCell *)cell forRowAtIndexPath:(NSIndexPath *)indexPath
```

因为 `headerView` 是被直接加到 self.view 上，而不是 `tableView`的一个 cell ，所以无法通过 `willDisplayCell`获取 `headerView` 的曝光回调，经过和同事聊完后决定开始对`headerView`来一波重构，将`headerView`加到`tableView`中。

## 二、将headerView重构为Cell

将 `headerView` 重构为 `UITableViewCell`，然后就可以通过 `willDisplayCell` 来获取cell的曝光事件，这个是目标。

涉及到UI重构的过程基本就是 遇山开山，遇水架桥，比较顺利，顺便记录下几个小知识点。

### 1. 想实现禁止 Cell 的 didSelect 回调，但要 cell 里的 view 仍旧能够响应手势，应该怎么做？

第一步：cell.selectionStyle =UITableViewCellSelectionStyleNone;

第二步：将所有元素都加到 cell.contentView 上，而非 cell 上

### 2. 自定义cell高度变更频繁，如何处理

将自定义cell update计算高度时的宏，和提供给tableView预设cell高度的宏保持一致。

### 3. tableView 的分割线可以用什么形式实现？

可以给 cell 添加元素，也可以使用 viewForFooterInSection 实现。

### 4. tableView reload Section 时不要闪一下

```
[self.tableView reloadSections:[NSIndexSet indexSetWithIndex:0] withRowAnimation:UITableViewRowAnimationNone];
```

使用上面这种方法reload section会闪一下，使用下面的方法reload时不会闪：

```
        [UIView setAnimationsEnabled:NO];
        [self.tableView beginUpdates];
        [self.tableView reloadSections:[NSIndexSet indexSetWithIndex:0] withRowAnimation:UITableViewRowAnimationNone];
        [self.tableView endUpdates];
        [UIView setAnimationsEnabled:YES];
```

#### （1）beginUpdates 和 endUpdates

1、beginUpdates 和 endUpdates必须成对使用

2、使用beginUpdates和endUpdates可以在改变一些行（row）的高度时自带动画，并且不需要Reload row（不用调用cellForRow，仅仅需要调用heightForRow，这样效率最高）。

3、在beginUpdates和endUpdates中执行insert,delete,select,reload row时，动画效果更加同步和顺滑，否则动画卡顿且table的属性（如row count）可能会失效。

4、在beginUpdates 和 endUpdates中执行 reloadData 方法和直接reloadData一样，没有相应的中间动画。

#### （2）[UIView setAnimationsEnabled:NO];

可以让动画失效：UIView动画失效

## 三、重构遇阻

在 UI 重构基本完成后，我发现只有一条评论时，评论总是展示不出来，debug发现是因为我现在默认给tableView新增了一个headerViewCell，那么之前通过section去ViewModel取数据源的方式也就应该（-1）了。

比如之前获取评论是通过 section 获取评论，section为0就对应第一条评论，依次递增。

现在 section == 0 是 headerViewCell，所以重构后获取评论前应该先判断有没有 headerViewCell， 如果有那么就要将 section - 1，再去 viewModel 中获取数据源。

其实重构到这里我已经大致发现重构方式有问题了，评论半屏就应该是section和viewModel一一对应的。

然后当我在尝试对所有通过 section 从 ViewModel 获取评论的方法进行修改时，我发现需要修改几十个方法，且处理后会把代码搞得不如以前那么好理解。

复杂说明做错了，手动去做（section-1）的操作在设计上非常不优雅， headerView 不应该加到 tableView 里，至少它不应该是以 UITableViewCell 的形式加入。

接着我进一步思考，headerVIew 能不能被当做 tableView.headerView ？

但查看代码发现，tableView.headerView 已经有对应的UI视图元素在使用了。

那么 headerView 能不能当做 section.headerView ?

做肯定是能做的，但这个设计会有两个问题：

- 当没有评论的时候，ViewModel中评论为空，这时还必须给 section == 0 设置一个 headerView，也就是说即使 ViewModel 为空，也要预留一个 cell
- headerView 和第一个 section 绑定后，如果后续有需求需要给每个 section 都添加一个 headerView，那 section == 0 的 headerView 又出现互斥的情况

经过以上思考后，我决定放弃重构的计划，还是按之前的UI设计，尝试在既有设计的基础上，优雅的解决 view 曝光的问题。

## 四、view 的曝光设计

曝光上报，我们的初衷是想使用苹果提供的`willDisplayCell`，只要 headerView 能抛给 ViewController 一个曝光的回调，那么问题就解决了，只是这个曝光回调的逻辑需要我们实现而已，这比重构的工作量和风险都小得多。

具体方案是：

headerView 中维护一个 isShow 的标志位，默认值为 NO

headerView 新增一个 delegate:

```
- (void)headerViewShowStateChanged:(BOOL)isShow
```

当 isShow 由 NO 转为 YES，又或者由 YES 转为 NO时，会进行方法的回调。

## 五、经验教训

前几次的重构工作过于顺利，导致我这次开展重构工作前，没有先进行重构方案可行性的调查。

所以开展重构前，除了要准备好 「完备的测试用例」 ，还需要评估「方案变更大不大」，重构后的方案是不是更优雅了。

如果只是为了解决一个问题而进行重构，导致方案变得更加ugly，那么就不应该进行重构。

------
**这个公众号会持续更新技术方案、关注业内技术动向，关注一下成本不高，错过干货损失不小。
↓↓↓**
![](https://tva1.sinaimg.cn/large/e6c9d24egy1gzzmv1p67mj21bi0hcwgh.jpg)