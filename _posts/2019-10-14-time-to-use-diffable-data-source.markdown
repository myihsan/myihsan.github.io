---
layout: post
title: Time to Use Diffable Data Source?
---

In the WWDC 2019 session of [Advances in UI Data Sources](https://developer.apple.com/videos/play/wwdc2019/220/), Apple introduced Diffable Data Source to avoid the complexity and some exceptions like below when updating a table/collection view.

> Terminating app due to uncaught exception 'NSInternalInconsistencyException', reason: 'Invalid update: invalid number of sections. The number of sections contained in the collection view after the update (10) must be equal to the number of sections contained in the collection view before the update (10), plus or minus the number of sections inserted or deleted (0 inserted, 1 deleted).'

## Usage

Let's look at the declaration of UICollectionViewDiffableDataSource.

``` swift
class UICollectionViewDiffableDataSource<SectionIdentifierType,ItemIdentifierType> : NSObject
where SectionIdentifierType : Hashable, ItemIdentifierType : Hashable
```

The requirement is the same for other Diffable Data Source classes. We need two types that implemented `Hashable`, one for the section, one for the cell. Diffable Data Source uses them to determine which kind of update should be done.

Then create the instance of Diffable Data Source for Our table/collection view with a `CellProvider`.

``` swift
// The instance will set itself as the data source by this initialization
let dataSource = UICollectionViewDiffableDataSource
    <Section, Item>(collectionView: collectionView) {
        collectionView, indexPath, item in
        // Setup our cell
}
```

After the preparations above, we can update our table/collection view by one line code.

``` swift
dataSource.apply(snapshot, animatingDifferences: true)
```

But what about the `snapshot`?  
It's just as its name a snapshot of our table/collection view, we can get the current snapshot by `dataSource.snapshot()` then edit it or just create a new one.

``` swift
var snapshot = NSDiffableDataSourceSnapshot<Section, Item>()
snapshot.appendSections([.main])
snapshot.appendItems(items)
```

Then just `apply` it to the data source.  
The data source will do all complexity works for us and perform proper animations if we need it.

## Obstructs

The usage is simple, but there are also some obstructs we have to face.

### iOS 13/macOS 10.15 or Later

The biggest obstruct we have to face since we can just update our table/collection view with `reloadData()` without animation and exceptions caused by the animation.

## Conclusion

It's time to use Diffable Data Source if our project is targeting iOS 13/macOS 10.15 and using a static data source which has no content change for any item.

Otherwise, we need to implement the data source to accomplish our requirements or use an alternative that has accomplished our requirements.

### Alternatives

- [DiffableDataSources](https://github.com/ra1028/DiffableDataSources)
- [RxDataSources](https://github.com/RxSwiftCommunity/RxDataSources)
