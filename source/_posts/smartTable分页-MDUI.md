---
title: smartTable分页-MDUI
date: 2018-07-28 20:58:50
tags: [smartTable, MDUI, 分页, 技术]
---

<!--more-->

### 1. 自实现页脚分页 HTML

```html
<div class="mdui-toolbar">
    <label style="margin-right: 5px">每页条目</label>
    <select class="mdui-select" style="margin-left: 0" ng-model="stItemsByPage"
            ng-options="
            item as item
            for
            item in [5,10,15,20]"></select>

    <div class="mdui-toolbar-spacer"></div>


    <button ng-click="selectPage(1)" class="mdui-btn mdui-btn-icon mdui-ripple"
            ng-disabled="disableLast"><i
            class="mdui-icon material-icons">first_page</i></button>
    <button ng-click="selectPage(currentPage - 1)" class="mdui-btn mdui-btn-icon mdui-ripple"
            ng-disabled="disableLast"><i
            class="mdui-icon material-icons">chevron_left</i></button>
    <a style="margin-right: 18px">{{currentPage}} of {{numPages}}</a>
    <button ng-click="selectPage(currentPage + 1)" class="mdui-btn mdui-btn-icon mdui-ripple"
            ng-disabled="disableNext"><i
            class="mdui-icon material-icons">chevron_right</i></button>
    <button ng-click="selectPage(numPages)" class="mdui-btn mdui-btn-icon mdui-ripple"
            ng-disabled="disableNext"><i
            class="mdui-icon material-icons">last_page</i></button>
</div>
```
