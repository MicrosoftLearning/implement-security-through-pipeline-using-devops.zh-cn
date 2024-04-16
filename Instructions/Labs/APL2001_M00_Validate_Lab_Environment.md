---
lab:
  title: 验证实验室环境
  module: 'Module 0: Welcome'
---

# 验证实验室环境

为了为实验室做好准备，必须正确设置好你的环境。 本页将指导你完成设置过程，以确保满足所有先决条件。

- 此实验室需要 Microsoft Edge**** 或支持 Azure DevOps 的浏览器[](https://learn.microsoft.com/azure/devops/server/compatibility?view=azure-devops#web-portal-supported-browsers)。

- **设置 Azure 订阅：** 如果还没有 Azure 订阅，请按照此页面上的说明创建一个订阅，或访问 [https://azure.microsoft.com/free](https://azure.microsoft.com/free) 免费注册。

- **设置 Azure DevOps 组织：** 如果还没有可用于本实验室的 Azure DevOps 组织，请按照[创建组织或项目集合](https://learn.microsoft.com/azure/devops/organizations/accounts/create-organization)中的说明创建一个。
  
- [适用于 Windows 的 Git 下载页面](https://gitforwindows.org/)。 此应用程序将作为本实验室先决条件安装。

- [Visual Studio Code](https://code.visualstudio.com/)。 此应用程序将作为本实验室先决条件安装。

- [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli)。 在自托管代理计算机上安装 Azure CLI。

## 创建 Azure DevOps 组织的说明（只需执行此操作一次）

> **注意**：如果已有“个人 Microsoft 帐户”**** 设置和与该帐户相关联的有效 Azure 订阅，请从步骤 3 开始。

1. 使用专用浏览器会话从以下链接获取新的“个人 Microsoft 帐户(MSA)”：`https://account.microsoft.com`。

1. 使用相同的浏览器会话，在 `https://azure.microsoft.com/free` 注册一个免费的 Azure 订阅。

1. 打开浏览器并导航到位于 `https://portal.azure.com` 的 Azure 门户，然后在 Azure 门户屏幕顶部搜索“Azure DevOps”****。 在结果页面中，单击“Azure DevOps 组织”。

1. 接下来，单击标记为“我的 Azure DevOps 组织”的链接或者直接导航到 `https://aex.dev.azure.com`。

1. 在“我们需要更多详细信息”页上，选择“继续” 。

1. 在左侧下拉框中，选择“默认目录”****，而不是“Microsoft 帐户”****。

1. 如果系统提示“我们需要更多详细信息”，请提供你的姓名、电子邮件地址以及位置，然后单击“继续”。

1. 回到 `https://aex.dev.azure.com` 并选择“默认目录”，单击蓝色按钮“创建新组织” 。

1. 单击“继续”即表示接受“服务条款”。

1. 如果系统提示“即将完成”，请将 Azure DevOps 组织名称保留为默认值（该名称需要是全局唯一名称），并在列表中选择靠近你的托管位置。

1. 在 Azure DevOps**** 中打开新创建的组织后，选择左下角的“组织设置”****。

1. 在“组织设置”**** 屏幕上选择“账单”****（打开此屏幕需要几秒钟时间）。

1. 选择“设置账单”****，并在屏幕右侧选择你的 Azure 订阅****，然后选择“保存”**** 以将该订阅与组织关联起来。

1. 当屏幕顶部显示链接的 Azure 订阅 ID 时，将 MS 托管 CI/CD 的付费并行作业数量从 0 更改为 1  。 然后选择底部的“保存”**** 按钮。

1. 最好至少等待 3 个小时再使用 CI/CD 功能****，以使新的设置在后端反映出来。 否则，仍会看到消息“未购买或授予托管并行”。

## 创建和配置 Azure DevOps 项目的说明（只需执行一次）

> 注意：在继续执行这些步骤之前，请确保已完成创建 Azure DevOps 组织的步骤。

为了遵循所有实验室说明，你需要设置一个新的 Azure DevOps 项目，创建基于 [eShopOnWeb](https://github.com/MicrosoftLearning/eShopOnWeb) 应用程序的存储库，并创建与你的 Azure 订阅的服务连接。

### 创建团队项目

首先你将创建一个 eShopOnWeb**** Azure DevOps 项目，供多个实验室使用。

1. 打开浏览器，并导航到你的 Azure DevOps 组织。

1. 选择“新建项目”**** 选项并使用以下设置：
   - 名称：eShopOnWeb
   - 可见性：专用
   - 高级：版本控制：Git
   - 高级：工作项流程：Scrum

1. 选择**创建**。

   ![创建项目](media/create-project.png)

### 导入 eShopOnWeb Git 存储库

现在需要将 eShopOnWeb 导入 Git 存储库。

1. 打开浏览器，并导航到你的 Azure DevOps 组织。

1. 打开以前创建的 eShopOnWeb**** 项目。

1. 依次选择“Repos”>“文件”****、“导入存储库”****，然后选择“导入”****。

1. 在“导入 Git 存储库”**** 窗口上，粘贴以下 URL `https://github.com/MicrosoftLearning/eShopOnWeb` 并选择“导入”****：

   ![导入存储库](media/import-repo.png)

1. 存储库按以下方式组织：

   - .ado 文件夹包含 Azure DevOps YAML 管道。
   - 设置 .devcontainer 文件夹容器，使用容器（在 VS Code 或 GitHub Codespaces 中本地进行）开发。
   - .Azure**** 文件夹包含 Bicep 和 ARM 基础结构即代码模板。
   - .github 文件夹容器 YAML GitHub 工作流定义。
   - src 文件夹包含用于实验室方案的 .NET 6 网站。 

1. 保持 Web 浏览器窗口处于打开状态。  

### 创建服务主体和服务连接以访问 Azure 资源

接下来，你将使用 Azure CLI 创建服务主体，并在 Azure DevOps 中创建服务连接，以便部署和访问你的 Azure 订阅中的资源。

1. 启动 Web 浏览器，导航到 `https://portal.azure.com` 处的 Azure 门户，使用用户帐户登录，该帐户须对你在本课程的实验室中使用的 Azure 订阅具有所有者角色，并在与此订阅关联的 Microsoft Entra 租户中具有全局管理员角色。

1. 在 Azure 门户中，选择页面顶部搜索文本框右侧的 **Cloud Shell** 图标。

1. 如果系统提示选择“Bash”或“PowerShell”，请选择“Bash”。

   > [!NOTE]
   > 如果这是你第一次启动 **Cloud Shell**，并看到“**未装载任何存储**”消息，请选择你在本实验室中使用的订阅，然后选择“**创建存储**”。

1. 在 Bash 提示符的 Cloud Shell 窗格中，运行以下命令以检索 Azure 订阅 ID 和订阅名称属性的值 ：

   ```bash
   subscriptionName=$(az account show --query name --output tsv)
   subscriptionId=$(az account show --query id --output tsv)
   echo $subscriptionName
   echo $subscriptionId
   ```

   > [!NOTE]
   > 将两个值都复制到文本文件中。 本课程的实验室需要用到它们。

1. 在 **Bash** 提示符的 **Cloud Shell** 窗格中，运行以下命令以创建服务主体：

   ```bash
   az ad sp create-for-rbac --name sp-eshoponweb-azdo --role contributor --scopes /subscriptions/$subscriptionId
   ```

   > [!NOTE]
   > 此命令将生成 JSON 输出。 将输出复制到文本文件中。 稍后需要它。

   > [!NOTE]
   > 记录 JSON 输出中包含的安全主体名称、其 ID 和租户 ID 的值。 本课程的实验室需要用到它们。

1. 切换回显示 Azure DevOps 门户的 Web 浏览器窗口，其中打开了 **eShopOnWeb** 项目，然后选择门户左下角的**项目设置**。

1. 选择“管道”下的“**服务连接**”，然后选择“**创建服务连接**”按钮。

   ![新建服务连接按钮的屏幕截图。](media/new-service-connection.png)

1. 在“新建服务连接”边栏选项卡上，选择“Azure 资源管理器”和“下一步”（可能需要向下滚动）。

1. 然后选择“**服务主体(手动)**，然后选择“**下一步**”。

1. 使用在前面的步骤中收集的信息填写空字段：

   - 订阅 ID 和名称。
   - 服务主体 ID（或 clientId/AppId）、服务主体密钥（或密码）和 TenantId。
   - 在“服务连接名称”中，键入 azure subs。 此名称将在 YAML 管道中引用，以引用服务连接，以便访问你的 Azure 订阅。

   ![Azure 服务连接配置的屏幕截图。](media/azure-service-connection.png)

1. 请勿勾选**为所有管道授予访问权限**。 选择**验证并保存**。

   > [!NOTE]
   > 不建议对生产环境使用**为所有管道授予访问权限**选项。 它仅在此实验室中用于简化管道的配置。

现在，你已完成继续操作本实验室所需的先决条件步骤。
