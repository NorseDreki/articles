MV*: Mitigating impedance mismatch between view and model; say welcome to Mapper
=======

> TL;DR Even though it is technically possible to bind Model to view and even though there might (and possibly would) a 1:1 field correspondence between them, you should understand this is only in the beginning, so the statement is: Model and ViewModel (data to be displayed in view, whatever you call it) are two separate concerns

#### Pillar Statement 1:
**Model and ViewModel (data to be displayed in view, whatever you call it) are two separate concerns**

***

Then we apply the same Principle we've been using: given this strong statement, we try to find a solution with it in mind.

So, layer of business (or application) logic, let's call it Service, returns us a Model. Again, Model does not care how UI looks like. Model participates in solving business problems and that has higher importance than how data is presented on screen. Screen presentation, size, platform might change but that must not affect goodfunction of business logic. With that said we derive that presentation logic depends (and is less important) on business (what a surprise).

Even though is it a well-known dependency direction, many people still don't get all the implications of this.

***

#### Pillar Statement 2:
**Since our Presenter receives Model object from business layer and also should handle its view corresponding data, there should be something which transforms Model object into view data (ViewModel)**

(If you are not sure why we need to transform Model to smth else, please refer to Pillar Statement 1.)

So we named out a responsibility: "Transform or map Model objects to a form usable by view". We could have put corresponding code to Presenter, but be careful! We don't want to violate SRP and separation of concerns.

Thus we dedicate a class for that, let's call it Mapper (you can pick whichever name, doesn't matter).

(Automapper ?)

So Mapper is stuff which takes Model and returns ViewModel. We can define its interface as follows:

```Kotlin
interface Mapper<in M, out VM : ViewModel> {
  fun map(model: M): VM
}
```

(Since it's SAM, define a type alias for function - it is better; no need for classes)

(`typealias` does not allow type bounds)

***

## Buts:
###### But creating dedicated entity Mapper requires too much abstraction

I hear it around so many times. But nobody from sayers dares to define how much is "too much"? Why is it too much? Silence is the answer.

To them, they'd rather put everything in one big class (or several ones) -- God Object. This is their level of comfort. Everything on top of that is called "too much abstraction" or "over-engineering".

So, since we have defined a clear responsibility, it is enough reason to spinn off a Mapper thing.

###### But writing mapping code is boring / too much to be written. Most of the fields are basically the same, so it's dumb typing.

If something seems to be boring to write is means it's quite mechanical. It means you already know how to write it which means quite a few time is required for that. Since actually writing code is one of the least time consuming activities -- this is not an argument at all, that something takes time to write it down.

Most of the time (80%) people read code, so structure and clarity have much higher priority. If you don't put mapping code into its own ...

structure matters! If developer wants to skip mapping code, he doesn't open corresponding file. Instead he focuses on smth else. Compare that to having to keep mapping code in Presenter where it gets in the way of understanding...

What happens if you skip mapping code and just use Model to back up view? Pretty soon you end up putting a field such as `isDraft` to ViewModel.
Then a developer working around business logic (and who is not really concerned with UI part of the app) will be wasting his time trying to understand where `isDraft` coming from. He will spend time realizing it is calculated and used only in view. This efforts stack up when more fields are added.

So, not only you do not separate concerns but also you make life of your pals harder, forcing them to look around and think why was the reason to add this field to Model. Don't screw your teammates.

Here is an example of mapper.
And mappers can be easily tested.
