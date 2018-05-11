Android DOs: how to isolate framework by example of resources
=======

> TL;DR Shielding off of a framework is the core concept of Clean Architecture. To solve that, create an interface for a specific area of functionality and provide Android-specific implementation of it.

First thing, to remind about the first concept of Clean Architecture:

>Independent of Frameworks. The architecture does not depend on the existence of some library of feature laden software. This allows you to use such frameworks as tools, rather than having to cram your system into their limited constraints.

The Android framework has always been quite intrusive in a sense that when developing app, we need to reach out to Android features (context, view, intent, activity, bundle) pretty often, almost in every class / file.

This creates a tight coupling between your application code and framework code, making your app non-scalable, hard to maintain and reason about.

In this article we'll see how start gradually removing direct dependencies to the Android framework, by using technique called "Extract Interface".

> The “Extract Interface” technique is a classical method to break dependencies that can be found in any good book about refactoring, such as Working Effectively with Legacy Code from Michael Feathers.(https://www.amazon.com/gp/product/0131177052/ref=as_li_tl?ie=UTF8&camp=1789&creative=9325&creativeASIN=0131177052&linkCode=as2&tag=fluentcpp-20&linkId=51838682a0ea89c919bbbacb47f19807)

"Extract Interface" is a great tool which also helps to keep the Dependency Principle ("dependencies must always point inwards"), so that code in inner circles can depend on that extracted interface, and code on the very outer circle (framework circle) would provide a concrete implementation (Android) which would be injected by a DI framework such as Dagger.

I'd pick `android.content.res.Resources` as an example to be used in this article, because of its triviality and appearance in day-to-day development.

Everyone knows that we'd access app's resources via `Activity` or `Application` `context`: `activity.context.resources`.

What's wrong with it?

## Problems

#### Context is a God Object, a all-in-one pile of almost everything developer needs from the framework

While the original intention of that is good: developer has all she needs at length of his arm, sadly it leads to problems such as well-known "massive view-controller" anti-pattern.

This anti-pattern is a trap which is very easy to fall into because when writing code in `Activity` (or in `Fragment`, `Adapter`, etc.), it seems very natural and right thing in the beginning, because `context` is around which means all framework's goodies are around too.

If you use `contex.resources` in some piece of code, you are not very sure if you can extract that code out of activity? But when `resources` is interface and injected, then it becomes clear.

#### Passsing context around opens a door to leak memory by leaking activity

#### Mixing ResourcesCompat and ContextCompat throughout the apps

#### Hard to test

## Solution

Let's define an interface which has methods we need to access resources, for example:

```Kotlin
interface AppResources {

    fun getString(id: Int, vararg formatArgs: Any): String

    fun getQuantityString(id: Int, quantity: Int, vararg formatArgs: Any): String

    fun getInteger(id: Int): Int

    fun getDrawable(id: Int): Drawable

    fun getColor(id: Int): Int
}
```

You might say that Android's `Resources` contains much more methods, but most of the time we don't need them all. And these five methods from the example above come from a pretty big real-world app: no more methods were needed even though app is pretty big! And of course you can add more as you go.

Android-specific implementation can be as follows:

```Kotlin
class AndroidAppResources(
    private val appContext: Context
) : AppResources {

    override fun getString(@StringRes id: Int, vararg formatArgs: Any) =
        appContext.resources.getString(id, *formatArgs)!!

    override fun getQuantityString(@PluralsRes id: Int, quantity: Int, vararg formatArgs: Any) =
        appContext.resources.getQuantityString(id, quantity, *formatArgs)!!

    override fun getInteger(@IntegerRes id: Int) =
        appContext.resources.getInteger(id)

    override fun getDrawable(@DrawableRes id: Int) =
        ContextCompat.getDrawable(appContext, id)!!

    override fun getColor(@ColorRes id: Int) =
        ContextCompat.getColor(appContext)
}
```

Also note a small nicety: since this class is entirely Android-specific, we can use Android Support Annotations such as `@StringRes`, even though `AppResources` interface doesn't have them declared.

Note how Android specifics `ContextCompat` vs `appContext.resources` are nicely centralized in one place. If implementation changes (say you've decided to use `ResourcesCompat` instead), that change would be everywhere in your app, no need to look for it and probably miss smth.

How do we get an instance of `AndroidAppResources`, in places where we intend to use it? Dependency Injection is in help, as usual.

Here is a Dagger example of how it could be defined in a module:

```Kotlin
@Module
class AppModule {
    @Provides
    @ScopeSingleton(AppModule::class)
    internal fun provideAppResources(appContext: Context): AppResources =
        AndroidAppResources(appContext)
}
```

Then we can request it in a client class:

```Kotlin
class SomeUsefulOne
@Inject constructor(
  private val appResources: AppResources
) {

  fun doGoodie() {
    val someString = appResources.getString(R.string.some_string)
  }
}
```


## Benefits of the solution

#### Application code is decoupled from framework code

#### Dependency rule is there

#### Reusable code as independent of framework

#### Clear responsibility

We created a class which has a clear responsibility: giving access to resources.

Before it was everything: what does `context` mean? What single

Limits the surface of

#### Testability

With no dependencies on context where only resources are needed, we can run tests on JVM and mock `Resources` using Mockito. No need to deal with context.

#### Encapsulates perils of android

`ResourcesCompat` vs `ContextCompat`, both

#### Mitigate impact of context God Object

## Summary

In this article we've learned how to draw a clear boundary between application and framework code, by example of Shielding usage of app resources using "Extract Interface" method. This allowed to extract code and make it independent of Android, even though it seemed not extractable at first sight. Also, that made that code testable.



```Kotlin
package com.upwork.android.mvvmp

import android.content.Context
import android.content.res.Resources.Theme
import android.support.annotation.*
import android.support.v4.content.res.ResourcesCompat

open class Resources(
    private val appContext: Context
) {
    open fun getString(@StringRes id: Int, vararg formatArgs: Any) =
        appContext.resources.getString(id, *formatArgs)!!

    open fun getDrawable(@DrawableRes id: Int, theme: Theme? = null) =
        ResourcesCompat.getDrawable(appContext.resources, id, theme)!!

    open fun getColor(@ColorRes id: Int, theme: Theme? = null) =
        ResourcesCompat.getColor(appContext.resources, id, theme)

    open fun getQuantityString(@PluralsRes id: Int, quantity: Int, vararg formatArgs: Any) =
        appContext.resources.getQuantityString(id, quantity, *formatArgs)!!

    fun getInteger(@IntegerRes id: Int) = appContext.resources.getInteger(id)
}
```
