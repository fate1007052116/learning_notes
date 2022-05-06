# Activiti

### 流程设计器下载地址

> https://camunda.com/products/camunda-bpm/modeler/
>
> ![Snip20210117_1](/Users/luo/Documents/开发笔记/images/Snip20210117_1.png)
>
> 这里要点击`cancel`
>
> 注意：他会把`assignee`和`candicate`的前缀改成自己的
>
> `activiti:candidateUsers="luo,gao"`还有`activiti:assignee="${manager}"`
>
> 部署的时候记得改回来，还要加上命名空间`xmlns:activiti="http://activiti.org/bpmn"`
>
> 否则，部署上去的`bpmn`根据`candidate`或`assignee`查询不到任务
>
> ```xml
> <definitions xmlns="http://www.omg.org/spec/BPMN/20100524/MODEL" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:activiti="http://activiti.org/bpmn" xmlns:bpmndi="http://www.omg.org/spec/BPMN/20100524/DI" xmlns:omgdc="http://www.omg.org/spec/DD/20100524/DC" xmlns:omgdi="http://www.omg.org/spec/DD/20100524/DI" xmlns:xsd="http://www.w3.org/2001/XMLSchema" targetNamespace="http://www.activiti.org/processdef" exporter="Camunda Modeler" exporterVersion="4.5.0">
> 
>       <userTask id="sid-09b7fd38-7714-4197-b9bd-557e8d1b0e28" name="部门审批" activiti:candidateUsers="luo,gao" />
>     <userTask id="sid-23e1f849-7fe6-421e-b67b-75c7d73b32cf" name="经理审批" activiti:assignee="${manager}" />
> ```
>
> 

