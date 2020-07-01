---
title: Activiti（5）流程定义文件—流程定义—流程模型 的相互转化
date: 2020-06-01 16:38:33
tags: Activiti
categories: Activiti
description: Activiti（5）流程定义文件—流程定义—流程模型 的相互转化
typora-root-url: ..
password: kiki
---

> 前面我们介绍了流程模型的创建、以及用流程模型部署流程定义。本节我们主要介绍，流程定义文件—流程定义—流程模型 的相互转化
>

# 1、回顾

## 1.1 、生成流程模型

```java
/**
     * 创建模型
     */
    @Test
    public void create() {
        String name = "请假流程";
        String key = "leave";
        String description = "这是一个简单的请假流程";

        try {
            ObjectMapper objectMapper = new ObjectMapper();
            ObjectNode editorNode = objectMapper.createObjectNode();
            editorNode.put("id", "canvas");
            editorNode.put("resourceId", "canvas");

            ObjectNode stencilSetNode = objectMapper.createObjectNode();
            stencilSetNode.put("namespace", "http://b3mn.org/stencilset/bpmn2.0#");

            editorNode.put("stencilset", stencilSetNode);

            ObjectNode modelObjectNode = objectMapper.createObjectNode();
            modelObjectNode.put(MODEL_NAME, name);
            modelObjectNode.put(ModelDataJsonConstants.MODEL_REVISION, 1);
            description = StringUtils.defaultString(description);
            modelObjectNode.put(MODEL_DESCRIPTION, description);

            Model newModel = repositoryService.newModel();
            newModel.setMetaInfo(modelObjectNode.toString());
            newModel.setName(name);
            newModel.setKey(StringUtils.defaultString(key));

            repositoryService.saveModel(newModel);
            repositoryService.addModelEditorSource(newModel.getId(), editorNode.toString().getBytes("utf-8"));

            System.out.println("生成的moduleId:"+newModel.getId());

        } catch (Exception e) {

        }
    }
```

## 1.2 、部署流程定义

```java
/**
     * 部署流程定义
     */
    @Test
    public void deployTest(){
        Deployment deployment = repositoryService.createDeployment()
                .addClasspathResource("leave.bpmn")
                .deploy();
        assertNotNull(deployment);
    }
```

## 1.3 、 引出问题

**第一个问题：**步骤 1.2 中，部署流程定义是直接使用 `leave.bpmn` 文件部署的。但是我们使用Activiti Modeler设计流程图后，流程模型数据是直接保存在数据库表中的（act_re_model）。并不是直接生成一个`.bpmn` 文件。这时如何部署流程定义呢？

- 方式一：使用Activiti Modeler 设计器保存在act_re_model表中的模型记录，导出一个`.bpmn` 文件后再部署（不方便）

- 方式二：直接使用 act_re_model 表中的模型记录来部署流程定义。（正确用法）

**第二个问题：**就是如果我们现在只有一个别人提供的`.bpmn` 文件，如何才能将它保存到`act_re_model`表中呢？

思路如下：

![image-20200602185522746](/images/activiti/activiti6-05/image-20200602185522746.png)

# 2、流程定义文件—流程定义—流程模型 相互转化

## 2.1 、 bpmn文件转流程定义（act_re_procdef）

> 上面的1.2 中，我们是直接将文件放在了`src/main/resources`目录下。下面我们改造一下。改为上传附件部署流程定义。

这里改动较大、把常用工具类类单独放在了workflow-common这个module中。贴出关键代码：

新建 ProcessDefinitionController.java

