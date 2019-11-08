> 在开发中 UITableView  是最常见的布局控件，除了我们熟知的一些使用方式，也有另一些相对常见但不常用的使用方式值得我们去了解，本篇文章只针对其中一种常见的使用方式作简单介绍。

废话不多说，先上效果图：

![效果图.gif](https://tva1.sinaimg.cn/large/006y8mN6gy1g8qtu8zdq3g30aa0i2wi6.gif)

以上效果样式可以说在我们手机中的 app 上非常常见，其实要做这种效果也非常简单，只需要会使用 UITableView 即可。

# 一、分析

在效果图中，我们可以看到当我每点击一个 cell，会产生两个结果：

1. cell 的标题变成红色
2. 点击的 cell 会展开显示详细信息

因此，我们在实现业务逻辑时就要考虑到这两个方面的具体实现。

# 二、具体实现

- 首先利用 UITableView 搭建主界面，这里就不赘述了，相信大家应该都会

- 其次对于每一个 cell 的内容，我们可以采用自定义 cell 的方式

  -  对于每一个标题和详细信息，我们都可以看成是同一个 cell，只有我们点击的时候才显示详细信息的内容，不点击时则不显示

  ![自定义的 cell](https://tva1.sinaimg.cn/large/006y8mN6gy1g8qtu9uudxj309t052t8s.jpg)

  - 这里我们可以采用控制 cell 高度的方式来实现详细信息是否显示的问题，当不显示时设置 cell 的高度为正常的 44，超出部分不显示；当需要显示详细信息的时候设置 cell 的高度为真实高度

- 代码实现

  ```objective-c
  /** 选中 cell 时调用 */
  - (void)tableView:(UITableView *)tableView didSelectRowAtIndexPath:(NSIndexPath *)indexPath {
      
      self.selCell = [tableView cellForRowAtIndexPath:indexPath];
      
      /** 业务逻辑 */
      self.isOpen = !self.isOpen;
      /** 标题初始文字颜色 */
      self.selCell.titleL.textColor = [UIColor blackColor];
      
      if (self.curRow != indexPath.row) {
          self.isOpen = YES;
          /** cell 展开时标题设置为红色 */
          self.selCell.titleL.textColor = [UIColor redColor];
      }
      self.curRow = indexPath.row;
      
      /** 刷新tableView，系统默认会有一个自带的动画 */
      [tableView beginUpdates];
      [tableView endUpdates];
  }

  /** 取消选中 cell 时调用 */
  - (void)tableView:(UITableView *)tableView didDeselectRowAtIndexPath:(NSIndexPath *)indexPath {
      
      /** 取消选中时，cell 处于关闭状态，标题改为初始黑色 */
      self.selCell.titleL.textColor = [UIColor blackColor];
  }

  /** 设置 cell 的高度 */
  - (CGFloat)tableView:(UITableView *)tableView heightForRowAtIndexPath:(NSIndexPath *)indexPath {
      
      /** 判断记录的行号是否为当前点击行 且 是否处于展开状态 */
      return self.curRow == indexPath.row && self.isOpen ? 150 : 44;
  }
  ```

经过以上步骤已经基本实现了我们需要的效果，如果有疑问可以参考效果源码进行查看。

[源码地址](https://github.com/mortal-master/BWTableView_SimpleUse)

