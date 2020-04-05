---
layout: post
title: Do We Need Unit Testing?
---

There are two main reasons that make us wonder if we need unit testing.

- Some unit tests are the same as the implementation
- Integration Testing is enough to ensure functionalities

Let's verify if these reasons are reasonable.

## Some Unit Tests Are the Same as the Implementation

We may indeed encounter this situation. Here is an example.

``` swift
// Implementation
func double(_ value: Int) { value * 2 }

// Unit testing
testValues.forEach { value in
    let expectedResult = value * 2
    let result = double(value)
    XCTAssertEqual(result, expectedResult)
}
```

However, this is a misunderstanding that the unit test is the same as the implementation.

Let's see an advantage for unit testing described in [Wikipedia](https://en.wikipedia.org/wiki/Unit_testing#As_executable_specifications) first.

> ### As executable specifications
>
> Using unit-tests as a design specification has one significant advantage over other design methods: The design document (the unit-tests themselves) can itself be used to verify the implementation. The tests will never pass unless the developer implements a solution according to the design.

To achieve the advantage above, it's better to write unit tests according to the specification literally. But the implementation only needs to satisfy the specification.

So the implementation can be something like:

``` swift
func double(_ value: Int) { value + value }
```

or

``` swift
// Let's assume this implemntation has better proformance than others
func double(_ value: Int) { value * 3 - value }
```

So the truth is the implementation is the same as some unit tests coincidentally, and it's fine since they have different purposes—implementation specification and description specification.

## Integration Testing Is Enough to Ensure Functionalities

Our final goal indeed is to ensure functionalities, but Integration testing can't ensure functionalities without the prerequisite that all units do not have any unexpected behavior.

Let's assume we need to implement a function to warn a bicycle owner that his/her bicycle will be destroyed at a specific date.

Here are some requirements:

- The date should be a date after three months later from now
- Get candidate dates from a destructor plant
- Choose the most recent date

But we implemented it without the first requirement by mistake.

``` swift
class Intermediary {

    let destructorPlant: DestructorPlant
    let warner: Warner

    init(destructorPlant: DestructorPlant, warner: Warner) {
        self.destructorPlant = destructorPlant
        self.warner = warner
    }

    func warnBicycleOwner(_ owner: BicycleOwner) {
        if let date = destructorPlant.getCandidateDates().first {
            warner.warn(owner, with: date)
        }
    }
}
```

You may think that the integration test to ensure the date is after three months from now will cover this mistake for us.

But what if the destructor plant plans to suspend their business for four months from now, but they forget to inform us? Unfortunately, we will pass that test and be affected by our mistake after one month.

We can detect this kind of issue by a unit test like below.

``` swift
// This test code can be simplified a lot by Cuckoo which I listed in the conclusion
class MockDestructorPlant: DestructorPlant {
    func getCandidateDates() -> Dates {
        let now = Date()
        let afterThreeMonths = Calendar.current.date(byAdding: .month, value: 3, to: now()
        return [now, afterThreeMonths]
     }
}

class MockWarner: Warner {
    var destructionDate: Date?
    func warn(owner: Owner, with date: Date) {
       destructionDate = date
    }
}

let mockDestructorPlant = MockDestructorPlant()
let mockWarner = MockWarner()
let intermediary = Intermediary(destructorPlant: mockDestructorPlant, warner: mockWarner)
let now = Date()
let afterThreeMonths = Calendar.current.date(byAdding: .month, value: 3, to: now()
intermediary.warnBicycleOwner(owner)
guard let destructionDate = mockWarner.destructionDate else {
    XCTFail("warn(owner:with:) wasn't called")
}
XCTAssertGreaterThanOrEqual(destructionDate, afterThreeMonths)
```

Let's write unit testing to cover some issues that integration testing can't. So that we don't have to make compensation like my apartment's staff for bike owners affected by the issue[^1].

## Conclution

We can say the two reasons that make us wonder if we need unit testing aren't reasonable, we do need unit testing.

Here are some libraries that can help us to write unit testing with Swift.

- [Quick](https://github.com/Quick/Quick)
- [Cuckoo](https://github.com/Brightify/Cuckoo)
- [SwiftCheck](https://github.com/typelift/SwiftCheck)

***

[^1]: Though I have made some changes in some details, it's a true story that happened in my apartment.