```java
package com.sxdx.workflow.activiti.rest.controller;


import com.sxdx.common.config.GlobalConfig;
import com.sxdx.common.constant.CodeEnum;
import com.sxdx.common.constant.Constants;
import com.sxdx.common.util.CommonResponse;
import com.sxdx.common.util.StringUtils;
import com.sxdx.common.util.file.FileUploadUtils;
import com.sxdx.workflow.activiti.rest.service.ProcessDefinitionService;
import org.activiti.engine.RepositoryService;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.multipart.MultipartFile;


@Controller
@RequestMapping("/definition")
public class ProcessDefinitionController {

    private static final Logger log = LoggerFactory.getLogger(ProcessDefinitionController.class);

    private String prefix = "definition";

    @Autowired
    private ProcessDefinitionService processDefinitionService;
    @Autowired
    private RepositoryService repositoryService;


    /**
     * 部署流程定义
     */
    @PostMapping("/upload")
    @ResponseBody
    public CommonResponse upload(@RequestParam("processDefinition") MultipartFile file) {
        try {
            if (!file.isEmpty()) {
                String extensionName = file.getOriginalFilename().substring(file.getOriginalFilename().lastIndexOf('.') + 1);
                if (!"bpmn".equalsIgnoreCase(extensionName)
                        && !"zip".equalsIgnoreCase(extensionName)
                        && !"bar".equalsIgnoreCase(extensionName)) {
                    return new CommonResponse().code(CodeEnum.FAILURE.getCode()).message("流程定义文件仅支持 bpmn, zip 和 bar 格式！");
                }
                // p.s. 此时 FileUploadUtils.upload() 返回字符串 fileName 前缀为 Constants.RESOURCE_PREFIX，需剔除
                // 详见: FileUploadUtils.getPathFileName(...)
                String fileName = FileUploadUtils.upload(GlobalConfig.getProfile() + "/processDefiniton", file);
                if (StringUtils.isNotBlank(fileName)) {
                    String realFilePath = GlobalConfig.getProfile() + fileName.substring(Constants.RESOURCE_PREFIX.length());
                    processDefinitionService.deployProcessDefinition(realFilePath);
                    return new CommonResponse().code(CodeEnum.SUCCESS.getCode()).message("部署成功");
                }
            }
            return new CommonResponse().code(CodeEnum.FAILURE.getCode()).message("不允许上传空文件！");
        }
        catch (Exception e) {
            log.error("上传流程定义文件失败！", e);
            return new CommonResponse().code(CodeEnum.FAILURE.getCode().toString()).message(e.getMessage());
        }
    }

}

```

我们使用postman做测试：

支持单个bpmn文件上传，也支持多个文件压缩成zip包、或者bar包

![image-20200602183752513](/images/activiti/activiti6-05/image-20200602183752513.png)

然后我们查看`act_re_deployment`和`act_re_procdef`表，可以看到已经生成了对应的流程定义记录（我执行了2次，所以2条记录）

![image-20200602184117885](/images/activiti/activiti6-05/image-20200602184117885.png)

![image-20200602184139312](/images/activiti/activiti6-05/image-20200602184139312.png)

然后我们就可以根据流程定义来发起流程了，本节我们主要介绍内容不是这个，我们先来介绍流程定义如何转化流程模型。

## 2.2、流程定义（act_re_procdef）转流程模型（act_re_model）

执行代码如下：

```ajva
/**
     * 转换流程定义为模型
     * @param processDefinitionId 流程定义id
     * @return
     * @throws UnsupportedEncodingException
     * @throws XMLStreamException
     */
    @PostMapping(value = "/convertToModel")
    @ResponseBody
    public CommonResponse convertToModel(@Param("processDefinitionId") String processDefinitionId)
            throws UnsupportedEncodingException, XMLStreamException {
        org.activiti.engine.repository.ProcessDefinition processDefinition = repositoryService.createProcessDefinitionQuery()
                .processDefinitionId(processDefinitionId).singleResult();
        InputStream bpmnStream = repositoryService.getResourceAsStream(processDefinition.getDeploymentId(),
                processDefinition.getResourceName());
        XMLInputFactory xif = XMLInputFactory.newInstance();
        InputStreamReader in = new InputStreamReader(bpmnStream, "UTF-8");
        XMLStreamReader xtr = xif.createXMLStreamReader(in);
        BpmnModel bpmnModel = new BpmnXMLConverter().convertToBpmnModel(xtr);

        BpmnJsonConverter converter = new BpmnJsonConverter();
        ObjectNode modelNode = converter.convertToJson(bpmnModel);
        Model modelData = repositoryService.newModel();
        modelData.setKey(processDefinition.getKey());
        modelData.setName(processDefinition.getResourceName());
        modelData.setCategory(processDefinition.getDeploymentId());

        ObjectNode modelObjectNode = new ObjectMapper().createObjectNode();
        modelObjectNode.put(ModelDataJsonConstants.MODEL_NAME, processDefinition.getName());
        modelObjectNode.put(ModelDataJsonConstants.MODEL_REVISION, 1);
        modelObjectNode.put(ModelDataJsonConstants.MODEL_DESCRIPTION, processDefinition.getDescription());
        modelData.setMetaInfo(modelObjectNode.toString());

        repositoryService.saveModel(modelData);

        repositoryService.addModelEditorSource(modelData.getId(), modelNode.toString().getBytes("utf-8"));

        return new CommonResponse().code(CodeEnum.SUCCESS.getCode()).message("转换成功");
    }

```

postman测试结果（参数是流程定义id，即`act_re_procdef`表的主键id）

![image-20200602190226097](/images/activiti/activiti6-05/image-20200602190226097.png)

转化成功后，我们查看`act_re_model`表，可以看到已经生成了对应的流程模型。

![image-20200602190443001](/images/activiti/activiti6-05/image-20200602190443001.png)

