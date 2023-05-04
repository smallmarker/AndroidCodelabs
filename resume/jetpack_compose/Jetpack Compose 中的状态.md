## 1\. 准备工作

此 Codelab 介绍了在 Jetpack Compose 中使用[状态](https://developer.android.com/jetpack/compose/state?hl=zh-cn)的相关核心概念。您将学习应用的状态如何决定界面中的显示内容、Compose 如何使用不同的 API 在状态发生变化时更新界面、如何优化可组合函数的结构以及如何在 Compose 环境中使用 ViewModel。

## **前提条件**

+   具备 Kotlin 语法方面的知识。
+   掌握 Compose 基本知识（您可以先学习 [Jetpack Compose 教程](https://developer.android.com/jetpack/compose/tutorial?hl=zh-cn)）。
+   对架构组件的 [`ViewModel`](https://developer.android.com/topic/libraries/architecture/viewmodel?hl=zh-cn) 有基本的了解。

## **学习内容**

+   如何看待 Jetpack Compose 界面中的状态和事件?
+   Compose 如何使用状态确定要在屏幕上显示的元素？
+   什么是状态提升？
+   有状态可组合函数和无状态可组合函数如何运作？
+   Compose 如何使用 `State<T>` API 自动跟踪状态？
+   内存和内部状态如何用于可组合函数中？使用 `remember` 和 `rememberSaveable` API。
+   如何使用列表和状态？使用 `mutableStateListOf` 和 `toMutableStateList` API。
+   如何结合使用 `ViewModel` 与 Compose？

## **所需条件**

+   [最新版 Android Studio](https://developer.android.com/studio?hl=zh-cn)。

### 推荐/可选

+   阅读 [Compose 编程思想](https://developer.android.com/jetpack/compose/mental-model?hl=zh-cn)。
+   在学习此 Codelab 之前，请先完成 [Jetpack Compose 基础知识 Codelab](https://codelabs.developers.google.com/codelabs/jetpack-compose-basics/?hl=zh-cn)。我们将在此 Codelab 中对状态进行全面探讨。

## **构建内容**

您将实现一个简单的健康应用：

![1.png](/images/compose_state/1.png)

该应用具有两项主要功能：

+   用于跟踪饮水量的饮水计数器。
+   全天内的待完成健康任务列表。

## 2\. 进行设置

## 启动新的 Compose 项目

1.  如需启动新的 Compose 项目，请打开 Android Studio。
2.  如果您位于 **Welcome to Android Studio** 窗口中，请点击 **Start a new Android Studio project**。如果您已打开 Android Studio 项目，请从菜单栏中依次选择 **File > New > New Project**。
3.  对于新项目，请从可用模板中选择 **Empty Compose Activity**。

![2.png](/images/compose_state/2.png)

4.  点击 **Next**，然后配置项目并将其命名为“**BasicStateCodelab**”。

请确保您选择的 minimumSdkVersion 至少为 API 级别 21，这是 Compose 支持的最低 API 级别。

当您选择 **Empty Compose Activity** 模板时，Android Studio 会在您的项目中完成以下设置：

+   `MainActivity` 类，其中配置的可组合函数可在屏幕上显示一些文本。
+   `AndroidManifest.xml` 文件，用于定义应用的权限、组件和自定义资源。
+   `build.gradle` 和 `app/build.gradle` 文件包含 Compose 所需的选项和依赖项。

## 此 Codelab 的解决方案

您可以从 GitHub 获取 `BasicStateCodelab` 的解决方案代码：

$ git clone https://github.com/googlecodelabs/android-compose-codelabs

或者，您也可以[下载代码库 Zip 文件](https://github.com/googlecodelabs/android-compose-codelabs/archive/main.zip)。

您可以在 `BasicStateCodelab` 项目中找到解决方案代码。我们建议您按照自己的节奏逐步完成 Codelab，并在需要帮助时查看解决方案。在此 Codelab 的学习过程中，我们会为您提供需要添加到项目的代码段。

## 3\. Compose 中的状态

应用的“状态”是指可以随时间变化的任何值。这是一个非常宽泛的定义，从 [Room](https://developer.android.com/jetpack/androidx/releases/room?hl=zh-cn) 数据库到类的变量，全部涵盖在内。

所有 Android 应用都会向用户显示状态。下面是 Android 应用中的一些状态示例：

+   聊天应用中最新收到的消息。
+   用户的个人资料照片。
+   在项列表中的滚动位置。

现在开始编写健康应用。

为简单起见，在此 Codelab 期间：

+   您可以在 `app` 模块的根 `com.codelabs.basicstatecodelab` 软件包中添加所有 Kotlin 文件。不过，在正式版应用中，文件应采用逻辑结构放在子软件包中。
+   您将在代码段中以内嵌方式硬编码所有字符串。在真实应用中，它们应作为字符串资源添加到 `strings.xml` 文件中，并使用 Compose 的 [`stringResource`](https://developer.android.com/jetpack/compose/resources?hl=zh-cn#strings) API 进行引用。

您需要构建的第一项功能是饮水计数器，用于记录您一天喝了多少杯水。

创建一个名为 `WaterCounter` 的可组合函数，其中包含一个 [`Text`](https://developer.android.com/reference/kotlin/androidx/compose/material/package-summary?hl=zh-cn#Text(kotlin.String,androidx.compose.ui.Modifier,androidx.compose.ui.graphics.Color,androidx.compose.ui.unit.TextUnit,androidx.compose.ui.text.font.FontStyle,androidx.compose.ui.text.font.FontWeight,androidx.compose.ui.text.font.FontFamily,androidx.compose.ui.unit.TextUnit,androidx.compose.ui.text.style.TextDecoration,androidx.compose.ui.text.style.TextAlign,androidx.compose.ui.unit.TextUnit,androidx.compose.ui.text.style.TextOverflow,kotlin.Boolean,kotlin.Int,kotlin.Function1,androidx.compose.ui.text.TextStyle)) 可组合项，用于显示饮水杯数。饮水杯数应存储在名为 `count` 的值中，您现在可以对其进行硬编码。

创建一个包含 `WaterCounter` 可组合函数的新文件 `WaterCounter.kt`，如下所示：

```auto
import androidx.compose.foundation.layout.padding
import androidx.compose.material.Text
import androidx.compose.runtime.Composable
import androidx.compose.ui.Modifier
import androidx.compose.ui.unit.dp

@Composable
fun WaterCounter(modifier: Modifier = Modifier) {
   val count = 0
   Text(
       text = "You've had $count glasses.",
       modifier = modifier.padding(16.dp)
   )
}
```

让我们创建一个表示整个屏幕的可组合函数，该函数将包括两个部分，即饮水计数器和健康任务列表。现阶段，我们只需添加计数器。

1.  创建一个代表主屏幕的文件 `WellnessScreen.kt`，然后调用 `WaterCounter` 函数：

```auto
import androidx.compose.runtime.Composable
import androidx.compose.ui.Modifier

@Composable
fun WellnessScreen(modifier: Modifier = Modifier) {
   WaterCounter(modifier)
}
```

2.  打开 `MainActivity.kt`。移除 `Greeting` 和 `DefaultPreview` 可组合项。在 activity 的 `setContent` 块内调用新创建的 `WellnessScreen` 可组合项，如下所示：

```auto
class MainActivity : ComponentActivity() {
   override fun onCreate(savedInstanceState: Bundle?) {
       super.onCreate(savedInstanceState)
       setContent {
           BasicStateCodelabTheme {
               // A surface container using the 'background' color from the theme
               Surface(
                   modifier = Modifier.fillMaxSize(),
                   color = MaterialTheme.colors.background
               ) {
                   WellnessScreen()
               }
           }
       }
   }
}
```

3.  如果现在运行应用，您会看到我们的基本饮水计数器屏幕，其中包含硬编码的饮水杯数。

![3.png](/images/compose_state/3.png)

`WaterCounter` 可组合函数的状态为变量 `count`。但是，静态状态无法修改，因此用处不大。如需解决此问题，请添加 [`Button`](https://developer.android.com/reference/kotlin/androidx/compose/material3/package-summary?hl=zh-cn#Button(kotlin.Function0,androidx.compose.ui.Modifier,kotlin.Boolean,androidx.compose.foundation.interaction.MutableInteractionSource,androidx.compose.material3.ButtonElevation,androidx.compose.ui.graphics.Shape,androidx.compose.foundation.BorderStroke,androidx.compose.material3.ButtonColors,androidx.compose.foundation.layout.PaddingValues,kotlin.Function1)) 以增加计数并跟踪全天的饮水杯数。

任何会导致状态修改的操作都称为“**事件**”，我们将在下一部分中对此进行详细介绍。

## 4\. Compose 中的事件

“状态”是指可以随时间变化的任何值，例如，聊天应用最新收到的消息。但是，是什么原因导致状态更新呢？在 Android 应用中，状态会根据事件进行更新。

事件是从应用外部或内部生成的输入，例如：

+   用户与界面互动，例如按下按钮。
+   其他因素，例如传感器发送新值或网络响应。

**应用的状态说明了要在界面中显示的内容，而事件则是一种机制，可在状态发生变化时导致界面发生变化。**

事件用于通知程序发生了某事。所有 Android 应用都有核心界面更新循环，如下所示：

![4.png](/images/compose_state/4.png)

+   事件：由用户或程序的其他部分生成。
+   更新状态：事件处理脚本会更改界面所使用的状态。
+   显示状态：界面会更新以显示新状态。

Compose 中的状态管理主要是了解状态和事件之间的交互方式。

现在，添加按钮以便用户通过添加更多饮水杯数来修改状态。

找到 `WaterCounter` 可组合函数，将 `Button` 添加到标签 `Text` 的下方。[`Column`](https://developer.android.com/reference/kotlin/androidx/glance/layout/package-summary?hl=zh-cn#column) 可帮助您垂直对齐 `Text` 与 `Button` 可组合项。您可以将外部内边距移至 `Column` 可组合项，并在 `Button` 的顶部添加一些额外的内边距，使其与 Text 分离。

`Button` 可组合函数接收 `onClick` [lambda 函数](https://kotlinlang.org/docs/lambdas.html) - 用户点击按钮时发生会发生此事件。稍后您会看到更多 lambda 函数示例。

将 `count` 更改为 `var`（而不是 `val`），使其具有可变性。

```auto
import androidx.compose.material.Button
import androidx.compose.foundation.layout.Column

@Composable
fun WaterCounter(modifier: Modifier = Modifier) {
   Column(modifier = modifier.padding(16.dp)) {
       var count = 0
       Text("You've had $count glasses.")
       Button(onClick = { count++ }, Modifier.padding(top = 8.dp)) {
           Text("Add one")
       }
   }
}
```

运行应用并点击按钮时，您会发现没有任何反应。为 `count` 变量设置不同的值不会使 Compose 将其检测为状态更改，因此不会产生任何效果。这是因为，当状态发生变化时，您尚未指示 Compose 应重新绘制屏幕（即“重组”可组合函数）。您将在接下来的步骤中解决此问题。

![5.gif](/images/compose_state/5.gif)

## 5\. 可组合函数中的记忆功能

Compose 应用通过调用可组合函数将数据转换为界面。**组合**是指 Compose 在执行可组合项时构建的界面描述。如果发生状态更改，Compose 会使用新状态重新执行受影响的可组合函数，从而创建更新后的界面。这一过程称为**重组**。Compose 还会查看各个可组合项需要哪些数据，以便仅重组数据发生了变化的组件，而避免重组未受影响的组件。

为此，**Compose 需要知道要跟踪的状态**，以便在收到更新时安排重组。

**Compose 采用特殊的状态跟踪系统，可以为读取特定状态的任何可组合项安排重组**。这让 Compose 能够实现精细控制，并且仅重组需要更改的可组合函数，而不是重组整个界面。这将通过同时跟踪针对状态的“写入”（即状态变化）和针对状态的“读取”来实现。

使用 Compose 的 [`State`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/State?hl=zh-cn) 和 [`MutableState`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/MutableState?hl=zh-cn) 类型让 Compose 能够观察到状态。

Compose 会跟踪每个读取状态 `value` 属性的可组合项，并在其 `value` 更改时触发重组。您可以使用 [`mutableStateOf`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/package-summary?hl=zh-cn#mutableStateOf(kotlin.Any,androidx.compose.runtime.SnapshotMutationPolicy)) 函数来创建可观察的 `MutableState`。它接受初始值作为封装在 `State` 对象中的参数，这样便可使其 `value` 变为可观察。

更新 `WaterCounter` 可组合项，以便 `count` 以 `0` 为初始值来使用 `mutableStateOf` API。当 `mutableStateOf` 返回 `MutableState` 类型时，您可以更新其 `value` 以更新状态，并且 Compose 会在其 `value` 被读取时触发这些函数的重组。

```auto
import androidx.compose.runtime.MutableState
import androidx.compose.runtime.mutableStateOf

@Composable
fun WaterCounter(modifier: Modifier = Modifier) {
   Column(modifier = modifier.padding(16.dp)) {
      // Changes to count are now tracked by Compose
       val count: MutableState<Int> = mutableStateOf(0)

       Text("You've had ${count.value} glasses.")
        Button(onClick = { count.value++ }, Modifier.padding(top = 8.dp)) {
           Text("Add one")
       }
   }
}
```

如前所述，对 `count` 所做的任何更改都会安排对自动重组读取 `count` 的 `value` 的所有可组合函数进行重组。在此情况下，点击按钮即会触发重组 `WaterCounter`。

如果现在运行应用，您会再次发现没有发生任何变化！

![6.gif](/images/compose_state/6.gif)

安排重组的过程没有问题。不过，当重组发生时，变量 `count` 会重新初始化为 0，因此我们需要通过某种方式在重组后保留此值。

为此，我们可以使用 [`remember`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/package-summary?hl=zh-cn#remember(kotlin.Function0)) 可组合内嵌函数。系统会在初始组合期间将由 **`remember`** 计算的值存储在组合中，并在重组期间一直保持存储的值。

`remember` 和 `mutableStateOf` 通常在可组合函数中一起使用。

还可以采用一些等效的编码方法，详见 [Compose 状态文档](https://developer.android.com/jetpack/compose/state?hl=zh-cn#state-in-composables)。

修改 `WaterCounter`，将对 `mutableStateOf` 的调用置于 `remember` 内嵌可组合函数的内部。

```auto
import androidx.compose.runtime.remember

@Composable
fun WaterCounter(modifier: Modifier = Modifier) {
    Column(modifier = modifier.padding(16.dp)) {
        val count: MutableState<Int> = remember { mutableStateOf(0) }
        Text("You've had ${count.value} glasses.")
        Button(onClick = { count.value++ }, Modifier.padding(top = 8.dp)) {
            Text("Add one")
        }
    }
}
```

或者，我们也可以使用 Kotlin 的[委托属性](https://kotlinlang.org/docs/delegated-properties.html)来简化 `count` 的使用。

您可以使用关键字 **by** 将 `count` 定义为 var。通过添加委托的 getter 和 setter 导入内容，我们可以间接读取 `count` 并将其设置为可变，而无需每次都显式引用 `MutableState` 的 `value` 属性。

现在，`WaterCounter` 如下所示：

```auto
import androidx.compose.runtime.getValue
import androidx.compose.runtime.setValue

@Composable
fun WaterCounter(modifier: Modifier = Modifier) {
   Column(modifier = modifier.padding(16.dp)) {
       var count by remember { mutableStateOf(0) }

       Text("You've had $count glasses.")
       Button(onClick = { count++ }, Modifier.padding(top = 8.dp)) {
           Text("Add one")
       }
   }
}
```

您选择的语法应该能够在您编写的可组合项中生成可读性最高的代码。

我们来回顾一下到目前为止的成果：

+   定义了一个持续记忆的变量，名称为 `count`。
+   创建了一个文本显示区，其中向用户呈现记忆的数字。
+   添加了一个按钮，每次点击该按钮都会导致记忆的数字递增。

这种安排可与用户形成数据流反馈循环：

+   界面向用户显示状态（当前计数显示为文本）。
+   用户生成的事件会与现有状态合并以生成新状态（点击按钮会为当前计数加一）

您的计数器已就绪且可正常运行！

![7.gif](/images/compose_state/7.gif)

## 6\. 状态驱动型界面

Compose 是一个声明性界面框架。它描述界面在特定状况下的状态，而不是在状态发生变化时移除界面组件或更改其可见性。调用重组并更新界面后，可组合项最终可能会进入或退出组合。

![8.png](/images/compose_state/8.png)

此方法可避免像针对视图系统那样手动更新视图的复杂性。这也不太容易出错，因为您不会忘记根据新状态更新视图，因为系统会自动执行此过程。

如果在初始组合期间或重组期间调用了可组合函数，则认为其**存在于**组合中。未调用的可组合函数（例如，由于该函数在 **if** 语句内调用且未满足条件）**不存在于**组合中。

如需详细了解[可组合项的生命周期](https://developer.android.com/jetpack/compose/lifecycle?hl=zh-cn#lifecycle-overview)，请参阅文档。

组合的输出是描述界面的树结构。

[Android Studio 的布局检查器工具](https://developer.android.com/studio/debug/layout-inspector?hl=zh-cn)可用于检查 Compose 生成的应用布局。我们接下来将执行此操作。

为了演示此过程，请修改代码，以根据状态显示界面。打开 `WaterCounter`，如果 `count` 大于 0，则显示 `Text`：

```auto
@Composable
fun WaterCounter(modifier: Modifier = Modifier) {
   Column(modifier = modifier.padding(16.dp)) {
       var count by remember { mutableStateOf(0) }

       if (count > 0) {
           // This text is present if the button has been clicked
           // at least once; absent otherwise
           Text("You've had $count glasses.")
       }
       Button(onClick = { count++ }, Modifier.padding(top = 8.dp)) {
           Text("Add one")
       }
   }
}
```

运行应用，然后依次选择 **Tools > Layout Inspector**，打开 Android Studio 的布局检查器工具。

您会看到一个分屏：左侧是组件树，右侧是应用预览。

点按屏幕左侧的根元素 `BasicStateCodelabTheme` 可浏览树。点击“Expand all”按钮，展开整个组件树。

点击屏幕右侧的某个元素即可访问树的相应元素。

![9.png](/images/compose_state/9.png)

如果您按应用上的 Add one 按钮：

+   计数增加到 1 且状态发生变化。
+   系统调用重组。
+   屏幕使用新元素重组。

现在，使用 Android Studio 的布局检查器工具检查组件树，您还会看到 `Text` 可组合项：

![10.png](/images/compose_state/10.png)

**状态驱动界面在给定时刻显示哪些元素。**

界面的不同部分可以依赖于相同的状态。修改 `Button`，使其在 `count` 达到 10 之前处于启用状态，并在达到 10 之后停用（即您达到当天的目标）。为此，请使用 `Button` 的 `enabled` 参数。

```auto
@Composable
fun WaterCounter(modifier: Modifier = Modifier) {
    ...
        Button(onClick = { count++ }, Modifier.padding(top = 8.dp), enabled = count < 10) {
    ...
}
```

现在运行应用。对 `count` 状态的更改决定是否显示 `Text`，以及是启用还是停用 `Button`。

![11.gif](/images/compose_state/11.gif)

## 7\. 组合中的记忆功能

[`remember`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/package-summary?hl=zh-cn#remember(kotlin.Function0)) 会将对象存储在组合中，而如果在重组期间未再次调用之前调用 `remember` 的来源位置，则会忘记对象。

为了直观呈现这种行为，我们将在应用中实现以下功能：当用户至少饮用了一杯水时，向用户显示有一项待执行的健康任务，同时用户也可以关闭此任务。由于可组合项应较小并可重复使用，因此请创建一个名为 `WellnessTaskItem` 的新可组合项，该可组合项根据以参数形式接收的字符串来显示健康任务，并显示一个 Close 图标按钮。

创建一个新文件 `WellnessTaskItem.kt`，并添加以下代码。您稍后将在此 Codelab 中使用此可组合函数。

```auto
import androidx.compose.foundation.layout.Row
import androidx.compose.material.Icon
import androidx.compose.material.IconButton
import androidx.compose.material.Text
import androidx.compose.material.icons.Icons
import androidx.compose.material.icons.filled.Close
import androidx.compose.runtime.Composable
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.unit.dp
import androidx.compose.foundation.layout.padding

@Composable
fun WellnessTaskItem(
    taskName: String,
    onClose: () -> Unit,
    modifier: Modifier = Modifier
) {
    Row(
        modifier = modifier, verticalAlignment = Alignment.CenterVertically
    ) {
        Text(
            modifier = Modifier.weight(1f).padding(start = 16.dp),
            text = taskName
        )
        IconButton(onClick = onClose) {
            Icon(Icons.Filled.Close, contentDescription = "Close")
        }
    }
}
```

`WellnessTaskItem` 函数会接收任务说明和 `onClose` lambda 函数（就像内置 `Button` 可组合项接收 `onClick` 一样）。

`WellnessTaskItem` 如下所示：

![12.png](/images/compose_state/12.png)

接下来为应用添加更多功能，请更新 `WaterCounter`，以在 `count` > 0 时显示 `WellnessTaskItem`。

当 `count` 大于 0 时，定义一个变量 `showTask`，用于确定是否显示 `WellnessTaskItem` 并将其初始化为 true。

添加新的 **if** 语句，以在 `showTask` 为 true 时显示 `WellnessTaskItem`。使用之前部分介绍的 API 来确保 `showTask` 值在重组后继续有效。

```auto
@Composable
fun WaterCounter() {
   Column(modifier = Modifier.padding(16.dp)) {
       var count by remember { mutableStateOf(0) }
       if (count > 0) {
           var showTask by remember { mutableStateOf(true) }
           if (showTask) {
               WellnessTaskItem(
                   onClose = { },
                   taskName = "Have you taken your 15 minute walk today?"
               )
           }
           Text("You've had $count glasses.")
       }

       Button(onClick = { count++ }, enabled = count < 10) {
           Text("Add one")
       }
   }
}
```

使用 `WellnessTaskItem` 的 `onClose` lambda 函数实现：在按下 X 按钮时，变量 `showTask` 更改为 `false`，且不再显示任务。

```auto
   ...
   WellnessTaskItem(
      onClose = { showTask = false },
      taskName = "Have you taken your 15 minute walk today?"
   )
   ...
```

接下来，添加一个带“Clear water count”文本的新 `Button`，并将其放置在“Add one”`Button` 旁边。[`Row`](https://developer.android.com/reference/androidx/leanback/widget/Row?hl=zh-cn) 可帮助对齐这两个按钮。您还可以向 `Row` 添加一些内边距。按下“Clear water count”按钮后，变量 `count` 会重置为 0。

`WaterCounter` 可组合函数应如下所示。

```auto
import androidx.compose.foundation.layout.Row

@Composable
fun WaterCounter(modifier: Modifier = Modifier) {
   Column(modifier = modifier.padding(16.dp)) {
       var count by remember { mutableStateOf(0) }
       if (count > 0) {
           var showTask by remember { mutableStateOf(true) }
           if (showTask) {
               WellnessTaskItem(
                   onClose = { showTask = false },
                   taskName = "Have you taken your 15 minute walk today?"
               )
           }
           Text("You've had $count glasses.")
       }

       Row(Modifier.padding(top = 8.dp)) {
           Button(onClick = { count++ }, enabled = count < 10) {
               Text("Add one")
           }
           Button(onClick = { count = 0 }, Modifier.padding(start = 8.dp)) {
               Text("Clear water count")
           }
       }
   }
}
```

运行应用时，屏幕会显示初始状态：

| ![img](/images/compose_state/13.png) | ![组件树示意图，显示了应用的初始状态，计数为 0](/images/compose_state/13_1.png) |
| ------------------------------------ | ------------------------------------------------------------ |

右侧是简化版组件树，可帮助您分析状态发生变化时会发生什么情况。`count` 和 `showTask` 是记住的值。

现在，您可以在应用中按以下步骤操作：

+   按下 Add one 按钮。此操作会递增 `count`（这会导致重组），并同时显示 `WellnessTaskItem` 和计数器 `Text`。

![组件树示意图，显示了状态变化，当用户点击 Add one 按钮时，系统会显示包含提示的文本和包含饮水杯数的文本。](/images/compose_state/14.png)

![9257d150a5952931.png](/images/compose_state/14_1.png)

+   按下 `WellnessTaskItem` 组件的 X（这会导致另一项重组）。`showTask` 现在为 false，这意味着不再显示 `WellnessTaskItem`。

![组件树示意图，指示当用户点击关闭按钮时，任务可组合项会消失。](/images/compose_state/15.png)

![6bf6d3991a1c9fd1.png](/images/compose_state/15_1.png)

+   按下 Add one 按钮（另一项重组）。如果您继续增加杯数，`showTask` 会记住您在下一次重组时关闭了 `WellnessTaskItem`。

| ![img](/images/compose_state/16.png) | ![img](/images/compose_state/16_1.png) |
| ------------------------------------ | -------------------------------------- |

 

+   按下 Clear water count 按钮可将 `count` 重置为 0 并导致重组。系统不会调用显示 `count` 的 `Text` 以及与 `WellnessTaskItem` 相关的所有代码，并且会退出组合。

![ae993e6ddc0d654a.png](/images/compose_state/17.png)

+   由于系统未调用之前调用 `showTask` 的代码位置，因此会忘记 `showTask`。这将返回第一步。

| ![img](/images/compose_state/18.png) | ![img](/images/compose_state/18_1.png) |
| ------------------------------------ | -------------------------------------- |

+   按下 Add one 按钮，使 `count` 大于 0（重组）。

![9257d150a5952931.png](/images/compose_state/19.png)

+   系统再次显示 `WellnessTaskItem` 可组合项，因为在退出上述组合时，之前的 `showTask` 值已被忘记。

如果我们要求 `showTask` 在 `count` 重置为 0 之后持续保留超过 `remember` 允许的时间（也就是说，即使重组期间未调用之前调用 `remember` 的代码位置），会发生什么？在接下来的部分中，我们将探讨如何修正这些问题以及更多示例。

现在，您已经了解了界面和状态在退出组合后的重置过程，请清除代码并返回到本部分开头的 `WaterCounter`：

```auto
@Composable
fun WaterCounter(modifier: Modifier = Modifier) {
    Column(modifier = modifier.padding(16.dp)) {
        var count by remember { mutableStateOf(0) }
        if (count > 0) {
            Text("You've had $count glasses.")
        }
        Button(onClick = { count++ }, Modifier.padding(top = 8.dp), enabled = count < 10) {
            Text("Add one")
        }
    }
}
```

## 8\. 在 Compose 中恢复状态

运行应用，为计数器增加一些饮水杯数，然后旋转设备。请确保已为设备启用自动屏幕旋转设置。

由于系统会在配置更改后（在本例中，即改变屏幕方向）重新创建 activity，因此已保存状态会被忘记：计数器会在重置为 0 后消失。

![34f1e94c368af98d.gif](/images/compose_state/20.gif)

如果您更改语言、在深色模式与浅色模式之间切换，或者执行任何导致 Android 重新创建运行中 activity 的其他配置更改时，也会发生相同的情况。

虽然 `remember` 可帮助您在重组后保持状态，但不会帮助您**在配置更改后保持状态**。为此，您必须使用 [`rememberSaveable`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/saveable/package-summary?hl=zh-cn#rememberSaveable(kotlin.Array,androidx.compose.runtime.saveable.Saver,kotlin.String,kotlin.Function0))，而不是 `remember`。

`rememberSaveable` 会自动保存可保存在 [`Bundle`](https://developer.android.com/reference/android/os/Bundle?hl=zh-cn) 中的任何值。对于其他值，您可以将其传入自定义 Saver 对象。如需详细了解如何[在 Compose 中恢复状态](https://developer.android.com/jetpack/compose/state?hl=zh-cn#restore-ui-state)，请参阅相关文档。

在 `WaterCounter` 中，将 `remember` 替换为 `rememberSaveable`：

```auto
import androidx.compose.runtime.saveable.rememberSaveable

@Composable
fun WaterCounter(modifier: Modifier = Modifier) {
        ...
        var count by rememberSaveable { mutableStateOf(0) }
        ...
}
```

现在运行应用并尝试进行一些配置更改。您应该会看到计数器已正确保存。

![45beaedccab5e331.gif](/images/compose_state/21.gif)

重新创建 activity 只是 `rememberSaveable` 的用例之一。我们稍后会在使用列表时探索另一个用例。

请根据应用的状态和用户体验需求来考虑是使用 `remember` 还是 `rememberSaveable`。

## 9\. 状态提升

使用 **`remember`** 存储对象的可组合项包含内部状态，这会使该可组合项**有状态**。在调用方不需要控制状态，并且不必自行管理状态便可使用状态的情况下，“有状态”会非常有用。但是，**具有内部状态的可组合项往往不易重复使用，也更难测试**。

**不保存任何状态的可组合项称为无状态可组合项**。如需创建**无状态**可组合项，一种简单的方法是使用状态提升。

Compose 中的状态提升是一种将状态移至可组合项的调用方以使可组合项无状态的模式。Jetpack Compose 中的常规状态提升模式是将状态变量替换为两个参数：

+   **value: T**：要显示的当前值
+   **onValueChange: (T) -> Unit**：请求更改值的事件，其中 T 是建议的新值

其中，此值表示任何可修改的状态。

以这种方式提升的状态具有一些重要的属性：

+   **单一可信来源**：通过移动状态，而不是复制状态，我们可确保只有一个可信来源。这有助于避免 bug。
+   **可共享**：可与多个可组合项共享提升的状态。
+   **可拦截**：无状态可组合项的调用方可以在更改状态之前决定忽略或修改事件。
+   **分离**：无状态可组合函数的状态可以存储在任何位置。例如，存储在 ViewModel 中。

请尝试为 `WaterCounter` 实现状态提升，以便从以上所有方法中受益。

## 有状态与无状态

当所有状态都可以从可组合函数中提取出来时，生成的可组合函数称为无状态函数。

重构 `WaterCounter` 可组合项，将其拆分为两部分：有状态和无状态计数器。

`StatelessCounter` 的作用是显示 `count`，并在您递增 `count` 时调用函数。为此，请遵循上述模式并传递状态 `count`（作为可组合函数的参数）和 lambda (`onIncrement`)（在需要递增状态时会调用此函数）。`StatelessCounter` 如下所示：

```auto
@Composable
fun StatelessCounter(count: Int, onIncrement: () -> Unit, modifier: Modifier = Modifier) {
   Column(modifier = modifier.padding(16.dp)) {
       if (count > 0) {
           Text("You've had $count glasses.")
       }
       Button(onClick = onIncrement, Modifier.padding(top = 8.dp), enabled = count < 10) {
           Text("Add one")
       }
   }
}
```

`StatefulCounter` 拥有状态。这意味着，它会存储 `count` 状态，并在调用 `StatelessCounter` 函数时对其进行修改。

```auto
@Composable
fun StatefulCounter(modifier: Modifier = Modifier) {
   var count by rememberSaveable { mutableStateOf(0) }
   StatelessCounter(count, { count++ }, modifier)
}
```

太棒了！您已将 `count` 从 `StatelessCounter` 提升到 `StatefulCounter`。

您可以将其插入到应用中，并使用 `StatefulCounter` 更新 `WellnessScreen`：

```auto
@Composable
fun WellnessScreen(modifier: Modifier = Modifier) {
   StatefulCounter(modifier)
}
```

如前所述，状态提升具有一些好处。我们将探索此代码的不同变体并详细介绍其中一些变体，**您无需在应用中复制以下代码段**。

1.  **您的无状态可组合项现在已可重复使用**。请看以下示例。

如需记录饮用水和果汁的杯数，请记住 `waterCount` 和 `juiceCount`，但请使用示例 `StatelessCounter` 可组合函数来显示两种不同的独立状态。

```auto
@Composable
fun StatefulCounter() {
    var waterCount by remember { mutableStateOf(0) }

    var juiceCount by remember { mutableStateOf(0) }

    StatelessCounter(waterCount, { waterCount++ })
    StatelessCounter(juiceCount, { juiceCount++ })
}
```

![8211bd9e0a4c5db2.png](/images/compose_state/22.png)

如果修改了 `juiceCount`，则重组 `StatefulCounter`。在重组期间，Compose 会识别哪些函数读取 `juiceCount`，并触发系统仅重组这些函数。

![2cb0dcdbe75dcfbf.png](/images/compose_state/23.png)

当用户点按以递增 `juiceCount` 时，系统会重组 `StatefulCounter`，同时也会重组 `juiceCount` 的 `StatelessCounter`。但不会重组读取 `waterCount` 的 `StatelessCounter`。

![7fe6ee3d2886abd0.png](/images/compose_state/24.png)

2.  **有状态可组合函数可以为多个可组合函数提供相同的状态**。

```auto
@Composable
fun StatefulCounter() {
   var count by remember { mutableStateOf(0) }

   StatelessCounter(count, { count++ })
   AnotherStatelessMethod(count, { count *= 2 })
}
```

在本例中，如果通过 `StatelessCounter` 或 `AnotherStatelessMethod` 更新计数，则系统会按预期重组所有项目。

由于可以共享提升的状态，因此请务必**仅传递可组合项所需的状态**，以避免不必要的重组并提高可重用性。

如需详细了解状态和状态提升，请参阅 [Compose 状态文档](https://developer.android.com/jetpack/compose/state?hl=zh-cn#state-hoisting)。

## 10\. 使用列表

接下来，添加应用的第二项功能，即健康任务列表。您可以对列表中的项执行以下两项操作：

+   勾选列表项，将任务标记为已完成。
+   从任务列表中移除不想完成的任务。

## 设置

1.  首先，修改列表项。您可以重复使用“组合中的记忆功能”部分中的 `WellnessTaskItem`，并将其更新为包含 [`Checkbox`](https://developer.android.com/reference/kotlin/androidx/compose/material/package-summary?hl=zh-cn#Checkbox(kotlin.Boolean,kotlin.Function1,androidx.compose.ui.Modifier,kotlin.Boolean,androidx.compose.foundation.interaction.MutableInteractionSource,androidx.compose.material.CheckboxColors))。请务必提升 `checked` 状态和 `onCheckedChange` 回调，使函数变为无状态。

![16b3672772c44395.png](/images/compose_state/25.png)

本部分的 `WellnessTaskItem` 可组合项应如下所示：

```auto
import androidx.compose.material.Checkbox

@Composable
fun WellnessTaskItem(
    taskName: String,
    checked: Boolean,
    onCheckedChange: (Boolean) -> Unit,
    onClose: () -> Unit,
    modifier: Modifier = Modifier
) {
    Row(
        modifier = modifier, verticalAlignment = Alignment.CenterVertically
    ) {
        Text(
            modifier = Modifier
                .weight(1f)
                .padding(start = 16.dp),
            text = taskName
        )
        Checkbox(
            checked = checked,
            onCheckedChange = onCheckedChange
        )
        IconButton(onClick = onClose) {
            Icon(Icons.Filled.Close, contentDescription = "Close")
        }
    }
}
```

2.  在同一文件中，添加一个有状态 `WellnessTaskItem` 可组合函数，用于定义状态变量 `checkedState` 并将其传递给同名的无状态方法。暂时不用担心 `onClose`，您可以传递空的 lambda 函数。

```auto
import androidx.compose.runtime.getValue
import androidx.compose.runtime.setValue
import androidx.compose.runtime.mutableStateOf
import androidx.compose.runtime.remember

@Composable
fun WellnessTaskItem(taskName: String, modifier: Modifier = Modifier) {
   var checkedState by remember { mutableStateOf(false) }

   WellnessTaskItem(
       taskName = taskName,
       checked = checkedState,
       onCheckedChange = { newValue -> checkedState = newValue },
       onClose = {}, // we will implement this later!
       modifier = modifier,
   )
}
```

3.  创建一个文件 `WellnessTask.kt`，对包含 ID 和标签的任务进行建模。将其定义为[数据类](https://kotlinlang.org/docs/data-classes.html)。

```auto
data class WellnessTask(val id: Int, val label: String)
```

4.  对于任务列表本身，请创建一个名为 `WellnessTasksList.kt` 的新文件，并添加一个方法用于生成一些虚假数据：

```auto
private fun getWellnessTasks() = List(30) { i -> WellnessTask(i, "Task # $i") }
```

请注意，在真实应用中，您将从[数据层](https://developer.android.com/jetpack/guide/data-layer?hl=zh-cn)获取数据。

5.  在 `WellnessTasksList.kt` 中，添加一个用于创建列表的可组合函数。定义 [`LazyColumn`](https://developer.android.com/reference/kotlin/androidx/compose/foundation/lazy/package-summary?hl=zh-cn#LazyColumn(androidx.compose.ui.Modifier,androidx.compose.foundation.lazy.LazyListState,androidx.compose.foundation.layout.PaddingValues,kotlin.Boolean,androidx.compose.foundation.layout.Arrangement.Vertical,androidx.compose.ui.Alignment.Horizontal,androidx.compose.foundation.gestures.FlingBehavior,kotlin.Boolean,kotlin.Function1)) 以及您所创建的列表方法中的列表项。如需帮助，请参阅[列表文档](https://developer.android.com/jetpack/compose/lists?hl=zh-cn)。

```auto
import androidx.compose.foundation.lazy.LazyColumn
import androidx.compose.foundation.lazy.items
import androidx.compose.runtime.Composable
import androidx.compose.ui.Modifier
import androidx.compose.runtime.remember

@Composable
fun WellnessTasksList(
    modifier: Modifier = Modifier,
    list: List<WellnessTask> = remember { getWellnessTasks() }
) {
    LazyColumn(
        modifier = modifier
    ) {
        items(list) { task ->
            WellnessTaskItem(taskName = task.label)
        }
    }
}
```

6.  将列表添加到 `WellnessScreen`。使用 `Column` 有助于列表与已有的计数器垂直对齐。

```auto
import androidx.compose.foundation.layout.Column

@Composable
fun WellnessScreen(modifier: Modifier = Modifier) {
   Column(modifier = modifier) {
       StatefulCounter()
       WellnessTasksList()
   }
}
```

7.  运行应用并试一下效果！现在，您应该能够勾选任务，但不能删除任务。我们将在稍后的部分中实现此功能。

![2903e244176c2dda.gif](/images/compose_state/26.gif)

## 在 LazyList 中恢复项状态

现在，我们来详细了解一下 `WellnessTaskItem` 可组合项中的一些内容。

`checkedState` 属于每个 `WellnessTaskItem` 可组合项，就像私有变量一样。当 `checkedState` 发生变化时，系统只会重组 `WellnessTaskItem` 的实例，而不是重组 `LazyColumn` 中的所有 `WellnessTaskItem` 实例。

请按以下步骤尝试应用的功能：

1.  勾选此列表顶部的所有元素（例如元素 1 和元素 2）。
2.  滚动到列表底部，使这些元素位于屏幕之外。
3.  滚动到顶部，查看之前勾选的列表项。
4.  请注意，它们处于未选中状态。

正如您在上一部分中看到的那样，其问题在于，当一个项退出组合时，系统会忘记之前记住的状态。对于 `LazyColumn` 上的项，当您滚动至项不可见的位置时，这些不可见的项会完全退出组合。

![d3c12f57cc98db16.gif](/images/compose_state/27.gif)

如何解决此问题？同样，使用 `rememberSaveable`。它采用保存的实例状态机制，可确保存储的值在重新创建 activity 或进程之后继续保留。得益于 `rememberSaveable` 与 `LazyList` 配合工作的方式，您的项在离开组合后也能继续保留。

只需在有状态 `WellnessTaskItem` 中将 `remember` 替换为 `rememberSaveable` 即可，如下所示：

```auto
import androidx.compose.runtime.saveable.rememberSaveable

var checkedState by rememberSaveable { mutableStateOf(false) }
```

![9b4ba365481ae97a.gif](/images/compose_state/28.gif)

## Compose 中的常见模式

请注意 [**`LazyColumn`**](https://developer.android.com/reference/kotlin/androidx/compose/foundation/lazy/package-summary?hl=zh-cn#LazyColumn(androidx.compose.ui.Modifier,androidx.compose.foundation.lazy.LazyListState,androidx.compose.foundation.layout.PaddingValues,kotlin.Boolean,androidx.compose.foundation.layout.Arrangement.Vertical,androidx.compose.ui.Alignment.Horizontal,androidx.compose.foundation.gestures.FlingBehavior,kotlin.Boolean,kotlin.Function1)) 的实现：

```auto
@Composable
fun LazyColumn(
...
    state: LazyListState = rememberLazyListState(),
...
```

可组合函数 `rememberLazyListState` 使用 `rememberSaveable` 为列表创建初始状态。重新创建 activity 后，无需任何编码即可保持滚动状态。

许多应用需要对滚动位置、列表项布局更改以及其他与列表状态相关的事件作出响应，并进行监听。延迟组件（例如 `LazyColumn` 或 `LazyRow`）可通过提升 [`LazyListState`](https://developer.android.com/reference/kotlin/androidx/compose/foundation/lazy/LazyListState?hl=zh-cn) 来支持此用例。如需详细了解此模式，请参阅[介绍列表中的状态的文档](https://developer.android.com/jetpack/compose/lists?hl=zh-cn#react-to-scroll-position)。

状态参数使用由公共 `rememberX` 函数提供的默认值是内置可组合函数中的常见模式。另一个示例可以在 [`Scaffold`](https://developer.android.com/reference/kotlin/androidx/compose/material/package-summary?hl=zh-cn#scaffold) 中找到，它使用 [`rememberScaffoldState`](https://developer.android.com/reference/kotlin/androidx/compose/material/package-summary?hl=zh-cn#rememberScaffoldState(androidx.compose.material.DrawerState,androidx.compose.material.SnackbarHostState)) 提升状态。

## 11\. 可观察的可变列表

接下来，如需添加从列表中移除任务的行为，第一步是让列表成为可变列表。

使用可变对象（例如 `ArrayList<T>` 或 `mutableListOf,`）对此不起作用。这些类型不会向 Compose 通知列表中的项已发生更改并安排界面重组。您需要使用使用其他 API。

您需要创建一个可由 Compose 观察的 `MutableList` 实例。此结构可允许 Compose 跟踪更改，以便在列表中添加或移除项时重组界面。

首先，定义可观察的 `MutableList`。扩展函数 [`toMutableStateList()`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/package-summary?hl=zh-cn#(kotlin.collections.Collection).toMutableStateList()) 用于根据初始可变或不可变的 `Collection`（例如 `List`）来创建可观察的 `MutableList`。

或者，您也可以使用工厂方法 [`mutableStateListOf`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/package-summary?hl=zh-cn#mutableStateListOf()) 来创建可观察的 `MutableList`，然后为初始状态添加元素。

1.  打开 `WellnessScreen.kt` 文件。将 `getWellnessTasks` 方法移至此文件中以便使用该方法。如需创建列表，请先调用 `getWellnessTasks()`，然后使用之前介绍的扩展函数 `toMutableStateList`。

```auto
import androidx.compose.runtime.remember
import androidx.compose.runtime.toMutableStateList

@Composable
fun WellnessScreen(modifier: Modifier = Modifier) {
   Column(modifier = modifier) {
       StatefulCounter()

       val list = remember { getWellnessTasks().toMutableStateList() }
       WellnessTasksList(list = list, onCloseTask = { task -> list.remove(task) })
   }
}

private fun getWellnessTasks() = List(30) { i -> WellnessTask(i, "Task # $i") }
```

2.  通过移除列表的默认值来修改 `WellnessTaskList` 可组合函数，因为列表会提升到屏幕级别。添加一个新的 lambda 函数参数 `onCloseTask`（用于接收 `WellnessTask` 以进行删除）。将 `onCloseTask` 传递给 `WellnessTaskItem`。

您还需要再进行一项更改。`items` 方法会接收一个 `key` 参数。默认情况下，每个项的状态均与该项在列表中的位置相对应。

在可变列表中，当数据集发生变化时，这会导致问题，因为实际改变位置的项会丢失任何记住的状态。

使用每个 `WellnessTaskItem` 的 `id` 作为每个项的键，即可轻松解决此问题。

如需详细了解[列表中的项键](https://developer.android.com/jetpack/compose/lists?hl=zh-cn)，请参阅相关文档。

`WellnessTaskList` 将如下所示：

```auto
@Composable
fun WellnessTasksList(
   list: List<WellnessTask>,
   onCloseTask: (WellnessTask) -> Unit,
   modifier: Modifier = Modifier
) {
   LazyColumn(modifier = modifier) {
       items(
           items = list,
           key = { task -> task.id }
       ) { task ->
           WellnessTaskItem(taskName = task.label, onClose = { onCloseTask(task) })
       }
   }
}
```

3.  修改 `WellnessTaskItem`：将 `onClose` lambda 函数作为参数添加到有状态 `WellnessTaskItem` 中并进行调用。

```auto
@Composable
fun WellnessTaskItem(
   taskName: String, onClose: () -> Unit, modifier: Modifier = Modifier
) {
   var checkedState by rememberSaveable { mutableStateOf(false) }

   WellnessTaskItem(
       taskName = taskName,
       checked = checkedState,
       onCheckedChange = { newValue -> checkedState = newValue },
       onClose = onClose,
       modifier = modifier,
   )
}
```

太棒了！此功能已经完成，现在已经可以从列表中删除项。

如果您点击每行中的 X，则事件会一直到达拥有状态的列表，从列表中移除相应项，并导致 Compose 重组界面。

![47f4a64c7e9a5083.png](/images/compose_state/29.png)

如果您尝试使用 `rememberSaveable()` 将列表存储在 `WellnessScreen` 中，则会发生运行时异常：

此错误消息指出，您需要提供[自定义 Saver](https://developer.android.com/jetpack/compose/state?hl=zh-cn#restore-ui-state)。但是，您不应使用 **`rememberSaveable`** 来存储需要长时间序列化或反序列化操作的大量数据或复杂数据结构。

使用 activity 的 [`onSaveInstanceState`](https://developer.android.com/reference/android/app/Activity?hl=zh-cn#onSaveInstanceState(android.os.Bundle)) 时，应遵循类似的规则；如需了解详情，请参阅[保存界面状态文档](https://developer.android.com/topic/libraries/architecture/saving-states?hl=zh-cn#onsaveinstancestate)。如果要执行此操作，您需要替代存储机制。如需详细了解其他[保留界面状态的选项](https://developer.android.com/topic/libraries/architecture/saving-states?hl=zh-cn#options)，请参阅相关文档。

接下来，我们来看看 ViewModel 在存储应用状态方面的作用。

## 12\. ViewModel 中的状态

屏幕或界面状态指示应在屏幕上显示的内容（例如任务列表）。**该状态通常会与层次结构中的其他层相关联，原因是其包含应用数据**。

界面状态描述屏幕上显示的内容，而应用逻辑则描述应用的行为方式以及应如何响应状态变化。逻辑分为两种类型：第一种是界面行为或界面逻辑，第二种是业务逻辑。

+   界面逻辑涉及如何在屏幕上显示状态变化（例如导航逻辑或显示信息提示控件）。
+   业务逻辑决定如何处理状态更改（例如付款或存储用户偏好设置）。该逻辑通常位于业务层或数据层，但绝不会位于界面层。

[ViewModel](https://developer.android.com/topic/libraries/architecture/viewmodel?hl=zh-cn) 提供界面状态以及对位于应用其他层中的业务逻辑的访问。此外，ViewModel 还会在配置更改后继续保留，因此其生命周期比组合更长。ViewModel 可以遵循 Compose 内容（即 activity 或 fragment）的主机的生命周期，也可以遵循导航图的目的地的生命周期（如果您使用的是 [Compose Navigation 库](https://developer.android.com/jetpack/compose/navigation?hl=zh-cn)）。

如需详细了解架构和界面层，请参阅[界面层文档](https://developer.android.com/jetpack/guide/ui-layer?hl=zh-cn#define-ui-state)。

## 迁移列表并移除方法

让我们将界面状态（列表）迁移到 ViewModel，并开始将业务逻辑提取到 ViewModel 中。

1.  创建文件 `WellnessViewModel.kt` 以添加 ViewModel 类。

将“数据源”`getWellnessTasks()` 移至 `WellnessViewModel`。

像前面一样使用 `toMutableStateList` 定义内部 `_tasks` 变量，并将 `tasks` 作为列表公开，这样将无法从 ViewModel 外部对其进行修改。

实现一个简单的 `remove` 函数，用于委托给列表的内置 remove 函数。

```auto
import androidx.compose.runtime.toMutableStateList
import androidx.lifecycle.ViewModel

class WellnessViewModel : ViewModel() {
    private val _tasks = getWellnessTasks().toMutableStateList()
    val tasks: List<WellnessTask>
        get() = _tasks

   fun remove(item: WellnessTask) {
       _tasks.remove(item)
   }
}

private fun getWellnessTasks() = List(30) { i -> WellnessTask(i, "Task # $i") }
```

2.  我们可以通过调用 [`viewModel()`](https://developer.android.com/reference/kotlin/androidx/lifecycle/viewmodel/compose/package-summary?hl=zh-cn#viewModel(androidx.lifecycle.ViewModelStoreOwner,kotlin.String,androidx.lifecycle.ViewModelProvider.Factory,androidx.lifecycle.viewmodel.CreationExtras)) 函数，从任何可组合项访问此 ViewModel。

如需使用此函数，请打开 `app/build.gradle` 文件，添加以下库，并在 Android Studio 中同步新的依赖项：

```auto
implementation "androidx.lifecycle:lifecycle-viewmodel-compose:{latest_version}"
```

您可以点击[此处](https://developer.android.com/jetpack/androidx/releases/lifecycle?hl=zh-cn)查看最新版本。

3.  打开 `WellnessScreen`。实例化 `wellnessViewModel` ViewModel，方法是以 Screen 可组合项的参数的形式调用 `viewModel()`，以便在测试此可组合项时进行替换，并根据需要进行提升。为 `WellnessTasksList` 提供任务列表，并为 `onCloseTask` lambda 提供 remove 函数。

```auto
import androidx.lifecycle.viewmodel.compose.viewModel

@Composable
fun WellnessScreen(
    modifier: Modifier = Modifier,
    wellnessViewModel: WellnessViewModel = viewModel()
) {
   Column(modifier = modifier) {
       StatefulCounter()

       WellnessTasksList(
           list = wellnessViewModel.tasks,
           onCloseTask = { task -> wellnessViewModel.remove(task) })
   }
}
```

`viewModel()` 会返回一个现有的 `ViewModel`，或在给定作用域内创建一个新的 ViewModel。只要作用域处于活动状态，ViewModel 实例就会一直保留。例如，如果在某个 activity 中使用了可组合项，则在该 activity 完成或进程终止之前，`viewModel()` 会返回同一实例。

大功告成！您已将 ViewModel 与部分状态和业务逻辑集成到了屏幕上。由于状态保留在组合之外并由 ViewModel 存储，因此对列表的更改在配置更改后继续有效。

ViewModel 在任何情况下（例如，对于系统发起的进程终止）都不会自动保留应用的状态。如需详细了解如何[保留应用的界面状态](https://developer.android.com/topic/libraries/architecture/saving-states?hl=zh-cn)，请参阅相关文档。

## 迁移选中状态

最后一个重构是将选中状态和逻辑迁移到 ViewModel。这样一来，代码将变得更简单且更易于测试，并且所有状态均由 ViewModel 管理。

1.  首先，修改 `WellnessTask` 模型类，使其能够存储选中状态并将 false 设置为默认值。

```auto
data class WellnessTask(val id: Int, val label: String, var checked: Boolean = false)
```

2.  在 ViewModel 中，实现一个 `changeTaskChecked` 方法，该方法将接收使用选中状态的新值进行修改的任务。

```auto
class WellnessViewModel : ViewModel() {
   ...
   fun changeTaskChecked(item: WellnessTask, checked: Boolean) =
       tasks.find { it.id == item.id }?.let { task ->
           task.checked = checked
       }
}
```

3.  在 `WellnessScreen` 中，通过调用 ViewModel 的 `changeTaskChecked` 方法为列表的 `onCheckedTask` 提供行为。函数现在应如下所示：

```auto
@Composable
fun WellnessScreen(
    modifier: Modifier = Modifier,
    wellnessViewModel: WellnessViewModel = viewModel()
) {
   Column(modifier = modifier) {
       StatefulCounter()

       WellnessTasksList(
           list = wellnessViewModel.tasks,
           onCheckedTask = { task, checked ->
               wellnessViewModel.changeTaskChecked(task, checked)
           },
           onCloseTask = { task ->
               wellnessViewModel.remove(task)
           }
       )
   }
}
```

4.  打开 `WellnessTasksList` 并添加 `onCheckedTask` lambda 函数参数，以便将其传递给 `WellnessTaskItem.`

```auto
@Composable
fun WellnessTasksList(
   list: List<WellnessTask>,
   onCheckedTask: (WellnessTask, Boolean) -> Unit,
   onCloseTask: (WellnessTask) -> Unit,
   modifier: Modifier = Modifier
) {
   LazyColumn(
       modifier = modifier
   ) {
       items(
           items = list,
           key = { task -> task.id }
       ) { task ->
           WellnessTaskItem(
               taskName = task.label,
               checked = task.checked,
               onCheckedChange = { checked -> onCheckedTask(task, checked) },
               onClose = { onCloseTask(task) }
           )
       }
   }
}
```

5.  清理 `WellnessTaskItem.kt` 文件。我们不再需要有状态方法，因为 CheckBox 状态将提升到列表级别。该文件仅包含以下可组合函数：

```auto
@Composable
fun WellnessTaskItem(
   taskName: String,
   checked: Boolean,
   onCheckedChange: (Boolean) -> Unit,
   onClose: () -> Unit,
   modifier: Modifier = Modifier
) {
   Row(
       modifier = modifier, verticalAlignment = Alignment.CenterVertically
   ) {
       Text(
           modifier = Modifier
               .weight(1f)
               .padding(start = 16.dp),
           text = taskName
       )
       Checkbox(
           checked = checked,
           onCheckedChange = onCheckedChange
       )
       IconButton(onClick = onClose) {
           Icon(Icons.Filled.Close, contentDescription = "Close")
       }
   }
}
```

6.  运行应用并尝试勾选任何任务。您会发现无法勾选任何任务。

![8e2e731c58123cd8.gif](/images/compose_state/30.gif)

这是因为 Compose 将跟踪 `MutableList` 与添加和移除元素相关的更改。这就是删除功能能够正常运行的原因。但是，它对行项的值（在本例中为 `checkedState`）的更改一无所知，除非您指示它跟踪这些值。

解决此问题的方法有两种：

+   更改数据类 `WellnessTask`，使 `checkedState` 变为 `MutableState<Boolean>`（而非 `Boolean`），这会使 Compose 跟踪项更改。
+   复制您要更改的项，从列表中移除相应项，然后将更改后的项重新添加到列表中，这会使 Compose 跟踪该列表的更改。

这两种方法各有利弊。例如，根据您所使用的列表的实现，移除和读取该元素可能会产生非常高的开销。

因此，假设您想要避免可能开销高昂的列表操作，并将 `checkedState` 设为可观察，因为这种方式更高效且更符合 Compose 的规范。

您的新 `WellnessTask` 应如下所示：

```auto
import androidx.compose.runtime.MutableState
import androidx.compose.runtime.mutableStateOf

data class WellnessTask(val id: Int, val label: String, val checked: MutableState<Boolean> = mutableStateOf(false))
```

如前所述，在本例中，您可以使用委托属性，这样可以更轻松地使用变量 `checked`。

将 `WellnessTask` 更改为类，而不是数据类。让 `WellnessTask` 在构造函数中接收默认值为 `false` 的 `initialChecked` 变量，然后可以使用工厂方法 `mutableStateOf` 来初始化 `checked` 变量并接受 `initialChecked` 作为默认值。

```auto
import androidx.compose.runtime.getValue
import androidx.compose.runtime.mutableStateOf
import androidx.compose.runtime.setValue

class WellnessTask(
    val id: Int,
    val label: String,
    initialChecked: Boolean = false
) {
    var checked by mutableStateOf(initialChecked)
}
```

大功告成！这种解决方案行之有效，并且所有更改在重组和配置更改后仍然保持有效！

![c8e049d56e7b0eac.gif](/images/compose_state/31.gif)

## 测试

现在，业务逻辑已重构为 ViewModel，而不是在可组合函数内形成耦合，因此单元测试要简单得多。

您可以使用插桩测试来验证 Compose 代码的正确行为以及界面状态是否正常运行。建议学习 Codelab [Compose 中的测试](https://developer.android.com/codelabs/jetpack-compose-testing?hl=zh-cn#0)，了解如何测试 Compose 界面。

## 13\. 结尾

太棒了！您已成功完成此 Codelab，并学习了用于在 Jetpack Compose 应用中处理状态的所有基本 API！

您了解了如何使用状态和事件在 Compose 中提取无状态可组合项，以及 Compose 如何使用状态更新来促使界面发生变化。

## [示例应用](https://github.com/android/compose-samples/)

+   [JetNews](https://github.com/android/compose-samples/tree/master/JetNews) 演示了此 Codelab 中介绍的最佳实践。

## 更多文档

+   [Compose 编程思想](https://developer.android.com/jetpack/compose/mental-model?hl=zh-cn)
+   [状态和 Jetpack Compose](https://developer.android.com/jetpack/compose/state?hl=zh-cn)
+   [Jetpack Compose 中的单向数据流](https://developer.android.com/jetpack/compose/architecture?hl=zh-cn#udf-compose)
+   [在 Compose 中恢复状态](https://developer.android.com/jetpack/compose/state?hl=zh-cn#restore-ui-state)
+   [ViewModel 概览](https://developer.android.com/topic/libraries/architecture/viewmodel?hl=zh-cn)
+   [Compose 和其他库](https://developer.android.com/jetpack/compose/libraries?hl=zh-cn)

## 参考 API

+   [`remember`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/package-summary.html?hl=zh-cn#remember(kotlin.Function0))
+   [`rememberSaveable`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/saveable/package-summary?hl=zh-cn#rememberSaveable(kotlin.Array,androidx.compose.runtime.saveable.Saver,kotlin.String,kotlin.Function0))
+   [`State`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/State?hl=zh-cn)
+   [`MutableState`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/MutableState?hl=zh-cn)
+   [`SnapshotStateList`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/snapshots/SnapshotStateList?hl=zh-cn)
+   [`ViewModel`](https://developer.android.com/topic/libraries/architecture/viewmodel?hl=zh-cn)