> [actiBmp idea插件下载地址](https://plugins.jetbrains.com/plugin/7429-actibpm/versions/stable/17789)
>
> 但`Idea 2020`使用的时候没有左边栏

### 1.创建出的`bpmn、xml`文件不能执行

> 极度注意：只能上传`bpmn`文件到数据库，不可以是`xml`，否则`ACT_RE_PROCDEF`表中将插入不了数据

```shell
[Validation set: 'activiti-executable-process' | Problem: 'activiti-process-definition-not-executable'] : All process definition are set to be non-executable (property 'isExecutable' on process). This is not allowed. - [Extra info : ]
```

解决方法：

设置`bpmn、xml`下面的`isExecutable="true"`就可解决

```xml

<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<bpmn:definitions xmlns:bpmn="http://www.omg.org/spec/BPMN/20100524/MODEL" xmlns:activiti="http://activiti.org/bpmn" xmlns:bpmndi="http://www.omg.org/spec/BPMN/20100524/DI" xmlns:dc="http://www.omg.org/spec/DD/20100524/DC" xmlns:di="http://www.omg.org/spec/DD/20100524/DI" xmlns:tns="http://bpmn.io/schema/bpmn" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" expressionLanguage="http://www.w3.org/1999/XPath" id="Definitions_18z4knx" name="" targetNamespace="http://bpmn.io/schema/bpmn" typeLanguage="http://www.w3.org/2001/XMLSchema">
  <bpmn:process id="Process_1nqyu80" isClosed="false" isExecutable="true" processType="None">

```

### 2.部署工作流的时候会操作的表

> `RE`：`resource`资源信息

#### （1）`ACT_RE_DEPLOYMENT`

流程定义的部署表，每一次部署工作流，对应的会在这张表中添加一条数据

> ![Snip20210101_2](/Users/luo/Documents/开发笔记/images/Snip20210101_2.png)

#### （2）`ACT_RE_PROCDEF`

流程定义表（如果测试中，没有操作这张表，要将`xml`扩展名改为`bpmn`）

> 对应的`xml`配置是` <bpmn:startEvent id="StartEvent_1ka4138" name="事件开始"/>`
>
> ```xml
> <bpmn:definitions xmlns:bpmn="http://www.omg.org/spec/BPMN/20100524/MODEL" xmlns:activiti="http://activiti.org/bpmn" xmlns:bpmndi="http://www.omg.org/spec/BPMN/20100524/DI" xmlns:dc="http://www.omg.org/spec/DD/20100524/DC" xmlns:di="http://www.omg.org/spec/DD/20100524/DI" xmlns:tns="http://bpmn.io/schema/bpmn" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" expressionLanguage="http://www.w3.org/1999/XPath" id="Definitions_18z4knx" name="" targetNamespace="http://bpmn.io/schema/bpmn" typeLanguage="http://www.w3.org/2001/XMLSchema">
>   <bpmn:process id="Process_1nqyu80" isClosed="false" isExecutable="true" processType="None" name="出差申请">
>     <bpmn:startEvent id="StartEvent_1ka4138" name="事件开始"/>
> ```
>
> ![Snip20210101_1](/Users/luo/Documents/开发笔记/images/Snip20210101_1.png)

#### （3）`ACT_GE_PROPERTY`

每次部署时都会先查询再更新这张表，

> ```sql
> select * from ACT_GE_PROPERTY where NAME_ = ? 
> 
> update ACT_GE_PROPERTY SET REV_ = 2, VALUE_ = 2501 where NAME_ = next.dbid and REV_ = 1 
> 
> -- 把 next.dbid 的 VALUE_ 和 REV_ 向上加
> ```
>
> 
>
> ![Snip20210101_4](/Users/luo/Documents/开发笔记/images/Snip20210101_4.png)

#### （5）`ACT_GE_BYTEARRAY`

流程资源表

> ![Snip20210101_5](/Users/luo/Documents/开发笔记/images/Snip20210101_5.png)

### 3.流程定义部署

（1）使用流程设计器，使用流程符号

> 在线`bpmn`设计网站：https://demo.bpmn.io/
>
> 注意，用这个网站设计了之后，需要设置`isExecutable="true"`，还要设置流程的名称
>
> ```xml
> <bpmn:process id="Process_1nqyu80" isClosed="false" isExecutable="true" processType="None" name="出差申请">
> ```

`bpmn`文件、`png`文件都是流程资源文件，用来描述流程，流程中需要的节点，节点的负责人

出差申请流程，请加申请流程，报销申请流程

（2）把流程的资源文件，进行部署

上传到数据库中，使用`java`代码进行流程的部署

`ACT_RE_DEPLOYMENT`和`ACT_RE_PROCDEF`是一对多的关系

>张三的出差申请在`ACT_RE_PROCDEF`中对应一条记录
>
>李四的出差申请也在`ACT_RE_PROCDEF`中对应一条记录
>
>不同的人在进行出差申请的时候会在`ACT_RE_PROCDEF`表中有多条记录
>
>而上面的`n`个人的记录对应的都只是一次`ACT_RE_DEPLOYMENT`的部署

### 4.查询个人待执行的任务

```sql
select distinct RES.* from ACT_RU_TASK RES inner join ACT_RE_PROCDEF D on RES.PROC_DEF_ID_ = D.ID_ WHERE RES.ASSIGNEE_ = '高孔燕' and D.KEY_ = 'myEvection' order by RES.ID_ asc LIMIT 2147483647 OFFSET 0
```

#### （1）ACT_RU_TASK

当前正在运行的任务

#### （2）ACT_RE_PROCDEF

流程实例表 

### 5.完成个人的审批

个人审批之后，任务上一个执行的状态会被移动到`ACT_HI_TASKINST`表中

![Snip20210104_1](/Users/luo/Documents/开发笔记/images/Snip20210104_1.png)

同时更新任务的当前状态，`ACT_RU_TASK`表

![Snip20210104_2](/Users/luo/Documents/开发笔记/images/Snip20210104_2.png)

#### （1）SQL分析

```sql
select * from ACT_RU_TASK where ID_ = 2505

select * from ACT_RU_VARIABLE where TASK_ID_ = 2505
```

1、查询流程定义表

```sql
select * from ACT_RE_PROCDEF where ID_ = 'myEvection:1:4'
```

![Snip20210104_3](/Users/luo/Documents/开发笔记/images/Snip20210104_3.png)

2、查询流程部署（得到已部署的流程的名字）

```sql
select * from ACT_RE_DEPLOYMENT where ID_ = 1 
```

![Snip20210104_4](/Users/luo/Documents/开发笔记/images/Snip20210104_4.png)

3、查询流程资源

```sql
select * from ACT_GE_BYTEARRAY where DEPLOYMENT_ID_ = 1 order by NAME_ asc 
```

![Snip20210104_5](/Users/luo/Documents/开发笔记/images/Snip20210104_5.png)

4、

```sql
select * from ACT_RE_PROCDEF where DEPLOYMENT_ID_ = 1 and KEY_ = 'myEvection' and (TENANT_ID_ = '' or TENANT_ID_ is null) 
```

![Snip20210104_7](/Users/luo/Documents/开发笔记/images/Snip20210104_7.png)

#### （2）更新操作

1、`ACT_GE_PROPERTY`

```sql
update ACT_GE_PROPERTY SET REV_ = 4, VALUE_ = 7501 where NAME_ = 'next.dbid' and REV_ = 3
```

2、`ACT_HI_TASKINST`：插入经理审批

```sql
insert into ACT_HI_TASKINST ( ID_, PROC_DEF_ID_, PROC_INST_ID_, EXECUTION_ID_, NAME_, PARENT_TASK_ID_, DESCRIPTION_, OWNER_, ASSIGNEE_, START_TIME_, CLAIM_TIME_, END_TIME_, DURATION_, DELETE_REASON_, TASK_DEF_KEY_, FORM_KEY_, PRIORITY_, DUE_DATE_, CATEGORY_, TENANT_ID_ ) values ( ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ? ) 
5002(String), myEvection:1:4(String), 2501(String), 2502(String), 经理审批(String), null, null, null, 罗俊华(String), 2021-01-04 21:26:19.819(Timestamp), null, null, null, null, Activity_1x9d48q(String), null, 50(Integer), null, null, (String)
```

![Snip20210104_8](/Users/luo/Documents/开发笔记/images/Snip20210104_8.png)

3、`ACT_HI_ACTINST`：插入到历史流程的参与者

```sql
insert into ACT_HI_ACTINST ( ID_, PROC_DEF_ID_, PROC_INST_ID_, EXECUTION_ID_, ACT_ID_, TASK_ID_, CALL_PROC_INST_ID_, ACT_NAME_, ACT_TYPE_, ASSIGNEE_, START_TIME_, END_TIME_, DURATION_, DELETE_REASON_, TENANT_ID_ ) values ( ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ? ) 
5001(String), myEvection:1:4(String), 2501(String), 2502(String), Activity_1x9d48q(String), 5002(String), null, 经理审批(String), userTask(String), 罗俊华(String), 2021-01-04 21:26:19.799(Timestamp), null, null, null, (String)
```

![Snip20210104_10](/Users/luo/Documents/开发笔记/images/Snip20210104_10.png)

4、`ACT_HI_IDENTITYLINK`：插入到历史流程参与者

```sql
insert into ACT_HI_IDENTITYLINK (ID_, TYPE_, USER_ID_, GROUP_ID_, TASK_ID_, PROC_INST_ID_) values (?, ?, ?, ?, ?, ?) 
5003(String), participant(String), 罗俊华(String), null, null, 2501(String)
```

![Snip20210104_9](/Users/luo/Documents/开发笔记/images/Snip20210104_9.png)

5、`ACT_RU_TASK`：正在运行的任务

```sql
insert into ACT_RU_TASK (ID_, REV_, NAME_, BUSINESS_KEY_, PARENT_TASK_ID_, DESCRIPTION_, PRIORITY_, CREATE_TIME_, OWNER_, ASSIGNEE_, DELEGATION_, EXECUTION_ID_, PROC_INST_ID_, PROC_DEF_ID_, TASK_DEF_KEY_, DUE_DATE_, CATEGORY_, SUSPENSION_STATE_, TENANT_ID_, FORM_KEY_, CLAIM_TIME_, APP_VERSION_) values (?, 1, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ? ) 
5002(String), 经理审批(String), null, null, null, 50(Integer), 2021-01-04 21:26:19.799(Timestamp), null, 罗俊华(String), null, 2502(String), 2501(String), myEvection:1:4(String), Activity_1x9d48q(String), null, null, 1(Integer), (String), null, null, null
```

![Snip20210104_11](/Users/luo/Documents/开发笔记/images/Snip20210104_11.png)

6、`ACT_RU_IDENTITYLINK`：当前正在执行的流程的参与者

```sql
insert into ACT_RU_IDENTITYLINK (ID_, REV_, TYPE_, USER_ID_, GROUP_ID_, TASK_ID_, PROC_INST_ID_, PROC_DEF_ID_) values (?, 1, ?, ?, ?, ?, ?, ?) 
5003(String), participant(String), 罗俊华(String), null, null, 2501(String), null
```

![Snip20210104_12](/Users/luo/Documents/开发笔记/images/Snip20210104_12.png)

6、`ACT_RU_EXECUTION`：更新

```sql
update ACT_RU_EXECUTION set REV_ = ?, BUSINESS_KEY_ = ?, PROC_DEF_ID_ = ?, ACT_ID_ = ?, IS_ACTIVE_ = ?, IS_CONCURRENT_ = ?, IS_SCOPE_ = ?, IS_EVENT_SCOPE_ = ?, IS_MI_ROOT_ = ?, PARENT_ID_ = ?, SUPER_EXEC_ = ?, ROOT_PROC_INST_ID_ = ?, SUSPENSION_STATE_ = ?, NAME_ = ?, IS_COUNT_ENABLED_ = ?, EVT_SUBSCR_COUNT_ = ?, TASK_COUNT_ = ?, JOB_COUNT_ = ?, TIMER_JOB_COUNT_ = ?, SUSP_JOB_COUNT_ = ?, DEADLETTER_JOB_COUNT_ = ?, VAR_COUNT_ = ?, ID_LINK_COUNT_ = ?, APP_VERSION_ = ? where ID_ = ? and REV_ = ? 
2(Integer), null, myEvection:1:4(String), Activity_1x9d48q(String), true(Boolean), false(Boolean), false(Boolean), false(Boolean), false(Boolean), 2501(String), null, 2501(String), 1(Integer), null, false(Boolean), 0(Integer), 0(Integer), 0(Integer), 0(Integer), 0(Integer), 0(Integer), 0(Integer), 0(Integer), null, 2502(String), 1(Integer)
```

![Snip20210104_14](/Users/luo/Documents/开发笔记/images/Snip20210104_14.png)

7、`ACT_HI_TASKINST`：更新历史任务实例

```sql
 update ACT_HI_TASKINST set PROC_DEF_ID_ = ?, EXECUTION_ID_ = ?, NAME_ = ?, PARENT_TASK_ID_ = ?, DESCRIPTION_ = ?, OWNER_ = ?, ASSIGNEE_ = ?, CLAIM_TIME_ = ?, END_TIME_ = ?, DURATION_ = ?, DELETE_REASON_ = ?, TASK_DEF_KEY_ = ?, FORM_KEY_ = ?, PRIORITY_ = ?, DUE_DATE_ = ?, CATEGORY_ = ? where ID_ = ? 
 myEvection:1:4(String), 2502(String), 创建出差申请(String), null, null, null, 高孔燕(String), null, 2021-01-04 21:26:19.732(Timestamp), 290472174(Long), null, Activity_10pvhm3(String), null, 50(Integer), null, null, 2505(String)
```

8、

```sql
update ACT_HI_ACTINST set EXECUTION_ID_ = ?, ASSIGNEE_ = ?, END_TIME_ = ?, DURATION_ = ?, DELETE_REASON_ = ? where ID_ = ? 
2502(String), 高孔燕(String), 2021-01-04 21:26:19.76(Timestamp), 290472223(Long), null, 2504(String)
```

9、删除

```sql
delete from ACT_RU_TASK where ID_ = ? and REV_ = ? 
2505(String), 1(Integer)
```

# 进阶

### 1.流程

![Snip20210112_1](/Users/luo/Documents/开发笔记/images/Snip20210112_1.png)



### 2.流程变量

不建议将大量的变量存储到`activiti`所管理的表中

![Snip20210113_1](/Users/luo/Documents/开发笔记/images/Snip20210113_1.png)

#### （1）流程变量的作用域

* 流程实例（`processInstance`）
* 任务（`task`）
* 执行实例（`execution`）

1、`global`变量

流程变量默认的作用域是`流程实例`，当一个流程变量的作用域为流程实例时，可以称为`global`变量

> 注意：
>
> `Global`变量：`userId`（变量名），`张三`（变量值）
>
> `Global`变量中变量名不允许重复，设置名称相同的变量，后设置的变量值会覆盖之前设置的变量值

2、`local`变量

任务和执行实例仅仅是针对一个任务和一个执行实例的范围，范围没有`流程实例`大，称为`local`变量

`local`变量由于在不同的任务或不同的执行实例中，作用域互不影响，变量名可以设置相同没有影响。

`local`变量也可以和`global`变量名相同，无影响

#### （2）使用流程变量的方式

![Snip20210113_1](/Users/luo/Documents/开发笔记/images/Snip20210113_1.png)

### 3.组任务

#### （1）办理流程

​	1、查询组任务

​		指定候选人，查询该候选人当前的待办任务，候选人不能立即办理任务

​	2、拾取任务

​		该任务组的所有候选人都可以拾取任务

​		将候选人组的任务变成个人任务。原来的候选人就变成了该任务的负责人

> 如果拾取之后不想办理该任务？
>
> 需要将已经拾取的任务归还到组里面，将个人任务变成组任务

​	3、查询个人任务

​		查询方式与个人任务部分相同，根据`assignee`查询用户负责的个人任务

​	4、办理个人任务

### 4.网关

> 用于控制流程的走向

#### （1）排他网关 Exclusive

用来再流程中实现决策，当流程执行到这个网关，所有分支都会判断条件是否为`true`，如果为`true`则执行该分支

> 注意：排他网关只会选择一个为`true`的分支执行。如果有两个分支的条件都是`true`，排他网关会选择`id`较小的那一条分支去执行
>
> 为什么要用排他网关？
>
> ​	不用排他网关也可以实现分支，例如：在连线的（`condition`）条件上设置分支条件
>
> ​		连线设置`condition`的缺点：如果条件都不满足，流程就结束了（异常结束）

如果网关出去的所有线都不满足条件（`false`），则系统抛出异常

#### （2）并行网关

允许将流程分为多条分支，也可以把多条分支汇聚到一起，并行网关的功能是基于`进入和外出`顺序流的

1、`fork`分支

并行后的所有外出顺序流，为每个顺序流都创建一个并行分支

2、`join`汇聚

所有抵达并行网关，在此等待的进入分支，直到所有进入顺序流的分支都抵达以后，流程就会通过汇聚网关。

> 注意：如果一个并行网关有多个进入和多个外出顺序流，他就同时具有分支和汇聚功能。这时，网关会先汇聚所有进入的顺序流，然后再切分成多个并行分支
>
> 与其他网关的主要区别是：
>
> **并行网关不会解析条件，即使顺序流中定义了条件，也会被忽略**



#### （3）包含网关 InclusiveGateway

包含网关可以看作是排他网关和并行网关的结合体

和排他网关一样，你可以在外出顺序流上定义条件，包含网关会解析他们。但是主要的区别是**包含网关可以选择多余一条顺序流，这和并行网关一样**

包含网关的功能是基于进入和外出顺序流的

​	1、分支

所有外出顺序流的条件都会被解析，结果为`true`的顺序流会以并行方式继续运行，会为每一个顺序流创建一个分支

​	2、汇聚

所有并行分支到达包含网关，会进入等待状态，直到每个包含流程的`token`的进入顺序流的分支都到达，这是与并行网关最大的不同。换句话说，包含网关只会等待被选中执行了的进入顺序流。在汇聚之后，流程会穿过网关继续执行。



# vue 整合 bpmn-js

```shell
sudo vue init webpack vue-bpmn

npm install bpmn-js
```