## 2.3、流程模型（act_re_model）转流程定义（act_re_procdef）

新建ModelerController.java：

```java
    /**
     * 根据Model部署流程，参数为act_re_model表的ID_字段
     */
    @RequestMapping(value = "/modeler/deploy/{modelId}")
    @ResponseBody
    public AjaxResult deploy(@PathVariable("modelId") String modelId, RedirectAttributes redirectAttributes) {
        try {
            Model modelData = repositoryService.getModel(modelId);
            ObjectNode modelNode = (ObjectNode) new ObjectMapper().readTree(repositoryService.getModelEditorSource(modelData.getId()));
            byte[] bpmnBytes = null;

            BpmnModel model = new BpmnJsonConverter().convertToBpmnModel(modelNode);
            bpmnBytes = new BpmnXMLConverter().convertToXML(model);

            String processName = modelData.getName() + ".bpmn20.xml";
            Deployment deployment = repositoryService.createDeployment().name(modelData.getName()).addString(processName, new String(bpmnBytes, "UTF-8")).deploy();
            LOGGER.info("部署成功，部署ID=" + deployment.getId());
            return success("部署成功");
        } catch (Exception e) {
            LOGGER.error("根据模型部署流程失败：modelId={}", modelId, e);

        }
        return error("部署失败");
    }

    /**
     * 导出model的xml文件
     */
    @RequestMapping(value = "/modeler/export/{modelId}")
    public void export(@PathVariable("modelId") String modelId, HttpServletResponse response) {
        try {
            Model modelData = repositoryService.getModel(modelId);
            BpmnJsonConverter jsonConverter = new BpmnJsonConverter();
            JsonNode editorNode = new ObjectMapper().readTree(repositoryService.getModelEditorSource(modelData.getId()));
            BpmnModel bpmnModel = jsonConverter.convertToBpmnModel(editorNode);

            // 流程非空判断
            if (!CollectionUtils.isEmpty(bpmnModel.getProcesses())) {
                BpmnXMLConverter xmlConverter = new BpmnXMLConverter();
                byte[] bpmnBytes = xmlConverter.convertToXML(bpmnModel);

                ByteArrayInputStream in = new ByteArrayInputStream(bpmnBytes);
                String filename = bpmnModel.getMainProcess().getId() + ".bpmn";
                response.setHeader("Content-Disposition", "attachment; filename=" + filename);
                IOUtils.copy(in, response.getOutputStream());
                response.flushBuffer();
            } else {
                try {
                    response.sendRedirect("/modeler/modelList");
                } catch (IOException ex) {
                    ex.printStackTrace();
                }
            }

        } catch (Exception e) {
            LOGGER.error("导出model的xml文件失败：modelId={}", modelId, e);
            try {
                response.sendRedirect("/modeler/modelList");
            } catch (IOException ex) {
                ex.printStackTrace();
            }
        }
    }
```

![image-20200602192147302](/images/activiti/activiti6-05/image-20200602192147302.png)

![image-20200602192240137](/images/activiti/activiti6-05/image-20200602192240137.png)



我们查看对应的数据库表：

![image-20200602192229729](/images/activiti/activiti6-05/image-20200602192229729.png)

![image-20200602192416109](/images/activiti/activiti6-05/image-20200602192416109.png)

可以看到生成了对应的流程定义，因为我们使用的是同一个KEY的流程，所以生成的记录的VERSION_字段（版本）自动递增变为了3。有一点需要说明。当我们直接使用流程定义KEY（这里是leaveCounterSign）发起流程时，系统会默认选择吧版本更高的流程定义。

## 2.4、将流程模型导出为bpmn文件

```java
 /**
     * 将流程模型导出为 bpmn文件
     */
    @RequestMapping(value = "/modeler/export/{modelId}")
    public void export(@PathVariable("modelId") String modelId, HttpServletResponse response) {
        try {
            Model modelData = repositoryService.getModel(modelId);
            BpmnJsonConverter jsonConverter = new BpmnJsonConverter();
            JsonNode editorNode = new ObjectMapper().readTree(repositoryService.getModelEditorSource(modelData.getId()));
            BpmnModel bpmnModel = jsonConverter.convertToBpmnModel(editorNode);

            // 流程非空判断
            if (!CollectionUtils.isEmpty(bpmnModel.getProcesses())) {
                BpmnXMLConverter xmlConverter = new BpmnXMLConverter();
                byte[] bpmnBytes = xmlConverter.convertToXML(bpmnModel);

                ByteArrayInputStream in = new ByteArrayInputStream(bpmnBytes);
                String filename = bpmnModel.getMainProcess().getId() + ".bpmn";
                response.setHeader("Content-Disposition", "attachment; filename=" + filename);
                IOUtils.copy(in, response.getOutputStream());
                response.flushBuffer();
            } else {
                logger.error("导出model的xml文件失败：modelId={}", modelId);
            }
        } catch (Exception e) {
            logger.error("导出model的xml文件失败：modelId={}", modelId, e);
        }
    }
```

