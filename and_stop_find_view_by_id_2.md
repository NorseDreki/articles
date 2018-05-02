Android Don'ts: please finally abandon `findViewById()` and its pals as it breaches encapsulation
=======

>TL;DR Manipulating widgets directly is a bad practice as it creates tight coupling between presentation code and screen layout. Consider using Android data binding and the ViewModel pattern instead.

_This article opens a series of articles about showing how to derive an efficient presentation + layout view set of patterns (MVx), based on existing known approaches. I'll show how this is suited to be used for any size and type of project (this is a scalable thing, very lean and based on principles of Clean Architecture).
We will be using "from the contrary" approach and show how to find a solution by deliberately putting ourselves into boundaries we want to have -- and how those boundaries help us to see that solution._

It looks like quite many developers are still using `findViewById()` (or its variations), despite Android data binding being mature for quite a while, and despite number of articles written on the MVVM pattern (which is the main beneficiary of data binding).

For example, `findViewById()` is seen leveraged even by features of the Kotlin language:

```Kotlin
val nameTextView by lazy { view!!.findViewById<TextView>(R.id.nameTextView) }
```

or

```Kotlin
val stateTextView: TextView by findView(this, R.id.stateTextView)
```

or

```Kotlin
val details: TextView? by bindOptionalView(R.id.details)
```

Even Kotlin's official library, [Kotlin Android Extensions](https://kotlinlang.org/docs/tutorials/android-plugin.html), took some effort to implement code generation so that widgets are avaialble in Activity's scope w/o calling `findViewById()` directly.

_In this article I'll explain why using `findViewById()` is a bad practice and also why widget binding provided by 3rd party libraries would not be an efficient solution._

So, problems with the approaches mentioned above are:

### Tight coupling between presentation code and view layout

It is coupled by:
- creating widget IDs in view layout and then referencing them in presentation code
- specifying widget types in presentation code
- presentation code knows about structure of layout
- altering widget's style from presentation code

Field implementaion might change: toggle button to checkbox.

Say you replaced `RelativeLayout` with `CoordinatorLayout`, now you need to make corresponding changes in code as well.

Say you changed color for checkmark.

Say you put a field in a container and want to control visibility by container, and not by field.

In some cases view might hold reference back to activity / presentation code causing a memory leak

Mixing declarative style (layout declaration) and imperative style (code)

View layout is decoupled from which and how data is shown on screen
What vs how

### Breach of view layout encapsulation

A follow-up for the previous one.

Widget IDs, widget types, styles applied, structure of nesting, widget attribute and method names -- all of them are implementaion details of a view (screen).

Reaching out to a particular widget (by using `findViewById()` or anything else) opens that gate for juggling view implementation details.

