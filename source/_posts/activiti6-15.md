---
title: Activiti（15） 历史数据
date: 2020-07-20 10:25:03
tags: Activiti
categories: Activiti
description: Activiti（15） 历史数据
typora-root-url: ..
password: kiki
---

本节我们介绍一下Activiti历史数据的查询。

历史数据的查询是相对于运行时数据来查询的，流程图跟踪就是根据历史数据来展示的，直观的展示当前流程的活动节点。除此外我们经常需要查询**历史流程实例**、**历史活动**、**历史流程变量**等

## 1、集成 Pagehelper分页插件

因为后面演示需要返回分页信息，所以我们集成下 Pagehelper分页插件。

- pom.xml

  ```xml
  <dependency>
      <groupId>com.github.pagehelper</groupId>
      <artifactId>pagehelper-spring-boot-starter</artifactId>
      <version>1.2.13</version>
      <!--使用spring boot2整合 pagehelper-spring-boot-starter必须排除一下依赖
                  因为pagehelper-spring-boot-starter也已经在pom依赖了mybatis与mybatis-spring
                  所以会与mybatis-plus-boot-starter中的mybatis与mybatis-spring发生冲突
              -->
      <exclusions>
          <exclusion>
              <groupId>org.mybatis</groupId>
              <artifactId>mybatis</artifactId>
          </exclusion>
          <exclusion>
              <groupId>org.mybatis</groupId>
              <artifactId>mybatis-spring</artifactId>
          </exclusion>
      </exclusions>
  </dependency>
  ```

- application.yaml

  ```yaml
  pagehelper:
    helperDialect: mysql
    # reasonable：分页合理化参数，默认值为false。当该参数设置为 true 时，pageNum<=0 时会查询第一页， 		pageNum>pages（超过总数时），会查询最后一页。默认false 时，直接根据参数进行查询。
    reasonable: true
    supportMethodsArguments: true
    params: count=countSql
  ```

- 示例代码

  ```java
  
  public PageInfo ecample(Integer pageNum, Integer pageSize,String keyWord) {
      //在mapper方法前通过指定page页码与size每页条数
      Page<Object> pageObject = PageHelper.startPage(pageNum, pageSize);
      //这里就无须指定页码与条数并且该方法的对应sql也无须写limit
      //sql示例 select * from employee where name = #{keyword}(注意无须写limit分页)
      List<Employee> emps = employeeMapper.getEmployeeByPage(keyWord);
      //PageInfo中包装了分页信息
      PageInfo<Employee> info = new PageInfo<>(emps);
      return info;
  }
  ```

- PageInfo类封装的信息详情

  ```java
  //当前页
  private int pageNum;   
  //每页的数量
  private int pageSize;
  //当页的数量
  private int size;
  //当前页面第一个元素在数据库中的行号
  private int startRow;
  //当前页面最后一个元素在数据库中的行号
  private int endRow;
  //总记录数
  private long total;
  //总页数
  private int pages;
  //结果集
  private List<T> list;
  //前一页 页码
  private int prePage;
  //下一页 页码
  private int nextPage;
  //是否为第一页
  private boolean isFirstPage;
  //是否为最后一页
  private boolean isLastPage;
  //是否还有上一页
  private boolean hasPreviousPage;
  //是否还有下一页
  private boolean hasNextPage;
  //导航页码数
  private int navigatePages;
  //所有导航页
  private int[] navigatepageNums;
  //导航页的第一页码
  private int navigateFirstPage;
  //导航页的最后一页码
  private int navigateLastPage;
  ```

  





## 2、流程历史管理

我们新建一个类来专门管理流程历史：ProcessHistoryController。流程历史主要就是利用7大Service接口中的`HistoryService`

### 2.1  获取流程的历史任务

