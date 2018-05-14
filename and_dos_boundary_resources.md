Android DOs: example on how to draw a clear boundary between application and framework code
=======

> TL;DR Shielding away from framework is the core concept of Clean Architecture. To achieve that, introduce an interface for each specific area of functionality and provide an Android implementation of it.

Let's refresh our minds by re-reading the core concept of [Clean Architecture](https://8thlight.com/blog/uncle-bob/2012/08/13/the-clean-architecture.html):

>Independent of Frameworks. The architecture does not depend on the existence of some library of feature laden software. This allows you to use such frameworks as tools, rather than having to cram your system into their limited constraints.

The Android framework has been very helpful to provide nice features at finger tips: it is almost everywhere we can reach out `Application`, `Activity` or `Context`, which provide the majority of those goodies. Easy to access!

But because of that there is a trap many developers fall into: a lot (or most) of application code is written in `Activity`s, `Fragment`s, `Adapter`s or other classes of the Android framework, making it a variation of the "massive view-controller" anti-pattern. Even when code is extracted to separate classes, there is almost always `Context` (or friends) being passed around.

`Context` (and friends) is an example of the God Object anti-pattern, since it's a bag for many things often non-relevant to each other. This creates tight coupling to framework, makes things harder to test, opens a door for memory leaks (for example, by passing `Context` to a place where it outlives `Activity`).

[Android's Guide to App Architecture](https://developer.android.com/topic/libraries/architecture/guide) agrees with these problems:

>The most important thing you should focus on is the separation of concerns in your app. It is a common mistake to write all your code in an Activity or a Fragment. Any code that does not handle a UI or operating system interaction should not be in these classes. Keeping them as lean as possible will allow you to avoid many lifecycle related problems. Don't forget that you don't own those classes, they are just glue classes that embody the contract between the OS and your app.

But how to mitigate them?

## Decoupling application and framework code by example of defining a clear contract for `Resources`

Let's pick `android.content.res.Resources` as an example, since it's a commonly used class well known by any developer.

There are two main problems:
1. `Resources` is typically accessed from `Activity`s `context.resources` which means that `context` is spread around to places where `Resources` is needed.
2. One of the core design principles is broken: ["Program to an interface, not an implementation"](https://www.artima.com/lejava/articles/designprinciples.html), because `android.content.res.Resources` is a platform-specific implementation of application resources.

We should depend on an interface instead.

To do so, let's find usages of `Resources` in application codebase. Based on those findings, apply the "Extract Interface" technique (you may refer to the book Working Effectively with Legacy Code by Michael Feathers, for full description of it).

Here is what I've got out of codebase of a pretty big application:

```Kotlin
interface AppResources {

    fun getString(id: Int, vararg formatArgs: Any): String

    fun getQuantityString(id: Int, quantity: Int, vararg formatArgs: Any): String

    fun getInteger(id: Int): Int

    fun getDrawable(id: Int): Drawable

    fun getColor(id: Int): Int
}
```

_(Yes, that few methods were used out of `Resources`, in a real-world production app.)_

Then Android-specific implementation of `AppResources` can be as follows:

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
        ContextCompat.getColor(appContext, id)
}
```

Key takeaways?
- We made the intention clear: now there is no need to pass around `context` when only `resources` is needed; also, `context` of `Application` is sufficient, no need for `Activity`s one (in this example).
- Implementation is centralized, easy to change; changes are applied to all call sites.
- When implementation changes, it can be compiled without recompiling modules with application logic.
- Testing is much easier as interface is mockable.
- Code of the interface can be used on another platform (Kotlin/Native is coming).

How to use the solution? Dependency injection would help to put a concrete implementation in place of the `AppResources` interface.

Here is an example of Dagger's `Module`:

```Kotlin
@Module
class AppModule {
    @Provides
    @ScopeSingleton(AppModule::class)
    internal fun provideAppResources(appContext: Context): AppResources =
        AndroidAppResources(appContext)
}
```

Then `AndroidAppResources` will be provided as a dependency in `SomeUsefulOne`:

```Kotlin
class SomeUsefulOne
@Inject constructor(
  private val appResources: AppResources
) {

  fun doGoodie() {
    val someString = appResources.getString(someStringId)
    ...
  }
}
```

## Summary

In this article we've learned how to draw a clear boundary between application and framework glue code, by example of shielding off application resources using the "Extract Interface" refactoring method. This allowed to keep concerns of two types of code separate.
