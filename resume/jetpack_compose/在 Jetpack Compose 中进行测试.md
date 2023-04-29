## 1\. 简介与设置

在此 Codelab 中，您将了解如何测试使用 [Jetpack Compose](https://developer.android.com/jetpack/compose?hl=zh-cn) 创建的界面。您将编写您的第一批测试，并在此过程中了解隔离测试、调试测试、语义树和同步。

## 所需条件

+   [最新版 Android Studio](https://developer.android.com/studio?hl=zh-cn)
+   了解 Kotlin
+   基本了解 Compose（如 `@Composable` 注解）
+   基本熟悉[修饰符](https://developer.android.com/jetpack/compose/layouts/basics?hl=zh-cn#modifiers)
+   可选：建议您在学习此 Codelab 之前先学习 [Jetpack Compose 基础知识 Codelab](https://codelabs.developers.google.com/codelabs/jetpack-compose-basics/?hl=zh-cn)

### 查看此 Codelab 的代码 (Rally)

您将使用 [Rally Material 研究](https://material.io/design/material-studies/rally.html#about-rally)作为此 Codelab 的基础。可在 android-compose-codelabs GitHub 代码库中找到此代码。如需克隆，请运行以下命令：

```auto
git clone https://github.com/googlecodelabs/android-compose-codelabs.git
```

下载完成后，打开 `TestingCodelab` 项目。

或者，您也可以下载两个 ZIP 文件：

+   [起始代码](https://github.com/googlecodelabs/android-compose-codelabs/archive/refs/heads/main.zip)
+   [解决方法](https://github.com/googlecodelabs/android-compose-codelabs/archive/refs/heads/end.zip)

打开 TestingCodelab 文件夹，其中包含一个名为 Rally 的应用。

## 查看项目结构

Compose 测试是插桩测试。这意味着，这些测试需要在设备（物理设备或模拟器）上运行。

Rally 已包含一些插桩界面测试。您可以在 androidTest 源代码集中找到这些测试：

![rally.png](/images/jetpack_compose_test/rally.png)

这是用于保存新测试的目录。您可以查看 `AnimatingCircleTests.kt` 文件来了解 Compose 测试的内容。

Rally 已配置好，但若要在新项目中启用 Compose 测试，只需在相关模块的 `build.gradle` 文件中设置测试依赖项即可，具体为：

```auto
androidTestImplementation "androidx.compose.ui:ui-test-junit4:$version"

debugImplementation "androidx.compose.ui:ui-test-manifest:$rootProject.composeVersion"
```

您可以运行应用来熟悉一下。

## 2\. 测试内容

我们将重点测试 Rally 的标签页栏，该栏包含一行[标签页](https://material.io/components/tabs)（“Overview”、“Accounts”和“Bills”）。此界面在实际使用场景中如下所示：

![test_content.gif](/images/jetpack_compose_test/test_content.gif)

在此 Codelab 中，您将测试该栏的界面。

此测试包含许多内容：

+   测试标签页是否会显示预期图标和文本
+   测试动画是否符合规范
+   测试触发的导航事件是否正确
+   测试界面元素在不同状态下的放置位置和距离
+   截取该栏的屏幕截图，并将其与之前截取的屏幕截图进行比较

对组件的测试程度或测试方式并没有具体的规定。您可以进行上述所有测试！在此 Codelab 中，您将通过验证以下各项来测试状态逻辑是否正确：

+   **标签页是否仅在选中状态下才显示标签**
+   **当前屏幕是否明确显示出了处于选中状态的标签页**

## 3\. 创建简单的界面测试

## 创建 TopAppBarTest 文件

在 `AnimatingCircleTests.kt` 所在的文件夹 (`app/src/androidTest/com/example/compose/rally`) 中创建一个新文件，并将其命名为 `TopAppBarTest.kt`。

Compose 提供一个 `ComposeTestRule`，调用 `createComposeRule()` 即可获得此规则。您可以通过此规则设置被测 Compose 内容并与其交互。

## 添加 ComposeTestRule

```auto
package com.example.compose.rally

import androidx.compose.ui.test.junit4.createComposeRule
import org.junit.Rule

class TopAppBarTest {

    @get:Rule
    val composeTestRule = createComposeRule()

    // TODO: Add tests
}
```

## 隔离测试

在 Compose 测试中，我们可以启动应用的主 activity，操作方式与您在 Android View 环境中使用 Espresso 执行同样的操作类似。您可以使用 `createAndroidComposeRule` 执行此操作。

```auto
// Don't copy this over

@get:Rule
val composeTestRule = createAndroidComposeRule(RallyActivity::class.java)
```

不过，在 Compose 中，我们可以通过对组件进行隔离测试来大幅简化测试工作。您可以选择要在测试中使用的 Compose 界面内容。这可通过 `ComposeTestRule` 的 `setContent` 方法完成，并且您可以在任何位置调用它（但只能调用一次）。

```auto
// Don't copy this over

class TopAppBarTest {

    @get:Rule
    val composeTestRule = createComposeRule()

    @Test
    fun myTest() {
        composeTestRule.setContent {
            Text("You can set any Compose content!")
        }
    }
}
```

我们要测试 TopAppBar，所以它是我们的关注重点。在 `setContent` 内调用 `RallyTopAppBar`，并让 Android Studio 补全参数的名称。

```auto
import androidx.compose.ui.test.junit4.createComposeRule
import com.example.compose.rally.ui.components.RallyTopAppBar
import org.junit.Rule
import org.junit.Test

class TopAppBarTest {

    @get:Rule
    val composeTestRule = createComposeRule()

    @Test
    fun rallyTopAppBarTest() {
        composeTestRule.setContent {
            RallyTopAppBar(
                allScreens = ,
                onTabSelected = { /*TODO*/ },
                currentScreen =
            )
        }
    }
}
```

## 可测试的可组合项的重要性

`RallyTopAppBar` 采用三个很容易提供的参数，因此我们可以传递由我们控制的虚构数据。例如：

```auto
    @Test
    fun rallyTopAppBarTest() {
        val allScreens = RallyScreen.values().toList()
        composeTestRule.setContent {
            RallyTopAppBar(
                allScreens = allScreens,
                onTabSelected = { },
                currentScreen = RallyScreen.Accounts
            )
        }
        Thread.sleep(5000)
    }
```

我们还添加了一个 `sleep()`，让您了解到底发生了什么。右键点击 `rallyTopAppBarTest`，然后点击“Run rallyTopAppBarTest()”。

![app_bar_test.png](/images/jetpack_compose_test/app_bar_test.png)

该测试显示了顶部应用栏（5 秒钟），但它与我们的预期不同：它使用的是浅色主题！

其原因在于，该栏是使用 Material 组件构建的，因此它应当在 [MaterialTheme](https://developer.android.com/reference/kotlin/androidx/compose/material/MaterialTheme?hl=zh-cn) 内，否则就会回退到“基准”样式的颜色。

[`MaterialTheme`](https://developer.android.com/reference/kotlin/androidx/compose/material/MaterialTheme?hl=zh-cn) 具有合适的默认设置，故而测试没有崩溃。由于我们不准备测试主题或截取屏幕截图，因此可以忽略此问题而继续使用默认的浅色主题。不过，如果您愿意，也可以使用 `RallyTheme` 封装 `RallyTopAppBar` 来解决此问题。

## 验证标签页是否处于选中状态

查找界面元素、检查其属性和执行操作是按照以下模式通过测试规则完成的：

```auto
composeTestRule{.finder}{.assertion}{.action}
```

在此测试中，您将查找“Accounts”一词，以验证是否显示了处于选中状态的标签页的标签。

![app_bar_account.png](/images/jetpack_compose_test/app_bar_account.png)

如需了解有哪些工具可用，建议您参阅 [Compose 测试备忘单](https://developer.android.com/jetpack/compose/testing-cheatsheet?hl=zh-cn)或[测试软件包参考文档](https://developer.android.com/reference/kotlin/androidx/compose/ui/test/package-summary?hl=zh-cn)。查找在此测试的情况下可能有帮助的查找器和断言。例如：`onNodeWithText`、`onNodeWithContentDescription`、`isSelected`、`hasContentDescription`、`assertIsSelected`...

每个标签页都有一个不同的内容说明：

+   Overview
+   Accounts
+   Bills

知道这一点后，将 `Thread.sleep(5000)` 替换为用于查找内容说明并断言其存在的语句：

```auto
import androidx.compose.ui.test.assertIsSelected
import androidx.compose.ui.test.onNodeWithContentDescription
...

@Test
fun rallyTopAppBarTest_currentTabSelected() {
    val allScreens = RallyScreen.values().toList()
    composeTestRule.setContent {
        RallyTopAppBar(
            allScreens = allScreens,
            onTabSelected = { },
            currentScreen = RallyScreen.Accounts
        )
    }

    composeTestRule
        .onNodeWithContentDescription(RallyScreen.Accounts.name)
        .assertIsSelected()
}
```

现在再次运行测试，这次测试应该会通过：

![compose_run.png](/images/jetpack_compose_test/compose_run.png)

**恭喜**！您已成功编写您的第一项 Compose 测试。在此过程中，您了解了如何进行隔离测试，以及如何使用查找器和断言。

这些内容很简单，但需要以前对相关组件（内容说明和 selected 属性）有所了解。下一步，您将了解如何检查有哪些属性可用。

## 4\. 调试测试

在以下步骤中，您将验证是否显示了当前标签页的标签（它全部使用大写）。

![account_test.png](/images/jetpack_compose_test/account_test.png)

一种可能的解决方法是尝试查找标签的文本并断言其存在：

```auto
import androidx.compose.ui.test.onNodeWithText
...

@Test
fun rallyTopAppBarTest_currentLabelExists() {
    val allScreens = RallyScreen.values().toList()
    composeTestRule.setContent {
        RallyTopAppBar(
            allScreens = allScreens,
            onTabSelected = { },
            currentScreen = RallyScreen.Accounts
        )
    }

    composeTestRule
        .onNodeWithText(RallyScreen.Accounts.name.uppercase())
        .assertExists()
}
```

不过，如果您运行该测试，结果会失败 😱

![account_run.png](/images/jetpack_compose_test/account_run.png)

在以下步骤中，您将了解如何使用语义树对该测试进行调试。

### 语义树

Compose 测试使用称为[**语义树**](https://developer.android.com/jetpack/compose/testing?hl=zh-cn#semantics)的结构来查找屏幕上的元素并读取其属性。这也是无障碍服务使用的结构，因为无障碍服务按理会被 [TalkBack](https://support.google.com/accessibility/android/answer/6283677?hl=zh-cn) 等服务读取。

您可以对节点使用 `printToLog` 函数来输出语义树。将一行新代码添加到测试中：

```auto
import androidx.compose.ui.test.onRoot
import androidx.compose.ui.test.printToLog
...

fun rallyTopAppBarTest_currentLabelExists() {
    val allScreens = RallyScreen.values().toList()
    composeTestRule.setContent {
        RallyTopAppBar(
            allScreens = allScreens,
            onTabSelected = { },
            currentScreen = RallyScreen.Accounts
        )
    }

    composeTestRule.onRoot().printToLog("currentLabelExists")

    composeTestRule
        .onNodeWithText(RallyScreen.Accounts.name.toUpperCase())
        .assertExists() // Still fails
}
```

现在，运行测试并在 Android Studio 中查看 Logcat（您可以查找 `currentLabelExists`）。

```auto
...com.example.compose.rally D/currentLabelExists: printToLog:
    Printing with useUnmergedTree = 'false'
    Node #1 at (l=0.0, t=63.0, r=1080.0, b=210.0)px
     |-Node #2 at (l=0.0, t=63.0, r=1080.0, b=210.0)px
       [SelectableGroup]
       MergeDescendants = 'true'
        |-Node #3 at (l=42.0, t=105.0, r=105.0, b=168.0)px
        | Role = 'Tab'
        | Selected = 'false'
        | StateDescription = 'Not selected'
        | ContentDescription = 'Overview'
        | Actions = [OnClick]
        | MergeDescendants = 'true'
        | ClearAndSetSemantics = 'true'
        |-Node #6 at (l=189.0, t=105.0, r=468.0, b=168.0)px
        | Role = 'Tab'
        | Selected = 'true'
        | StateDescription = 'Selected'
        | ContentDescription = 'Accounts'
        | Actions = [OnClick]
        | MergeDescendants = 'true'
        | ClearAndSetSemantics = 'true'
        |-Node #11 at (l=552.0, t=105.0, r=615.0, b=168.0)px
          Role = 'Tab'
          Selected = 'false'
          StateDescription = 'Not selected'
          ContentDescription = 'Bills'
          Actions = [OnClick]
          MergeDescendants = 'true'
          ClearAndSetSemantics = 'true'
```

查看语义树可以看到有一个 `SelectableGroup` 包含 3 个子元素，这些子元素就是顶部应用栏的标签页。结果显示，没有哪一个 `text` 属性的值为“ACCOUNTS”，这正是测试失败的原因。不过，**每个标签页都有内容说明**。您可以在 `RallyTopAppBar.kt` 内查看 `RallyTab` 可组合项中对此属性是如何设置的：

```auto
private fun RallyTab(text: String...)
...
    Modifier
        .clearAndSetSemantics { contentDescription = text }
```

此修饰符会清除来自后代的属性并设置自己的内容说明，因此您会看到“Accounts”而不是“ACCOUNTS”。

将查找器 `onNodeWithText` 替换为 `onNodeWithContentDescription`，然后再次运行测试：

```auto
fun rallyTopAppBarTest_currentLabelExists() {
    val allScreens = RallyScreen.values().toList()
    composeTestRule.setContent {
        RallyTopAppBar(
            allScreens = allScreens,
            onTabSelected = { },
            currentScreen = RallyScreen.Accounts
        )
    }

    composeTestRule
        .onNodeWithContentDescription(RallyScreen.Accounts.name)
        .assertExists()
}
```

![test_rule.png](/images/jetpack_compose_test/test_rule.png)

恭喜！您已修复了测试，并了解了 `ComposeTestRule`、隔离测试、查找器、断言和使用语义树进行调试。

**不过，也有一个坏消息**：这项测试并不是很有用！仔细查看语义树就会发现，无论三个标签页是否处于选中状态，它们的内容说明都会显示在树中。我们必须继续深入！

## 5\. 合并和未合并的语义树

语义树总是会尽可能地精简，仅显示相关的信息。

例如，在我们的 `TopAppBar` 中，没有必要将图标和标签作为不同的节点。我们来看看“Overview”节点：

![selectable.png](/images/jetpack_compose_test/selectable.png)

```auto
        |-Node #3 at (l=42.0, t=105.0, r=105.0, b=168.0)px
        | Role = 'Tab'
        | Selected = 'false'
        | StateDescription = 'Not selected'
        | ContentDescription = 'Overview'
        | Actions = [OnClick]
        | MergeDescendants = 'true'
        | ClearAndSetSemantics = 'true'
```

此节点包含专门为 [`selectable`](https://developer.android.com/reference/kotlin/androidx/compose/foundation/selection/package-summary?hl=zh-cn#selectable(androidx.compose.ui.Modifier,kotlin.Boolean,kotlin.Boolean,androidx.compose.ui.semantics.Role,kotlin.Function0)) 组件定义的属性（例如 `Selected` 和 `Role`），以及对整个标签页的内容说明。这些都是很笼统的属性，对简单的测试非常有用。有关图标或文本的详细信息在此是多余的，因此不会显示。

Compose 会在某些可组合项（例如 `Text`）中自动公开这些语义属性。您也可以自定义和**合并**这些属性，用来表示由一个或多个后代组成的单个组件。例如，您可以表示一个包含 `Text` 可组合项的 `Button`。属性 `MergeDescendants = 'true'` 表示，**此节点有后代，但已合并到此节点中**。在测试中，我们常常需要访问所有节点。

为了验证标签页中的 `Text` 是否会显示，我们可以将 `useUnmergedTree = true` 传递给 `onRoot` 查找器，查询**未合并的**语义树。

```auto
@Test
fun rallyTopAppBarTest_currentLabelExists() {
    val allScreens = RallyScreen.values().toList()
    composeTestRule.setContent {
        RallyTopAppBar(
            allScreens = allScreens,
            onTabSelected = { },
            currentScreen = RallyScreen.Accounts
        )
    }

    composeTestRule.onRoot(useUnmergedTree = true).printToLog("currentLabelExists")

}
```

现在，Logcat 中的输出稍长一些：

```auto
    Printing with useUnmergedTree = 'true'
    Node #1 at (l=0.0, t=63.0, r=1080.0, b=210.0)px
     |-Node #2 at (l=0.0, t=63.0, r=1080.0, b=210.0)px
       [SelectableGroup]
       MergeDescendants = 'true'
        |-Node #3 at (l=42.0, t=105.0, r=105.0, b=168.0)px
        | Role = 'Tab'
        | Selected = 'false'
        | StateDescription = 'Not selected'
        | ContentDescription = 'Overview'
        | Actions = [OnClick]
        | MergeDescendants = 'true'
        | ClearAndSetSemantics = 'true'
        |-Node #6 at (l=189.0, t=105.0, r=468.0, b=168.0)px
        | Role = 'Tab'
        | Selected = 'true'
        | StateDescription = 'Selected'
        | ContentDescription = 'Accounts'
        | Actions = [OnClick]
        | MergeDescendants = 'true'
        | ClearAndSetSemantics = 'true'
        |  |-Node #9 at (l=284.0, t=105.0, r=468.0, b=154.0)px
        |    Text = 'ACCOUNTS'
        |    Actions = [GetTextLayoutResult]
        |-Node #11 at (l=552.0, t=105.0, r=615.0, b=168.0)px
          Role = 'Tab'
          Selected = 'false'
          StateDescription = 'Not selected'
          ContentDescription = 'Bills'
          Actions = [OnClick]
          MergeDescendants = 'true'
          ClearAndSetSemantics = 'true'
```

节点 3 仍没有后代：

```auto
        |-Node #3 at (l=42.0, t=105.0, r=105.0, b=168.0)px
        | Role = 'Tab'
        | Selected = 'false'
        | StateDescription = 'Not selected'
        | ContentDescription = 'Overview'
        | Actions = [OnClick]
        | MergeDescendants = 'true'
        | ClearAndSetSemantics = 'true'
```

但是，节点 6（处于选中状态的标签页）有一个后代，我们现在可以看到“Text”属性：

```auto
        |-Node #6 at (l=189.0, t=105.0, r=468.0, b=168.0)px
        | Role = 'Tab'
        | Selected = 'true'
        | StateDescription = 'Selected'
        | ContentDescription = 'Accounts'
        | Actions = [OnClick]
        | MergeDescendants = 'true'
        |  |-Node #9 at (l=284.0, t=105.0, r=468.0, b=154.0)px
        |    Text = 'ACCOUNTS'
        |    Actions = [GetTextLayoutResult]
```

为了验证我们希望看到的正确行为，您将编写一个匹配器，用于查找一个文本为“ACCOUNTS”且父节点的内容说明为“Accounts”的节点。

再次查看 [Compose 测试备忘单](https://developer.android.com/jetpack/compose/testing-cheatsheet?hl=zh-cn)，并试着设法编写该匹配器。请注意，您可以将 `and` 和 `or` 等布尔运算符与匹配器结合起来使用。

所有查找器都有一个名为 `useUnmergedTree` 的参数。将其设为 `true` 可使用未合并的树。

请试着不看解决方法自行编写该测试！

#### *解决方法*

```auto
import androidx.compose.ui.test.hasParent
import androidx.compose.ui.test.hasText
...

@Test
fun rallyTopAppBarTest_currentLabelExists() {
    val allScreens = RallyScreen.values().toList()
    composeTestRule.setContent {
        RallyTopAppBar(
            allScreens = allScreens,
            onTabSelected = { },
            currentScreen = RallyScreen.Accounts
        )
    }

    composeTestRule
        .onNode(
            hasText(RallyScreen.Accounts.name.uppercase()) and
            hasParent(
                hasContentDescription(RallyScreen.Accounts.name)
            ),
            useUnmergedTree = true
        )
        .assertExists()
}
```

开始运行测试：

![selectable_run.png](/images/jetpack_compose_test/selectable_run.png)

恭喜！您已在这一步中了解了属性合并以及合并和未合并的语义树。

## 6\. 同步

您编写的任何测试都必须与被测对象正确同步。例如，当您使用 `onNodeWithText` 等查找器时，测试会一直等到应用进入空闲状态后才查询语义树。如果不同步，测试就可能会在元素显示之前查找元素，或者不必要地等待。

这一步我们将使用“Overview”屏幕，在应用运行时，此屏幕如下所示：

![sync_page.gif](/images/jetpack_compose_test/sync_page.gif)

请注意，“Alerts”卡片显示了反复闪烁的动画，吸引用户注意该元素。

创建另一个名为 `OverviewScreenTest` 的测试类，并添加以下内容：

```auto
package com.example.compose.rally

import androidx.compose.ui.test.assertIsDisplayed
import androidx.compose.ui.test.junit4.createComposeRule
import androidx.compose.ui.test.onNodeWithText
import com.example.compose.rally.ui.overview.OverviewBody
import org.junit.Rule
import org.junit.Test

class OverviewScreenTest {

    @get:Rule
    val composeTestRule = createComposeRule()

    @Test
    fun overviewScreen_alertsDisplayed() {
        composeTestRule.setContent {
            OverviewBody()
        }

        composeTestRule
            .onNodeWithText("Alerts")
            .assertIsDisplayed()
    }
}
```

如果运行此测试，您会发现测试永远无法完成（它会在 30 秒后超时）。

![sync_run.png](/images/jetpack_compose_test/sync_run.png)

错误消息如下：

```auto
androidx.compose.ui.test.junit4.android.ComposeNotIdleException: Idling resource timed out: possibly due to compose being busy.
IdlingResourceRegistry has the following idling resources registered:
- [busy] androidx.compose.ui.test.junit4.android.ComposeIdlingResource@d075f91
```

此消息大致表示 Compose 一直处于忙碌状态，因此没有办法与测试同步应用。

您可能已经猜到了，问题出在无穷无尽的闪烁动画上。应用永不空闲，因此测试无法继续。

我们来看看这个无穷尽动画的实现：

### app/src/main/java/com/example/compose/rally/ui/overview/OverviewBody.kt

```auto
var currentTargetElevation by remember {  mutableStateOf(1.dp) }
LaunchedEffect(Unit) {
    // Start the animation
    currentTargetElevation = 8.dp
}
val animatedElevation = animateDpAsState(
    targetValue = currentTargetElevation,
    animationSpec = tween(durationMillis = 500),
    finishedListener = {
        currentTargetElevation = if (currentTargetElevation > 4.dp) {
            1.dp
        } else {
            8.dp
        }
    }
)
Card(elevation = animatedElevation.value) { ... }
```

此代码本质上是等待一个动画结束 (`finishedListener`)，然后再次运行该动画。

修复该测试的一种方法是在开发者选项中停用动画。在 `View` 环境中，这是处理此问题的一种公认方式。

在 Compose 中，由于设计动画 API 时就已考虑到可测试性，因此可以使用正确的 API 来解决该问题。我们可以使用[无限动画](https://developer.android.com/jetpack/compose/animation?hl=zh-cn#rememberinfinitetransition)，而不是重新开始 [`animateDpAsState`](https://developer.android.com/jetpack/compose/animation?hl=zh-cn#animate-as-state) 动画。

将 `OverviewScreen` 中的代码替换为正确的 API：

```auto
import androidx.compose.animation.core.RepeatMode
import androidx.compose.animation.core.VectorConverter
import androidx.compose.animation.core.animateValue
import androidx.compose.animation.core.infiniteRepeatable
import androidx.compose.animation.core.rememberInfiniteTransition
import androidx.compose.animation.core.tween
import androidx.compose.ui.unit.Dp
...

    val infiniteElevationAnimation = rememberInfiniteTransition()
    val animatedElevation: Dp by infiniteElevationAnimation.animateValue(
        initialValue = 1.dp,
        targetValue = 8.dp,
        typeConverter = Dp.VectorConverter,
        animationSpec = infiniteRepeatable(
            animation = tween(500),
            repeatMode = RepeatMode.Reverse
        )
    )
    Card(elevation = animatedElevation) {
```

如果您运行该测试，现在测试会通过：

![sync_result_run.png](/images/jetpack_compose_test/sync_result_run.png)

恭喜！您已在这一步中了解了同步以及动画对测试的影响。

## 7\. 选做练习

在以下步骤中，您将使用某项操作（请参阅[测试备忘单](https://developer.android.com/jetpack/compose/testing-cheatsheet?hl=zh-cn)）来验证点击 `RallyTopAppBar` 的不同标签页是否会更改选中状态。

提示：

+   该测试的范围需要包括由 `RallyApp` 拥有的状态。
+   验证状态，不要验证行为。对界面的状态使用断言，而不要依赖于已调用的对象和调用方式。

此练习不提供解决方法。

## 8\. 结尾

恭喜！您已完成**在 Jetpack Compose 中进行测试**。现在，您已经有了基本的构建块，可以针对 Compose 界面制定良好的测试策略了。

如果您想详细了解测试和 Compose。