```java
@Autowired
private ProcessHistoryService processHistoryService;

@ApiOperation(value = "获取流程的历史任务",notes = "获取流程的历史任务")
@GetMapping (value = "/getHistoryTaskList")
public CommonResponse getHistoryTaskList(
    @RequestParam(value = "finished", required = false,defaultValue = "2")
    @ApiParam(value = "是否已完成( 0：已完成的任务  1：未完成的任务 2：全部任务（默认）)" ,required = false)String finished,
    @RequestParam(value = "processInstanceId", required = true)
    @ApiParam(value = "流程实例ID" ,required = true)String processInstanceId){
    List<HistoricTaskInstance> list = processHistoryService.getHistoryTaskList(finished, processInstanceId);
    return new CommonResponse().code(CodeEnum.SUCCESS.getCode()).data(list);
}


------------------------------------------------------------------
    
@Override
public List<HistoricTaskInstance> getHistoryTaskList(String finished,String processInstanceId) {
    List<HistoricTaskInstance> list = null;
    if (finished.equals("0")){
        list = historyService // 历史任务Service
            .createHistoricTaskInstanceQuery() // 创建历史任务实例查询
            .orderByHistoricTaskInstanceStartTime().asc() //排序
            //.taskAssignee(GlobalConfig.getOperator()) // 指定办理人
            .processInstanceId(processInstanceId)
            .finished() // 查询已经完成的任务
            .list();
    }else if (finished.equals("1")){
        list = historyService // 历史任务Service
            .createHistoricTaskInstanceQuery() // 创建历史任务实例查询
            .orderByHistoricTaskInstanceStartTime().asc()
            //.taskAssignee(GlobalConfig.getOperator()) // 指定办理人
            .processInstanceId(processInstanceId)
            .unfinished()// 查询未完成的任务
            .list();
    }else{
        list = historyService // 历史任务Service
            .createHistoricTaskInstanceQuery() // 创建历史任务实例查询
            .orderByHistoricTaskInstanceStartTime().asc()
            //.taskAssignee(GlobalConfig.getOperator()) // 指定办理人
            .processInstanceId(processInstanceId)
            .list();
    }

    return list;
}
```

![image-20200720182209097](/images/activiti6-15/image-20200720182209097.png)

### 2.2 获取流程的历史活动

历史活动和历史任务有什么区别呢？历史活动包含历史任务，历史任务只包含任务节点的数据（用户任务、服务任务等）。历史活动还包含所有节点信息。

```java
@ApiOperation(value = "获取流程的历史活动",notes = "获取流程的历史活动")
@GetMapping (value = "/getHistoryActInstanceList")
public CommonResponse getHistoryActInstanceList(
    @RequestParam(value = "finished", required = false,defaultValue = "2")
    @ApiParam(value = "是否已完成( 0：已完成的任务  1：未完成的任务 2：全部任务（默认）)" ,required = false)String finished,
    @RequestParam(value = "processInstanceId", required = true)
    @ApiParam(value = "流程实例ID" ,required = true)String processInstanceId){
    
    List<HistoricActivityInstance> list = processHistoryService.getHistoryActInstanceList(finished, processInstanceId);
    return new CommonResponse().code(CodeEnum.SUCCESS.getCode()).data(list);
}


-----------------------------------------------------------------------------
@Override
public List<HistoricActivityInstance> getHistoryActInstanceList(String finished, String processInstanceId) {
    List<HistoricActivityInstance> list = null;
    if (finished.equals("0")){
        list = historyService // 历史任务Service
            .createHistoricActivityInstanceQuery()
            .orderByHistoricActivityInstanceStartTime().asc()
            .processInstanceId(processInstanceId)
            .finished()
            .list();
    }else if (finished.equals("1")){
        list = historyService // 历史任务Service
            .createHistoricActivityInstanceQuery()
            .orderByHistoricActivityInstanceStartTime().asc()
            .processInstanceId(processInstanceId)
            .unfinished()
            .list();
    }else{
        list = historyService // 历史任务Service
            .createHistoricActivityInstanceQuery()
            .orderByHistoricActivityInstanceStartTime().asc()
            .processInstanceId(processInstanceId)
            .list();
    }

    return list;
}
```

