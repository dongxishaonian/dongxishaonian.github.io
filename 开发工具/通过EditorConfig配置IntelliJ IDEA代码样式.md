[返回上层](index)

# 通过EditorConfig配置IntelliJ IDEA代码样式

# overview

在idea中配置代码样式规范通常有两种方式：

1. （IntelliJ IDEA Code Style XML）在IDEA的Preferences>Editor>Code Style中进行配置。
2. （EditorConfig）使用EditorConfig进行配置。

# 方式一（IntelliJ IDEA Code Style XML）

IDEA在设置层面提供了个性化代码规范的能力，打开`Preferences>Editor>Code Style`界面：

![image-20211125165057401](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-11-154908.png)

我们可以对需要的语言或文件类型进行格式规范的设置。

该方式也提供了导入/导出配置的功能，用户可以再个性化代码规范后，将其导出文件，并将配置文件分享给他人使用。

## 导出规范配置文件

导出方式有两种：

1. IDEA code style XML（**适用于方式一的配置方式，可将该文件分享给他人，从而保证大家的代码样式配置能一致！）**
2. EditorConfig File（适用于方式二的配置方式）

![image-20211125165456430](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-11-154930.png)

## 导入规范配置文件

导入支持Idea code style XML格式和EclipseXMl格式，如下图：

![image-20211125165733752](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-11-154944.png)

# 方式二（EditorConfig）

 EditorConfig是IDEA的另一种非常高效的设置代码样式配置的方式，他和方式一（Code Style XML）相比有如下优势：

1. 开箱即用，无需多余的设置。
2. 更细粒度的代码样式配置（可针对文件夹而非整个项目）

## 配置方式

### 确保你的IDEA开启了EditorConfig（自带，默认开启）

进入Preferences>Plugins，查看插件列表中是否开启了EditorConfig插件。

### 在目标文件夹内新建一个.EditorConfig文件

这里我们在项目的根目录中创建一个EditorConfig文件，即她在整个项目目录中生效。

<img src="/Users/chengang/Downloads/image2021-11-26_10-29-41.png" alt="image2021-11-26_10-29-41" style="zoom: 33%;" />

### 设置EditorConfig

![image2021-11-26_10-40-49](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-11-155144.png)

您的项目中可以有多个配置文件，每个文件都在不同的目录中。每当您在 IDE 中打开文件时，它都会检查同一目录中是否还有 .editorconfig 文件。如果没有，它将通过目录结构向上搜索。它不会停止，直到找到包含 root=true 的 .editorconfig 文件。您最顶层的配置文件应始终包含 root=true。

这意味着它可以在搜索过程中定位和加载多个配置文件。应用找到的所有文件的配置。层次结构中较深的文件优先于层次结构中较高的文件。这意味着更深的文件可以扩展和覆盖上面文件中的任何内容。

### 预览模式

在修改各种配置选项时，检查这些选项如何反映在实际代码文件中很有用。 在每个配置部分（例如 [Java] 或 [*]）旁边，您可以看到一个小眼睛图标。

![preview (2)](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-11-155406.png)

单击它时，您可以选择一个文件，该文件将用于预览您的更改。每当您对配置文件进行更改时，它都会反映在您的预览文件中。

![editor-preview](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-11-155454.png)

请注意，所有这些更改仅供预览，不会更改文件的内容。

**配置完成后，当我们格式化文件代码，EditorConfig就会自动生效！**

# 配置SaveAction实现文件保存自动格式化

您可以将 IDE 配置为在保存更改时自动重新格式化修改文件中的代码。

1. 在 Settings/Preferences对话框中 ⌘ , 选择Tools | Actions on Save.
2. 启用Reformat code 选项。

此外，您可以单击Configure scope 来指定要从重新格式化中排除的文件名和目录的模式。

# 总结

本文主要介绍了IDEA配置代码样式的两种方式，相比于第一种XML形式的配置方式，方式二更加便捷高效，只需创建一个EditorConfig配置，无需其他配置，并且我们将EditorConfig放置在项目目录中可以做到更好的版本化。同时搭配IDEA提供的Actions on Save功能能更有效的保证团队代码样式的一致性。
