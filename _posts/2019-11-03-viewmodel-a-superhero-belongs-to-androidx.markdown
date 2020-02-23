---
layout: post
title: ViewModel—a Superhero Belongs to AndroidX
---

Have you ever afraid of configuration changes so that even disable rotation and split-screen for your app to avoid it? `ViewModel` is a superhero to help us out from it.

## Power

### Allows Data to Survive Configuration Changes

> The ViewModel class allows data to survive configuration changes such as screen rotations.  
>
> From: [ViewModel Overview](https://developer.android.com/topic/libraries/architecture/viewmodel)

Let's review the ways to allows data to survive configuration changes without `ViewModel`.

#### onSaveInstanceState(Bundle)

Save our data to a `Bundle` when `onSaveInstanceState(Bundle)` and retrieve it in any lifecycle method that holds `savedInstanceState` like `onCreate(Bundle)`.

It's simple to do this if our data is one of the basic types. But when it comes to a class we created, we need to implement `Parcelable` for our class.

It's not that hard but still needs some works.

#### Hold Our Data Outside of Activity

Holding our data in the other class, for example, `Presenter` in MVP (Model–view–presenter) pattern, enable our data to elapse from `Activity`'s lifecycle to survive configuration changes.

There is no need to implement `Parcelable`, easy, right?

Not really. In this kind of approach, we need to manage the lifecycle of our data by ourselves, and if we do it wrong, it will end up with memory leaks.

#### What About ViewModel

`ViewModel` uses the second approach with a handy lifecycle.

![The lifecycle of a ViewModel](/assets/images/viewmodel-lifecycle.png)

With this lifecycle, we can simply hold our data in `ViewModel`, and then our data will be released after `Activity` finished. If we have some extra release works to do, we can do it in `onCleared()`.

### ViewModelScope

If we are using Kotlin coroutines to deal with our asynchronous tasks, `ViewModelScope` is definitely another power of `ViewModel` that can relieve us from some duplicated works.

We can use `viewModelScope` property to launch coroutines that we want to cancel with `ViewModel`, as shown in the following example:

``` kotlin
viewModelScope.launch(IO) {
    // Any coroutine will be canceled when ViewModel will be cleared
}
```

We don't need to manage our coroutine scope to cancel it in `onCleared()` anymore.

## How to Use

Because `ViewModel` has its life cycle and it's related to an `Activity` or a `Fragment`, we need to use `ViewModelProviders` with an instance of `Activity` or `Fragment` to get it as follow:

``` kotlin
override fun onCreate(savedInstanceState: Bundle?) {
    // Create a ViewModel the first time the system calls an activity's onCreate() method.
    // Re-created activities receive the same MyViewModel instance created by the first activity.

    // Use the 'by viewModels()' Kotlin property delegate
    // from the activity-ktx artifact
    val model: MyViewModel by viewModels()
}
```

## Relation with MVVM (Model–View–ViewModel) Pattern

Is this class related to MVVM pattern?

Yes and no. It's recommended to use `ViewModel` as the viewmodel in MVVM pattern by [Guide to app architecture](https://developer.android.com/jetpack/docs/guide). However, we don't have to understand the concepts of MVVM, but just take it as a tool to deal with configuration changes.

## Where to Go From Here

In this post, I only introduce the abilities of `ViewModel` itself. With `LiveData` and MVVM pattern, `ViewModel` will be more powerful and may change your way of developing an Android app completely. I may write some posts to talk about it, but if you are eager for learning something right now, you can refer the links below:

- [ViewModel Overview](https://developer.android.com/topic/libraries/architecture/viewmodel)
- [LiveData Overview](https://developer.android.com/topic/libraries/architecture/livedata)
- [Guide to app architecture](https://developer.android.com/jetpack/docs/guide)
