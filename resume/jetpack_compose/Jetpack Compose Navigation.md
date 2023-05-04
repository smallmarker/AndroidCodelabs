## 1\. 简介

## **所需条件**

+   [最新版 Android Studio](https://developer.android.com/studio?hl=zh-cn)
+   了解 Kotlin 和[尾随 lambda](https://kotlinlang.org/docs/lambdas.html#lambda-expression-syntax)
+   对导航及其术语（例如返回堆栈）有基本的了解
+   对 Compose 有基本的了解
+   建议您在学习此 Codelab 之前先完成[“Jetpack Compose 基础知识”Codelab](https://github.com/smallmarker/AndroidCodelabs/blob/main/resume/jetpack_compose/Jetpack%20Compose%20%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86.md)
+   对 Compose 中的状态管理有基本的了解
+   建议您在学习此 Codelab 之前先完成[“Jetpack Compose 中的状态”Codelab](https://github.com/smallmarker/AndroidCodelabs/blob/main/resume/jetpack_compose/Jetpack%20Compose%20%E4%B8%AD%E7%9A%84%E7%8A%B6%E6%80%81.md)

## **使用 Compose 进行导航**

[Navigation](https://developer.android.com/guide/navigation?hl=zh-cn) 是一个 Jetpack 库，用于在应用中从一个目的地导航到另一个目的地。Navigation 库还提供了一个专用工件，用于[使用 Jetpack Compose 实现一致而惯用的导航方式](https://developer.android.com/jetpack/compose/navigation?hl=zh-cn)。此 Codelab 会重点介绍这个工件 (`navigation-compose`)。

## **实践内容**

您将使用 [Rally Material 研究](https://material.io/design/material-studies/rally.html#about-rally)作为此 Codelab 的基础，从而实现 Jetpack Navigation 组件并在可组合的 Rally 屏幕之间实现导航。

## **学习内容**

+   将 Jetpack Navigation 与 Jetpack Compose 配合使用的基础知识
+   在可组合项之间导航
+   将自定义标签页栏可组合项集成到导航层次结构中
+   使用实参进行导航
+   使用深层链接进行导航
+   测试导航

## 2\. 设置

如需自行学习，请克隆此 Codelab 的起始代码（`main` 分支）。

$ git clone https://github.com/googlecodelabs/android-compose-codelabs.git

或者，您也可以下载两个 ZIP 文件：

+   [起始代码](https://github.com/googlecodelabs/android-compose-codelabs/archive/refs/heads/main.zip)
+   [解决方案](https://github.com/googlecodelabs/android-compose-codelabs/archive/refs/heads/end.zip)

现在，您已下载相应代码，请在 Android Studio 中打开 **NavigationCodelab** 项目文件夹。现已准备就绪，可以开始开发项目了。

## 3\. Rally 应用概览

首先，您应当熟悉 Rally 应用及其代码库。运行应用并稍加探索。

Rally 有三个主屏幕作为可组合项：

1.  **`OverviewScreen`** - 所有财务交易和提醒的概览
2.  **`AccountsScreen`** - 有关现有账户的数据分析
3.  **`BillsScreen`** - 预定支出

| ![img](/images/compose_navigation/1_1.png) | ![img](/images/compose_navigation/1_2.png) | ![img](/images/compose_navigation/1_3.png) |
| ------------------------------------------ | ------------------------------------------ | ------------------------------------------ |

在屏幕的最顶部，Rally 使用**自定义标签页栏可组合项 (**`RallyTabRow`**)** 在这三个屏幕之间进行导航。点按每个图标应该会展开当前所选内容，您也会转到其对应的屏幕：

| ![img](/images/compose_navigation/2.png) | ![img](/images/compose_navigation/2_1.png) |
| ---- | ---- |

在导航到这些可组合屏幕时，您还可以将它们视为导航目的地，因为我们希望在特定时间点到达各个目的地。这些目的地是在 `RallyDestinations.kt` 文件中预定义的。

在该文件中，您会找到所有三个定义为对象的主要目的地（`Overview, Accounts` 和 `Bills`），以及一个日后要添加到应用的 `SingleAccount`。每个对象都从 `RallyDestination` 接口扩展，并且包含有关每个目的地的必要信息，以便进行导航：

1.  顶栏的 `icon`
2.  字符串 `route`（这对 Compose Navigation 而言是必需的，可作为指向相应目的地的路径）
3.  `screen`，表示相应目的地的整个可组合项

运行应用时，您会发现自己实际上可以在当前使用顶栏的目的地之间导航。然而，应用实际上并未使用 Compose Navigation，但它当前使用的导航机制依靠手动切换一些可组合项和触发[重组](https://developer.android.com/jetpack/compose/mental-model?hl=zh-cn#recomposition)来显示新内容。因此，此 Codelab 的目标是成功迁移并实现 Compose Navigation。

## 4\. 迁移到 Compose Navigation

迁移到 Jetpack Compose 所涉及的基本步骤如下：

1.  添加最新的 [Compose Navigation 依赖项](https://mvnrepository.com/artifact/androidx.navigation/navigation-compose)
2.  设置 [`NavController`](https://developer.android.com/jetpack/compose/navigation?hl=zh-cn#getting-started)
3.  添加 [`NavHost`](https://developer.android.com/jetpack/compose/navigation?hl=zh-cn#create-navhost) 并创建导航图
4.  准备路线以在不同的应用目的地之间导航
5.  将当前导航机制替换为 Compose Navigation

下面我们将更详细地逐一介绍这些步骤。

## 添加 Navigation 依赖项

打开应用的 build 文件（位于 `app/build.gradle`）。在“dependencies”区段中，添加 `navigation-compose` 依赖项。

```auto
dependencies {
  implementation "androidx.navigation:navigation-compose:{latest_version}"
  // ...
}
```

您可以点击[此处](https://developer.android.com/jetpack/androidx/releases/navigation?hl=zh-cn)找到最新版 Navigation Compose。

现在，同步项目，然后您就可以开始使用 Compose 中的 Navigation 了。

## **设置 NavController**

使用 Compose 中的 Navigation 时，[`NavController`](https://developer.android.com/jetpack/compose/navigation?hl=zh-cn#getting-started) 是核心组件。它可跟踪返回堆栈可组合条目、使堆栈向前移动、支持对返回堆栈执行操作，以及在不同目的地状态之间导航。由于 `NavController` 是导航的核心，因此在设置 Compose Navigation 时必须先创建它。

`NavController` 是通过调用 [`rememberNavController()`](https://developer.android.com/reference/kotlin/androidx/navigation/compose/package-summary?hl=zh-cn#rememberNavController(kotlin.Array)) 函数获取的。这将创建并[记住](https://developer.android.com/jetpack/compose/state?hl=zh-cn#state-in-composables) `NavController`，它可以在配置更改后继续存在（使用 [`rememberSaveable`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/saveable/package-summary?hl=zh-cn#rememberSaveable(kotlin.Array,androidx.compose.runtime.saveable.Saver,kotlin.String,kotlin.Function0))）。

一定要创建 `NavController` 并将其放置在可组合项层次结构的顶层（通常位于 `App` 可组合项中）。之后，所有需要引用 `NavController` 的可组合项都可以访问它。这么做符合[状态提升](https://developer.android.com/jetpack/compose/state?hl=zh-cn#state-hoisting)的原则，并可确保 `NavController` 是在可组合屏幕之间导航和维护返回堆栈的主要可信来源。

打开 `RallyActivity.kt`。使用 `RallyApp` 内的 `rememberNavController()` 获取 `NavController`，因为它是整个应用的根可组合项和入口点：

```auto
import androidx.navigation.compose.rememberNavController
// ...

@Composable
fun RallyApp() {
    RallyTheme {
        var currentScreen: RallyDestination by remember { mutableStateOf(Overview) }
        val navController = rememberNavController()
        Scaffold(
            // ...
        ) {
            // ...
       }
}
```

## **Compose Navigation 中的路线**

如前所述，Rally 应用有三个主要目的地，以及一个日后要添加的额外目的地 (`SingleAccount`)。这些目的地在 `RallyDestinations.kt` 中定义。我们已经提到，每个目的地都有定义的 `icon`、`route` 和 `screen`：

| ![img](/images/compose_navigation/3_1.png) | ![img](/images/compose_navigation/3_2.png) | ![img](/images/compose_navigation/3_3.png) |
| ------------------------------------------ | ------------------------------------------ | ------------------------------------------ |

接下来，将这些目的地添加到导航图中，其中 `Overview` 作为应用启动时的起始目的地。

使用 Compose 中的 Navigation 时，导航图中的每个可组合目的地都与一个[**路线**](https://developer.android.com/jetpack/compose/navigation?hl=zh-cn#create-navhost)相关联。路线用字符串表示，用于定义指向可组合项的路径，并指引您的 `navController` 到达正确的位置。您可以将其视为指向特定目的地的隐式深层链接。**每个目的地都必须有一条唯一的路线**。

为此，我们会使用每个 `RallyDestination` 对象的 `route` 属性。例如，`Overview.route` 是将您转到 `Overview` 屏幕可组合项的路线。

## **使用导航图调用 NavHost 可组合项**

下一步是添加 [`NavHost`](https://developer.android.com/jetpack/compose/navigation?hl=zh-cn#create-navhost) 并创建导航图。

Navigation 的 3 个主要部分是 `NavController`、`NavGraph` 和 `NavHost`。`NavController` 始终与一个 `NavHost` 可组合项相关联。`NavHost` 充当容器，负责显示导航图的当前目的地。当您在可组合项之间进行导航时，`NavHost` 的内容会自动进行[重组](https://developer.android.com/jetpack/compose/mental-model?hl=zh-cn#recomposition)。此外，它还会将 `NavController` 与导航图 ([`NavGraph`](https://developer.android.com/reference/androidx/navigation/NavGraph?hl=zh-cn)) 相关联，后者用于标出能够在其间进行导航的可组合目的地。它实际上是一系列可提取的目的地。

返回到 `RallyActivity.kt` 中的 `RallyApp` 可组合项。将 `Scaffold` 内的 `Box` 可组合项（其中包含当前屏幕的内容，用于手动切换屏幕）替换为新的 `NavHost`（可按照以下代码示例进行创建）。

传入我们在上一步中创建的 `navController`，以将其挂接到这个 `NavHost`。如前所述，每个 `NavController` 都必须与一个 `NavHost` 相关联。

`NavHost` 还需要一个 `startDestination` 路线才能知道在应用启动时显示哪个目的地，因此请将其设置为 `Overview.route`。此外，传递 `Modifier` 以接受外部 `Scaffold` 内边距，然后将其应用于 `NavHost`。

最后一个形参 `builder: NavGraphBuilder.() -> Unit` 负责定义和构建导航图。该形参使用的是 [Navigation Kotlin DSL](https://developer.android.com/guide/navigation/navigation-kotlin-dsl?hl=zh-cn#navgraphbuilder) 中的 lambda 语法，因此可作为函数正文内部的尾随 lambda 传递并从括号中取出：

```auto
import androidx.navigation.compose.NavHost
...

Scaffold(...) { innerPadding ->
    NavHost(
        navController = navController,
        startDestination = Overview.route,
        modifier = Modifier.padding(innerPadding)
    ) {
       // builder parameter will be defined here as the graph
    }
}
```

## 向 Nav**Graph** 添加目的地

现在，您可以定义导航图以及 `NavController` 可导航到的目的地。如上所述，`builder` 形参要求使用函数，因此 Navigation Compose 提供了 [`NavGraphBuilder.composable`](https://developer.android.com/reference/kotlin/androidx/navigation/compose/package-summary?hl=zh-cn#(androidx.navigation.NavGraphBuilder).composable(kotlin.String,%20kotlin.collections.List,%20kotlin.collections.List,%20kotlin.Function1)) 扩展函数，以便轻松将各个可组合目的地添加到导航图中，并定义必要的导航信息。

第一个目的地是 `Overview`，因此您需要通过 `composable` 扩展函数添加它，并为其设置唯一字符串 `route`。此操作只会将目的地添加到导航图中，因此您还需要定义导航到此目的地时要显示的实际界面。此外，还可通过 `composable` 函数正文内部的尾随 lambda 完成此操作，这是 [Compose 中常用的模式](https://developer.android.com/jetpack/compose/kotlin?hl=zh-cn#trailing-lambdas)：

```auto
import androidx.navigation.compose.composable
// ...

NavHost(
    navController = navController,
    startDestination = Overview.route,
    modifier = Modifier.padding(innerPadding)
) {
    composable(route = Overview.route) {
        Overview.screen()
    }
}
```

我们会按照这种模式将所有三个主屏幕可组合项添加为三个目的地：

```auto
NavHost(
    navController = navController,
    startDestination = Overview.route,
    modifier = Modifier.padding(innerPadding)
) {
    composable(route = Overview.route) {
        Overview.screen()
    }
    composable(route = Accounts.route) {
        Accounts.screen()
    }
    composable(route = Bills.route) {
        Bills.screen()
    }
}
```

现在运行应用，您会看到 `Overview` 作为起始目的地，以及其对应的界面。

我们之前提到过自定义顶部标签页栏可组合项 `RallyTabRow`，该可组合项之前处理了屏幕之间的手动导航。此时，它尚未与新导航相关联，因此您可以验证一下点击标签页是否会更改所显示屏幕可组合项的目的地。接下来，我们来解决这个问题！

## 5\. 将 RallyTabRow 与导航集成

在此步骤中，您需要将 `RallyTabRow` 与 `navController` 和导航图连接起来，以便其导航到正确的目的地。

为此，您需要使用新的 `navController`，为 `RallyTabRow` 的 `onTabSelected` 回调定义正确的导航操作。此回调定义了选择特定标签页图标时应发生的情况，并通过 [`navController.navigate(route)`](https://developer.android.com/jetpack/compose/navigation?hl=zh-cn#nav-to-composable) 执行导航操作`.`

遵循此指南，在 `RallyActivity` 中找到 `RallyTabRow` 可组合项及其回调形参 `onTabSelected`。

由于我们希望点按标签页后导航到特定目的地，因此您还需要知道所选择的确切标签页图标。幸运的是，`onTabSelected: (RallyDestination) -> Unit` 形参已提供这些信息。您将使用这些信息和 `RallyDestination` 路线来指引 `navController` 并在用户选择标签页时调用 `navController.navigate(newScreen.route)`：

```auto
@Composable
fun RallyApp() {
    RallyTheme {
        var currentScreen: RallyDestination by remember { mutableStateOf(Overview) }
        val navController = rememberNavController()
        Scaffold(
            topBar = {
                RallyTabRow(
                    allScreens = rallyTabRowScreens,
                    // Pass the callback like this,
                    // defining the navigation action when a tab is selected:
                    onTabSelected = { newScreen ->
                        navController.navigate(newScreen.route)
                    },
                    currentScreen = currentScreen,
                )
            }
```

如果现在运行应用，可以验证点按 `RallyTabRow` 中的各个标签页是否确实会导航到正确的可组合目的地。不过，您目前可能已经注意到以下两个问题：

1.  重按同一标签页会启动同一目的地的多个副本
2.  标签页的界面与所显示的正确目的地不符，也就是说，无法正常展开和收起所选标签页：

| ![img](/images/compose_navigation/4_1.png) | ![img](/images/compose_navigation/4_2.png) |
| ------------------------------------------ | ------------------------------------------ |

下面我们来解决这两个问题！

## **启动目的地的单个副本**

为了解决第一个问题，并确保返回堆栈顶部最多只有给定目的地的一个副本，Compose Navigation API 提供了一个 [`launchSingleTop`](https://developer.android.com/reference/kotlin/androidx/navigation/NavOptionsBuilder?hl=zh-cn#launchSingleTop()) 标志，您可以将其传递给 `navController.navigate()` 操作，如下所示：

`navController.navigate(route) { launchSingleTop = true }`

由于您希望在整个应用中实现这种行为，因此对于每个目的地，不要将此标志复制粘贴到所有的 .`navigate(...)` 调用中，而是将其提取到 `RallyActivity` 底部的辅助扩展程序中。

```auto
import androidx.navigation.NavHostController
// ...

fun NavHostController.navigateSingleTopTo(route: String) =
    this.navigate(route) { launchSingleTop = true }
```

现在，您可以将 `navController.navigate(newScreen.route)` 调用替换为 .`navigateSingleTopTo(...)`。重新运行应用，然后验证一下：在顶栏中多次点击其图标时，现在是否只会获得单个目的地的一个副本：

```auto
@Composable
fun RallyApp() {
    RallyTheme {
        var currentScreen: RallyDestination by remember { mutableStateOf(Overview) }
        val navController = rememberNavController()
        Scaffold(
            topBar = {
                RallyTabRow(
                    allScreens = rallyTabRowScreens,
                    onTabSelected = { newScreen ->
                        navController
                            .navigateSingleTopTo(newScreen.route)
                    },
                    currentScreen = currentScreen,
                )
            }
```

## **控制导航选项和返回堆栈状态**

除了 `launchSingleTop` 之外，您还可以使用 [`NavOptionsBuilder`](https://developer.android.com/reference/kotlin/androidx/navigation/NavOptionsBuilder?hl=zh-cn) 中的其他标志进一步控制和自定义导航行为。由于 `RallyTabRow` 的作用类似于 [`BottomNavigation`](https://developer.android.com/reference/kotlin/androidx/compose/material/package-summary?hl=zh-cn#BottomNavigation(androidx.compose.ui.Modifier,androidx.compose.ui.graphics.Color,androidx.compose.ui.graphics.Color,androidx.compose.ui.unit.Dp,kotlin.Function1))，因此您还应想一想，在导航到和离开目的地时是否要保存并恢复目的地状态。例如，如果要滚动到“Overview”屏幕底部，然后导航到“Accounts”屏幕，接着返回，那么您是否要保持滚动位置？是否要重按 `RallyTabRow` 中的同一目的地，以重新加载屏幕状态？这些都是有效问题，应根据您自己的应用设计要求来确定。

我们将介绍同一 `navigateSingleTopTo` 扩展函数中可供您使用的一些其他选项：

+   `launchSingleTop = true` - 如上所述，这可确保返回堆栈顶部最多只有给定目的地的一个副本
+   在 Rally 应用中，这意味着，多次重按同一标签页不会启动同一目的地的多个副本
+   `popUpTo(startDestination) { saveState = true }` - 弹出到导航图的起始目的地，以免在您选择标签页时在返回堆栈上积累大量目的地
+   在 Rally 中，这意味着，在任何目的地按下返回箭头都会将整个返回堆栈弹出到“Overview”屏幕
+   `restoreState = true` - 确定此导航操作是否应恢复 `PopUpToBuilder.saveState` 或 `popUpToSaveState` 属性之前保存的任何状态。请注意，如果之前未使用要导航到的目的地 ID 保存任何状态，**此项不会产生任何影响**
+   在 Rally 中，这意味着，重按同一标签页会保留屏幕上之前的数据和用户状态，而无需重新加载

您可以将所有这些选项逐一添加到代码中，然后在添加每个选项后运行应用，并在添加每个标志后验证确切行为。这样一来，您就可以在实践中了解每个标志是如何更改导航和返回堆栈状态的：

```auto
import androidx.navigation.NavHostController
import androidx.navigation.NavGraph.Companion.findStartDestination
// ...

fun NavHostController.navigateSingleTopTo(route: String) =
    this.navigate(route) {
        popUpTo(
            this@navigateSingleTopTo.graph.findStartDestination().id
        ) {
            saveState = true
        }
        launchSingleTop = true
        restoreState = true
}
```

## **解决标签页界面问题**

在此 Codelab 开始时，`RallyTabRow` 虽然仍在使用手动导航机制，但会使用 `currentScreen` 变量来确定是展开还是收起各个标签页。

不过，在您完成更改后，`currentScreen` 将不再更新。因此，`RallyTabRow` 内展开和收起所选标签页的操作不再起作用。

如需使用 Compose Navigation 重新启用此行为，您需要知道在每个时间点显示的当前目的地是什么（或者用导航术语来说，当前返回堆栈条目的顶部是什么），然后在此行为每次更改时更新 `RallyTabRow`。

如需以 [`State`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/State?hl=zh-cn) 的形式获取返回堆栈中当前目的地的实时更新，您可以使用 [`navController.currentBackStackEntryAsState()`](https://developer.android.com/reference/kotlin/androidx/navigation/compose/package-summary?hl=zh-cn#(androidx.navigation.NavController).currentBackStackEntryAsState())，然后获取其当前 `destination:`

```auto
import androidx.navigation.compose.currentBackStackEntryAsState
import androidx.compose.runtime.getValue
// ...

@Composable
fun RallyApp() {
    RallyTheme {
        val navController = rememberNavController()

        val currentBackStack by navController.currentBackStackEntryAsState()
        // Fetch your currentDestination:
        val currentDestination = currentBackStack?.destination
        // ...
    }
}
```

`currentBackStack?.destination` 会返回 [`NavDestination`](https://developer.android.com/reference/kotlin/androidx/navigation/NavDestination?hl=zh-cn)`.`如需重新正确更新 `currentScreen`，您需要想方设法将返回值 [`NavDestination`](https://developer.android.com/reference/kotlin/androidx/navigation/NavDestination?hl=zh-cn) 与 Rally 的三个主要屏幕可组合项之一进行匹配。您必须确定当前显示的目的地，以便随后将这些信息传递给 `RallyTabRow.`如前所述，每个目的地都有一条唯一的路线，因此我们可以使用此 String 路线作为类似的 ID，以进行严格的对比并找到唯一匹配。

如需更新 `currentScreen`，您需要迭代 `rallyTabRowScreens` 列表，以找到匹配路线，然后返回对应的 `RallyDestination`。Kotlin 为此提供了一个便捷的 [`.find()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/find.html) 函数：

```auto
import androidx.navigation.compose.currentBackStackEntryAsState
import androidx.compose.runtime.getValue
// ...

@Composable
fun RallyApp() {
    RallyTheme {
        val navController = rememberNavController()

        val currentBackStack by navController.currentBackStackEntryAsState()
        val currentDestination = currentBackStack?.destination

        // Change the variable to this and use Overview as a backup screen if this returns null
        val currentScreen = rallyTabRowScreens.find { it.route == currentDestination?.route } ?: Overview
        // ...
    }
}
```

由于已将 `currentScreen` 传递给 `RallyTabRow`，因此您可以运行应用，并验证标签页栏界面现在是否已相应更新。

## 6\. 从 RallyDestination 中提取屏幕可组合项

到目前为止，为简单起见，我们一直使用 `RallyDestination` 接口中的 `screen` 属性以及通过该属性扩展的屏幕对象，将可组合界面添加到 `NavHost (RallyActivity.kt`) 中：

```auto
import com.example.compose.rally.ui.overview.OverviewScreen
// ...

NavHost(
    navController = navController,
    startDestination = Overview.route,
    modifier = Modifier.padding(innerPadding)
) {
    composable(route = Overview.route) {
        Overview.screen()
    }
    // ...
}
```

不过，此 Codelab 中的后续步骤（例如点击事件）需要将额外信息直接传递给可组合屏幕。在生产环境中，肯定需要传递更多数据。

若要实现这一目标，正确且更简洁的方式是将可组合项直接添加到 `NavHost` 导航图中，然后从 `RallyDestination` 中提取。之后，`RallyDestination` 和屏幕对象将仅保存导航专属信息（例如 `icon` 和 `route`），还将与任何与 Compose 界面相关的信息分离。

打开 `RallyDestinations.kt`。将每个屏幕的可组合项从 `RallyDestination` 对象的 `screen` 形参提取到 `NavHost` 中对应的 `composable` 函数中，以替换之前的 `.screen()` 调用，如下所示`:`

```auto
import com.example.compose.rally.ui.accounts.AccountsScreen
import com.example.compose.rally.ui.bills.BillsScreen
import com.example.compose.rally.ui.overview.OverviewScreen
// ...

NavHost(
    navController = navController,
    startDestination = Overview.route,
    modifier = Modifier.padding(innerPadding)
) {
    composable(route = Overview.route) {
        OverviewScreen()
    }
    composable(route = Accounts.route) {
        AccountsScreen()
    }
    composable(route = Bills.route) {
        BillsScreen()
    }
}
```

此时，您可以从 `RallyDestination` 及其对象中安全移除 `screen` 形参：

```auto
interface RallyDestination {
    val icon: ImageVector
    val route: String
}

/**
 * Rally app navigation destinations
 */
object Overview : RallyDestination {
    override val icon = Icons.Filled.PieChart
    override val route = "overview"
}
// ...
```

再次运行应用，并验证一切是否像先前一样正常运行。现在，您已经完成此步骤，接下来可以在可组合屏幕中设置点击事件了。

## **对 OverviewScreen 启用点击**

目前，`OverviewScreen` 中的所有点击事件都已被忽略。也就是说，“Accounts”和“Bills”的子部分“SEE ALL”按钮可供点击，但实际上不会转到任何位置。此步骤的目的是为这些点击事件启用导航。

![“Overview”屏幕的录屏内容：滚动到最终点击目的地并尝试点击。点击操作尚未实现，因此无法正常运行。](/images/compose_navigation/5.gif)

`OverviewScreen` 可组合项可以接受多个函数作为回调，以设置为点击事件，在本示例中，这应该是可让您转到 `AccountsScreen` 或 `BillsScreen` 的导航操作。我们来将这些导航回调传递给 `onClickSeeAllAccounts` 和 `onClickSeeAllBills`，以导航到相关目的地。

打开 `RallyActivity.kt`，在 `NavHost` 中找到 `OverviewScreen`，然后将 `navController.navigateSingleTopTo(...)` 传递给具有相应路线的两个导航回调：

```auto
OverviewScreen(
    onClickSeeAllAccounts = {
        navController.navigateSingleTopTo(Accounts.route)
    },
    onClickSeeAllBills = {
        navController.navigateSingleTopTo(Bills.route)
    }
)
```

`navController` 现在将获得足够的信息（例如确切目的地的路线）`,`可在用户点击按钮时导航到正确的目的地。如果您查看 `OverviewScreen` 的实现，便会发现这些回调已设置为相应的 `onClick` 形参`:`

```auto
@Composable
fun OverviewScreen(...) {
    // ...
    AccountsCard(
        onClickSeeAll = onClickSeeAllAccounts,
        onAccountClick = onAccountClick
    )
    // ...
    BillsCard(
        onClickSeeAll = onClickSeeAllBills
    )
}
```

如前所述，若能将 `navController` 保持在导航层次结构的顶层并将其提升到 `App` 可组合项级别（例如，不将其直接传递给 `OverviewScreen)`，就可以轻松地单独预览、重复使用和测试 `OverviewScreen` 可组合项，而不必依赖于实际或模拟的 `navController` 实例。此外，改为传递回调还可让您快速更改点击事件！

## 7\. 使用实参导航到 SingleAccountScreen

让我们为 `Accounts` 和 `Overview` 屏幕添加一些新功能！目前，这些屏幕会显示一个包含多种不同类型账户（“Checking”“Home Savings”等）的列表。

| ![img](/images/compose_navigation/6_1.png) | ![img](/images/compose_navigation/6_2.png) |
| ------------------------------------------ | ------------------------------------------ |



不过，点击这些账户类型（目前）不会起任何作用。接下来，让我们解决这个问题！在点按每个账户类型后，我们希望看到一个包含完整账户详细信息的新屏幕。为此，我们需要向 `navController` 提供与所点击的确切账户类型有关的其他信息。这可以通过[实参](https://developer.android.com/jetpack/compose/navigation?hl=zh-cn#nav-with-args)实现。

实参是一类非常强大的工具，它们会将一个或多个实参传递给路线，从而使导航路线变为动态形式。它支持根据所提供的不同实参显示不同的信息。

在 `RallyApp` 中，通过向现有 `NavHost:` 添加新的 `composable` 函数，将新的目的地 `SingleAccountScreen`（用于处理这些具体账户的显示操作）添加到导航图中：

```auto
import com.example.compose.rally.ui.accounts.SingleAccountScreen
// ...

NavHost(
    navController = navController,
    startDestination = Overview.route,
    modifier = Modifier.padding(innerPadding)
) {
    ...
    composable(route = SingleAccount.route) {
        SingleAccountScreen()
    }
}
```

## **设置 SingleAccountScreen 到达目的地**

当您到达 `SingleAccountScreen` 时，此目的地将需要更多信息才能知道打开时应显示的确切账户类型。我们可以使用实参传递此类信息。您需要指明，其路线还需要一个实参 `{account_type}`。如果您查看 `RallyDestination` 及其 `SingleAccount` 对象，便会发现该实参已定义为 `accountTypeArg` String，供您使用。

如需在导航时随路线一起传递实参，您需要按照以下模式将它们附加在一起：`"route/{argument}"`。在本示例中，应如下所示：`"${SingleAccount.route}/{${SingleAccount.accountTypeArg}}"`。请注意，$ 符号用于转义变量：

```auto
import androidx.navigation.NavType
import androidx.navigation.compose.navArgument
// ...

composable(
    route =
        "${SingleAccount.route}/{${SingleAccount.accountTypeArg}}"
) {
    SingleAccountScreen()
}
```

这样可确保当操作被触发以导航到 `SingleAccountScreen` 时，还必须传递 `accountTypeArg` 实参，否则导航将会失败。您可以将其视为其他要导航到 `SingleAccountScreen` 的目的地需要遵循的签名或协定。

第二步是让此 `composable` 知道它应该接受实参。为此，您可以定义其 `arguments` 形参。您可以根据需要定义任意数量的实参，因为 `composable` 函数默认接受实参列表。在本示例中，您只需添加一个名为 `accountTypeArg` 的实参，并将其类型指定为 `String`，即可提高安全性。如果您未明确设置类型，系统将根据此实参的默认值推断出其类型：

```auto
import androidx.navigation.NavType
import androidx.navigation.compose.navArgument
// ...

composable(
    route =
        "${SingleAccount.route}/{${SingleAccount.accountTypeArg}}",
    arguments = listOf(
        navArgument(SingleAccount.accountTypeArg) { type = NavType.StringType }
    )
) {
    SingleAccountScreen()
}
```

这样就可以做到完美运行，您可以选择保留代码，如下所示。不过，由于我们的所有目的地专属信息都位于 `RallyDestinations.kt` 及其对象中，因此我们会继续采用相同的方法（就像我们在前面为 `Overview`、`Accounts,` 和 `Bills` 采取的方法一样）并将此实参列表迁移到 `SingleAccount:`

```auto
object SingleAccount : RallyDestination {
    // ...
    override val route = "single_account"
    const val accountTypeArg = "account_type"
    val arguments = listOf(
        navArgument(accountTypeArg) { type = NavType.StringType }
    )
}
```

将之前的实参替换为 `SingleAccount.arguments`，现在返回到相应的 NavHost `composable`。这还可以确保让 `NavHost` 尽可能简洁且易读：

```auto
composable(
    route = "${SingleAccount.route}/{${SingleAccount.accountTypeArg}}",
    arguments =  SingleAccount.arguments
) {
    SingleAccountScreen()
}
```

现在，您已使用实参定义了 `SingleAccountScreen` 的完整路线，接下来要确保将此 `accountTypeArg` 进一步向下传递给 `SingleAccountScreen` 可组合项，这样它就会知道要正确显示哪个账户类型。如果您查看 `SingleAccountScreen` 的实现，就会发现它已设置完毕且正在等待接受 `accountType` 形参：

```auto
fun SingleAccountScreen(
    accountType: String? = UserData.accounts.first().name
) {
   // ...
}
```

简而言之，到目前为止：

+   您已确保我们定义请求实参的路线，作为先前目的地的信号
+   您已确保 `composable` 知道它需要接受实参

最后一步是以某种方式真正**检索传递的实参**值。

在 Compose Navigation 中，每个 `NavHost` 可组合函数都可以访问当前的 [`NavBackStackEntry`](https://developer.android.com/reference/kotlin/androidx/navigation/NavBackStackEntry?hl=zh-cn)，该类用于保存当前路线的相关信息，以及返回堆栈中条目的已传递实参。您可以使用该类从 `navBackStackEntry` 中获取所需的 `arguments` 列表，然后搜索并检索所需的确切实参，将其进一步向下传递给可组合屏幕。

在这种情况下，您需要从 `navBackStackEntry` 请求 `accountTypeArg`。然后，您需要将其进一步向下传递给 `SingleAccountScreen'` 的 `accountType` 形参。

您也可以为实参提供一个默认值（如果尚未提供）作为占位符，并包含这种极端情况，以提高代码的安全性。

现在，您的代码应如下所示：

```auto
NavHost(...) {
    // ...
    composable(
        route =
          "${SingleAccount.route}/{${SingleAccount.accountTypeArg}}",
        arguments = SingleAccount.arguments
    ) { navBackStackEntry ->
        // Retrieve the passed argument
        val accountType =
            navBackStackEntry.arguments?.getString(SingleAccount.accountTypeArg)

        // Pass accountType to SingleAccountScreen
        SingleAccountScreen(accountType)
    }
}
```

您的 `SingleAccountScreen` 现已获得在您导航到那里时显示正确账户类型所需的必要信息。如果您查看 `SingleAccountScreen,` 的实现，就会发现它已经将传递的 `accountType` 与 `UserData` 源进行匹配，以获取相应的账户详细信息。

我们再来执行一项次要优化任务，将 `"${SingleAccount.route}/{${SingleAccount.accountTypeArg}}"` 路线也移至 `RallyDestinations.kt` 及其 `SingleAccount` 对象中`:`

```auto
object SingleAccount : RallyDestination {
    // ...
    override val route = "single_account"
    const val accountTypeArg = "account_type"
    val routeWithArgs = "${route}/{${accountTypeArg}}"
    val arguments = listOf(
        navArgument(accountTypeArg) { type = NavType.StringType }
    )
}
```

同样，在相应的 `NavHost composable:` 中进行替换

```auto
// ...
composable(
    route = SingleAccount.routeWithArgs,
    arguments = SingleAccount.arguments
) {...}
```

## **设置“Accounts”和“Overview”的起始目的地**

现在，您已经定义了 `SingleAccountScreen` 路线及其成功导航到 `SingleAccountScreen` 所需要和接受的实参，接下来您需要确保是从上一个目的地（即您来自的目的地）传递同一 `accountTypeArg` 实参。

如您所见，这包含两个方面：一个是提供并传递实参的起始目的地，另一个是接受该实参并使用它显示正确信息的到达目的地。**这两者都需要进行明确定义**。

例如，当您位于 `Accounts` 目的地，并点按“Checking”账户类型时，“Accounts”目的地需要将“Checking”String（已附加到“single\_account”String 路线）作为实参传递，以便成功打开相应的 `SingleAccountScreen`。其 String 路线将如下所示：`"single_account/Checking"`

使用 `navController.navigateSingleTopTo(...),` 时，您需要使用与其完全相同的路线，并包含传递的实参，如下所示：

`navController.navigateSingleTopTo("${SingleAccount.route}/$accountType")`。

将此导航操作回调传递给 `OverviewScreen` 和 `AccountsScreen` 的 `onAccountClick` 形参。请注意，这些形参已预定义为 `onAccountClick: (String) -> Unit`，其中 String 作为输入。也就是说，当用户点按 `Overview` 和 `Account` 中的特定账户类型时，该账户类型 String 已经可供您使用，并且可以作为导航实参轻松传递：

```auto
OverviewScreen(
    // ...
    onAccountClick = { accountType ->
        navController
          .navigateSingleTopTo("${SingleAccount.route}/$accountType")
    }
)
// ...

AccountsScreen(
    // ...
    onAccountClick = { accountType ->
        navController
          .navigateSingleTopTo("${SingleAccount.route}/$accountType")
    }
)
```

为确保内容易于阅读，您可以将此导航操作提取到私有辅助扩展函数中：

```auto
import androidx.navigation.NavHostController
// ...
OverviewScreen(
    // ...
    onAccountClick = { accountType ->
        navController.navigateToSingleAccount(accountType)
    }
)

// ...

AccountsScreen(
    // ...
    onAccountClick = { accountType ->
        navController.navigateToSingleAccount(accountType)
    }
)

// ...

private fun NavHostController.navigateToSingleAccount(accountType: String) {
    this.navigateSingleTopTo("${SingleAccount.route}/$accountType")
}
```

此时，当您运行应用时，可以点击每个账户类型，系统会将您转到显示给定账户数据的对应 `SingleAccountScreen`。

![“Overview”屏幕的录屏内容：滚动到最终点击目的地并尝试点击。现在，点击会将用户引导至目的地。](/images/compose_navigation/7.gif)

## 8\. 启用深层链接支持

除了添加实参之外，您还可以添加[深层链接](https://developer.android.com/jetpack/compose/navigation?hl=zh-cn#deeplinks)，将特定网址、操作和/或 MIME 类型与可组合项关联起来。在 Android 中，深层链接是指将用户直接转到应用内特定目的地的链接。Navigation Compose 支持[隐式深层链接](https://developer.android.com/guide/navigation/navigation-deep-link?hl=zh-cn#implicit)。调用隐式深层链接（例如，当用户点击某个链接）时，Android 可以将应用打开到相应的目的地。

在此部分中，您将添加一个新的深层链接，用于导航到包含相应账户类型的 `SingleAccountScreen` 可组合项，还要让此深层链接向外部应用公开。我们来回顾一下，此可组合项的路线是 `"single_account/{account_type}"`，这也是您将用于深层链接的路线，其中包含一些与深层链接相关的细微更改。

默认情况下，系统不会启用向外部应用公开深层链接这一功能，因此您还必须向应用的 `manifest.xml` 文件添加 `<intent-filter>` 元素，这是第一步。

首先，向应用的 `AndroidManifest.xml` 添加深层链接。您需要通过 `<activity>` 内的 `<intent-filter>` 创建一个新的 intent 过滤器，相应操作为 `VIEW`，类别为 `BROWSABLE` 和 `DEFAULT`。

然后，在该过滤器内，您需要使用 `data` 标记添加 **`scheme`**（`rally` - 应用名称）和 **`host`**（`single_account` - 导航到可组合项的路线），以定义精确的深层链接。这将为您提供 `rally://single_account` 作为深层链接网址。

请注意，您无需在 `AndroidManifest` 中声明 `account_type` 实参。该实参稍后会附加到 `NavHost` 可组合函数内。

```auto
<activity
    android:name=".RallyActivity"
    android:windowSoftInputMode="adjustResize"
    android:label="@string/app_name"
    android:exported="true">
    <intent-filter>
        <action android:name="android.intent.action.MAIN" />
        <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>
    <intent-filter>
        <action android:name="android.intent.action.VIEW" />
        <category android:name="android.intent.category.DEFAULT" />
        <category android:name="android.intent.category.BROWSABLE" />
        <data android:scheme="rally" android:host="single_account" />
    </intent-filter>
</activity>
```

## 触发并验证深层链接

现在，您可以在 `RallyActivity` 中响应传入的 intent。

可组合项 `SingleAccountScreen` 已经接受实参，但现在还需要接受新创建的深层链接，才能在其深层链接触发时启动此目的地。

在 `SingleAccountScreen` 的可组合函数内，再添加一个形参 `deepLinks`。与 `arguments,` 类似，它还接受 `navDeepLink` 列表，因为您可以定义多个指向同一目的地的深层链接。传递 `uriPattern`，匹配清单 `rally://singleaccount` 的 `intent-filter` 中定义的一个 uriPattern，但这次您还需附加其 `accountTypeArg` 实参：

```auto
import androidx.navigation.navDeepLink
// ...

composable(
    route = SingleAccount.routeWithArgs,
    // ...
    deepLinks = listOf(navDeepLink {
        uriPattern = "rally://${SingleAccount.route}/{${SingleAccount.accountTypeArg}}"
    })
)
```

您知道接下来该怎么做，对吧？将此列表移至 `RallyDestinations SingleAccount:`

```auto
object SingleAccount : RallyDestination {
    // ...
    val arguments = listOf(
        navArgument(accountTypeArg) { type = NavType.StringType }
    )
    val deepLinks = listOf(
       navDeepLink { uriPattern = "rally://$route/{$accountTypeArg}"}
    )
}
```

同样，在相应的 `NavHost` 可组合项中进行替换：

```auto
// ...
composable(
    route = SingleAccount.routeWithArgs,
    arguments = SingleAccount.arguments,
    deepLinks = SingleAccount.deepLinks
) {...}
```

## 使用 adb 测试深层链接

现在，您的应用和 `SingleAccountScreen` 已准备好处理深层链接。为了测试它能否正常运行，请在已连接的模拟器或设备上重新安装 Rally，打开命令行并执行以下命令，以便模拟深层链接启动：

```auto
adb shell am start -d "rally://single_account/Checking" -a android.intent.action.VIEW
```

系统会带您直接进入“Checking”账户，不过您也可以针对所有其他账户类型验证它能否正常运行。

## 9\. 将 NavHost 提取到 RallyNavHost

您的 `NavHost` 现已完成。不过，为了使其可测试并保持 `RallyActivity` 更加简洁，您可以将当前 `NavHost` 及其辅助函数（如 `navigateToSingleAccount`）从 `RallyApp` 可组合项提取到它自己的可组合函数，并将其命名为 `RallyNavHost`。

`RallyApp` 是应使用 `navController` 直接处理的唯一一个可组合项。如前所述，其他每个嵌套的可组合屏幕应该仅获得导航回调，而不是 `navController` 本身。

因此，新的 `RallyNavHost` 将接受 `RallyApp` 中的 `navController` 和 `modifier` 作为形参：

```auto
@Composable
fun RallyNavHost(
    navController: NavHostController,
    modifier: Modifier = Modifier
) {
    NavHost(
        navController = navController,
        startDestination = Overview.route,
        modifier = modifier
    ) {
        composable(route = Overview.route) {
            OverviewScreen(
                onClickSeeAllAccounts = {
                    navController.navigateSingleTopTo(Accounts.route)
                },
                onClickSeeAllBills = {
                    navController.navigateSingleTopTo(Bills.route)
                },
                onAccountClick = { accountType ->
                   navController.navigateToSingleAccount(accountType)
                }
            )
        }
        composable(route = Accounts.route) {
            AccountsScreen(
                onAccountClick = { accountType ->
                   navController.navigateToSingleAccount(accountType)
                }
            )
        }
        composable(route = Bills.route) {
            BillsScreen()
        }
        composable(
            route = SingleAccount.routeWithArgs,
            arguments = SingleAccount.arguments,
            deepLinks = SingleAccount.deepLinks
        ) { navBackStackEntry ->
            val accountType =
              navBackStackEntry.arguments?.getString(SingleAccount.accountTypeArg)
            SingleAccountScreen(accountType)
        }
    }
}

fun NavHostController.navigateSingleTopTo(route: String) =
    this.navigate(route) { launchSingleTop = true }

private fun NavHostController.navigateToSingleAccount(accountType: String) {
    this.navigateSingleTopTo("${SingleAccount.route}/$accountType")
}
```

现在，将新的 `RallyNavHost` 添加到 `RallyApp`，然后重新运行应用，验证一切是否像先前一样正常运行：

```auto
fun RallyApp() {
    RallyTheme {
    ...
        Scaffold(
        ...
        ) { innerPadding ->
            RallyNavHost(
                navController = navController,
                modifier = Modifier.padding(innerPadding)
            )
        }
     }
}
```

## 10\. 测试 Compose Navigation

从此 Codelab 的开头开始，您就确保不将 `navController` 直接传入任何可组合项（高级应用除外），而是将导航回调作为形参传递。这样一来，您的所有可组合项均可[单独测试](https://developer.android.com/jetpack/compose/navigation?hl=zh-cn#testing)，因为它们不需要在测试中使用 `navController` 实例。

一定要测试整个 Compose Navigation 机制能否在应用中按预期运行，方法是测试传递给可组合项的 `RallyNavHost` 和导航操作。这些将是此部分的主要目标。如需单独测试各个可组合函数，请务必查看[在 Jetpack Compose 中进行测试](https://developer.android.com/codelabs/jetpack-compose-testing?hl=zh-cn) Codelab。

若要开始测试，我们首先需要添加必要的测试依赖项，因此请返回到应用的 build 文件（位于 `app/build.gradle`）。在测试依赖项部分中，添加 `navigation-testing` 依赖项：

```auto
dependencies {
// ...
  androidTestImplementation "androidx.navigation:navigation-testing:$rootProject.composeNavigationVersion"
  // ...
}
```

## 准备 NavigationTest 类

您的 `RallyNavHost` 可独立于 `Activity` 本身进行测试。

由于此测试仍会在 Android 设备上运行，因此您需要创建测试目录 `/app/src/androidTest/java/com/example/compose/rally`，然后创建一个新的测试文件来测试类并将其命名为 `NavigationTest`。

首先，若要使用 Compose 测试 API，以及使用 Compose 进行测试并控制可组合项和应用，请添加 [Compose 测试规则](https://developer.android.com/reference/kotlin/androidx/compose/ui/test/junit4/ComposeTestRule?hl=zh-cn)：

```auto
import androidx.compose.ui.test.junit4.createComposeRule
import org.junit.Rule

class NavigationTest {

    @get:Rule
    val composeTestRule = createComposeRule()

}
```

## **编写首个测试**

创建一个公共 `rallyNavHost` 测试函数，并为其添加 `@Test` 注解。在该函数中，首先需要设置要测试的 Compose 内容。您可以使用 `composeTestRule` 的 `setContent` 执行此操作。它接受一个可组合形参作为正文，并且支持您编写 Compose 代码，以及在测试环境中添加可组合项，就像您在常规生产环境应用中一样。

在 `setContent,` 内，您可以设置当前测试对象 `RallyNavHost`，并将新的 `navController` 实例传递给该对象。Navigation Testing 工件提供了一个便捷的 [`TestNavHostController`](https://developer.android.com/reference/kotlin/androidx/navigation/testing/TestNavHostController?hl=zh-cn) 供您使用。接下来，我们来添加此步骤：

```auto
import androidx.compose.ui.platform.LocalContext
import androidx.navigation.compose.ComposeNavigator
import androidx.navigation.testing.TestNavHostController
import org.junit.Assert.fail
import org.junit.Test
// ...

class NavigationTest {

    @get:Rule
    val composeTestRule = createComposeRule()

    lateinit var navController: TestNavHostController

    @Test
    fun rallyNavHost() {
        composeTestRule.setContent {
            // Creates a TestNavHostController
            navController =
                TestNavHostController(LocalContext.current)
            // Sets a ComposeNavigator to the navController so it can navigate through composables
            navController.navigatorProvider.addNavigator(
                ComposeNavigator()
            )
            RallyNavHost(navController = navController)
        }
        fail()
    }
}
```

如果您复制了上述代码，`fail()` 调用会确保您的测试一直失败，直到出现实际断言为止。它用于提醒您完成测试的实现。

如需验证是否显示了正确的屏幕可组合项，您可以使用其 `contentDescription` 并断言它会显示。在此 Codelab 中，“Accounts”和“Overview”目的地的 `contentDescription` 之前已设置完毕，因此您已经可以使用它们进行测试验证。

首次验证时，您应该检查“Overview”屏幕在 `RallyNavHost` 首次初始化时是否会作为第一个目的地显示。您还应重命名测试来反映这一点，不妨将其命名为 `rallyNavHost_verifyOverviewStartDestination`。为此，请将 `fail()` 调用替换为以下代码：

```auto
import androidx.compose.ui.test.assertIsDisplayed
import androidx.compose.ui.test.onNodeWithContentDescription
// ...

class NavigationTest {

    @get:Rule
    val composeTestRule = createComposeRule()

    lateinit var navController: TestNavHostController

    @Test
    fun rallyNavHost_verifyOverviewStartDestination() {
        composeTestRule.setContent {
            navController =
                TestNavHostController(LocalContext.current)
            navController.navigatorProvider.addNavigator(
                ComposeNavigator()
            )
            RallyNavHost(navController = navController)
        }

        composeTestRule
            .onNodeWithContentDescription("Overview Screen")
            .assertIsDisplayed()
    }
}
```

再次运行测试，并验证测试是否会通过。

由于您需要以相同的方式为即将到来的每项测试设置 `RallyNavHost`，因此您可以将其初始化部分提取到带注解的 `@Before` 函数中，以避免多余的重复代码并确保测试更加简洁：

```auto
import org.junit.Before
// ...

class NavigationTest {

    @get:Rule
    val composeTestRule = createComposeRule()
    lateinit var navController: TestNavHostController

    @Before
    fun setupRallyNavHost() {
        composeTestRule.setContent {
            navController =
                TestNavHostController(LocalContext.current)
            navController.navigatorProvider.addNavigator(
                ComposeNavigator()
            )
            RallyNavHost(navController = navController)
        }
    }

    @Test
    fun rallyNavHost_verifyOverviewStartDestination() {
        composeTestRule
            .onNodeWithContentDescription("Overview Screen")
            .assertIsDisplayed()
    }
}
```

## 在测试中导航

您可以通过多种方式测试导航实现，具体方法是：点击界面元素，然后验证显示的目的地；或者比较预期路线与当前路线。

### **通过界面点击和屏幕 contentDescription 进行测试**

如果您要测试具体应用的实现，最好在界面中执行点击操作。接下来的文本内容可以验证这一点：在“Overview”屏幕中，点击“Accounts”子部分中的“SEE ALL”按钮后，您便会转到“Accounts”目的地：

![5a9e82acf7efdd5b.png](/images/compose_navigation/8.png)

您将再次在 `OverviewScreenCard` 可组合项中使用为此特定按钮设置的 `contentDescription`，从而通过 `performClick()` 模拟对该按钮的点击，并验证“Accounts”目的地随后是否会显示：

```auto
import androidx.compose.ui.test.performClick
// ...

@Test
fun rallyNavHost_clickAllAccount_navigatesToAccounts() {
    composeTestRule
        .onNodeWithContentDescription("All Accounts")
        .performClick()

    composeTestRule
        .onNodeWithContentDescription("Accounts Screen")
        .assertIsDisplayed()
}
```

您可以按照此模式在应用中测试其余所有的点击导航操作。

### 通过界面点击和路线对比进行测试

您还可以使用 `navController` 检查您的断言，只需将当前 String 路线与预期路线进行比较即可。为此，请按照与上一部分中相同的步骤，在界面中执行点击操作，然后使用 `navController.currentBackStackEntry?.destination?.route` 来比较当前路线与预期路线。

您还需要再执行一个步骤，即确保先在“Overview”屏幕上滚动到“Bills”子部分，否则测试将会失败，因为它无法找到具有 `contentDescription`“All Bills”的节点：

```auto
import androidx.compose.ui.test.performScrollTo
import org.junit.Assert.assertEquals
// ...

@Test
fun rallyNavHost_clickAllBills_navigateToBills() {
    composeTestRule.onNodeWithContentDescription("All Bills")
        .performScrollTo()
        .performClick()

    val route = navController.currentBackStackEntry?.destination?.route
    assertEquals(route, "bills")
}
```

您可以按照这些模式涵盖任何其他导航路线、目的地和点击操作，以便完成测试类。立即运行整套测试，验证它们是否均已通过。

## 11\. 结尾

恭喜，您已成功完成此 Codelab！您可以[在此处找到解决方案代码](https://github.com/googlecodelabs/android-compose-codelabs/tree/end/NavigationCodelab)，并将其与您的代码进行比较。

您已向 Rally 应用添加了 Jetpack Compose Navigation，现在已经熟悉了它的关键概念。您学习了如何设置可组合目的地的导航图、定义导航路线和操作、通过实参向路线传递额外信息、设置深层链接以及测试导航。

如需获取更多主题和信息（例如[底部导航栏集成](https://developer.android.com/jetpack/compose/navigation?hl=zh-cn#bottom-nav)、多模块导航和[嵌套图](https://developer.android.com/jetpack/compose/navigation?hl=zh-cn#nested-nav)），您可以访问 [Now in Android GitHub 代码库](https://github.com/android/nowinandroid)，了解具体的实现方式。

## 相关文章

+   [“测试 Compose”Codelab](https://developer.android.com/codelabs/jetpack-compose-testing?hl=zh-cn)
+   [“Jetpack Compose 中的常见性能问题”视频](https://www.youtube.com/watch?v=EOQB8PTLkpY&hl=zh-cn)

如需详细了解 Jetpack Navigation，请参阅以下内容：

+   [Jetpack 导航](https://developer.android.com/guide/navigation?hl=zh-cn)
+   [“了解 Jetpack Navigation”Codelab](https://developer.android.com/codelabs/android-navigation?hl=zh-cn)

## 参考文档

+   [“使用 Compose 进行导航”文档](https://developer.android.com/jetpack/compose/navigation?hl=zh-cn)
+   [Navigation Compose 参考文档](https://developer.android.com/reference/kotlin/androidx/navigation/compose/package-summary?hl=zh-cn)