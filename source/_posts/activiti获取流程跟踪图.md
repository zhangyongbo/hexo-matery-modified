---
title: activiti获取流程跟踪图
top: false
cover: false
password: ''
toc: true
mathjax: true
summary: 根据流程id获取流程实时跟踪图，已执行节点线路标红。
tags:
  - Activiti
categories:
  - Java
date: 2019-08-31 14:18:53
---

```java
/**
 * 根据流程实例id，获取流程实时跟踪图片Base64码
 * 已添加“data:image/png;base64,”
 *
 * @param processInstanceId 流程实例id
 * @return 图片base64码
 */
@RequestMapping("/generateProcessImageBase64")
public String generateProcessImageBase64(String processInstanceId) {
    //获取历史流程实例
    HistoricProcessInstance historicProcessInstance = historyService.createHistoricProcessInstanceQuery()
            .processInstanceId(processInstanceId).singleResult();
    //获取历史流程定义
    ProcessDefinitionEntity processDefinitionEntity = (ProcessDefinitionEntity) ((RepositoryServiceImpl)
            repositoryService).getDeployedProcessDefinition(historicProcessInstance.getProcessDefinitionId());
    //查询历史节点，需要按照执行顺序排序，因为程序中需要判断流程最后的节点是否为结束节点
    //按照开始时间进行升序
    List<HistoricActivityInstance> historicActivityInstanceList = historyService
            .createHistoricActivityInstanceQuery().processInstanceId(processInstanceId)
            .orderByHistoricActivityInstanceStartTime().asc().list();
    //已执行历史节点
    List<String> executedActivityIdList = new ArrayList();
    for (HistoricActivityInstance historicActivityInstance : historicActivityInstanceList) {
        executedActivityIdList.add(historicActivityInstance.getActivityId());
    }
    BpmnModel bpmnModel = repositoryService.getBpmnModel(processDefinitionEntity.getId());
    //已执行flow的集和
    List<String> executedFlowIdList = executedFlowIdList(bpmnModel, historicActivityInstanceList);
    ProcessDiagramGenerator processDiagramGenerator = processEngine.getProcessEngineConfiguration()
            .getProcessDiagramGenerator();
    InputStream inputStream = processDiagramGenerator.generateDiagram(bpmnModel, "png", executedActivityIdList,
            executedFlowIdList, processEngine.getProcessEngineConfiguration().getActivityFontName(),
            processEngine.getProcessEngineConfiguration().getLabelFontName(), processEngine
                    .getProcessEngineConfiguration().getAnnotationFontName(), processEngine
                    .getProcessEngineConfiguration().getClassLoader(), 1.0);
    String image = Base64Convert.getBase64FromInputStream(inputStream);
    //返回添加png图片的base64标识
    return "data:image/png;base64," + image;

}

/**
 * 已执行flow集合
 *
 * @param bpmnModel                    模型
 * @param historicActivityInstanceList 已执行的节点，需要按照执行顺序排序，因为程序中需要判断流程最后的节点是否为结束节点
 * @return 已执行的flow
 */
private static List<String> executedFlowIdList(BpmnModel bpmnModel, List<HistoricActivityInstance>
        historicActivityInstanceList) {
    List<String> executedFlowIdList = new ArrayList();
    int count = 0;
    if ("endEvent".equals(historicActivityInstanceList.get(historicActivityInstanceList.size() - 1)
            .getActivityType())) {
        count = historicActivityInstanceList.size();
    } else {
        count = historicActivityInstanceList.size() - 1;
    }
    for (int i = 0; i < count; i++) {
        HistoricActivityInstance hai = historicActivityInstanceList.get(i);
        FlowNode flowNode = (FlowNode) bpmnModel.getFlowElement(hai.getActivityId());
        List<SequenceFlow> sequenceFlows = flowNode.getOutgoingFlows();
        if (sequenceFlows.size() > 1) {
            HistoricActivityInstance nextHai = historicActivityInstanceList.get(i + 1);
            for (SequenceFlow sequenceFlow : sequenceFlows) {
                if (sequenceFlow.getTargetRef().equals(nextHai.getActivityId())) {
                    executedFlowIdList.add(sequenceFlow.getId());
                }
            }
        } else {
            if (sequenceFlows.size() != 0)
                executedFlowIdList.add(sequenceFlows.get(0).getId());
        }
    }
    return executedFlowIdList;
}
```