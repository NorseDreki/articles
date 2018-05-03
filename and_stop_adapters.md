Android Don'ts: life is too short for subclassing `RecyclerView.Adapter` and `RecyclerView.ViewHolder`
=======

> TL;DR Commonplace task to display collection of items on screen requires _way too much_ ceremony: subclassing Adapter and ViewHolder (which quickly become large classes). Development time can be saved by using [binding-collection-adapter](https://github.com/evant/binding-collection-adapter) library; powered with Android Data Binding; throw away Adapter and ViewHolder

_This is a small article from this series: "". It describes a step which is necessary on our way to the Clean Architecture and Separation Of Concerns on Android. Being able not to write Adapter and ViewHolder not only saves time but reduces amount of code dependent on the Android Framework. With bca, view layout (XML) is the only place to interact with RecyclerView._

There are very few business apps for Android which wouldn't use `RecyclerView`: lists, grids, etc. are almost everywhere since it's a natural way to display collection of items.

Sadly there is a trap: once picked `RecyclerView`, you need to subclass `Adapter` and `ViewHolder` (yes, in real applications you can't really live with SimpleXXXXadapter). And those guys tend to grow in size (especially `Adapter`). And if you're working on a pretty sized app with dozens of screens, the majority of which are collection-based, how much pain does it take to support existing screens and add new ones? Unbearable sorrow.

Yes, most of them still stay small, but is there a need to have this kind of boilerplate?

One more major reason why those should be gone is because ViewHolder forces you to use `findViewById()` -- in order to reach out widgets, and it is a [bad practice](). If we want the Clean Architecture, we can't tolerate presence of it.

## Solution

There have been a truly great library: [binding-collection-adapter](https://github.com/evant/binding-collection-adapter), which is mature enough, but still underused.

As of now, we can't live without this libraries as it seems not to have competitors.

I will not fall into description of this library, refer to it on its page.

Here is an example on how to use it with our setup.


```xml
<?xml version="1.0" encoding="utf-8"?>
<layout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:bind="http://schemas.android.com/apk/res-auto">

    <data>

        <import type="me.tatarka.bindingcollectionadapter2.LayoutManagers"/>

        <import type="android.support.v7.widget.LinearLayoutManager"/>

        <variable
            name="viewModel"
            type="com.example.VenuesViewModel"/>
    </data>

    <android.support.v7.widget.RecyclerView
        ...
        bind:itemBinding="@{viewModel.itemBinding}"
        bind:items="@{viewModel.items}"
        bind:layoutManager="@{LayoutManagers.linear(LinearLayoutManager.HORIZONTAL, false)}"/>
```

```Kotlin
viewManager = LinearLayoutManager(this)
        viewAdapter = MyAdapter(myDataset)

        recyclerView = findViewById<RecyclerView>(R.id.my_recycler_view).apply {
            // use this setting to improve performance if you know that changes
            // in content do not change the layout size of the RecyclerView
            setHasFixedSize(true)

            // use a linear layout manager
            layoutManager = viewManager

            // specify an viewAdapter (see also next example)
            adapter = viewAdapter

        }
```


```Kotlin
class LayoutItemBinding<T : Any>() : OnItemBind<T> {
    override fun onItemBind(itemBinding: ItemBinding<*>, position: Int, item: T) {
        val layout = (item as? HasLayout)?.layoutId ?:
            item.javaClass.getAnnotation(Layout::class.java).value
        itemBinding.set(BR.viewModel, layout)
    }
}

class DefaultDataBinder : DataBinder {
    override fun bind(view: View, viewModel: ViewModel) {
        DataBindingUtil.bind<ViewDataBinding>(view).run {
            setVariable(BR.viewModel, viewModel)
            executePendingBindings()
        }
    }
}
```

```Java
public class SavedSearchViewModel implements ViewModel, HasLayout {
    public final ObservableField<String> name = new ObservableField<>();
    public final ObservableField<String> summary = new ObservableField<>();
    public final ObservableField<String> id = new ObservableField<>();

    public final PublishSubject<View> onClicked = PublishSubject.create();

    @Override
    public int getLayoutId() {
        return R.layout.saved_search_item;
    }
}
```

class EmbeddedSuggestedFreelancerViewModel(
    override val id: String
) : ViewModel, HasLayout , HasFreelancerId {
    override val layoutId = R.layout.suggested_freelancer_embedded_item

    val name = ObservableField<String>()
    val location = ObservableField<String>()
    val hourlyRate = ObservableField<String>()
    val image = ObservableField<String>()

    val onClicked = PublishSubject.create<View>()
}


@SuggestedFreelancersScope
class EmbeddedSuggestedFreelancersViewModel
@Inject internal constructor(
    override val itemBinding: OnItemBind<ViewModel>,
    override val errorState: ErrorStateViewModel
) : AccessoryViewModel, HasScreenState, HasOnItemClicked,
    HasListItems, HasLoadingMoreIndicator, HasErrorState, HasRefresh,
    HasScrolling, HasItemsTotal, HasEmptyState, HasLayout {
    override val layoutId = R.layout.suggested_freelancer_embedded_view

    override val isRefreshEnabled = ObservableProperty(false)
    override val isRefreshing = ObservableProperty(false)
    override val screenState = ObservableField(ScreenState.CONTENT)
    val embeddedScreenState = ObservableField(EmbeddedScreenState.CONTENT)
    override var itemsTotal: Int? = null

    override val onScrolledToEnd = PublishSubject.create<Void>()!!
    override val onScrollStateChanged = PublishSubject.create<Void>()!!

    override val items = ObservableArrayList<ViewModel>()
    val embeddedItems = ObservableArrayList<ViewModel>()

    override val emptyState = ActionableAlertViewModel()
    override val loadingMoreIndicator = LoadingMoreIndicatorViewModel()

    override val onItemClicked = PublishSubject.create<ViewModel>()!!

    val onShowMoreClicked: PublishSubject<View> = PublishSubject.create()

    fun newItem(id: String) =
        EmbeddedSuggestedFreelancerViewModel(id).apply {
            onClicked
                .map { this }
                .subscribe(onItemClicked)
        }
}