*This is breach of [encapsulation](https://en.wikipedia.org/wiki/Encapsulation_(computer_programming)), one of the core principles in OOP.*

Instead of doing this, we should allow view to do its job on its own, guaranteeing that we won't be controlling it imperatively. How? For example, by handing a ViewModel to a view, so that view would _react_ to changes in ViewModel and would update _itself_ accordingly.

### Staying imperative instead of reactive

When reaching out to a particular widget and calling its method so change text or set click listener, it is imperative (directly micro-managing what others should do).

When providing ViewModel so that view wires to interested streams of data on its own and also pushes to other streams upon user interaction, it is **reactive**.

Since reactive programming has gained a lot of traction, won't spend time explaining why it is superior to staying imperative.

### Burden when matching changed widget IDs in presentation code

This might seem like a small thing at first, but it was very annoying (at least for me), especially in bigger projects.

Say you've created a draft layout for a screen, compiled project (so that widget IDs are visible in code) and wrote some presentation code which acts upon those widgets.

Then you add just one new widget to layout and want to write presentation code for it (since you've got some ideas you'd better code write away until they vanish) -- you need to wait project to compile so that IDs show up in code.

Or being in the middle of presentation code you've come up with some better names for widgets, you change names right away... and wait... wait... for compilation... to complete.

And even for medium-sized project these compilation cycles stack up and eat away a noticeable junk of precios development time (not to mention it's quite boring to wait).

### Coupling to Android framework

Any framework is an implementation detail (refer to Clean Architecture), so whenever we developers can, we should stay decoupled from it.

Remember: framework is an entity outside boundaries of application logic.

Also, in theory, we should be able to swap one framework with another.
Say I want a mobile web application: screen layout is almost the same, presentation logic is the same. Why can't I port my code and compile it for web target? (Especially given that Kotlin compiles to JavaScript!)

## Solution

A way to go would be to create an abstract view using the ViewModel pattern. This decouples presentation code from screen layout so that both evolve independently. All changes your presentation code wants to do with view it does with ViewModel, and Android data binding makes sure those changes are propagated to the view.

Example of how a corresponding ViewModel would look like:

```Kotlin
class VenueViewModel {
    val image = ObservableField<String>()
    val name = ObservableField<String>()
    val location = ObservableField<String>()

    val onClicked = PublishSubject.create<View>()
}
```

Example of how view layout would look like w/o IDs:

```xml
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:bind="http://schemas.android.com/apk/res-auto">

    <data>
        <variable
            name="viewModel"
            type="com.example.VenueViewModel"/>
    </data>

    <LinearLayout
        ...
        bind:onClick="@{viewModel.onClicked}">

        <ImageView
            ...
            bind:placeholder="@{@drawable/ic_placeholder_24dp}"
            bind:src="@{viewModel.image}"/>

        <TextView
            ...
            android:text="@{viewModel.name}"/>

        <TextView
            ...
            android:text="@{viewModel.location}"/>

    </LinearLayout>

</layout>
```

Then somewhere in your presentation code (we'll talk about patterns of presentation code in other articles)

```Kotlin
viewModel.location.set(userLocation.lastSeen())
...
viewModel.onClicked.takeUntil().subscribe { ... }
```

## Benefits of this solution:

- **No more clutter for view layout code**

How many times did you find yourself doing some part of view styling and layout in XML and other part -- in Kotlin / Java code? Over times, as screen becomes more complex, it is often takes some (significant) time to find where a particular style is applied (say you hunting a bug): you need to go through XML and several source code files.

Also, view layout in XML is _declarative_, whereas things such as `locationTextView.setText()` are _imperative_: thus one type of job (view layout) is done in two ways, adding to a pile of confusion. It's better to choose just one way and be consistent with it.

And yes, declarative is always better than imperative.

- **Developers can work in parallel without interfering each other**

One developer can be changing look and feel of a screen (say to match design updates), whereas another one can be fixing a bug in presentation code for that same screen.

When both of them submit their pull requests (you do use pull requests, don't you?), there will be no conflicts between them. Also, these developers won't spend time waiting on each other or asking which lines are safe to change.

And experienced teams know how much time can be saved avoiding that extra coordination.

- **Much easier testing**

Testing on Android is still hard, also because launching instrumentation tests against UI is very, very slow. This seems a primary reason why many teams give up TDD or even doing testing at all.

With the solution proposed, you can be testing against ViewModel instead. That would allow to run tests much faster (since no emulator is involved), thus enabling TDD.

_Also, I'm not saying there should be no tests against UI (such as with Espresso), only showing a way to enable TDD. I will write about balance between tests against UI and against ViewModel in a separate article._


- **Conforms to the Reactive Programming**

Benefits of this style are already explained elsewhere, so will skip that part.

To quote the definition:

>In computing, reactive programming is a declarative programming paradigm concerned with data streams and the propagation of change.

In the ViewModel sketch shown above each field is a data stream consumed either by view layout (which listens, for example, for location change) or by presentation code (which listens for button clicks).

- **Conforms to the Clean Architecture**

So far, community has become pretty receptive with the concept of the [Clean Architecture](https://8thlight.com/blog/uncle-bob/2012/08/13/the-clean-architecture.html), however adoption of this idea is far from being high, especially concerning topic being discussed in this article.

Short recap: presentation code (sitting at "Interface Adapters") and view layout code (at "External Interfaces") belong to different layers. And according to the Dependency Rule, dependencies can only point inwards, which means presentation code _must not_ depend on view layout.

And `findViewById()` joyfully breaks that rule, forcing inner layer to be coupled to an outer one.

By using ViewModel, we fix that problem: now view layout depends on ViewModel, which is perfectly fine as ViewModel sits at more inner circle.

- **Step towards reusing code across different platforms**

To quote a relevant feature of the Clean Architecture:

>Independent of UI. The UI can change easily, without changing the rest of the system. A Web UI could be replaced with a console UI, for example, without changing the business rules.

We want to be independent of UI not because we are such purists and snobs willing to make an extra effort to support a "console UI" (which will never be needed).

But we are practicists who wants to reuse code which is already there. So we have our ViewModels written in Kotlin, and with aid of Kotlin/Native we can use them for mobile web and iOS.

But this is a big topic, no more details here -- there will be articles on how to make ViewModels really independent of framework and reusable.

- less recompilation cycles

## Summary / recap

In this article, we used `findViewById()` as an occasion to dive deeper in a world of Clean Architecture and Separation Of Concerns. You've seen how many problems `findViewById()` brings and how it is an obstacle to Clean Architecture. To solve this, the ViewModel pattern was exampled. We've also seen how many benefits this kind of decoupling brings and how it frees our hands. In next articles journey to real Clean Architecture and real Separation of Concerns will be continued. See the next article about the ViewModel.
