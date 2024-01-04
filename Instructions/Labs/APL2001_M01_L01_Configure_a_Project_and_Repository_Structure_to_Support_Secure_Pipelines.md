---
lab:
  title: 配置项目和存储库结构以支持安全管道
  module: 'Module 1: Configure a project and repository structure to support secure pipelines'
---

# 配置项目和存储库结构以支持安全管道

在本实验室中，了解如何在 Azure DevOps 中配置项目和存储库结构，以支持安全管道。 本实验室介绍组织项目和存储库、分配权限以及管理安全文件的最佳做法。

此练习大约需要 30 分钟。

## 准备工作

需要 Azure 订阅、Azure DevOps 组织和 eShopOnWeb 应用程序才能遵循实验室。

- 按照步骤 [验证实验室环境](APL2001_M00_Validate_Lab_Environment.md)。

## 说明

### 练习 1：配置安全项目结构

在本练习中，你将通过创建新项目并为其分配项目权限来配置安全项目结构。 将职责和资源分成具有特定权限的不同项目或存储库支持安全性。

#### 任务 1：创建新项目

1. 导航到 Azure DevOps 门户 `https://dev.azure.com` 并打开组织。

1. **打开门户左下角的组织设置**，然后在 **“常规”部分下打开“项目**”。

1. **选择“新建项目**”选项并使用以下设置：
   - 名称： **eShopSecurity**
   - 可见性：专用
   - 高级：版本控制：Git
   - 高级：工作项流程：Scrum

    ![具有指定设置的新项目对话框的屏幕截图。](media/new-team-project.png)

1. 选择“创建”以创建新项目。

1. 现在，可以通过单击 Azure DevOps 界面左上角的“项目”下拉菜单在不同的团队项目之间切换。

    ![Azure DevOps 团队项目 eShopOnWeb 和 eShopSecurity 的屏幕截图。](media/azure-devops-projects.png)

可以通过转到“项目设置”菜单并选择相应的团队项目，单独管理每个团队项目的权限和设置。 如果有多个用户或团队处理不同的项目，则还可以单独为每个项目分配权限。

#### 任务 2：创建新的存储库并分配项目权限

1. 选择 Azure DevOps 门户左上角的组织名称，然后选择新的 **eShopSecurity** 项目。

1. 选择 **Repos** 菜单项。

1. 选择“ **初始化** ”按钮，通过添加 README.md 文件来初始化新存储库。

1. 打开 **门户左下角的“项目设置** ”菜单，然后选择“存储库” **部分下的“存储库** ”。

1. 选择新的 **eShopSecurity** 存储库，然后选择“ **安全** ”选项卡。

1. 取消检查 **“继承”切换按钮，从父级删除“继承**”权限。

1. 选择 **“参与者**”组，并为除“读取 **”以外的**所有权限选择“**拒绝**”下拉列表。 这将阻止来自参与者组的所有用户访问存储库。

1. 在“用户”下选择用户，然后选择 **“允许** ”按钮以允许所有权限。

    ![存储库安全设置的屏幕截图，允许读取和拒绝所有其他权限。](media/repository-security.png)

1. （可选）添加要授予对存储库的访问权限的特定用户组或用户，并从 eShopOnWeb 项目运行管道。 单击搜索框，输入组的名称，将其选中，然后设置要允许或拒绝组或用户的权限。

    > [!NOTE]
    > 确保 eShopOnWeb 项目中具有相同的组。 这样，就可以从 eShopOnWeb 项目运行管道，并访问 eShopSecurity 项目中的存储库。

1. 更改将自动保存。

现在，只有你分配了权限的用户，管理员才能访问存储库。 如果要允许特定用户从 eShopOnWeb 项目访问存储库并运行管道，这非常有用。

### 练习 2：配置管道和模板结构以支持安全管道

#### 任务 1：导入并运行 CI 管道

1. 导航到 Azure DevOps 门户 `https://dev.azure.com` 并打开组织。

1. 在 **Azure DevOps 中打开 eShopOnWeb** 项目。

1. 转到“管道”>“管道”。

1. 选择“创建管道”按钮****。

1. 选择“Azure Repos Git (YAML)”。

1. 选择“eShopOnWeb”存储库。

1. 选择“现有 Azure Pipelines YAML 文件”。

1. 选择“/.ado/eshoponweb-ci.yml”文件，然后单击“继续”。

1. 单击“运行”按钮以运行管道。

1. 管道将采用基于项目名称的名称。 让我们重命名它，以便更好地识别管道。

1. 转到“管道 > 管道”，然后单击最近创建的管道。 选择省略号 (...)，然后选择重命名。

1. 将其命名为 eshoponweb-ci，然后单击“保存”。

#### 创建服务主体并配置其对 Azure 资源的访问权限。

在此任务中，你将使用 Azure CLI 创建服务主体，这将允许 Azure DevOps：

