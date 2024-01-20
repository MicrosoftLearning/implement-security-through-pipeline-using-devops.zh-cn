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

1. 导航到 Azure DevOps 门户 `https://dev.azure.com` 并打开你的组织。

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

1. 通过取消选中“**继承**”切换按钮，从父级移除“继承”权限。

1. 选择“**参与者**”组，并为除“**读取**”以外的所有权限选择“**拒绝**”下拉列表。 此操作将阻止来自参与者组的所有用户访问存储库。

1. 在“用户”下选择用户，然后选择“**允许**”按钮以允许所有权限。

   > [!NOTE]
   > 如果未在“**用户**”部分看到姓名，请在“**搜索用户或组**”文本框中输入姓名，然后在结果列表中选择它。

   ![存储库安全设置的屏幕截图，其中允许读取并拒绝所有其他权限。](media/repository-security.png)

1. 更改将自动保存。

现在，只有你分配了权限的用户和管理员才能访问存储库。 如果要允许特定用户从 eShopOnWeb 项目访问存储库并运行管道，这会非常有用。

### 练习 2：配置管道和模板结构以支持安全管道

#### 任务 1：导入并运行 CI 管道

1. 导航到 Azure DevOps 门户 `https://dev.azure.com` 并打开你的组织。

1. 在 Azure DevOps 中打开 **eShopOnWeb** 项目。

1. 转到 **“管道”>“管道”**。

1. 选择“**创建管道**”按钮。

1. 选择“Azure Repos Git (YAML)”。

1. 选择“eShopOnWeb”存储库。

1. 选择“现有 Azure Pipelines YAML 文件”。

1. 选择“**/.ado/eshoponweb-ci.yml**”文件，然后选择“**继续**”。

1. 选择“**运行**”按钮以运行管道。

   > [!NOTE]
   > 管道将采用基于项目名称的名称。 你将重命名它，以便更轻松地标识管道。

1. 转到“**管道”>“管道**”，然后选择最近创建的管道。 选择省略号，然后选择“**重命名/移动**”选项。

1. 将其命名为 **eshoponweb-ci**，然后选择“**保存**”。

#### 任务 2：导入并运行 CD 管道

1. 转到 **“管道”>“管道”**。

1. 选择“**新建管道**”按钮。

1. 选择“Azure Repos Git (YAML)”。

1. 选择“eShopOnWeb”存储库。

1. 选择“现有 Azure Pipelines YAML 文件”。

1. 选择“**/.ado/eshoponweb-cd-webapp-code.yml**”文件，然后选择“**继续**”。

1. 在“变量”部分下的 YAML 管道定义中，自定义：

   - **AZ400-EWebShop-NAME** 为喜欢的名称，例如 **rg-eshoponweb-secure**。
   - 要部署资源的 Azure 区域的名称的**位置**，例如 **southcentralus**。
   - YOUR-SUBSCRIPTION-ID，替换为你的 Azure 订阅 ID。
   - 具有要部署的 Web 应用全局唯一名称的 **az400-webapp-NAME**，例如字符串 **eshoponweb-lab-secure-**，后跟一个随机六位数字。 

1. 选择“**保存并运行**”，然后选择直接提交到主分支。

1. 再次选择“**保存并运行**”。

1. 打开管道运行。 如果看到消息“此管道需要访问资源的权限，然后此运行才能继续部署到 WebApp”，请选择“**查看**”和“**允许**”，然后再次选择“**允许**”。 这是允许管道创建 Azure 应用服务资源所必需的。

   ![允许从 YAML 管道访问的屏幕截图。](media/pipeline-deploy-permit-resource.png)

1. 部署可能需要几分钟才能完成，请等待管道执行。 管道会在 CI 管道完成后触发，其中包括以下任务：

   - **AzureResourceManagerTemplateDeployment**：使用 bicep 模板部署 Azure 应用服务 Web 应用。
   - **AzureRmWebAppDeployment**：将网站发布到 Azure 应用服务 Web 应用。

1. 管道将采用基于项目名称的名称。 让我们重命名它，以便更好地识别管道。

