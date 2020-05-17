---
layout: post
title: No More Manual Underscore Prefix in Swift
---

In some languages, we may have to use underscore prefix for properties. Typical usage is to keep the modifiability inside and keep the accessibility outside. If you are not familiar with Swift, the following code may come to mind.

``` swift
class Foo {
    private var _bar: Int
    var bar: Int { _bar }
}
```

But this is unnecessary in Swift.

## Lower Access Level for Setter

We can give a setter a lower access level than its corresponding getter like below:

``` swift
class Foo {
    private(set) var bar: Int { _bar }
}
```

But what about the modifiability is determined by the type? For example, `CurrentValueSubject` and `AnyPublisher` in Combine or `BehaviorRelay` and `Observable` in RxSwift.

## Property Wrapper

Let's see the official sample code first.

``` swift
@propertyWrapper
struct TwelveOrLess {
    private var number: Int
    init() { self.number = 0 }
    var wrappedValue: Int {
        get { return number }
        set { number = min(newValue, 12) }
    }
}

struct SmallRectangle {
    @TwelveOrLess var height: Int
    @TwelveOrLess var width: Int
}
```

And here is a version without `@propertyWrapper`:

``` swift
struct SmallRectangle {
    private var _height = TwelveOrLess()
    private var _width = TwelveOrLess()
    var height: Int {
        get { return _height.wrappedValue }
        set { _height.wrappedValue = newValue }
    }
    var width: Int {
        get { return _width.wrappedValue }
        set { _width.wrappedValue = newValue }
    }
}
```

I thought this is just an image for us to understand the property wrapper until yesterday[^1].

However, We can actually use the underscore prefix version of our wrapped properties. So using a property wrapper can avoid creating properties with a modifiable type. Here is an example:

``` swift
@propertyWrapper
struct CurrentValueSubjectWrapper<Element> {
    private let subject: CurrentValueSubject<Element, Never>
    let wrappedValue: AnyPublisher<Element, Never>

    init(_ value: Element) {
        self.subject = CurrentValueSubject(value)
        self.wrappedValue = subject.eraseToAnyPublisher()
    }

    func send(_ input: Element) {
        subject.send(input)
    }
}
```

Then we can use it like below:

``` swift
class Thermometer {
    @CurrentValueSubjectWrapper(0)
    var temperature: AnyPublisher<Double, Never>

    func measure() {
        ...
        _temperature.send(newTemperature)
    }
}
```

We got the the underscore prefix version of the temperature for free.

## Conclusion

There may be other purposes to use an underscore prefix, but thanks to Swift syntax, we can avoid using an underscore prefix to keep the modifiability inside and keep the accessibility outside.

## References

[^1]: [propertyWrapperWithAppAndTest.swift](https://gist.github.com/marty-suzuki/cd2a97841777f2783fc2b1055c9ec445)