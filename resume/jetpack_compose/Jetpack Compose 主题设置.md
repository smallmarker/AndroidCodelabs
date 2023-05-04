## 1\. 简介

在此 Codelab 中，您将学习如何使用 [Jetpack Compose](https://developer.android.com/jetpack/compose?hl=zh-cn) 的主题设置 API 来设置应用的样式。我们将了解如何自定义颜色、形状和排版，以便在整个应用中以一致的方式运用这些元素，从而支持多个主题（例如浅色主题和深色主题）。

## 学习内容

在此 Codelab 中，您将学习：

+   Material Design 入门指南以及如何针对您的品牌对其进行自定义
+   Compose 如何实现 Material Design 系统
+   如何在应用中定义和使用颜色、排版和形状
+   如何设置组件的样式
+   如何支持浅色主题和深色主题

## 构建内容

在此 Codelab 中，我们将设置新闻阅读应用的样式。我们将从未设置样式的应用入手，应用所学的内容来设置应用的主题，并为设置深色主题提供支持。

| ![显示应用样式前的新闻阅读应用 Jetnews 的图片。](/images/compose_theme_setting/1_1.png) | ![显示应用样式后的新闻阅读应用 Jetnews 的图片。](/images/compose_theme_setting/1_2.png) | ![显示将样式设为深色主题的新闻阅读应用 Jetnews 的图片。](/images/compose_theme_setting/1_3.png) |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 构建前：未设置样式的应用                                     | 构建后：已设置样式的应用                                     | 构建后：深色主题                                             |

## 前提条件

+   有使用 [Kotlin 语法](https://developer.android.com/jetpack/compose/kotlin?hl=zh-cn)（包括 lambda）的经验。
+   [对 Compose 有基本的了解](https://codelabs.developers.google.com/codelabs/jetpack-compose-basics/?hl=zh-cn#0)。
+   对 [Compose 布局](https://developer.android.com/codelabs/jetpack-compose-layouts?hl=zh-cn)（例如 `Row`、`Column` 和 `Modifier`）有基本的了解。

## 2\. 准备工作

在此步骤中，您将下载此 Codelab 的代码，其中包含一个简单的新闻阅读器应用，我们将设置该应用的样式。

## 所需条件

+   [最新版 Android Studio](https://developer.android.com/studio?hl=zh-cn)

## 下载代码

如果您已安装 git，只需运行以下命令即可。如需检查是否已安装 git，请在终端或命令行中输入 `git --version`，并验证其是否正确执行。

git clone https://github.com/googlecodelabs/android-compose-codelabs.git
cd android-compose-codelabs/ThemingCodelab

如果您未安装 git，可以点击下方按钮下载此 Codelab 的全部代码：

在 Android Studio 中打开项目，然后依次选择“File”>“Import Project”，接着找到 `ThemingCodelab` 目录。

此项目包含 3 个主要软件包：

+   `com.codelab.theming.data` - 该软件包包含模型类和示例数据。在此 Codelab 中，您无需修改该软件包。
+   `com.codelab.theming.ui.start` - 该软件包是此 Codelab 的起点，您应该**在该软件包中**完成此 Codelab 中要求的所有更改。
+   `com.codelab.theming.ui.finish` - 该软件包是此 Codelab 的最终状态，供您参考。

## 构建并运行应用

该应用有 2 个运行配置，分别反映了此 Codelab 的起始状态和最终状态。选择任意配置并按运行按钮，即可将相应代码部署到您的设备或模拟器。

![2.png](/images/compose_theme_setting/2.png)

该应用还包含 [Compose 布局预览](https://developer.android.com/jetpack/compose/preview?hl=zh-cn)。在 `start`/`finish` 软件包中浏览到 `Home.kt` 并打开设计视图，即会显示一些预览，从而根据您的界面代码进行快速迭代：

![3.png](/images/compose_theme_setting/3.png)

## 3\. Material 主题设置

Jetpack Compose 提供了 [Material Design](https://material.io/design/introduction) 的实现，后者是一个用于创建数字化界面的综合设计系统。Material Design [组件](https://material.io/components)（按钮、卡片、开关等）在 [Material 主题设置](https://material.io/design/material-theming/)的基础上构建而成，Material 主题设置是一种系统化的方法，用于**自定义** Material Design 以更好地反映您产品的品牌。一个 Material 主题由[颜色](https://material.io/design/color/)、[排版](https://material.io/design/typography/)和[形状](https://material.io/design/shape/)属性组成。如果您自定义这些属性，相应设置会自动反映在您用来构建应用的组件中。

了解 Material 主题设置有助于您了解如何为 Jetpack Compose 应用设置主题，因此我们会在下面简要介绍相关概念。如果您已熟悉 Material 主题设置，可以跳过这个部分继续学习后面的内容。

## 颜色

Material Design 定义了一些从语义上命名的[颜色](https://material.io/design/color/)，供您在整个应用中使用：

![4.png](/images/compose_theme_setting/4.png)

primary 是主要品牌颜色，secondary 用于提供强调色。您可以为形成对比的区域提供颜色更深/更浅的变体。background 和 surface 这两种颜色用于那些容纳在概念上驻留在应用“Surface”的组件的容器。此外，Material 还定义了“on”颜色，即针对具名颜色上层的内容使用的颜色；例如，“surface”色容器中的文本应采用“on surface”颜色。Material 组件已配置为使用这些主题颜色。例如，[悬浮操作按钮](https://material.io/components/buttons-floating-action-button)的默认颜色为 `secondary`，[卡片](https://material.io/components/cards)的默认颜色为 `surface`，诸如此类。

通过定义具名颜色，可以提供备用调色板，例如浅色主题和深色主题：

![5.png](/images/compose_theme_setting/5.png)

此外，我们还建议您定义一个小调色板，并在整个应用中一致地使用相应颜色。[Material 颜色工具](https://material.io/resources/color/)可以帮助您挑选颜色和创建调色板，甚至能够确保相应组合可访问。

## 排版

同样，Material 还[定义](https://material.io/design/typography/)了一些从语义上命名的字体样式：

![5_1.png](/images/compose_theme_setting/5_1.png)

虽然您可能不会按主题来更改字体样式，但使用字体比例可提升应用内部的一致性。如果您提供自己的字体和其他字体自定义设置，相应设置将反映在您在应用中使用的 Material 组件中，例如，[应用栏](https://material.io/components/app-bars-top)默认使用 `h6` 样式，[按钮](https://material.io/components/buttons)默认使用 `button`。Material [字体比例生成器工具](https://material.io/design/typography/the-type-system.html#type-scale)可以帮助您构建字体比例。

## 形状

Material 支持系统地使用[形状](https://material.io/design/shape/)来呈现您的品牌并传达出品牌理念。它定义了 3 个类别：小型、中型和大型组件；每种组件都可以定义要使用的形状，从而自定义角的样式（切角和圆角）和大小。

![5_2.png](/images/compose_theme_setting/5_2.png)

如果您自定义形状主题，相应设置将反映在众多组件中，例如，默认情况下，[按钮](https://material.io/components/buttons)和[文本字段](https://material.io/components/text-fields)使用小型形状主题，[卡片](https://material.io/components/cards)和[对话框](https://material.io/components/dialogs)使用中型形状主题，[动作条](https://material.io/components/sheets-bottom)使用大型形状主题。如需查看组件和形状主题的完整对应关系，请点击[此处](https://material.io/design/shape/applying-shape-to-ui.html#shape-scheme)。Material [形状自定义工具](https://material.io/design/shape/about-shape.html#shape-customization-tool)可帮助您生成形状主题。

## 基准

Material 默认采用“基准”主题，即紫色的配色方案、[Roboto](https://fonts.google.com/specimen/Roboto?hl=zh-cn) 字体比例，以及以上图片所示的略呈圆形的形状。如果您未指定或自定义主题，组件就会使用基准主题。

## 4\. 定义主题

## MaterialTheme

在 Jetpack Compose 中实现主题设置的核心元素是 [`MaterialTheme`](https://developer.android.com/reference/kotlin/androidx/compose/material/package-summary?hl=zh-cn#MaterialTheme(androidx.compose.material.Colors,androidx.compose.material.Typography,androidx.compose.material.Shapes,kotlin.Function0)) 可组合项。如果将此可组合项放在 Compose 层次结构中，您就可以为其中的所有组件指定颜色、字体和形状的自定义设置。下面是此可组合项在库中的定义方式：

```auto
@Composable
fun MaterialTheme(
    colors: Colors,
    typography: Typography,
    shapes: Shapes,
    content: @Composable () -> Unit
) { ...
```

以后，您可以使用 `MaterialTheme` `object` 检索传递到此可组合项的形参，以公开 `colors`、`typography` 和 `shapes` 属性。后面，我们将逐一进行深入介绍。

打开 `Home.kt` 并找到 `Home` 可组合函数 - 这是应用的主入口点。请注意，虽然我们声明了 `MaterialTheme`，但并未指定任何参数，因此会获得默认的“基准”样式：

```auto
@Composable
fun Home() {
  ...
  MaterialTheme {
    Scaffold(...
```

我们来创建颜色、字体和形状参数，为我们的应用实现主题。

## 创建主题

如需集中设置样式，我们建议您创建自己的可组合项，用于封装和配置 `MaterialTheme`。这样一来，您就可以在一个位置指定自己的主题自定义设置，并在多个位置（例如跨多个屏幕或 `@Preview`）轻松地重复使用这些自定义设置。您可以根据需要创建多个主题可组合项。例如，如果您想针对应用的不同部分支持不同的样式，就可以这样做。

在 `com.codelab.theming.ui.start.theme` 软件包中，新建一个名为 `Theme.kt` 的文件。添加一个名为 `JetnewsTheme` 的新可组合函数，此函数可接受其他可组合项作为内容并封装 `MaterialTheme`：

```auto
@Composable
fun JetnewsTheme(content: @Composable () -> Unit) {
  MaterialTheme(content = content)
}
```

现在，切换回 `Home.kt`，并将 `MaterialTheme` 替换为 `JetnewsTheme`（并将其导入）：

```auto
-  MaterialTheme {
+  JetnewsTheme {
    ...
```

在此屏幕上，`@Preview` 不会立即显示任何变化。更新 `PostItemPreview` 和 `FeaturedPostPreview` 以使用新的 `JetnewsTheme` 可组合项来封装其内容，以便预览使用新的主题：

```auto
@Preview("Featured Post")
@Composable
private fun FeaturedPostPreview() {
  val post = remember { PostRepo.getFeaturedPost() }
+ JetnewsTheme {
    FeaturedPost(post = post)
+ }
}
```

## 颜色

我们要在应用中实现的调色板如下所示（目前只是一个浅色调色板；我们很快会回来为设置深色主题提供支持）：

![6.png](/images/compose_theme_setting/6.png)

Compose 中的颜色是使用 [`Color`](https://developer.android.com/reference/kotlin/androidx/compose/ui/graphics/Color.html?hl=zh-cn) 类定义的。借助多个构造函数，您可以将颜色指定为 `ULong`，也可以按单独的颜色通道来指定颜色。

在 `theme` 软件包中创建一个新文件 `Color.kt`。在此文件中将以下颜色添加为顶级公共属性：

```auto
val Red700 = Color(0xffdd0d3c)
val Red800 = Color(0xffd00036)
val Red900 = Color(0xffc20029)
```

现在，我们已经定义了应用的颜色。接下来，我们将其合并到 `MaterialTheme` 所需的 [`Colors`](https://developer.android.com/reference/kotlin/androidx/compose/material/Colors?hl=zh-cn) 对象中，从而将特定颜色分配到 Material 的具名颜色。切换回 `Theme.kt`，然后添加以下代码：

```auto
private val LightColors = lightColors(
    primary = Red700,
    primaryVariant = Red900,
    onPrimary = Color.White,
    secondary = Red700,
    secondaryVariant = Red900,
    onSecondary = Color.White,
    error = Red800
)
```

下面，我们要使用 [`lightColors`](https://developer.android.com/reference/kotlin/androidx/compose/material/package-summary?hl=zh-cn#lightColors(androidx.compose.ui.graphics.Color,%20androidx.compose.ui.graphics.Color,%20androidx.compose.ui.graphics.Color,%20androidx.compose.ui.graphics.Color,%20androidx.compose.ui.graphics.Color,%20androidx.compose.ui.graphics.Color,%20androidx.compose.ui.graphics.Color,%20androidx.compose.ui.graphics.Color,%20androidx.compose.ui.graphics.Color,%20androidx.compose.ui.graphics.Color,%20androidx.compose.ui.graphics.Color,%20androidx.compose.ui.graphics.Color)) 函数来构建 `Colors`，这样即可提供合理的默认值，让我们不必将构成 Material 调色板的所有颜色全都指定出来。例如，请注意，我们尚未指定 `background` 颜色或许多“on”颜色，我们将使用默认值。

现在，让我们在应用中使用这些颜色。请更新 `JetnewsTheme` 可组合项以使用新的 `Colors`：

```auto
@Composable
fun JetnewsTheme(content: @Composable () -> Unit) {
  MaterialTheme(
+   colors = LightColors,
    content = content
  )
}
```

打开 `Home.kt` 并刷新预览。请注意，新的配色方案会反映在 `TopAppBar` 等组件中。

## 排版

我们要在应用中实现的字体比例如下所示：

![7.png](/images/compose_theme_setting/7.png)

在 Compose 中，我们可以定义 [`TextStyle`](https://developer.android.com/reference/kotlin/androidx/compose/ui/text/TextStyle?hl=zh-cn) 对象，以定义设置一些文本的样式所需的信息。下面是其属性的示例：

```auto
data class TextStyle(
    val color: Color = Color.Unset,
    val fontSize: TextUnit = TextUnit.Inherit,
    val fontWeight: FontWeight? = null,
    val fontStyle: FontStyle? = null,
    val fontFamily: FontFamily? = null,
    val letterSpacing: TextUnit = TextUnit.Inherit,
    val background: Color = Color.Unset,
    val textAlign: TextAlign? = null,
    val textDirection: TextDirection? = null,
    val lineHeight: TextUnit = TextUnit.Inherit,
    ...
)
```

我们所需的字体比例要针对标题使用 [Montserrat](https://fonts.google.com/specimen/Montserrat?hl=zh-cn)，并针对正文文本使用 [Domine](https://fonts.google.com/specimen/Domine?hl=zh-cn)。相关字体文件已添加到您项目的 `res/fonts` 文件夹中。

在 `theme` 软件包中创建一个新文件 `Typography.kt`。首先，我们将定义 [`FontFamily`](https://developer.android.com/reference/kotlin/androidx/compose/ui/text/font/FontFamily?hl=zh-cn)（结合了每个 [`Font`](https://developer.android.com/reference/kotlin/androidx/compose/ui/text/font/Font?hl=zh-cn) 的不同粗细）：

```auto
private val Montserrat = FontFamily(
    Font(R.font.montserrat_regular),
    Font(R.font.montserrat_medium, FontWeight.W500),
    Font(R.font.montserrat_semibold, FontWeight.W600)
)

private val Domine = FontFamily(
    Font(R.font.domine_regular),
    Font(R.font.domine_bold, FontWeight.Bold)
)
```

现在创建一个 `MaterialTheme` 接受的 [`Typography`](https://developer.android.com/reference/kotlin/androidx/compose/material/Typography?hl=zh-cn) 对象，为比例中的每个语义样式指定 `TextStyle`：

```auto
val JetnewsTypography = Typography(
    h4 = TextStyle(
        fontFamily = Montserrat,
        fontWeight = FontWeight.W600,
        fontSize = 30.sp
    ),
    h5 = TextStyle(
        fontFamily = Montserrat,
        fontWeight = FontWeight.W600,
        fontSize = 24.sp
    ),
    h6 = TextStyle(
        fontFamily = Montserrat,
        fontWeight = FontWeight.W600,
        fontSize = 20.sp
    ),
    subtitle1 = TextStyle(
        fontFamily = Montserrat,
        fontWeight = FontWeight.W600,
        fontSize = 16.sp
    ),
    subtitle2 = TextStyle(
        fontFamily = Montserrat,
        fontWeight = FontWeight.W500,
        fontSize = 14.sp
    ),
    body1 = TextStyle(
        fontFamily = Domine,
        fontWeight = FontWeight.Normal,
        fontSize = 16.sp
    ),
    body2 = TextStyle(
        fontFamily = Montserrat,
        fontSize = 14.sp
    ),
    button = TextStyle(
        fontFamily = Montserrat,
        fontWeight = FontWeight.W500,
        fontSize = 14.sp
    ),
    caption = TextStyle(
        fontFamily = Montserrat,
        fontWeight = FontWeight.Normal,
        fontSize = 12.sp
    ),
    overline = TextStyle(
        fontFamily = Montserrat,
        fontWeight = FontWeight.W500,
        fontSize = 12.sp
    )
)
```

打开 `Theme.kt` 并更新 `JetnewsTheme` 可组合项，以使用新的 `Typography`：

```auto
@Composable
fun JetnewsTheme(content: @Composable () -> Unit) {
  MaterialTheme(
    colors = LightColors,
+   typography = JetnewsTypography,
    content = content
  )
}
```

打开 `Home.kt` 并刷新预览，以查看新排版的实际效果。

## 形状

我们想在应用中使用形状来呈现我们的品牌并表达品牌理念。我们想在一些元素上使用切角形状：

![8.png](/images/compose_theme_setting/8.png)

Compose 提供了 [`RoundedCornerShape`](https://developer.android.com/reference/kotlin/androidx/compose/foundation/shape/RoundedCornerShape?hl=zh-cn) 类和 [`CutCornerShape`](https://developer.android.com/reference/kotlin/androidx/compose/foundation/shape/CutCornerShape?hl=zh-cn) 类，可用于定义形状主题。

在 `theme` 软件包中创建一个新文件 `Shape.kt`，并添加以下代码：

```auto
val JetnewsShapes = Shapes(
    small = CutCornerShape(topStart = 8.dp),
    medium = CutCornerShape(topStart = 24.dp),
    large = RoundedCornerShape(8.dp)
)
```

打开 `Theme.kt` 并更新 `JetnewsTheme` 可组合项，以使用这些 [`Shapes`](https://developer.android.com/reference/kotlin/androidx/compose/material/Shapes?hl=zh-cn)：

```auto
@Composable
fun JetnewsTheme(content: @Composable () -> Unit) {
  MaterialTheme(
    colors = LightColors,
    typography = JetnewsTypography,
+   shapes = JetnewsShapes,
    content = content
  )
}
```

打开 `Home.kt` 并刷新预览，以查看显示精选博文的 `Card` 如何反映新应用的形状主题。

## 深色主题

在应用中支持[深色主题](https://developer.android.com/guide/topics/ui/look-and-feel/darktheme?hl=zh-cn)不仅有助于您的应用在用户设备上更好地集成（从 Android 10 开始，设备上已提供全局深色主题切换开关），还有助于降低能耗以及为满足无障碍功能需求提供支持。Material 提供了关于如何创建深色主题的[设计指南](https://material.io/design/color/dark-theme.html)。以下是我们想为深色主题实现的备用调色板：

![9.png](/images/compose_theme_setting/9.png)

打开 `Color.kt` 并添加以下颜色：

```auto
val Red200 = Color(0xfff297a2)
val Red300 = Color(0xffea6d7e)
```

现在，打开 `Theme.kt` 并添加以下代码：

```auto
private val DarkColors = darkColors(
    primary = Red300,
    primaryVariant = Red700,
    onPrimary = Color.Black,
    secondary = Red300,
    onSecondary = Color.Black,
    error = Red200
)
```

现在，更新 `JetnewsTheme`：

```auto
@Composable
fun JetnewsTheme(
+ darkTheme: Boolean = isSystemInDarkTheme(),
  content: @Composable () -> Unit
) {
  MaterialTheme(
+   colors = if (darkTheme) DarkColors else LightColors,
    typography = JetnewsTypography,
    shapes = JetnewsShapes,
    content = content
  )
}
```

此时，我们已经添加了用于判断是否使用深色主题的新形参，并将其默认设置为查询设备的[全局设置](https://developer.android.com/reference/kotlin/androidx/compose/foundation/package-summary?hl=zh-cn#isSystemInDarkTheme())。这样即可为我们提供很实用的默认值，但如果我们要让特定屏幕始终/永不采用深色主题，或要让 `@Preview` 采用深色主题，我们仍可轻松进行替换。

打开 `Home.kt` 并为 `FeaturedPost` 可组合项创建新的预览，此预览能够以深色主题显示该可组合项：

```auto
@Preview("Featured Post • Dark")
@Composable
private fun FeaturedPostDarkPreview() {
    val post = remember { PostRepo.getFeaturedPost() }
    JetnewsTheme(darkTheme = true) {
        FeaturedPost(post = post)
    }
}
```

刷新预览窗格以查看深色主题预览。

![10.png](/images/compose_theme_setting/10.png)

## 5\. 处理颜色

在上一步骤中，我们了解了如何创建自己的主题，以为您的应用设置颜色、字体样式和形状。所有 Material 组件开箱即可使用这些自定义功能。例如，`FloatingActionButton` 可组合项默认使用主题中的 `secondary` 颜色，但您可以通过为此参数指定不同的值来设置备用颜色：

```auto
@Composable
fun FloatingActionButton(
  backgroundColor: Color = MaterialTheme.colors.secondary,
  ...
) {
```

有时，您并不想使用默认设置；本部分将介绍如何在您的应用中使用颜色。

## 原色

如前所述，Compose 提供了一个 `Color` 类。您可以在本地创建这些类，并将其保留在 `object` 等元素中：

```auto
Surface(color = Color.LightGray) {
  Text(
    text = "Hard coded colors don't respond to theme changes :(",
    textColor = Color(0xffff00ff)
  )
}
```

`Color` 中有许多有用的方法，例如 `copy`，您可以通过此方法使用不同的 alpha/red/green/blue 值来创建新的颜色。

## 主题颜色

一种更灵活的方法是从主题中检索颜色：

```auto
Surface(color = MaterialTheme.colors.primary)
```

下面，我们要使用 `MaterialTheme` `object`，其 `colors` 属性会返回在 `MaterialTheme` 可组合项中设置的 `Colors`。这意味着，我们只需为主题提供不同的颜色集，即可支持不同的外观和风格，而无需处理应用代码。例如，我们的 `AppBar` 使用 `primary` 颜色，屏幕背景使用 `surface` 颜色；如果更改主题颜色，相应设置会反映在以下可组合项中：

![11.png](/images/compose_theme_setting/11.png)

![11_2.png](/images/compose_theme_setting/11_2.png)

由于主题中的每种颜色都是 `Color` 实例，因此我们还可以使用 `copy` 方法轻松地“派生”颜色：

```auto
val derivedColor = MaterialTheme.colors.onSurface.copy(alpha = 0.1f)
```

下面，我们要复制 `onSurface` 颜色，但要将不透明度设为 10%。此方法可确保颜色能够在不同主题下正常显示，而无需硬编码静态颜色。

## Surface 颜色和内容颜色

许多组件都接受一对颜色和“内容颜色”：

```auto
Surface(
  color: Color = MaterialTheme.colors.surface,
  contentColor: Color = contentColorFor(color),
  ...

TopAppBar(
  backgroundColor: Color = MaterialTheme.colors.primarySurface,
  contentColor: Color = contentColorFor(backgroundColor),
  ...
```

这样一来，您不仅可以设置可组合项的颜色，而且还能为“内容”（即包含在其中的可组合项）提供默认颜色。默认情况下，许多可组合项都使用这种内容颜色，例如 `Text` 颜色或 `Icon` 色调。[`contentColorFor`](https://developer.android.com/reference/kotlin/androidx/compose/material/package-summary?hl=zh-cn#contentColorFor(androidx.compose.ui.graphics.Color)) 方法可以为任何主题颜色检索适当的“on”颜色，例如，如果您设置 `primary` 背景，它就会返回 `onPrimary` 作为内容颜色。如果您设置非主题背景颜色，则应自行提供合理的内容颜色。

```auto
Surface(color = MaterialTheme.colors.primary) {
  Text(...) // default text color is 'onPrimary'
}
Surface(color = MaterialTheme.colors.error) {
  Icon(...) // default tint is 'onError'
}
```

您可以使用 [`LocalContentColor`](https://developer.android.com/reference/kotlin/androidx/compose/material/package-summary?hl=zh-cn#LocalContentColor()) [`CompositionLocal`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/CompositionLocal?hl=zh-cn) 来检索与当前背景形成对比的颜色：

```auto
BottomNavigationItem(
  unselectedContentColor = LocalContentColor.current ...
```

当设置任何元素的颜色时，最好使用 [`Surface`](https://developer.android.com/reference/kotlin/androidx/compose/material/package-summary?hl=zh-cn#Surface(androidx.compose.ui.Modifier,androidx.compose.ui.graphics.Shape,androidx.compose.ui.graphics.Color,androidx.compose.ui.graphics.Color,androidx.compose.foundation.BorderStroke,androidx.compose.ui.unit.Dp,kotlin.Function0)) 来实现此目的，因为它会设置适当的内容颜色 [`CompositionLocal`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/CompositionLocal?hl=zh-cn) 值。请慎用直接 [`Modifier.background`](https://developer.android.com/reference/kotlin/androidx/compose/foundation/package-summary?hl=zh-cn#background(androidx.compose.ui.Modifier,androidx.compose.ui.graphics.Color,androidx.compose.ui.graphics.Shape)) 调用，这种调用**不会**设置适当的内容颜色。

```auto
-Row(Modifier.background(MaterialTheme.colors.primary)) {
+Surface(color = MaterialTheme.colors.primary) {
+  Row(
...
```

目前，我们的 `Header` 组件始终具有 `Color.LightGray` 背景。这在浅色主题中看起来没有问题，但在深色主题中，就会与背景形成高度对比。此外，它们也不指定特定的文本颜色，因此会继承可能不会与背景形成对比的当前内容颜色：

![12.png](/images/compose_theme_setting/12.png)

接下来，让我们解决这个问题。在 `Home.kt` 的 `Header` 可组合项中，移除用于指定硬编码颜色的 `background` 修饰符。改为将 `Text` 封装在包含主题派生颜色的 `Surface` 中，并指定相应内容应采用 `primary` 颜色：

```auto
+ Surface(
+   color = MaterialTheme.colors.onSurface.copy(alpha = 0.1f),
+   contentColor = MaterialTheme.colors.primary,
+   modifier = modifier
+ ) {
  Text(
    text = text,
    modifier = Modifier
      .fillMaxWidth()
-     .background(Color.LightGray)
      .padding(horizontal = 16.dp, vertical = 8.dp)
  )
+ }
```

## 内容 Alpha 值

通常情况下，我们希望通过强调或弱化内容来突出重点并体现出视觉上的层次感。Material Design [建议](https://material.io/design/color/text-legibility.html)采用不同的不透明度来传达这些不同的重要程度。

Jetpack Compose 通过 [`LocalContentAlpha`](https://developer.android.com/reference/kotlin/androidx/compose/material/package-summary?hl=zh-cn#LocalContentAlpha()) 实现此功能。您可以通过为此 [`CompositionLocal`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/CompositionLocal?hl=zh-cn) [提供](https://developer.android.com/reference/kotlin/androidx/compose/runtime/package-summary?hl=zh-cn#CompositionLocalProvider(kotlin.Array,kotlin.Function0))一个值来为层次结构指定内容 Alpha 值。子可组合项可以使用此值，例如 [`Text`](https://developer.android.com/reference/kotlin/androidx/compose/material/package-summary?hl=zh-cn#Text(kotlin.String,androidx.compose.ui.Modifier,androidx.compose.ui.graphics.Color,androidx.compose.ui.unit.TextUnit,androidx.compose.ui.text.font.FontStyle,androidx.compose.ui.text.font.FontWeight,androidx.compose.ui.text.font.FontFamily,androidx.compose.ui.unit.TextUnit,androidx.compose.ui.text.style.TextDecoration,androidx.compose.ui.text.style.TextAlign,androidx.compose.ui.unit.TextUnit,androidx.compose.ui.text.style.TextOverflow,kotlin.Boolean,kotlin.Int,kotlin.Function1,androidx.compose.ui.text.TextStyle)) 和 [`Icon`](https://developer.android.com/reference/kotlin/androidx/compose/material/package-summary?hl=zh-cn#Icon(androidx.compose.ui.graphics.vector.ImageVector,kotlin.String,androidx.compose.ui.Modifier,androidx.compose.ui.graphics.Color)) 默认使用 `LocalContentColor` 的组合，已调整为使用 `LocalContentAlpha`。Material 指定了一些标准 Alpha 值（[`high`](https://developer.android.com/reference/kotlin/androidx/compose/material/ContentAlpha?hl=zh-cn#high())、[`medium`](https://developer.android.com/reference/kotlin/androidx/compose/material/ContentAlpha?hl=zh-cn#medium())、[`disabled`](https://developer.android.com/reference/kotlin/androidx/compose/material/ContentAlpha?hl=zh-cn#disabled())），这些值由 [`ContentAlpha`](https://developer.android.com/reference/kotlin/androidx/compose/material/ContentAlpha?hl=zh-cn) 对象建模。请注意，`MaterialTheme` 默认将 `LocalContentAlpha` 设置为 `ContentAlpha.high`。

```auto
// By default, both Icon & Text use the combination of LocalContentColor &
// LocalContentAlpha. De-emphasize content by setting a different content alpha
CompositionLocalProvider(LocalContentAlpha provides ContentAlpha.medium) {
    Text(...)
}
CompositionLocalProvider(LocalContentAlpha provides ContentAlpha.disabled) {
    Icon(...)
    Text(...)
}
```

这样即可方便又一致地突出组件的重要性。

我们将使用内容 Alpha 值来阐明精选博文的信息层次结构。在 `Home.kt` 的 `PostMetadata` 可组合项中，重点突出元数据 `medium`：

```auto
+ CompositionLocalProvider(LocalContentAlpha provides ContentAlpha.medium) {
  Text(
    text = text,
    modifier = modifier
  )
+ }
```

![13.png](/images/compose_theme_setting/13.png)

## 深色主题

如我们所见，若要在 Compose 中实现深色主题，您只需提供不同的颜色集并通过主题查询颜色即可。下面是一些需要注意的例外情况：

您可以检查您是否在浅色主题中运行：

```auto
val isLightTheme = MaterialTheme.colors.isLight
```

此值由 [lightColors](https://developer.android.com/reference/kotlin/androidx/compose/material/package-summary?hl=zh-cn#lightColors(androidx.compose.ui.graphics.Color,androidx.compose.ui.graphics.Color,androidx.compose.ui.graphics.Color,androidx.compose.ui.graphics.Color,androidx.compose.ui.graphics.Color,androidx.compose.ui.graphics.Color,androidx.compose.ui.graphics.Color,androidx.compose.ui.graphics.Color,androidx.compose.ui.graphics.Color,androidx.compose.ui.graphics.Color,androidx.compose.ui.graphics.Color,androidx.compose.ui.graphics.Color))/[darkColors](https://developer.android.com/reference/kotlin/androidx/compose/material/package-summary?hl=zh-cn#darkColors(androidx.compose.ui.graphics.Color,androidx.compose.ui.graphics.Color,androidx.compose.ui.graphics.Color,androidx.compose.ui.graphics.Color,androidx.compose.ui.graphics.Color,androidx.compose.ui.graphics.Color,androidx.compose.ui.graphics.Color,androidx.compose.ui.graphics.Color,androidx.compose.ui.graphics.Color,androidx.compose.ui.graphics.Color,androidx.compose.ui.graphics.Color,androidx.compose.ui.graphics.Color)) 构建器函数设置。

在 Material 中，如果采用的是深色主题，高度较高的 Surface 会获得[高度叠加层](https://material.io/design/color/dark-theme.html#properties)（其背景颜色会变浅）。在使用深色调色板时，系统会自动实现此效果：

```auto
Surface(
  elevation = 2.dp,
  color = MaterialTheme.colors.surface, // color will be adjusted for elevation
  ...
```

在我们的应用中，我们可以在我们使用的 `TopAppBar` 和 `Card` 组件中看到上述自动行为；默认情况下，这两种组件的高度分别设为 4dp 和 1dp，因此，在深色主题中，它们的背景颜色会自动变浅，以更好地表现相应高度：

![14.png](/images/compose_theme_setting/14.png)

Material Design [建议](https://material.io/design/color/dark-theme.html#custom-application)避免在深色主题中使用大面积的明亮颜色。一种常见模式是在浅色主题中将容器设为 `primary` 颜色，并在深色主题中将其设为 `surface` 颜色；许多组件都默认使用此策略，例如[应用栏](https://material.io/components/app-bars-top)和[底部导航栏](https://material.io/components/bottom-navigation)。为了便于实现，`Colors` 提供了 `primarySurface` 颜色，以准确完成上述行为，并且这些组件都默认使用此颜色。

目前，我们的应用将应用栏设置为 `primary` 颜色；若要遵循此指南，只需将其切换为 `primarySurface` 或移除此形参（因为此形参为默认设置）即可。在 `AppBar` 可组合项中，更改 `TopAppBar` 的 `backgroundColor` 参数：

```auto
@Composable
private fun AppBar() {
  TopAppBar(
    ...
-   backgroundColor = MaterialTheme.colors.primary
+   backgroundColor = MaterialTheme.colors.primarySurface
  )
}
```
## 6\. 处理文本

在处理文本时，我们使用 [`Text`](https://developer.android.com/reference/kotlin/androidx/compose/material/package-summary?hl=zh-cn#Text(kotlin.String,androidx.compose.ui.Modifier,androidx.compose.ui.graphics.Color,androidx.compose.ui.unit.TextUnit,androidx.compose.ui.text.font.FontStyle,androidx.compose.ui.text.font.FontWeight,androidx.compose.ui.text.font.FontFamily,androidx.compose.ui.unit.TextUnit,androidx.compose.ui.text.style.TextDecoration,androidx.compose.ui.text.style.TextAlign,androidx.compose.ui.unit.TextUnit,androidx.compose.ui.text.style.TextOverflow,kotlin.Boolean,kotlin.Int,kotlin.Function1,androidx.compose.ui.text.TextStyle)) 可组合项来显示文本，使用 [`TextField`](https://developer.android.com/reference/kotlin/androidx/compose/material/package-summary?hl=zh-cn#TextField(kotlin.String,kotlin.Function1,androidx.compose.ui.Modifier,kotlin.Boolean,kotlin.Boolean,androidx.compose.ui.text.TextStyle,kotlin.Function0,kotlin.Function0,kotlin.Function0,kotlin.Function0,kotlin.Boolean,androidx.compose.ui.text.input.VisualTransformation,androidx.compose.foundation.text.KeyboardOptions,androidx.compose.foundation.text.KeyboardActions,kotlin.Boolean,kotlin.Int,androidx.compose.foundation.interaction.MutableInteractionSource,androidx.compose.ui.graphics.Shape,androidx.compose.material.TextFieldColors)) 和 [`OutlinedTextField`](https://developer.android.com/reference/kotlin/androidx/compose/material/package-summary?hl=zh-cn#OutlinedTextField(kotlin.String,kotlin.Function1,androidx.compose.ui.Modifier,kotlin.Boolean,kotlin.Boolean,androidx.compose.ui.text.TextStyle,kotlin.Function0,kotlin.Function0,kotlin.Function0,kotlin.Function0,kotlin.Boolean,androidx.compose.ui.text.input.VisualTransformation,androidx.compose.foundation.text.KeyboardOptions,androidx.compose.foundation.text.KeyboardActions,kotlin.Boolean,kotlin.Int,androidx.compose.foundation.interaction.MutableInteractionSource,androidx.compose.material.TextFieldColors)) 进行文本输入，并使用 [`TextStyle`](https://developer.android.com/reference/kotlin/androidx/compose/ui/text/TextStyle?hl=zh-cn) 对文本应用单一样式。我们可以使用 [`AnnotatedString`](https://developer.android.com/reference/kotlin/androidx/compose/ui/text/AnnotatedString.html?hl=zh-cn) 对文本应用多种样式。

正如我们在设置颜色时所看到的那样，用于显示文本的 Material 组件将获取我们的主题排版自定义设置：

```auto
Button(...) {
  Text("This text will use MaterialTheme.typography.button style by default")
}
```

实现此目的要比使用默认参数（如在设置颜色时所看到的那样）略微复杂一些。这是因为组件本身往往不会显示文本，而是提供[槽 API](https://developer.android.com/jetpack/compose/layouts/basics?hl=zh-cn#slot-based-layouts)，让您能够传入 `Text` 可组合项。那么，组件是如何设置主题排版样式的呢？在后台，它们使用 [`ProvideTextStyle`](https://developer.android.com/reference/kotlin/androidx/compose/material/package-summary?hl=zh-cn#ProvideTextStyle(androidx.compose.ui.text.TextStyle,kotlin.Function0)) 可组合项（本身就使用 [`CompositionLocal`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/CompositionLocal?hl=zh-cn)）来设置“current”`TextStyle`。如果您未提供具体的 `textStyle` 参数，`Text` 可组合项会默认查询此“current”样式。

例如，通过 Compose 的 [`Button`](https://cs.android.com/androidx/platform/frameworks/support/+/androidx-main:compose/material/material/src/commonMain/kotlin/androidx/compose/material/Button.kt?hl=zh-cn) 类和 [`Text`](https://cs.android.com/androidx/platform/frameworks/support/+/androidx-main:compose/material/material/src/commonMain/kotlin/androidx/compose/material/Text.kt?hl=zh-cn) 类：

```auto
@Composable
fun Button(
    // many other parameters
    content: @Composable RowScope.() -> Unit
) {
  ...
  ProvideTextStyle(MaterialTheme.typography.button) { //set the "current" text style
    ...
    content()
  }
}

@Composable
fun Text(
    // many, many parameters
    style: TextStyle = LocalTextStyle.current // get the value set by ProvideTextStyle
) { ...
```

## 主题文本样式

就像处理颜色时一样，最好从当前主题中检索 `TextStyle`，从而鼓励您使用一组数量少且一致的样式，并使其更易于维护。`MaterialTheme.typography` 会检索在 `MaterialTheme` 可组合项中设置的 `Typography` 实例，让您能够使用自己定义的样式：

```auto
Text(
  style = MaterialTheme.typography.subtitle2
)
```

如果您需要自定义 `TextStyle`，可以对其执行 `copy` 操作并替换相关属性（它只是一个 `data class`），或者让 `Text` 可组合项接受大量样式形参，这些形参会叠加到任何 `TextStyle` 的上层：

```auto
Text(
  text = "Hello World",
  style = MaterialTheme.typography.body1.copy(
    background = MaterialTheme.colors.secondary
  )
)
```

```auto
Text(
  text = "Hello World",
  style = MaterialTheme.typography.subtitle2,
  fontSize = 22.sp // explicit size overrides the size in the style
)
```

在我们的应用中，许多地方都会自动应用主题 `TextStyle`，例如，`TopAppBar` 将其 `title` 的样式设为 `h6`，而 [`ListItem`](https://developer.android.com/reference/kotlin/androidx/compose/material/package-summary?hl=zh-cn#ListItem(androidx.compose.ui.Modifier,kotlin.Function0,kotlin.Function0,kotlin.Boolean,kotlin.Function0,kotlin.Function0,kotlin.Function0)) 将其主要文本和辅助文本的样式分别设为 `subtitle1` 和 `body2`。

接下来，我们要将主题排版样式应用于应用的其余部分。将 `Header` 设为使用 `subtitle2`；对于 `FeaturedPost` 中的文本，将标题设为 `h6`，并将作者信息和元数据设为 `body2`：

```auto
@Composable
fun Header(...) {
  ...
  Text(
    text = text,
+   style = MaterialTheme.typography.subtitle2
```

![15.png](/images/compose_theme_setting/15.png)

## 多种样式

如果您需要对某些文本应用多种样式，可以使用 [`AnnotatedString`](https://developer.android.com/reference/kotlin/androidx/compose/ui/text/AnnotatedString.html?hl=zh-cn) 类来应用标记，从而为一系列文本添加 [`SpanStyle`](https://developer.android.com/reference/kotlin/androidx/compose/ui/text/SpanStyle?hl=zh-cn)。您可以动态添加这些元素，也可以使用 DSL 语法来创建内容：

```auto
val text = buildAnnotatedString {
  append("This is some unstyled text\n")
  withStyle(SpanStyle(color = Color.Red)) {
    append("Red text\n")
  }
  withStyle(SpanStyle(fontSize = 24.sp)) {
    append("Large text")
  }
}
```

接下来，我们要为描述应用中的各个博文的标签设置样式。目前，它们使用与元数据其余部分相同的文本样式；我们将使用 `overline` 文本样式和背景颜色来区分它们。在 `PostMetadata` 可组合项中：

```auto
+ val tagStyle = MaterialTheme.typography.overline.toSpanStyle().copy(
+   background = MaterialTheme.colors.primary.copy(alpha = 0.1f)
+ )
post.tags.forEachIndexed { index, tag ->
  ...
+ withStyle(tagStyle) {
    append(" ${tag.toUpperCase()} ")
+ }
}
```

![16.png](/images/compose_theme_setting/16.png)
## 7\. 处理形状

与颜色和排版一样，如果设置形状主题，相应设置会反映在 Material 组件中。例如，`Button` 会获取为小型组件设置的形状：

```auto
@Composable
fun Button( ...
  shape: Shape = MaterialTheme.shapes.small
) {
```

与颜色一样，Material 组件使用默认参数，因此您可以直接查看组件将要使用的形状类别，或提供替代方案。如需查看组件和形状类别的完整对应关系，请参阅此[文档](https://material.io/design/shape/applying-shape-to-ui.html#shape-scheme)。

请注意，有些组件会使用经过修改的主题形状，以适应其上下文的要求。例如，默认情况下，`TextField` 使用小型形状主题，但它会对底角应用零边角大小：

```auto
@Composable
fun FilledTextField(
  // other parameters
  shape: Shape = MaterialTheme.shapes.small.copy(
    bottomStart = ZeroCornerSize, // overrides small theme style
    bottomEnd = ZeroCornerSize // overrides small theme style
  )
) {
```

![17.png](/images/compose_theme_setting/17.png)

## 主题形状

当然，在您创建自己的组件时，您可以自行使用各种形状；为此，您需要使用接受形状的可组合项或 `Modifier`（例如，[`Surface`](https://developer.android.com/reference/kotlin/androidx/compose/material/package-summary?hl=zh-cn#Surface(androidx.compose.ui.Modifier,androidx.compose.ui.graphics.Shape,androidx.compose.ui.graphics.Color,androidx.compose.ui.graphics.Color,androidx.compose.foundation.BorderStroke,androidx.compose.ui.unit.Dp,kotlin.Function0))、[`Modifier.clip`](https://developer.android.com/reference/kotlin/androidx/compose/ui/draw/package-summary?hl=zh-cn#clip(androidx.compose.ui.Modifier,androidx.compose.ui.graphics.Shape))、[`Modifier.background`](https://developer.android.com/reference/kotlin/androidx/compose/foundation/package-summary?hl=zh-cn#background(androidx.compose.ui.Modifier,androidx.compose.ui.graphics.Color,androidx.compose.ui.graphics.Shape))、[`Modifier.border`](https://developer.android.com/reference/kotlin/androidx/compose/foundation/package-summary?hl=zh-cn#border(androidx.compose.ui.Modifier,androidx.compose.foundation.BorderStroke,androidx.compose.ui.graphics.Shape)) 等）。

```auto
@Composable
fun UserProfile(
  ...
  shape: Shape = MaterialTheme.shapes.medium
) {
  Surface(shape = shape) {
    ...
  }
}
```

接下来，我们要将形状主题添加到 `PostItem` 中显示的图片；我们要对其应用主题的 `small` 形状，并使用 `clip``Modifier` 切去左上角：

```auto
@Composable
fun PostItem(...) {
  ...
  Image(
    painter = painterResource(post.imageThumbId),
+   modifier = Modifier.clip(shape = MaterialTheme.shapes.small)
  )
```

![18.png](/images/compose_theme_setting/18.png)
## 8\. 组件“样式”

Compose 没有提供用于提取组件样式（例如，Android View 样式或 CSS 样式）的明确方法。由于所有 Compose 组件都是用 Kotlin 编写的，因此还可通过其他方法来实现相同的目的。您可以改为创建自己的自定义组件库，并在整个应用中使用这些组件。

我们已经在我们的应用中这样做了：

```auto
@Composable
fun Header(
  text: String,
  modifier: Modifier = Modifier
) {
  Surface(
    color = MaterialTheme.colors.onSurface.copy(alpha = 0.1f),
    contentColor = MaterialTheme.colors.primary,
    modifier = modifier.semantics { heading() }
  ) {
    Text(
      text = text,
      style = MaterialTheme.typography.subtitle2,
      modifier = Modifier
        .fillMaxWidth()
        .padding(horizontal = 16.dp, vertical = 8.dp)
    )
  }
}
```

`Header` 可组合项本质上是样式化的 `Text`，可供我们在整个应用中使用。

我们都看到了，所有组件都是由较低级别的构建块构造而成的，您可以使用同样的构建块来自定义 Material 组件。例如，我们看到 `Button` 使用 `ProvideTextStyle` 可组合项为传递给它的内容设置默认文本样式。您可以使用完全相同的机制来设置自己的文本样式：

```auto
@Composable
fun LoginButton(
    onClick: () -> Unit,
    modifier: Modifier = Modifier,
    content: @Composable RowScope.() -> Unit
) {
    Button(
        colors = ButtonConstants.defaultButtonColors(
            backgroundColor = MaterialTheme.colors.secondary
        ),
        onClick = onClick,
        modifier = modifier
    ) {
        ProvideTextStyle(...) { // set our own text style
            content()
        }
    }
}
```

在此示例中，我们创建了自己的 `LoginButton`“样式”，方法是封装标准 `Button` 类，然后指定特定属性（例如不同的 `backgroundColor` 和文本样式）。

此外，也没有默认样式（即自定义某个组件类型的默认外观的方法）的概念。同样，为实现此目的，您可以创建您自己的组件，用于封装和自定义库组件。例如，您想自定义应用中所有 `Button` 的形状，但不想更改小型形状主题，因为更改小型形状主题会影响其他（非 `Button`）组件。如需实现此目的，您可以创建自己的可组合项并在整个应用中使用此可组合项：

```auto
@Composable
fun AcmeButton(
  // expose Button params consumers should be able to change
) {
  val acmeButtonShape: Shape = ...
  Button(
    shape = acmeButtonShape,
    // other params
  )
}
```

## 9\. 结尾

恭喜您，您已成功完成了此 Codelab，并设置了 Jetpack Compose 应用的样式！

您实现了 Material 主题，自定义了整个应用中使用的颜色、排版和形状，以展现您的品牌并提升一致性。您添加了对浅色主题和深色主题的支持。

## 相关文章

+   [Compose 基础知识](https://codelabs.developers.google.com/codelabs/jetpack-compose-basics/?hl=zh-cn#0)
+   [Compose 布局](https://codelabs.developers.google.com/codelabs/jetpack-compose-layouts?hl=zh-cn)
+   [Compose 中的状态](https://codelabs.developers.google.com/codelabs/jetpack-compose-state?hl=zh-cn)
+   [在现有应用中采用 Compose](https://codelabs.developers.google.com/codelabs/jetpack-compose-migration?hl=zh-cn)

## 深入阅读

+   [Compose 主题设置指南](https://developer.android.com/jetpack/compose/themes?hl=zh-cn)
+   [Material 主题设置](https://material.io/design/material-theming/)

## [示例应用](https://github.com/android/compose-samples/)

+   演示多个主题的 Owl
+   演示动态主题的 Jetcaster
+   演示如何实现自定义设计系统的 Jetsnack
