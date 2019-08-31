---
title: activiti ie下保存模型问题
top: false
cover: false
password: ''
toc: true
mathjax: true
summary: activiti在IE下保存模型失败
tags:
  - Java
  - Activiti
  - 转
categories:
  - Java
date: 2019-08-31 16:15:13
---

**错误：Enclosed Exception: 元素类型 "path" 必须后跟属性规范 ">" 或 "/>"**

modeler作图使用svg（可缩放矢量图形），在ie中如果使用了箭头（即svg中的path元素），则model保存不成功，提示错误`Unexpected error: could not save model`，IDE console中提示：`元素类型 "path" 必须后跟属性规范 ">" 或 "/>"`。在ie  F12开启调试模式，与chrome对比发现是因为modeler.html中的path的`marker-start`和`marker-end`属性值多了引号，导致在保存时path出错。

IE：

```
<path xmlns="http://www.w3.org/2000/svg" id="sid-7278F570-B74E-4EE6-A21A-52A3B3BF3C26_1" fill="none" marker-end="url(&quot;#sid-7278F570-B74E-4EE6-A21A-52A3B3BF3C26end&quot;)" marker-start="url(&quot;#sid-7278F570-B74E-4EE6-A21A-52A3B3BF3C26start&quot;)" stroke="#585858" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M 355.391 135 L 399.375 135"/>
```

chrome：

```
<path id="sid-CB6F9DD4-D150-4072-8EF6-75B5AE54E9BF_1" d="M210.609375 135L254.15625 135 " stroke="#585858" fill="none" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" marker-start="url(#sid-CB6F9DD4-D150-4072-8EF6-75B5AE54E9BFstart)" marker-end="url(#sid-CB6F9DD4-D150-4072-8EF6-75B5AE54E9BFend)"></path>
```



对比可知，chrome中的`setAttributeNS`函数不会对url()做特殊处理，而IE中会对url()中的字符用引号包裹！在此设置断点重新画一个path后，发现引号删不掉（nodevalue无法修改为无引号），把值设置为url(bbbb)会自动变为url("bbbb")！



# 解决方法

在`toolbar-default-actions.js`中的`$scope.save = function (successCallback){}`函数中增加以下红色代码：
```javascript

var svgDOM = DataManager.serialize(svgClone);
//解决ie保存报错
svgDOM = svgDOM.replace(/marker-start="url\("#/g,"marker-start=\"url(#").replace(/start"\)"/g,"start\)\"");
svgDOM = svgDOM.replace(/marker-mid="url\("#/g,"marker-mid=\"url(#").replace(/mid"\)"/g,"mid\)\"");
svgDOM = svgDOM.replace(/marker-end="url\("#/g,"marker-end=\"url(#").replace(/end"\)"/g,"end\)\"");
//解决ie保存报错
```
接下来清除缓存即可