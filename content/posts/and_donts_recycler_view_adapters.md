---
title: "Android DON'Ts: life is too short for subclassing `RecyclerView.Adapter`"
date: 2021-05-21T16:04:10+03:00
draft: false
---

Android DON'Ts: life is too short for subclassing `RecyclerView.Adapter`
=======

> TL;DR Displaying collection of items on screen is such a common task so it should optimized by _not_ wasting time subclassing `RecyclerView.Adapter` (or any other `Adapter` or `ViewHolder`). Development effort is saved by using the [binding-collection-adapter](https://github.com/evant/binding-collection-adapter) library and the ViewModel pattern, both also giving a positive side effect of keeping application and framework code separate.

Lists and grids appear almost in every app, and even though `Adapter`s (and `ViewHolder`s) are typically small enough, there is almost always at least one big `Adapter` per project.

## Problems

#### Time spent to write `Adapter`s and `ViewHolder`s

That would be insignificant for small apps, but if your app contains more that a dozen of screens, `Adapter`s stack up and form a big chunk of code.

#### Coupling to the Android Framework

To quote the [Android's Guide to App Architecture](https://developer.android.com/topic/libraries/architecture/guide):

>The most important thing you should focus on is the separation of concerns in your app. It is a common mistake to write all your code in an Activity or a Fragment. Any code that does not handle a UI or operating system interaction should not be in these classes. Keeping them as lean as possible will allow you to avoid many lifecycle related problems. Don't forget that you don't own those classes, they are just glue classes that embody the contract between the OS and your app.

Even though this paragraph speaks about `Activity`s and `Fragment`s, it is very applicable to `Adapter`s as well.

With that said, `Adapter`s belong to the Android framework, so putting application logic in them is not a good idea since it creates tight coupling and leads to a non-scalable application structure.

#### Coupling to `findViewById()`, as a special case

When using `Adapter`, we would typically use `ViewHolder` as well as it's the official way to bind collection item to view layout. But that way encourages usage of `findViewById()`, to reach out widgets defined in that layout.

Dealing with `findViewById()` is not a good idea, for reasons described here: [Android DON’Ts: please abandon findViewById() and its pals as it breaches encapsulation](https://medium.com/@alexey_dmitriev/android-donts-please-abandon-findviewbyid-and-its-pals-as-it-breaches-encapsulation-f9df88356787).

## Solution

[binding-collection-adapter](https://github.com/evant/binding-collection-adapter) is a great library which allows to avoid writing `Adapter` code manually.

_(Please refer to the library's documentation to get a complete idea how it works.)_

By combining this library with the ViewModel pattern and backing it with the [Android Data Binding library](https://developer.android.com/topic/libraries/data-binding/), we get a clean and non-verbose solution.

ViewModel is as follows:

_(Code used for demonstration is simplified, for purpose of demonstration.)_

```Kotlin
class SomeViewModel : ViewModel {
    val items = ObservableArrayList<ViewModel>()
    val itemBinding = LayoutItemBinding<ViewModel>(R.layout.some_item_view)
}

interface ViewModel
```

Where `items` is a list of `ViewModel`s. Each `ViewModel` contains per-item data to be displayed in `RecyclerView`.

`itemBinding` defines a way to bind collection item's data to its view layout:

```Kotlin
class LayoutItemBinding<T: ViewModel>(val layoutId: Int) : OnItemBind<T> {
    override fun onItemBind(itemBinding: ItemBinding<*>, position: Int, item: T) {
        itemBinding.set(BR.viewModel, layoutId)
    }
}
```

To recap, for each `ViewModel` held in `items`, there would be a layout inflated from `R.layout.some_item_view`. Data from each `ViewModel` would be bound to its corresponding inflated view, through the `viewModel` variable defined in `some_item_view`:

```xml
<?xml version="1.0" encoding="utf-8"?>
<layout
    ...
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

Adding items to collection can be as follows:

```Kotlin
val itemViewModel = ItemViewModel(title, description)

someViewModel.items.add(itemViewModel)
```

After Android Data Binding executes pending bindings, data from `itemViewModel` would appear as a `RecyclerView`s item on screen.

## Benefits of the solution

#### Less boilerplate to write and maintain

No need to manually subclass `Adapter` and `ViewHolder` and write glue code to manipulate widgets. In the example above we saw how the [binding-collection-adapter](https://github.com/evant/binding-collection-adapter) library and ViewModel pattern can be used to bind data directly to `RecyclerView`'s item.

#### Fewer chances to mix framework and application code

`Adapter` is similar to `Activity` in a way that many developers tend to put application logic inside `Adapter` class. When there is no `Adapter`, temptation to put logic in it loses its origin.

#### Step towards Clean Architecture and Separation of Concerns

Application and framework code should be kept separate from each other.

From the example shown above:

```Kotlin
bind:itemBinding="@{viewModel.itemBinding}"
bind:items="@{viewModel.items}"
```

and `LayoutItemBinding` class are those few lines which glue collection of items to be displayed (which comes from `viewModel` and belongs to application code) with view layout on screen (which belongs to framework-specific code).

## Summary

We figured out a way to save time and become more efficient by _not_ writing `Adapter` classes: the [binding-collection-adapter](https://github.com/evant/binding-collection-adapter) library. With that solution, we also made a step towards Clean Architecture and Separation Of Concerns, because concerns of application and framework-specific code are no longer intermixed in `Adapter` classes.
