## Pre requirement

#### 集群各节点密钥创建

- 通过root创建密钥

- 采用RSA算法
- 各节点密钥创建位置: /root/.ssh
- 各节点私钥名称设定: id_rsa

#### 执行器配置文件准备

- 准备运行执行器所需的配置文件

  - 需要在pre/pre_set_env.sh文件中先通过SERVERS参数设定集群所有节点的ip

  ```shell
  cd deploy
  sudo sh script/pre/pre_set_env.sh
  ```

#### 调度中心、执行器编译

```shell
cd src/xxl-job-admin
mvn install
cd ../xxl-job-executor-samples/xxl-job-executor-sample-springboot
mvn install
```

## K8S部署

#### mysql部署

- 创建mysql服务(包括pv, pvc, service)

  ```shell 
  kubectl apply -f deploy/mysql.yml
  ```
  
- 导入sql

  ```shell 
  PODNAME=$(kubectl get pod | grep mysql | awk '{print $1}')
  kubectl cp doc/db/tables_xxl_job.sql $PODNAME:/tmp/tables_xxl_job.sql
  kubectl exec -it $PODNAME -- mysql -u root --password=123456 -e 'source /tmp/tables_xxl_job.sql;'
  ```

#### CloudWrecker部署

- 定义执行器名字AppName：
  - 在deploy.yaml文件中executor部分，对env中NAME的值进行定义
  - 该值必须与调度中心注册执行器是的AppName一致

- 通过脚本部署ConfigMap，配置调度中心所需的相关变量

  ```shell
  sh deploy/script/script.sh
  ```

- 通过skaffold进行部署

  ```shell
  cd deploy
  skaffold run -d registry.cn-shenzhen.aliyuncs.com/cloud_wrecker
  ```

#### 卸载

  ```shell
  kubectl delete -f mysql.yml
  kubectl delete -f deploy.yaml
  ```
