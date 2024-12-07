---
lab:
  title: 配置项目和存储库结构以支持安全管道
  module: 'Module 1: Configure a project and repository structure to support secure pipelines'
---

# 配置项目和存储库结构以支持安全管道

在本实验室中，你将了解如何在 Azure DevOps 中配置项目和存储库结构，以支持安全管道。 本实验室介绍组织项目和存储库、分配权限以及管理安全文件的最佳做法。

这些练习大约需要 **30** 分钟才能完成。

## 准备工作

需要 Azure 订阅、Azure DevOps 组织和 eShopOnWeb 应用程序才能遵循实验室。

- 按照步骤 [验证实验室环境](APL2001_M00_Validate_Lab_Environment.md)。

## 说明

### 练习 1：配置安全项目结构

在本练习中，你将通过创建新项目并为其分配项目权限来配置安全项目结构。 将职责和资源分解到具有特定权限的不同项目或存储库支持安全性。

#### 任务 1：创建新团队项目

1. 导航到 Azure DevOps 门户 `https://aex.dev.azure.com` 并打开你的组织。

1. 打开门户左下角的“**组织设置**”，然后在“常规”部分下打开“**项目**”。

1. 选择“新建项目”**** 选项并使用以下设置：

   - 名称：**eShopSecurity**
   - 可见性：专用
   - 高级：版本控制：Git
   - 高级：工作项流程：Scrum

   ![具有指定设置的新项目对话框的屏幕截图。](media/new-team-project.png)

1. 选择“创建”以创建新项目。

1. 现在，可以通过单击 Azure DevOps 门户左上角的 Azure DevOps 图标在不同的项目之间进行切换。

   ![Azure DevOps 团队项目 eShopOnWeb 和 eShopSecurity 的屏幕截图。](media/azure-devops-projects.png)

可以通过转到“项目设置”菜单并选择相应的团队项目来单独管理每个项目的权限和设置。 如有多个用户或团队处理不同的项目，还可以单独为每个项目分配权限。

#### 任务 2：创建新存储库并分配项目权限

1. 选择 Azure DevOps 门户左上角的组织名称，然后选择新的“**eShopSecurity**”项目。

1. 选择“**Repos**”菜单。

1. 选择“**初始化**”按钮，以通过添加 README.md 文件来初始化新存储库。

1. 打开门户左下角的“**项目设置**”菜单，然后选择“Repos”部分下的“**存储库**”。

1. 选择新的“**eShopSecurity**”存储库，然后选择“**安全**”选项卡。

   > **备注**：确保仅选择特定存储库中的“安全”选项卡，而不是为项目中的所有存储库选择该选项卡。 如果选择所有存储库，则可能无法访问项目中的其他存储库。

1. 通过取消选中“**继承**”切换按钮，从父级移除“继承”权限。

1. 选择“**参与者**”组，并为除“**管理权限**”和“**读取**”以外的所有权限选择“**拒绝**”下拉列表。 此操作将阻止来自参与者组的所有用户访问存储库。

   > **备注**：在实际方案中，你还将拒绝对“参与者”组的管理权限。 对于此实验室，我们允许参与者组管理权限，以允许你完成实验室。

1. 在“用户”下选择用户，然后选择“**允许**”按钮以允许所有权限。

   > **备注**：如果未在“**用户**”部分看到名称，请在“**搜索用户或组**”文本框中输入名称，然后在结果列表中选择它。

   ![存储库安全设置的屏幕截图，其中允许读取并拒绝所有其他权限。](media/repository-security.png)

1. 更改将自动保存。

现在，只有你分配了权限的用户和管理员才能访问存储库。 如果要允许特定用户从 eShopOnWeb 项目访问存储库并运行管道，这会非常有用。

### 练习 2：配置管道和模板结构以支持安全管道

#### 任务 1：导入并运行 CI 管道

