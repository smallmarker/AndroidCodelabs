## 1\. 简介

![wear_os.png](/images/compose_for_wear_os/wear_os.png)

借助 Compose for Wear OS，您可以将您学到的有关使用 Jetpack Compose 构建应用的知识应用于穿戴式设备。

由于内置了对 Material You 的支持，Compose for Wear OS [可以简化并加快界面开发](https://developer.android.com/jetpack/compose/why-adopt?hl=zh-cn#less-code)，并可帮助您使用更少的代码创建精美的应用。

对于本 Codelab，我们希望您具备一些 Compose 方面的知识，但当然您不必是这方面的专家。

您将创建一些专用于 Wear 的可组合项（简单的和复杂的都有），最后，您可以开始编写适用于 Wear OS 的应用。立即开始吧！

## 学习内容

+   与您之前的 Compose 使用体验有何相似/不同之处
+   简单的可组合项及其在 Wear OS 上的运作方式
+   专用于 Wear OS 的可组合项
+   Wear OS 的 `LazyColumn` (`ScalingLazyColumn)`
+   Wear OS 的 `Scaffold` 版本

## 构建内容

您将构建一个简单的应用，该应用会显示一个可滚动列表，在其中列出已针对 Wear OS 优化的可组合项。

由于您将使用 [`Scaffold`](https://developer.android.com/reference/kotlin/androidx/wear/compose/material/package-summary?hl=zh-cn#Scaffold(androidx.compose.ui.Modifier,kotlin.Function0,kotlin.Function0,kotlin.Function0,kotlin.Function0,kotlin.Function0))，因此您还会在顶部以曲线文本样式显示时间，呈现[晕影效果](https://developer.android.com/reference/kotlin/androidx/wear/compose/material/package-summary?hl=zh-cn#Vignette(androidx.wear.compose.material.VignettePosition,androidx.compose.ui.Modifier))，最后会沿着设备侧边显示滚动指示器。

以下是完成此 Codelab 后，您构建的应用的样子。

![wear_os_scaffold.gif](/images/compose_for_wear_os/wear_os_scaffold.gif)

## 前提条件

+   对 [Android 开发](https://developer.android.com/?hl=zh-cn)有基本的了解
+   对 [Kotlin](https://developer.android.com/kotlin?hl=zh-cn) 有基本的了解
+   具备 [Compose](https://developer.android.com/jetpack/compose?hl=zh-cn) 方面的基础知识

## 2\. 准备设置

在此步骤中，您将设置环境并下载入门级项目。

## 所需条件

+   最新的稳定版 [Android Studio](https://developer.android.com/studio?hl=zh-cn)
+   Wear OS 设备或模拟器（第一次使用？[这里](https://developer.android.com/training/wearables/apps/creating?hl=zh-cn#emulator)介绍了设置方法。）

## 下载代码

如果您已安装 git，则只需运行以下命令即可克隆[此代码库](https://github.com/googlecodelabs/compose-for-wear-os)中的代码。如需检查是否已安装 git，请在终端或命令行中输入 `git --version`，并验证其是否正确执行。

git clone https://github.com/googlecodelabs/compose-for-wear-os.git
cd compose-for-wear-os

如果您未安装 git，可以点击下方按钮下载此 Codelab 的全部代码：

您可以通过更改工具栏中的运行配置，随时在 Android Studio 中运行其中任一模块。

![studio_config.png](/images/compose_for_wear_os/studio_config.png)

## 在 Android Studio 中打开项目

1.  在“Welcome to Android Studio”窗口中，选择  **`[Open an Existing Project]`**
2.  选择文件夹 **`[Download Location]`**。
3.  Android Studio 导入项目后，请测试您可以在 Wear OS 模拟器或实体设备上运行 `start` 和 `finished` 模块。
4.  `start` 模块应如以下屏幕截图所示。您将在其中完成所有工作。

![hello_world.png](/images/compose_for_wear_os/hello_world.png)

## 探索起始代码

+   `build.gradle` 中包含基本的应用配置。它包含创建 Composable Wear OS 应用所需的依赖项。我们将讨论 Jetpack Compose 版本与 Wear OS 版本之间的相似之处和不同之处。
+   `main > AndroidManifest.xml` 包含创建 Wear OS 应用所需的元素。此内容与非 Compose 应用相同，并且与移动应用类似，因此我们不讨论这一点。
+   `main > theme/` 文件夹包含 Compose 针对主题使用的 `Color`、`Type` 和 `Theme` 文件。
+   `main > MainActivity.kt` 包含使用 Compose 创建应用的样板文件。它还包含应用的顶级可组合项（如 `Scaffold` 和 `ScalingLazyList`）。
+   `main > ReusableComponents.kt` 包含适用于我们将创建的大多数 Wear 专用可组合项的函数。我们将在此文件中完成许多任务。

## 3\. 查看依赖项

您所做的大多数与 Wear 相关的依赖项更改都将位于顶级[架构层](https://developer.android.com/jetpack/compose/layering?hl=zh-cn)（下面红色突出显示的部分）。

![material.png](/images/compose_for_wear_os/material.png)

这意味着，当您以 Wear OS 为目标平台时，您已经结合 Jetpack Compose 使用过的许多依赖项不会发生变化。例如，界面、运行时、编译器和动画依赖项将保持不变。

然而，您需要使用正确的 Wear OS **Material**、**Foundation** 和 **Navigation** 库，这些库不同于您之前使用过的库。

我们在下方进行了对比，进一步阐明两者之间的差异：

<table class="vertical-rules"><tbody><tr><td colspan="1" rowspan="1"><p><strong>Wear OS 依赖项</strong> (androidx.wear.*)</p></td><td colspan="1" rowspan="1"><p><strong>比较</strong></p></td><td colspan="1" rowspan="1"><p><strong>标准依赖项</strong> (androidx.*)</p></td></tr><tr><td colspan="1" rowspan="1"><p><a href="https://developer.android.com/reference/kotlin/androidx/wear/compose/material/package-summary?hl=zh-cn" target="_blank">androidx.wear.compose:compose-material</a></p></td><td colspan="1" rowspan="1"><p><strong>代替</strong><em></em></p></td><td colspan="1" rowspan="1"><p>androidx.compose.material:material <strong>₁</strong></p></td></tr><tr><td colspan="1" rowspan="1"><p><a href="https://developer.android.com/reference/kotlin/androidx/wear/compose/navigation/package-summary?hl=zh-cn" target="_blank">androidx.wear.compose:compose-navigation</a></p></td><td colspan="1" rowspan="1"><p><strong>代替</strong><em></em></p></td><td colspan="1" rowspan="1"><p>androidx.navigation:navigation-compose</p></td></tr><tr><td colspan="1" rowspan="1"><p><a href="https://developer.android.com/reference/kotlin/androidx/wear/compose/foundation/package-summary?hl=zh-cn" target="_blank">androidx.wear.compose:compose-foundation</a></p></td><td colspan="1" rowspan="1"><p><strong>加上</strong><em></em></p></td><td colspan="1" rowspan="1"><p>androidx.compose.foundation:foundation</p></td></tr></tbody></table>

1\. 开发者可以继续使用其他与 Material 相关的库，例如借助 Compose Material 库扩展的 Material 涟漪和 Material 图标。

打开 `build.gradle`，在 `start` 模块中搜索“**`TODO: Review Dependencies`**”。（此步骤仅用于检查依赖项，您无需添加任何代码。）

start/build.gradle:

```auto
// TODO: Review Dependencies
// General Compose dependencies
implementation "androidx.activity:activity-compose:$activity_compose_version"
implementation "androidx.compose.ui:ui-tooling-preview:$compose_version"
implementation "androidx.compose.foundation:foundation:$compose_version"
implementation "androidx.compose.material:material-icons-extended:$compose_version"

// Compose for Wear OS Dependencies
implementation "androidx.wear.compose:compose-material:$wear_compose_version"

// Foundation is additive, so you can use the standard version in your Wear OS app.
implementation "androidx.wear.compose:compose-foundation:$wear_compose_version"
```

您应该认识许多常规 Compose 依赖项，因此我们不介绍这些依赖项。

让我们转到 Wear OS 依赖项。

如前所述，此项目仅包含 Wear OS 专用版本的 **`material`** (`androidx.wear.compose:compose-material`)。也就是说，您将不会在项目中看到或使用 `androidx.compose.material:material`。

需要注意的是，您可以将其他 Material 库与 Wear Material 结合使用。在本 Codelab 中，我们实际上将通过添加 `androidx.compose.material:material-icons-extended` 来实现这一点。

最后，我们添加了适用于 Compose 的 Wear **`foundation`** 库 (`androidx.wear.compose:compose-foundation`)。这是附加的，因此您可以将该库与您之前用过的标准 **`foundation`** 搭配使用。实际上，您可能已经意识到，我们已将它添加到常规 Compose 依赖项中！

现在，我们已经了解了依赖项，接下来，我们来看一下主应用。

## 4\. 查看 MainActivity

**我们要在**

**`start`**

**模块完成所有任务，因此请确保打开的每个文件都位于这个模块中。**

首先，打开 `start` 模块中的 `MainActivity`。

这是一个非常简单的类，用于扩展 `ComponentActivity` 并使用 `setContent { WearApp() }` 来创建界面。

根据您之前所掌握的 Compose 方面的知识，您应该已对此很熟悉。我们只需设置界面即可。

向下滚动到 `WearApp()` 可组合函数。在我们谈论代码本身之前，您应该会看到很多 TODO 分散在整个代码中。这些 TODO 分别表示此 Codelab 中的步骤。您可以暂时忽略它们。

代码应如下所示：

**函数 WearApp() 中的代码**：

```auto
WearAppTheme {
    // TODO: Swap to ScalingLazyListState
    val listState = rememberLazyListState()

    /* *************************** Part 4: Wear OS Scaffold *************************** */
    // TODO (Start): Create a Scaffold (Wear Version)

        // Modifiers used by our Wear composables.
        val contentModifier = Modifier.fillMaxWidth().padding(bottom = 8.dp)
        val iconModifier = Modifier.size(24.dp).wrapContentSize(align = Alignment.Center)

        /* *************************** Part 3: ScalingLazyColumn *************************** */
        // TODO: Create a ScalingLazyColumn (Wear's version of LazyColumn)
        LazyColumn(
            modifier = Modifier.fillMaxSize(),
            contentPadding = PaddingValues(
                top = 32.dp,
                start = 8.dp,
                end = 8.dp,
                bottom = 32.dp
            ),
            verticalArrangement = Arrangement.Center,
            state = listState
        ) {

            // TODO: Remove item; for beginning only.
            item { StartOnlyTextComposables() }

            /* ******************* Part 1: Simple composables ******************* */
            item { ButtonExample(contentModifier, iconModifier) }
            item { TextExample(contentModifier) }
            item { CardExample(contentModifier, iconModifier) }

            /* ********************* Part 2: Wear unique composables ********************* */
            item { ChipExample(contentModifier, iconModifier) }
            item { ToggleChipExample(contentModifier) }
        }

    // TODO (End): Create a Scaffold (Wear Version)

}
```

我们先来设置主题 `WearAppTheme { }`。这与您之前编写代码的方式完全相同，也就是说，您可以使用颜色、排版和形状设置 [`MaterialTheme`](https://developer.android.com/jetpack/compose/themes/material?hl=zh-cn)。

然而，对于 Wear OS，我们通常建议使用针对圆形和非圆形设备优化的默认 [Material Wear 形状](https://developer.android.com/reference/kotlin/androidx/wear/compose/material/Shapes?hl=zh-cn)，因此，如果您深入探索 `theme/Theme.kt`，就会发现我们并未替换形状。

如果您愿意，可以打开 `theme/Theme.kt` 做进一步的探索，但同样，它与之前是一样的。

接下来，我们将为要构建的 Wear 可组合项创建一些修饰符，这样我们就不必每次都进行指定。主要是将内容居中并添加一些内边距。

然后，我们创建一个 [`LazyColumn`](https://developer.android.com/jetpack/compose/lists?hl=zh-cn)，用于为一系列项生成垂直滚动的列表（就像您之前做的那样）。

**代码**：

```auto
item { StartOnlyTextComposables() }

/* ******************* Part 1: Simple composables ******************* */
item { ButtonExample(contentModifier, iconModifier) }
item { TextExample(contentModifier) }
item { CardExample(contentModifier, iconModifier) }

/* ********************* Part 2: Wear unique composables ********************* */
item { ChipExample(contentModifier, iconModifier) }
item { ToggleChipExample(contentModifier) }
```

就这些项本身而言，只有 `StartOnlyTextComposables()` 会生成界面。（我们将随着您在 Codelab 中的进展来填充其余内容。）

这些函数实际上位于 `ReusableComponents.kt` 文件中，我们将在下一部分中进行介绍。

让我们先来了解一下 Compose for Wear OS 吧！

## 5\. 添加简单的可组合项

我们将从您可能早已熟悉的三个可组合项（`Button`、`Text` 和 `Card`）开始。

首先，我们将移除 hello world 可组合项。

搜索“**`TODO: Remove item`**”并**清除**注释及其下方的一行：

**第 1 步**

```auto
// TODO: Remove item; for beginning only.
item { StartOnlyTextComposables() }
```

接下来，我们要添加第一个可组合项。

## 创建 Button 可组合项

在 `start` 模块中打开 `ReusableComponents.kt` 并搜索“**`TODO: Create a Button Composable`**”，并将当前的可组合方法替换为以下代码。

**第 2 步**

```auto
// TODO: Create a Button Composable (with a Row to center)
@Composable
fun ButtonExample(
    modifier: Modifier = Modifier,
    iconModifier: Modifier = Modifier
) {
    Row(
        modifier = modifier,
        horizontalArrangement = Arrangement.Center
    ) {
        // Button
        Button(
            modifier = Modifier.size(ButtonDefaults.LargeButtonSize),
            onClick = { /* ... */ },
        ) {
            Icon(
                imageVector = Icons.Rounded.Phone,
                contentDescription = "triggers phone action",
                modifier = iconModifier
            )
        }
    }
}
```

`ButtonExample()` 可组合函数（此代码所在位置）现在会生成一个居中按钮。

我们来了解一下此代码。

在这里，`Row` 仅用于将 [`Button`](https://developer.android.com/reference/kotlin/androidx/wear/compose/material/package-summary?hl=zh-cn#Button(kotlin.Function0,androidx.compose.ui.Modifier,kotlin.Boolean,androidx.wear.compose.material.ButtonColors,androidx.compose.foundation.interaction.MutableInteractionSource,kotlin.Function1)) 可组合项在圆形屏幕上居中。您可以看到，我们通过应用已在 `MainActivity` 中创建的修饰符并将其传递给此函数来执行此操作。之后，当我们在圆形屏幕上滚动时，我们希望确保内容不会被截断（这正是我们要让其居中的原因）。

接下来，我们创建 `Button` 本身。代码与您之前用于 Button 的代码相同，但在本例中，我们使用的是 [`ButtonDefault.LargeButtonSize`](https://developer.android.com/reference/kotlin/androidx/wear/compose/material/ButtonDefaults?hl=zh-cn#LargeButtonSize())。这些都是针对 Wear OS 设备优化过的预设尺寸，因此请务必使用它们！

然后，我们将点击事件设置为空白 lamba。在本例中，这些可组合项仅用于演示，所以我们不需要它们。然而，在实际应用中，我们将通过与 ViewModel 等进行通信以执行业务逻辑。

然后，我们在按钮内设置一个图标。此代码与您之前看到过的 `Icon` 代码相同。此外，我们还要从 `androidx.compose.material:material-icons-extended` 库中获取图标。

最后，我们要设置我们之前为图标设置的修饰符。

运行应用后，您应该会看到如下内容：

![button.png](/images/compose_for_wear_os/button.png)

这些代码您之前可能已经编写过（太棒了）。不同之处在于，您现在可以获取针对 Wear OS 优化的按钮。

很简单，我们再来看看其他代码。

## 创建 Text 可组合项

在 `ReusableComponents.kt` 中，搜索“**`TODO: Create a Text Composable`**”并将当前的可组合方法替换为以下代码。

**第 3 步**

```auto
// TODO: Create a Text Composable
@Composable
fun TextExample(modifier: Modifier = Modifier) {
    Text(
        modifier = modifier,
        textAlign = TextAlign.Center,
        color = MaterialTheme.colors.primary,
        text = stringResource(R.string.device_shape)
    )
}
```

我们创建 [`Text`](https://developer.android.com/reference/kotlin/androidx/wear/compose/material/package-summary?hl=zh-cn#Text(kotlin.String,androidx.compose.ui.Modifier,androidx.compose.ui.graphics.Color,androidx.compose.ui.unit.TextUnit,androidx.compose.ui.text.font.FontStyle,androidx.compose.ui.text.font.FontWeight,androidx.compose.ui.text.font.FontFamily,androidx.compose.ui.unit.TextUnit,androidx.compose.ui.text.style.TextDecoration,androidx.compose.ui.text.style.TextAlign,androidx.compose.ui.unit.TextUnit,androidx.compose.ui.text.style.TextOverflow,kotlin.Boolean,kotlin.Int,kotlin.Function1,androidx.compose.ui.text.TextStyle)) 可组合项，设置其修饰符，对齐文本，设置颜色，最后通过字符串资源设置文本本身。

Compose 开发者应该会非常熟悉 Text 可组合项的外观，其代码实际上与您之前使用的代码完全相同。

我们来看一下它是什么样的：

![text.png](/images/compose_for_wear_os/text.png)

`TextExample()` 可组合函数（我们放置代码的位置）现在会在主 Material 颜色中生成 Text 可组合项。

该字符串是从我们的 `res/values/strings.xml` 文件中拉取的。实际上，如果您查看 `res/values` 文件夹，应该会看到两个 `strings.xml` 资源文件。

Wear OS 是为圆形和非圆形设备提供字符串资源，因此，如果我们在方形模拟器上运行字符串资源，字符串将发生变化：

![square_device.png](/images/compose_for_wear_os/square_device.png)

目前的过程一切顺利。我们再来看看最后一个相似的可组合项 `Card`。

## 创建 Card 可组合项

在 `ReusableComponents.kt` 中，搜索“**`TODO: Create a Card`**”并将当前的可组合方法替换为以下代码。

**第 4 步**

```auto
// TODO: Create a Card (specifically, an AppCard) Composable
@Composable
fun CardExample(
    modifier: Modifier = Modifier,
    iconModifier: Modifier = Modifier
) {
    AppCard(
        modifier = modifier,
        appImage = {
            Icon(
                imageVector = Icons.Rounded.Message,
                contentDescription = "triggers open message action",
                modifier = iconModifier
            )
        },
        appName = { Text("Messages") },
        time = { Text("12m") },
        title = { Text("Kim Green") },
        onClick = { /* ... */ }
    ) {
        Text("On my way!")
    }
}
```

Wear 略有不同，因为有两张主卡片：[`AppCard`](https://developer.android.com/reference/kotlin/androidx/wear/compose/material/package-summary?hl=zh-cn#AppCard(kotlin.Function0,kotlin.Function1,kotlin.Function1,kotlin.Function1,androidx.compose.ui.Modifier,kotlin.Function1,androidx.compose.ui.graphics.painter.Painter,androidx.compose.ui.graphics.Color,androidx.compose.ui.graphics.Color,androidx.compose.ui.graphics.Color,androidx.compose.ui.graphics.Color,kotlin.Function1)) 和 [`TitleCard`](https://developer.android.com/reference/kotlin/androidx/wear/compose/material/package-summary?hl=zh-cn#TitleCard(kotlin.Function0,kotlin.Function1,androidx.compose.ui.Modifier,kotlin.Function1,androidx.compose.ui.graphics.painter.Painter,androidx.compose.ui.graphics.Color,androidx.compose.ui.graphics.Color,androidx.compose.ui.graphics.Color,kotlin.Function1))。

在本例中，我们想在卡片中添加 [`Icon`](https://developer.android.com/reference/kotlin/androidx/wear/compose/material/package-summary?hl=zh-cn#Icon(androidx.compose.ui.graphics.vector.ImageVector,kotlin.String,androidx.compose.ui.Modifier,androidx.compose.ui.graphics.Color))，因此我们将使用 `AppCard`。（`TitleCard` 包含的槽较少，请参阅[卡片](https://developer.android.com/training/wearables/compose/cards?hl=zh-cn)指南以了解有关详情）。

我们创建 `AppCard` 可组合项，设置其修饰符，添加 `Icon`，添加多个 `Text` 可组合项参数（分别用于配置卡片上的不同空间），最后将主要内容文本设置在最后。

我们来看一下它是什么样的：

![appcard.png](/images/compose_for_wear_os/appcard.png)

这时，您可能意识到，这些可组合项的 Compose 代码与您之前用过的代码几乎相同，这很不错吧？您可以重复利用您已获得的所有知识！

好的，我们来看一些新的可组合项。

## 6\. 添加 Wear 专用可组合项

在这一部分中，我们将探索 `Chip` 和 `ToggleChip` 可组合项。

## 创建 Chip 可组合项

[实际上，Material 准则中已指定了 Chip](https://material.io/components/chips)，但标准 Material 库中没有实际的可组合函数。

Chip 是一种一键式快捷操作，对屏幕空间有限的 Wear 设备尤其有用。

下面是 [`Chip`](https://developer.android.com/reference/kotlin/androidx/wear/compose/material/package-summary?hl=zh-cn#Chip(kotlin.Function0,androidx.wear.compose.material.ChipColors,androidx.compose.ui.Modifier,kotlin.Boolean,androidx.compose.foundation.layout.PaddingValues,androidx.compose.ui.graphics.Shape,androidx.compose.foundation.interaction.MutableInteractionSource,androidx.compose.ui.semantics.Role,kotlin.Function1)) 可组合函数的几种变体，您可以通过它们了解您可以创建什么：

![chip.png](/images/compose_for_wear_os/chip.png)

让我们开始编写代码吧。

在 `ReusableComponents.kt` 中，搜索“**`TODO: Create a Chip`**”并将当前的可组合方法替换为以下代码。

**第 5 步**

```auto
// TODO: Create a Chip Composable
@Composable
fun ChipExample(
    modifier: Modifier = Modifier,
    iconModifier: Modifier = Modifier
) {
    Chip(
        modifier = modifier,
        onClick = { /* ... */ },
        label = {
            Text(
                text = "5 minute Meditation",
                maxLines = 1,
                overflow = TextOverflow.Ellipsis
            )
        },
        icon = {
            Icon(
                imageVector = Icons.Rounded.SelfImprovement,
                contentDescription = "triggers meditation action",
                modifier = iconModifier
            )
        },
    )
}
```

`Chip` 可组合项所用的许多参数都与其他可组合项（修饰符和 `onClick`）所用的参数相同，因此我们无需查看这些参数。

此外，它还有一个标签（我们为其创建一个 `Text` 可组合项）和一个图标。

`Icon` 代码应该与您在其他可组合项中看到的代码完全相同，但对于此代码，我们将从 `androidx.compose.material:material-icons-extended` 库中提取 `Self Improvement` 图标。

让我们来看一下它是什么样的（注意向下滚动）：

![self_Improvement.png](/images/compose_for_wear_os/self_Improvement.png)

我们来看一下 `Toggle` 的一个变体，即 `ToggleChip` 可组合项。

## 创建 ToggleChip 可组合项

[`ToggleChip`](https://developer.android.com/reference/kotlin/androidx/wear/compose/material/package-summary?hl=zh-cn#ToggleChip(kotlin.Boolean,kotlin.Function1,kotlin.Function1,kotlin.Function0,androidx.compose.ui.Modifier,kotlin.Function1,kotlin.Function1,androidx.wear.compose.material.ToggleChipColors,kotlin.Boolean,androidx.compose.foundation.interaction.MutableInteractionSource,androidx.compose.foundation.layout.PaddingValues,androidx.compose.ui.graphics.Shape)) 与 `Chip` 类似，但可使用户与单选按钮、切换开关或复选框进行交互。

![toggle_chip.png](/images/compose_for_wear_os/toggle_chip.png)

在 `ReusableComponents.kt` 中，搜索“**`TODO: Create a ToggleChip`**”并将当前的可组合方法替换为以下代码。

**第 6 步**

```auto
// TODO: Create a ToggleChip Composable
@Composable
fun ToggleChipExample(modifier: Modifier = Modifier) {
    var checked by remember { mutableStateOf(true) }
    ToggleChip(
        modifier = modifier,
        checked = checked,
        toggleControl = {
            Switch(
                checked = checked,
                modifier = Modifier.semantics {
                    this.contentDescription = if (checked) "On" else "Off"
                }
            )
        },
        onCheckedChange = {
            checked = it
        },
        label = {
            Text(
                text = "Sound",
                maxLines = 1,
                overflow = TextOverflow.Ellipsis
            )
        }
    )
}
```

现在，`ToggleChipExample()` 可组合函数（此代码所在位置）使用切换开关（而不是复选框或单选按钮）生成 `ToggleChip`。

首先，我们创建一个 `MutableState`。由于其他功能主要用于提供界面演示，方便您了解 Wear 提供哪些功能，因此我们尚未在其他函数中执行此操作。

在普通应用中，您可能希望传递选中状态以及处理点按的 lambda，因此可组合项可以是无状态的（[点击此处了解详情](https://developer.android.com/jetpack/compose/state?hl=zh-cn#stateful-vs-stateless)）。

在本例中，我们只是使用操作切换开关来简单展示 `ToggleChip` 的实际效果（即使我们不对状态执行任何操作）。

接下来，我们设置修饰符、选中状态和切换开关控件，为我们提供所需的开关。

然后，我们创建一个 lambda 来改变状态，最后使用 `Text` 可组合项（以及一些基本参数）设置标签。

我们来看一下它是什么样的：

![toogle_chip_text.png](/images/compose_for_wear_os/toogle_chip_text.png)

现在，您已经看到了许多特定于 Wear OS 的可组合项，并且如前所述，大多数代码与您之前编写的代码几乎相同。

我们再来看一些更高级的代码。

## 7\. 迁移到 ScalingLazyColumn

您可能在移动应用中使用了 [`LazyColumn`](https://developer.android.com/jetpack/compose/lists?hl=zh-cn#lazy) 来生成垂直滚动列表。

由于圆形设备的顶部和底部较小，因此显示内容的空间较小。因此，Wear OS 有自己的 `LazyColumn` 版本，可以更好地支持这些圆形设备。

[`ScalingLazyColumn`](https://developer.android.com/reference/kotlin/androidx/wear/compose/material/package-summary?hl=zh-cn#ScalingLazyColumn(androidx.compose.ui.Modifier,androidx.wear.compose.material.ScalingLazyListState,androidx.compose.foundation.layout.PaddingValues,kotlin.Boolean,androidx.compose.foundation.layout.Arrangement.Vertical,androidx.compose.ui.Alignment.Horizontal,androidx.compose.foundation.gestures.FlingBehavior,kotlin.Boolean,androidx.wear.compose.material.ScalingParams,androidx.wear.compose.material.ScalingLazyListAnchorType,androidx.wear.compose.material.AutoCenteringParams,kotlin.Function1)) 对 `LazyColumn` 进行了扩展，以支持屏幕顶部和底部的缩放和透明度，使内容更易于用户阅读。

下面是一个演示：

![LazyColumn.gif](/images/compose_for_wear_os/LazyColumn.gif)

注意，当红色对象靠近中心位置时，会放大到完整尺寸，然后随着移开而缩小（并且变得更透明）。

以下是一个来自应用的具体示例：

![lazycolumn_example.gif](/images/compose_for_wear_os/lazycolumn_example.gif)

我们发现这确实有助于提高可读性。

现在，您已了解了 [`ScalingLazyColumn`](https://developer.android.com/reference/kotlin/androidx/wear/compose/material/package-summary?hl=zh-cn#ScalingLazyColumn(androidx.compose.ui.Modifier,androidx.wear.compose.material.ScalingLazyListState,androidx.compose.foundation.layout.PaddingValues,kotlin.Boolean,androidx.compose.foundation.layout.Arrangement.Vertical,androidx.compose.ui.Alignment.Horizontal,androidx.compose.foundation.gestures.FlingBehavior,kotlin.Boolean,androidx.wear.compose.material.ScalingParams,androidx.wear.compose.material.ScalingLazyListAnchorType,androidx.wear.compose.material.AutoCenteringParams,kotlin.Function1)) 的实际应用，接下来让我们开始转换 `LazyColumn`。

## 转换为 ScalingLazyListState

在 `MainActivity.kt` 中，搜索“**`TODO: Swap to ScalingLazyListState`**”并将下面的注释和行替换为以下代码。

**第 7 步**

```auto
// TODO: Swap to ScalingLazyListState
val listState = rememberScalingLazyListState()
```

名称几乎完全相同，只是减去了“缩放”部分。就像 `LazyListState` 处理 `LazyColumn` 的状态一样，`ScalingLazyListState` 也处理 `ScalingLazyColumn` 的状态。

## 转换为 ScalingLazyColumn

接下来，我们换入 `ScalingLazyColumn`。

在 `MainActivity.kt` 中，搜索“**`TODO: Swap a ScalingLazyColumn`**”。首先，将 `LazyColumn` 替换为 `ScalingLazyColumn`。

然后，将 `contentPadding` 和 `verticalArrangement` 一起移除。`ScalingLazyColumn` 已提供默认设置，可保证实现更好的默认视觉效果，因为大多数视口会填充列表项。在许多情况下，使用默认形参就足够了，如果您在顶部有标题，我们建议您将其作为第一项放入 [`ListHeader`](https://developer.android.com/reference/kotlin/androidx/wear/compose/material/package-summary?hl=zh-cn#ListHeader(androidx.compose.ui.Modifier,androidx.compose.ui.graphics.Color,androidx.compose.ui.graphics.Color,kotlin.Function1)) 中。否则，请考虑将 `autoCentering` 的 `itemIndex` 设为 0，从而为第一项提供足够的内边距。

**第 8 步**

```auto
// TODO: Swap a ScalingLazyColumn (Wear's version of LazyColumn)
ScalingLazyColumn(
    modifier = Modifier.fillMaxSize(),
    autoCentering = AutoCenteringParams(itemIndex = 0),
    state = listState
```

大功告成！我们来看一下它是什么样的：

![lazycolumn_complete.png](/images/compose_for_wear_os/lazycolumn_complete.png)

当您滚动屏幕时，可以看到内容已缩放，且透明度在屏幕的顶部和底部经过调整，只需执行极少工作即可完成迁移！

在上下移动冥想可组合项时，您可以真正观察它。

现在谈到最后一个主题，Wear OS 的 `Scaffold`。

## 8\. 添加 Scaffold

[`Scaffold`](https://developer.android.com/reference/kotlin/androidx/wear/compose/material/package-summary?hl=zh-cn#Scaffold(androidx.compose.ui.Modifier,kotlin.Function0,kotlin.Function0,kotlin.Function0,kotlin.Function0,kotlin.Function0)) 提供了一种布局结构，该布局可帮助您按常见模式排列屏幕，就像在移动设备上显示一样，但不是[应用栏、FAB、抽屉式导航栏或其他移动设备专用元素](https://developer.android.com/jetpack/compose/layouts/material?hl=zh-cn#scaffold)这样的布局，它支持包含顶级组件的四种 Wear 专用布局：时间、晕影效果、滚动/位置指示器和页面指示器。

它还可以处理圆形和非圆形设备。

这些信息大致是这样显示的：

| ![time_text.png](/images/compose_for_wear_os/time_text.png) | ![vignette.png](/images/compose_for_wear_os/vignette.png) | ![positionIndicator.png](/images/compose_for_wear_os/PositionIndicator.png) | ![PageIndicator.png](/images/compose_for_wear_os/PageIndicator.png) |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| [`TimeText`](https://developer.android.com/reference/kotlin/androidx/wear/compose/material/package-summary?hl=zh-cn#TimeText(androidx.compose.ui.Modifier,androidx.wear.compose.material.TimeSource,androidx.compose.ui.text.TextStyle,androidx.compose.foundation.layout.PaddingValues,kotlin.Function0,kotlin.Function1,kotlin.Function0,kotlin.Function1,kotlin.Function0,kotlin.Function1)) | [`Vignette`](https://developer.android.com/reference/kotlin/androidx/wear/compose/material/package-summary?hl=zh-cn#Vignette(androidx.wear.compose.material.VignettePosition,androidx.compose.ui.Modifier)) | [`PositionIndicator`](https://developer.android.com/reference/kotlin/androidx/wear/compose/material/package-summary?hl=zh-cn#PositionIndicator(androidx.compose.foundation.ScrollState,androidx.compose.ui.Modifier,kotlin.Boolean)) | [`PageIndicator`](https://developer.android.google.cn/reference/kotlin/androidx/wear/compose/material/package-summary?hl=zh-cn#HorizontalPageIndicator(androidx.wear.compose.material.PageIndicatorState,androidx.compose.ui.Modifier,androidx.wear.compose.material.PageIndicatorStyle,androidx.compose.ui.graphics.Color,androidx.compose.ui.graphics.Color,androidx.compose.ui.unit.Dp,androidx.compose.ui.unit.Dp,androidx.compose.ui.graphics.Shape)) |

我们将详细介绍前三个组件，不过，我们首先搭建 Scaffold。

## 添加 Scaffold

现在，我们来为 `Scaffold` 添加样板文件。

找到“**`TODO (Start): Create a Scaffold (Wear Version)`**”并在其下方添加以下代码。

**第 9 步**

```auto
// TODO (Start): Create a Scaffold (Wear Version)
Scaffold(
    timeText = { },
    vignette = { },
    positionIndicator = { }
) {
```

我们会在后续步骤中依次介绍各个参数。目前，我们不会生成任何界面。

接下来，请务必将右括号添加到正确的位置。找到“**`TODO (End): Create a Scaffold (Wear Version)`**”并添加相应的括号：

**第 10 步**

```auto
// TODO (End): Create a Scaffold (Wear Version)
}
```

我们先运行该代码。您应会看到类似下图的内容：

![round_device.png](/images/compose_for_wear_os/round_device.png)

请注意，它与之前添加 Scaffold 的行为没有任何实际区别，但一旦开始实现组件，就会发生变化。

我们先从三个参数 `TimeText` 中的第一个参数开始。

## TimeText

[`TimeText`](https://developer.android.com/reference/kotlin/androidx/wear/compose/material/package-summary?hl=zh-cn#TimeText(androidx.compose.ui.Modifier,androidx.wear.compose.material.TimeSource,androidx.compose.ui.text.TextStyle,androidx.compose.foundation.layout.PaddingValues,kotlin.Function0,kotlin.Function1,kotlin.Function0,kotlin.Function1,kotlin.Function0,kotlin.Function1)) 在后台使用曲线文本样式，让开发者可以轻松显示时间，而无需放置可组合项或与时间相关的类执行任何操作。

在应用中任何屏幕的顶部，[Material 准则都要求您显示时间](https://developer.android.com/training/wearables/design/overlays?hl=zh-cn#time)。看起来应如下所示：

![play_list.png](/images/compose_for_wear_os/play_list.png)

添加 `TimeText` 实际上很简单。

找到“**`timeText = { },`**”并将其替换为以下代码：

**第 11 步**

```auto
timeText = {
    TimeText(modifier = Modifier.scrollAway(listState))
},
```

首先，我们创建一个 `TimeText` 可组合项。我们可以[添加额外参数](https://developer.android.com/reference/kotlin/androidx/wear/compose/material/package-summary?hl=zh-cn#TimeText(androidx.compose.ui.Modifier,androidx.wear.compose.material.TimeSource,androidx.compose.ui.text.TextStyle,androidx.compose.foundation.layout.PaddingValues,kotlin.Function0,kotlin.Function1,kotlin.Function0,kotlin.Function1,kotlin.Function0,kotlin.Function1))以在时间之前和/或之后添加文本，但我们希望这尽可能简单。

使用列表等可滚动元素创建 `TimeText` 时，如果用户开始向上滚动项列表，`TimeText` 应该淡出视图。为此，我们将添加 `Modifier.scrollAway`，以便根据滚动状态使 `TimeText` 垂直滚动进入/离开视图。

请尝试运行它。现在，您应该可以看到时间；如果滚动，时间将淡出。

![time_text_1.png](/images/compose_for_wear_os/time_text_1.png)

接下来，我们将介绍 `Vignette`。

## 添加晕影效果

当显示可滚动屏幕时，[`Vignette`](https://developer.android.com/reference/kotlin/androidx/wear/compose/material/package-summary?hl=zh-cn#Vignette(androidx.wear.compose.material.VignettePosition,androidx.compose.ui.Modifier)) 会对穿戴式设备屏幕的上边缘和下边缘进行模糊处理。

开发者可根据用例指定对顶部和/或底部进行模糊处理。

示例如下：

![vignette_1.png](/images/compose_for_wear_os/vignette_1.png)

我们只有一个屏幕（可滚动屏幕），所以我们希望展示此晕影效果，以提高可读性。现在我们就开始吧。

找到“**`vignette = { },`**”并将其替换为以下代码：

**第 12 步**

```auto
vignette = {
    // Only show a Vignette for scrollable screens. This code lab only has one screen,
    // which is scrollable, so we show it all the time.
    Vignette(vignettePosition = VignettePosition.TopAndBottom)
},
```

如需详细了解何时应该以及不应该展示晕影效果，请阅读相关评论。在本例中，我们希望始终显示晕影效果，并且希望对屏幕的顶部和底部进行模糊处理。

![vignette_2.png](/images/compose_for_wear_os/vignette_2.png)

查看顶部或底部（尤其是在紫色可组合项上）时，您应该会看到效果。

结束 `Scaffold` 的最后一个参数 `PositionIndicator`。

## 添加 PositionIndicator

[`PositionIndicator`](https://developer.android.com/reference/kotlin/androidx/wear/compose/material/package-summary?hl=zh-cn#PositionIndicator(androidx.wear.compose.material.ScalingLazyListState,androidx.compose.ui.Modifier,kotlin.Boolean))（也称为滚动指示器）是屏幕右侧的指示符，用于根据您传入的状态对象类型显示当前指示符的位置。在本例中是 [`ScalingLazyListState`](https://developer.android.com/reference/kotlin/androidx/wear/compose/material/ScalingLazyListState?hl=zh-cn)。

示例如下：

![positionIndicator_1.png](/images/compose_for_wear_os/positionIndicator_1.png)

您可能会好奇，为什么位置指示器需要处于 `Scaffold` 级别，而不是 `ScalingLazyColumn` 级别。

由于屏幕是弧形的，因此位置指示器需要位于表盘中央 (`Scaffold`)，而不仅仅是在视口中央 (`ScalingLazyColumn`)。否则，指示器可能会被截断。

例如，在下方应用中，我们可以假设“Playlist”可组合项不在可滚动的区域内。位置指示器位于 `ScalingLazyColumn` 中央，但未占据整个屏幕。因此，您会发现位置指示器大部分被截断了。

![positionIndicator_2.png](/images/compose_for_wear_os/positionIndicator_2.png)

不过，如果我们改为将位置指示器放在整个可见界面的中央（`Scaffold` 为您提供的位置），您就可以清楚地看到它。

![positionIndicator_3.png](/images/compose_for_wear_os/positionIndicator_3.png)

这表明 `PositionIndicator` 需要 `ScalingLazyListState`（用于告知指示器其在滚动列表中的位置）位于 `Scaffold` 的上方。

如果您特别善于观察，可能已注意到[我们已经抬高了](https://developer.android.com/jetpack/compose/state?hl=zh-cn#state-hoisting) `Scaffold` 上方的 `ScalingLazyListState` 以隐藏/显示时间；因此，在此示例中，我们已经采取了此做法。

现在，我们已经介绍了位置指示器应位于 Scaffold 中的原因，下面我们将其添加到我们的应用中。

找到“**`positionIndicator = { }`**”并将其替换为以下代码：

**第 12 步**

```auto
positionIndicator = {
    PositionIndicator(
        scalingLazyListState = listState
    )
}
```

很简单，`PositionIndicator` 需要滚动状态才能正确渲染，现在它能实现！

它还具有一项很好的功能，即可以在用户未滚动时自行隐藏。

我们使用的是 `ScalingLazyListState`，但 `PositionIndicator` 有许多其他滚动选项，例如 `ScrollState`、`LazyListState`，甚至能够处理旋转的侧面按钮或

旋转边框。要查看所有选项，请[查看每个版本的 KDocs](https://developer.android.com/reference/kotlin/androidx/wear/compose/material/package-summary?hl=zh-cn#PositionIndicator(androidx.compose.foundation.ScrollState,androidx.compose.ui.Modifier,kotlin.Boolean))。

现在，我们来预览一下效果如何：

![positionIndicator_4.png](/images/compose_for_wear_os/positionIndicator_4.png)

请尝试上下滚动。您应该只会在滚动时看到滚动指示器显示。

太棒了，您已完成了大多数 Wear OS 可组合项的界面演示！

## 9\. 恭喜

恭喜！您已学习了在 Wear OS 上使用 Compose 的基础知识！

现在，您可以重新应用您掌握的所有 Compose 知识，打造精美的 Wear OS 应用！

## 后续操作

查看其他 Wear OS Codelab：

+   [创建您的第一个 Wear OS 卡片](https://developer.android.com/codelabs/wear-tiles?hl=zh-cn)
+   [利用 Ongoing Activity API，以全新方式吸引 Wear 用户](https://developer.android.com/codelabs/ongoing-activity?hl=zh-cn)

## 深入阅读

+   [Compose for Wear OS 现推出 1.0 版](https://android-developers.googleblog.com/2022/07/compose-for-wear-os-10-stable.html)博文
+   GitHub 上的 Compose for Wear OS 的[简单示例和复杂示例](https://github.com/android/wear-os-samples#wear-os-samples-repository)
+   如需更多指南，请参阅[借助 Wear OS 开发适用于手表的应用](https://developer.android.com/wear?hl=zh-cn)
