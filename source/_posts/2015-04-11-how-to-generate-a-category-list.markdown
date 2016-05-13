---
layout: post
title: "如何生成分级的类别列表？"
date: 2015-04-11 21:21
comments: true
categories: ['web', 'javascript', '递归']
---

## 分类问题
最近在做一个网页时，有这样一个需求，事实上很多电商之类的网站都有这样的需求：分类。然而这种分类又可以有无穷多子类，如：

```
书
  |`- 自然科学
  |      |`- 物理
  |      |      |`- 经典物理
  |      |       `- 量子物理
  |       `- 数学
   `- 小说
         |`- 都市爱情
          `- 乡村爱情
唱片
  |`- 爵士
   `- 摇滚
```
<!--more-->
这样的数据，在数据库中一般有以下设计：

- **ID**：分类的唯一标识；
- **PID**：父类的 ID；
- **Name**：分类名。

于是，在上例中，数据库中保存的数据可能是这样的：

ID|PID|Name|
---|---|---
1|0|书       
2|0|唱片
3|1|自然科学
4|1|小说
5|2|爵士
6|2|摇滚
7|3|物理
8|7|经典物理
9|7|量子物理
10|3|数学
...|...|...

现在需要做一个添加项目的表单，其中一个需要填的数据就是该项目的分类，如何生成一个下拉选择，使得分类条目按层级关系列表出来？如下所示：

<select name=cate>
  <option value=1>书</option>
  <option value=3>&nbsp;&nbsp;&nbsp;&nbsp;自然科学</option>
  <option value=7>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;物理</option>
  <option value=8>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;经典物理</option>
  <option value=9>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;量子物理</option>
  <option value=10>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;数学</option>
  <option value=?>...</option>
</select>

很直观的一个思路是先从数据库查出所有的分类保存到一个数组中，然后从 `PID = 0` 的开始处理，可能用到多重循环。

## 树的遍历
事实上，如何分类的层级关系可以写成一个棵树的话，如下图，那么生成列表这个问题就是一个**前序遍历**问题了。

![分类树](/images/post/category-tree.png '分类树')

假设存在一个虚拟的 `Root` 节点，从 `Root` 开始，前序遍历，则遍历顺序依次是：

1. Root
2. 书
3. 自然科学
4. 物理
5. 量子物理
6. 经典物理
7. 数学
8. 小说
9. 都市爱情
10. 乡村爱情
11. 唱片
12. 摇滚
13. 爵士

然后根据节点深度对结果作一下缩进，目的就达到了。我们可以发现，同级中的顺序并不那么重要，先访问*数学*和先访问*物理*对下拉选择并没有影响，只要不同分类之间的层级关系正确就好。

## 递归的思路
对于从数据库中取出来的分类数组，要正确构建成一棵树还是有点工作量的，应该可以有更好的方法。于是有了下面的递归的思路：

```javascript
var categories = [{id: 'xxx', pid: 'xxx', name: 'xxx'}, {}, {}];

function insert(pid, level) {
	var arr = [];
	categories.forEach(function(o) {
		if (o.pid == pid) {
			o.level = level;
			arr.push(o);
			arr = arr.concat(insert(o.id, level + 1));
		}
	});
	return arr;
}

var list = insert(0, 0);
...

```

上面代码中，`category` 是从后台请求的未经处理的所有分类数据，函数 `insert` 需要两个参数：`pid` 是父类的 **ID**，`level` 是分类所处的深度。`insert` 返回一个数组，该数组的元素是以指定 `pid` 为祖先的元素。

该算法的效率为 `N * d`，`N` 为分类个数，`d` 为层级深度
