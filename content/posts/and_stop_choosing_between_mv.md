Android: how to stop choosing between MV* pattern and pick the best option (hint: we will synthesize our own pattern based on existing ones).
=======

>TL;DR Even though there have been a lot of articles on MV*, and especially MV* patterns are described in a fuzzy way so there is no real clear distinction between them, we need to revisit the whole approach and synthesize the most effective pattern, based on what we already know, lean and efficient.

We will be using a method which can be called "off the floor": define something we definitely don't want to do then deal with options which are left.

We will go step-by-step and you'll see how efficient this method can be.
We will build up based on assumptions on previous steps.

Strong Pillar 1:
> we want view to be decoupled from the rest of the code. We want them to evolve independently.

_This is for testing, parallel work, declarative / imperative, different types of developers, due to Clean Architecture, mutli-platform code (platform-independent code)_

Armed with this pillar, we go across MV* patterns and cut off "Controller", "Presenter" (in its classical form, ...), "Intent" and everything else, except "ViewModel".

Why? Because looks like ViewModel gives us the only option to stay (really) decoupled from view. How? Because ViewModel is basically a bunch of observables used either by view or by presentation logic (or both).

And the name does not matter:
1. We need something which truly decouples view from presentation, so that presentation doesn't know about view srtucture. Also view must not have any reference back to presentation code.
2. Controllers, ... are coupled with views so that they try to reach out controls by their specific IDs and then manipulate them. Instead the whole thing should be reactive.
3. These limitations leave us with basically the only option: there should be a class which holds observables ... (Mediator? )
4. We have strong support for data binding (from platforn) which basically is some sort of DI engine: it helps you to inject your ViewModel to a view (not really -- you bind it manually).
5. You should not directly bind Model objects to View because those are two separate concerns which evolve independently. Even though Model seems to correspond to view as 1:1 in each field, that happens only at early stage of app development. Then addition / removal of even single attribute will bring you a lot of pain (more details here: ...)

So we're done in our first step: we looked across all MV* patterns and figured out that ViewModel (or view state) is the only option which fits our needs.

In next chapters we'll continue deriving other parts of the pattern.
To read more about perils and virtues of ViewModel, follow:
To read in details, why reaching out controls with `findViewById()` is a bad thing: ...

Recap:
Why did we ditch Controller? Because it wants to control too much.
Why did we ditch Presenter? (not really)

Also see how putting ourselves into boundaries drawn by Strong Pillar, we actually facilitated our choice. This is an example how strictly following a (mindfully) chosen rule can give us a great benefit.

Now we are lost anymore in the sea of MV* patterns. We've carefully chosen our desired boundaries and by respecting them we were able to quickly find a solution which fits us. We also know this solution is superior to others since others don't conform to the boundaries.

Why should we choose boundaries? Actually, we are not choosing boundaries, but benefits (testability, separation of concerns). Then within our problem area these benefits is king, we value them more than anything else. So it makes particular patterns less valuable than benefits -- it becomes easy to choose.

Also, it is quite natural for benefits to imply limitations. We embrace them instead of fighting them -- and ultimately they show us a way to a proper solution. So, respect limitations.