让我们首先导入名为 [eshoponweb-ci.yml](https://github.com/MicrosoftLearning/eShopOnWeb/blob/main/.ado/eshoponweb-ci.yml) 的 CI 管道。

1. 导航到 Azure DevOps 门户 `https://aex.dev.azure.com` 并打开你的组织。

1. 在 Azure DevOps 中打开 **eShopOnWeb** 项目。

1. 转到“**管道 > 管道**”。

1. 选择“**创建管道**”按钮。

1. 选择“Azure Repos Git (YAML)”。

1. 选择“eShopOnWeb”存储库。

1. 选择“现有 Azure Pipelines YAML 文件”。

1. 选择“/.ado/eshoponweb-ci.yml”文件，然后单击“继续”。

1. 选择“**运行**”按钮以运行管道。

   > **备注**：管道将采用基于项目名称的名称。 你将重命名它，以便更轻松地标识管道。

1. 转到“**管道”>“管道**”，然后选择最近创建的管道。 选择省略号，然后选择“**重命名/移动**”选项。

1. 将其命名为 **eshoponweb-ci**，然后选择“**保存**”。

#### 任务 2：导入并运行 CD 管道

> **备注**：在此任务中，你将导入并运行名为 [eshoponweb-cd-webapp-code.yml](https://github.com/MicrosoftLearning/eShopOnWeb/blob/main/.ado/eshoponweb-cd-webapp-code.yml) 的 CD 管道。

1. 转到“**管道 > 管道**”。

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

现在，应在 eShopOnWeb 项目中运行两个管道。

![成功执行的 CI/CD 管道的屏幕截图。](media/pipeline-successful-executed.png)

#### 任务 3：将 CD 管道变量移动到 YAML 模板

在此任务中，你将创建一个 YAML 模板来存储 CD 管道中使用的变量。 通过此操作，可以在其他管道中重用模板。

1. 转到“**Repos**”，然后转到“**文件**”。

1. 展开“**.ado**”文件夹并选择“**新建文件**”。

1. 将文件命名为 **eshoponweb-secure-variables.yml**，然后选择“**创建**”。

1. 将 CD 管道中使用的变量部分添加到新文件。 该文件应如下所示：

   ```yaml
   variables:
     resource-group: 'rg-eshoponweb-secure'
     location: 'southcentralus' #the name of the Azure region you want to deploy your resources
     templateFile: 'infra/webapp.bicep'
     subscriptionid: 'YOUR-SUBSCRIPTION-ID'
     azureserviceconnection: 'azure subs' #the name of the service connection to your Azure subscription
     webappname: 'eshoponweb-lab-secure-XXXXXX' #the globally unique name of the web app
   ```

   > **重要提示**：将变量的值替换为环境的值（资源组、位置、订阅 ID、Azure 服务连接以及 Web 应用名称）。

1. 选择“**提交**”，在提交注释文本框中输入 `[skip ci]`，然后选择“**提交**”。

   > **备注**：通过向提交添加 `[skip ci]` 注释，将会阻止自动管道执行，此时，默认情况下此操作会在每次更改存储库后运行。

1. 从存储库中的文件列表打开 **eshoponweb-cd-webapp-code.yml** 管道定义，并将变量部分替换为以下内容：

   ```yaml
   variables:
     - template: eshoponweb-secure-variables.yml
   ```

1. 选择“**提交**”，接受默认注释，然后选择“**提交**”以再次运行管道。

1. 验证管道运行是否已成功完成。

现在，你有一个 YAML 模板，其中包含 CD 管道中使用的变量。 如果需要部署相同的资源，可以在其他管道中重用此模板。 此外，运营团队还可以控制部署资源的资源组和位置以及模板值中的其他信息，你无需对管道定义进行任何更改。

#### 任务 4：将 YAML 模板移动到单独的存储库和项目

在此任务中，你将 YAML 模板移动到单独的存储库和项目。

1. 在 eShopSecurity 项目中，转到“**Repos”>“文件**”。

1. 创建名为 **eshoponweb-secure-variables.yml** 的新文件。

1. 将文件 **.ado/eshoponweb-secure-variables.yml** 的内容从 eShopOnWeb 存储库复制到新文件。

1. 提交更改。

1. 在 eShopOnWeb 存储库中打开 **eshoponweb-cd-webapp-code.yml** 管道定义。

1. 在管道定义中的变量部分之前，将以下内容添加到资源部分：

   ```yaml
     repositories:
       - repository: eShopSecurity
         type: git
         name: eShopSecurity/eShopSecurity #name of the project and repository
   ```

1. 将变量部分替换为以下内容：

   ```yaml
   variables:
     - template: eshoponweb-secure-variables.yml@eShopSecurity #name of the template and repository
   ```

   ![包含新变量和资源部分的管道定义屏幕截图。](media/pipeline-variables-resource-section.png)

1. 选择“**提交**”，接受默认注释，然后选择“**提交**”以再次运行管道。

1. 导航到管道运行并验证管道是否正在使用 eShopSecurity 存储库中的 YAML 文件。

   ![使用 eShopSecurity 存储库中的 YAML 模板进行管道执行的屏幕截图。](media/pipeline-execution-using-template.png)

现在，你已在单独的存储库和项目中拥有 YAML 文件。 如果需要部署相同的资源，可以在其他管道中重用此文件。 此外，运营团队还可以通过修改 YAML 文件中的值来控制资源组、位置、安全性和资源部署位置和其他信息，并且无需对管道定义进行任何更改。

> [!IMPORTANT]
> 请记住删除在 Azure 门户中创建的资源，以避免不必要的费用。

## 审阅

在本实验室中，你已了解如何在 Azure DevOps 中配置和组织安全项目与存储库结构。 通过有效管理权限，可以确保适当的用户有权访问所需的资源，同时维护 DevOps 管道和过程的安全性和完整性。
