---
lab:
  title: 扩展管道以使用多个模板
  module: 'Module 5: Extend a pipeline to use multiple templates'
---

# 扩展管道以使用多个模板

在本实验室中，探索将管道扩展到多个模板的重要性，以及如何使用 Azure DevOps 执行此操作。 本实验室介绍创建多阶段管道、创建变量模板、创建作业模板以及创建阶段模板的基本概念和最佳做法。

此练习大约需要 **20** 分钟。

## 准备工作

需要 Azure 订阅、Azure DevOps 组织和 eShopOnWeb 应用程序才能遵循实验室。

- 按照步骤 [验证实验室环境](APL2001_M00_Validate_Lab_Environment.md)。

## 说明

### 练习 1：创建多阶段 YAML 管道

#### 任务 1：创建多阶段主 YAML 管道

1. 导航到 Azure DevOps 门户 `https://dev.azure.com` 并打开你的组织。

1. 打开 eShopOnWeb**** 项目。

1. 转到 **“管道”>“管道”**。

1. 选择“ **新建管道** ”按钮。

1. 选择“Azure Repos Git (YAML)”。

1. 选择“eShopOnWeb”存储库。

1. 选择“初学者管道”。

1. 将 azure-pipelines.yml**** 文件的内容替换为以下代码：

    ```YAML
    trigger:
    - main

    pool:
      vmImage: 'windows-latest'

    stages:
    - stage: Dev
      jobs:
      - job: Build
        steps:
        - script: echo Build
    - stage: Test
      jobs:
      - job: Test
        steps:
        - script: echo Test
    - stage: Production
      jobs:
      - job: Deploy
        steps:
        - script: echo Deploy

    ```

1. 选择**保存并运行**。 选择是要直接提交到主分支还是创建新分支。 选择“保存并运行”**** 按钮。

   > [!NOTE]
   > 如果选择创建新分支，则需要创建一个拉取请求以将更改合并到主分支。

1. 你将看到管道在三个阶段（开发、测试和生产）和相应的作业中运行。 等待管道完成并返回到“管道”**** 页。

    ![管道在三个阶段和相应作业中运行的屏幕截图](media/eshoponweb-pipeline-multi-stage.png)

1. 在刚刚创建的管道右侧，选择“...”****（更多选项），然后选择“重命名/移动”****。

1. 将管道重命名为 eShopOnWeb-MultiStage-Main****，然后选择“保存”****。

#### 任务 2：创建变量模板

1. 转到 Repos > 文件****。

1. 展开 .ado**** 文件夹，然后单击“新建文件”****。

1. 将文件命名为 eshoponweb-variables.yml**** ，然后单击“创建”****。

1. 将以下代码添加到该文件：

    ```YAML
    variables:
      resource-group: 'YOUR-RESOURCE-GROUP-NAME'
      location: 'southcentralus' #name of the Azure region you want to deploy your resources
      templateFile: '.azure/bicep/webapp.bicep'
      subscriptionid: 'YOUR-SUBSCRIPTION-ID'
      azureserviceconnection: 'YOUR-AZURE-SERVICE-CONNECTION-NAME'
      webappname: 'YOUR-WEB-APP-NAME'

    ```

    > [!IMPORTANT]
    > 将变量的值替换为环境的值（资源组、位置、订阅 ID、Azure 服务连接以及 Web 应用名称）。

1. 选择“提交”****，添加注释，然后选择“提交”**** 按钮。

#### 任务 3：准备管道以使用模板

1. 转到 **“管道”>“管道”**。

1. 打开 eShopOnWeb-MultiStage-Main**** 管道。

1. 选择“编辑”。

1. 将 azure-pipelines.yml**** 文件的内容替换为以下代码：

    ```YAML
    trigger:
    - main
    variables:
    - template: .ado/eshoponweb-variables.yml
    
    stages:
    - stage: Dev
      jobs:
      - template: .ado/eshoponweb-ci.yml
    - stage: Test
      jobs:
      - template: .ado/eshoponweb-cd-webapp-code.yml
    - stage: Production
      jobs:
      - job: Deploy
        steps:
        - script: echo Deploy to Production or Swap

    ```

1. 保存管道。

1. 选择是要直接提交到主分支还是创建新分支。 选择“保存”按钮。

   > [!NOTE]
   > 如果选择创建新分支，则需要创建一个拉取请求以将更改合并到主分支。

#### 任务 4：更新 CI/CD 模板

1. 转到 **“管道”>“管道”**。

1. 编辑 eshoponweb-ci**** 管道。

1. 删除“作业”**** 部分中的所有内容。

    ```YAML
    #NAME THE PIPELINE SAME AS FILE (WITHOUT ".yml")
    # trigger:
    # - main
    
    resources:
      repositories:
        - repository: self
          trigger: none
    
    stages:
    - stage: Build
      displayName: Build .Net Core Solution

    ```

1. 保存管道。

1. 转到 **“管道”>“管道”**。

1. 编辑 eshoponweb-cd-webapp-code**** 管道。

1. 删除“作业”**** 部分中的所有内容。

    ```YAML
    #NAME THE PIPELINE SAME AS FILE (WITHOUT ".yml")
    
    # Trigger CD when CI executed successfully
    resources:
      pipelines:
        - pipeline: eshoponweb-ci
          source: eshoponweb-ci # given pipeline name
          trigger: true

    variables:
      resource-group: 'rg-eshoponweb'
      location: 'southcentralus'
      templateFile: '.azure/bicep/webapp.bicep'
      subscriptionid: ''
      azureserviceconnection: 'azure subs'
      webappname: 'eshoponweb-lab'
      # webappname: 'webapp-windows-eshop'
    
    stages:
    - stage: Deploy
      displayName: Deploy to WebApp`

    ```

1. 将下载**** 步骤更新为：

    ```YAML
    - download: current
      artifact: Website
    - download: current
      artifact: Bicep
    ```

1. 保存管道。

1. （可选）更新生产步骤，将应用程序部署到另一个环境，或交换部署槽位。

#### 任务 5：运行主管道

1. 转到 **“管道”>“管道”**。

1. 打开 eShopOnWeb-MultiStage-Main**** 管道。

1. 选择“运行管道”。

1. 等待管道完成并检查结果。

    ![管道在三个阶段和相应作业中运行的屏幕截图](media/multi-stage-completed.png)

### 练习 2：删除 Azure 实验室资源

1. 在 Azure 门户中，打开创建的资源组，并为本实验室中的所有已创建资源选择“删除资源组”****。

    ![“删除资源组”按钮的屏幕截图。](media/delete-resource-group.png)

    > [!WARNING]
    > 记得删除所有不再使用的已创建的 Azure 资源。 删除未使用的资源可确保不会出现意外费用。

## 审阅

在本实验室中，你了解了将管道扩展到多个模板的重要性，以及如何使用 Azure DevOps 执行此操作。 本实验室介绍创建多阶段管道、创建变量模板、创建作业模板以及创建阶段模板的基本概念和最佳做法。
