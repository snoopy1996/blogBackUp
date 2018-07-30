---
title: $modal
date: 2018-07-28 20:56:45
tags: [AngularJs模态框, $motal, 技术]
---

# $motal

$motal 是一个可以迅速创建模态窗口的服务，创建部分页，控制器，并关联他们

- $motal 只有一个方法**open(options)**
  <!--more-->

---

## open 方法

- templateUrl：模态窗口的地址
- template：用于显示 html 标签
- scope：一个作用域为模态的内容使用（事实上，\$modal 会创建一个当前作用域的子作用域）默认为$rootScope
- controller：为 modal 指定的控制器，初始化\$scope，该控制器可用\$modalInstance 注入
- resolve：定义一个成员并将他传递给\$modal 指定的控制器，相当于 routes 的一个 reslove 属性，如果需要传递一个 objec 对象，需要使用 angular.copy()
- backdrop：控制背景，允许的值：true（默认），false（无背景），“static” - 背景是存在的，但点击模态窗口之外时，模态窗口不关闭
- keyboard：当按下 Esc 时，模态对话框是否关闭，默认为 ture
- windowClass：指定一个 class 并被添加到模态窗口中
- open 方法返回一个模态实例，该实例有如下属性
- close(result)：关闭模态窗口并传递一个结果
- dismiss(reason)：撤销模态方法并传递一个原因
- result：一个契约，当模态窗口被关闭或撤销时传递
- opened：一个契约，当模态窗口打开并且加载完内容时传递的变量
- \$modalInstance 扩展了两个方法\$close(result)、\$dismiss(reason)，这些方法很容易关闭窗口并且不需要额外的控制器
- closePromise 返回一个回调函数，用于模态框关闭之后要做的事情.
