Android Don'ts: life is too short for subclassing `RecyclerView.Adapter` and `RecyclerView.ViewHolder`
=======

> TL;DR Commonplace task to display collection of items on screen requires _way too much_ ceremony: subclassing Adapter and ViewHolder (which quickly become large classes). Development time can be saved by using [binding-collection-adapter](https://github.com/evant/binding-collection-adapter) library; powered with Android Data Binding; throw away Adapter and ViewHolder

_This is a small article from this series: "". It describes a step which is necessary on our way to the Clean Architecture and Separation Of Concerns on Android. Being able not to write Adapter and ViewHolder not only saves time but reduces amount of code dependent on the Android Framework. With bca, view layout (XML) is the only place to interact with RecyclerView._

There are very few business apps for Android which wouldn't use `RecyclerView`: lists, grids, etc. are almost everywhere since it's a natural way to display collection of items.

Sadly there is a trap: once picked `RecyclerView`, you need to subclass `Adapter` and `ViewHolder` (yes, in real applications you can't really live with SimpleXXXXadapter). And those guys tend to grow in size (especially `Adapter`). And if you're working on a pretty sized app with dozens of screens, the majority of which are collection-based, how much pain does it take to support existing screens and add new ones? Unbearable sorrow.

Yes, most of them still stay small, but is there a need to have this kind of boilerplate?

One more major reason why those should be gone is because ViewHolder forces you to use `findViewById()` -- in order to reach out widgets, and it is a [bad practice](). If we want the Clean Architecture, we can't tolerate presence of it.

## Problems

#### Waste of time to write code for `Adapter` and `ViewHolder`

Yes, normally `Adapter`s are quite small and stay in limits of 100 lines, and `ViewHolder`s are even smaller, ...

But if those can be easily avoided, why do you need to write them at all?

And as number of screens grows, these become just more lines to maintain.

#### Coupling to the Android Framework

As it happens with `Activity`s, it is quite often when developers put business or application logic in `Adapter`s.

But adapters belong to the Android framework, developers don't own these classes

[Android's Guide to App Architecture](https://developer.android.com/topic/libraries/architecture/guide)

>The most important thing you should focus on is the separation of concerns in your app. It is a common mistake to write all your code in an Activity or a Fragment. Any code that does not handle a UI or operating system interaction should not be in these classes. Keeping them as lean as possible will allow you to avoid many lifecycle related problems. Don't forget that you don't own those classes, they are just glue classes that embody the contract between the OS and your app.

#### Coupling to `findViewById()`

In ViewHolder, widgets are typically being reached out in the same manner as in Activity -- using `findViewById()`. And this in not a great idea, more is written here: [Android Dont's]().



## Solution

There have been a truly great library: [binding-collection-adapter](https://github.com/evant/binding-collection-adapter), which is mature enough, but still underused.

As of now, we can't live without this libraries as it seems not to have competitors.

I will not fall into description of this library, refer to it on its page.

Here is an example on how to use it with our setup.

```Kotlin
class SomeViewModel() : ViewModel {
    val itemBinding = LayoutItemBinding<ViewModel>(R.layout.some_view)
    val items = ObservableArrayList<ViewModel>()
}

interface ViewModel
```

```xml
<?xml version="1.0" encoding="utf-8"?>
<layout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:bind="http://schemas.android.com/apk/res-auto">

    <data>
        <import type="android.support.v7.widget.LinearLayoutManager"/>

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

```Kotlin
class LayoutItemBinding<T: ViewModel>(val layoutId: Int) : OnItemBind<T> {
    override fun onItemBind(itemBinding: ItemBinding<*>, position: Int, item: T) {
        itemBinding.set(BR.viewModel, layoutId)
    }
}
```

## Benefits of the solution:

#### Less boilerplate to write and maintain

#### Fewer chances to mix framework and application code

#### Step towards Clean Architecture and Separation of Concerns
