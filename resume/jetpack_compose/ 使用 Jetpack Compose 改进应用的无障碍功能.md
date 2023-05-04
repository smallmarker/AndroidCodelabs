## 1\. 简介

在此 Codelab 中，您将学习如何使用 Jetpack Compose 改进应用的无障碍功能。我们将详细介绍几个常见的用例，并逐步改进一个示例应用。我们将介绍触摸目标大小、内容描述、点击标签，等等。

视力受损、色盲、听力受损、精细动作失能的人、以及有认知障碍和许多其他残疾的人可以使用 Android 设备来处理他们日常生活中的各种事务。如果您能够在开发应用时考虑无障碍功能，那么您便可以改善用户体验，对具有这些需求以及其他无障碍功能需求的用户来说尤其如此。

在此 Codelab 的学习过程中，我们将使用 [TalkBack](https://support.google.com/accessibility/android/answer/6283677?hl=zh-cn) 手动测试代码更改。TalkBack 是一项主要供视力受损的人使用的无障碍服务。此外，请确保使用其他无障碍服务（例如[开关控制](https://support.google.com/accessibility/android/answer/6122836?hl=zh-cn)）测试对代码所做的任何更改。

![在 Jetnews 的主屏幕上移动的 TalkBack 焦点矩形。TalkBack 读出的文字显示在屏幕的底部。](/images/compose_barrier/1.gif)

TalkBack 在 Jetnews 应用中的实际运用。

## 学习内容

在此 Codelab 中，您将学习：

+   如何通过增大触摸目标大小来辅助精细动作失能的用户。
+   什么是语义属性以及如何更改语义属性。
+   如何向可组合项提供信息，让其使用起来更便捷。

## 所需条件

+   有使用 Kotlin 语法（包括 lambda）的经验。
+   有使用 Compose 的基本经验。建议您在学习此 Codelab 之前先学习 [Jetpack Compose 基础知识 Codelab](https://codelabs.developers.google.com/codelabs/jetpack-compose-basics/?hl=zh-cn)。
+   [启用了 TalkBack](https://support.google.com/accessibility/android/answer/6007100?hl=zh-cn) 的 Android 设备或模拟器。

## 构建内容

在此 Codelab 中，我们将改进一个新闻阅读应用的无障碍功能。我们将从一个缺少重要无障碍功能的应用入手，运用我们所学的知识来使我们的应用更适合需要无障碍功能的人使用。

## 2\. 准备工作

在此步骤中，您将下载此 Codelab 的代码，其中包含一个简单的新闻阅读器应用。

## 所需条件

+   [Android Studio Dolphin](https://developer.android.com/studio?hl=zh-cn)

## 获取代码

此 Codelab 的代码可以在 [android-compose-codelabs GitHub 代码库](https://github.com/googlecodelabs/android-compose-codelabs)中找到。如需克隆该代码库，请运行以下命令：

$ git clone https://github.com/googlecodelabs/android-compose-codelabs

或者，您也可以下载两个 ZIP 文件：

+   [起点](https://github.com/googlecodelabs/android-compose-codelabs/archive/refs/heads/main.zip)
+   [解决方案](https://github.com/googlecodelabs/android-compose-codelabs/archive/refs/heads/end.zip)

## 查看示例应用

您刚刚下载的代码包含提供的所有 Compose Codelab 的代码。为了完成此 Codelab，请在 Android Studio 中打开 **`AccessibilityCodelab`** 项目。

我们建议您从 `main` 分支中的代码着手，按照自己的节奏逐步完成此 Codelab。

## 设置 TalkBack

在此 Codelab 的学习过程中，我们将使用 TalkBack 来检查我们所做的更改。当您使用实体设备进行测试时，请按照[这些说明来开启 TalkBack](https://support.google.com/accessibility/android/answer/6007100?hl=zh-cn)。默认情况下，模拟器未安装 TalkBack。请选择包含 Play 商店的模拟器，并[下载 Android 无障碍套件](https://play.google.com/store/apps/details?id=com.google.android.marvin.talkback&hl=zh-cn)。

## 3\. 触摸目标大小

屏幕上可供用户点击、触摸或以其他方式互动的所有元素都应足够大，让用户能够进行可靠的互动。您应确保这些元素的**宽度和高度至少为 48dp**。

如果这些控件的大小会动态地调整，或者根据其内容的大小进行调整，那么不妨考虑使用 [`sizeIn`](https://developer.android.com/reference/kotlin/androidx/compose/foundation/layout/package-summary?hl=zh-cn#(androidx.compose.ui.Modifier).sizeIn(androidx.compose.ui.unit.Dp,androidx.compose.ui.unit.Dp,androidx.compose.ui.unit.Dp,androidx.compose.ui.unit.Dp)) 修饰符为其尺寸设置下限。

某些 Material 组件会为您设置这些大小。例如，Button 可组合项的 [`MinHeight`](https://developer.android.com/reference/kotlin/androidx/compose/material/ButtonDefaults?hl=zh-cn#MinHeight()) 设置为 36dp，并使用 8dp 的垂直内边距。这加起来就是要求的 48dp 高度。

当我们打开示例应用并运行 TalkBack 时，我们会注意到，文章卡片中叉号图标的触摸目标非常小。我们希望此触摸目标至少为 48dp。

在下面的屏幕截图中，左侧是原始应用，而右侧是改进的解决方案。

![比较一个列表项，左侧显示的是叉号图标的小轮廓，右侧显示的是大轮廓。](/images/compose_barrier/2.png)

让我们来看看相应的实现，看一下此可组合项的大小。打开 `PostCards.kt` 并查找 `PostCardHistory` 可组合项。如您所见，该实现将溢出菜单图标的大小设置为 24dp：

```auto
@Composable
fun PostCardHistory(post: Post, navigateToArticle: (String) -> Unit) {
   // ...

   Row(
       // ...
   ) {
       // ...
       CompositionLocalProvider(LocalContentAlpha provides ContentAlpha.medium) {
           Icon(
               imageVector = Icons.Default.Close,
               contentDescription = stringResource(R.string.cd_show_fewer),
               modifier = Modifier
                   .clickable { openDialog = true }
                   .size(24.dp)
           )
       }
   }
   // ...
}
```

如需增大此 `Icon` 的触摸目标大小，我们可以加上内边距：

```auto
@Composable
fun PostCardHistory(post: Post, navigateToArticle: (String) -> Unit) {
   // ...
   Row(
       // ...
   ) {
       // ...
       CompositionLocalProvider(LocalContentAlpha provides ContentAlpha.medium) {
           Icon(
               imageVector = Icons.Default.Close,
               contentDescription = stringResource(R.string.cd_show_fewer),
               modifier = Modifier
                   .clickable { openDialog = true }
                   .padding(12.dp)
                   .size(24.dp)
           )
       }
   }
   // ...
}
```

在我们的用例中，有一种更简单的方法来确保触摸目标至少为 48dp。我们可以使用 Material 组件 `IconButton`，该组件将为我们处理此问题：

```auto
@Composable
fun PostCardHistory(post: Post, navigateToArticle: (String) -> Unit) {
   // ...
   Row(
       // ...
   ) {
       // ...
       CompositionLocalProvider(LocalContentAlpha provides ContentAlpha.medium) {
           IconButton(onClick = { openDialog = true }) {
               Icon(
                   imageVector = Icons.Default.Close,
                   contentDescription = stringResource(R.string.cd_show_fewer)
               )
           }
       }
   }
   // ...
}
```

使用 TalkBack 浏览屏幕时，现在会正确显示 48dp 的触摸目标区域。此外，`IconButton` 还添加了涟漪效果，向用户表明该元素是可点击的。

## 4\. 点击标签

默认情况下，应用中的可点击元素不会提供任何关于该元素会在点击时做什么的信息。因此，TalkBack 等无障碍服务将会使用非常宽泛的默认描述。

为了向具有无障碍功能需求的用户提供最佳体验，我们可以提供具体的描述，解释当用户点击此元素时会发生什么情况。

在 Jetnews 应用中，用户可点击各个文章卡片以阅读整篇文章。默认情况下，这样将读出可点击元素的内容，紧接着是文字“Double tap to activate”（点按两次即可激活）。我们希望更具体，使用“Double tap to read article”（点按两次即可阅读文章）。下面是原始版本与理想解决方案的比较：

![在启用了 TalkBack 的情况下录制的两个屏幕 - 点按垂直列表中的文章以及水平轮播界面中的文章。](/images/compose_barrier/3_1.gif)

更改可组合项的点击标签。之前（左侧）与之后（右侧）的比较。

`clickable` 修饰符包含一个参数，可让您直接设置此点击标签。

让我们再来看看 `PostCardHistory` 实现：

```auto
@Composable
fun PostCardHistory(
   // ...
) {
   Row(
       Modifier.clickable { navigateToArticle(post.id) }
   ) {
       // ...
   }
}
```

如您所见，此实现使用了 `clickable` 修饰符。如需设置点击标签，我们可以设置 `onClickLabel` 参数：

```auto
@Composable
fun PostCardHistory(
   // ...
) {
   Row(
       Modifier.clickable(
               // R.string.action_read_article = "read article"
               onClickLabel = stringResource(R.string.action_read_article)
           ) {
               navigateToArticle(post.id)
           }
   ) {
       // ...
   }
}
```

TalkBack 现在会正确读出“Double tap to **read article**”。

主屏幕上的其他文章卡片具有相同的通用点击标签。让我们来看看 `PostCardPopular` 可组合项的实现并更新其点击标签：

```auto
@Composable
fun PostCardPopular(
   // ...
) {
   Card(
       shape = MaterialTheme.shapes.medium,
       modifier = modifier.size(280.dp, 240.dp),
       onClick = { navigateToArticle(post.id) }
   ) {
       // ...
   }
}
```

此可组合项在内部使用 `Card` 可组合项，后者不允许您直接设置点击标签。您可以改用 `semantics` 修饰符设置点击标签：

```auto
@OptIn(ExperimentalMaterialApi::class)
@Composable
fun PostCardPopular(
   post: Post,
   navigateToArticle: (String) -> Unit,
   modifier: Modifier = Modifier
) {
   val readArticleLabel = stringResource(id = R.string.action_read_article)
   Card(
       shape = MaterialTheme.shapes.medium,
       modifier = modifier
          .size(280.dp, 240.dp)
          .semantics { onClick(label = readArticleLabel, action = null) },
       onClick = { navigateToArticle(post.id) }
   ) {
       // ...
   }
}
```

## 5\. 自定义操作

许多应用都会显示某种列表，列表中的每一项都包含一项或多项操作。使用屏幕阅读器时，浏览此类列表可能会变得单调乏味，因为相同的操作会被反复聚焦。

我们可以向可组合项添加自定义无障碍操作。这样，就可以将与同一列表项相关的操作归为一组。

在 Jetnews 应用中，我们显示用户可以阅读的文章列表。每个列表项都包含一项操作，指明用户希望少看到此主题。在本部分中，我们会将此操作移至一项自定义无障碍操作，这样浏览列表就会变得更容易。

左侧显示的是默认情况，其中每个叉号图标都可聚焦。右侧显示的是解决方案，其中该操作包含在 TalkBack 的自定义操作中：

![在启用了 TalkBack 的情况下录制的两个屏幕。左侧的屏幕显示文章项上的叉号图标是如何可选择的。点按两次可打开一个对话框。右侧的屏幕显示使用点按三次手势打开一个自定义“Actions”菜单。当用户点按“Show fewer of this”操作时，系统会打开相同的对话框。](/images/compose_barrier/4_1.gif)

向文章项添加自定义操作。之前（左侧）与之后（右侧）的比较。

让我们打开 `PostCards.kt` 并查看 `PostCardHistory` 可组合项的实现。请注意使用 `Modifier.clickable` 和 `onClick` 的 `Row` 和 `IconButton` 的可点击属性：

```auto
@Composable
fun PostCardHistory(post: Post, navigateToArticle: (String) -> Unit) {
   // ...
   Row(
       Modifier.clickable(
           onClickLabel = stringResource(R.string.action_read_article)
       ) {
           navigateToArticle(post.id)
       }
   ) {
       // ...
       CompositionLocalProvider(LocalContentAlpha provides ContentAlpha.medium) {
           IconButton(onClick = { openDialog = true }) {
               Icon(
                   imageVector = Icons.Default.Close,
                   contentDescription = stringResource(R.string.cd_show_fewer)
               )
           }
       }
   }
   // ...
}
```

默认情况下，`Row` 和 `IconButton` 可组合项都可点击，因此都会由 TalkBack 聚焦。列表中的每一项都是如此，这意味着，在浏览列表时需要进行大量的滑动。我们希望与 `IconButton` 相关的操作作为一项自定义操作包含在列表项中。我们可以使用 `clearAndSetSemantics` 修饰符来告知无障碍服务不要与此 `Icon` 进行互动：

```auto
@Composable
fun PostCardHistory(post: Post, navigateToArticle: (String) -> Unit) {
   // ...
   Row(
       Modifier.clickable(
           onClickLabel = stringResource(R.string.action_read_article)
       ) {
           navigateToArticle(post.id)
       }
   ) {
       // ...
       CompositionLocalProvider(LocalContentAlpha provides ContentAlpha.medium) {
            IconButton(
                modifier = Modifier.clearAndSetSemantics { },
                onClick = { openDialog = true }
            ) {
                Icon(
                    imageVector = Icons.Default.Close,
                    contentDescription = stringResource(R.string.cd_show_fewer)
                )
            }
       }
   }
   // ...
}
```

不过，通过移除 `IconButton` 的语义，现在无法再执行该操作了。我们可以将该操作添加到列表项中，方法是在 `semantics` 修饰符中添加一项自定义操作：

```auto
@Composable
fun PostCardHistory(post: Post, navigateToArticle: (String) -> Unit) {
   // ...
   val showFewerLabel = stringResource(R.string.cd_show_fewer)
   Row(
        Modifier
            .clickable(
                onClickLabel = stringResource(R.string.action_read_article)
            ) {
                navigateToArticle(post.id)
            }
            .semantics {
                customActions = listOf(
                    CustomAccessibilityAction(
                        label = showFewerLabel,
                        // action returns boolean to indicate success
                        action = { openDialog = true; true }
                    )
                )
            }
   ) {
       // ...
       CompositionLocalProvider(LocalContentAlpha provides ContentAlpha.medium) {
            IconButton(
                modifier = Modifier.clearAndSetSemantics { },
                onClick = { openDialog = true }
            ) {
                Icon(
                    imageVector = Icons.Default.Close,
                    contentDescription = stringResource(R.string.cd_show_fewer)
                )
            }
       }
   }
   // ...
}
```

现在，我们可以使用 TalkBack 中的自定义操作弹出菜单来应用该操作。随着列表项中的操作数不断增加，这将变得越来越有意义。

## 6\. 视觉元素描述

并不是应用的每个用户都能够看到或解释应用中显示的视觉元素，如图标和插图。无障碍服务也无法仅仅根据视觉元素的像素来弄清楚这些元素的意思。这使得开发者有必要将有关应用中的视觉元素的更多信息传递给无障碍服务。

[`Image`](https://developer.android.com/reference/kotlin/androidx/compose/foundation/package-summary?hl=zh-cn#Image(androidx.compose.ui.graphics.vector.ImageVector,kotlin.String,androidx.compose.ui.Modifier,androidx.compose.ui.Alignment,androidx.compose.ui.layout.ContentScale,kotlin.Float,androidx.compose.ui.graphics.ColorFilter)) 和 [`Icon`](https://developer.android.com/reference/androidx/wear/compose/material/IconKt?hl=zh-cn#Icon(androidx.compose.ui.graphics.vector.ImageVector,kotlin.String,androidx.compose.ui.Modifier,androidx.compose.ui.graphics.Color)) 等视觉可组合项包含一个 `contentDescription` 参数。您可以在其中传递该视觉元素的**本地化**描述，如果该元素是纯装饰性的，则传递 `null`。

在我们的应用中，文章屏幕缺少一些内容描述。让我们运行应用并选择顶部的文章以导航到文章屏幕。

![在启用了 TalkBack 的情况下录制的两个屏幕 - 点按文章屏幕中的返回按钮。在左侧的屏幕中，TalkBack 读出“Button—double tap to activate”。在右侧的屏幕中，TalkBack 读出“Navigate up—double tap to activate”。](/images/compose_barrier/5_1.gif)

添加视觉内容描述。之前（左侧）与之后（右侧）的比较。

如果我们不提供任何信息，当用户点按左上角的导航图标时，TalkBack 将简单地读出“Button, double tap to activate”。这样并没有告诉用户有关当他们激活该按钮时将会执行什么操作的任何信息。让我们打开 `ArticleScreen.kt`：

```auto
@Composable
fun ArticleScreen(
   // ...
) {
   // ...
   Scaffold(
       topBar = {
           InsetAwareTopAppBar(
               title = {
                   // ...
               },
               navigationIcon = {
                   IconButton(onClick = onBack) {
                       Icon(
                           imageVector = Icons.Filled.ArrowBack,
                           contentDescription = null
                       )
                   }
               }
           )
       }
   ) {
       // ...
   }
}
```

向 Icon 添加有意义的内容描述：

```auto
@Composable
fun ArticleScreen(
   // ...
) {
   // ...
   Scaffold(
       topBar = {
           InsetAwareTopAppBar(
               title = {
                   // ...
               },
               navigationIcon = {
                   IconButton(onClick = onBack) {
                       Icon(
                           imageVector = Icons.Filled.ArrowBack,
                           contentDescription = stringResource(
                               R.string.cd_navigate_up
                           )
                       )
                   }
               }
           )
       }
   ) {
       // ...
   }
}
```

这篇文章中的另一个视觉元素是标题图片。在本例中，此图片是纯装饰性的，它没有显示我们需要传达给用户的任何信息。因此，将内容描述设置为 `null`，当我们使用无障碍服务时，会跳过该元素。

屏幕中的最后一个视觉元素是个人资料照片。在本例中，我们使用的是通用头像，因此没有必要在此处添加内容描述。当我们使用此作者的实际个人资料照片时，我们可以让他们为其[提供合适的内容描述](https://jakearchibald.com/2021/great-alt-text/)。

## 7\. 标题

当屏幕包含大量的文字时，就像我们的文章屏幕一样，有视觉障碍的用户很难快速找到他们想要查找的版块。为了帮助解决此问题，我们可以指明文字的哪些部分是标题。然后，用户就可以通过向上或向下滑动来快速浏览这些不同的标题。

默认情况下，没有可组合项被标记为标题，因此不可能进行导航。我们希望文章屏幕提供按标题导航：

![在启用了 TalkBack 的情况下录制的两个屏幕 - 使用向下滑动手势在标题之间导航。在左侧的屏幕中，TalkBack 读出“No next heading”。在右侧的屏幕中，用户循环浏览标题，TalkBack 大声读出每个标题。](/images/compose_barrier/6_1.gif)

添加标题。之前（左侧）与之后（右侧）的比较。

文章中的标题是在 `PostContent.kt` 中定义的。让我们打开该文件并滚动到 `Paragraph` 可组合项：

```auto
@Composable
private fun Paragraph(paragraph: Paragraph) {
   // ...
   Box(modifier = Modifier.padding(bottom = trailingPadding)) {
       when (paragraph.type) {
           // ...
           ParagraphType.Header -> {
               Text(
                   modifier = Modifier.padding(4.dp),
                   text = annotatedString,
                   style = textStyle.merge(paragraphStyle)
               )
           }
           // ...
       }
   }
}
```

此处，`Header` 被定义为一个简单的 `Text` 可组合项。我们可以设置 [`heading`](https://developer.android.com/reference/kotlin/androidx/compose/ui/semantics/package-summary?hl=zh-cn#(androidx.compose.ui.semantics.SemanticsPropertyReceiver).heading()) 语义属性，以指明此可组合项是标题。

```auto
@Composable
private fun Paragraph(paragraph: Paragraph) {
   // ...
   Box(modifier = Modifier.padding(bottom = trailingPadding)) {
       when (paragraph.type) {
           // ...
           ParagraphType.Header -> {
               Text(
                   modifier = Modifier.padding(4.dp)
                     .semantics { heading() },
                   text = annotatedString,
                   style = textStyle.merge(paragraphStyle)
               )
           }
           // ...
       }
   }
}
```
## 8\. 自定义合并

正如我们在前面的步骤中所看到的，TalkBack 等无障碍服务按元素在屏幕中导航。默认情况下，Jetpack Compose 中至少设置了一个语义属性的每个低级可组合项会获得焦点。例如，`Text` 可组合项设置了 `text` 语义属性，因此会获得焦点。

不过，如果屏幕上有太多可聚焦的元素，当用户逐个浏览这些元素时，会导致混乱。我们可以使用 [`semantics`](https://developer.android.com/reference/kotlin/androidx/compose/ui/semantics/package-summary?hl=zh-cn#(androidx.compose.ui.Modifier).semantics(kotlin.Boolean,kotlin.Function1)) 修饰符及其 `mergeDescendants` 属性将可组合项合并在一起。

让我们看一下文章屏幕。大多数元素都获得了正确级别的焦点。但是，文章的元数据目前是作为几个单独的项目朗读的。可以通过将其合并为一个可聚焦实体来加以改进：

![在启用了 TalkBack 的情况下录制的两个屏幕。在左侧的屏幕中，为“Author”和“Metadata”字段显示了单独的绿色 TalkBack 矩形。在右侧的屏幕中，围绕这两个字段显示了一个矩形，并且 TalkBack 读出了串联的内容。](/images/compose_barrier/7_1.gif)

合并可组合项。之前（左侧）与之后（右侧）的比较。

让我们打开 `PostContent.kt` 并查看 `PostMetadata` 可组合项：

```auto
@Composable
private fun PostMetadata(metadata: Metadata) {
   // ...
   Row {
       Image(
           // ...
       )
       Spacer(Modifier.width(8.dp))
       Column {
           Text(
               // ...
           )

           CompositionLocalProvider(LocalContentAlpha provides ContentAlpha.medium) {
               Text(
                   // ..
               )
           }
       }
   }
}
```

我们可以让顶级行合并它的后代，这样就会产生我们想要的行为：

```auto
@Composable
private fun PostMetadata(metadata: Metadata) {
   // ...
   Row(Modifier.semantics(mergeDescendants = true) {}) {
       Image(
           // ...
       )
       Spacer(Modifier.width(8.dp))
       Column {
           Text(
               // ...
           )

           CompositionLocalProvider(LocalContentAlpha provides ContentAlpha.medium) {
               Text(
                   // ..
               )
           }
       }
   }
}
```
## 9\. 切换开关和复选框

当 TalkBack 选定 `Switch` 和 `Checkbox` 等可切换元素时，会大声读出其选中状态。不过，如果没有上下文，就很难理解这些可切换元素指的是什么。我们可以通过提升可切换状态来包含可切换元素的上下文，这样用户就可以通过按可组合项本身或描述它的标签来切换 `Switch` 或 `Checkbox`。

我们可以在“Interests”屏幕中看到一个这样的例子。您可以从主屏幕中打开抽屉式导航栏来导航到该屏幕。在“Interests”屏幕上，有用户可以订阅的主题列表。默认情况下，此屏幕上的复选框与其标签是分开聚焦的，这使得很难理解其上下文。我们希望整个 `Row` 可切换：

![在启用了 TalkBack 的情况下录制的两个屏幕 - 显示了“Interests”屏幕，其中包含可选择主题的列表。在左侧的屏幕中，TalkBack 单独选中每个复选框。在右侧的屏幕中，TalkBack 选择了整行。](/images/compose_barrier/8_1.gif)

使用复选框。之前（左侧）与之后（右侧）的比较。

让我们打开 `InterestsScreen.kt` 并查看 `TopicItem` 可组合项的实现：

```auto
@Composable
private fun TopicItem(itemTitle: String, selected: Boolean, onToggle: () -> Unit) {
   // ...
   Row(
       modifier = Modifier
           .padding(horizontal = 16.dp, vertical = 8.dp)
   ) {
       // ...
       Checkbox(
           checked = selected,
           onCheckedChange = { onToggle() },
           modifier = Modifier.align(Alignment.CenterVertically)
       )
   }
}
```

如您所见，`Checkbox` 有一个 `onCheckedChange` 回调，用于处理元素的开关切换。我们可以将此回调提升到整个 `Row` 的级别：

```auto
@Composable
private fun TopicItem(itemTitle: String, selected: Boolean, onToggle: () -> Unit) {
   // ...
   Row(
       modifier = Modifier
           .toggleable(
               value = selected,
               onValueChange = { _ -> onToggle() },
               role = Role.Checkbox
           )
           .padding(horizontal = 16.dp, vertical = 8.dp)
   ) {
       // ...
       Checkbox(
           checked = selected,
           onCheckedChange = null,
           modifier = Modifier.align(Alignment.CenterVertically)
       )
   }
}
```
## 10\. 状态描述

在上一步中，我们将开关切换行为从 `Checkbox` 提升到了父级 `Row`。我们可以通过为可组合项的状态添加自定义描述来进一步改进此元素的无障碍功能。

默认情况下，`Checkbox` 状态读作“Ticked”或“Not ticked”。我们可以将此描述替换为我们自己的自定义描述：

![在启用了 TalkBack 的情况下录制的两个屏幕 - 点按“Interests”屏幕中的一个主题。在左侧的屏幕中，TalkBack 朗读的是“Not ticked”，而在右侧的屏幕中，TalkBack 朗读的是“Not subscribed”。](/images/compose_barrier/9_1.gif)

添加状态描述。之前（左侧）与之后（右侧）的比较。

我们可以继续使用在最后一步中改写的 `TopicItem` 可组合项：

```auto
@Composable
private fun TopicItem(itemTitle: String, selected: Boolean, onToggle: () -> Unit) {
   // ...
   Row(
       modifier = Modifier
           .toggleable(
               value = selected,
               onValueChange = { _ -> onToggle() },
               role = Role.Checkbox
           )
           .padding(horizontal = 16.dp, vertical = 8.dp)
   ) {
       // ...
       Checkbox(
           checked = selected,
           onCheckedChange = null,
           modifier = Modifier.align(Alignment.CenterVertically)
       )
   }
}
```

我们可以使用 `semantics` 修饰符中的 `stateDescription` 属性来添加自定义状态描述：

```auto
@Composable
private fun TopicItem(itemTitle: String, selected: Boolean, onToggle: () -> Unit) {
   // ...
   val stateNotSubscribed = stringResource(R.string.state_not_subscribed)
   val stateSubscribed = stringResource(R.string.state_subscribed)
   Row(
       modifier = Modifier
           .semantics {
               stateDescription = if (selected) {
                   stateSubscribed
               } else {
                   stateNotSubscribed
               }
           }
           .toggleable(
               value = selected,
               onValueChange = { _ -> onToggle() },
               role = Role.Checkbox
           )
           .padding(horizontal = 16.dp, vertical = 8.dp)
   ) {
       // ...
       Checkbox(
           checked = selected,
           onCheckedChange = null,
           modifier = Modifier.align(Alignment.CenterVertically)
       )
   }
}
```

## 11\. 结尾

恭喜您，您已成功完成了此 Codelab，并详细了解了如何使用 Compose 改进应用的无障碍功能。您学习了触摸目标、视觉元素描述和状态描述。您添加了点击标签、标题和自定义操作。您知道了如何添加自定义合并，以及如何使用切换开关和复选框。将学到的这些知识运用到应用中将会极大地改进其无障碍功能！

## 文档

如需获得关于上述主题的更多信息和指导，请参阅以下文档：

+   [使用 Jetpack Compose 改进应用的无障碍功能](https://developer.android.com/jetpack/compose/accessibility?hl=zh-cn)
+   [Jetpack Compose 中的语义](https://developer.android.com/jetpack/compose/semantics?hl=zh-cn)