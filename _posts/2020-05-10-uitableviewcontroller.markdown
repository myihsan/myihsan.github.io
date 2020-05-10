---
layout: post
title: UITableViewController
---

> Create a custom subclass of UITableViewController for each table view that you manage.

This is a suggestion in the document of `UITableViewController`. We can indeed ignore this suggestion since it's more flexible to implement the `UITableViewDataSource` and the `UITableViewDelegate` directly, but the `UITableViewController` does more than set the data source and the delegate.

## Behaviors Implemented by UITableViewController

If there is any behavior implemented by `UITableViewController` that it's hard for us to implement by ourselves, we should listen to the suggestion.

So let's check the behaviors one by one.

> It automatically loads the table view archived in a storyboard or nib file. Access the table view using the tableView property.

We can achieve this behavior by `@IBOutlet`.

> It sets the data source and the delegate of the table view to self.

Of course, we can set them by ourselves.

> It implements the viewDidAppear(_:) method and automatically flashes the table view's scroll indicators when it first appears.

This is a reasonable way to tell the user that the content is scrollable, but we can also achieve by ourselves with one line code:

``` swift
tableView.flashScrollIndicators()
```

> It implements the setEditing(_:animated:) method and automatically toggles the edit mode of the table when the user taps an Edit\|Done button in the navigation bar.

We do not always need this behavior, and we can achieve it by `UINavigationItem` if we need it.

> It automatically resizes its table view to accommodate the appearance or disappearance of the onscreen keyboard.

We also do not always need this behavior, and we can achieve it by `UITableView.contentInset` if we need it.

> It implements the viewWillAppear(_:) method and automatically reloads the data for its table view on first appearance. It clears its selection (with or without animation, depending on the request) every time the table view is displayed; you can disable this behavior by changing the value in the clearsSelectionOnViewWillAppear property.

The last one, it seems like we can achieve this behavior by:

``` swift
override func viewWillAppear(_ animated: Bool) {
    ...
    if let indexPathForSelectedRow = tableView.indexPathForSelectedRow {
        tableView.deselectRow(at: indexPathForSelectedRow, animated: true)
    }
}
```

But if we try to use the interactive pop gesture, we will find out that `UITableViewController` will change the selection state gradually along with the gesture instead of deselecting the row once `viewWillAppear` called like the implementation above. And this behavior even works on a custom `UITableViewCell.selectedBackgroundView`.

Sure that we can observe the `UINavigationController.interactivePopGestureRecognizer` and get the pop progress from `UIViewController.transitionCoordinator?.percentComplete` to modify `UITableViewCell.selectedBackgroundView` according to the progress. But this implementation is a little complicate, we had better to use `UITableViewController` since it's added from iOS 3.2 which means it's widely tested.

Finally, here is a reason to listen to that suggestion.

## Demerits

However, you may think that there are also some demerits when using `UITableViewController`. Let's check if we can overcome them effortlessly.

### The Root View is a UITableView

If we use a `UITableView`, we can add it to the root view and layout it as we want. But when we use `UITableViewController`, the `UITableView` is the root view, the only thing we can do is to add other views on the `UITableView`.

However, we can overcome this by making the `UITableViewController` as a child view controller than layout it's `view` as we want just like we layout a `UITableView`.

``` swift
addChild(tableViewController)
view.addSubView(tableViewController.tableView!)
// Set frame or Auto Layout constrants
tableViewController.didMove(toParent: self)
```

And all behaviors that `UITableViewController` implemented will still work.

### Have to Make a Subclass for Each UITableView

Since the `UITableViewController` itself is the `UITableViewDataSource` and the `UITableViewDelegate` of its `UITableView`, so we may have to make a subclass of `UITableViewController` for a different `UITableViewDataSource`/`UITableViewDelegate`.

However, we can create a common subclass to overcome this demerit since our goal is to use `UITableViewController` like a `UITableView`.

``` swift
class TableViewController: UITableViewController {
    var dataSource: UITableViewDataSource?
    var delegate: UITableViewDelegate?
    // Override all methods of UITableViewDataSource and UItableViewDelegate
    // to forward to `dataSource` and `delegate`
}
```

### Other Demerits

There may be other demerits that I haven't noticed, but with the code above, we can use `UITableViewController` like a `UITableView`, so there should be some ways to overcome them painlessly.

## A Tip for Popover or iOS 13 Style Modal

You have noticed that a popover or iOS 13 style modal breaks the behavior of clearing the selection since dismissing such a modal will not call `viewWillAppear(_:)`.

We can call `viewWillAppear(_:)` by ourselves to solve this issue. This approach may not be a proper way for a normal `UIViewController` since it's a lifecycle method, but if we only use `UITableViewController` like a `UITableView`, I think it's alright.

Set `UIAdaptivePresentationControllerDelegate` before present a view controller.

``` swift
viewController.presentationController?.delegate = self
present(viewController, animated: true)
```

Call the `UITableViewController`'s `viewWillAppear(_:)` when the modal is going to dismiss.

``` swift
extension ViewController: UIAdaptivePresentationControllerDelegate {
    func presentationControllerWillDismiss(_ presentationController: UIPresentationController) {
        tableViewController.viewWillAppear(true)
    }
}
```

That's it. The behavior of clearing the selection will work again.

## Conclution

Since we can use `UITableViewController` like a `UITableView` effortlessly, let's listen to that suggestion and get all merits of `UITableViewController` for free.
