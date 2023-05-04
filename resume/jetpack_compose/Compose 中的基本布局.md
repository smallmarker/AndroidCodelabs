## 1\. 简介

Compose 是一个界面工具包，可让您轻松实现应用的设计。您只需描述自己想要的界面外观，Compose 会负责在屏幕上进行绘制。在此 Codelab 中，您将了解如何编写 Compose 界面。本文假定您了解[基础知识 Codelab](https://developer.android.com/codelabs/jetpack-compose-basics?hl=zh-cn) 中介绍的概念，因此请务必先完成该 Codelab。在基础知识 Codelab 中，您学习了如何使用 `Surfaces`、`Rows` 和 `Columns` 实现简单布局。您还使用 `padding`、`fillMaxWidth` 和 `size` 等修饰符扩充了这些布局。

在此 Codelab 中，您将实现一个**更真实、更复杂的布局**，在此过程中，您还会了解各种**开箱即用型可组合项**和**修饰符**。完成此 Codelab 后，您应该能够将基本应用的设计转换为有效代码。

## 学习内容

在此 Codelab 中，您将学习：

+   如何借助修饰符扩充可组合项？
+   如何通过 Column 和 LazyRow 等标准布局组件定位可组合子项？
+   如何通过对齐方式和排列方式更改可组合子项在父项中的位置？
+   如何借助 Scaffold 和 Bottom Navigation 等 Material 可组合项创建详细布局？
+   如何使用槽位 API 构建灵活的可组合项？

## 所需条件

+   [最新版 Android Studio](https://developer.android.com/studio?hl=zh-cn)。
+   有使用 Kotlin 语法（包括 lambda）的经验。
+   有使用 Compose 的基本经验。如果您还没有相关经验，请先完成[“Jetpack Compose 基础知识”Codelab](https://codelabs.developers.google.com/codelabs/jetpack-compose-basics/?hl=zh-cn)，然后再开始此 Codelab。
+   大致了解可组合项和修饰符的含义。

## 构建内容

在此 Codelab 中，您将根据设计人员提供的模拟实现真实的应用设计。**MySoothe** 是一款健康应用，其中列出了可改善身心健康的各种方法。这款应用包含两个版块，一个列出了您喜爱的合集，另一个列出了各种体育锻炼。该应用如下所示：

## <img src="/images/compose_layout/1.png" alt="1.png" style="zoom:50%;" />

## 2\. 准备工作

在此步骤中，您需要下载包含主题和一些基本设置的代码。

## 获取代码

此 Codelab 的代码可以在 [android-compose-codelabs GitHub 代码库](https://github.com/googlecodelabs/android-compose-codelabs)中找到。如需克隆该代码库，请运行以下命令：

$ git clone https://github.com/googlecodelabs/android-compose-codelabs

或者，您也可以下载两个 ZIP 文件：

+   [起始代码](https://github.com/googlecodelabs/android-compose-codelabs/archive/refs/heads/main.zip)
+   [解决方案](https://github.com/googlecodelabs/android-compose-codelabs/archive/refs/heads/end.zip)

## 查看代码

下载的代码包含适用于所有可用 Compose Codelab 的代码。为了完成此 Codelab，请在 Android Studio 中打开 **`BasicLayoutsCodelab`** 项目。

我们建议您从 `main` 分支中的代码着手，按照自己的节奏逐步完成此 Codelab。

## 3\. 从制定计划着手

下面我们来详细了解一下设计：

<img src="/images/compose_layout/2.png" alt="2.png" style="zoom:50%;" />

在需要实现设计时，最好先清楚了解其结构。不要立即开始编码，而应**分析设计本身**。如何**将此界面拆分为多个可重复利用的部分**？

接下来，我们试着分析一下设计。在最高抽象级别，我们可以将此设计细分为两部分：

+   屏幕上的内容。
+   底部导航栏。

<img src="/images/compose_layout/3.png" alt="3.png" style="zoom:50%;" />

展开细目后，您可以看到屏幕内容包含三个子部分：

+   搜索栏。
+   “Align your body”版块。
+   “Favorite collections”版块。

<img src="/images/compose_layout/4.png" alt="4.png" style="zoom:50%;" />

在每个版块中，您还可以看到一些可重复利用的较低级别组件：

+   “align your body”元素，显示在可水平滚动的行中。

<img src="/images/compose_layout/5.png" alt="5.png" style="zoom:50%;" />

+   “favorite collection”卡片，显示在可水平滚动的网格中。

<img src="/images/compose_layout/6.png" alt="6.png" style="zoom:50%;" />

现在，您已经分析了设计，接下来可以开始为每个已确定的界面部分实现可组合项。先从最低级别的可组合项着手，然后继续将它们组合成更复杂的可组合项。完成此 Codelab 后，您的新应用将与所提供的设计相似。

## 4\. 搜索栏 - 修饰符

第一个要转换为可组合项的元素是搜索栏。我们再来看一下设计：

<img src="/images/compose_layout/7.png" alt="7.png" style="zoom:50%;" />

只通过上面的屏幕截图，很难在实现该设计时让像素完美呈现。通常情况下，设计人员应传达更多关于设计的信息。他们可以授权您访问他们的设计工具，或分享所谓的用红线标注的设计。在此示例中，我们的设计人员提交了用红线标注的设计，方便您读出任何尺寸值。该设计采用 8dp 网格叠加层的方式显示，以便您轻松查看各个元素之间和周围的空间大小。此外，还明确添加了一些间距，以阐明特定尺寸。

<img src="/images/compose_layout/8.png" alt="8.png" style="zoom:50%;" />

您可以看到，搜索栏的高度应为 56 密度无关像素。它还应填充其父项的全宽。

若要实现搜索栏，请使用名为[文本字段](https://material.io/components/text-fields)的 Material 组件。Compose Material 库包含一个名为 [`TextField`](https://developer.android.com/reference/kotlin/androidx/compose/material/package-summary?hl=zh-cn#TextField(kotlin.String,kotlin.Function1,androidx.compose.ui.Modifier,kotlin.Boolean,kotlin.Boolean,androidx.compose.ui.text.TextStyle,kotlin.Function0,kotlin.Function0,kotlin.Function0,kotlin.Function0,kotlin.Boolean,androidx.compose.ui.text.input.VisualTransformation,androidx.compose.foundation.text.KeyboardOptions,androidx.compose.foundation.text.KeyboardActions,kotlin.Boolean,kotlin.Int,androidx.compose.foundation.interaction.MutableInteractionSource,androidx.compose.ui.graphics.Shape,androidx.compose.material.TextFieldColors)) 的可组合项，后者是此 Material 组件的实现。

从基本的 `TextField` 实现着手。在您的代码库中，打开 `MainActivity.kt` 并搜索 `SearchBar` 可组合项。

在名为 `SearchBar` 的可组合项内，编写基本的 `TextField` 实现：

```auto
import androidx.compose.material.TextField

@Composable
fun SearchBar(
   modifier: Modifier = Modifier
) {
   TextField(
       value = "",
       onValueChange = {},
       modifier = modifier
   )
}
```

需要注意以下几点：

+   您已对文本字段的值进行了硬编码，`onValueChange` 回调不会执行任何操作。由于此 Codelab 主要介绍布局，因此您可以忽略与状态相关的一切内容。

+   `SearchBar` 可组合函数接受 `modifier` 形参，并将其传递给 `TextField`。这是符合 [Compose 准则](https://android.googlesource.com/platform/frameworks/support/+/androidx-main/compose/docs/compose-api-guidelines.md#elements-accept-and-respect-a-modifier-parameter)的最佳实践。这样一来，方法调用方就可以修改可组合项的外观和风格，使其更加灵活且可重复利用。对于此 Codelab 中的所有可组合项，您应继续遵循这一最佳实践。

我们来看一看此可组合项的预览。请注意，您可以使用 Android Studio 中的[预览功能](https://developer.android.com/jetpack/compose/tooling?hl=zh-cn#preview)快速迭代各个可组合项。`MainActivity.kt` 包含您要在此 Codelab 中构建的所有可组合项的预览。在此示例中，`SearchBarPreview` 方法会渲染我们的 `SearchBar` 可组合项，并提供一些背景和内边距，以便提供更多上下文。完成您刚才添加的实现后，预览应如下所示：

<img src="/images/compose_layout/9.png" alt="9.png" style="zoom:50%;" />

不过，还缺少一些内容。首先，我们使用[修饰符](https://developer.android.com/jetpack/compose/modifiers?hl=zh-cn)修正可组合项的尺寸。

编写可组合项时，您可以使用**修饰符**执行以下操作：

+   更改可组合项的尺寸、布局、行为和外观。
+   添加信息，例如无障碍标签。
+   处理用户输入。
+   添加高级互动，例如使元素可点击、可滚动、可拖动或可缩放。

您调用的每个可组合项都有一个 `modifier` 形参，您可以设置该形参以适应相应可组合项的外观、风格和行为。设置修饰符时，您可以将多个修饰符方法串联起来，以创建更复杂的调适。

在此示例中，搜索栏的高度应至少为 56dp，并填充其父项的宽度。为了找到适合搜索栏的修饰符，您可以浏览[修饰符列表](https://developer.android.com/jetpack/compose/modifiers-list?hl=zh-cn)，并查看[尺寸部分](https://developer.android.com/jetpack/compose/modifiers-list?hl=zh-cn#Size)。对于高度，您可以使用 [`heightIn`](https://developer.android.com/reference/kotlin/androidx/compose/foundation/layout/package-summary?hl=zh-cn#(androidx.compose.ui.Modifier).heightIn(androidx.compose.ui.unit.Dp,androidx.compose.ui.unit.Dp)) 修饰符。这可确保该可组合项具有特定的最小高度。但是，如果用户放大系统字体大小，该高度会随之变高。对于宽度，您可以使用 [`fillMaxWidth`](https://developer.android.com/reference/kotlin/androidx/compose/foundation/layout/package-summary?hl=zh-cn#(androidx.compose.ui.Modifier).fillMaxWidth(kotlin.Float)) 修饰符。此修饰符可确保搜索栏占用其父项的所有水平空间。

更新修饰符以匹配下面的代码：

```auto
import androidx.compose.material.TextField

@Composable
fun SearchBar(
   modifier: Modifier = Modifier
) {
   TextField(
       value = "",
       onValueChange = {},
       modifier = modifier
           .fillMaxWidth()
           .heightIn(min = 56.dp)
   )
}
```

在此示例中，由于一个修饰符会影响宽度，另一个修饰符会影响高度，因此这些修饰符的顺序无关紧要。

您还必须设置 `TextField` 的一些形参。您可以试着设置形参值，以使可组合项类似于设计。同样，以下设计供您参考：

<img src="/images/compose_layout/10.png" alt="10.png" style="zoom:50%;" />

更新实现时应采取以下步骤：

+   添加搜索图标。`TextField` 包含一个可接受其他可组合项的 `leadingIcon` 形参。您可以在其中设置 `Icon`，在此示例中应为 `Search` 图标。请务必使用正确的 Compose `Icon` 导入。
+   将文本字段的背景颜色设为 MaterialTheme 的 `surface` 颜色。您可以使用 `TextFieldDefaults.textFieldColors` 替换特定颜色。
+   添加占位符文本“Search”（以字符串资源 `R.string.placeholder_search` 的形式显示）。

完成后，您的可组合项应如下所示：

```auto
import androidx.compose.foundation.layout.fillMaxWidth
import androidx.compose.foundation.layout.heightIn
import androidx.compose.ui.res.stringResource
import androidx.compose.material.Icon
import androidx.compose.material.MaterialTheme
import androidx.compose.material.Text
import androidx.compose.material.TextField
import androidx.compose.material.TextFieldDefaults
import androidx.compose.material.icons.Icons
import androidx.compose.material.icons.filled.Search

@Composable
fun SearchBar(
   modifier: Modifier = Modifier
) {
   TextField(
       value = "",
       onValueChange = {},
       leadingIcon = {
           Icon(
               imageVector = Icons.Default.Search,
               contentDescription = null
           )
       },
       colors = TextFieldDefaults.textFieldColors(
           backgroundColor = MaterialTheme.colors.surface
       ),
       placeholder = {
           Text(stringResource(R.string.placeholder_search))
       },
       modifier = modifier
           .fillMaxWidth()
           .heightIn(min = 56.dp)
   )
}
```

请注意：

+   您添加了一个用于显示搜索图标的 `leadingIcon`。此图标不需要内容说明，因为文本字段的占位符已经说明了文本字段的含义。请注意，内容说明通常用于实现无障碍功能，以文本形式向应用用户呈现图片或图标。

+   如需调整文本字段的背景颜色，请设置 `colors` 属性。该可组合项包含一个组合形参，而不是每种颜色的单独形参。在这种情况下，您可以传入 `TextFieldDefaults` 数据类的副本，您可以通过该类仅更新不同的颜色。在此示例中，即仅更新背景颜色。
+   您设置的是最小高度，而不是固定高度。建议您采用这种方法，这样一来，如果用户在系统设置中增加字体大小，文本字段的大小仍会增加。

在此步骤中，您了解了如何使用可组合项的形参和修饰符来更改可组合项的外观和风格。这适用于 Compose 和 Material 库提供的可组合项，以及您自己编写的可组合项。您应始终考虑通过提供形参来自定义所编写的可组合项。您还应添加 `modifier` 属性，以便从外部调整可组合项的外观和风格。

## 5\. Align your body - 对齐

接下来，您要实现的可组合项是“Align your body”元素。我们来看看该元素的设计，包括它旁边的红线设计：

<img src="/images/compose_layout/11.png" alt="11.png" style="zoom:30%;" /> <img src="/images/compose_layout/11_2.png" alt="11_2.png" style="zoom:82%;" />

红线设计现在还包含面向基线的间距。以下是我们从中获得的信息：

+   图片高度应为 88dp。
+   文本基线与图片之间的间距应为 24dp。
+   基线与元素底部的间距应为 8dp。
+   文本的排版样式应为 H3。

如需实现此可组合项，您需要一个 [`Image`](https://developer.android.com/reference/kotlin/androidx/compose/foundation/package-summary?hl=zh-cn#Image(androidx.compose.ui.graphics.painter.Painter,kotlin.String,androidx.compose.ui.Modifier,androidx.compose.ui.Alignment,androidx.compose.ui.layout.ContentScale,kotlin.Float,androidx.compose.ui.graphics.ColorFilter)) 和一个 [`Text`](https://developer.android.com/reference/kotlin/androidx/compose/material/package-summary?hl=zh-cn#Text(kotlin.String,androidx.compose.ui.Modifier,androidx.compose.ui.graphics.Color,androidx.compose.ui.unit.TextUnit,androidx.compose.ui.text.font.FontStyle,androidx.compose.ui.text.font.FontWeight,androidx.compose.ui.text.font.FontFamily,androidx.compose.ui.unit.TextUnit,androidx.compose.ui.text.style.TextDecoration,androidx.compose.ui.text.style.TextAlign,androidx.compose.ui.unit.TextUnit,androidx.compose.ui.text.style.TextOverflow,kotlin.Boolean,kotlin.Int,kotlin.Function1,androidx.compose.ui.text.TextStyle)) 可组合项。您还需要将它们添加到 [`Column`](https://developer.android.com/reference/kotlin/androidx/compose/foundation/layout/package-summary?hl=zh-cn#Column(androidx.compose.ui.Modifier,androidx.compose.foundation.layout.Arrangement.Vertical,androidx.compose.ui.Alignment.Horizontal,kotlin.Function1)) 中，以便一个位于另一个下方。

在您的代码中找到 `AlignYourBodyElement` 可组合项，使用以下基本实现更新其内容：

```auto
import androidx.compose.foundation.Image
import androidx.compose.foundation.layout.Column
import androidx.compose.ui.res.painterResource

@Composable
fun AlignYourBodyElement(
   modifier: Modifier = Modifier
) {
   Column(
       modifier = modifier
   ) {
       Image(
           painter = painterResource(R.drawable.ab1_inversions),
           contentDescription = null
       )
       Text(
           text = stringResource(R.string.ab1_inversions)
       )
   }
}
```

请注意：

+   您将图片的 `contentDescription` 设为 null，因为这张图片是纯装饰性的。图片下方的文本充分描述了图片的含义，因此无需为图片添加额外的说明。
+   您使用的是经过硬编码的图片和文本。在下一步中，您要为这些内容改用 `AlightYourBodyElement` 可组合项中提供的形参，使其变为动态形式。

查看此可组合项的预览：

<img src="/images/compose_layout/12.png" alt="12.png" style="zoom:50%;" />

您需要进行一些改进。最值得注意的是，图片过大，且形状不是圆形。您可以使用 [`size`](https://developer.android.com/reference/kotlin/androidx/compose/foundation/layout/package-summary?hl=zh-cn#(androidx.compose.ui.Modifier).size(androidx.compose.ui.unit.Dp)) 和 [`clip`](https://developer.android.com/reference/kotlin/androidx/compose/ui/draw/package-summary?hl=zh-cn#(androidx.compose.ui.Modifier).clip(androidx.compose.ui.graphics.Shape)) 修饰符以及 `contentScale` 形参调整 `Image` 可组合项。

`size` 修饰符会调整可组合项以适应特定尺寸，这类似于您在上一步中看到的 `fillMaxWidth` 和 `heightIn` 修饰符。`clip` 修饰符的工作原理有所不同，用于**调整可组合项的外观**。您可以将该修饰符设置为任何 [`Shape`](https://developer.android.com/reference/kotlin/androidx/compose/ui/graphics/Shape?hl=zh-cn)，然后它会按照相应形状对可组合项的内容进行裁剪。

图片也需要正确缩放。为此，我们可以使用 `Image` 的 `contentScale` 形参。具体选项有很多，最值得注意的是：

![13.png](/images/compose_layout/13.png)

在此示例中，剪裁类型就是要使用的正确类型。应用修饰符和形参后，您的代码应如下所示：

```auto
import androidx.compose.foundation.layout.size
import androidx.compose.foundation.shape.CircleShape
import androidx.compose.ui.draw.clip
import androidx.compose.ui.layout.ContentScale
@Composable
fun AlignYourBodyElement(
   modifier: Modifier = Modifier
) {
   Column(
       modifier = modifier
   ) {
       Image(
           painter = painterResource(R.drawable.ab1_inversions),
           contentDescription = null,
           contentScale = ContentScale.Crop,
           modifier = Modifier
               .size(88.dp)
               .clip(CircleShape)
       )
       Text(
           text = stringResource(R.string.ab1_inversions)
       )
   }
}
```

现在，您的设计应如下所示：

![14.png](/images/compose_layout/14.png)

接下来，通过设置 `Column` 的对齐方式来水平对齐文本。

一般来说，若要对齐父容器中的可组合项，您应设置该父容器的**对齐方式**。因此，您应告知父项如何对齐其子项，而不是告知子项将其自身放置在父项中。

对于 `Column`，您可以决定其子项的水平对齐方式。具体选项包括：

+   Start
+   CenterHorizontally
+   End

对于 `Row`，您可以设置垂直对齐。具体选项类似于 `Column` 的选项：

+   Top
+   CenterVertically
+   Bottom

对于 `Box`，您可以同时使用水平对齐和垂直对齐。具体选项包括：

+   TopStart
+   TopCenter
+   TopEnd
+   CenterStart
+   Center
+   CenterEnd
+   BottomStart
+   BottomCenter
+   BottomEnd

容器的所有子项都将遵循这一相同的对齐模式。您可以通过向单个子项添加 [`align`](https://developer.android.com/reference/kotlin/androidx/compose/foundation/layout/ColumnScope?hl=zh-cn#(androidx.compose.ui.Modifier).align(androidx.compose.ui.Alignment.Horizontal)) 修饰符来替换其行为。

对于此设计，文本应水平居中。为此，请将 `Column` 的 `horizontalAlignment` 设置为水平居中：

```auto
import androidx.compose.ui.Alignment
@Composable
fun AlignYourBodyElement(
   modifier: Modifier = Modifier
) {
   Column(
       horizontalAlignment = Alignment.CenterHorizontally,
       modifier = modifier
   ) {
       Image(
           //..
       )
       Text(
           //..
       )
   }
}
```

实现这些部分后，您只需进行一些细微更改，即可使可组合项与设计完全相同。如果遇到问题，不妨尝试自行实现这些部分，也可以引用最终代码。考虑采取以下步骤：

+   将图片和文本变为动态形式。将它们作为实参传递给可组合函数。别忘了更新相应的预览，并传入一些经过硬编码的数据。
+   更新文本以使用合适的排版样式。
+   更新文本元素的基线间距。

执行完这些步骤后，您的代码应如下所示：

```auto
import androidx.compose.foundation.layout.paddingFromBaseline

@Composable
fun AlignYourBodyElement(
   @DrawableRes drawable: Int,
   @StringRes text: Int,
   modifier: Modifier = Modifier
) {
   Column(
       modifier = modifier,
       horizontalAlignment = Alignment.CenterHorizontally
   ) {
       Image(
           painter = painterResource(drawable),
           contentDescription = null,
           contentScale = ContentScale.Crop,
           modifier = Modifier
               .size(88.dp)
               .clip(CircleShape)
       )
       Text(
           text = stringResource(text),
           style = MaterialTheme.typography.h3,
           modifier = Modifier.paddingFromBaseline(
               top = 24.dp, bottom = 8.dp
           )
       )
   }
}

@Preview(showBackground = true, backgroundColor = 0xFFF0EAE2)
@Composable
fun AlignYourBodyElementPreview() {
   MySootheTheme {
       AlignYourBodyElement(
           text = R.string.ab1_inversions,
           drawable = R.drawable.ab1_inversions,
           modifier = Modifier.padding(8.dp)
       )
   }
}
```
## 6\. “Favorite collection”卡片 - Material Surface

下一个可组合项的实现方式与“Align your body”元素类似。设计（包括红线）如下所示：

![15.png](/images/compose_layout/15.png)

![15_2.png](/images/compose_layout/15_2.png)

此示例提供了可组合项的完整尺寸。您可以看到，文本同样还应为 H3。

此容器拥有某种背景颜色，不同于整个屏幕的背景颜色。它还带有圆角。由于设计人员没有指定颜色，因此我们可以假定颜色将由主题定义。对于此类容器，我们使用 Material 的 [`Surface`](https://developer.android.com/reference/kotlin/androidx/compose/material/package-summary?hl=zh-cn#Surface(androidx.compose.ui.Modifier,androidx.compose.ui.graphics.Shape,androidx.compose.ui.graphics.Color,androidx.compose.ui.graphics.Color,androidx.compose.foundation.BorderStroke,androidx.compose.ui.unit.Dp,kotlin.Function0)) 可组合项。

您可以根据自己的需求调整 `Surface`，只需设置其形参和修饰符即可。在此示例中，表面应有圆角。针对这种情况，您可以使用 `shape` 形参。对于上一步中的图片，您将形状设置为 `Shape`，但在这一步中，您将使用来自我们的 Material 主题的值。

我们来看一看效果如何：

```auto
import androidx.compose.foundation.layout.Row
import androidx.compose.material.Surface

@Composable
fun FavoriteCollectionCard(
   modifier: Modifier = Modifier
) {
   Surface(
       shape = MaterialTheme.shapes.small,
       modifier = modifier
   ) {
       Row {
           Image(
               painter = painterResource(R.drawable.fc2_nature_meditations),
               contentDescription = null
           )
           Text(
               text = stringResource(R.string.fc2_nature_meditations)
           )
       }
   }
}
```

我们来看一下此实现的预览：

<img src="/images/compose_layout/16.png" alt="16.png" style="zoom:50%;" />

接下来，运用在上一步中学到的经验。设置图片尺寸，然后在容器中对其进行剪裁。设置 `Row` 的宽度，并与其子项垂直对齐。先尝试自行实施这些更改，然后再查看解决方案代码！

现在，您的代码如下所示：

```auto
import androidx.compose.foundation.layout.width

@Composable
fun FavoriteCollectionCard(
   modifier: Modifier = Modifier
) {
   Surface(
       shape = MaterialTheme.shapes.small,
       modifier = modifier
   ) {
       Row(
           verticalAlignment = Alignment.CenterVertically,
           modifier = Modifier.width(192.dp)
       ) {
           Image(
               painter = painterResource(R.drawable.fc2_nature_meditations),
               contentDescription = null,
               contentScale = ContentScale.Crop,
               modifier = Modifier.size(56.dp)
           )
           Text(
               text = stringResource(R.string.fc2_nature_meditations)
           )
       }
   }
}
```

预览现在应如下所示：

![17.png](/images/compose_layout/17.png)

如需完成此可组合项，请实施以下步骤：

+   将图片和文本变为动态形式。将它们作为实参传入可组合函数。
+   更新文本以使用合适的排版样式。
+   更新图片与文本之间的间距。

最终结果应如下所示：

```auto
@Composable
fun FavoriteCollectionCard(
   @DrawableRes drawable: Int,
   @StringRes text: Int,
   modifier: Modifier = Modifier
) {
   Surface(
       shape = MaterialTheme.shapes.small,
       modifier = modifier
   ) {
       Row(
           verticalAlignment = Alignment.CenterVertically,
           modifier = Modifier.width(192.dp)
       ) {
           Image(
               painter = painterResource(drawable),
               contentDescription = null,
               contentScale = ContentScale.Crop,
               modifier = Modifier.size(56.dp)
           )
           Text(
               text = stringResource(text),
               style = MaterialTheme.typography.h3,
               modifier = Modifier.padding(horizontal = 16.dp)
           )
       }
   }
}

//..

@Preview(showBackground = true, backgroundColor = 0xFFF0EAE2)
@Composable
fun FavoriteCollectionCardPreview() {
   MySootheTheme {
       FavoriteCollectionCard(
           text = R.string.fc2_nature_meditations,
           drawable = R.drawable.fc2_nature_meditations,
           modifier = Modifier.padding(8.dp)
       )
   }
}
```
## 7\. “Align your body”行 - 排列

现在，您已经创建了屏幕上显示的基本可组合项，接下来可以开始创建屏幕的不同部分了。

先从“Align your body”可滚动行着手。

![18.gif](/images/compose_layout/18.gif)

此组件的红线设计如下所示：

![19.png](/images/compose_layout/19.png)

请注意，一个网格块代表 8dp。因此，在此设计中，该行中的第一项内容前和最后一项内容后均留有 16dp 的间距。各项内容之间的间距为 8dp。

在 Compose 中，您可以使用 [`LazyRow`](https://developer.android.com/reference/kotlin/androidx/compose/foundation/lazy/package-summary?hl=zh-cn#LazyRow(androidx.compose.ui.Modifier,androidx.compose.foundation.lazy.LazyListState,androidx.compose.foundation.layout.PaddingValues,kotlin.Boolean,androidx.compose.foundation.layout.Arrangement.Horizontal,androidx.compose.ui.Alignment.Vertical,androidx.compose.foundation.gestures.FlingBehavior,kotlin.Boolean,kotlin.Function1)) 可组合项实现这种可滚动行。如需详细了解延迟列表（例如 `LazyRow` 和 `LazyColumn`），请参阅[有关列表的文档](https://developer.android.com/jetpack/compose/lists?hl=zh-cn)。在此 Codelab 中，了解 `LazyRow` 只会渲染屏幕上显示的元素（而不是同时渲染所有元素）就足够了，这有助于让应用保持出色性能。

先从此 `LazyRow` 的基本实现着手：

```auto
import androidx.compose.foundation.lazy.LazyRow
import androidx.compose.foundation.lazy.items

@Composable
fun AlignYourBodyRow(
   modifier: Modifier = Modifier
) {
   LazyRow(
       modifier = modifier
   ) {
       items(alignYourBodyData) { item ->
           AlignYourBodyElement(item.drawable, item.text)
       }
   }
}
```

如您所见，`LazyRow` 的子项不是可组合项。您应改用延迟列表 DSL，它可提供 [`item`](https://developer.android.com/reference/kotlin/androidx/compose/foundation/lazy/LazyListScope?hl=zh-cn#item(kotlin.Any,kotlin.Any,kotlin.Function1)) 和 [`items`](https://developer.android.com/reference/kotlin/androidx/compose/foundation/lazy/package-summary?hl=zh-cn#(androidx.compose.foundation.lazy.LazyListScope).items(kotlin.collections.List,kotlin.Function1,kotlin.Function1,kotlin.Function2)) 等方法，这些方法会以列表项的形式发出可组合项。对于所提供的 `alignYourBodyData` 中的各项，您可以发出之前实现的 `AlignYourBodyElement` 可组合项。

请注意显示方式：

<img src="/images/compose_layout/20.png" alt="20.png" style="zoom:50%;" />

我们在红线设计中看到的间距仍未显示。若要实现这些部分，您必须了解**排列方式**。

在上一步中，您了解了对齐方式，它用于在**交叉轴**上对齐容器的子项。对于 `Column`，交叉轴是水平轴；对于 `Row`，交叉轴则是垂直轴。

不过，我们也可以决定如何在容器的**主轴**（对于 `Row`，是水平轴；对于 `Column`，是垂直轴）上放置可组合子项。

对于 `Row`，您可以选择以下排列方式：

<img src="/images/compose_layout/21.gif" alt="21.gif" style="zoom:80%;" />

对于 `Column`：

<img src="/images/compose_layout/22.gif" alt="22.gif" style="zoom:70%;" />

除了这些排列方式之外，您还可以使用 `Arrangement.spacedBy()` 方法，在每个可组合子项之间添加固定间距。

在此示例中，您需要使用 `spacedBy` 方法，因为您需要在 `LazyRow` 中的各项之间留出 8dp 的间距。

```auto
import androidx.compose.foundation.layout.Arrangement

@Composable
fun AlignYourBodyRow(
   modifier: Modifier = Modifier
) {
   LazyRow(
       horizontalArrangement = Arrangement.spacedBy(8.dp),
       modifier = modifier
   ) {
       items(alignYourBodyData) { item ->
           AlignYourBodyElement(item.drawable, item.text)
       }
   }
}
```

现在，设计如下所示：

<img src="/images/compose_layout/23.png" alt="23.png" style="zoom:50%;" />

此外，您需要在 `LazyRow` 两侧添加一定尺寸的内边距。在此示例中，添加一个简单的内边距修饰符并不能达到目的。请尝试向 `LazyRow` 添加内边距，看看其行为方式：

![24.gif](/images/compose_layout/24.gif)

如您所见，滚动时，第一个和最后一个可见项在屏幕两侧被截断。

为了保持相同的内边距，同时确保在父级列表的边界内滚动内容时内容不会被截断，所有列表都需提供一个名为 `contentPadding` 的形参。

```auto
@Composable
fun AlignYourBodyRow(
   modifier: Modifier = Modifier
) {
   LazyRow(
       horizontalArrangement = Arrangement.spacedBy(8.dp),
       contentPadding = PaddingValues(horizontal = 16.dp),
       modifier = modifier
   ) {
       items(alignYourBodyData) { item ->
           AlignYourBodyElement(item.drawable, item.text)
       }
   }
}
```
## 8\. “Favorite collections”网格 - 延迟网格

接下来，您要实现的是屏幕的“Favorite collections”版块。这个可组合项需要的是网格，而不是单个行：

![25.gif](/images/compose_layout/25.gif)

您可以按照与上一部分类似的方式实现此版块，具体方法是创建一个 `LazyRow`，让各项都包含一个具有两个 `FavoriteCollectionCard` 实例的 `Column`。但在此步骤中，您需要使用 [`LazyHorizontalGrid`](https://developer.android.com/reference/kotlin/androidx/compose/foundation/lazy/grid/package-summary?hl=zh-cn#LazyHorizontalGrid(androidx.compose.foundation.lazy.grid.GridCells,androidx.compose.ui.Modifier,androidx.compose.foundation.lazy.grid.LazyGridState,androidx.compose.foundation.layout.PaddingValues,kotlin.Boolean,androidx.compose.foundation.layout.Arrangement.Horizontal,androidx.compose.foundation.layout.Arrangement.Vertical,androidx.compose.foundation.gestures.FlingBehavior,kotlin.Boolean,kotlin.Function1))，以便更好地将各项映射到网格元素。

首先实现包含两个固定行的简单网格：

```auto
import androidx.compose.foundation.lazy.grid.GridCells
import androidx.compose.foundation.lazy.grid.LazyHorizontalGrid
import androidx.compose.foundation.lazy.grid.items

@Composable
fun FavoriteCollectionsGrid(
   modifier: Modifier = Modifier
) {
   LazyHorizontalGrid(
       rows = GridCells.Fixed(2),
       modifier = modifier
   ) {
       items(favoriteCollectionsData) { item ->
           FavoriteCollectionCard(item.drawable, item.text)
       }
   }
}
```

如您所见，只需将上一步中的 `LazyRow` 替换为 `LazyHorizontalGrid` 即可。

不过，这样还无法得到正确的结果：

<img src="/images/compose_layout/26.png" alt="26.png" style="zoom:50%;" />

网格占用的空间与其父项相同，这意味着，“favorite collection”卡片会在垂直方向被过度拉伸。调整可组合项，以便网格单元的大小以及各单元之间的间距正确无误。

结果应如下所示：

```auto
@Composable
fun FavoriteCollectionsGrid(
   modifier: Modifier = Modifier
) {
   LazyHorizontalGrid(
       rows = GridCells.Fixed(2),
       contentPadding = PaddingValues(horizontal = 16.dp),
       horizontalArrangement = Arrangement.spacedBy(8.dp),
       verticalArrangement = Arrangement.spacedBy(8.dp),
       modifier = modifier.height(120.dp)
   ) {
       items(favoriteCollectionsData) { item ->
           FavoriteCollectionCard(
               drawable = item.drawable,
               text = item.text,
               modifier = Modifier.height(56.dp)
           )
       }
   }
}
```
## 9\. 首页部分 - 槽位 API

在 MySoothe 主屏幕中，有多个**版块**都遵循同一模式。每个版块都有一个标题，其中包含的内容因版块而异。我们想要实现的设计如下所示：

<img src="/images/compose_layout/27.png" alt="27.png" style="zoom:50%;" />

如您所见，每个版块都有一个**标题**和一个**槽位**。标题包含一些与其相关的间距和样式信息。可以使用不同的内容动态填充槽位，具体取决于版块。

如需实现这个灵活的版块容器，您可以使用所谓的槽位 API。在进行这项实现之前，请先阅读文档页面上有关[基于槽位的布局](https://developer.android.com/jetpack/compose/layouts/basics?hl=zh-cn#slot-based-layouts)的部分。这有助于您了解什么是基于槽位的布局，以及如何使用槽位 API 构建此类布局。

调整 `HomeSection` 可组合项以接收标题和槽位内容。您还应调整关联的预览，以调用这个包含“Align your body”标题及相关内容的 `HomeSection`：

```auto
@Composable
fun HomeSection(
   @StringRes title: Int,
   modifier: Modifier = Modifier,
   content: @Composable () -> Unit
) {
   Column(modifier) {
       Text(stringResource(title))
       content()
   }
}

@Preview(showBackground = true, backgroundColor = 0xFFF0EAE2)
@Composable
fun HomeSectionPreview() {
   MySootheTheme {
       HomeSection(R.string.align_your_body) {
           AlignYourBodyRow()
       }
   }
}
```

您可为可组合项的槽位使用 `content` 形参。这样一来，当您使用 `HomeSection` 可组合项时，便可使用尾随 lambda 填充内容槽位。当可组合项提供多个要填充的槽位时，您可为这些槽位指定有意义的名称，用于在更大的可组合项容器中代表其功能。例如，Material 的 [`TopAppBar`](https://material.io/components/app-bars-top) 为 `title`、`navigationIcon` 和 `actions` 提供槽位。

我们来看一下这个版块采用该实现后的效果：

<img src="/images/compose_layout/28.png" alt="28.png" style="zoom:50%;" />

Text 可组合项需要更多信息才能与设计保持一致。更新该可组合项，以便它：

+   采用全部大写的形式显示（提示：您可以使用 `String` 的 `uppercase()` 方法实现此目的）。
+   采用 H2 排版。
+   留有契合红线设计的内边距。

您的最终解决方案应如下所示：

```auto
import java.util.*

@Composable
fun HomeSection(
   @StringRes title: Int,
   modifier: Modifier = Modifier,
   content: @Composable () -> Unit
) {
   Column(modifier) {
       Text(
           text = stringResource(title).uppercase(Locale.getDefault()),
           style = MaterialTheme.typography.h2,
           modifier = Modifier
               .paddingFromBaseline(top = 40.dp, bottom = 8.dp)
               .padding(horizontal = 16.dp)
       )
       content()
   }
}
```
## 10\. 主屏幕 - 滚动

现在，您已经创建了所有单独的构建块，接下来可以将它们组合成一个全屏实现了。

您尝试实现的设计如下所示：

<img src="/images/compose_layout/29.png" alt="29.png" style="zoom:50%;" />

只需依序放置搜索栏和这两个版块。您需要添加一定尺寸的间距，以确保一切都契合设计。我们之前没有使用过 [`Spacer`](https://developer.android.com/reference/kotlin/androidx/compose/foundation/layout/package-summary?hl=zh-cn#Spacer(androidx.compose.ui.Modifier)) 可组合项，它可帮助我们在 `Column` 中添加额外的间距。如果您改为设置 `Column` 的内边距，便会看到之前在“Favorite Collections”网格中出现的相同截断行为。

```auto
@Composable
fun HomeScreen(modifier: Modifier = Modifier) {
   Column(modifier) {
       Spacer(Modifier.height(16.dp))
       SearchBar(Modifier.padding(horizontal = 16.dp))
       HomeSection(title = R.string.align_your_body) {
           AlignYourBodyRow()
       }
       HomeSection(title = R.string.favorite_collections) {
           FavoriteCollectionsGrid()
       }
       Spacer(Modifier.height(16.dp))
   }
}
```

虽然设计与大多数设备尺寸都非常契合，但如果设备的高度不足（例如在横屏模式下），设计需要能够垂直滚动。这就需要您添加滚动行为。

如前所述，`LazyRow` 和 `LazyHorizontalGrid` 等延迟布局会自动添加滚动行为。但是，您不一定总是需要延迟布局。一般来说，**在列表中有许多元素或需要加载大型数据集时，您需要使用延迟布局**，因此一次发出所有项不仅会降低性能，还会拖慢应用的运行速度。如果列表中的元素数量有限，您也可以选择使用简单的 `Column` 或 `Row`，然后**手动添加滚动行为**。为此，您可以使用 [`verticalScroll`](https://developer.android.com/reference/kotlin/androidx/compose/foundation/package-summary?hl=zh-cn#(androidx.compose.ui.Modifier).verticalScroll(androidx.compose.foundation.ScrollState,kotlin.Boolean,androidx.compose.foundation.gestures.FlingBehavior,kotlin.Boolean)) 或 [`horizontalScroll`](https://developer.android.com/reference/kotlin/androidx/compose/foundation/package-summary?hl=zh-cn#(androidx.compose.ui.Modifier).horizontalScroll(androidx.compose.foundation.ScrollState,kotlin.Boolean,androidx.compose.foundation.gestures.FlingBehavior,kotlin.Boolean)) 修饰符。这些修饰符需要 [`ScrollState`](https://developer.android.com/reference/kotlin/androidx/compose/foundation/ScrollState?hl=zh-cn)，后者包含当前的滚动状态，可用于从外部修改滚动状态。在此示例中，您不需要修改滚动状态，只需使用 [`rememberScrollState`](https://developer.android.com/reference/kotlin/androidx/compose/foundation/package-summary?hl=zh-cn#rememberScrollState(kotlin.Int)) 创建一个持久的 `ScrollState` 实例。

最终结果应如下所示：

```auto
import androidx.compose.foundation.rememberScrollState
import androidx.compose.foundation.verticalScroll

@Composable
fun HomeScreen(modifier: Modifier = Modifier) {
   Column(
       modifier
           .verticalScroll(rememberScrollState())
           .padding(vertical = 16.dp)
   ) {
       Spacer(Modifier.height(16.dp))
       SearchBar(Modifier.padding(horizontal = 16.dp))
       HomeSection(title = R.string.align_your_body) {
           AlignYourBodyRow()
       }
       HomeSection(title = R.string.favorite_collections) {
           FavoriteCollectionsGrid()
       }
       Spacer(Modifier.height(16.dp))
   }
}
```

如需验证可组合项的滚动行为，请限制预览的高度，并在[互动式预览](https://developer.android.com/jetpack/compose/tooling?hl=zh-cn#preview-interactive)中运行它：

```auto
@Preview(showBackground = true, backgroundColor = 0xFFF0EAE2, heightDp = 180)
@Composable
fun ScreenContentPreview() {
   MySootheTheme { HomeScreen() }
}
```
## 11\. 底部导航栏 - Material

现在，您已经实现了屏幕内容，可以开始添加窗口装饰了。就 MySoothe 而言，用户可以通过底部导航栏在不同的屏幕之间切换。

首先，实现此底部导航栏可组合项本身，然后将其添加到应用中。

下面我们来看一下设计：

<img src="/images/compose_layout/30.png" alt="30.png" style="zoom:50%;" />

幸运的是，您无需自己从头开始实现整个可组合项。您可以使用 Compose Material 库中的 [`BottomNavigation`](https://developer.android.com/reference/kotlin/androidx/compose/material/package-summary?hl=zh-cn#BottomNavigation(androidx.compose.ui.Modifier,androidx.compose.ui.graphics.Color,androidx.compose.ui.graphics.Color,androidx.compose.ui.unit.Dp,kotlin.Function1)) 可组合项。在 `BottomNavigation` 可组合项内，您可以添加一个或多个 [`BottomNavigationItem`](https://developer.android.com/reference/kotlin/androidx/compose/material/package-summary?hl=zh-cn#(androidx.compose.foundation.layout.RowScope).BottomNavigationItem(kotlin.Boolean,kotlin.Function0,kotlin.Function0,androidx.compose.ui.Modifier,kotlin.Boolean,kotlin.Function0,kotlin.Boolean,androidx.compose.foundation.interaction.MutableInteractionSource,androidx.compose.ui.graphics.Color,androidx.compose.ui.graphics.Color)) 元素，然后 Material 库会自动为其设置样式。

先从此底部导航栏的基本实现着手：

```auto
import androidx.compose.material.BottomNavigation
import androidx.compose.material.BottomNavigationItem
import androidx.compose.material.icons.filled.AccountCircle
import androidx.compose.material.icons.filled.Spa

@Composable
private fun SootheBottomNavigation(modifier: Modifier = Modifier) {
   BottomNavigation(modifier) {
       BottomNavigationItem(
           icon = {
               Icon(
                   imageVector = Icons.Default.Spa,
                   contentDescription = null
               )
           },
           label = {
               Text(stringResource(R.string.bottom_navigation_home))
           },
           selected = true,
           onClick = {}
       )
       BottomNavigationItem(
           icon = {
               Icon(
                   imageVector = Icons.Default.AccountCircle,
                   contentDescription = null
               )
           },
           label = {
               Text(stringResource(R.string.bottom_navigation_profile))
           },
           selected = false,
           onClick = {}
       )
   }
}
```

此基本实现如下所示：

<img src="/images/compose_layout/31.png" alt="31.png" style="zoom:50%;" />

您应该进行一些样式调整。首先，您可以通过设置底部导航栏的 `backgroundColor` 形参来更新其背景颜色。为此，您可以使用 Material 主题中的背景颜色。通过设置背景颜色，图标和文本的颜色会自动适应主题的 `onBackground` 颜色。最终解决方案应如下所示：

```auto
@Composable
private fun SootheBottomNavigation(modifier: Modifier = Modifier) {
   BottomNavigation(
       backgroundColor = MaterialTheme.colors.background,
       modifier = modifier
   ) {
       BottomNavigationItem(
           icon = {
               Icon(
                   imageVector = Icons.Default.Spa,
                   contentDescription = null
               )
           },
           label = {
               Text(stringResource(R.string.bottom_navigation_home))
           },
           selected = true,
           onClick = {}
       )
       BottomNavigationItem(
           icon = {
               Icon(
                   imageVector = Icons.Default.AccountCircle,
                   contentDescription = null
               )
           },
           label = {
               Text(stringResource(R.string.bottom_navigation_profile))
           },
           selected = false,
           onClick = {}
       )
   }
}
```

## 12\. MySoothe 应用 - Scaffold

在这最后一步中，创建全屏实现，包含底部导航栏。使用 Material 的 [`Scaffold`](https://developer.android.com/jetpack/compose/layouts/material?hl=zh-cn#scaffold) 可组合项。对于实现 Material Design 的应用，`Scaffold` 提供了**可配置的顶级可组合项**。它包含可用于各种 Material 概念的槽位，其中一个就是底部栏。在此底部栏中，您可以放置在上一步中创建的底部导航栏可组合项。

实现 MySootheApp 可组合项。这是应用的顶级可组合项，因此您应该：

+   应用 `MySootheTheme` Material 主题。
+   添加 `Scaffold`。
+   将底部栏设置为 `SootheBottomNavigation` 可组合项。
+   将内容设置为 `HomeScreen` 可组合项。

最终结果应如下所示：

```auto
import androidx.compose.material.Scaffold

@Composable
fun MySootheApp() {
   MySootheTheme {
       Scaffold(
           bottomBar = { SootheBottomNavigation() }
       ) { padding ->
           HomeScreen(Modifier.padding(padding))
       }
   }
}
```

您的实现现已完成！如果想要检查您实现的设计能否让像素完美呈现，可以下载下面的图片，然后将其与您自己的预览实现进行比较。

<img src="/images/compose_layout/32.png" alt="32.png" style="zoom:50%;" />

## 13\. 结尾

恭喜您，您已成功完成了此 Codelab，并详细了解了 Compose 中的布局。通过实现真实的设计，您了解了修饰符、对齐方式、排列方式、延迟布局、槽位 API、滚动行为以及 Material 组件。

## 文档

如需获得关于上述主题的更多信息和指导，请参阅以下文档：

+   [Compose 中的布局](https://developer.android.com/jetpack/compose/layouts?hl=zh-cn)
+   [修饰符](https://developer.android.com/jetpack/compose/modifiers?hl=zh-cn)