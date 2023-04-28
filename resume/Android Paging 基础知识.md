## 1\. 简介

## **学习内容**

+   Paging 库有哪些主要组件。
+   如何将 Paging 库添加到项目中。

## **构建内容**

在此 Codelab 中，您将从一个示例应用开始构建，该应用中会显示一个报道列表。该列表是静态的，其中包含 500 篇报道，并且所有报道都保存在手机内存中：

![paging.png](./images/paging_basic/paging.png)

学完此 Codelab 后，您将：

+   …了解分页的概念。
+   …了解 Paging 库的核心组件。
+   …了解如何使用 Paging 库实现分页。

完成后，您将得到一款具备以下特征的应用：

+   …成功实现分页。
+   …当提取更多数据时，能够有效地向用户传达相关信息。

以下是最终界面的简要预览：

![paging_preview.gif](./images/paging_basic/paging_preview.gif)

## **所需条件**

+   [最新版本的 Android Studio](https://developer.android.com/studio/index.html?hl=zh-cn)。

## **建议条件**

+   熟悉以下架构组件：[ViewModel](https://developer.android.com/reference/androidx/lifecycle/ViewModel?hl=zh-cn)、[View Binding](https://developer.android.com/topic/libraries/view-binding?hl=zh-cn)，以及[应用架构指南](https://developer.android.com/topic/libraries/architecture/guide.html?hl=zh-cn)中建议的架构。有关架构组件的介绍，请查看[“带 View 的 Room”Codelab](https://codelabs.developers.google.com/codelabs/android-room-with-a-view/index.html?index=..%2F..%2Findex&hl=zh-cn#0)。
+   熟悉[协程](https://kotlinlang.org/docs/reference/coroutines-overview.html)和 Kotlin [Flow](https://kotlinlang.org/docs/reference/coroutines/flow.html)。有关 Flow 的说明，请查看[“带 Kotlin Flow 和 LiveData 的高级协程”Codelab](https://developer.android.com/codelabs/advanced-kotlin-coroutines?hl=zh-cn)。

## 2\. 设置您的环境

在此步骤中，您将下载完整的 Codelab 代码，然后运行一个简单的示例应用。

为帮助您尽快入门，我们准备了一个入门级项目，您可以在此项目的基础上进行构建。

如果您已安装 git，只需运行以下命令即可。如需检查是否已安装 git，请在终端或命令行中输入 `git --version`，并验证其是否正确执行。

 git clone https://github.com/googlecodelabs/android-paging

如果您未安装 git，可以点击下载此 Codelab 的全部代码：[下载源码](https://github.com/googlecodelabs/android-paging/archive/main.zip)

代码会整理到两个文件夹中，即 `basic` 和 `advanced`。对于此 Codelab，我们只关注 `basic` 文件夹。

`basic` 文件夹中还有另外两个文件夹：`start` 和 `end`。我们将开始处理 `start` 文件夹中的代码，在 Codelab 结束时，`start` 文件夹中的代码应该与 `end` 文件夹中的代码相同。

1.  在 Android Studio 中的 `basic/start` 目录中打开项目。
2.  在设备或模拟器上运行 **`app`** 运行配置。

![app_config.png](./images/paging_basic/app_config.png)

我们应该可以看到一个报道列表！滚动到列表底部，确认列表是静态的，换句话说，当我们到达列表末尾时，系统不会提取更多项。滚动回顶部，验证是否所有项仍在列表中。

## 3\. 分页简介

要向用户显示信息，最常用的方式是使用列表。不过，有时这些列表只会为用户提供一个小窗口，供他们查看其中的所有内容。当用户滚动浏览可用信息时，他们往往会想提取更多数据来补充已经看到的信息。每次提取数据时，都必须高效且流畅，以免增量加载对用户体验造成负面影响。增量加载对性能也有好处，因为应用不需要一次性在内存中保存大量数据。

这种增量提取信息的过程称为“分页”，其中每个“页面”对应一个要提取的数据块。如需请求页面，要分页的数据源通常需要进行查询，从而定义所需信息。此 Codelab 的其余部分将介绍 Paging 库，并展示它将如何帮助您快速、高效地在应用中实现分页。

## Paging 库的核心组件

Paging 库的核心组件如下：

+   **`PagingSource`** - 用于为特定页面查询加载数据块的基类。它是数据层的一部分，通常从 [`DataSource`](https://developer.android.com/jetpack/guide/data-layer?hl=zh-cn#architecture) 类公开，随后由 `Repository` 公开以在 `ViewModel` 中使用。
+   **`PagingConfig`** - 用于定义确定分页行为的形参的类。这包括页面大小、是否启用占位符等。
+   **`Pager`** - 负责生成 `PagingData` 流的类。这取决于 `PagingSource`，因此应在 `ViewModel` 中创建。
+   **`PagingData`** - 用于存储分页数据的容器。每次数据刷新都会有一个相应的单独 `PagingData` 发送，并由其自己的 `PagingSource` 提供支持。
+   **`PagingDataAdapter`** - 用于在 `RecyclerView` 中呈现 `PagingData` 的 `RecyclerView.Adapter` 子类。`PagingDataAdapter` 可以使用工厂方法连接到 Kotlin `Flow`、`LiveData`、RxJava `Flowable`、RxJava `Observable` 甚至静态列表。`PagingDataAdapter` 会监听内部 `PagingData` 加载事件，并在网页加载时高效更新界面。

![paging_source.jpeg](./images/paging_basic/paging_source.jpeg)

在以下部分中，您将实现上述每个组件的示例。

## 4\. 项目概览

在当前形式下，应用显示的是一个静态报道列表。每篇报道都有一个标题、说明和创建日期。静态列表适用于项的数量较少的情况，但随着数据集的扩大，这种列表不能很好地进行扩展。为解决此问题，我们将使用 Paging 库来实现分页，但我们首先要介绍一下应用中已有的组件。

应用遵循[应用架构指南](https://developer.android.com/topic/libraries/architecture/guide.html?hl=zh-cn)中推荐的架构。每个软件包都包含以下内容：

**数据层**：

+   `ArticleRepository`：负责提供报道列表并将其保存在内存中。
+   `Article`：代表数据模型的类，它代表从数据层提取的信息。

**界面层**：

+   `Activity`、`RecyclerView.Adapter` 和 `RecyclerView.ViewHolder`：负责在界面中显示列表的类。
+   `ViewModel`：负责创建界面需要显示的状态的状态容器。

代码库使用 `articleStream` 字段在 `Flow` 中公开所有报道。进而，界面层中的 `ArticleViewModel` 会读取相应信息，然后让其做好准备，以供 `ArticleActivity` 中的界面通过 `state` 字段（即 [`StateFlow`](https://medium.com/androiddevelopers/things-to-know-about-flows-sharein-and-statein-operators-20e6ccb2bc74)）使用。

在代码库中将报道作为 `Flow` 公开后，当所呈现的报道随着时间推移而发生更改时，代码库可以对报道进行更新。例如，如果报道的标题发生更改，系统可以轻松地将相应更改传达给 `articleStream` 的收集器。在 `ViewModel` 中对界面状态使用 `StateFlow` 可以确保，即使我们停止收集界面状态（例如，在配置更改期间重新创建 `Activity` 时），当我们重新开始收集界面状态时，依然可以从上次离开的地方直接恢复操作。

如前所述，代码库中的当前 `articleStream` 仅呈现当天的新闻。这对有些用户来说可能已经足够了，而其他人在滚动浏览当天提供的所有报道后，可能会想查看旧报道。用户的这种期望使得报道非常适合通过分页进行显示。我们应该通过报道探索分页的其他原因如下：

+   `ViewModel` 将所有加载到内存中的项保存在 `items` `StateFlow` 中。当数据集变得非常大时，这是一个主要问题，因为这可能会影响性能。
+   报道列表越大，当其中一篇或多篇报道发生更改时，更新相应报道的成本就越高。

Paging 库可帮助您解决这些问题，同时还会提供一致的 API，用于在应用中增量提取数据（分页）。

## 5\. 定义数据源

在实现分页时，我们需要确保满足以下条件：

+   正确处理来自界面的数据请求，确保不会针对同一查询同时触发多个请求。
+   在内存中保留合理数量的检索数据。
+   触发提取更多数据的请求，对我们已经获取的数据进行补充。

我们可以通过 `PagingSource` 实现所有这些目的。`PagingSource` 会指定如何以增量数据块的形式来检索数据，从而定义数据源。然后，`PagingData` 对象会从 `PagingSource` 提取数据，响应用户在 `RecyclerView` 中滚动生成的加载提示。

我们的 `PagingSource` 将会加载报道。在 `data/Article.kt` 中，您会发现定义如下所示的模型`:`

```auto
data class Article(
    val id: Int,
    val title: String,
    val description: String,
    val created: LocalDateTime,
)
```

为了构建 `PagingSource`，您需要定义以下内容：

+   **分页键的类型** - 用于请求更多数据的网页查询类型的定义。在本例中，我们会提取特定报道 ID 之后或之前的报道，因为这两个 ID 肯定是有序且递增的。
+   **已加载数据的类型** - 每个页面都返回一个报道 `List`，因此类型为 `Article`。
+   **从何处检索数据** - 这通常是指数据库、网络资源或分页数据的任何其他来源。不过，在此 Codelab 中，我们将使用本地生成的数据。

在 `data` 软件包中，我们将在名为 `ArticlePagingSource.kt` 的新文件中创建一个 `PagingSource` 实现：

```auto
package com.example.android.codelabs.paging.data

import androidx.paging.PagingSource
import androidx.paging.PagingState

class ArticlePagingSource : PagingSource<Int, Article>() {
    override suspend fun load(params: LoadParams<Int>): LoadResult<Int, Article> {
        TODO("Not yet implemented")
    }
   override fun getRefreshKey(state: PagingState<Int, Article>): Int? {
        TODO("Not yet implemented")
    }
}
```

`PagingSource` 需要我们实现两个函数：`load()` 和 `getRefreshKey()`。

Paging 库将调用 `load()` 函数，以异步方式提取更多数据，用于在用户滚动过程中显示。`LoadParams` 对象保存有与加载操作相关的信息，包括以下信息：

+   **要加载的页面的键** - 如果这是第一次调用 `load()`，`LoadParams.key` 将为 `null`。在这种情况下，必须定义初始页面键。对于我们的项目，我们将报道 ID 用作键。此外，我们还要在初始页面键的 `ArticlePagingSource` 文件顶部添加一个为 `0` 的 `STARTING_KEY` 常量。
+   **加载大小** - 请求加载内容的数量。

`load()` 函数会返回一个 `LoadResult`。`LoadResult` 可以是以下类型之一：

+   `LoadResult.Page`（如果结果返回成功）。
+   `LoadResult.Error`（如果发生错误）。
+   `LoadResult.Invalid`（如果 `PagingSource` 因无法再保证其结果的完整性而应失效）。

`LoadResult.Page` 有三个必需的实参：

+   `data`：所提取的项的 `List`。
+   `prevKey`：如果 `load()` 方法需要提取先于当前页面显示的项，它会使用这个键。
+   `nextKey`：如果 `load()` 方法需要提取晚于当前页面显示的项，它会使用这个键。

…以及两个可选实参：

+   `itemsBefore`：要在加载的数据前面显示的占位符的数量。
+   `itemsAfter`：要在加载的数据后面显示的占位符的数量。

我们的加载键是 `Article.id` 字段。我们可以将其用作键，因为每增加一篇报道，`Article` ID 就会加 1；也就是说，报道 ID 是单调递增的整数。

如果相应方向没有更多数据要加载，则 `nextKey` 或 `prevKey` 为 `null`。在本例中，对于 `prevKey`：

+   如果 `startKey` 与 `STARTING_KEY` 相同，我们将返回 null，因为我们无法在此键后面加载更多项。
+   否则，我们会获取列表中的第一个项并在它后面加载 `LoadParams.loadSize`，以确保永远不会返回小于 `STARTING_KEY` 的键。为此，我们要定义 `ensureValidKey()` 方法。

添加以下函数以检查分页键是否有效：

```auto
class ArticlePagingSource : PagingSource<Int, Article>() {
   ...
   /**
     * Makes sure the paging key is never less than [STARTING_KEY]
     */
    private fun ensureValidKey(key: Int) = max(STARTING_KEY, key)
}
```

对于 `nextKey`：

+   由于我们支持加载无限的项，因此我们要传入 `range.last + 1`。

此外，由于每篇报道都有一个 `created` 字段，因此我们还需要为该字段生成一个值。将以下代码添加到文件顶部：

```auto
private val firstArticleCreatedTime = LocalDateTime.now()

class ArticlePagingSource : PagingSource<Int, Article>() {
   ...
}
```

有了上述所有代码，我们现在就可以实现 `load()` 函数了：

```auto
import kotlin.math.max
...

private val firstArticleCreatedTime = LocalDateTime.now()

class ArticlePagingSource : PagingSource<Int, Article>() {
   override suspend fun load(params: LoadParams<Int>): LoadResult<Int, Article> {
        // Start paging with the STARTING_KEY if this is the first load
        val start = params.key ?: STARTING_KEY
        // Load as many items as hinted by params.loadSize
        val range = start.until(start + params.loadSize)

        return LoadResult.Page(
            data = range.map { number ->
                Article(
                    // Generate consecutive increasing numbers as the article id
                    id = number,
                    title = "Article $number",
                    description = "This describes article $number",
                    created = firstArticleCreatedTime.minusDays(number.toLong())
                )
            },

            // Make sure we don't try to load items behind the STARTING_KEY
            prevKey = when (start) {
                STARTING_KEY -> null
                else -> ensureValidKey(key = range.first - params.loadSize)
            },
            nextKey = range.last + 1
        )
    }

    ...
}
```

接下来，我们需要实现 `getRefreshKey()`。当 Paging 库因其后备 `PagingSource` 中的数据发生更改而需要重新加载界面项时，系统会调用该方法。这种 `PagingSource` 底层数据发生更改且需要在界面中进行更新的情况称为“失效”。失效后，Paging 库会创建一个新的 `PagingSource` 来重新加载数据，并通过发出新的 `PagingData` 来通知界面。我们将在后面的部分中详细了解失效。

从新的 `PagingSource` 加载时，系统会调用 `getRefreshKey()` 来提供新 `PagingSource` 应开始加载的键，从而确保用户在刷新后不会丢失其在列表中的当前位置。

以下两种原因之一会导致 Paging 库中发生失效：

+   您对 `PagingAdapter` 调用了 `refresh()`。
+   您对 `PagingSource` 调用了 `invalidate()`。

返回的键（本例中为 `Int`）将通过 `LoadParams` 实参传递到新 `PagingSource` 中 `load()` 方法的下一次调用。为防止项在失效后跳动，我们需要确保返回的键会加载足够的项来填充屏幕。这样可提高新的一组项中包含失效数据中存在的项的可能性，从而有助于保持当前的滚动位置。我们来看一下我们应用中的相应实现：

```auto
   // The refresh key is used for the initial load of the next PagingSource, after invalidation
   override fun getRefreshKey(state: PagingState<Int, Article>): Int? {
        // In our case we grab the item closest to the anchor position
        // then return its id - (state.config.pageSize / 2) as a buffer
        val anchorPosition = state.anchorPosition ?: return null
        val article = state.closestItemToPosition(anchorPosition) ?: return null
        return ensureValidKey(key = article.id - (state.config.pageSize / 2))
    }
```

在上面的代码段中，我们使用了 `PagingState.anchorPosition`。如果您好奇 Paging 库是如何知道要提取更多项的，这就是线索！当界面尝试从 `PagingData` 读取项时，它会尝试在特定索引处读取。如果读取了数据，相应数据会显示在界面中。不过，如果没有数据，Paging 库就会知道自己需要提取数据以满足失败的读取请求。读取时成功提取数据的最后一个索引是 `anchorPosition`。

刷新时，我们会获取最接近 `anchorPosition` 的 `Article` 键，并将其用作加载键。这样，当我们从新的 `PagingSource` 再次开始加载时，获取一组项就会包含已加载的项，从而确保流畅且一致的用户体验。

至此，您已经完全定义了 `PagingSource`。下一步是将其连接到界面。

## 6\. 为界面生成 PagingData

在当前实现中，我们在 `ArticleRepository` 中使用 `Flow<List<Article>>` 将加载的数据公开给 `ViewModel`。反过来，`ViewModel` 会使用 `stateIn` 运算符保持始终可用的数据状态，以便向界面进行公开。

使用 Paging 库时，我们将改为从 `ViewModel` 公开 `Flow<PagingData<Article>>`。`PagingData` 是一种类型，用于封装我们已加载的数据，并帮助 Paging 库决定何时提取更多数据，还可确保我们不会对同一页面提出两次请求。

为了构建 `PagingData`，我们将使用 `Pager` 类中的几种不同的构建器方法之一，具体取决于我们想要使用哪个 API 将 `PagingData` 传递到应用的其他层：

+   Kotlin `Flow` - 使用 `Pager.flow`。
+   `LiveData` - 使用 `Pager.liveData`.
+   RxJava `Flowable` - 使用 `Pager.flowable`。
+   RxJava `Observable` - 使用 `Pager.observable`。

我们在应用中已使用了 `Flow`，故将继续使用此方法。但不是使用 `Flow<List<Article>>`，而是用 `Flow<PagingData<Article>>`。

无论您使用哪种 `PagingData` 构建器，都必须传递以下参数：

+   **`PagingConfig`**。该类用于设置关于如何从 `PagingSource` 加载内容的选项，例如提前多久加载、初始加载请求的大小，等等。您必须定义的唯一必需形参是页面大小，即应在每个页面中加载的项数。默认情况下，Paging 会将您加载的所有页面保存在内存中。为确保系统在用户滚动时不会浪费内存，请在 `PagingConfig` 中设置 `maxSize` 形参。默认情况下，如果 Paging 可以统计未加载项的数量以及`enablePlaceholders` 配置标志为 `true`，那么 Paging 将返回 null 作为尚未加载内容的占位符。这样，您就可以在适配器中显示占位符视图。为了简化此 Codelab 中的工作，我们通过传递 `enablePlaceholders = false` 停用占位符。
+   一个**函数，用于定义如何创建** `PagingSource`。在本例中，我们将创建 `ArticlePagingSource`，因此我们需要用一个函数来告知 Paging 库该如何操作。

接下来，我们来修改 `ArticleRepository`！

## **更新** **`ArticleRepository`**

+   删除 `articlesStream` 字段。
+   添加一个名为 `articlePagingSource()` 的方法，用于返回我们刚刚创建的 `ArticlePagingSource`。

```auto
class ArticleRepository) {

    fun articlePagingSource() = ArticlePagingSource()
}
```

## **清理** **`ArticleRepository`**

Paging 库可以为我们做很多事情：

+   处理内存缓存。
+   在接近列表末尾时请求数据。

这意味着，除了 `articlePagingSource()` 之外，`ArticleRepository` 中的所有其他对象均可移除。现在，您的 `ArticleRepository` 应如下所示：

```auto
package com.example.android.codelabs.paging.data

import androidx.paging.PagingSource

class ArticleRepository {
    fun articlePagingSource() = ArticlePagingSource()
}
```

现在，`ArticleViewModel` 中应包含编译错误。我们来看看需要做出哪些更改！

## 7\. 在 ViewModel 中请求并缓存 PagingData

在修正编译错误之前，我们先来看看 `ViewModel`。

```auto
class ArticleViewModel(...) : ViewModel() {

    val items: StateFlow<List<Article>> = ...
}
```

为了在 `ViewModel` 中集成 Paging 库，我们要将 `items` 的返回值类型从 `StateFlow<List<Article>>` 更改为 `Flow<PagingData<Article>>`。为此，首先要在文件顶部添加一个名为 `ITEMS_PER_PAGE` 的专用常量：

```auto
private const val ITEMS_PER_PAGE = 50

class ArticleViewModel {
    ...
}
```

接下来，我们将 `items` 更新为 `Pager` 实例的输出结果。为此，我们要向 `Pager` 传递两个形参：

+   一个 `PagingConfig`，其 `pageSize` 为 `ITEMS_PER_PAGE` 且已停用占位符
+   一个 `PagingSourceFactory`，用于提供我们刚刚创建的 `ArticlePagingSource` 的实例

```auto
class ArticleViewModel(...) : ViewModel() {

   val items: Flow<PagingData<Article>> = Pager(
        config = PagingConfig(pageSize = ITEMS_PER_PAGE, enablePlaceholders = false),
        pagingSourceFactory = { repository.articlePagingSource() }
    )
        .flow
        ...
}
```

接下来，为了在配置或导航发生更改时保持分页状态，我们使用 `cachedIn()` 方法向其传递 [`androidx.lifecycle.viewModelScope`](https://developer.android.com/reference/kotlin/androidx/lifecycle/package-summary?hl=zh-cn#(androidx.lifecycle.ViewModel).viewModelScope:kotlinx.coroutines.CoroutineScope)。

完成上述更改后，我们的 `ViewModel` 应如下所示：

```auto
package com.example.android.codelabs.paging.ui

import androidx.lifecycle.ViewModel
import androidx.lifecycle.viewModelScope
import androidx.paging.Pager
import androidx.paging.PagingConfig
import androidx.paging.PagingData
import androidx.paging.cachedIn
import com.example.android.codelabs.paging.data.Article
import com.example.android.codelabs.paging.data.ArticleRepository
import com.example.android.codelabs.paging.data.ITEMS_PER_PAGE
import kotlinx.coroutines.flow.Flow

private const val ITEMS_PER_PAGE = 50

class ArticleViewModel(
    private val repository: ArticleRepository,
) : ViewModel() {

    val items: Flow<PagingData<Article>> = Pager(
        config = PagingConfig(pageSize = ITEMS_PER_PAGE, enablePlaceholders = false),
        pagingSourceFactory = { repository.articlePagingSource() }
    )
        .flow
        .cachedIn(viewModelScope)
}
```

关于 `PagingData` 还需要注意的一点是，它是一个独立的类型，包含要在 `RecyclerView` 中显示的数据的可变更新流。每次发出的 `PagingData` 都是完全独立的，并且如果后备 `PagingSource` 因底层数据集发生更改而失效，系统可以针对单个查询发出多个 `PagingData` 实例。因此，应独立于其他 `Flows` 公开 `PagingData` 的 `Flows`。

大功告成！现在，我们已在 `ViewModel` 实现分页功能！

## 8\. 将适配器与 PagingData 配合使用

如需将 `PagingData` 绑定到 `RecyclerView`，请使用 `PagingDataAdapter`。每当系统加载 `PagingData` 内容时，`PagingDataAdapter` 就会收到通知，然后它会通知 `RecyclerView` 进行更新。

## 更新 `ArticleAdapter` 以便与 `PagingData` 流配合使用：

+   目前，`ArticleAdapter` 实现的是 `ListAdapter`。请将其改为实现 `PagingDataAdapter`。类主体的其余部分保持不变：

```auto
import androidx.paging.PagingDataAdapter
...

class ArticleAdapter : PagingDataAdapter<Article, RepoViewHolder>(ARTICLE_DIFF_CALLBACK) {
// body is unchanged
}
```

到目前为止，我们已执行很多变更，现在只需再执行一步操作就可以运行应用了，那就是连接界面！

## 9\. 在界面中使用 PagingData

在当前实现中，我们有一个名为 `binding.setupScrollListener()` 的方法，该方法会在满足特定条件时调用 `ViewModel` 来加载更多数据。Paging 库会自动执行所有相关操作，因此我们可以删除该方法及其用法。

接下来，由于 `ArticleAdapter` 不再是 `ListAdapter`，而是 `PagingDataAdapter`，因此我们要进行以下两项细微更改：

+   我们将 `Flow` 中的终端运算符从 `ViewModel` 切换为 `collectLatest`，而不是 `collect`。
+   我们使用 `submitData()`（而非 `submitList()`）通知 `ArticleAdapter` 有更改。

我们对 `pagingData` `Flow` 使用 `collectLatest`，以便在发出新的 `pagingData` 实例时，取消收集之前发出的 `pagingData`。

完成这些更改后，`Activity` 应如下所示：

```auto
import kotlinx.coroutines.flow.collectLatest

class ArticleActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        val binding = ActivityArticlesBinding.inflate(layoutInflater)
        val view = binding.root
        setContentView(view)

        val viewModel by viewModels<ArticleViewModel>(
            factoryProducer = { Injection.provideViewModelFactory(owner = this) }
        )

        val items = viewModel.items
        val articleAdapter = ArticleAdapter()

        binding.bindAdapter(articleAdapter = articleAdapter)

        lifecycleScope.launch {
            repeatOnLifecycle(Lifecycle.State.STARTED) {
                items.collectLatest {
                    articleAdapter.submitData(it)
                }
            }
        }
    }
}

private fun ActivityArticlesBinding.bindAdapter(
    articleAdapter: ArticleAdapter
) {
    list.adapter = articleAdapter
    list.layoutManager = LinearLayoutManager(list.context)
    val decoration = DividerItemDecoration(list.context, DividerItemDecoration.VERTICAL)
    list.addItemDecoration(decoration)
}
```

现在，应用应能进行编译并运行。您已成功将应用迁移到 Paging 库！

![paging_data.gif](./images/paging_basic/paging_data.gif)

## 10\. 在界面中显示加载状态

当 Paging 库提取更多项以在界面中显示时，最佳实践是向用户指明更多数据即将显示。幸运的是，Paging 库提供了一种便捷的方式，让您能够使用 `CombinedLoadStates` 类型来访问其加载状态。

`CombinedLoadStates` 实例描述了 Paging 库中可加载数据的所有组件的加载状态。在本例中，我们仅关注 `ArticlePagingSource` 的 `LoadState`，因此我们将主要在 `CombinedLoadStates.source` 字段中使用 `LoadStates` 类型。您可以通过 `PagingDataAdapter.loadStateFlow` 经 `PagingDataAdapter` 访问 `CombinedLoadStates`。

`CombinedLoadStates.source` 是一种 `LoadStates` 类型，它具有针对三种不同类型 `LoadState` 的字段：

+   `LoadStates.append`：适用于在用户当前位置之后获取的项的 `LoadState`。
+   `LoadStates.prepend`：适用于在用户当前位置之前获取的项的 `LoadState`。
+   `LoadStates.refresh`：适用于初始加载的 `LoadState`。

每个 `LoadState` 本身可以是下列状态之一：

+   `LoadState.Loading`：正在加载项。
+   `LoadState.NotLoading`：未加载项。
+   `LoadState.Error`：加载时发生错误。

在本例中，我们只关心 `LoadState` 是否为 `LoadState.Loading`，因为我们的 `ArticlePagingSource` 不包含错误情况。

首先，我们要向界面顶部和底部添加进度条，用于在任一方向指示提取的加载状态。

在 `activity_articles.xml` 中，添加两个 `LinearProgressIndicator` 栏，如下所示：

```auto
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".ui.ArticleActivity">

    <androidx.recyclerview.widget.RecyclerView
        android:id="@+id/list"
        android:layout_width="0dp"
        android:layout_height="0dp"
        android:scrollbars="vertical"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

    <com.google.android.material.progressindicator.LinearProgressIndicator
        android:id="@+id/prepend_progress"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:indeterminate="true"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

    <com.google.android.material.progressindicator.LinearProgressIndicator
        android:id="@+id/append_progress"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:indeterminate="true"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

接下来，我们通过从 `PagingDataAdapter` 收集 `LoadStatesFlow` 来响应 `CombinedLoadState`。在 `ArticleActivity.kt` 中收集状态：

```auto
override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        ...

        lifecycleScope.launch {
            repeatOnLifecycle(Lifecycle.State.STARTED) {
                articleAdapter.loadStateFlow.collect {
                    binding.prependProgress.isVisible = it.source.prepend is Loading
                    binding.appendProgress.isVisible = it.source.append is Loading
                }
            }
        }
        lifecycleScope.launch {
        ...
    }
```

最后，我们在 `ArticlePagingSource` 中添加一点延迟以模拟加载过程：

```auto
private const val LOAD_DELAY_MILLIS = 3_000L

class ArticlePagingSource : PagingSource<Int, Article>() {

    override suspend fun load(params: LoadParams<Int>): LoadResult<Int, Article> {
        val start = params.key ?: STARTING_KEY
        val range = startKey.until(startKey + params.loadSize)

        if (start != STARTING_KEY) delay(LOAD_DELAY_MILLIS)
        return ...

}
```

再次运行应用并滚动到列表底部。您应该会看到，当 Paging 库提取更多项时，底部进度条会显示；当提取完成时，底部进度条会消失！

![page_preview_1.gif](./images/paging_basic/page_preview_1.gif)

## 11\. 即将完成

我们来快速回顾一下所学内容。我们…：

+   …学习了分页的概览以及进行分页的必要性。
+   …通过创建 `Pager`、定义 `PagingSource` 和发出 `PagingData`，在应用中添加了分页。
+   …使用 `cachedIn` 运算符在 `ViewModel` 中缓存了 `PagingData`。
+   …利用 `PagingDataAdapter` 在界面中使用了 `PagingData`。
+   …使用 `PagingDataAdapter.loadStateFlow` 响应了 `CombinedLoadStates`。

大功告成！如需了解更高级的分页概念，请查看高级分页 [Codelab](https://developer.android.com/codelabs/android-paging?hl=zh-cn#0)！