1. 转到“**管道”>“管道**”，然后选择最近创建的管道。 选择省略号，然后选择“**重命名/移动**”选项。

1. 将其命名为 **eshoponweb-cd-webapp-code**，然后选择“**保存**”。

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
     templateFile: '.azure/bicep/webapp.bicep'
     subscriptionid: 'YOUR-SUBSCRIPTION-ID'
     azureserviceconnection: 'azure subs' #the name of the service connection to your Azure subscription
     webappname: 'eshoponweb-lab-secure-XXXXXX' #the globally unique name of the web app
   ```

   > [!IMPORTANT]
   > 将变量的值替换为环境的值（资源组、位置、订阅 ID、Azure 服务连接以及 Web 应用名称）。

1. 选择“**提交**”，在提交注释文本框中输入 `[skip ci]`，然后选择“**提交**”。

   > [!NOTE]
   > 通过向提交添加 `[skip ci]` 注释，将会阻止自动管道执行，此时，默认情况下此操作会在每次更改存储库后运行。 

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

1. 将以下内容添加到资源部分：

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

### 练习 2：执行 Azure 和 Azure DevOps 资源的清理

在本练习中，你将移除在此实验室中创建的 Azure 和 Azure DevOps 资源。

#### 任务 1：删除 Azure 资源

1. 在 Azure 门户中，导航到包含已部署资源的资源组 **rg-eshoponweb-secure**，然后选择“**删除资源组**”以删除在此实验室中创建的所有资源。

   ![“删除资源组”按钮的屏幕截图。](media/delete-resource-group.png)

   > [!WARNING]
   > 记得删除所有不再使用的已创建的 Azure 资源。 删除未使用的资源可确保不会出现意外费用。

#### 任务 2：移除 Azure DevOps 管道

1. 导航到 Azure DevOps 门户 `https://dev.azure.com` 并打开你的组织。

1. 打开 eShopOnWeb**** 项目。

1. 转到 **“管道”>“管道”**。

1. 转到“**管道”>“管道**”并删除现有管道。

#### 任务 3：重新创建 Azure DevOps 存储库

1. 在 Azure DevOps 门户中的 **eShopOnWeb** 项目中，选择左下角的“**项目设置**”。

1. 在左侧垂直菜单“**项目设置**”的“**Repos**”部分中，选择“**存储库**”。

1. 在“**所有存储库**”窗格中，将鼠标悬停在 **eShopOnWeb** 存储库条目的最右端，直到显示“**更多选项**”省略号图标，选择该图标，然后在“**更多选项**”菜单中，选择“**重命名**”。  

1. 在“**重命名 eShopOnWeb 存储库**”窗口中的“**存储库名称**”文本框中，输入 **eShopOnWeb_old**，然后选择“**重命名**”。

1. 返回“**所有存储库**”窗格，选择“**+ 创建**”。

1. 在“**创建存储库**”窗格的“**存储库名称**”文本框中，输入 **eShopOnWeb**，取消选中“**添加自述文件**”复选框，然后选择“**创建**”。

1. 返回到“**所有存储库**”窗格，将鼠标悬停在 **eShopOnWeb_old** 存储库条目的最右端，直到显示“**更多选项**”的省略号图标，选择它，然后在“**更多选项**”菜单中，选择“**删除**”。  

1. 在“**删除 eShopOnWeb_old 存储库**”窗口中，输入 **eShopOnWeb_old** 并选择“**删除**”。

1. 在 Azure DevOps 门户的左侧导航菜单中，选择“**Repos**”。

1. 在 **eShopOnWeb 中为空。** 添加一些代码!” 窗格中，选择“**导入存储库**”。

1. 在“导入 Git 存储库”**** 窗口上，粘贴以下 URL `https://github.com/MicrosoftLearning/eShopOnWeb` 并选择“导入”****：

## 审阅

在本实验室中，你已了解如何在 Azure DevOps 中配置和组织安全项目与存储库结构。 通过有效管理权限，可以确保适当的用户有权访问所需的资源，同时维护 DevOps 管道和过程的安全性和完整性。
