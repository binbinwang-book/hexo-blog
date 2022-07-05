---
title: rebase --onto 实现commit批量复制
date: 2021-11-29 12:45
tags: [git]
categories: [技术方案]
---

# rebase --onto 实现commit批量复制

今晚在解答正在读大学表弟的学习问题时，他问到了一个问题：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gwvbw0dqnzj311k0aemyj.jpg)

这个截图里说到的`git rebase xxx^ yyy --onto master`是什么意思，重不重要呀，平时有什么样的应用场景。

这是一个好问题，这种指令也可以提升我自己平时的开发效率，所以记录成本文。

## 应用背景

在我们的实际开发的过程中，我们的项目中会存在多个分支。

在某些情况下，可能需要将某一个分支上的 commit 复制到另一个分支上去。

![](https://tva1.sinaimg.cn/large/008i3skNgy1gwvbyalcu0j30xq0cq0tm.jpg)

比如在实际开发过程中，我经常会基于 `trunk_ultron_test` 拉一个分支 `trunk_ultron_test_dev`，然后开发完之后会把代码合入到`trunk_ultron_test`体验完之后，还需要把需求的commit合并到`trunk_ultron_test`的祖先分支`trunk_dev`中。

一般的做法是找出需求的节点，比如：

```
commitId_0 [yyy feature] 做了0事情
commitId_A [xxx feature] 做了A事情
commitId_B [xxx feature] 做了B事情
commitId_C [xxx feature] 做了C事情
commitId_D [xxx feature] 做了D事情
commitId_E [xxx feature] 做了E事情
```

常规的做法就是基于`trunk`拉一条分支`trunk_dev`，然后把这些commit节点`cherry pick commitId_A , commitId_B , commitId_C , commitId_D , commitId_E `到`trunk_dev`分支上，处理完冲突后再进行push。

这种场景下，commit节点少还好，节点一多，`cherry-pick`的效率就非常低了，鉴于`commitId_A`和`commitId_E`都是连续的节点，有没有一个方法可以让我指定`commitId_A`作为开启点，`commitId_E`作为结束点，就能把中间的节点都一起合并呢？

有的，这就是本文要介绍 rebase --onto 

## rebase --onto

我基于 `main`分支拉了一个`main_dev`的分支，并在 `main_dev`分支保存了4个commit（A、B、C、D）：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gwvc6yw7kij313b0u075w.jpg)

接着我想把（A、B、C、D）合并到`main`分支中，那么我就可以使用 `rebase --onto`命令，官方教程是：

```
// startpoint 第一个 commit id, endpoint 最后一个 commit id，branchName 就是目标分支了。
$ git rebase [startpoint] [endpoint] --onto [branchName]

执行 git rebase 命令之后，我们发现当前的 HEAD 处于游离状态。
所以我们需要使用 git reset 命令，将 master 所指向的 commit id 设置为当前 HEAD 所指向的 commit id。 
```

操作截图为：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gwvc8ljscij315q0t6jxf.jpg)

可以发现，经过 `rebase --onto`之后，`main`分支已经有了完整的 A、B、C、D 的log日志：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gwvc9yh6e8j30qg0cot9r.jpg)

------
**这个公众号会持续更新技术方案、关注业内技术动向，关注一下成本不高，错过干货损失不小。
↓↓↓**
![](https://tva1.sinaimg.cn/large/e6c9d24egy1gzzmv1p67mj21bi0hcwgh.jpg)