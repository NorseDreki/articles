Android Don'ts: life is too short for subclassing `RecyclerView.Adapter`
=======

> TL;DR Displaying collection of items on screen is such a common task that it should optimized by _not_ wasting effort subclassing `RecyclerView.Adapter` (and `RecyclerView.ViewHolder`). Development time is saved by the [binding-collection-adapter](https://github.com/evant/binding-collection-adapter) library which also gives a positive side effect of keeping application and framework code separate.

Lists and grids appear almost in every app, and even though `Adapter`s (and `ViewHolder`s) are typically small enough, there tends to be at least one large `Adapter` per project.

And if your app contains dozens of screens, even code of small ones stacks to form up a big chunk.

## Problems

#### Time spent to write code for `Adapter` and `ViewHolder`

Time saved here can be used to do more interesting tasks :)

#### Coupling to the Android Framework

To quote the [Android's Guide to App Architecture](https://developer.android.com/topic/libraries/architecture/guide):

>The most important thing you should focus on is the separation of concerns in your app. It is a common mistake to write all your code in an Activity or a Fragment. Any code that does not handle a UI or operating system interaction should not be in these classes. Keeping them as lean as possible will allow you to avoid many lifecycle related problems. Don't forget that you don't own those classes, they are just glue classes that embody the contract between the OS and your app.

Even though this paragraph speaks about `Activity` and `Fragment`, it is applicable to `Adapter` as well.

So, `Adapter`s belong to the Android framework, so putting application logic in them in not a good idea since it creates tight coupling.

#### Coupling to `findViewById()`, as a special case

`ViewHolder`, which comes together with `Adapter`, has a design which makes it easy to be using `findViewById()` to reach out to widgets in view layout. And this is not a good idea, for the reasons described here: [Android Don'ts: please abandon `findViewById()` and its pals as it breaches encapsulation]().

## Solution

This is a great library which lets you to avoid writing `Adapter`s and `ViewHolder`s manually: [binding-collection-adapter](https://github.com/evant/binding-collection-adapter). It leverages power of the [Android Data Binding library](), to do the job.

_(Please refer to the library's documentation to get a complete idea how it works.)_

And by combining this library with the ViewModel pattern (which has more explanation here: []()), we get a lean solution which allows us to display collection of items on screen.

ViewModel is as follows:

_(Code used for examples is simplified, for purposes of demonstration. For example, usage of Dependency Injection is omitted.)_

```Kotlin
class SomeViewModel : ViewModel {
    val items = ObservableArrayList<ViewModel>()
    val itemBinding = LayoutItemBinding<ViewModel>(R.layout.some_item_view)
}
```

Where `items` is a list of `ViewModel`s each holding data to be displayed in an item of `RecyclerView`.

`itemBinding` is a class which describes a way to bind item and its data to a view layout:

```Kotlin
class LayoutItemBinding<T: ViewModel>(val layoutId: Int) : OnItemBind<T> {
    override fun onItemBind(itemBinding: ItemBinding<*>, position: Int, item: T) {
        itemBinding.set(BR.viewModel, layoutId)
    }
}
```

So, for each `ViewModel` held in `items`, there would be a layout inflated from `R.layout.some_item_view`. And data from that `ViewModel` would be bound to that inflated view, though the `viewModel` variable defined in `some_item_view`.

And `ViewModel` is a type:

```Kotlin
interface ViewModel
```

Then adding items to collection yields to:

```Kotlin
val itemViewModel = ItemViewModel()

someViewModel.items.add(itemViewModel)
```

After Android Data Binding would execute pending bindings, data from `itemViewModel` would appear inside a `RecyclerView`s item on screen.

Finally, view layout which holds that `RecyclerView`:

```xml
<?xml version="1.0" encoding="utf-8"?>
<layout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:bind="http://schemas.android.com/apk/res-auto">

    <data>
        <import type="me.tatarka.bindingcollectionadapter2.LayoutManagers"/>

        <variable
            name="viewModel"
            type="com.example.SomeViewModel"/>
    </data>

    <android.support.v7.widget.RecyclerView
        ...
        bind:itemBinding="@{viewModel.itemBinding}"
        bind:items="@{viewModel.items}"
        bind:layoutManager="@{LayoutManagers.linear()}"/>
```



## Benefits of the solution:

#### Less boilerplate to write and maintain

No need to manually subclass `Adapter` and `ViewHolder` and write glue code to be able to manipulate widgets. In the example above we saw how the ViewModel pattern can be used to bind data directly to `RecyclerView`'s items.

#### Fewer chances to mix framework and application code

`Adapter` is similar to `Activity` in a way that many developers tend to put application logic inside `Adapter` class. When there are no `Adapter`s, temptation to put logic in them loses its origin.

#### Step towards Clean Architecture and Separation of Concerns

Both of these concepts are about how to keep application and framework code decoupled. In the example shown above

```Kotlin
bind:itemBinding="@{viewModel.itemBinding}"
bind:items="@{viewModel.items}"
bind:layoutManager="@{LayoutManagers.linear()}
```

are those few lines which glue collection of items to be displayed (which comes from a ViewModel and belongs to application code) with framework-specific code (which is XML layouts for `RecyclerView` and item).

## Summary

We figured out how to save time and become more efficient by _not_ writing `Adapter` classes: the [binding-collection-adapter](https://github.com/evant/binding-collection-adapter) library can do that job automatically. Also, we made a step towards Clean Architecture and Separation Of Concerns, because `Adapter` is quite a typical place to mix concerns of framework and application code.
