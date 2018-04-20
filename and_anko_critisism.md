Android: criticism of Anko layout DSL
=======

> TL;DR View layout and presentation logic are two **strictly separate** concerns which evolve independently thus they must be kept separate in your code base.

Even though Anko is a very good library overall, facilitating Android developers' lives and showcasing Kotlin language features (such as type-safe DSL builders for the case of Anko layouts), the idea to mix view layout with presentation logic doesn't seem right.

But why bother and not mix?

One scenario which happens in real world pretty often:

Say you're working in a team, on a pretty large project, over a course of some years. Number of screens grows, code base as well.

Your UI layout and style is driven by UI / UX designer(s). During months and years, even though app colors stays the same, screen of the same type don't really look the same. Say, product details screen designed at app inception does not look the same to, say offer details screen, especially when compared side by side. Why did it happen? Sometimes designers come and go, sometimes the same designer doesn't pay attention to consistency.

So, stakeholders decide that it's critical for user retention for the app to have uniform styles. Also, other apps which are spin-offs from the main one, should use the same style and should be build from the same visual components.

So designers are given a task to go over all existing screens (dozens of them) and create a library of visual components, future-proof ones.

For example, let it be "File Attachments" block: ...

When done, developers are given a task to apply components to the whole app.
Now imagine two scenarios:

* Rainy day:

You need to go though all of your view code, delete assorted attributes setup (such as text size) and replace them with style references (as designers provided components with clear styles). In a meantime, other members of the team are changing presentation logic for some subset of screens (on behalf of other tickets, just a normal flow of development, day-to-day stuff).

Also you realize that, with some new component, you need to change layout implementation from RelativeLayout to CoordinatorLayout. Then you realize that Anko does not yet support all the features of CoordinatorLayout, you get stuck for a while trying to workaround this.

So both you and other developers end up changing same lines of code (since view layout code is intermixed with event handling code).

Solutions are not perfect: you either block other devs' tickets or let them submit their PRs. Either way is bad: PRs will sit there unmerged until you fix view layouts -- but then you need to solve merge conflicts (a lot of them). Even if you try to do screens in small batches, still extra communication, orchestration and planning is needed (and we all know how much time communication consumes).

* Sunny day:

Your view layouts are completely decoupled from presentation logic (be it vanilla Android views XML or Anko with strictly cut off presentation logic), you just go over screens and update them with components. Since _data_ of screens hasn't changed (remember, this is about making design consistent), XML layouts update does not interfere with anything else: ViewModel for each screen stays the same; and there is no other connection to presentation logic.

So even though you still need to create `styles.xml` and clean up each individual `.xml` file, from individual attributes, to point it to particular styles, this work does not interfere with others'.

Others might work at their own pace, their pull requests will be merged independently of yours, and there will be no merge conflicts to solve.

And guess what? Quite soon, designers will adjust look and feel of some components (as they spot some of their bugs and visual quirks), and you will be able to quickly re-iterate on `styles.xml` and view layouts once again, once again -- without interfering with work of the other developers.

This is separation of concerns in actions, and this is just one example why it is so critically important, especially for upper-mid-size and big projects: it saves time of the whole team, it saves stakeholders' money, it decouples pace of different teams (UI / UX, developers).

I'm no fan of Android views XML, it is verbose, cumbersome; AS visual editor does not help much, so most of the time you end up writing layouts manually.

But for newcomers, Anko kind of encourages developers to mix view layout code with presentation logic -- by giving examples in which `onClick` closure sits next line to `verticalLayout`.

Why is this so important? Because when it's easy to do something, people will do it, without thinking of consequences: way of least resistance.

Codebases of many projects (not only Android) are plagued with "Massive View Controller" anti-pattern, in which "Controller" (Activity, ...) becomes God Object which handles application logic (responds to Model changes), handles view events, adjust view styles (e.g. adjusts font color, toggles visibility).

Ironically, since Android framework does not provide a handy enough way to construct / change view layout hierarchy (it's still easier to define it in XML -- at least some framework-enforced separation of concerns), "Controller" is not as massive as it could have been otherwise.

But Anko's motto is to eliminate XML -- last borderline is erased -- and for inexperienced developers that will mean all-in party: most likely, everything mentioned above would sit in one file just because they can.
