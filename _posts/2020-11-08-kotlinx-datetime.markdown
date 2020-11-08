---
layout: post
title: kotlinx-datetime
---

KotlinX multiplatform date/time library–[`kotlinx-datetime`](https://github.com/Kotlin/kotlinx-datetime) is still under pre-release, but it has the potential to become the major date/time library in Kotlin world. So it's worth having a peek.

Let's go through some common cases using kotlinx-datetime.

## Getting Date Components

```kotlin
val instant = Clock.System.now()
val localDateTime = instant.toLocalDateTime(TimeZone.currentSystemDefault())
val month = localDateTime.monthNumber
```

A littile inconvenience since there is no `LocalDateTime.now()` like the `LocalDateTime` in Java.

## Setting Date Components

It seems like there the only way to set date components is from the constructor of the `LocalDateTime`.

Since `LocalDateTime` in kotlinx-datetime is a wrapper for `LocalDateTime` in Java, we should get this feature in a future release.

But if we only want to add an amount of date/time units for our date/time, we can use the `plus` function of `Instant`.

```kotlin
val otherInstant instant.plus(DateTimePeriod(months = 2, days = 7), TimeZone.currentSystemDefault())
```

## Comparing Two Date/Time

```Kotlin
instant.periodUntil(otherInstant, TimeZone.currentSystemDefault()) // DateTimePeriod(months = 2, days = 7)
```

Or get the difference of a particular unit like `Temporal` in Java.

```Kotlin
instant.until(otherInstant), DateTimeUnit.MONTH, TimeZone.currentSystemDefault()) // 2
```

## Conclutions for v0.1

It's great that we finally have an official date/time library. However, its features are still limited.

If we compare it with `java.time`, we can tell that they are similar and `java.time` is handier for now. But `kotlinx-datetime` also has its things such as `DateTimePeriod`, so I think it's expectable that `kotlinx-datetime` may catch up or even surpass `java.time` in the future.

Let's start to keep an eye on `kotlinx-datetime` and contribute to it if we have a chance.
