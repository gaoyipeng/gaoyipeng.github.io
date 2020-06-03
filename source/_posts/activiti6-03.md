---
title: Activiti（3）集成Activiti Modeler
date: 2020-05-28 14:00:14
tags: Activiti
categories: Activiti
description: Activiti（3）集成Activiti Modeler
typora-root-url: ..
password: kiki
---

> Activiti Modeler 是 Activiti 官方提供的一款在线流程设计的前端插件，可以方便流程设计与开发人员绘制流程图，保存流程模型，部署至流程定义等等。

本节我们能就记录如何在spring boot项目中集成Activiti Modeler。

# 1、材料准备

首先我们需要获取activiti-explorer.zip，这个是activiti-5.22.0才有的。

链接：https://pan.baidu.com/s/1Qd633JJbddF2Hz5EpXTbAw 
提取码：8d5o

后续所有的资料都可以通过这个地址下载。

# 2、集成

## 2.1 集成静态资源

- 在resources下新建static/modeler目录

  找到 activiti-5.22.0\wars\activiti-explorer.war，将如图3个目录复制到static/modeler目录下。

![image-20200528160054524](/images/activiti/activiti6-03/image-20200528160054524.png)

- 将下载好的汉化文件stencilset.json复制到src/main/resources目录下
- 用下载好的汉化文件en.json替换 static/modeler/editor-app/i18n/en.json文件



## 2.2 集成后端代码

- 在 com/sxdx/workflow/activiti/rest 下新建modeler目录，将下载文件的 activiti-5.22.0\Activiti-master\modules\activiti-modeler\src\main\java\org\activiti\rest\editor 目录下的所有文件拷贝到此目录下：

  ![image-20200528161729033](/images/activiti/activiti6-03/image-20200528161729033.png)

## 2.3 代码修改

![image-20200528173114196](/images/activiti/activiti6-03/image-20200528173114196.png)

依次修改如下代码：

1、static/modeler/editor-app/app-cfg.js

![image-20200528162141138](/images/activiti/activiti6-03/image-20200528162141138.png)

2、static/modeler/editor-app/configuration/toolbar-default-actions.js。

![image-20200528162335896](/images/activiti/activiti6-03/image-20200528162335896.png)

3、com/sxdx/workflow/activiti/rest/modeler 下的3个java文件中的所有方法的访问路径前都添加  `/modeler`

![image-20200528162512739](/images/activiti/activiti6-03/image-20200528162512739.png)



# 3、访问验证

启动服务，访问 http://localhost:8080/modeler/modeler.html

![image-20200528162901706](/images/activiti/activiti6-03/image-20200528162901706.png)

会发现只有一个布局，各种功能及组件都没有显示出来。**这是因为我们没有定义流程模型**。

# 4、创建一个模型

## 4.1 代码创建一个流程模型

创建一个测试类 ModelerControllerTest

```
package com.sxdx.workflow.activiti.rest;

import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.node.ObjectNode;
import org.activiti.editor.constants.ModelDataJsonConstants;
import org.activiti.engine.RepositoryService;
import org.activiti.engine.repository.Model;
import org.apache.commons.lang3.StringUtils;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

import javax.annotation.Resource;

import static org.activiti.editor.constants.ModelDataJsonConstants.MODEL_DESCRIPTION;
import static org.activiti.editor.constants.ModelDataJsonConstants.MODEL_NAME;

@SpringBootTest
@RunWith(SpringRunner.class)
public class ModelerControllerTest {

    @Resource
    private RepositoryService repositoryService;

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
}

```

结果：

![image-20200528171503269](/images/activiti/activiti6-03/image-20200528171503269.png)

我们查看一下 act_re_model 表

![image-20200528182553777](/images/activiti/activiti6-03/image-20200528182553777.png)

## 4.2  再次访问 

这次访问: http://localhost:8080/modeler/modeler.html?modelId=1, 这次就显示正常了。接下来我们就可以绘制流程图了。

**注意**：这个modelId参数是必须的，否则modeler的组件无法显示。

![image-20200528183846729](/images/activiti/activiti6-03/image-20200528183846729.png)