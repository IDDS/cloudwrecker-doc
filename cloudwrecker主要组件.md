# 模块功能

## 调用中心

### 任务调度组件JobScheduleHelper
维护一个待触发任务队列，存放即将触发的任务，达到触发时间时触发任务调度。
功能：
* 主要负责定时周期性任务的触发
* 读取表xxl_job_info中触发时间在5000ms之内的任务，分以下三种情况对任务进行处理：
    * 任务触发时间已过期5s，不进行调度，更新下一次触发时间。
    * 任务触发时间已过，过期时间小于5s，直接触发任务调度，更新下一次触发时间，如果下一次触发时间在5s内，将任务放入待触发队列。
    * 未到任务触发时间，更新下一次触发时间，将任务放入待触发任务队列。

### 任务调度触发组件JobTriggerPoolHelper
将任务调度到执行器执行，通过RPC调用将相关参数发送给执行器。

### 任务失败告警组件JobFailMonitorHelper
读取表xxl_job_log，获取失败任务日志。对于剩余重试次数的任务，触发任务调度，实现失败重试功能。对于配置了告警邮箱的任务，发送任务失败的告警邮件。

### 注册信息更新组件JobRegistryMonitorHelper
刷新执行器和调度中心的注册地址，移除表xxl_job_registry中失效的记录。

### 任务回调处理组件AdminBizImpl
处理执行器回调传输过来的信息，对于执行成功的任务，触发子任务的执行，对于执行失败的任务，记录失败日志。

### 任务统计报告组件JobLogReportHelper
统计任务执行情况，包括总任务数，失败任务数和成功任务数，用于数据展示面板。

### k8s资源更新组件ResourceUpdateHelper
从k8s集群的apiserver获取node、pod、container等资源的信息。

## 执行器
### 任务执行组件ExecutorBizImpl
接收调度中心发送的任务参数，根据任务配置选择相应的工作线程。每个工作线程都与一种类型的jobHandler进行绑定，工作线程中有一个任务队列，队列中存放的是配置了同一种jobHander的任务，任务被依次取出并交给jobHandler执行，任务执行的结果存放在回调队列中。

### 任务回调触发组件TriggerCallbackThread
从回调队列中取出任务执行结果，使用AdminBizClient将结果发送给调度中心。

### 调度中心client AdminBizClient
通过RPC调用调度中心的回调接口将任务执行结果发送给调度中心，调用注册接口进行执行器注册。

### 日志清理组件
定期清理执行器保存在本地的日志。

### 各种类型的jobHandler
每种类型的jobHandler对应一种故障场景的实现，比如NodeCPUJobHandler就是对node节点注入cpu故障的实现。jobHandler通过http调用blade server，将故障的参数配置发送给blade server，控制故障注入的时间。

### blade server
chaosblade的server版本，可以通过restful api来调用，实现故障注入。blade server部署在k8s集群中的某一个节点上。
