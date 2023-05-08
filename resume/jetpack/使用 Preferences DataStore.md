## 1\. 简介

## 什么是 DataStore？

DataStore 是一个经过改进的新数据存储解决方案，旨在取代 SharedPreferences。DataStore 基于 Kotlin 协程和 Flow 构建而成，提供以下两种不同的实现：一种是 **Proto DataStore**，用于存储**类型化对象**（由[协议缓冲区](https://developers.google.com/protocol-buffers?hl=zh-cn)支持）；另一种是 **Preferences DataStore**，用于存储**键值对**。数据以异步、一致和事务性的方式存储，有助于避免 SharedPreferences 的一些缺点。

## 学习内容

+   什么是 DataStore？为什么应该使用它？
+   如何将 DataStore 添加到您的项目中？
+   Preferences DataStore 和 Proto DataStore 之间的区别及其各自的优点。
+   如何使用 Preferences DataStore？
+   如何从 SharedPreferences 迁移到 Preferences DataStore？

## **构建内容**

在此 Codelab 中，您将从一个示例应用入手开始构建。该应用会显示一个任务列表，其中的任务可以按照完成状态进行过滤，并可按优先级和截止时间进行排序。

![fcb2ffa4e6b77f33.gif](/images/dataStore_use/1.gif)

“显示已完成任务”过滤器的布尔值标志保存在内存中。使用 `SharedPreferences` 对象将排序顺序存留到磁盘。

在此 Codelab 中，您将完成以下任务，学会如何使用 **Preferences DataStore**：

+   在 DataStore 中存留“已完成”状态过滤器。
+   将排序顺序从 SharedPreferences 迁移到 DataStore。

此外，我们还建议您学完整个 Proto DataStore Codelab，以便更好地了解这两者之间的区别。

## **所需条件**

+   Android Studio Arctic Fox。
+   熟悉以下架构组件：[LiveData](https://developer.android.com/reference/kotlin/androidx/lifecycle/LiveData?hl=zh-cn)、[ViewModel](https://developer.android.com/reference/kotlin/androidx/lifecycle/ViewModel?hl=zh-cn)、[视图绑定](https://developer.android.com/topic/libraries/view-binding?hl=zh-cn)，以及[应用架构指南](https://developer.android.com/topic/libraries/architecture/guide.html?hl=zh-cn)中建议的架构。
+   熟悉[协程](https://kotlinlang.org/docs/reference/coroutines-overview.html)和 Kotlin [Flow](https://kotlinlang.org/docs/reference/coroutines/flow.html)。

有关架构组件的介绍，请查看[“Room with a View”Codelab](https://codelabs.developers.google.com/codelabs/android-room-with-a-view-kotlin/?hl=zh-cn)。有关 Flow 的说明，请查看[“带 Kotlin Flow 和 LiveData 的高级协程”Codelab](https://codelabs.developers.google.com/codelabs/advanced-kotlin-coroutines?hl=zh-cn)。

## 2\. 准备工作

在此步骤中，您将下载完整的 Codelab 代码，然后运行一个简单的示例应用。

为帮助您尽快入门，我们准备了一个入门级项目，您可以在此项目的基础上进行构建。

如果您已安装 git，只需运行以下命令即可。如需检查是否已安装 git，请在终端或命令行中输入 `git --version`，并验证其是否正确执行。

 git clone https://github.com/googlecodelabs/android-datastore

初始状态代码位于 `master` 分支中。解决方案代码位于 [`preferences_datastore`](https://github.com/googlecodelabs/android-datastore/tree/preferences_datastore) 分支中。

如果您未安装 git，可以点击下方按钮下载此 Codelab 的全部代码：

[下载源代码](https://github.com/googlecodelabs/android-datastore/archive/master.zip)

1.  解压缩代码，然后在 Android Studio Arctic Fox 中打开项目。
2.  在设备或模拟器上运行 **app** 运行配置。

![b3c0dfdb92dfed77.png](/images/dataStore_use/2.png)

应用运行并显示任务列表：

![d3972939a2de88ba.png](/images/dataStore_use/3.png)

## 3\. 项目概览

在应用中，您可以看到一个任务列表。每个任务都具有以下属性：名称、“已完成”状态、优先级和截止时间。

为简化我们需要使用的代码，应用仅允许您执行以下两项操作：

+   切换“显示已完成任务”的可见性，默认情况下，这些任务处于隐藏状态
+   按优先级和/或截止时间对任务排序

应用遵循[应用架构指南](https://developer.android.com/topic/libraries/architecture/guide.html?hl=zh-cn)中推荐的架构。每个软件包都包含以下内容：

**`data`**

+   `Task` 模型类。
+   `TasksRepository` 类 - 负责提供任务。为简单起见，该类会返回硬编码数据，并通过 `Flow` 提供该数据，以呈现更真实的场景。
+   `UserPreferencesRepository` 类 - 用于存储 `SortOrder`，定义为 `enum`。当前的**排序顺序根据枚举值名称以 `String` 的形式存储在 SharedPreferences** 中。该类提供了用于存储和获取排序顺序的同步方法。

**`ui`**

+   与使用 `RecyclerView` 显示 `Activity` 相关的类。
+   `TasksViewModel` 类负责界面逻辑。

`TasksViewModel` - 用于存储在构建以下要在界面中显示的数据时所需的全部元素：任务列表、`showCompleted` 和 `sortOrder` 标志，所有这些元素均封装在 `TasksUiModel` 对象中。每当其中某个值发生变化时，我们都必须重构一个新的 `TasksUiModel`。为此，我们需要组合以下 3 个元素：

+   `Flow<List<Task>>` - 检索自 `TasksRepository`。
+   `MutableStateFlow<Boolean>` - 存储最新的 `showCompleted` 标志，该标志仅保存在**内存中**。
+   `MutableStateFlow<SortOrder>` - 存储最新的 `sortOrder` 值。

为确保正确更新界面，仅在 Activity 启动时才公开 `LiveData<TasksUiModel>`。

我们的代码存在几个问题：

+   我们在初始化 `UserPreferencesRepository.sortOrder` 时阻断了磁盘 IO 上的界面线程。这可能会导致界面卡顿。
+   `showCompleted` 标志仅保存在内存中，因此每次用户打开应用时，该标志都会重置。与 `sortOrder` 一样，该标志在应用关闭后仍应保留。
+   我们目前使用 SharedPreferences 来永久保存数据，但我们在内存中保留了 `MutableStateFlow`，可以手动修改该值，以便获取有关更改的通知。如果在应用的其他地方修改了该值，则很容易发生错误。
+   在 `UserPreferencesRepository` 中，我们公开了两种更新排序顺序的方法：`enableSortByDeadline()` 和 `enableSortByPriority()`。这两种方法都依赖当前的排序顺序值，但如果在一个方法结束之前调用另一个方法，则最终值可能会出错。此外还需要注意，由于这些方法是在界面线程上调用的，因此它们可能会导致界面卡顿和严格模式违例。

我们一起来看看如何使用 DataStore 帮助我们解决这些问题。

## 4\. DataStore - 基础知识

您可能经常需要存储较小或简单的数据集。为此，您过去可能使用过 SharedPreferences，但此 API 也存在一系列缺点。Jetpack DataStore 库旨在解决这些问题，从而创建一个简单、安全性更高的异步 API 来存储数据。它提供 2 种不同的实现：

+   Preferences DataStore
+   Proto DataStore

<table class="vertical-rules"><tbody><tr><td colspan="1" rowspan="1"><p><strong>功能</strong></p></td><td colspan="1" rowspan="1"><p><strong>SharedPreferences</strong></p></td><td colspan="1" rowspan="1"><p><strong>PreferencesDataStore</strong></p></td><td colspan="1" rowspan="1"><p><strong>ProtoDataStore</strong></p></td></tr><tr><td colspan="1" rowspan="1"><p>异步 API</p></td><td colspan="1" rowspan="1"><p>✅（仅用于通过<a href="https://developer.android.com/reference/android/content/SharedPreferences.OnSharedPreferenceChangeListener?hl=zh-cn" target="_blank">监听器</a>读取已更改的值）</p></td><td colspan="1" rowspan="1"><p>✅（通过 <code translate="no" dir="ltr">Flow</code> 以及 RxJava 2 和 3 <code translate="no" dir="ltr">Flowable</code>）</p></td><td colspan="1" rowspan="1"><p>✅（通过 <code translate="no" dir="ltr">Flow</code> 以及 RxJava 2 和 3 <code translate="no" dir="ltr">Flowable</code>）</p></td></tr><tr><td colspan="1" rowspan="1"><p>同步 API</p></td><td colspan="1" rowspan="1"><p>✅（但无法在界面线程上安全调用）</p></td><td colspan="1" rowspan="1"><p>❌</p></td><td colspan="1" rowspan="1"><p>❌</p></td></tr><tr><td colspan="1" rowspan="1"><p>可在界面线程上安全调用</p></td><td colspan="1" rowspan="1"><p>❌1</p></td><td colspan="1" rowspan="1"><p>✅（这项工作已在后台移至 <code translate="no" dir="ltr">Dispatchers.IO</code>）</p></td><td colspan="1" rowspan="1"><p>✅（这项工作已在后台移至 <code translate="no" dir="ltr">Dispatchers.IO</code>）</p></td></tr><tr><td colspan="1" rowspan="1"><p>可以提示错误</p></td><td colspan="1" rowspan="1"><p>❌</p></td><td colspan="1" rowspan="1"><p>✅</p></td><td colspan="1" rowspan="1"><p>✅</p></td></tr><tr><td colspan="1" rowspan="1"><p>不受运行时异常影响</p></td><td colspan="1" rowspan="1"><p>❌2</p></td><td colspan="1" rowspan="1"><p>✅</p></td><td colspan="1" rowspan="1"><p>✅</p></td></tr><tr><td colspan="1" rowspan="1"><p>包含一个具有强一致性保证的事务性 API</p></td><td colspan="1" rowspan="1"><p>❌</p></td><td colspan="1" rowspan="1"><p>✅</p></td><td colspan="1" rowspan="1"><p>✅</p></td></tr><tr><td colspan="1" rowspan="1"><p>处理数据迁移</p></td><td colspan="1" rowspan="1"><p>❌</p></td><td colspan="1" rowspan="1"><p>✅</p></td><td colspan="1" rowspan="1"><p>✅</p></td></tr><tr><td colspan="1" rowspan="1"><p>类型安全</p></td><td colspan="1" rowspan="1"><p>❌</p></td><td colspan="1" rowspan="1"><p>❌</p></td><td colspan="1" rowspan="1"><p>✅ 使用<a href="https://developers.google.com/protocol-buffers?hl=zh-cn" target="_blank">协议缓冲区</a></p></td></tr></tbody></table>

1 SharedPreferences 有一个看上去可以在界面线程中安全调用的同步 API，但是该 API 实际上执行磁盘 I/O 操作。此外，`apply()` 会阻断 `fsync()` 上的界面线程。每次有服务启动或停止以及每次 activity 在应用中的任何地方启动或停止时，系统都会触发待处理的 `fsync()` 调用。界面线程在 `apply()` 调度的待处理 `fsync()` 调用上会被阻断，这通常会导致 [ANR](https://developer.android.com/topic/performance/vitals/anr?hl=zh-cn)。

2 SharedPreferences 会将解析错误作为运行时异常抛出。

### Preferences DataStore 与 Proto DataStore

虽然 Preferences DataStore 和 Proto DataStore 都允许保存数据，但它们保存数据的方式不同：

+   与 SharedPreferences 一样，**Preferences DataStore** 根据键访问数据，而无需事先定义架构。
+   **Proto DataStore** 使用[协议缓冲区](https://developers.google.com/protocol-buffers?hl=zh-cn)来定义架构。使用协议缓冲区可**持久保留强类型数据**。与 XML 和其他类似的数据格式相比，协议缓冲区速度更快、规格更小、使用更简单，并且更清楚明了。虽然使用 Proto DataStore 需要学习新的序列化机制，但我们认为 Proto DataStore 有着强大的类型优势，值得学习。

### **Room 与 DataStore**

如果您需要实现部分更新、引用完整性或大型/复杂数据集，您应考虑使用 Room，而不是 DataStore。DataStore 非常适合小型或简单的数据集，但不支持部分更新或参照完整性。

## 5\. Preferences DataStore 概览

Preference DataStore API 类似于 SharedPreferences，但与后者相比存在一些显著差异：

+   以事务方式处理数据更新
+   公开表示当前数据状态的 Flow
+   不提供存留数据的方法（`apply()`、`commit()`）
+   不返回对其内部状态的可变引用
+   通过类型化键提供类似于 `Map` 和 `MutableMap` 的 API

接下来我们看看如何将其添加到项目中，并将 SharedPreferences 迁移到 DataStore。

## **添加依赖项**

更新 build.gradle 文件以添加以下 Preference DataStore 依赖项：

```auto
implementation "androidx.datastore:datastore-preferences:1.0.0"
```

## 6\. 在 Preferences DataStore 中存留数据

尽管 `showCompleted` 标志和 `sortOrder` 标志都是用户偏好设置，但目前两者以两种不同的对象来表示。因此，我们的一个目标是在 `UserPreferences` 类中统一这两个标志，并使用 DataStore 将其存储在 `UserPreferencesRepository` 中。目前，`showCompleted` 标志保存在内存的 `TasksViewModel` 中。

首先，在 `UserPreferencesRepository` 中创建 `UserPreferences` 数据类。目前，它应该只有一个字段：`showCompleted`。稍后我们将添加排序顺序。

```auto
data class UserPreferences(val showCompleted: Boolean)
```

## **创建 DataStore**

为了创建 DataStore 实例，我们使用 `preferencesDataStore` 委托，并将 `Context` 作为接收器。为简单起见，在此 Codelab 中，我们在 `TasksActivity` 中执行该操作：

```auto
private const val USER_PREFERENCES_NAME = "user_preferences"

private val Context.dataStore by preferencesDataStore(
    name = USER_PREFERENCES_NAME
)
```

`preferencesDataStore` 委托可确保我们有一个 DataStore 实例在应用中具有该名称。目前，`UserPreferencesRepository` 是作为单例实现的，因为它用于存储 `sortOrderFlow` 并避免将它与 `TasksActivity` 的生命周期相关联。由于 `UserPreferenceRepository` 只会处理来自 Datastore 的数据，而不会创建和存储任何新对象，因此我们已经可以移除单例实现：

+   移除 `companion object`
+   将 `constructor` 设为公开

`UserPreferencesRepository` 应该获取一个 `DataStore` 实例作为构造函数参数。现在，我们可以将 `Context` 保留为参数，因为 SharedPreferences 需要用到它，但稍后我们会将其移除。

```auto
class UserPreferencesRepository(
    private val userPreferencesStore: DataStore<UserPreferences>,
    context: Context
) { ... }
```

下面，我们在 `TasksActivity` 中更新 `UserPreferencesRepository` 的构造，并传入 `dataStore`：

```auto
viewModel = ViewModelProvider(
    this,
    TasksViewModelFactory(
        TasksRepository,
        UserPreferencesRepository(dataStore, this)
    )
).get(TasksViewModel::class.java)
```

## **从 Preferences DataStore 读取数据**

Preferences DataStore 公开 `Flow<Preferences>` 中存储的数据，每当偏好设置发生变化时，Flow<Preferences> 就会发出该数据。我们不希望公开整个 `Preferences` 对象，而是要公开 `UserPreferences` 对象。为此，我们必须映射 `Flow<Preferences>`，根据键获取感兴趣的布尔值，并构造一个 `UserPreferences` 对象。

因此，我们首先需要定义 `show_completed` 键，这是一个 `booleanPreferencesKey` 值，我们将其声明为私有 `PreferencesKeys` 对象中的成员。

```auto
private object PreferencesKeys {
  val SHOW_COMPLETED = booleanPreferencesKey("show_completed")
}
```

我们将公开一个基于 `dataStore.data: Flow<Preferences>` 构造的 `userPreferencesFlow: Flow<UserPreferences>`，然后将其映射，以检索正确的偏好设置：

```auto
val userPreferencesFlow: Flow<UserPreferences> = dataStore.data
    .map { preferences ->
        // Get our show completed value, defaulting to false if not set:
        val showCompleted = preferences[PreferencesKeys.SHOW_COMPLETED]?: false
        UserPreferences(showCompleted)
    }
```

### **处理读取数据时的异常**

当 DataStore 从文件读取数据时，如果读取数据期间出现错误，系统会抛出 `IOExceptions`。我们可以通过以下方式处理这些事务：在 `map()` 之前使用 [`catch()`](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/catch.html) Flow 运算符，并且在抛出的异常是 `IOException` 时发出 `emptyPreferences()`。如果出现其他类型的异常，最好重新抛出该异常。

```auto
val userPreferencesFlow: Flow<UserPreferences> = dataStore.data
    .catch { exception ->
        // dataStore.data throws an IOException when an error is encountered when reading data
        if (exception is IOException) {
            emit(emptyPreferences())
        } else {
            throw exception
        }
    }.map { preferences ->
        // Get our show completed value, defaulting to false if not set:
        val showCompleted = preferences[PreferencesKeys.SHOW_COMPLETED]?: false
        UserPreferences(showCompleted)
    }
```

## **将数据写入 Preferences DataStore**

如需写入数据，DataStore 提供挂起 `DataStore.edit(transform: suspend (MutablePreferences) -> Unit)` 函数，该函数接受 `transform` 块，让我们能够以事务方式更新 DataStore 中的状态。

传递给转换块的 `MutablePreferences` 将保持以前所有运行编辑的最新状态。在 `transform` 完成后且 `edit` 完成之前，对 `transform` 块中 `MutablePreferences` 的所有更改都将应用于磁盘。在 `MutablePreferences` 中设置一个值会使所有其他偏好设置保持不变。

注意：请勿尝试修改转换块之外的 `MutablePreferences`。

现在我们来创建一个挂起函数，以便我们能够更新 `UserPreferences` 的 `showCompleted` 属性，此函数称为 `updateShowCompleted()`，用于调用 `dataStore.edit()` 并设置新值：

```auto
suspend fun updateShowCompleted(showCompleted: Boolean) {
    dataStore.edit { preferences ->
        preferences[PreferencesKeys.SHOW_COMPLETED] = showCompleted
    }
}
```

如果在读取或写入磁盘时发生错误，`edit()` 可能会抛出 `IOException`。如果转换块中出现任何其他错误，`edit()` 将抛出异常。

此时，应用可以成功编译，但是我们刚刚在 `UserPreferencesRepository` 中创建的功能不会被使用。

## 7\. 从 SharedPreferences 迁移到 Preferences DataStore

排序顺序保存在 SharedPreferences 中。让我们将其迁移到 DataStore 中。为此，让我们先更新 `UserPreferences` 以存储排序顺序：

```auto
data class UserPreferences(
    val showCompleted: Boolean,
    val sortOrder: SortOrder
)
```

## **从 SharedPreferences 迁移**

为了能够将排序顺序迁移到 DataStore，我们需要更新 DataStore 构建器以向迁移列表传入 `SharedPreferencesMigration`。DataStore 能够自动从 `SharedPreferences` 迁移到 DataStore。迁移需在 DataStore 中的任何数据访问操作可发生之前运行。这意味着，必须在 `DataStore.data` 发出任何值之前和 `DataStore.edit()` 可以更新数据之前，成功完成迁移。

注意：由于键只能从 SharedPreferences 迁移一次，因此在代码迁移到 DataStore 之后，您应停止使用旧 SharedPreferences。

首先，我们在 `TasksActivity` 中更新 Datastore 创建代码：

```auto
private const val USER_PREFERENCES_NAME = "user_preferences"

private val Context.dataStore by preferencesDataStore(
    name = USER_PREFERENCES_NAME,
    produceMigrations = { context ->
        // Since we're migrating from SharedPreferences, add a migration based on the
        // SharedPreferences name
        listOf(SharedPreferencesMigration(context, USER_PREFERENCES_NAME))
    }
)
```

然后，将 `sort_order` 添加到我们的 `PreferencesKeys`：

```auto
private object PreferencesKeys {
    ...
    // Note: this has the the same name that we used with SharedPreferences.
    val SORT_ORDER = stringPreferencesKey("sort_order")
}
```

所有键都将迁移到我们的 DataStore，并从用户偏好设置 SharedPreferences 中删除。现在，我们可以从 `Preferences` 获取并更新基于 `SORT_ORDER` 键的 `SortOrder`。

## **从 DataStore 中读取排序顺序**

现在，让我们更新 `userPreferencesFlow` 以同时检索 `map()` 转换中的排序顺序：

```auto
val userPreferencesFlow: Flow<UserPreferences> = dataStore.data
    .catch { exception ->
        if (exception is IOException) {
            emit(emptyPreferences())
        } else {
            throw exception
        }
    }.map { preferences ->
        // Get the sort order from preferences and convert it to a [SortOrder] object
        val sortOrder =
            SortOrder.valueOf(
                preferences[PreferencesKeys.SORT_ORDER] ?: SortOrder.NONE.name)

        // Get our show completed value, defaulting to false if not set:
        val showCompleted = preferences[PreferencesKeys.SHOW_COMPLETED] ?: false
        UserPreferences(showCompleted, sortOrder)
    }
```

## **将排序顺序保存到 DataStore**

目前，`UserPreferencesRepository` 仅公开一种用于设置排序顺序标志的同步方法，并且存在并发问题。我们公开了两种更新排序顺序的方法：`enableSortByDeadline()` 和 `enableSortByPriority()`；这两种方法都依赖当前的排序顺序值，但如果在一个方法结束之前调用另一个方法，则最终值可能会出错。

由于 Datastore 保证以事务方式进行数据更新，所以我们不会再遇到这个问题。接下来，让我们一起执行以下更改：

+   将 `enableSortByDeadline()` 和 `enableSortByPriority()` 更新为使用 `dataStore.edit()` 的 `suspend` 函数。
+   在 `edit()` 的转换块中，我们将从 Preferences 参数中获取 `currentOrder`，而不是从 `_sortOrderFlow` 字段中进行检索。
+   我们可以直接在偏好设置中更新排序顺序，而不是调用 `updateSortOrder(newSortOrder)`。

具体实现如下所示。

```auto
suspend fun enableSortByDeadline(enable: Boolean) {
    // edit handles data transactionally, ensuring that if the sort is updated at the same
    // time from another thread, we won't have conflicts
    dataStore.edit { preferences ->
        // Get the current SortOrder as an enum
        val currentOrder = SortOrder.valueOf(
            preferences[PreferencesKeys.SORT_ORDER] ?: SortOrder.NONE.name
        )

        val newSortOrder =
            if (enable) {
                if (currentOrder == SortOrder.BY_PRIORITY) {
                    SortOrder.BY_DEADLINE_AND_PRIORITY
                } else {
                    SortOrder.BY_DEADLINE
                }
            } else {
                if (currentOrder == SortOrder.BY_DEADLINE_AND_PRIORITY) {
                    SortOrder.BY_PRIORITY
                } else {
                    SortOrder.NONE
                }
            }
        preferences[PreferencesKeys.SORT_ORDER] = newSortOrder.name
    }
}

suspend fun enableSortByPriority(enable: Boolean) {
    // edit handles data transactionally, ensuring that if the sort is updated at the same
    // time from another thread, we won't have conflicts
    dataStore.edit { preferences ->
        // Get the current SortOrder as an enum
        val currentOrder = SortOrder.valueOf(
            preferences[PreferencesKeys.SORT_ORDER] ?: SortOrder.NONE.name
        )

        val newSortOrder =
            if (enable) {
                if (currentOrder == SortOrder.BY_DEADLINE) {
                    SortOrder.BY_DEADLINE_AND_PRIORITY
                } else {
                    SortOrder.BY_PRIORITY
                }
            } else {
                if (currentOrder == SortOrder.BY_DEADLINE_AND_PRIORITY) {
                    SortOrder.BY_DEADLINE
                } else {
                    SortOrder.NONE
                }
            }
        preferences[PreferencesKeys.SORT_ORDER] = newSortOrder.name
    }
}
```

现在，您可以移除 `context` 构造函数参数和使用的所有 SharedPreferences。

## 8\. 更新 TasksViewModel 以使用 UserPreferencesRepository

现在，`UserPreferencesRepository` 在 DataStore 中存储了 `show_completed` 和 `sort_order` 标志，并提供了 `Flow<UserPreferences>`。接下来，让我们更新并使用 `TasksViewModel`。

移除 `showCompletedFlow` 和 `sortOrderFlow`，创建一个名为 `userPreferencesFlow` 的值并用 `userPreferencesRepository.userPreferencesFlow` 对该值进行初始化：

```auto
private val userPreferencesFlow = userPreferencesRepository.userPreferencesFlow
```

在 `tasksUiModelFlow` 创建中，将 `showCompletedFlow` 和 `sortOrderFlow` 替换为 `userPreferencesFlow`。请相应地替换参数。

调用 `filterSortTasks` 时，传入 `userPreferences` 的 `showCompleted` 和 `sortOrder`。您的代码应如下所示：

```auto
private val tasksUiModelFlow = combine(
        repository.tasks,
        userPreferencesFlow
    ) { tasks: List<Task>, userPreferences: UserPreferences ->
        return@combine TasksUiModel(
            tasks = filterSortTasks(
                tasks,
                userPreferences.showCompleted,
                userPreferences.sortOrder
            ),
            showCompleted = userPreferences.showCompleted,
            sortOrder = userPreferences.sortOrder
        )
    }
```

`showCompletedTasks()` 函数现在应已更新为调用 `userPreferencesRepository.updateShowCompleted()`。由于这是一个挂起函数，因此请在 `viewModelScope` 中创建一个新的协程：

```auto
fun showCompletedTasks(show: Boolean) {
    viewModelScope.launch {
        userPreferencesRepository.updateShowCompleted(show)
    }
}
```

`userPreferencesRepository` 函数、`enableSortByDeadline()` 和 `enableSortByPriority()` 现在属于挂起函数，因此还应在 `viewModelScope` 中启动的新协程中调用它们：

```auto
fun enableSortByDeadline(enable: Boolean) {
    viewModelScope.launch {
       userPreferencesRepository.enableSortByDeadline(enable)
    }
}

fun enableSortByPriority(enable: Boolean) {
    viewModelScope.launch {
        userPreferencesRepository.enableSortByPriority(enable)
    }
}
```

## 清理 `UserPreferencesRepository`

现在我们来移除已经不需要的字段和方法。您应能删除以下内容：

+   `_sortOrderFlow`
+   `sortOrderFlow`
+   `updateSortOrder()`
+   `private val sortOrder: SortOrder`

我们的应用现在应能成功进行编译。运行一下，看看 `show_completed` 和 `sort_order` 标志是否能成功保存。

查看 Codelab 代码库的 `preferences_datastore` 分支，并与您的更改进行比较。

## 9\. 小结

现在，您已迁移到 Preferences DataStore，下面让我们总结一下所学的内容：

+   SharedPreferences 存在一些缺点：包括看上去可以在界面线程中安全调用的同步 API，没有发出错误信号的机制，缺少事务性 API 等。
+   DataStore 可替代 SharedPreferences，解决 API 的大部分问题。
+   DataStore 有一个使用 Kotlin 协程和 Flow 的完全异步 API，可以处理数据迁移，保证数据一致性并处理数据损坏问题。