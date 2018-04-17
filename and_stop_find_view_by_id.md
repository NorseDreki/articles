Android: please finally disrank and abandon `findViewById()` and friends
=======

> TL;DR Data binding and view model patterns are all in favor instead.

It looks like even though Android's data binding has been around for quite a lot of time, and the community has been quite hands-on with MVP / MVVM families of patters, people still tend to use `findViewById()` -- it is creeping through good libraries and articles.

I'll explain why it's not an optimal way, and how to become more efficient.

Disadvantages:

### 1. Too much typing

You need to type IDs two times, in view and in presenter.



### 2. Coupling between view and code

Field implementaion might change: toggle button to checkbox.

Say you replaced `RelativeLayout` with `CoordinatorLayout`, now you need to make corresponding changes in code as well.

Say you changed color for checkmark.

Say you put a field in a container and want to control visibility by container, and not by field.

### Wastes build cycles when view ID changes in view layout

View IDs in view layout must match those in code. How many times did you find yourself coming up with a better name for a view ID (and maybe also renaming some others, to keep naming consistent) and then rebuilding the app so that new IDs show up in code? And even in mid-size app that would be a lengthy cycle.

Say you're a smart one and just type new ID w/o rebuilding, but how many minutes can you keep up w/o IntelliSense if writing code blindly?

### 3. Wasting build time to generate `R.java`?

That might sound... but data binding proc can take more time.

### Abuse of reactive approach

Reactive UI is almost a de-facto standard (will skip explanation, there are many good articles proving that), and `findViewById()` gets in the way of that:
with Reactive, we should say: ok, view, here are some streams (`Subject`s), publish to those when user interaction happens. On the other side, you gotta listen to some of `Observable`s and update yourself when we push new data to those.

But with `findViewById()`, instead of trusting view with its responsibility (style itself, react on data changes, propagate user input), we reach out to it and then shake and harass it by getting through its internals: e.g., calling `setText()`, `setOnClickListener()`, and (God save you) `setLayoutDimensionsTralala()`.

### Breach of encapsulation / Exposes view implementation

That relates to the previous point: instead of allowing view to do its job on its own, we make our app aware of view's internal specifics.

Why should we know how to set view's style, margins, paddings, etc.?
Why should we know even how to set view's text?
Why should we know structure and naming of view event handlers?

Again, we should flip that and let the view be plugged in (as opposed to be controlled).


### Coupling with Android framework

Any framework is an implementation detail (refer to Clean Architecture), so whenever we developers can, we should stay decoupled from it.

Remember: framework is an entity outside boundaries of application logic.

Also, in theory, we should be able to swap one framework with another.
Say I want a mobile web application: screen layout is almost the same, presentation logic is the same. Why can't I port my code and compile it for web target? (Especially given that Kotlin compiles to JavaScript!)


### Easier testing

instrumentation tests (on devices / emulators) are freaking slow. Even though you use Robolectic to test on JVM, it requires a lot of ceremony to spin up activities and views.

When using ViewModel, you can test against it, not against views.
That shortens your TDD cycle a lot (you should do TDD; slow build and test times is also probably the main reason why TDD or just testing is abandoned for Android).

Testing against ViewModel as opposed to real views on screen (using Espresso, for example) are not mutually exclusive, you can still have your Espresso tests (and probably, you should), but key takeaway here is that you can have less (even much less) UI tests, because unit tests (those which test against ViewModel) would cover most of the cases.

Of course, there is always chance that somebody on team would not wire ViewModel to view 100% correct, but you can cover that with much fewer exploratory tests.

Maintaining and running UI tests is another (long and sometimes painful) story for a separate blog post.

So presenter dev cycle could look like:
- create a test (Spek) file for presenter
- write a non-compiling test so its assertion verifies against ViewModel
- create ViewModel class, add corresponding Field
- implement functionality in presenter so that test passes

Thus:
- you would be growing contract between view and presenter, step-by-step
- your design will be driven by TDD, which mean it would be sufficient and clear
-

Recap:

##### We use view ViewModel to establish clear contract between View and presenters

##### There is a boundary between Presenter and View, and ViewModel instance is an embodiement of contract transferred across it

##### We can change view implementation and (ultimately) even reuse ViewModel and Presenter for different platforms (since Kotlin is heading that way)

##### View layout is decoupled from which and how data is shown on screen

##### Tests do not require Android instrumentation = fast

##### Dependency inversion principle is kept: view is dependent on view model, so that dependency points inwards; compare: presenter depends on view type, dependency points outwards

##### View implementaion changes

##### Decoupling from Android framework shortens your TDD cycle since during those cycles Android framework is not compiled
