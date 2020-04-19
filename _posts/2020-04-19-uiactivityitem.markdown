---
layout: post
title: UIActivityItem
---

There is no type named `UIActivityItem`, but if you ever used the `UIActivityViewController` to share contents, you may notice that the first parameter is called `activityItems`. It's an array of `Any`. If the element of it has a type I think the name would be `UIActivityItem`.

Let's talk about the `UIActivityItem` today.

## Common Usage

``` swift
let activityItem = URL(string: "https://www.apple.com")!
let activityItems = [activityItem]
let activityViewController = UIActivityViewController(activityItems: activityItems, applicationActivities: nil)
present(activityViewController, animated: true)
```

It's simple. We also can share other things like texts, images, and files.

If the element of the `activityItems` is literally an `Any`, this post will end here. But it's not, here is a class named `UIActivityItemProvider`.

## UIActivityItemProvider

Like its name, this class can provide the actual item to share. The difference with set the item to `activityItems` directly is the timing to create the item.

To set the item to `activityItems` directly, we have to create the item first, with `UIActivityItemProvider` we can start creating the item after the user selected a destination to share.

If creating the item is a time-consuming task, this little timing change can make a big improvement for the user who opened the `UIActivityViewController` by mistake.

We have to make a subclass since all properties of it are get-only.

``` swift
class ActivityItemProvider: UIActivityItemProvider {
    override var item: Any {
        // Create the item here
    }
}
```

The getter of the `item` will not block the main thread, so it's better to make a progress indicator for the user. A HUD may be handy in this situation, here is a [post]({% post_url 2020-04-12-make-a-heads-up-display-like-the-one-in-books-app %}) that discuss the HUD.

And for the same reason, the user can cancel the sharing by a drag-down gesture or tapping the close button. We can check the result of the sharing this way:

``` swift
activityViewController.completionWithItemsHandler = { activityType, completed, returnedItems, error in
    if !completed {
        // Dismiss the progress indicator here
    }  
}
```

You may already notice that we have to provide a `placeholderItem` to create a `UIActivityItemProvider`. It's reasonable since the `UIActivityViewController` doesn't know what to display in the preview header.

| <img src="/assets/images/uiactivityviewcontroller-preview-header.png" alt="HUD by UIAlertController" width="200"/> |
|:-:|
| Preview header |

However, it seems like only work properly with a URL. But we can control the content to display in the preview header by `UIActivityItemSource`.

## UIActivityItemSource

`UIActivityItemSource` is the protocol that `UIActivityItemProvider` adopted. So we should be able to achieve what we have done by `UIActivityItemProvider` using `UIActivityItemSource`.

``` swift
class ActivityItemSource: NSObject, UIActivityItemSource {
    func activityViewControllerPlaceholderItem(_ activityViewController: UIActivityViewController) -> Any {
        // Placeholder item
    }

    func activityViewController(_ activityViewController: UIActivityViewController, itemForActivityType activityType: UIActivity.ActivityType?) -> Any? {
        // Actual item
        // We even can create the item depends on the `activityType`
    }
}
```

However, `activityViewController(_:dataTypeIdentifierForActivityType:)` will block the main thread, so when we need to run a time-consuming task to create the item, we had better to subclass from `UIActivityItemProvider` and override the method we need instead of adopting `UIActivityItemSource` directly.

To customize the preview header, we have to implement an optional method:

``` swift
import LinkPresentation
class ActivityItemProvider: UIActivityItemProvider {
    ...
    override func activityViewControllerLinkMetadata(_ activityViewController: UIActivityViewController) -> LPLinkMetadata? {
        let linkMetadata = LPLinkMetadata()
        linkMetadata.title = "Apple" // Title
        linkMetadata.originalURL = URL(string:"https://www.apple.com/")! // Domain
        let imageProvider = NSItemProvider(contentsOf: Bundle.main.url(forResource: "apple", withExtension: "jpg"))
        linkMetadata.imageProvider = imageProvider // Icon image
        return linkMetadata
    }
}
```

Since this API use `LinkPresentation` to display the preview header, this usage may look a little weird when we provide a non-URL item. But setting the title and the icon image should be enough in most cases.

## Conclusion

Instead of always setting the `activityItems` for `UIActivityViewController` directly, let's keep it in mind that we can use `UIActivityItemProvider` or `UIActivityItemSource` to provide a better user experience.
