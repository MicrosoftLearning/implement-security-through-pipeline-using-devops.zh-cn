---
lab:
  title: 配置管道以安全地使用变量和参数
  module: 'Module 7: Configure pipelines to securely use variables and parameters'
---

# 配置管道以安全地使用变量和参数

在本实验室中，了解如何配置管道，以安全地使用变量和参数。

此练习大约需要 30 分钟。

## 开始之前

需要 Azure 订阅、Azure DevOps 组织和 eShopOnWeb 应用程序才能遵循实验室。

- 按照步骤 [验证实验室环境](APL2001_M00_Validate_Lab_Environment.md)。

## 说明

### 确保参数和变量类型

#### 任务 1：导入并运行 CI 管道

让我们首先导入名为 [eshoponweb-ci.yml](https://github.com/MicrosoftLearning/eShopOnWeb/blob/main/.ado/eshoponweb-ci.yml) 的 CI 管道。

1. 导航到 Azure DevOps 门户 `https://dev.azure.com` 并打开组织。

1. 打开 eShopOnWeb 项目。

1. 转到“管道”>“管道”。

1. 选择“ **新建管道** ”按钮。

1. 选择“Azure Repos Git (YAML)”。

1. 选择“eShopOnWeb”存储库。

1. 选择“现有 Azure Pipelines YAML 文件”。

1. 选择“/.ado/eshoponweb-ci.yml”文件，然后单击“继续”。

1. 单击“运行”按钮以运行管道。

1. 管道将采用基于项目名称的名称。 让我们重命名它，以便更好地识别管道。

1. 转到“管道 > 管道”，然后单击最近创建的管道。 单击省略号和“重命名/移动”选项。

1. 将其 **命名为 eshoponweb-ci-parameters** ，然后选择“ **保存**”。

#### 任务 2：确保 YAML 管道的参数类型

在此任务中，你将为管道设置参数和参数类型。

1. 转到 **管道>管道** 并选择 **eshoponweb-ci-parameters** 管道。

1. 选择**编辑**。

1. 在 parameters 数组的顶部，添加以下内容：

    ```YAML
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

    stages:
    - stage: Build
      displayName: Build .Net Core Solution
    ```

1. 将“还原”、“生成”和“测试”任务中的硬编码路径替换为刚刚创建的参数。
   - **将项目**“**/*.sln”替换为项目：“还原”和“生成”任务中的 ${{ parameters.dotNetProjects }}。
   - **将项目**：“tests/UnitTests/*.csproj”替换为“Test”任务中的 ${{ parameters.testProjects }}。

    新的 YAML 文件应如下所示：

    ```YAML
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

1. 保存管道并运行管道，它应成功运行。

    ![管道运行的屏幕截图，其中包含参数。](media/pipeline-parameters-run.png)

#### 任务 2：保护变量和参数

在此任务中，你将使用变量组保护管道中的变量和参数。

1. 转到 **管道>库**。

1. 选择“+ 变量组”以创建新的变量组。 为它命名，如 **BuildConfigurations**。

1. 添加名为 **buildConfiguration** 的变量并将其值设置为 `Release`。

1. 保存变量组。

    ![包含 BuildConfigurations 的变量组的屏幕截图。](media/eshop-variable-group.png)

1. 选择“管道权限”，然后选择 **+** 符号以添加管道****。

1. 选择 **eshoponweb-ci-parameters** 管道以允许管道使用变量组。

    ![屏幕截图显示管道权限](media/pipeline-permissions.png)

1. （可选）还可以通过单击“安全 **”** 按钮设置特定用户或组，以编辑变量组。

1. 返回到 YAML 文件，在参数下的顶部，通过添加以下内容来引用变量组：

    ```YAML
    variables:
      - group: BuildConfigurations
    
    ```

1. 在“生成”任务中，将命令“build”替换为以下行，以利用变量组中的生成配置。

    ```YAML
    command: 'build'
    projects: ${{ parameters.dotNetProjects }}
    configuration: $(buildConfiguration)
    
    ```

1. 保存并运行管道。 它应成功运行，并将生成配置设置为 `Release`. 可以通过查看“生成”任务的日志来验证这一点。

按照此方法，可以使用变量组来保护变量和参数，而无需在 YAML 文件中对其进行硬编码。

#### 任务 3：验证强制变量和参数

在此任务中，将在管道执行之前验证强制变量。

1. 转到“管道”>“管道”。

1. 打开 **eshoponweb-ci-parameters** 管道，然后选择“ **编辑**”。

1. 将新阶段添加为名为 **Validate** 的第一个阶段，以在管道执行之前验证强制变量。

    ```YAML
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
    > 此阶段将运行脚本来验证 buildConfiguration 变量。 如果未设置变量，脚本将失败，管道将停止。

1. **通过添加 dependsOn：在生成阶段下验证，使生成**阶段依赖于 **“验证**”阶段：

    ```YAML
    - stage: Build
      displayName: Build .Net Core Solution
      dependsOn: Validate
    
    ```

1. 保存并运行管道。 它将成功运行，因为 buildConfiguration 变量是在变量组中设置的。

1. 若要测试验证，请从变量组中删除 buildConfiguration 变量，或删除变量组，然后再次运行管道。 失败并出现以下错误：

    应会看到以下错误：

    ```YAML
    Error: buildConfiguration variable is not set
    
    ```

    ![管道运行的屏幕截图，其中显示了验证失败。](media/pipeline-validation-fail.png)

1. 将变量组和 buildConfiguration 变量添加回变量组，然后再次运行管道。 它应已成功运行。

## 审阅

在本实验室中，了解如何配置管道，以安全地使用变量和参数。
