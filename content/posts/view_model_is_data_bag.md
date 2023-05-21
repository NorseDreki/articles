ViewModel should be just data bag
=======

> TL;DR Since we have Data Binding, Don't want to end up in another "Massive View Controller (Whatever)"-like situation? Keep ViewModel lean: let its Single Responsibility be just passing data between View and Presenter (or Controller).

> This post is about what lean, efficient, clean ViewModel should be.
THis post is about why using ViewModel (although there is plenty of other MV* patterns) is a more efficient way.
This post is about to show a way on how to derive a new and very pragmatic pattern replacing MVC / MVP / MVVM.

Say you're experienced enough to know how the MVC pattern could quickly turn into a mess due to too many things happening in Controller. So you picked a different pattern: MVP, MVVM, ... . But over time you might notice that the "VM" or "P", even though originally meaning something else, becomes another Controller, despite its facia.

There seems to be no clear definition of what ViewModel is (at least none of what I've found so far). No clear definition adds even more to an already existing big pile of confusion in a world of MVC / MVP / MVVM.

I believe its definition should be:
"ViewModel is a (view abstraction) design pattern which suggests developer to create a dedicated class holding only data (model) required by view, and _nothing else_. ViewModel is a contract between"

Next I will explain how I've come up with such a definition, step-by-step.

#### Starting point.

Ok, why do we need to use ViewModel at all? There is plenty of other patterns, all of the same level, of varying shapes. Why do we need to stick to this pattern, especially in a particular form?

##### Axiom (Statement/Precondition) 1:
>Separation of view's visual representation and data to be displayed in it is a virtue.

(Why is it claimed to be axiom?)

- view code is typically declarative (Android XML, XAML, HTML)
- view model and presentation logic evolve independently (different types of developers working on different places at a time, without blocking each other)
- loose coupling to frameworks (clean architecture)
- loose coupling between view and everything else (ViewModel is the single contact point)

Is there any other pattern which allows to have "abstract view"? Looks like not.

Thus we can say that ViewModel is an "abstract view" (in a such relationship as class is an implementation of abstract data type)


#### Given that we agreed with (1), what do we do next?

What do we put in ViewModel.
Since


###### - This is too few code for a single class - just fields
And it is a good thing, since SRP stands here: ViewModel is a contract between view and presentation logic. It does **not** make any actions and does not reach hands out to change view imperatively.

###### - Where do I put my presentation logic then?
You'd put it in a Presenter. Presenter has that ViewModel injected. It changes viewmodel and reacts to user changes.

###### - But this is already too many components then: MV, then ViewModel, then Presenter. Overkill!
If you count by letters, yes, there will be more of them.
If you count by responsibilities or concerns, they are still the same: setting up view fields, fetching data, reacting to user input.
Using MVC, ... does not save you from doing these jobs. On the opposite, these concerns are not separate: you just juggle too many of them in a single place (controller).
Creating an _explicit_ class for ViewModel with _no behavior_ clearly factors out a concern which is ...

###### - But this is an overhead in typing / too many source files
This is not an overhead in typing: you need to type names of those fields anyways, as well as presentation logic. Now it is done in two separate places.
Yes, technically there is an overhead in resulting in a new file / class being created -- but your IDE contains tools to scaffold a class if you are too lazy to type its declaration.

###### - I'm struggling navigating between view -- ViewModel -- presenter -- Whatever
Then it's better for you to establish and follow a naming convention for those:
- proposal_view.xml
- ProposalViewModel.kt
- ProposalPresenter
(AttachmentSummary)

Then just use "Open file..." shortcut of your IDE and start typing "Propo...": you get even more visibility than before since you can navigate to data right away, as opposed to open a fat view model and start scrolling for the needed part.

###### -

>I hear "but this is extra typing / boilerplate argument quite a lot", and I am very surprised with it (not an argument to me at all): _typing itself_ is one of the least time consuming activities in software construction. Most of the time (good) developers think. Once you know the solution, typing happens _very_ quickly (and you have full IDE support for that: code completion, rename / move refactoring, ...).
Also, once you've learned a good pattern (such as how to correctly use ViewModel), developing new screens (typical ones) becomes even **mechanical**: you know the pattern, it works, you just blindly follow it.
That approach has a lot of Benefits: code looks the same no matter who worked on a particular screen, so you feel like at home and grok new screen pretty quickly; bugs have no place to hide; you can onboard new devs and get them up speed pretty quickly since all they have to do is simply follow the pattern (in the meantime learning ideas behind it, of course.)



#####


##### Statement 2:
> Defining explicit boundaries between view and presentation logic is a virtue.

Why?
Even more advanced developers tend to quickly fall into God Object / massive view controller pit. Making `ViewModel` just data bag prevents that trap.

##### Statement 3:
> We can leverage the power of data binding since it's wide-spread.






#### Benefits:
- testing
- indep of frameworks


How ViewModel is connected with everything else? Is it even a part of MVVM (if yes, where presentation logic goes?)

ViewModel, View state, ...

View is just a bunch of UI controls, and is platform-specific.
View should not know details about how it's controlled.
View observes changes from ViewModel.
View channels changes from user to ViewModel.
View is typically written in a platform-specific way, file, ... 
Interprets user gestures.

ViewModel does not have a direct reference to view.
View is defined declaratively.

>The term means "Model of a View", and can be thought of as abstraction of the view, but it also provides a specialization of the Model that the View can use for data-binding.  In this latter role the ViewModel contains data-transformers that convert Model types into View types, and it contains Commands the View can use to interact with the Model.


>"The controller layer frequently contains many lines of code. To make this quantity of code more manageable, it is sometimes useful to subdivide the controller layer further into “model-controllers” and “view-controllers”.

>A model-controller is a controller that concerns itself mostly with the model layer. It “owns” the model; its primary responsibilities are to manage the model and communicate with any view-controller
objects. Action methods that apply to the model as a whole will typically be implemented in a model-controller. The document architecture provides a number of these methods for you; for example, NSDocument automatically handles action methods related to saving files.
A view-controller is a controller that concerns itself mostly with the viewlayer. It “owns” the interface (the views); its primary responsibilities are to manage the interface and communicate with the
model-controller. Action methods concerned with data displayed in a view will typically be
implemented in a view-controller."

>Instead of the controller of the MVC pattern, or the presenter of the MVP pattern, MVVM has a binder. In the view model, the binder mediates communication between the view and the data binder.

>by removing virtually all GUI code ("code-behind") from the view layer.[3] Instead of requiring user experience (UX) developers to write GUI code, they can use the framework markup language (e.g., XAML) and create data bindings to the view model, which is written and maintained by application developers. The separation of roles allows interactive designers to focus on UX needs rather than programming of business logic.

>The result is the model and framework drive as much of the operations as possible, eliminating or minimizing application logic which directly manipulates the view (e.g., code-behind).

ViewModel survives configuration changes


ViewModel is an abstraction of view.



With introduction of Android Architecture Components, their ViewModel is misused (and actually, misdesigned).

`this.props` in React
Silverlight, WPF

> In short, the UI part of the application is being developed using different tools, languages and by a different person than is the business logic or data backend.

> Model/View/ViewModel also relies on one more thing:  a general mechanism for data binding.
