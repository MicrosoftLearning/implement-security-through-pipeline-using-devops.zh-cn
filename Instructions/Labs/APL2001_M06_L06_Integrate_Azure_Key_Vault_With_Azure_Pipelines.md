---
lab:
  title: 将 Azure Key Vault 与 Azure Pipelines 集成
  module: 'Module 6: Configure secure access to Azure Repos from pipelines'
---

# 将 Azure Key Vault 与 Azure Pipelines 集成

Azure Key Vault 可安全存储和管理敏感数据，例如密钥、密码和证书。 Azure Key Vault 支持硬件安全模块以及各种加密算法和密钥长度。 通过使用 Azure 密钥保管库，可最大程度地降低通过源代码泄漏敏感数据的可能性，而这是开发人员的一个常见错误。 访问 Azure Key Vault 需要正确的身份验证和授权，从而支持对其内容进行细化的权限管理。

此练习大约需要 30 分钟。

## 准备工作

需要 Azure 订阅、Azure DevOps 组织和 eShopOnWeb 应用程序才能遵循实验室。

- 按照步骤 [验证实验室环境](APL2001_M00_Validate_Lab_Environment.md)。

## 说明

在本实验室中，你将了解如何使用以下步骤将 Azure 密钥保管库与 Azure Pipelines 集成：

- 创建 Azure Key Vault 以将 ACR 密码存储为机密。
- 创建 Azure 服务主体以访问 Azure Key Vault 的机密。
- 配置权限以允许服务主体读取机密。
- 配置管道，以从 Azure Key Vault 检索密码并将其传递到后续任务。

### 练习 1：设置 CI 管道以生成 eShopOnWeb 容器

设置 CI YAML 管道，用于：

- 创建 Azure 容器注册表以保留容器映像
- 使用 Docker Compose 生成和推送 eshoppublicapi 和 eshopwebmvc 容器映像。 将仅部署 eshopwebmvc 容器。

#### 任务 1：（如果已完成，请跳过此任务）创建服务主体

在此任务中，你将使用 Azure CLI 创建服务主体，这将允许 Azure DevOps：

- 在 Azure 订阅上部署资源。
- 对稍后创建的密钥保管库机密具有读取访问权限。

你需要一个服务主体从 Azure Pipelines 部署 Azure 资源。 由于我们要检索管道中的机密，因此创建 Azure 密钥保管库时，我们需要为该服务授予权限。

从管道定义内部连接到 Azure 订阅或从项目设置页面（自动选项）新建服务连接时，Azure Pipeline 会自动创建服务主体。 你也可以从门户或使用 Azure CLI 手动创建服务主体，然后在项目中重复使用。

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

1. 在 Bash 提示符的 Cloud Shell 窗格中，运行以下命令以创建服务主体（将 myServicePrincipalName 替换为任意由字母和数字组成的唯一字符串）和 mySubscriptionID（替换为你的 Azure subscriptionId）：

    ```bash
    az ad sp create-for-rbac --name myServicePrincipalName \
                         --role contributor \
                         --scopes /subscriptions/mySubscriptionID
    ```

    > [!NOTE]
    > 注意：此命令将生成 JSON 输出。 将输出复制到文本文件中。 本实验室中稍后会用到它。

1. 接下来，导航到 Azure DevOps 门户 `https://dev.azure.com` 并打开组织。

1. 从 Azure DevOps 门户导航到 EShopOnWeb 项目。 单击“项目设置 > 服务连接”（在“管道”下）和“新建服务连接”。

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

#### 任务 2：设置和运行 CI 管道

在此任务中，你将导入现有的 CI YAML 管道定义、修改并运行它。 它将创建新的 Azure 容器注册表 (ACR) 并生成/发布 eShopOnWeb 容器映像。

1. 导航到 Azure DevOps 门户 `https://dev.azure.com` 并打开组织。

1. 从 Azure DevOps 门户导航到 EShopOnWeb 项目。 转到“管道 > 管道”，然后单击“创建管道”（或“新建管道”）。

1. 在“你的代码在哪里?”窗口中，选择“Azure Repos Git (YAML)”并选择“eShopOnWeb”存储库。

1. 在“配置”部分，选择“现有 Azure Pipelines YAML 文件”。 提供以下路径 /.ado/eshoponweb-ci-dockercompose.yml，然后单击“继续”。

    ![YAML 文件中管道选择的屏幕截图。](media/select-ci-container-compose.png)

1. 在 YAML 管道定义中的变量部分中，通过将 AZ400-EWebShop-NAME 替换为**首选项的名称来自定义资源组名称，**例如 rg-eshoponweb**，将 YOUR-SUBSCRIPTION-ID** 替换为**自己的 Azure subscriptionId，并选择最靠近位置的位置，例如 **southcentralus**。**

1. （可选）可以使用在上一实验室中创建的自承载代理，将当前设置为 Microsoft 托管的代理的池名称更新为所创建的 **代理池的名称 eShopOnWebSelfPool**。

    不使用：

    ```YAML
      - job: Build
        pool:
          vmImage: ubuntu-latest
    
    ```

    使用：

    ```YAML
      - job: Build
        pool: eShopOnWebSelfPool
    
    ```

    > [!NOTE]
    > 若要使用自承载代理运行管道，需要运行代理并安装所有必备组件，例如 Visual Studio 来生成解决方案。 如果没有安装必备组件，可以使用 Microsoft 托管的代理。

