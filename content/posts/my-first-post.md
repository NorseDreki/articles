---
title: "My First Post"
date: 2023-05-21T16:21:28+03:00
draft: false
---

# Ghggghjkghjg kjhgkjhgk jgh jkghk gkjhg gk ggh

kjhgkhjgkjhgkjhgjh kjg khjg khgkg hkjgkg jkhg khjgjkhg k kg jh.

## Problems

#### Time spent to write `Adapter`s and `ViewHolder`s

That would be insignificant for small apps, but if your app contains more that a dozen of screens, `Adapter`s stack up and form a big chunk of code.

#### Coupling to the Android Framework

To quote the [Android's Guide to App Architecture](https://developer.android.com/topic/libraries/architecture/guide):

>The most important thing you should focus on is the separation of concerns in your app. It is a common mistake to write all your code in an Activity or a Fragment. Any code that does not handle a UI or operating system interaction should not be in these classes. Keeping them as lean as possible will allow you to avoid many lifecycle related problems. Don't forget that you don't own those classes, they are just glue classes that embody the contract between the OS and your app.

Even though this paragraph speaks about `Activity`s and `Fragment`s, it is very applicable to `Adapter`s as well.

With that said, `Adapter`s belong to the Android framework, so putting application logic in them is not a good idea since it creates tight coupling and leads to a non-scalable application structure.