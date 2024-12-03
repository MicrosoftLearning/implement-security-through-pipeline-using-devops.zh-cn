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

在本练习中，将在 Azure DevOps 中创建多阶段 YAML 管道。

#### 任务 1：创建多阶段主 YAML 管道

1. 导航到 Azure DevOps 门户 `https://aex.dev.azure.com` 并打开你的组织。

1. 打开 eShopOnWeb**** 项目。

1. 转到 **“管道”>“管道”**。

1. 单击“**新建管道**”按钮。

1. 选择“Azure Repos Git (YAML)”。

1. 选择“eShopOnWeb”存储库。

1. 选择“初学者管道”。

1. 将 azure-pipelines.yml**** 文件的内容替换为以下代码：

   ```yaml
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

1. 选择**保存并运行**。 选择直接提交到主分支，然后再次选择“**保存并运行**”。

1. 你将看到管道在三个阶段（开发、测试和生产）和相应的作业中运行。 等待管道完成，然后并导航回到“**管道**”页。

   ![管道在三个阶段和相应作业中运行的屏幕截图](media/eshoponweb-pipeline-multi-stage.png)

1. 在刚刚创建的管道右侧，选择“...”****（更多选项），然后选择“重命名/移动”****。

1. 将管道重命名为 eShopOnWeb-MultiStage-Main****，然后选择“保存”****。

#### 任务 2：创建变量模板

1. 转到 Repos > 文件****。

1. 展开 .ado**** 文件夹，然后单击“新建文件”****。

1. 将文件命名为 eshoponweb-variables.yml**** ，然后单击“创建”****。

1. 将以下代码添加到该文件：

   ```yaml
   variables:
     resource-group: 'YOUR-RESOURCE-GROUP-NAME'
     location: 'centralus'
     templateFile: 'infra/webapp.bicep'
     subscriptionid: 'YOUR-SUBSCRIPTION-ID'
     azureserviceconnection: 'YOUR-AZURE-SERVICE-CONNECTION-NAME'
     webappname: 'YOUR-WEB-APP-NAME'
   ```

1. 将变量的值替换为环境的值：

   - 将 **YOUR-RESOURCE-GROUP-NAME** 替换为要在此实验室中使用的资源组名称，例如 **rg-eshoponweb-secure**。
   - 将**位置**变量的值设置为要部署资源的 Azure 区域的名称，例如 **centralus**。
   - 将 **YOUR-SUBSCRIPTION-ID** 替换为 Azure 订阅 ID。
   - 将 **YOUR-AZURE-SERVICE-CONNECTION-NAME** 替换为 **azure subs**
   - 将 **YOUR-WEB-APP-NAME** 替换为要部署的 Web 应用的全局唯一名称，例如，字符串 **eshoponweb-lab-multi-123456**，后跟一个随机的六位数。  

1. 选择“**提交**”，在提交注释文本框中输入 `[skip ci]`，然后选择“**提交**”。

   > **备注**：通过向提交添加 `[skip ci]` 注释，将会阻止自动管道执行，此时，默认情况下此操作会在每次更改存储库后运行。 

#### 任务 3：准备管道以使用模板

1. 在 Azure DevOps 门户中的“**eShopOnWeb**”项目页上，转到“**Repos**”。

1. 在存储库的根目录中，选择包含 **eShopOnWeb-MultiStage-Main** 管道定义的“**azure-pipelines.yml**”。

1. 单击“**编辑**”按钮。

1. 将 azure-pipelines.yml**** 文件的内容替换为以下代码：

   ```yaml
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

1. 选择“**提交**”，在提交注释文本框中输入 `[skip ci]`，然后选择“**提交**”。

#### 任务 4：更新 CI/CD 模板

1. 在 **eShopOnWeb** 项目的 **Repos** 中，选择“**.ado**”目录，然后选择“**eshoponweb-ci.yml**”文件。

1. 单击“**编辑**”按钮。

1. 删除“作业”**** 部分中的所有内容。

   ```yaml
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

1. 选择“**提交**”，在提交注释文本框中输入 `[skip ci]`，然后选择“**提交**”。

1. 在 **eShopOnWeb** 项目的 **Repos** 中，选择“**.ado**”目录，然后选择“**eshoponweb-cd-webapp-code.yml**”文件。

1. 单击“**编辑**”按钮。

1. 删除“作业”**** 部分中的所有内容。

   ```yaml
    # NAME THE PIPELINE SAME AS FILE (WITHOUT ".yml") #
    # Trigger CD when CI executed successfully
    
    resources:
      pipelines:
        - pipeline: eshoponweb-ci
          source: eshoponweb-ci # given pipeline name
          trigger: true
    
    repositories:
      - repository: eShopSecurity
        type: git
        name: eShopSecurity/eShopSecurity # name of the project and repository
    
    variables:
      - template: eshoponweb-secure-variables.yml@eShopSecurity # name of the template and repository
    
    stages:
      - stage: Test
        displayName: Testing WebApp
        jobs:
          - deployment: Test
            pool: eShopOnWebSelfPool
            environment: Test
            strategy:
              runOnce:
                deploy:
                  steps:
                    - script: echo Hello world! Testing environments!
    
      - stage: Deploy
        displayName: Deploy to WebApp
   ```

1. 将 **#download 项目**步骤的现有内容替换为：

   ```yaml
       - download: current
         artifact: Website
       - download: current
         artifact: Bicep
   ```

1. 选择“**提交**”，在提交注释文本框中输入 `[skip ci]`，然后选择“**提交**”。

#### 任务 5：运行主管道

1. 转到 **“管道”>“管道”**。

1. 打开 eShopOnWeb-MultiStage-Main**** 管道。

1. 选择“运行管道”。

   > **注意**：如果收到消息“管道需要访问资源的权限才能继续运行”，请选择“**查看**”，选择“**允许**”，然后再次选择“**允许**”以允许管道运行。

   > **备注**：如果部署阶段中任何作业失败，请导航到“管道运行”页，然后选择“**重新运行失败的作业***”。

1. 等待管道完成并检查结果。

   ![管道在三个阶段和相应作业中运行的屏幕截图](media/multi-stage-completed.png)

> [!IMPORTANT]
> 请记住删除在 Azure 门户中创建的资源，以避免不必要的费用。

## 审阅

在本实验室中，你已了解如何使用 Azure DevOps 将管道扩展到多个模板中。 本实验室介绍了创建多阶段管道、创建变量模板、作业模板和阶段模板的基本概念和最佳做法。