1. 选择“保存”以直接提交到主分支，或者为此提交创建新分支。

1. 再次选择“保存并运行”  。

    > [!NOTE]
    > 如果选择创建新分支，则需要创建一个拉取请求以将更改合并到主分支。

1. 打开管道。 重要说明：如果看到消息“此管道需要访问资源的权限，然后才能继续运行 Docker Compose to ACI”，请再次单击“查看”、“允许”和“允许”。 这是允许管道创建资源所必需的。

    ![YAML 管道中允许访问的屏幕截图。](media/pipeline-permit-resource.png)

1. 生成可能需要几分钟才能完成，等待管道执行。 生成定义由以下任务构成：
      - AzureResourceManagerTemplateDeployment 使用 bicep 部署 Azure 容器注册表。
      - PowerShell 任务获取 bicep 输出（ACR 登录服务器）并创建管道变量。
      - DockerCompose 任务生成 eShopOnWeb 的容器映像并将其推送到 Azure 容器注册表。

1. 管道将采用基于项目名称的名称。 让我们重命名它，以便更好地识别管道。

1. 转到 **最近创建的管道上的管道>管道** ，将鼠标悬停在执行的管道上，然后选择省略号和 **“重命名/移动** ”选项。

1. 将其命名为 eshoponweb-ci-dockercompose，然后单击“保存”。

1. 执行完成后，在 Azure 门户中打开之前定义的资源组，并选择表示管道部署Azure 容器注册表（ACR）的条目。

    > [!NOTE]
    > 若要在注册表中查看存储库，需要分配提供此类访问权限的角色。 将用于此目的的 AcrPull 角色。

1. 在“访问控制(IAM)”页上，选择“+ 添加”，然后从下拉菜单中选择“添加角色分配”************。

1. 在 **“添加角色分配 **”页的**“角色**”选项卡上，选择 **“AcrPull**”，然后选择“**下一步**”。

1. 在 **“成员** ”选项卡上，单击“ **+ 选择成员**”，选择用户帐户，单击“ **选择**”，然后选择“ **下一步**”。

1. 选择“ **查看 + 分配** ”，并在分配成功完成后刷新浏览器页面。

1. 返回“容器注册表”页，在左侧的垂直菜单栏中，在 **“服务** ”部分中，选择“ **存储库**”。

1. 验证注册表是否包含映像 **eshoppublicapi** 和 **eshopwebmvc**。 仅在部署阶段使用 eshopwebmvc。

    ![Azure 门户中“创建新容器”页的屏幕截图。](media/azure-container-registry.png)

1. 单击“访问密钥”并复制密码值，该值将用于以下任务，因为我们会在 Azure 密钥保管库中将其保留为机密。

    ![访问密钥设置中 ACR 密码的屏幕截图。](media/acr-password.png)

#### 任务 3：创建 Azure 密钥保管库

在本任务中，你将使用 Azure 门户创建 Azure 密钥保管库。

对于本实验室场景，我们将有一个 Azure 容器实例 (ACI)，用于拉取并运行存储在 Azure 容器注册表 (ACR) 中的容器映像。 我们打算将 ACR 的密码作为机密存储到密钥保管库中。

1. 在 Azure 门户中的“搜索资源、服务和文档”文本框中，键入“密钥保管库”，然后按 Enter 键。

1. 选择“密钥保管库”边栏选项卡，单击“创建 > 密钥保管库”。

1. 在“创建密钥保管库”边栏选项卡的“基本信息”选项卡中，指定以下设置，然后单击“下一步”：

    | 设置 | 值 |
    | --- | --- |
    | 订阅 | 你在此实验室中使用的 Azure 订阅的名称 |
    | 资源组 | 资源组名称 **rg-eshoponweb** |
    | 密钥保管库名称 | 任何唯一的有效名称，如 ewebshop-kv-NAME（替换 NAME） |
    | 区域 | 靠近实验室环境位置的 Azure 区域 |
    | 定价层 | **标准** |
    | 保留已删除保管库的天数 | **7** |
    | 清除保护 | **禁用清除保护** |

1. **在“创建密钥保管库 **”边栏选项卡的**“访问配置**”选项卡上的 **“权限模型**”部分中，选择“**保管库访问策略**”。 

1. 在 **“访问策略** ”部分中，选择“ **+ 创建** ”以设置新策略。

    > **注意**：你需要保护对密钥保管库的访问，只允许得到授权的应用程序和用户进行访问。 若要从保管库访问数据，你需要提供对先前创建的服务主体的读取（获取/列出）权限，以便在管道中进行身份验证。

    - 在“权限”边栏选项卡上，选中“机密权限”下的“获取”和“列出”权限。 选择**下一步**。
    - 在“主体”边栏选项卡上，使用给定的“ID”或“名称”搜索以前创建的服务主体。 再次选择**下一步**和**下一步**。
    - 在“查看 + 创建”边栏选项卡上，选择“创建” 。

