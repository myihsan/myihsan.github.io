---
layout: post
title: Providing Default Value for Decodable Property by Property Wrapper
---

To provide a default value for a `Decodable` type's properties, we need to define `CodingKeys` and use it to implement `init(from:)`.

```swift
enum Bar: String, Decodable {
    case case1, case2
}

struct Foo: Decodable {
    let array: [Int]
    let string: String
    let int: Int
    let double: Double
    let bar: Bar

    private enum CodingKeys: CodingKey {
        case array, string, int, double, bar
    }

    init(from decoder: Decoder) throws {
        let container = decoder.container(keyedBy: CodingKeys.self)
        array = decoder.decodeIfPresent([Int].self, forKey: .array) ?? []
        string = decoder.decodeIfPresent(String.self, forKey: .string) ?? ""
        int = decoder.decodeIfPresent(Int.self, forKey: .int) ?? 0
        double = decoder.decodeIfPresent(Double.self, forKey: .double) ?? -1
        bar = decoder.decodeIfPresent(Bar.self, forKey: .bar) ?? .case2
    }
}
```

It's easy but troublesome since we have to write more than treble code even there is only one property we want to provide a default value.

Moreover, we have to define a normal initializer and provide default values again if we need.

```swift
struct Foo: Decodable {
    ...
    init(array: [Int] = [], string: String = "", int: Int = 0, double: Double = -1, bar: Bar = .case2) {
        self.array = array
        self.string = string
        self.int = int
        self.double = double
        self.bar = bar
    }
}
```

This time the amount of code is not the only problem. We may set different default values than `init(from:)` by mistake.

Property wrapper comes to rescue.

Let's define a protocol that holds a default value.

```swift
protocol Default {
    associatedtype Value

    static var defaultValue: Value { get }
}
```

With this protocol, we can define the property wrapper to provide the default value.

```swift
@propertyWrapper
struct DefaultDecodable<D: Default>: Decodable where D.Value: Decodable {
    let wrappedValue: D.Value

    init(from decoder: Decoder) throws {
        let container = try decoder.singleValueContainer()
        wrappedValue = try container.decode(D.Value.self)
    }

    init() {
        wrappedValue = D.defaultValue
    }
}
```

You may notice that we only use the default value in `init()`, but how can we use it when decoding? Here is the magic.

```swift
extension KeyedDecodingContainer {
    func decode<D>(_ type: DefaultDecodable<D>.Type,
                   forKey key: Key) throws -> DefaultDecodable<D> {
        try decodeIfPresent(type, forKey: key) ?? .init()
    }
}
```

With this overload, we can provide the default value for the property using `DefaultDecodable`.

Let's define a protocol to provide an empty default value.

```swift
protocol EmptyInitializable {
    init()
}

extension Array: EmptyInitializable {}
extension String: EmptyInitializable {}
```

Then implement `Default` for `EmptyInitializable`.

```swift
struct EmptyDefault<Value: EmptyInitializable>: Default {
    static var defaultValue: Value {
        .init()
    }
}
```

Now we can use them this way.

```swift
struct Foo: Decodable {
    @DefaultDecodable<EmptyDefault> var array: [Int]
    @DefaultDecodable<EmptyDefault> var string: String
    ...
}
```

It's a little tedious, let's organize them.

```swift
enum CommonDefault { // Just for a namespace
    struct Empty<Value: EmptyInitializable>: Default {
        static var defaultValue: Value {
            .init()
        }
    }
}

enum DefaultBy {
    typealias Empty<Value: Decodable & EmptyInitializable> = DefaultDecodable<CommonDefault.Empty<Value>>
}
```

Then we can use them this way.

```swift
struct Foo: Decodable {
    @DefaultBy.Empty var array: [Int]
    @DefaultBy.Empty var string: String
    ...
}
```

It's better. Keep going for `Numeric` types.

```swift
enum CommonDefault {
    ...
    struct Zero<Value: Numeric>: Default {
        static var defaultValue: Value {
            .zero
        }
    }

    struct One<Value: Numeric>: Default {
        static var defaultValue: Value {
            1
        }
    }

    struct MinusOne<Value: Numeric>: Default {
        static var defaultValue: Value {
            -1
        }
    }
}

enum DefaultBy {
    ...
    typealias Zero<Value: Decodable & Numeric> = DefaultDecodable<CommonDefault.Zero<Value>>

    typealias One<Value: Decodable & Numeric> = DefaultDecodable<CommonDefault.One<Value>>

    typealias MinusOne<Value: Decodable & Numeric> = DefaultDecodable<CommonDefault.MinusOne<Value>>
}
```

With them, it would be enough for most of the cases.

```swift
struct Foo: Decodable {
    ...
    @DefaultBy.Zero var int: Int
    @DefaultBy.MinusOne var double: Double
    ...
}
```

Finally, let's handle the case that a type has its own default value.

```swift
protocol SelfDefault: Default {
    static var defaultValue: Self { get }
}

extension Bar: SelfDefault {
    var defaultValue: Bar { .case2 }
}

enum DefaultBy {
    ...
    typealias `Self`<Value: Decodable & SelfDefault> = DefaultDecodable<Value>
}
```

But this is impossible since the type implemented `SelfDefault` may be itself or its subclass.

Let's use itself expressly.

```swift
protocol SelfDefault { // No need to inherit `Default` anymore
    static var defaultValue: Self { get }
}

enum CommonDefault {
    ...
    struct `Self`<Value: SelfDefault>: Default {
        static var defaultValue: Value {
            Value.defaultValue
        }
    }
}

enum DefaultBy {
    ...
    typealias `Self`<Value: Decodable & SelfDefault> = DefaultDecodable<CommonDefault.Self<Value>>
}
```

That's it. Now our `Foo` is far more straightforward, and the auto-generated initializer still exists.

```swift
struct Foo: Decodable {
    @DefaultBy.Empty var array: [Int]
    @DefaultBy.Empty var string: String
    @DefaultBy.Zero var int: Int
    @DefaultBy.MinusOne var double: Double
    @DefaultBy.Self var bar: Bar
}
```

The final result is [here](https://github.com/myihsan/DefaultDecodableWrapper).

## Conclusions

This kind of issue should be fixed by the server-side, but if we don't control the server-side, a property wrapper can help us deal with it elegantly.

The power of property wrapper may be beyond our imagination, so when some issues may be solved by a getter or/and a setter, let's give it a try to use property wrappers.