![image-20200720182819300](/images/activiti6-15/image-20200720182819300.png)

### 2.3 获取流程历史流程变量

```java
@ApiOperation(value = "获取流程历史流程变量",notes = "获取流程历史流程变量")
@GetMapping (value = "/getHistoryProcessVariables")
public CommonResponse getHistoryProcessVariables(
    @RequestParam(value = "processInstanceId", required = true)
    @ApiParam(value = "流程实例ID" ,required = true)String processInstanceId){
    List<HistoricVariableInstance> list = processHistoryService.getHistoryProcessVariables(processInstanceId);
    return new CommonResponse().code(CodeEnum.SUCCESS.getCode()).data(list);
}

------------------------------------------------------------------
@Override
public List<HistoricVariableInstance> getHistoryProcessVariables( String processInstanceId) {
    List<HistoricVariableInstance> list = null;
    list = historyService
        .createHistoricVariableInstanceQuery()//创建一个历史的流程变量查询对象
        .processInstanceId(processInstanceId)//
        .list();
    return list;
}
```

![image-20200720183118953](/images/activiti6-15/image-20200720183118953.png)

### 2.4 获取已归档的流程实例

已归档就是流程结束的实例。

```java
@ApiOperation(value = "获取已归档的流程实例",notes = "获取已归档的流程实例")
@GetMapping (value = "/getFinishedInstanceList")
public CommonResponse getFinishedInstanceList(
          @RequestParam(value = "processInstanceId", required = true)
          @ApiParam(value = "流程定义ID" ,required = true)String processDefinitionId,
          @RequestParam(value = "pageNum", required = false,defaultValue = "1")
          @ApiParam(value = "页码" ,required = false)int pageNum,
          @RequestParam(value = "pageSize", required = false,defaultValue = "10")
          @ApiParam(value = "条数" ,required = false)int pageSize){
    PageInfo<HistoricProcessInstance> list = processHistoryService.getFinishedInstanceList(processDefinitionId,  pageNum,  pageSize);
    return new CommonResponse().code(CodeEnum.SUCCESS.getCode()).data(list);
}

----------------------------------------------------------
@Override
public PageInfo<HistoricProcessInstance> getFinishedInstanceList(String processDefinitionId,int pageNum, int pageSize) {
    List<HistoricProcessInstance> list = null;

    PageHelper.startPage(pageNum, pageSize);
    list = historyService.createHistoricProcessInstanceQuery()
        .processDefinitionId(processDefinitionId)
        .finished().list();
    PageInfo<HistoricProcessInstance> pageAttachments = new PageInfo<>(list);
    return pageAttachments;
}
```

![image-20200720183531375](/images/activiti6-15/image-20200720183531375.png)