测试结果：

![image-20200602194029052](/images/activiti/activiti6-05/image-20200602194029052.png)

如果直接使用浏览器访问，则会下载bpmn文件

![image-20200602194230562](/images/activiti/activiti6-05/image-20200602194230562.png)

## 2.5、创建流程模型

这个我们在第3节已经介绍过，不做其他介绍。直接贴代码：

```java
    /**
     * 创建模型
     */
    @RequestMapping(value = "/modeler/create")
    @ResponseBody
    public CommonResponse create(@RequestParam(value = "name",required=true) String name,
                       @RequestParam(value = "key",required=true) String key,
                       @RequestParam(value = "description",required=true) String description) {
       /* String name = "请假流程";
        String key = "qingjia";
        String description = "这是一个简单的请假流程";*/

        try {
            ObjectMapper objectMapper = new ObjectMapper();
            ObjectNode editorNode = objectMapper.createObjectNode();
            editorNode.put("id", "canvas");
            editorNode.put("resourceId", "canvas");

            ObjectNode stencilSetNode = objectMapper.createObjectNode();
            stencilSetNode.put("namespace", "http://b3mn.org/stencilset/bpmn2.0#");

            editorNode.put("stencilset", stencilSetNode);

            ObjectNode modelObjectNode = objectMapper.createObjectNode();
            modelObjectNode.put(MODEL_NAME, name);
            modelObjectNode.put(ModelDataJsonConstants.MODEL_REVISION, 1);
            description = StringUtils.defaultString(description);
            modelObjectNode.put(MODEL_DESCRIPTION, description);

            Model newModel = repositoryService.newModel();
            newModel.setMetaInfo(modelObjectNode.toString());
            newModel.setName(name);
            newModel.setKey(StringUtils.defaultString(key));

            repositoryService.saveModel(newModel);
            repositoryService.addModelEditorSource(newModel.getId(), editorNode.toString().getBytes("utf-8"));

            System.out.println("生成的moduleId:"+newModel.getId());
            return new CommonResponse().code(CodeEnum.SUCCESS.getCode()).message("部署成功").data(newModel);
        } catch (Exception e) {
            logger.error("创建模型失败");
        }
        return new CommonResponse().code(CodeEnum.SUCCESS.getCode()).message("部署成功");
    }
```

![image-20200603164205645](/images/activiti/activiti6-05/image-20200603164205645.png)

## 2.6 、查看流程模型

访问 http://localhost:8080/modeler/modeler.html?modelId=1，就可以愉快的在上面设计流程图了。

![image-20200603164324344](/images/activiti/activiti6-05/image-20200603164324344.png)

流程定义文件—流程定义—流程模型 之间的相互转化，就介绍到这里。

# 7、删除已部署的流程定义

```
/**
     * 删除已部署的流程定义
     * @param deploymentId 流程部署ID
     */
    @GetMapping(value = "/process/delete/{deploymentId}")
    @ApiOperation(value = "删除已部署的流程定义",notes = "删除已部署的流程定义")
    public CommonResponse delete(@PathVariable("deploymentId") @ApiParam("流程定义ID (act_re_deployment表id)")  String deploymentId) {

        List<ProcessInstance> instanceList = runtimeService.createProcessInstanceQuery()
                .deploymentId(deploymentId)
                .list();
        if (!CollectionUtils.isEmpty(instanceList)) {
            // 存在流程实例的流程定义
            throw new CommonException("删除失败，存在运行中的流程实例");
        }
        repositoryService.deleteDeployment(deploymentId, true); // true 表示级联删除引用，比如 act_ru_execution 数据

        return new CommonResponse().code(CodeEnum.SUCCESS.getCode()).message("删除流程实例成功");
    }
```

![image-20200617155645515](/images/activiti6-05/image-20200617155645515.png)

# 8、删除流程模型

```java
@GetMapping(value = "/modeler/delete/{modelId}")
    @ApiOperation(value = "删除流程模型",notes = "删除流程模型")
    public CommonResponse remove(@PathVariable("modelId") @ApiParam("流程模型Id") String modelId) {
       repositoryService.deleteModel(modelId);
       return new CommonResponse().code(CodeEnum.SUCCESS.getCode()).message("删除流程模型成功");
    }
```

![image-20200617163620124](/images/activiti6-05/image-20200617163620124.png)