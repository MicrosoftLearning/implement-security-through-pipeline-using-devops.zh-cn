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

- [.NET SDK - 最新版本](https://dotnet.microsoft.com/download/visual-studio-sdks)。 在自托管代理计算机上安装 .NET SDK。

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

   > **注意**：**等待几分钟再使用 CI/CD 功能**，以使新的设置在后端反映出来。 否则，仍会看到消息“未购买或授予托管并行”。

1. 在“组织设置”中，转到“管道”部分并单击“设置”  。

1. 将“禁用经典生成管道的创建”和“禁用经典发布管道的创建”的开关切换到“关闭”

   > **注意**：如果将“**禁用经典发布管道的创建**”开关设置为“**开**”，会隐藏经典发布管道创建选项，例如 DevOps 项目“**管道**”部分中的“**发布**”菜单。

1. 在“组织设置”中，转到“安全性”部分并单击“策略”  。

1. 将“允许公共项目”的开关切换为“开启”

   > **注意**：某些实验室中使用的扩展可能需要公共项目才能使用免费版本。

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
   - src 文件夹包含用于实验室方案的 .NET 8 网站。****

1. 保持 Web 浏览器窗口处于打开状态。  

1. 转到“**Repos > 分支**”。

1. 将鼠标指针悬停在主分支上，然后单击列右侧的省略号。

1. 单击“设置为默认分支”。

### 创建服务连接以访问 Azure 资源

接下来，将在 Azure DevOps 中创建服务连接，以便部署和访问 Azure 订阅中的资源。

1. 启动 Web 浏览器，导航到 Azure DevOps 门户，打开 **eShopOnWeb** 项目，然后在门户左下角选择“**项目设置**”。

1. 选择“管道”下的“**服务连接**”，然后选择“**创建服务连接**”按钮。

   ![新建服务连接按钮的屏幕截图。](media/new-service-connection.png)

1. 在“新建服务连接”边栏选项卡上，选择“Azure 资源管理器”和“下一步”（可能需要向下滚动）。

1. 选择“**工作负荷联合身份验证（自动）**，然后选择“**下一步**”。

   > **注意**：如果希望手动配置服务连接，还可以使用“**工作负荷联合身份验证（手动）**”。 按照 [Azure DevOps 文档](https://learn.microsoft.com/azure/devops/pipelines/library/connect-to-azure)中的步骤手动创建服务连接。

1. 使用信息填写空字段：
    - **订阅**：选择 Azure 订阅。
    - **资源组**：使用要在其中部署资源的资源组。
    - **服务连接名称**：键入**`azure subs`**。 访问 Azure 订阅时，将在 YAML 管道中引用此名称。

1. 确保未选中“**向所有管道授予访问权限**”选项，然后选择“**保存**”。

   > **注意：** 不建议对生产环境使用“**向所有管道授予访问权限**”选项。 它仅在此实验室中用于简化管道的配置。

   > **注意**：如果看到一条错误消息，指示你没有创建服务连接所需的权限，请重试或手动配置服务连接。

现在，你已完成继续操作本实验室所需的先决条件步骤。
