Android DOs: example on how to draw a clear boundary between application and framework code
=======

> TL;DR Shielding away from framework is the core concept of Clean Architecture. To achieve that, introduce an interface for each specific area of functionality and provide an Android implementation of it.

Let's refresh our minds by re-reading the core concept of [Clean Architecture](https://8thlight.com/blog/uncle-bob/2012/08/13/the-clean-architecture.html):

>Independent of Frameworks. The architecture does not depend on the existence of some library of feature laden software. This allows you to use such frameworks as tools, rather than having to cram your system into their limited constraints.

The Android framework has been very helpful to provide nice features at finger tips: it is almost everywhere we can reach out `Application`, `Activity` or `Context`, which provide the majority of those goodies. Easy to access!

But because of that there is a trap many developers fall into: a lot (or most) of application code is written in `Activity`s, `Fragment`s, `Adapter`s or other classes of the Android framework, making it a variation of the "massive view-controller" anti-pattern. Even when code is extracted to separate classes, there is almost always `Context` (or `View`) being passed around.

`Context` (and friends) is an example of the God Object anti-pattern, since it's a bag for many, many things, often non-relevant to each other. This creates tight coupling to framework, makes things harder to test, opens a door for memory leaks (for example, by passing `Context` around).

[Android's Guide to App Architecture](https://developer.android.com/topic/libraries/architecture/guide) agrees with the stuff metioned above:

>The most important thing you should focus on is the separation of concerns in your app. It is a common mistake to write all your code in an Activity or a Fragment. Any code that does not handle a UI or operating system interaction should not be in these classes. Keeping them as lean as possible will allow you to avoid many lifecycle related problems. Don't forget that you don't own those classes, they are just glue classes that embody the contract between the OS and your app.

But how do we solve those problems?

## Decoupling application and framework code by example of defining a clear contract for `Resources`

There is the `android.content.res.Resources` class typically accessed from `activity.context.resources`, let's pick it as an example, since it's a commonly used tool.

In order to use `resources`, most often `context` is being passed around, from `Activity`.

Recall one of the core principles:

>Program to an Interface, not an implementation

?

`Resources` is a concrete implementation of concept of application resources, provided by the Android framework. We should not depend on it, we should depend on an interface instead. If we do so, we would also not break the Dependency Inversion Principle.

To do so, let's check app's codebase and find usages of `Resources`. Based on that, the "Extract Interface" technique can be used (from the book Working Effectively with Legacy Code by Michael Feathers). Here is what I've got scanning one pretty big sized application:

```Kotlin
interface AppResources {

    fun getString(id: Int, vararg formatArgs: Any): String

    fun getQuantityString(id: Int, quantity: Int, vararg formatArgs: Any): String

    fun getInteger(id: Int): Int

    fun getDrawable(id: Int): Drawable

    fun getColor(id: Int): Int
}
```

_(Yes, there are more methods in `Resources`, but real world apps use much less of it. It's not that hard to add a couple of new methods when you need them, is it?)_

Then we provide an Android-specific implementation, which can be as follows:

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

What are the benefits in this solution?
- We made intention clear: now there is no need to pass around `context` if we mean to use only `resources` out of it.
- Resource-oriented implementation is located in one place, it is easy to change and changes are centralized.
- When implementation changes, it can be compiled without recompiling modules with application logic.
- Code which uses resources depends on interface, not on implementation, thus keeping code decoupled and satisfying the dependency rule.
- Testing is much easier as interface is easily mockable.
- Code of the interface can be used on another platform (see Kotlin/Native).

Also note a small nicety: since this class is entirely Android-specific, we can use Android Support Annotations such as `@StringRes`, even though `AppResources` interface doesn't have them declared.


How do we get an instance of `AndroidAppResources`, in places where we intend to use it? Dependency Injection is to help, as usual.

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
    val someString = appResources.getString(someStringId)
    ...
  }
}
```

`SomeUsefulOne` and `AppResources` belong to application or business code, as opposed to framework glue code. Both types of code reside in separate modules, so can be changed, compiled and reused independently.


## Summary

In this article we've learned how to draw a clear boundary between application and framework code, by example of Shielding usage of app resources using "Extract Interface" method. This allowed to extract code and make it independent of Android, even though it seemed not extractable at first sight. Also, that made that code testable.
