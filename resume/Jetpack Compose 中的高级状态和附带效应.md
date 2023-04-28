## 1\. 简介

在此 Codelab 中，您将学习与 [Jetpack Compose](https://developer.android.com/jetpack/compose?hl=zh-cn) 中的[状态](https://developer.android.com/jetpack/compose/state?hl=zh-cn)和[附带效应](https://developer.android.com/jetpack/compose/side-effects?hl=zh-cn) API 相关的高级概念。我们将了解如何为逻辑并不简单的有状态可组合项创建状态容器，如何从 Compose 代码创建协程并调用挂起函数，以及如何触发附带效应以完成不同的用例。

## 学习内容

+   如何从 Compose 代码观察数据流以更新界面。
+   如何为有状态可组合项创建状态容器。
+   附带效应 API，如 [`LaunchedEffect`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/package-summary?hl=zh-cn#LaunchedEffect(kotlin.Any,kotlin.coroutines.SuspendFunction1))、[`rememberUpdatedState`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/package-summary?hl=zh-cn#rememberUpdatedState(kotlin.Any))、[`DisposableEffect`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/package-summary?hl=zh-cn#DisposableEffect(kotlin.Any,kotlin.Function1))、[`produceState`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/package-summary?hl=zh-cn#produceState(kotlin.Any,kotlin.coroutines.SuspendFunction1)) 和 [`derivedStateOf`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/package-summary?hl=zh-cn#derivedStateOf(kotlin.Function0))。
+   如何使用 [`rememberCoroutineScope`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/package-summary?hl=zh-cn#rememberCoroutineScope(kotlin.Function0)) API 在可组合项中创建协程并调用挂起函数。

## 所需条件

+   [最新版 Android Studio](https://developer.android.com/studio?hl=zh-cn)
+   有使用 Kotlin 语法（包括 lambda）的经验。
+   有使用 Compose 的基本经验。建议您在学习此 Codelab 之前先学习 [Jetpack Compose 基础知识 Codelab](https://codelabs.developers.google.com/codelabs/jetpack-compose-basics/?hl=zh-cn)。
+   了解 [Compose 中的基本状态概念](https://developer.android.com/jetpack/compose/state?hl=zh-cn)，如单向数据流 (UDF)、ViewModel、状态提升、无状态/有状态可组合项、槽位 API，以及 `remember` 和 `mutableStateOf` 状态 API。如需获得这些知识，建议您阅读[“状态和 Jetpack Compose”文档](https://developer.android.com/jetpack/compose/state?hl=zh-cn)或完成[“在 Jetpack Compose 中使用状态”Codelab](https://developer.android.com/codelabs/jetpack-compose-state?hl=zh-cn#0)。
+   具备有关 [Kotlin 协程](https://developer.android.com/kotlin/coroutines?hl=zh-cn)的基础知识。
+   对[可组合项的生命周期](https://developer.android.com/jetpack/compose/lifecycle?hl=zh-cn)有基本的了解。

## 构建内容

在此 Codelab 中，我们将从一个未完成的应用（即 [Crane 材料研究](https://material.io/design/material-studies/crane.html)应用）着手，并且将添加一些功能来改进该应用。

![1.gif](/images/compose_status/1.gif)

## 2\. 准备工作

## 获取代码

此 Codelab 的代码可以在 [android-compose-codelabs GitHub 代码库](https://github.com/googlecodelabs/android-compose-codelabs)中找到。如需克隆该代码库，请运行以下命令：

$ git clone https://github.com/googlecodelabs/android-compose-codelabs

或者，您也能以 Zip 文件的形式下载该代码库：

[file_download下载 ZIP 文件](https://github.com/googlecodelabs/android-compose-codelabs/archive/main.zip)

## 查看示例应用

您刚刚下载的代码包含提供的所有 Compose Codelab 的代码。为了完成此 Codelab，请在 Android Studio 中打开 **`AdvancedStateAndSideEffectsCodelab`** 项目。

我们建议您从 main 分支中的代码着手，按照自己的节奏逐步完成此 Codelab。

在本 Codelab 中，系统会为您显示需要添加到项目的代码段。在某些地方，您还需要移除在代码段的注释中明确提及的代码。

## 熟悉代码并运行示例应用

先花点时间浏览一下项目结构，然后再运行应用。

![2.png](/images/compose_status/2.png)

从 main 分支运行应用时，您会看到某些功能（如抽屉式导航栏或加载航班目的地的功能）不起作用！这就是我们在此 Codelab 的后续步骤中要改进的地方。

![3.gif](/images/compose_status/3.gif)

## 界面测试

系统会对应用执行 `androidTest` 文件夹中提供的非常基本的界面测试。对于 `main` 和 `end` 分支，始终都应通过这些测试。

## \[可选\] 在详情屏幕上显示地图

完全没有必要按照相关说明在详情屏幕上显示城市地图。不过，如果您想要看到地图，那么需要获取个人 API 密钥，如[“地图”文档](https://developers.google.com/maps/documentation/android-sdk/get-api-key?hl=zh-cn)中所述。按如下方式在 `local.properties` 文件中添加该密钥：

```auto
// local.properties file
google.maps.key={insert_your_api_key_here}
```

## 此 Codelab 的解决方案

如需使用 git 获取 `end` 分支，请使用以下命令：

$ git clone -b end https://github.com/googlecodelabs/android-compose-codelabs

或者，您也可以在此处下载解决方案代码：

[file_download下载最终代码](https://github.com/googlecodelabs/android-compose-codelabs/archive/end.zip)

## 常见问题解答

+   [如何安装 Android Studio？](https://developer.android.com/studio/install?hl=zh-cn)
+   [如何针对开发设置设备？](http://developer.android.com/tools/device.html?hl=zh-cn)

## 3\. 界面状态生成流水线

您可能已经注意到，从 `main` 分支运行应用时，航班目的地列表为空！

如要解决此问题，您必须完成以下两个步骤：

+   在 [`ViewModel`](https://developer.android.com/topic/libraries/architecture/viewmodel?hl=zh-cn) 中添加逻辑以生成[界面状态](https://developer.android.com/topic/architecture/ui-layer/stateholders?hl=zh-cn#elements-ui)。在本示例中，即推荐目的地列表。
+   从界面取用该界面状态，这样就会在画面上显示界面。

在本部分中，您将完成第一步。

良好的应用架构会分层级，遵循基本的良好系统设计实践，例如关注点分离和可测试性。

[界面状态生成](https://developer.android.com/topic/architecture/ui-layer/state-production?hl=zh-cn)是指以下过程：应用访问数据层、应用业务规则（如果需要），以及公开要从界面取用的界面状态。

此应用中的数据层已实现。现在，您将生成状态（推荐目的地列表），以便界面可以取用该状态。

有几个 API 可用于生成界面状态。如要了解替代方案，请参阅[状态生成流水线中的输出类型](https://developer.android.com/topic/architecture/ui-layer/state-production?hl=zh-cn#output-types)文档。一般而言，最好使用 Kotlin 的 [`StateFlow`](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-state-flow/) 生成界面状态。

如要生成界面状态，请按以下步骤操作：

1.  打开 `home/MainViewModel.kt`。
2.  定义一个类型为 [`MutableStateFlow`](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-mutable-state-flow/) 的私有 `_suggestedDestinations` 变量，用于表示推荐目的地列表，并将空列表设置为起始值。

```auto
private val _suggestedDestinations = MutableStateFlow<List<ExploreModel>>(emptyList())
```

3.  定义第二个不可变变量 `suggestedDestinations`，类型为 `StateFlow`。这是可从界面取用的公开只读变量。建议您公开只读变量，并在内部使用可变变量。这样做可确保界面状态无法修改，除非通过 `ViewModel` 使其成为单一可信来源。扩展函数 [`asStateFlow`](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/as-state-flow.html) 会将可变流转换为不可变流。

```auto
private val _suggestedDestinations = MutableStateFlow<List<ExploreModel>>(emptyList())

val suggestedDestinations: StateFlow<List<ExploreModel>> = _suggestedDestinations.asStateFlow()
```

4.  在 `ViewModel` 的 init 块中，添加来自 `destinationsRepository` 的调用，以便从数据层获取目的地。

```auto
private val _suggestedDestinations = MutableStateFlow<List<ExploreModel>>(emptyList())

val suggestedDestinations: StateFlow<List<ExploreModel>> = _suggestedDestinations.asStateFlow()

init {
    _suggestedDestinations.value = destinationsRepository.destinations
}
```

5.  最后，取消注释在此类中找到的内部变量 `_suggestedDestinations` 使用情况，以便可通过来自界面的事件正确更新该变量。

这样就完成第一步了！现在，`ViewModel` 能够生成界面状态。在下一步中，您将从界面取用此状态。

## 4\. 从 ViewModel 安全地使用流

航班目的地列表仍然为空。在上一步中，您在 `MainViewModel` 中生成了界面状态。现在，您将使用要在界面中显示并由 `MainViewModel` 公开的界面状态。

打开 `home/CraneHome.kt` 文件并查看 `CraneHomeContent` 可组合项。

被分配给一个记住的空列表的 `suggestedDestinations` 的定义上面有一条 TODO 注释。这就是屏幕上显示的内容：一个空列表！在此步骤中，我们将解决该问题，并显示 `MainViewModel` 公开的推荐目的地。

![4.png](/images/compose_status/4.png)

打开 `home/MainViewModel.kt` 并查看 `suggestedDestinations` [StateFlow](https://developer.android.com/kotlin/flow/stateflow-and-sharedflow?hl=zh-cn)，该 StateFlow 初始化为 `destinationsRepository.destinations`，并且会在调用 `updatePeople` 或 `toDestinationChanged` 函数时得到更新。

您希望每当有新项被发送到 `suggestedDestinations` 数据流时 `CraneHomeContent` 可组合项中的界面都会更新。您可以使用 [`collectAsStateWithLifecycle()`](https://developer.android.com/reference/kotlin/androidx/lifecycle/compose/package-summary?hl=zh-cn#(kotlinx.coroutines.flow.Flow).collectAsStateWithLifecycle(kotlin.Any,androidx.lifecycle.Lifecycle,androidx.lifecycle.Lifecycle.State,kotlin.coroutines.CoroutineContext)) 函数。`collectAsStateWithLifecycle()` 会以生命周期感知型方式从 `StateFlow` 收集值并通过 Compose 的 [State](https://developer.android.com/reference/kotlin/androidx/compose/runtime/State?hl=zh-cn) API 表示最新值。这样会使读取该状态值的 Compose 代码在发出新项时重组。

如需开始使用 `collectAsStateWithLifecycle` API，请先在 `app/build.gradle` 中添加以下[依赖项](https://developer.android.com/jetpack/androidx/releases/lifecycle?hl=zh-cn)。变量 `lifecycle_version` 已在项目中使用适当版本进行定义。

```auto
dependencies {
    implementation "androidx.lifecycle:lifecycle-runtime-compose:$lifecycle_version"
}
```

返回 `CraneHomeContent` 可组合项，并将分配 `suggestedDestinations` 的代码行替换为 `ViewModel` 的 `suggestedDestinations` 属性上的 `collectAsStateWithLifecycle` 调用：

```auto
import androidx.lifecycle.compose.collectAsStateWithLifecycle

@Composable
fun CraneHomeContent(
    onExploreItemClicked: OnExploreItemClicked,
    openDrawer: () -> Unit,
    modifier: Modifier = Modifier,
    viewModel: MainViewModel = viewModel(),
) {
    val suggestedDestinations by viewModel.suggestedDestinations.collectAsStateWithLifecycle()
    // ...
}
```

如果您运行应用，您会看到目的地列表已填充，并且每当您点按旅行人数时，目的地都会发生变化。

![5.gif](/images/compose_status/5.gif)

## 5\. LaunchedEffect 和 rememberUpdatedState

在该项目中，有一个目前未使用的 `home/LandingScreen.kt` 文件。我们想要向应用添加一个着陆屏幕，它有可能会用于在后台加载需要的所有数据。

着陆屏幕将占据整个屏幕，并在屏幕中间显示应用的徽标。理想情况下，我们会显示该屏幕，在所有数据加载完毕之后，我们会通知调用方可以使用 `onTimeout` 回调关闭着陆屏幕。

建议使用 Kotlin 协程在 Android 中执行异步操作。应用在启动时通常会使用协程在后台加载内容。Jetpack Compose 提供了可让您在界面层中安全使用协程的 API。由于此应用不与后端进行通信，因此我们将使用协程的 [`delay`](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/delay.html) 函数来模拟在后台加载内容。

Compose 中的附带效应是指发生在可组合函数作用域之外的应用状态的变化。将状态更改为显示/隐藏着陆屏幕的操作将发生在 `onTimeout` 回调中，由于在调用 `onTimeout` 之前我们需要先使用协程加载内容，因此状态变化必须发生在协程的上下文中！

如需从可组合项内安全地调用挂起函数，请使用 [`LaunchedEffect`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/package-summary?hl=zh-cn#LaunchedEffect(kotlin.Any,kotlin.coroutines.SuspendFunction1)) API，该 API 会在 Compose 中触发协程作用域限定的附带效应。

当 `LaunchedEffect` 进入组合时，它会启动一个协程，并将代码块作为参数传递。如果 `LaunchedEffect` 退出组合，协程将取消。

虽然接下来的代码不正确，但让我们看看如何使用此 API，并探讨为什么下面的代码是错误的。我们将在此步骤的后面调用 `LandingScreen` 可组合项。

```auto
// home/LandingScreen.kt file

import androidx.compose.runtime.LaunchedEffect
import kotlinx.coroutines.delay

@Composable
fun LandingScreen(onTimeout: () -> Unit, modifier: Modifier = Modifier) {
    Box(modifier = modifier.fillMaxSize(), contentAlignment = Alignment.Center) {
        // Start a side effect to load things in the background
        // and call onTimeout() when finished.
        // Passing onTimeout as a parameter to LaunchedEffect
        // is wrong! Don't do this. We'll improve this code in a sec.
        LaunchedEffect(onTimeout) {
            delay(SplashWaitTime) // Simulates loading things
            onTimeout()
        }
        Image(painterResource(id = R.drawable.ic_crane_drawer), contentDescription = null)
    }
}
```

某些附带效应 API（如 `LaunchedEffect`）会将可变数量的键作为参数，用于在其中一个键发生更改时**重新开始**效应。您发现错误了吗？我们不希望在此可组合函数的调用方传递不同的 `onTimeout` lambda 值时重启 `LaunchedEffect`。这会让 `delay` 再次启动，使得我们无法满足相关要求。

接下来，让我们解决这个问题。如需在[此可组合项的生命周期](https://developer.android.com/jetpack/compose/lifecycle?hl=zh-cn)内仅触发一次附带效应，请将常量用作键，例如 `LaunchedEffect(Unit) { ... }`。不过，现在又有一个问题。

如果 `onTimeout` 在附带效应正在进行时发生变化，效应结束时不一定会调用最后一个 `onTimeout`。如需保证调用最后一个 `onTimeout`，请使用 [`rememberUpdatedState`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/package-summary?hl=zh-cn#rememberUpdatedState(kotlin.Any)) API 记住 `onTimeout`。此 API 会捕获并更新最新值：

```auto
// home/LandingScreen.kt file

import androidx.compose.runtime.getValue
import androidx.compose.runtime.rememberUpdatedState
import kotlinx.coroutines.delay

@Composable
fun LandingScreen(onTimeout: () -> Unit, modifier: Modifier = Modifier) {
    Box(modifier = modifier.fillMaxSize(), contentAlignment = Alignment.Center) {
        // This will always refer to the latest onTimeout function that
        // LandingScreen was recomposed with
        val currentOnTimeout by rememberUpdatedState(onTimeout)

        // Create an effect that matches the lifecycle of LandingScreen.
        // If LandingScreen recomposes or onTimeout changes,
        // the delay shouldn't start again.
        LaunchedEffect(Unit) {
            delay(SplashWaitTime)
            currentOnTimeout()
        }

        Image(painterResource(id = R.drawable.ic_crane_drawer), contentDescription = null)
    }
}
```

当长期存在的 lambda 或对象表达式引用在组合期间计算的参数或值时，您应使用 `rememberUpdatedState`，这在使用 `LaunchedEffect` 时可能很常见。

## 显示着陆屏幕

现在，我们需要在应用打开后显示着陆屏幕。打开 `home/MainActivity.kt` 文件，并查看首次调用的 `MainScreen` 可组合项。

在 `MainScreen` 可组合项中，我们只需添加一种内部状态，用来跟踪是否应显示着陆屏幕：

```auto
// home/MainActivity.kt file

import androidx.compose.runtime.getValue
import androidx.compose.runtime.mutableStateOf
import androidx.compose.runtime.remember
import androidx.compose.runtime.setValue

@Composable
private fun MainScreen(onExploreItemClicked: OnExploreItemClicked) {
    Surface(color = MaterialTheme.colors.primary) {
        var showLandingScreen by remember { mutableStateOf(true) }
        if (showLandingScreen) {
            LandingScreen(onTimeout = { showLandingScreen = false })
        } else {
            CraneHome(onExploreItemClicked = onExploreItemClicked)
        }
    }
}
```

如果您现在运行应用，您应该会看到 `LandingScreen` 出现并在 2 秒后消失。

![6.gif](/images/compose_status/6.gif)

## 6\. rememberCoroutineScope

在此步骤中，我们将使抽屉式导航栏正常工作。目前，如果您尝试点按汉堡式菜单，什么都不会发生。

打开 `home/CraneHome.kt` 文件，并查看 `CraneHome` 可组合项，看看我们需要在何处打开抽屉式导航栏：在 `openDrawer` 回调中！

在 `CraneHome` 中，有一个包含 `DrawerState` 的 [`scaffoldState`](https://developer.android.com/reference/kotlin/androidx/compose/material/ScaffoldState?hl=zh-cn)。[`DrawerState`](https://developer.android.com/reference/kotlin/androidx/compose/material/DrawerState?hl=zh-cn) 具有以程序化方式打开和关闭抽屉式导航栏的方法。不过，如果您尝试在 `openDrawer` 回调中编写 `scaffoldState.drawerState.open()`，您会收到一条错误消息！这是因为，`open` 函数是一个**挂起**函数。我们再次进入协程的领域。

除了可让您从界面层安全调用协程的 API 之外，某些 Compose API 是挂起函数。用于打开抽屉式导航栏的 API 就是一个这样的例子。挂起函数除了能够运行异步代码之外，还可以帮助表示随着时间的推移出现的概念。由于打开抽屉式导航栏需要一些时间和移动，而且还有可能需要动画，这可以通过挂起函数完美地反映出来，挂起函数将在被调用的地方暂停协程的执行，直到它完成，然后再继续执行。

必须在协程中调用 `scaffoldState.drawerState.open()`。我们能做些什么呢？`openDrawer` 是一个简单的回调函数，因此：

+   我们不能简单地在其中调用挂起函数，因为 `openDrawer` 不在协程的上下文中执行。
+   我们不能像之前一样使用 `LaunchedEffect`，因为我们不能在 `openDrawer` 中调用可组合项。我们并不在组合中。

我们希望启动一个协程；我们应使用哪个作用域呢？理想情况下，我们希望 `CoroutineScope` 能够遵循其调用点的生命周期。如果使用 [`rememberCoroutineScope`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/package-summary?hl=zh-cn#rememberCoroutineScope(kotlin.Function0)) API，则会返回一个 `CoroutineScope`，该 CoroutineScope 会绑定到它在组合中的调用点。一旦退出组合，作用域将自动取消。有了这个作用域，即使您不在组合中（例如，在 `openDrawer` 回调中），也可以启动协程。

```auto
// home/CraneHome.kt file

import androidx.compose.runtime.rememberCoroutineScope
import kotlinx.coroutines.launch

@Composable
fun CraneHome(
    onExploreItemClicked: OnExploreItemClicked,
    modifier: Modifier = Modifier,
) {
    val scaffoldState = rememberScaffoldState()
    Scaffold(
        scaffoldState = scaffoldState,
        modifier = Modifier.statusBarsPadding(),
        drawerContent = {
            CraneDrawer()
        }
    ) {
        val scope = rememberCoroutineScope()
        CraneHomeContent(
            modifier = modifier,
            onExploreItemClicked = onExploreItemClicked,
            openDrawer = {
                scope.launch {
                    scaffoldState.drawerState.open()
                }
            }
        )
    }
}
```

如果您运行应用，您会看到当您点按汉堡式菜单图标时，系统会打开抽屉式导航栏。

![7.gif](/images/compose_status/7.gif)

## LaunchedEffect 与 rememberCoroutineScope

在这种情况下无法使用 `LaunchedEffect`，因为我们需要触发调用以在组合之外的常规回调中创建协程。

回顾一下使用 `LaunchedEffect` 的着陆屏幕步骤，您可以使用 `rememberCoroutineScope` 并调用 `scope.launch { delay(); onTimeout(); }` 而不使用 `LaunchedEffect` 吗？

您本来可以这样做，而且似乎可行，但这样并不正确。如[“Compose 编程思想”文档](https://developer.android.com/jetpack/compose/mental-model?hl=zh-cn#any-order)中所述，Compose 可以随时调用可组合项。`LaunchedEffect` 可以保证当对该可组合项的调用使其进入组合时将会执行附带效应。如果您在 `LandingScreen` 的主体中使用 `rememberCoroutineScope` 和 `scope.launch`，则每次 Compose 调用 `LandingScreen` 时都会执行协程，而不管该调用是否使其进入组合。因此，您会浪费资源，而且不会在受控环境中执行此附带效应。

## 7\. 创建状态容器

您注意到了吗？如果您点按“Choose Destination”，您可以修改该字段，并根据搜索输入过滤城市。此外，您或许还注意到了，每当您修改“Choose Destination”时，文本样式都会发生变化。

![8.gif](/images/compose_status/8.gif)

打开 `base/EditableUserInput.kt` 文件。`CraneEditableUserInput` 有状态可组合项接受一些参数，如 `hint` 和 `caption`，后者对应于图标旁边的可选文本。例如，当您搜索目的地时，会出现 `caption`“To”。

```auto
// base/EditableUserInput.kt file - code in the main branch

@Composable
fun CraneEditableUserInput(
    hint: String,
    caption: String? = null,
    @DrawableRes vectorImageId: Int? = null,
    onInputChanged: (String) -> Unit
) {
    // TODO Codelab: Encapsulate this state in a state holder
    var textState by remember { mutableStateOf(hint) }
    val isHint = { textState == hint }

    ...
}
```

## 为什么？

用于更新 `textState` 以及确定显示的内容是否对应于提示的逻辑全部都在 `CraneEditableUserInput` 可组合项的主体中。这就带来了一些缺点：

+   `TextField` 的值未提升，因而无法从外部进行控制，这使得测试更加困难。
+   此可组合项的逻辑可能会变得更加复杂，并且内部状态可能会更容易不同步。

通过创建负责此可组合项的内部状态的状态容器，您可以将所有状态变化集中在一个位置。这样，状态不同步就更难了，并且相关的逻辑全部归在一个类中。此外，此状态很容易向上提升，并且可以从此可组合项的调用方使用。

在这种情况下，提升状态是一种不错的做法，因为这是一个低级界面组件，可能会在应用的其他部分中重复使用。因此，它越灵活越可控，就越好。

## 创建状态容器

由于 `CraneEditableUserInput` 是一个可重复使用的组件，让我们在同一文件中创建一个名为 `EditableUserInputState` 的常规类作为状态容器，如下所示：

```auto
// base/EditableUserInput.kt file

import androidx.compose.runtime.getValue
import androidx.compose.runtime.mutableStateOf
import androidx.compose.runtime.setValue

class EditableUserInputState(private val hint: String, initialText: String) {

    var text by mutableStateOf(initialText)
       private set

    fun updateText(newText: String) {
       text = newText
    }

    val isHint: Boolean
        get() = text == hint
}
```

该类应具有以下特征：

+   `text` 是 `String` 类型的可变状态，就像在 `CraneEditableUserInput` 中一样。请务必使用 `mutableStateOf`，以便 Compose 跟踪值的更改，并在发生更改时重组。
+   `text` 是具有私有 `set` 的 `var`，因此无法直接从类外部改变它。您可以公开 `updateText` 事件来对此变量进行修改，从而将该类设为单一可信来源，而不是公开此变量。
+   该类将 `initialText` 作为用于初始化 `text` 的依赖项。
+   用于判断 `text` 是否为提示的逻辑在按需执行检查的 `isHint` 属性中。

如果将来逻辑变得更加复杂，我们只需要对一个类进行更改：`EditableUserInputState`。

## 记住状态容器

始终需要记住状态容器，以使其留在组合中，而不是每次都创建一个新的。最好在同一文件中创建一个执行此操作的方法，以移除样板并避免可能发生的任何错误。在 `base/EditableUserInput.kt` 文件中，添加以下代码：

```auto
// base/EditableUserInput.kt file

@Composable
fun rememberEditableUserInputState(hint: String): EditableUserInputState =
    remember(hint) {
        EditableUserInputState(hint, hint)
    }
```

如果我们只是使用 `remember` 记住此状态，它在 activity 重新创建后不会继续留存。为了解决此问题，我们可以改用 [**`rememberSaveable`**](https://developer.android.com/reference/kotlin/androidx/compose/runtime/saveable/package-summary?hl=zh-cn#rememberSaveable(kotlin.Array,androidx.compose.runtime.saveable.Saver,kotlin.String,kotlin.Function0)) API，它的行为方式与 `remember` 类似，但存储的值在 activity 和进程重新创建后会继续留存。在内部，它使用保存的实例状态机制。

对于可以存储在 [`Bundle`](https://developer.android.com/reference/android/os/Bundle?hl=zh-cn) 内的对象，`rememberSaveable` 可以做所有这些工作，而无需任何额外的操作。对于我们在项目中创建的 `EditableUserInputState` 类，却并非如此。因此，我们需要告知 `rememberSaveable` 如何使用 [`Saver`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/saveable/package-summary?hl=zh-cn#Saver(kotlin.Function2,kotlin.Function1)) 保存和恢复此类的实例。

## 创建自定义保存器

[`Saver`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/saveable/package-summary?hl=zh-cn#Saver(kotlin.Function2,kotlin.Function1)) 描述了如何将对象转换为 [`Saveable`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/saveable/Saver?hl=zh-cn)（可保存）的内容。[`Saver`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/saveable/package-summary?hl=zh-cn#Saver(kotlin.Function2,kotlin.Function1)) 的实现需要替换两个函数：

+   `save` - 将原始值转换为可保存的值。
+   `restore` - 将恢复的值转换为原始类的实例。

在本例中，我们可以使用一些现有的 Compose API，如 [`listSaver`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/saveable/package-summary?hl=zh-cn#listSaver(kotlin.Function2,kotlin.Function1)) 或 [`mapSaver`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/saveable/package-summary?hl=zh-cn#mapSaver(kotlin.Function2,kotlin.Function1))（用于存储要保存在 `List` 或 `Map` 中的值），以减少我们需要编写的代码量，而不是为 `EditableUserInputState` 类创建 [`Saver`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/saveable/package-summary?hl=zh-cn#Saver(kotlin.Function2,kotlin.Function1)) 的自定义实现。

最好将 `Saver` 定义放置在与其一起使用的类附近。由于需要被静态访问，因此让我们在 `companion object` 中为 `EditableUserInputState` 添加 `Saver`。在 `base/EditableUserInput.kt` 文件中，添加 `Saver` 的实现：

```auto
// base/EditableUserInput.kt file

import androidx.compose.runtime.saveable.Saver
import androidx.compose.runtime.saveable.listSaver

class EditableUserInputState(private val hint: String, initialText: String) {
    var text by mutableStateOf(initialText)

    val isHint: Boolean
        get() = text == hint

    companion object {
        val Saver: Saver<EditableUserInputState, *> = listSaver(
            save = { listOf(it.hint, it.text) },
            restore = {
                EditableUserInputState(
                    hint = it[0],
                    initialText = it[1],
                )
            }
        )
    }
}
```

在本例中，我们将 `listSaver` 用作实现细节，在保存器中存储和恢复 `EditableUserInputState` 的实例。

现在，我们可以在之前创建的 `rememberEditableUserInputState` 方法的 `rememberSaveable`（而不是 `remember`）中使用此保存器：

```auto
// base/EditableUserInput.kt file
import androidx.compose.runtime.saveable.rememberSaveable

@Composable
fun rememberEditableUserInputState(hint: String): EditableUserInputState =
    rememberSaveable(hint, saver = EditableUserInputState.Saver) {
        EditableUserInputState(hint, hint)
    }
```

这样，`EditableUserInput` 记住的状态就会在进程和 activity 重新创建后继续留存。

## 使用状态容器

我们将要使用 `EditableUserInputState` 而不是 `text` 和 `isHint`，但我们不希望只将其用作 `CraneEditableUserInput` 中的内部状态，因为调用方可组合项无法控制状态。相反，我们希望提升 `EditableUserInputState`，以便调用方可以控制 `CraneEditableUserInput` 的状态。如果我们提升状态，那么可组合项就可以在预览中使用，并且更容易进行测试，因为您能够从调用方修改其状态。

为此，我们需要更改可组合函数的参数，并在需要时为其提供默认值。由于我们可能希望允许 `CraneEditableUserInput` 带有空提示，因此我们添加一个默认参数：

```auto
@Composable
fun CraneEditableUserInput(
    state: EditableUserInputState = rememberEditableUserInputState(""),
    caption: String? = null,
    @DrawableRes vectorImageId: Int? = null
) { /* ... */ }
```

您或许已经注意到，`onInputChanged` 参数不存在了！由于状态可以提升，因此如果调用方想要知道输入是否发生了更改，它们可以控制状态并将该状态传入此函数。

接下来，我们需要调整函数主体，以使用提升的状态，而不是之前使用的内部状态。重构后，函数应如下所示：

```auto
@Composable
fun CraneEditableUserInput(
    state: EditableUserInputState = rememberEditableUserInputState(""),
    caption: String? = null,
    @DrawableRes vectorImageId: Int? = null
) {
    CraneBaseUserInput(
        caption = caption,
        tintIcon = { !state.isHint },
        showCaption = { !state.isHint },
        vectorImageId = vectorImageId
    ) {
        BasicTextField(
            value = state.text,
            onValueChange = { state.updateText(it) },
            textStyle = if (state.isHint) {
                captionTextStyle.copy(color = LocalContentColor.current)
            } else {
                MaterialTheme.typography.body1.copy(color = LocalContentColor.current)
            },
            cursorBrush = SolidColor(LocalContentColor.current)
        )
    }
}
```

## 状态容器调用方

由于我们更改了 `CraneEditableUserInput` 的 API，因此需要在调用它的所有位置进行检查，以确保传入适当的参数。

在项目中，我们只在一个位置调用此 API，那就是在 `home/SearchUserInput.kt` 文件中。打开该文件并转到 `ToDestinationUserInput` 可组合函数；您应该会在该位置看到一个构建错误。由于提示现在是状态容器的一部分，并且我们希望在组合中设置此 `CraneEditableUserInput` 实例的自定义提示，因此我们需要记住 `ToDestinationUserInput` 级别的状态，并将其传入 `CraneEditableUserInput`：

```auto
// home/SearchUserInput.kt file

import androidx.compose.samples.crane.base.rememberEditableUserInputState

@Composable
fun ToDestinationUserInput(onToDestinationChanged: (String) -> Unit) {
    val editableUserInputState = rememberEditableUserInputState(hint = "Choose Destination")
    CraneEditableUserInput(
        state = editableUserInputState,
        caption = "To",
        vectorImageId = R.drawable.ic_plane
    )
}
```

## snapshotFlow

上面的代码缺少在输入更改时通知 `ToDestinationUserInput` 的调用方的功能。由于应用的结构，我们不希望在层次结构中将 `EditableUserInputState` 提升到任何更高的级别。而且，我们也不希望将其他可组合项（如 `FlySearchContent`）与此状态相结合。我们如何从 `ToDestinationUserInput` 调用 `onToDestinationChanged` lambda 并且仍使此可组合项可重复使用呢？

我们可以在每次输入更改时使用 `LaunchedEffect` 触发附带效应，并调用 `onToDestinationChanged` lambda：

```auto
// home/SearchUserInput.kt file

import androidx.compose.runtime.LaunchedEffect
import androidx.compose.runtime.rememberUpdatedState
import androidx.compose.runtime.snapshotFlow
import kotlinx.coroutines.flow.collect
import kotlinx.coroutines.flow.filter

@Composable
fun ToDestinationUserInput(onToDestinationChanged: (String) -> Unit) {
    val editableUserInputState = rememberEditableUserInputState(hint = "Choose Destination")
    CraneEditableUserInput(
        state = editableUserInputState,
        caption = "To",
        vectorImageId = R.drawable.ic_plane
    )

    val currentOnDestinationChanged by rememberUpdatedState(onToDestinationChanged)
    LaunchedEffect(editableUserInputState) {
        snapshotFlow { editableUserInputState.text }
            .filter { !editableUserInputState.isHint }
            .collect {
                currentOnDestinationChanged(editableUserInputState.text)
            }
    }
}
```

我们之前已经使用了 `LaunchedEffect` 和 `rememberUpdatedState`，但上面的代码还使用了一个新的 API！我们使用 [**`snapshotFlow`**](https://developer.android.com/reference/kotlin/androidx/compose/runtime/package-summary?hl=zh-cn#snapshotFlow(kotlin.Function0)) **API** 将 Compose [`State<T>`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/State?hl=zh-cn) 对象转换为 [Flow](https://developer.android.com/kotlin/flow?hl=zh-cn)。当在 `snapshotFlow` 内读取的状态发生变化时，Flow 会向收集器发出新值。在本例中，我们将状态转换为 Flow，以使用 Flow 运算符的强大功能。这样，我们就可以在 `text` 不是 `hint` 时使用 `filter` 进行过滤，并使用 `collect` 收集发出的项，以通知父级当前的目的地发生了变化。

在此 Codelab 的这一步骤中没有任何视觉变化，但我们改进了这部分代码的质量。如果您现在运行应用，您应该会看到一切都像以前一样运作。

## 8\. DisposableEffect

当您点按某个目的地时，系统会打开详情屏幕，您可以看到相应的城市在地图上的位置。该代码位于 `details/DetailsActivity.kt` 文件中。在 `CityMapView` 可组合项中，我们调用 `rememberMapViewWithLifecycle` 函数。如果您打开此函数（它位于 `details/MapViewUtils.kt` 文件中），您会看到它未关联到任何生命周期！它只是记住 `MapView` 并对其调用 `onCreate`：

```auto
// details/MapViewUtils.kt file - code in the main branch

@Composable
fun rememberMapViewWithLifecycle(): MapView {
    val context = LocalContext.current
    // TODO Codelab: DisposableEffect step. Make MapView follow the lifecycle
    return remember {
        MapView(context).apply {
            id = R.id.map
            onCreate(Bundle())
        }
    }
}
```

虽然应用运行良好，但这也是一个问题，因为 `MapView` 未遵循正确的生命周期。因此，它不知道应用何时转至后台、View 何时应暂停，等等。让我们来解决这一问题！

由于 `MapView` 是 View 而不是可组合项，因此我们希望它遵循使用它的 Activity 的生命周期，以及组合的生命周期。这意味着，我们需要创建一个 [`LifecycleEventObserver`](https://developer.android.com/reference/androidx/lifecycle/LifecycleEventObserver?hl=zh-cn) 来监听生命周期事件并在 [`MapView`](https://developers.google.com/android/reference/com/google/android/gms/maps/MapView?hl=zh-cn) 上调用正确的方法。然后，我们需要将此观察器添加到当前 activity 的生命周期。

我们首先创建一个函数，该函数返回 `LifecycleEventObserver`，在给定某个事件的情况下，它会在 `MapView` 中调用相应的方法：

```auto
// details/MapViewUtils.kt file

import androidx.lifecycle.Lifecycle
import androidx.lifecycle.LifecycleEventObserver

private fun getMapLifecycleObserver(mapView: MapView): LifecycleEventObserver =
    LifecycleEventObserver { _, event ->
        when (event) {
            Lifecycle.Event.ON_CREATE -> mapView.onCreate(Bundle())
            Lifecycle.Event.ON_START -> mapView.onStart()
            Lifecycle.Event.ON_RESUME -> mapView.onResume()
            Lifecycle.Event.ON_PAUSE -> mapView.onPause()
            Lifecycle.Event.ON_STOP -> mapView.onStop()
            Lifecycle.Event.ON_DESTROY -> mapView.onDestroy()
            else -> throw IllegalStateException()
        }
    }
```

现在，我们需要将此观察器添加到当前的生命周期，我们可以使用当前的 [`LifecycleOwner`](https://developer.android.com/reference/androidx/lifecycle/LifecycleOwner?hl=zh-cn) 与 `LocalLifecycleOwner` 组合局部函数来获取该生命周期。不过，仅仅添加观察器是不够的；我们还需要能够将其移除！我们需要一种附带效应，可以在效应退出组合时告知我们，以便我们可以执行一些清理代码。我们寻找的附带效应 API 是 [`DisposableEffect`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/package-summary?hl=zh-cn#DisposableEffect(kotlin.Any,kotlin.Function1))。

`DisposableEffect` 适用于在键发生变化或可组合项退出组合后需要清理的附带效应。最终的 `rememberMapViewWithLifecycle` 代码正好起到这种作用。在项目中实现以下代码行：

```auto
// details/MapViewUtils.kt file

import androidx.compose.runtime.DisposableEffect
import androidx.compose.ui.platform.LocalLifecycleOwner

@Composable
fun rememberMapViewWithLifecycle(): MapView {
    val context = LocalContext.current
    val mapView = remember {
        MapView(context).apply {
            id = R.id.map
        }
    }

    val lifecycle = LocalLifecycleOwner.current.lifecycle
    DisposableEffect(key1 = lifecycle, key2 = mapView) {
        // Make MapView follow the current lifecycle
        val lifecycleObserver = getMapLifecycleObserver(mapView)
        lifecycle.addObserver(lifecycleObserver)
        onDispose {
            lifecycle.removeObserver(lifecycleObserver)
        }
    }

    return mapView
}
```

将观察器添加到了当前的 `lifecycle`，只要当前的生命周期发生变化或者此可组合项退出组合，就会将其移除。对于 `DisposableEffect` 中的 `key`，如果 `lifecycle` 或 `mapView` 发生变化，系统会移除观察器并再次将其添加到正确的 `lifecycle`。

通过我们刚刚所做的更改，`MapView` 将始终遵循当前 `LifecycleOwner` 的 `lifecycle`，并且其行为就像在 View 环境中使用它时一样。

您可以随意运行应用并打开详情屏幕，以确保 `MapView` 仍能正确呈现。这一步骤中没有任何视觉变化。

## 9\. produceState

在本部分中，我们将改进详情屏幕的启动方式。`details/DetailsActivity.kt` 文件中的 `DetailsScreen` 可组合项会从 ViewModel 同步获取 `cityDetails`，并在结果成功时调用 `DetailsContent`。

不过，`cityDetails` 在界面线程上的加载成本可能会变得越来越高，它可以使用协程将数据的加载工作移至其他线程。让我们来改进此代码，添加一个加载屏幕，并在数据准备就绪时显示 `DetailsContent`。

为屏幕状态建模的一种方法是使用以下类，它涵盖了所有的可能性：要在屏幕上显示的数据，以及加载和错误信号。将 `DetailsUiState` 类添加到 `DetailsActivity.kt` 文件：

```auto
// details/DetailsActivity.kt file

data class DetailsUiState(
    val cityDetails: ExploreModel? = null,
    val isLoading: Boolean = false,
    val throwError: Boolean = false
)
```

我们可以使用一个数据流（即 `DetailsUiState` 类型的 `StateFlow`）映射屏幕需要显示的内容和 ViewModel 层中的 `UiState`，ViewModel 会在信息准备就绪时更新该数据流，而 Compose 会使用您已了解的 `collectAsStateWithLifecycle()` API 收集该数据流。

不过，为了本练习，我们将实现一种替代方案。如果我们希望将 `uiState` 映射逻辑移至 Compose 环境，我们可以使用 [`produceState`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/package-summary?hl=zh-cn#produceState(kotlin.Any,kotlin.coroutines.SuspendFunction1)) API。

`produceState` 可让您将非 Compose 状态转换为 Compose [状态](https://developer.android.com/reference/kotlin/androidx/compose/runtime/State?hl=zh-cn)。它会启动一个作用域限定为组合的协程，该协程可使用 `value` 属性将值推送到返回的 `State`。与 `LaunchedEffect` 一样，`produceState` 也采用键来取消和重新开始计算。

对于我们的用例，我们可以使用 `produceState` 发出初始值为 `DetailsUiState(isLoading = true)` 的 `uiState` 更新，如下所示：

```auto
// details/DetailsActivity.kt file

import androidx.compose.runtime.produceState

@Composable
fun DetailsScreen(
    onErrorLoading: () -> Unit,
    modifier: Modifier = Modifier,
    viewModel: DetailsViewModel = viewModel()
) {

    val uiState by produceState(initialValue = DetailsUiState(isLoading = true)) {
        // In a coroutine, this can call suspend functions or move
        // the computation to different Dispatchers
        val cityDetailsResult = viewModel.cityDetails
        value = if (cityDetailsResult is Result.Success<ExploreModel>) {
            DetailsUiState(cityDetailsResult.data)
        } else {
            DetailsUiState(throwError = true)
        }
    }

    // TODO: ...
}
```

接下来，根据 `uiState`，我们会显示数据、显示加载屏幕或报告错误。下面是 `DetailsScreen` 可组合项的完整代码：

```auto
// details/DetailsActivity.kt file

import androidx.compose.foundation.layout.Box
import androidx.compose.material.CircularProgressIndicator

@Composable
fun DetailsScreen(
    onErrorLoading: () -> Unit,
    modifier: Modifier = Modifier,
    viewModel: DetailsViewModel = viewModel()
) {
    val uiState by produceState(initialValue = DetailsUiState(isLoading = true)) {
        val cityDetailsResult = viewModel.cityDetails
        value = if (cityDetailsResult is Result.Success<ExploreModel>) {
            DetailsUiState(cityDetailsResult.data)
        } else {
            DetailsUiState(throwError = true)
        }
    }

    when {
        uiState.cityDetails != null -> {
            DetailsContent(uiState.cityDetails!!, modifier.fillMaxSize())
        }
        uiState.isLoading -> {
            Box(modifier.fillMaxSize()) {
                CircularProgressIndicator(
                    color = MaterialTheme.colors.onSurface,
                    modifier = Modifier.align(Alignment.Center)
                )
            }
        }
        else -> { onErrorLoading() }
    }
}
```

如果您运行应用，您会看到在显示城市详情之前如何出现了指示“正在加载”的旋转图标。

![9.gif](/images/compose_status/9.gif)

## 10\. derivedStateOf

我们要对 Crane 做的最后一项改进是，每当您在航班目的地列表中滚动时，经过屏幕的第一个元素之后，都会显示一个用于“滚动至顶部”的按钮。点按该按钮会使您前往列表中的第一个元素。

![10.gif](/images/compose_status/10.gif)

打开包含此代码的 `base/ExploreSection.kt` 文件。`ExploreSection` 可组合项对应于您在 Scaffold 的背景幕中看到的内容。

为了计算用户是否已经过第一项，我们可以使用 [`LazyColumn`](https://developer.android.com/jetpack/compose/lists?hl=zh-cn) 的 [`LazyListState`](https://developer.android.com/reference/kotlin/androidx/compose/foundation/lazy/LazyListState?hl=zh-cn) 并检查 `listState.firstVisibleItemIndex > 0`。

简单实现如下所示：

```auto
// DO NOT DO THIS - It's executed on every recomposition
val showButton = listState.firstVisibleItemIndex > 0
```

该解决方案的效率并不高，因为每当 `firstVisibleItemIndex` 发生变化时，读取 `showButton` 的可组合函数都会重组，这种情况经常会在滚动时发生。不过，我们希望该函数仅在条件在 `true` 和 `false` 之间变化时重组。

有一个 API 可让我们做到这一点，那就是 [`derivedStateOf`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/package-summary?hl=zh-cn#derivedStateOf(kotlin.Function0)) API。

`listState` 是一个可观察的 Compose `State`。我们的计算 `showButton` 也需要是一个 Compose `State`，因为我们希望界面在其值发生变化时重组，并显示或隐藏按钮。

当您想要的某个 Compose `State` 衍生自另一个 `State` 时，请使用 `derivedStateOf`。每当内部状态发生变化时，系统都会执行 `derivedStateOf` 计算块，但只有当计算结果与上一项不同时，可组合函数才会重组。这样可以最大限度地减少读取 `showButton` 的函数的重组次数。

在这种情况下，使用 `derivedStateOf` API 是一种更好且更高效的替代方案。我们还会使用 `remember` API 来封装调用，因此计算得出的值在重组后继续有效。

```auto
// Show the button if the first visible item is past
// the first item. We use a remembered derived state to
// minimize unnecessary recompositions
val showButton by remember {
    derivedStateOf {
        listState.firstVisibleItemIndex > 0
    }
}
```

您应该已经熟悉 `ExploreSection` 可组合项的新代码。我们使用 `Box` 将根据条件显示的 `Button` 放置在 `ExploreList` 的顶部。我们会使用 `rememberCoroutineScope` 在 `Button` 的 `onClick` 回调内调用 `listState.scrollToItem` 挂起函数。

```auto
// base/ExploreSection.kt file

import androidx.compose.material.FloatingActionButton
import androidx.compose.runtime.derivedStateOf
import androidx.compose.runtime.getValue
import androidx.compose.runtime.remember
import androidx.compose.runtime.rememberCoroutineScope
import androidx.compose.foundation.layout.navigationBarsPadding
import kotlinx.coroutines.launch

@Composable
fun ExploreSection(
    modifier: Modifier = Modifier,
    title: String,
    exploreList: List<ExploreModel>,
    onItemClicked: OnExploreItemClicked
) {
    Surface(modifier = modifier.fillMaxSize(), color = Color.White, shape = BottomSheetShape) {
        Column(modifier = Modifier.padding(start = 24.dp, top = 20.dp, end = 24.dp)) {
            Text(
                text = title,
                style = MaterialTheme.typography.caption.copy(color = crane_caption)
            )
            Spacer(Modifier.height(8.dp))
            Box(Modifier.weight(1f)) {
                val listState = rememberLazyListState()
                ExploreList(exploreList, onItemClicked, listState = listState)

                // Show the button if the first visible item is past
                // the first item. We use a remembered derived state to
                // minimize unnecessary compositions
                val showButton by remember {
                    derivedStateOf {
                        listState.firstVisibleItemIndex > 0
                    }
                }
                if (showButton) {
                    val coroutineScope = rememberCoroutineScope()
                    FloatingActionButton(
                        backgroundColor = MaterialTheme.colors.primary,
                        modifier = Modifier
                            .align(Alignment.BottomEnd)
                            .navigationBarsPadding()
                            .padding(bottom = 8.dp),
                        onClick = {
                            coroutineScope.launch {
                                listState.scrollToItem(0)
                            }
                        }
                    ) {
                        Text("Up!")
                    }
                }
            }
        }
    }
}
```

如果运行应用，您会看到：一旦您滚动并经过屏幕的第一个元素，该按钮就会出现在底部。

## 11\. 结尾

恭喜您，您已成功完成了此 Codelab，并学习了 Jetpack Compose 应用中的状态和附带效应 API 的高级概念！

您学习了如何创建状态容器、附带效应 API（如 [`LaunchedEffect`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/package-summary?hl=zh-cn#LaunchedEffect(kotlin.Any,kotlin.coroutines.SuspendFunction1))、[`rememberUpdatedState`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/package-summary?hl=zh-cn#rememberUpdatedState(kotlin.Any))、[`DisposableEffect`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/package-summary?hl=zh-cn#DisposableEffect(kotlin.Any,kotlin.Function1))、[`produceState`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/package-summary?hl=zh-cn#produceState(kotlin.Any,kotlin.coroutines.SuspendFunction1)) 和 [`derivedStateOf`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/package-summary?hl=zh-cn#derivedStateOf(kotlin.Function0))），以及如何在 Jetpack Compose 中使用协程。