1. 从实验室计算机启动 Web 浏览器，导航到 Azure 门户，并使用用户帐户登录，该帐户在本实验室中将使用的 Azure 订阅中具有所有者角色，并在与此订阅关联的 Azure AD 租户中具有全局管理员角色。

1. 在 Azure 门户中，单击页面顶部搜索文本框右侧的 Cloud Shell 图标。

1. 如果系统提示选择“Bash”或“PowerShell”，请选择“Bash”。

   > [!NOTE]
   > 注意：如果这是第一次启动 Cloud Shell，并看到“未装载任何存储”消息，请选择在本实验室中使用的订阅，然后选择“创建存储”  。

1. 在 Bash 提示符的 Cloud Shell 窗格中，运行以下命令以检索 Azure 订阅 ID 和订阅名称属性的值 ：

    ```bash
    az account show --query id --output tsv
    az account show --query name --output tsv
    ```

    > [!NOTE]
    > 注意：将两个值都复制到文本文件中。 稍后将在本实验室用到它们。

1. 在 Bash 提示符的 Cloud Shell 窗格中，运行以下命令以创建服务主体：

    ```bash
    az ad sp create-for-rbac --name myServicePrincipalName \
                         --role contributor \
                         --scopes /subscriptions/mySubscriptionID
    ```

    > [!NOTE]
    > 将 **myServicePrincipalName** 替换为包含字母和数字（例如 **AzureDevOpsSP**）的任何唯一字符字符串，并将 **mySubscriptionID** 替换为 Azure subscriptionId。

    > [!NOTE]
    > 注意：此命令将生成 JSON 输出。 将输出复制到文本文件中。 本实验室中稍后会用到它。

1. 接下来，导航到 Azure DevOps 门户 `https://dev.azure.com` 并打开组织。

1. **打开 eShopOnWeb** 项目，然后选择**门户左下角的项目设置**。

1. 在“管道”下，选择“服务连接”，然后选择“创建服务连接”。

    ![创建连接按钮屏幕截图。](media/new-service-connection.png)

1. 在“新建服务连接”边栏选项卡上，选择“Azure 资源管理器”和“下一步”（可能需要向下滚动）。

1. 选择“**服务主体(手动)**”，然后选择“**下一步**”。

1. 使用在前面的步骤中收集的信息填写空字段：
    - 订阅 ID 和名称。
    - 服务主体 ID（或 clientId）、密钥（或密码）和 TenantId。
    - 在“服务连接名称”中，键入 azure subs。 需要 Azure DevOps 服务连接来与 Azure 订阅通信时，将在 YAML 管道中引用此名称。

        ![连接服务配置的屏幕截图。](media/azure-service-connection.png)

1. 授予对所有管道的访问权限。 选择**验证并保存**。

    > [!NOTE]
    > **不建议对生产环境授予对所有管道**的访问权限选项。 它仅用于此实验室来简化管道的配置。

#### 任务 3：导入并运行 CD 管道

1. 转到“管道”>“管道”。

1. 选择“ **新建管道** ”按钮。

1. 选择“Azure Repos Git (YAML)”。

1. 选择“eShopOnWeb”存储库。

1. 选择“现有 Azure Pipelines YAML 文件”。

1. 选择“/.ado/eshoponweb-cd-webapp-code.yml”文件，然后单击“继续”。

1. 在变量部分下的 YAML 管道定义中，自定义：
   - **AZ400-EWebShop-NAME** ，其首选项的名称， **例如 rg-eshoponweb-secure**。
   - **要部署资源的 Azure 区域的名称的位置** ， **例如 southcentralus**。
   - YOUR-SUBSCRIPTION-ID，替换为你的 Azure 订阅 ID。
   - **az400eshop-NAME**，具有要部署的 Web 应用名称具有全局唯一名称， **例如 eshoponweb-lab-secure**。

1. 选择“保存”以直接提交到主分支，或者为此提交创建新分支。

1. 再次选择“保存并运行”  。

    > [!NOTE]
    > 如果选择创建新分支，则需要创建一个拉取请求以将更改合并到主分支。

1. 打开管道。 重要说明：如果看到消息“此管道需要访问资源的权限，然后才能继续运行 Docker Compose to ACI”，请再次单击“查看”、“允许”和“允许”。 这是允许管道创建资源所必需的。

    ![YAML 管道中允许访问的屏幕截图。](media/pipeline-deploy-permit-resource.png)

1. 部署可能需要几分钟才能完成，等待管道执行。 CD 定义由以下任务构成：
      - 资源：它已准备好根据 CI 管道完成自动触发。 它还会下载 bicep 文件的存储库。
      - AzureResourceManagerTemplateDeployment：使用 bicep 模板部署 Azure Web 应用。
1. 管道将采用基于项目名称的名称。 让我们重命名它，以便更好地识别管道。

1. 转到“管道 > 管道”，然后单击最近创建的管道。 选择省略号 (...)，然后选择重命名。