```json
{
  "code": "0000",
  "data": {
    "total": 1,
    "list": [
      {
        "id": "67501",
        "processInstanceId": "67501",
        "processDefinitionId": "purchase-error-event-subprocess-kiki:1:65004",
        "startTime": "2020-07-15 07:06:37",
        "endTime": "2020-07-15 07:40:16",
        "durationInMillis": 2018557,
        "deleteReason": null,
        "endActivityId": "catchErrorForPayment",
        "businessKey": null,
        "startUserId": "kafeitu",
        "startActivityId": "startevent1",
        "superProcessInstanceId": null,
        "tenantId": "",
        "name": null,
        "localizedName": null,
        "description": null,
        "localizedDescription": null,
        "processDefinitionKey": "purchase-error-event-subprocess-kiki",
        "processDefinitionName": "办公用品采购--事件子流程(异常)",
        "processDefinitionVersion": 1,
        "deploymentId": "65001",
        "queryVariables": null,
        "processVariables": {},
        "persistentState": {
          "processDefinitionId": "purchase-error-event-subprocess-kiki:1:65004",
          "durationInMillis": 2018557,
          "endStateName": "catchErrorForPayment",
          "deploymentId": "65001",
          "businessKey": null,
          "name": null,
          "processDefinitionName": "办公用品采购--事件子流程(异常)",
          "endTime": "2020-07-15 07:40:16",
          "deleteReason": null,
          "processDefinitionVersion": 1,
          "superProcessInstanceId": null,
          "processDefinitionKey": "purchase-error-event-subprocess-kiki"
        },
        "deleted": false,
        "inserted": false,
        "updated": false
      }
    ],
    "pageNum": 1,
    "pageSize": 1,
    "size": 1,
    "startRow": 0,
    "endRow": 0,
    "pages": 1,
    "prePage": 0,
    "nextPage": 0,
    "isFirstPage": true,
    "isLastPage": true,
    "hasPreviousPage": false,
    "hasNextPage": false,
    "navigatePages": 8,
    "navigatepageNums": [
      1
    ],
    "navigateFirstPage": 1,
    "navigateLastPage": 1
  }
}
```

此处查出了已完结的流程信息，并且包含分页信息。

## 3、获取流程定义列表

定义模块，补充一个获取流程定义列表的方法。



## 4、分页插件的问题

在第一步我们就集成了pageHelper分页插件，但是在本地各种测试时发现查询activiti数据和使用mybatis-plus方法时，pageHelper总是不起作用。经过一番查找得出如下结论。

### 4.1 mybatis-plus 在3.x 版本后不再支持pageHelper插件。

具体原因参考：

[https://github.com/baomidou/mybatis-plus/blob/3.0/CHANGELOG.md](https://github.com/baomidou/mybatis-plus/blob/3.0/CHANGELOG.md)

https://github.com/baomidou/mybatis-plus/issues/801

![image-20200729111514298](/images/activiti6-15/image-20200729111514298.png)

Mybatis-Plus3 升级了自己的分页插件，也很强大。使用文档可以参考：https://mybatis.plus/guide/page.html

但是不是意味着在系统中不应该出现pageHelper的包呢？不是的。

我们都知道MP是对mybatis的增强，如果我们不使用Mybatis帮我们封装好的方法（通用mapper等），只使用Mybatis的方法，自己写接口、xml配置，pageHelper依然还是可以使用的。

```java
public List<Leave>  getAll(){
        /**
         * 如果使用MP帮我们封装好的方法，例如这个 selectBatchIds，则PageHelper分页失效。但是getAll方法是自己写的，则可以继续使用
         * PageHelper.startPage(2,2);
         * List<Integer> list = new ArrayList<>();
         *     list.add(1);
         *     list.add(2);
         *     list.add(3);
         *  List<Leave> leaveList = leaveMapper.selectBatchIds(list);
         *  PageInfo<Leave> pageInfo = new PageInfo<>(leaveList);//分页失效
         */
        PageHelper.startPage(2,2);
        List<Leave> leaveList = leaveMapper.getAll();
        PageInfo<Leave> pageInfo = new PageInfo<>(leaveList);//分页有效
        return leaveList;
    }
```

```java
@Repository
public interface LeaveMapper extends BaseMapper<Leave> {
    List<Leave> getAll();
}
```

```xml
<select id="getAll" resultType="com.sxdx.workflow.activiti.rest.entity.Leave">
    select <include refid="Base_Column_List" />
    from oa_leave where 1=1
</select>
```

### 4.2  pageHelper不支持activiti 

各种尝试后，发现`pageHelper`不支持`activiti` 。原因是activiti做复杂查询貌似`activiti`底层会做多次查询，而`pageHelper`默认只查询最近的那条SQL。所以只能使用activiti 自带的 `listPage` 分页方法了。

参考：https://github.com/pagehelper/Mybatis-PageHelper/issues/152



