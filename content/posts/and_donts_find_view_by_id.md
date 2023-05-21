---
title: "Android Don'ts: please abandon `findViewById()` and its pals as it breaches encapsulation"
date: 2022-05-21T16:04:10+03:00
draft: false
---

Android Don'ts: please abandon `findViewById()` and its pals as it breaches encapsulation
=======

>TL;DR Manipulating widgets directly is a bad habit as it breaches encapsulation and creates a tight coupling between presentation code and view layout declaration. Consider using the ViewModel pattern instead, backed by the Android Data Binding library.

_This article opens a series in which we'll be deriving an efficient set of patterns related to view layout and presentation code. Armed with good parts of MVx patterns, Clean Architecture, Separation Of Concerns, Reactive Programming, we'll shape them up to be suitable for day-to-day screen development routine, regardless of size of application. We'll be staying lean, but be doing the right thing, not just the quickest thing possible._

It seems many developers keep using `findViewById()` (or its variations), despite popularity of the ViewModel pattern and maturity of the [Android Data Binding library](https://developer.android.com/topic/libraries/data-binding/).

My guess it's because `findViewById()` is an easy thing to get in touch with: it spreads across the official documentation and open source repos. Also, articles highlighting trendy Kotlin features give it a shot as well, for example:

```Kotlin
val nameTextView by lazy { view!!.findViewById<TextView>(R.id.nameTextView) }
```
or

```Kotlin
val details: TextView? by bindOptionalView(R.id.details)
```
Even the official library, [Kotlin Android Extensions](https://kotlinlang.org/docs/tutorials/android-plugin.html), took effort implementing code generation: widgets become available in Activity's scope without direct calls to `findViewById()`.

Even though nice language features and well-made libraries do make life easier, there are problems in the foundation of `findViewById()` itself.

## Problems

#### Tight coupling between view layout and presentation code

Examples:
- Creating widget IDs in view layout and referencing them in presentation code -- coupling by IDs.
- Specifying widget types when declaring variables in presentation code.

  * Say you've changed widget type from `CheckBox` to `ToggleButton` or replaced `RelativeLayout` with `CoordinatorLayout`, now presentation code should be updated as well.

- Presentation code knows view layout structure and how widgets are nested within container.

  * Say you've wrapped a widget with a container and now want to control visibility by that container -- presentation code should not call `setVisibility()` on the widget anymore.

- Widgets' look and feel is manipulated by presentation code, although most of the time it's possible to set those in XML.

- Sometimes view layout might hold reference back to presentation code, thus making two-way coupling.

- ...list goes on.

On top of that, there is coupling between two different types of code: view layout is _declarative_ (it says _what_ widgets should be in layout and doesn't say _how_ that should be achieved), and presentation code is _imperative_ (it says _how_ widgets should be manipulated, step-by-step; and _what_ is not very clear, through those steps).

#### Breach of view layout encapsulation

A continuation of the previous one.

Widget IDs, widget types, styles applied, structure of nesting, widget attribute and method names -- these are implementation details of view layout.

`findViewById()` is nothing more but a way to break into view layout, get knowledge about its internal structure and start manipulating it based on that knowledge.

*This is breach of [encapsulation](https://en.wikipedia.org/wiki/Encapsulation_(computer_programming)), one of the core principles in OOP.*

Instead, we should allow view layout to do its job, treating it as an obscure entity with its own responsibility, guaranteeing no micro-management from presentation code.

#### Staying imperative instead of following Reactive Programming

Telling widget to set text, change color, change layout parameters, assigning click listener is imperative.

Cases when view layout manipulates presentation code by directly calling its methods in it, are imperative, too.

By Reactive Programming, we should rely on streams of data instead. Interested parties (presentation code, view layout) can subscribe to relevant sources and _react_ to data in those streams according to their responsibilities.

#### Coupling to the Android framework

Since presentation code knows about `findViewById()`, widget types, widget method names, ..., it stays coupled to the Android framework, because these features belong to it.

But to quote a statement from [Clean Architecture](https://8thlight.com/blog/uncle-bob/2012/08/13/the-clean-architecture.html):

>Independent of Frameworks. The architecture does not depend on the existence of some library of feature laden software. This allows you to use such frameworks as tools, rather than having to cram your system into their limited constraints.

Also, the "Independent of Frameworks" thing is important because we'd want more code to be reused for different platforms: iOS, mobile Web (this would be the topic for other series of articles, but to give a clue, Kotlin/Native is heading in that direction).

## Solution

_(I've stripped off comparison of solutions because I'd like to shift focus to importance of Clean Architecture and Separation Of Concerns. This solution does conform to both, being its main virtue. If you feel other approaches might have been considered, let's discuss that in comments.)_

What is something simple which separates presentation code from implementation detail of view layout? What is something view-like (meaning it has data to be displayed on screen) but generic enough not to bind presentation code to widget types, etc.? Sounds like the ViewModel pattern.

_(We will take a closer look at ViewModel and its relations in a separate article, only an introduction is given here.)_

This is an example how view layout would look like, without widget IDs, but with a ViewModel attached (please make sure you are familiar with the [Android Data Binding library](https://developer.android.com/topic/libraries/data-binding/)):

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

Suppose, we decided to update this layout to:

```xml
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:bind="http://schemas.android.com/apk/res-auto">

    <data>
        <variable
            name="viewModel"
            type="com.example.VenueViewModel"/>
    </data>

    <TextView
        ...
        android:text='@{viewModel.name + ", " + viewModel.location}'
        bind:drawableStart="@{viewModel.image}"
        bind:onClick="@{viewModel.onClicked}"/>

</layout>
```

The change was done with ease, without touching presentation code at all and even without recompilation. And the latter might not look like a big deal, but in a project with many dozens of screens it does save _a lot_ of time.

Then, contents of the corresponding ViewModel:
```Kotlin
class VenueViewModel {
    val image = ObservableField<String>()
    val name = ObservableField<String>()
    val location = ObservableField<String>()

    val onClicked = PublishSubject.create<View>()
}
```
Now your presentation code is completely decoupled from view layout:
```Kotlin
//to propagate changes to view
viewModel.location.set(lastSeen)
...
//to react to user input
viewModel.onClicked.subscribe { navigateElsewhere() }
```

_Both ViewModel and presentation code is simplified, many details are omitted -- this will be covered in upcoming articles._

## Benefits of the solution

#### View-related code is not scattered

How many times did you find yourself doing part of view styling and layout in XML and other part -- in Kotlin (Java) code? As screen becomes more complex, it might take significant time to find where a particular style is applied: you need to go through XML and several source code files.

The ViewModel pattern forces you to shift view-related code to view, since presentation code no longer has a direct reference to view layout.

As a positive side effect of this we have clear separation between two types of code: view layout is _declarative_, presentation code is _imperative_.

#### Developers can work in parallel without interfering each other

One developer can be updating look and feel of a screen, whereas another one can be fixing a bug in presentation code for that same screen, at the same time.

When both of them submit their pull requests, there will be no conflicts between them. Also, these developers won't spend time waiting on each other or asking each other which lines are safe to change.

And all experienced teams know how much time can be saved avoiding that extra coordination.

#### Testing becomes much easier

Testing on Android is still hard, also because launching instrumentation tests against UI is very slow. This seems to be a primary reason why many teams give up TDD or even doing testing at all.

With the current solution, you can be testing against ViewModel instead. That would allow to run tests much faster (since no emulator is involved), thus enabling TDD.

_Also, I'm not saying there should be no tests against UI. I will write about balance between tests against UI and against ViewModel in a separate article._

#### Conforms to Reactive Programming

To quote the definition:

>In computing, reactive programming is a declarative programming paradigm concerned with data streams and the propagation of change.

In the ViewModel code shown above each field is a data stream consumed either by view layout (listening, for example, for location change) or by presentation code (listening for button clicks).

Thus, encapsulation of view layout is intact as view layout takes its own responsibility to wire up to streams of data and update itself accordingly.

#### Conforms to Clean Architecture

For example, [Clean Architecture](https://8thlight.com/blog/uncle-bob/2012/08/13/the-clean-architecture.html) says presentation code (sitting at the "Interface Adapters" circle) and view layout code (at "External Interfaces") belong to different layers. And according to the Dependency rule, dependencies can only point inwards, meaning that presentation code _must not_ depend on view layout, because "Interface Adapters" can't depend on "External Interfaces".

This statement does not hold when using `findViewById()`, and the ViewModel pattern fixes that.

Also, the current solution conforms to other important rules: "Independent of Frameworks", "Testable", "Independent of UI".

#### Step towards reusing code across different platforms

To quote Clean Architecture:

>Independent of UI. The UI can change easily, without changing the rest of the system. A Web UI could be replaced with a console UI, for example, without changing the business rules.

We want to be independent of UI not because we want to make a random extra effort.

Instead, we want to leverage reusability because UI-independent and platform-independent code can be shared (Kotlin/Native has already been heading in that direction). I'll be writing about this topic in upcoming articles.

## Summary

We highlighted problems brought by usage of `findViewById()` and found a solution to mitigate those. We also saw how scalable the solution is and how it conforms to Clean Architecture and Separation Of Concerns.
