---
layout: post
title: Repeating Alerts like Messages
---

Messages will repeat using notification to alert us about new messages many times[^1] [^2] to ensure that we are not missing new messages. Today, let's talk about how to implement it by ourselves.

Suppose all we want to repeat is a remote notification. In that case, all we need to do is removing any keys that would trigger user interactions from the payload to silence the notification and scheduling a local notification along with a repeating notification created by a `UNTimeIntervalNotificationTrigger`.

```swift
let trigger = UNTimeIntervalNotificationTrigger(timeInterval: 60 * 5, repeats: true)
let uuidString = UUID().uuidString
let request = UNNotificationRequest(identifier: uuidString, content: content, trigger: nil)
let repeatingRequest = UNNotificationRequest(identifier: uuidString, content: content, trigger: trigger)
let userNotificationCenter = UNUserNotificationCenter.current()
userNotificationCenter.add(request) { error in
    ...
}
userNotificationCenter.add(repeatingRequest) { error in
    ...
}
```

Don't forget to remove `repeatingRequest` after the user responded to the notification or launched the app.

```swift
func userNotificationCenter(_ center: UNUserNotificationCenter, didReceive response: UNNotificationResponse, withCompletionHandler completionHandler: @escaping () -> Void) {
    let identifier = response.notification.request.identifier
    center.removePendingNotificationRequests(withIdentifiers: [identifier])
}
```

A limit of `UNTimeIntervalNotificationTrigger` for repeating notifications is that the `timeInterval` must be 60 seconds or greater.

But how to repeat a notification created by `UNCalendarNotificationTrigger`?

The `repeats` in the initialization means to reschedule the notification request each time the notification is delivered to the next cycle, which has the same `dateComponents`, so it's not what we want.

We have to change the approach. Add multiple requests to simulate the behavior of a repeating notification.

```swift
...
let calendar = Calendar.current
for offsetCount in 0 ..< 10 {
    let date = Calendar.current.date(byAdding: .minute, value: offsetCount * 5, to: someDate)!
    var dateComponents = Calendar.current.dateComponents(in: .current, from: date)
    // http://www.openradar.me/35247464
    dateComponents.quarter = nil
    let trigger = UNCalendarNotificationTrigger(dateMatching: dateComponents, repeats: false)
    let repeatingRequest = UNNotificationRequest(identifier: uuidString, content: content, trigger: trigger)
    userNotificationCenter.add(repeatingRequest) { error in
        ...
    }
}
```

Only the last `repeatingRequest` was fulfilled. It seems like we cannot use the same identifier for every `repeatingRequest`.

So let's use a different identifier for each `repeatingRequest`.

```swift
let repeatingRequest = UNNotificationRequest(identifier: "\(uuidString)@\(date.timeIntervalSince1970.description)",
 content: content, trigger: trigger)
```

Also, update the code to remove it accordingly.

```swift
func userNotificationCenter(_ center: UNUserNotificationCenter, didReceive response: UNNotificationResponse, withCompletionHandler completionHandler: @escaping () -> Void) {
    let uuidString = response.notification.request.identifier.split(separator: "@").first!
    center.getPendingNotificationRequests { requests in
        let identifiers = requests
            .map(\.identifier)
            .filter { $0.starts(with: uuidString) }
        center.removePendingNotificationRequests(withIdentifiers: identifiers)
    }
}
```

It just works, and it's even able to break the limit of 60 seconds. However, there are some side effects of this approach.

Since we are using different identifiers, every notification is count as a new one. Therefore, we should consider using `threadIdentifier` to group related notifications.

Besides, the system only keeps the soonest 64 requests[^3], so we should add requests according to their priority.

For example, cut down the repeat times.

```swift
userNotificationCenter.getPendingNotificationRequests { requests in
    let remaining = 64 - requests.count
    guard remaining > 0 else {
        // Drop the most unimportant request or let the user decide
        return
    }
    for offsetCount in 0 ..< min(remaining, 10) {
        ...
    }
}
```

## Conclusions

For a remote notification, use `UNTimeIntervalNotificationTrigger` to repeat.

For a notification to be delivered at a specific date and time, also schedule the following notifications to make it look repeating. Be careful of the limit of the number of notification requests.

***

[^1]: We can change the setting from Settings > Notifications > Messages > Repeat Alerts.
[^2]: This feature is not working on some of my devices.
[^3]: It's only documented in [UILocalNotification](https://developer.apple.com/documentation/uikit/uilocalnotification), but it seems still correct.

