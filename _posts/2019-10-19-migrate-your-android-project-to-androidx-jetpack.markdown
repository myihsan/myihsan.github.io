---
layout: post
title: Migrate Your Android Projects to AndroidX (Jetpack)
---

It's frustrating to know there are still so many Android projects that are not using Jetpack, even the projects created after the Jetpack released.

The original title that came up with me is

> Start your new Android project with Jetpack (AndroidX)

But after trying to create a new project with Android Studio (3.5.1), I found out that the option of `Use androidx.* artifacts` is checked by default, and we cannot uncheck it. It seems like using Jetpack is not just a recommendation but a requirement for a new project now.

So let's talk about migrating an existing project to AndroidX.

## Why

> Jetpack is a suite of libraries, tools, and guidance to help developers write high-quality apps easier. These components help you follow best practices, free you from writing boilerplate code, and simplify complex tasks, so you can focus on the code you care about.

According to the description from [Jetpack](https://developer.android.com/jetpack), there are so many components that we can use to accelerate our development, and there is a [sample project](https://github.com/android/sunflower) illustrating best practices with Jetpack.  
I would like to talk about some of them in future posts.

You may ask

> I don't need those fancy new things, do I still need to migrate?

The answer is still positive if you are using [Support Library](https://developer.android.com/topic/libraries/support-library/index).

> You can continue to use the support library. Historical artifacts (those versioned 27 and earlier, and packaged as android.support.\*) will remain available on Google Maven. However, **all new library development will occur in the AndroidX library**.

You can confirm this fact at [MvnRepository](https://mvnrepository.com). For example [`com.android.support.appcompat-v7`](https://mvnrepository.com/artifact/com.android.support/appcompat-v7) stopped update at Sep 2018, and moved to [`androidx.appcompat.appcompat`](https://mvnrepository.com/artifact/androidx.appcompat/appcompat).

So the migration to AndroidX is an inevitable process you need to do as soon as you can for most of the projects.

## How

There is an official page of [Migrating to Android](https://developer.android.com/jetpack/androidx/migrate).

### Summary

1. Update `com.android.support.*` to 28.0.0[^1](optional)
2. **Refactor \> Migrate to AndroidX** from the menu bar of Android Studio
3. Do some manual mappings if something went wrong

- [Maven artifact mappings](https://developer.android.com/jetpack/androidx/migrate/artifact-mappings)
- [Class mappings](https://developer.android.com/jetpack/androidx/migrate/class-mappings)

If you still have issues after the steps above, you may need to update dependencies to the version that supports AndroidX.

Not that hard, right?

## Let's do it

Let's do it right away for your projects, even your company's, and I hope this post can help you to convince your superior if you have to.

## Where to Go From Here

[ViewModel—a superhero belongs to AndroidX]({% post_url 2019-11-03-viewmodel-a-superhero-belongs-to-androidx %})

***

[^1]: A version that is binary equivalent to `androidx.*` 1.0.0. We can solve some issues related to the update of Support Library before migrating to AndroidX.
