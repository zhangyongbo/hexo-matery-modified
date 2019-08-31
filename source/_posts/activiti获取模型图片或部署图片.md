---
title: activiti获取模型图片或部署图片
top: false
cover: false
password: ''
toc: true
mathjax: true
summary: 获取activiti的模型图片或部署流程图片
tags:
  - Java
  - Activiti
categories:
  - Java
date: 2019-08-31 16:10:14
---

```java
/**
 * 获取流程图片base64码
 * 已添加“data:image/png;base64,”
 *
 * @param deploymentId model的id/部署id
 * @param type         传入参数类型 deploy/model
 * @return 图片base64码
 */
@RequestMapping(value = "/imageBase64")
public String imageBase64(String deploymentId, String type) {
    String image = "";
    if ("deploy".equals(type)) {
        // 从仓库中找需要展示的文件
        List<String> names = repositoryService.getDeploymentResourceNames(deploymentId);
        String imageName = null;
        for (String name : names) {
            if (name.indexOf(".png") >= 0) {
                imageName = name;
                break;
            }
        }
        // 通过部署ID和文件名称得到文件的输入流
        InputStream in = repositoryService.getResourceAsStream(deploymentId, imageName);
        image = Base64Convert.getBase64FromInputStream(in);
    } else if ("model".equals(type)) {
        //模型模块
        byte[] bytes = repositoryService.getModelEditorSourceExtra(deploymentId);
        image = Base64.encodeBase64String(bytes);
    }
    return "data:image/png;base64," + image;
}


```