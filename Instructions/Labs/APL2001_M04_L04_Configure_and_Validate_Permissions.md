---
lab:
  title: 配置和验证权限
  module: 'Module 4: Configure and validate permissions'
---

# 配置和验证权限

在本实验室中，你将设置一个遵循最小权限原则的安全环境，确保成员只能访问执行任务所需的资源，将潜在的安全风险降到最低。 它涉及在 Azure DevOps 中配置和验证用户和管道权限以及设置审批和分支检查。

此练习大约需要 **20** 分钟。

## 准备工作

需要 Azure 订阅、Azure DevOps 组织和 eShopOnWeb 应用程序才能遵循实验室。

- 按照步骤 [验证实验室环境](APL2001_M00_Validate_Lab_Environment.md)。
- 按照实验室“[为安全管道配置代理和代理池](APL2001_M02_L02_Configure_Agents_And_Agent_Pools_for_Secure_Pipelines.md)”或“[安装自托管代理](https://learn.microsoft.com/azure/devops/pipelines/agents/windows-agent)”中的步骤安装自托管代理。

## 说明

### 练习 0：（如果已完成，请跳过此任务）导入和运行 CI/CD 管道

在本练习中，你将在 Azure DevOps 项目中导入并运行 CI 管道。

#### 任务 1：（如果已完成，请跳过此任务）导入并运行 CI 管道

让我们首先导入名为 [eshoponweb-ci.yml](https://github.com/MicrosoftLearning/eShopOnWeb/blob/main/.ado/eshoponweb-ci.yml) 的 CI 管道。

1. 导航到 Azure DevOps 门户 `https://aex.dev.azure.com` 并打开你的组织。

1. 在 Azure DevOps 中打开 **eShopOnWeb** 项目。

1. 转到 **“管道”>“管道”**。

1. 选择“**创建管道**”按钮。

1. 选择“Azure Repos Git (YAML)”。

1. 选择“eShopOnWeb”存储库。

1. 选择“现有 Azure Pipelines YAML 文件”。

1. 选择“/.ado/eshoponweb-ci.yml”文件，然后单击“继续”。

1. 选择“**运行**”按钮以运行管道。

   > **备注**：管道将采用基于项目名称的名称。 你将重命名它，以便更轻松地标识管道。

1. 转到“**管道”>“管道**”，然后选择最近创建的管道。 选择省略号，然后选择“**重命名/移动**”选项。

1. 将其命名为 **eshoponweb-ci**，然后选择“**保存**”。

#### 任务 2：（如果已完成，请跳过此任务）导入并运行 CD 管道

> **备注**：在此任务中，你将导入并运行名为 [eshoponweb-cd-webapp-code.yml](https://github.com/MicrosoftLearning/eShopOnWeb/blob/main/.ado/eshoponweb-cd-webapp-code.yml) 的 CD 管道。

1. 转到 **“管道”>“管道”**。

1. 选择“**新建管道**”按钮。

1. 选择“Azure Repos Git (YAML)”。

1. 选择“eShopOnWeb”存储库。

1. 选择“现有 Azure Pipelines YAML 文件”。

1. 选择“**/.ado/eshoponweb-cd-webapp-code.yml**”文件，然后选择“**继续**”。

1. 在 YAML 管道定义中，将变量部分设为：

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

1. 选择“**保存并运行**”，然后选择直接提交到主分支。

1. 再次选择“**保存并运行**”。

1. 打开管道运行。 如果看到消息“此管道需要访问资源的权限，然后才能继续运行‘部署到 WebApp’”，请选择“**查看**”、“**允许**”，然后再次选择“**允许**”。 这是允许管道创建 Azure 应用服务资源所必需的。

   ![允许从 YAML 管道访问的屏幕截图。](media/pipeline-deploy-permit-resource.png)

1. 部署可能需要几分钟才能完成，请等待管道执行。 管道会在 CI 管道完成后触发，其中包括以下任务：

   - **AzureResourceManagerTemplateDeployment**：使用 bicep 模板部署 Azure 应用服务 Web 应用。
   - **AzureRmWebAppDeployment**：将网站发布到 Azure 应用服务 Web 应用。

   > **备注**：如果部署失败，请导航到管道运行页，然后选择“**重新运行失败的作业**”，以调用另一个管道运行。

   > **备注**：管道将采用基于项目名称的名称。 让我们重命名它，以便更好地识别管道。

1. 转到“**管道”>“管道**”，然后选择最近创建的管道。 选择省略号，然后选择“**重命名/移动**”选项。

1. 将其命名为 eshoponweb-cd-webapp-code，然后单击“保存”。

### 练习 1：配置和验证审批和分支检查

在本练习中，你将配置并验证 CD 管道的审批和分支检查。

#### 任务 1：创建环境并添加审批和检查

1. 在 Azure DevOps 门户中的“**eShopOnWeb**”项目页上，选择“**管道 > 环境**”。

1. 选择“创建环境”。

1. 将该环境命名为“**测试**”，选择“**无**”作为资源，然后选择“**创建**”。

1. 在“**测试**”环境中，选择“**审批并检查**”选项卡。

1. 选择“审批”。

1. 在“**审批者**”文本框中，输入用户名。

1. 如果未启用，请选中标记为“允许审批者批准自己的运行”的框。

1. 提供“**批准部署以进行测试**”说明，并选择“**创建**”。

   ![包含说明的环境审批的屏幕截图。](media/add-environment-approvals.png)

1. 单击“**新增**”按钮，选择“**分支控制**”，然后选择“**下一步**”。

1. 在“**允许的分支**”字段中，保留默认值并选择“**创建**”。 如果需要，可以添加更多分支。

   ![包含主分支的环境分支控件的屏幕截图。](media/add-environment-branch-control.png)

1. 创建名为“**生产**”的另一个环境，并执行相同的步骤来添加审批和分支控制。 若要区分环境，请添加“**批准部署到生产**”说明，并将允许的分支设置为 **refs/heads/main**。

> **备注**：可以添加更多环境并为其配置审批和分支控制。 此外，你还可以配置“**安全性**”，以将用户或组添加到具有“*用户*”、“*创建者*”或“*读取者*”等角色的环境中。

#### 任务 2：将 CD 管道配置为使用新环境

1. 在 Azure DevOps 门户中的“**eShopOnWeb**”项目页上，选择“**管道 > 管道**”。

1. 打开管道 **eshoponweb-cd-webapp-code**。

1. 选择“编辑”  。

1. 在管道 YAML 文件中选择 **#下载项目**注释上方的行，向上直到**阶段：** 行，并将内容替换为以下代码：

   ```yaml
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
     jobs:
     - deployment: Deploy
       pool: eShopOnWebSelfPool
       environment: Production
       strategy:
         runOnce:
           deploy:
             steps:
             - checkout: self
   ```

   > **备注**：需要将上述代码后面的所有行向右移动六个空格，以确保满足 YAML 缩进规则。

   管道应如下所示：

   ![包含新部署的管道的屏幕截图。](media/pipeline-add-yaml-deployment.png)

   > [!IMPORTANT]
   > 确认**池**名称与在上一个实验室中创建的名称相同。

1. 单击“**验证并保存**”，选择直接提交到主分支，然后单击“**保存**”。

1. 管道将自动触发。 打开管道运行。

   > **备注**：如果收到消息“此管道需要访问资源的权限，然后才能继续测试 WebApp”，请选择“**查看**”、“**允许**”，然后再次选择“**允许**”。

1. 打开管道的“**测试 WebApp**”阶段，并记下消息“**有 1 个审批需要你的批准，然后此运行才能进入测试 WebApp 阶段**。” 选择“**审批**”，然后选择“**批准**”。

   ![要批准的测试阶段的管道的屏幕截图。](media/pipeline-test-environment-approve.png)

1. 等待管道完成，打开管道日志，并检查是否已成功执行**测试 WebApp** 阶段。

   ![管道日志的屏幕截图，其中测试 WebApp 阶段已成功执行”。](media/pipeline-test-environment-success.png)

1. 返回到管道，你将看到“**部署到 WebApp**”阶段在等待审批。 依次选择“**审批**”和“**批准**”，就像针对**测试 WebApp** 阶段所做的那样。

   > **备注**：如果收到消息“此管道需要访问资源的权限，然后才能继续部署 WebApp”，请选择“**查看**”、“**允许**”，然后再次选择“**允许**”。

1. 等待管道完成，并检查是否已成功执行**部署到 WebApp** 阶段。

   ![要批准的“部署到 WebApp”的管道的屏幕截图。](media/pipeline-deploy-environment-success.png)

> **备注**：应该能够在测试和生产环境中通过审批和分支检查成功运行管道。

> [!IMPORTANT]
> 请记住删除在 Azure 门户中创建的资源，以避免不必要的费用。

## 审阅

在本实验室中，你学习了如何设置一个遵循最小特权原则的安全环境，以确保成员只能访问执行任务所需的资源，将潜在的安全风险降到最低。 你在 Azure DevOps 中配置并验证了用户和管道权限，并设置了审批和分支检查。
