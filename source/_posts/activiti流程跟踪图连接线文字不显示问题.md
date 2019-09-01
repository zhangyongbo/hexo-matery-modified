---
title: activiti流程跟踪图连接线文字不显示问题
top: false
cover: false
password: ''
toc: true
mathjax: true
summary: 解决activiti生成的流程跟踪图，连接线上面没有描述文字问题。
tags:
  - Activiti
categories:
  - Java
date: 2019-08-31 14:17:27
---

需要反编译流程图片处理类，新建同包同名类，复制反编译内容，添加连接线文字处理。
`activiti-image-generator-5.22.0.jar!/org/activiti/image/impl/DefaultProcessDiagramGenerator.class`
587行左右`if(labelGraphicInfo != null) {`添加else处理。
```java
if (labelGraphicInfo != null) {
    processDiagramCanvas.drawLabel(sequenceFlow.getName(), labelGraphicInfo, false);
} else {
    //反编译class得到，主要处理线文字不显示
    GraphicInfo lineCenter = getLineCenter(graphicInfoList);
    processDiagramCanvas.drawLabel(sequenceFlow.getName(), lineCenter, false);
}
```