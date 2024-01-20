---
lab:
  title: 配置管道以安全地使用变量和参数
  module: 'Module 7: Configure pipelines to securely use variables and parameters'
---

# 配置管道以安全地使用变量和参数

在本实验室中，你将了解如何配置管道，以安全地使用变量和参数。

此练习大约需要 **20** 分钟。

## 准备工作

需要 Azure 订阅、Azure DevOps 组织和 eShopOnWeb 应用程序才能遵循实验室。

- 按照步骤 [验证实验室环境](APL2001_M00_Validate_Lab_Environment.md)。

## 说明

### 练习 1：确保参数和变量类型

#### 任务 1：导入并运行 CI 管道

首先导入名为 [eshoponweb-ci.yml](https://github.com/MicrosoftLearning/eShopOnWeb/blob/main/.ado/eshoponweb-ci.yml) 的 CI 管道。

1. 导航到 Azure DevOps 门户 `https://dev.azure.com` 并打开你的组织。

1. 打开 eShopOnWeb 项目。

1. 转到 **“管道”>“管道”**。

1. 选择“Create Pipeline”。

1. 选择“Azure Repos Git (YAML)”。

1. 选择“eShopOnWeb”存储库。

1. 选择“现有 Azure Pipelines YAML 文件”。

1. 选择“**/.ado/eshoponweb-ci.yml**”文件，然后选择“**继续**”。

1. 选择“**运行**”按钮以运行管道。

   > [!NOTE]
   > 管道将采用基于项目名称的名称。 为了更好地识别该管道，请为其重命名。

1. 转到“**管道”>“管道**”，然后选择最近创建的管道。 选择省略号和“**重命名/移动**”选项。

1. 将其命名为 **eshoponweb-ci-parameters**，然后选择“**保存**”。

#### 任务 2：确保 YAML 管道的参数类型

在此任务中，你将为管道设置参数和参数类型。

1. 转到“**管道”>“管道**”并选择“**eshoponweb-ci-parameters**”管道。

1. 选择“编辑”。

1. 将以下参数和资源部分添加到 YAML 文件的顶部：

   ```yaml
   parameters:
   - name: dotNetProjects
     type: string
     default: '**/*.sln'
   - name: testProjects
     type: string
     default: 'tests/UnitTests/*.csproj'

   resources:
     repositories:
     - repository: self
       trigger: none

   ```

1. 将 `Restore`、`Build` 和 `Test` 任务中的硬编码路径替换为刚刚创建的参数。

   - **将项目**：`**/*.sln` 替换为 `Restore` 和 `Build` 任务中的项目 ${{ parameters.dotNetProjects }}。
   - **将项目**：`tests/UnitTests/*.csproj` 替换为 `Test` 任务中的项目 ${{ parameters.testProjects }}。

   YAML 文件步骤部分中的 `Restore`、`Build` 和 `Test` 任务外观应如下所示：

   ```yaml
       steps:
       - task: DotNetCoreCLI@2
         displayName: Restore
         inputs:
           command: 'restore'
           projects: ${{ parameters.dotNetProjects }}
           feedsToUse: 'select'
   
       - task: DotNetCoreCLI@2
         displayName: Build
         inputs:
           command: 'build'
           projects: ${{ parameters.dotNetProjects }}
   
       - task: DotNetCoreCLI@2
         displayName: Test
         inputs:
           command: 'test'
           projects: ${{ parameters.testProjects }}

   ```

1. 保存管道并运行它。 验证管道运行是否成功完成。

   ![带有参数的管道运行屏幕截图。](media/pipeline-parameters-run.png)

#### 任务 3：保护变量和参数

在此任务中，你将使用变量组保护管道中的变量和参数。

1. 转到“**管道”>“库**”。

1. 选择“**+ 变量组**”按钮以创建名为 **BuildConfigurations** 的新变量组。

1. 添加名为 **buildConfiguration** 的变量并将其值设置为 `Release`。

1. 保存变量组。

   ![包含 BuildConfigurations 的变量组的屏幕截图。](media/eshop-variable-group.png)

1. 选择“**管道权限**”按钮，然后选择“**+**”以添加新管道。

1. 选择 **eshoponweb-ci-parameters** 管道以允许管道使用变量组。

   ![管道权限的屏幕截图。](media/pipeline-permissions.png)

   > [!NOTE]
   > 还可以通过单击“**安全**”按钮设置能够编辑变量组的特定用户或组。

1. 转到 **“管道”>“管道”**。

1. 打开 **eshoponweb-ci-parameters** 管道，然后选择“**编辑**”。

1. 在 yml 文件顶部的参数下，通过添加以下内容来引用变量组：

   ```yaml
   variables:
     - group: BuildConfigurations
   ```

1. 在“生成”任务中，将命令：“build”替换为以下行，以利用变量组中的生成配置。

   ```yaml
           command: 'build'
           projects: ${{ parameters.dotNetProjects }}
           configuration: $(buildConfiguration)
   ```

1. 保存管道并运行它。 在将生成配置设置为 `Release` 的情况下，它应会成功运行。 可以通过查看“生成”任务的日志来验证这一点。

> [!NOTE]
> 按照此方法，可以通过使用变量组来保护变量和参数，而无需在 YAML 文件中对其进行硬编码。

#### 任务 4：验证强制变量和参数

在此任务中，你将在管道执行之前验证强制变量。

1. 转到 **“管道”>“管道”**。

1. 打开 **eshoponweb-ci-parameters** 管道，然后选择“**编辑**”。

1. 在阶段部分的开头（紧跟在 `stage:` 行后面），添加一个名为**验证**的新阶段，以在管道执行之前验证强制变量。

   ```yaml
   - stage: Validate
     displayName: Validate mandatory variables
     jobs:
     - job: ValidateVariables
       pool:
         vmImage: ubuntu-latest
       steps:
       - script: |
           if [ -z "$(buildConfiguration)" ]; then
             echo "Error: buildConfiguration variable is not set"
             exit 1
           fi
         displayName: 'Validate Variables'
    ```

   > [!NOTE]
   > 此阶段将会运行脚本来验证 buildConfiguration 变量。 如果未设置变量，则脚本将失败，并且管道将停止。

1. 通过在**生成**阶段的开头添加 `dependsOn: Validate`，将**生成**阶段设置为依赖于**验证**阶段：

   ```yaml
   - stage: Build
     displayName: Build .Net Core Solution
     dependsOn: Validate
      ```

1. 保存管道并运行它。 它将成功运行，因为 buildConfiguration 变量是在变量组中设置的。

1. 要测试验证，请从变量组中移除 buildConfiguration 变量，或删除变量组，然后再次运行管道。 它应会失败并出现以下错误：

   ```yaml
   Error: buildConfiguration variable is not set   
   ```

   ![管道运行的屏幕截图，其中显示了验证失败。](media/pipeline-validation-fail.png)

1. 将变量组和 buildConfiguration 变量添加回变量组，然后再次运行管道。 它应已成功运行。

## 审阅

在本实验室中，你已了解如何配置管道以安全地使用变量和参数，以及如何验证强制变量和参数。