1. 返回到“创建密钥保管库”边栏选项卡，单击“查看 + 创建”>“创建”

    > [!NOTE]
    > 注意：等待预配 Azure 密钥保管库。 此过程应该会在 1 分钟内完成。

1. 在“部署完成”面板上，选择“转到资源”。

1. 在“Azure 密钥保管库”边栏选项卡左侧垂直菜单中的“对象”部分，单击“机密”********。

1. 在“机密”窗格中，选择“+ 生成/导入” 。

1. 在“创建机密”边栏选项卡上，指定以下设置并单击“创建”（将其他设置保留为默认值）：

    | 设置 | 值 |
    | --- | --- |
    | 上传选项 | **手动** |
    | 名称 | acr-secret |
    | 值 | 在上一任务中复制的 ACR 访问密码 |

#### 任务 3：创建连接到 Azure 密钥保管库的变量组

在此任务中，你将在 Azure DevOps 中创建一个变量组，该变量组将使用服务连接（服务主体）从密钥保管库中检索 ACR 密码机密

1. 导航到 Azure DevOps 门户 `https://dev.azure.com` 并打开组织。

1. 从 Azure DevOps 门户导航到 EShopOnWeb 项目。

1. 在 Azure DevOps 门户的垂直导航窗格中，选择“管道 > 库”。 选择“+ Variable group”。

1. 在“新建变量组”边栏选项卡上，指定以下设置：

    | 设置 | 值 |
    | --- | --- |
    | 变量组名称 | eshopweb-vg |
    | 从 Azure KV 链接机密... | **enable** |
    | Azure 订阅 | 可用 Azure 服务连接 > Azure 订阅 |
    | 密钥保管库名称 | 你的密钥保管库名称|

1. 在“变量”下，单击“+ 添加”，然后选择 acr-secret 机密。 选择“确定”。

1. 选择“保存”。

    ![创建变量组的屏幕截图。](media/vg-create.png)

#### 任务 4：设置 CD 管道以在 Azure 容器实例 (ACI) 中部署容器

在此任务中，你将导入一个 CD 管道，对其进行自定义并运行，以部署之前在 Azure 容器实例中创建的容器映像。

1. 导航到 Azure DevOps 门户 `https://dev.azure.com` 并打开组织。

1. 从 Azure DevOps 门户导航到 EShopOnWeb 项目。 转到“管道”，然后选择“新建管道”。

1. 在“你的代码在哪里?”窗口中，选择“Azure Repos Git (YAML)”并选择“eShopOnWeb”存储库。

1. 在“配置”部分，选择“现有 Azure Pipelines YAML 文件”。 提供以下路径 /.ado/eshoponweb-cd-aci.yml，然后单击“继续”。

1. 在 YAML 管道定义中，自定义：

    - YOUR-SUBSCRIPTION-ID，替换为你的 Azure 订阅 ID。
    - az400eshop-NAME，替换 NAME 使其全局唯一。
    - YOUR-ACR.azurecr.io 和 ACR-USERNAME 与 ACR 登录服务器（这两者都需要 ACR 名称，可以在“ACR > 访问密钥”上查看）。
    - rg-az400-container-NAME，其中包含之前在实验室中定义的资源组名称。

1. 单击“保存并运行”，等待管道成功执行。

    > 注意：部署可能需要几分钟才能完成。 CD 定义由以下任务构成：
    - 资源：已准备好根据 CI 管道完成自动触发。 它还会下载 bicep 文件的存储库。
    - 变量（对于部署阶段）连接到变量组，以使用 Azure 密钥保管库机密 acr-secret
    - AzureResourceManagerTemplateDeployment 使用 bicep 模板部署 Azure 容器实例 (ACI)，并提供 ACR 登录参数以允许 ACI 从 Azure 容器注册表 (ACR) 下载以前创建的容器映像。

1. 管道将采用基于项目名称的名称。 让我们重命名它，以便更好地识别管道。

1. 转到**管道>管道**，选择最近创建的管道，选择省略号，然后选择“重命名/移动 **”** 选项。

1. 将其命名为 eshoponweb-cd-aci，然后单击“保存”。

### 练习 2：删除 Azure 实验室资源

1. 在 Azure 门户中，打开创建的资源组，然后单击“删除资源组”。

    ![“删除资源组”按钮的屏幕截图。](media/delete-resource-group.png)

    > [!WARNING]
    > 记得删除所有不再使用的新建 Azure 资源。 删除未使用的资源可确保不会出现意外费用。

## 审阅

在本实验室中，你已使用以下步骤将 Azure Key Vault 与 Azure DevOps 管道集成：

- 创建了一个 Azure 服务主体，用于提供对 Azure 密钥保管库机密的访问权限，并验证从 Azure DevOps 到 Azure 的部署。
- 运行从 Git 存储库导入的 2 个 YAML 管道。
- 配置了管道，以使用 ADO 变量组从 Azure 密钥保管库检索密码并将其用于后续任务。
