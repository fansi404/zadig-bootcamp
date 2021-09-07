summary: 基于 Spring Boot 的微服务持续交付
id: springboot
categories: Spring Boot
environments: Web
status: Published
feedback link: https://github.com/koderover/zadig-bootcamp/issues

# 基于 Spring Boot 的微服务持续交付

## 概述

Duration: 0:01:00

本文介绍如何在 Zadig 上快速搭建 Spring Boot 项目。Spring Boot 是构建 Java 后端应用程序的一种非常流行的框架，本文案例项目主要包含 Maven 构建的 Worker 、DB(Postgres) 以及 Redis 这三个服务，以下步骤包含从 Code 到 Ship 的整个过程的演示。

## 准备工作

Duration: 0:02:00

- 项目案例源码：[`https://github.com/koderover/zadig/tree/main/examples/voting-app/worker`](https://github.com/koderover/Zadig/tree/main/examples/voting-app/worker)（也可配置自己的私有仓库，然后[集成代码源](https://docs.koderover.com/zadig/settings/codehost/github/)）
- 服务 Yaml 文件：[`https://github.com/koderover/zadig/tree/main/examples/voting-app/freestyle-k8s-specifications/worker`](https://github.com/koderover/Zadig/tree/main/examples/voting-app/freestyle-k8s-specifications/worker)
- 服务 Dockerfile 文件：[`https://github.com/koderover/zadig/blob/main/examples/voting-app/worker/Dockerfile.j`](https://github.com/koderover/Zadig/blob/main/examples/voting-app/worker/Dockerfile.j)

## 项目配置

Duration: 0:02:00

- 创建项目，具体内容如下图所示：

![创建项目](./img/springboot_create_project.png "创建项目")

- 接下来会看到创建成功的提示：

![创建项目](./img/springboot_succeeded_to_create_project.png "创建项目成功提示")
## 创建服务

Duration: 0:02:00

- 创建服务，如下图所示：

![创建服务](./img/springboot_createService.png "创建服务")

Positive
: 创建服务有两种方式，第一种是选择平台编辑直接把 Yaml 内容粘到系统中，第二种是选择从仓库导入 Yaml 文件，示例中采用第二种。
- 点击仓库托管，在弹出的窗口中选择代码仓库，同步 `examples`->`voting-app`->`freestyle-k8s-specifications`->`worker` 文件目录，加载 `worker` 服务 Yaml 配置文件。
- 系统会自动检测 Yaml 格式是否合法，右侧会自动解析出来系统变量、自定义变量和服务组件，这里也可以继续添加自定义变量。具体过程如下图所示：

![创建服务](./img/springboot_load_service_yaml.gif "加载服务配置")

- 创建构建，这里可以选择 worker-e2e 这个服务组件添加构建，然后选择代码库和添加构建脚本，具体如下图所示：

![创建构建](./img/springboot_create_build.png "创建构建")

构建脚本如下：

```dockerfile
cd $WORKSPACE/zadig/examples/voting-app/worker
docker build -t $IMAGE -f Dockerfile.j .
docker push $IMAGE
```
添加完成后，点击`保存构建`。

- 点击仓库托管，继续添加 [DB](https://github.com/koderover/zadig/tree/main/examples/voting-app/freestyle-k8s-specifications/db)(Postgres) 和 [Redis](https://github.com/koderover/zadig/tree/main/examples/voting-app/freestyle-k8s-specifications/redis) 服务，如下图所示。


![添加db和redis](./img/springboot_add_db.png "添加db和redis")

Positive
: 1. 实际场景中已存在现有可用的中间件服务，在网络联通情况下可以直接使用 <br> 2. 若同一中间件服务针对不同环境需要使用不同的服务地址，可以通过设置环境变量的方式配置

- 添加成功后，点击下一步，完成服务配置。

## 加入运行环境

Duration: 0:01:00

- 进入「加入运行环境」，系统会自动创建两套集成环境和三条工作流，两套集成环境可分别给开发和测试使用，三条工作流也会自动绑定对应的集成环境以达到持续交付的目的。具体如下图所示：

![创建环境](./img/springboot_create_project_result.png "创建环境")

## 工作流交付

Duration: 0:01:00

- 点击运行工作流 `voting-app-workflow-dev` 对 `worker` 服务进行更新，来实现 `dev` 环境的持续交付

![运行工作流](./img/springboot_run_dev_worker.png "运行工作流")
![运行工作流](./img/springboot_run_pipeline_result.png "运行工作流")

- `qa` 环境的持续交付类似的操作，不赘述。

到此，您已熟悉 Zadig 的基本功能了，下面将展示如何配置自动触发工作流。

## 配置自动触发工作流

Duration: 0:04:00

- 创建基于 Github 事件的触发器，修改 `voting-app-workflow-dev` 工作流，为 `dev` 环境设置 Github 事件触发器，系统目前支持 push 和 pull_request 两种事件，具体如下图所示：

Positive
: 前提条件：需要配置 GitHub Webhook，全局 Webhook 请参考 [GitHub Webhook](https://docs.koderover.com/zadig/settings/webhook-config/#gitlab-webhook-%E9%85%8D%E7%BD%AE)

![为工作流添加 Webhook](./img/springboot_create_webhook.png "为工作流添加 Webhook")

- 提交 [代码库](https://github.com/koderover/zadig/tree/main/examples/voting-app/worker) PR，在 Check runs 中会展示具体的系统工作流的状态，具体如下图所示：

![创建 PR](./img/springboot_create_pr.png "创建 PR")

- 点击任务链接可以跳转到 Zadig 系统里查看工作流具体执行信息，如下图所示：

![workflow 执行详情](./img/springboot_webhook_triggered_pipeline.png "workflow 执行详情")

- 待工作流执行完毕，可看到 `dev` 环境 `worker` 服务的镜像已经被更新了，具体如下图所示：

![worker 镜像更新](./img/springboot_triggered_pipeline_env_stats.png "worker 镜像更新")