1. 将其命名为 eshoponweb-cd-webapp-code，然后单击“保存”。

现在，应在 eShopOnWeb 项目中运行两个管道。

![成功执行的 CI/CD 管道的屏幕截图。](media/pipeline-successful-executed.png)

#### 任务 4：将 CD 管道变量移动到 YAML 模板

在此任务中，你将创建一个 YAML 模板来存储 CD 管道中使用的变量。 这样，就可以在其他管道中重复使用模板。

1. 转到 Repos **，然后**转到**“文件**”。

1. 展开 **.ado** 文件夹并选择“  **新建文件**”。

1. 将文件 **命名为 eshoponweb-secure-variables.yml** ，然后选择“ **创建**”。

1. 将 CD 管道中使用的 variables 节添加到新文件。 该文件应如下所示：

    ```YAML
    variables:
      resource-group: 'rg-eshoponweb-secure'
      location: 'southcentralus' #name of the Azure region you want to deploy your resources
      templateFile: '.azure/bicep/webapp.bicep'
      subscriptionid: 'YOUR-SUBSCRIPTION-ID'
      azureserviceconnection: 'YOUR-AZURE-SERVICE-CONNECTION-NAME'
      webappname: 'eshoponweb-lab-secure'

    ```

    > [!IMPORTANT]
    > 将变量的值替换为环境的值（资源组、位置、订阅 ID、Azure 服务连接和 Web 应用名称）。

1. 选择“**提交**”，添加批注，然后选择“提交 **”** 按钮。

1. **打开 eshoponweb-cd-webapp-code.yml** 管道定义，并将 variables 节替换为以下内容：

    ```YAML
    variables:
      - template: eshoponweb-secure-variables.yml
    ```

    > [!NOTE]
    > 如果使用模板文件的其他路径，则需要更新管道定义中的路径。

1. 选择  以保存并运行管道

现在，你有一个 YAML 模板，其中包含 CD 管道中使用的变量。 如果需要部署相同的资源，可以在其他管道中重复使用此模板。 此外，运营团队还可以控制部署资源的资源组和位置，以及模板值中的其他信息，无需对管道定义进行任何更改。

#### 任务 5：将 YAML 模板移动到单独的存储库和项目

在此任务中，你将 YAML 模板移动到单独的存储库和项目。

1. 在 eShopSecurity 项目中，转到 **Repos >文件**。

1. 创建名为 **eshoponweb-secure-variables.yml** 的新文件。

1. 将文件 **.ado/eshoponweb-secure-variables.yml** 的内容从 eShopOnWeb 存储库复制到新文件。

1. 提交更改。

1. **从 eShopOnWeb 项目打开 eshoponweb-cd-webapp-code.yml** 管道定义。

1. 将以下  添加到资源部分：

    ```YAML
    resources:
      repositories:
        - repository: eShopSecurity
          type: git
          name: eShopSecurity/eShopSecurity #name of the project and repository

    ```

1. 将 variables 节替换为以下内容。

    ```YAML
    variables:
      - template: eshoponweb-secure-variables.yml@eShopSecurity #name of the template and repository
    ```

    ![包含新变量和资源部分的管道定义的屏幕截图。](media/pipeline-variables-resource-section.png)

1. 选择  以保存并运行管道 你将看到管道正在使用 eShopSecurity 存储库中的 YAML 模板。

    ![使用 eShopSecurity 存储库中的 YAML 模板执行的管道执行的屏幕截图。](media/pipeline-execution-using-template.png)

现在，已在单独的存储库和项目中拥有 YAML 模板。 如果需要部署相同的资源，可以在其他管道中重复使用这些模板。 此外，运营团队还可以控制资源组、位置、安全性和部署资源的位置以及模板值中的其他信息，无需对管道定义进行任何更改。

### 练习 2：执行 Azure 和 Azure DevOps 资源的清理

在本练习中，你将删除在此实验室中创建的 Azure 和 Azure DevOps 资源。

#### 任务 1：删除 Azure 实验室资源

1. 在Azure 门户中，打开创建的资源组，并为本实验室中的所有已创建资源选择“**删除资源组**”。

    ![“删除资源组”按钮的屏幕截图。](media/delete-resource-group.png)

    > [!WARNING]
    > 记得删除所有不再使用的新建 Azure 资源。 删除未使用的资源可确保不会出现意外费用。

#### 取消了 Azure DevOps 管道

1. 导航到 Azure DevOps 门户 `https://dev.azure.com` 并打开组织。

1. 打开 **eShopOnWeb** 项目。

1. 转到“管道”>“管道”。

1. 转到 **管道>管道** 并删除现有管道。

## 审阅

在本实验室中，了解如何在 Azure DevOps 中配置项目和存储库结构，以支持安全管道。 通过有效管理权限，可以确保适当的用户有权访问所需的资源，同时维护 DevOps 管道和流程的安全性和完整性。
