---
title: "SwiftUI - Animated Updates of List Row Subviews"
date: 2022-08-28 16:47:00 +0000
draft: false
---

There are a few things to note when trying to animate specific subviews in a List's row. I've created a [playground app](https://github.com/bogdanbolchis/ListRowUpdates) for experimenting with this, and i've put my learnings in this post.

{{< rawhtml >}}
<video controls muted playsinline style="max-width: 40%">
	<source src="/animating-list-row-subviews.mp4" type="video/mp4">
</video>
{{< /rawhtml >}}

### Components

The [playground app](https://github.com/bogdanbolchis/ListRowUpdates/blob/main/ListRowUpdates.swiftpm/ContentView.swift/) contains the following components:

- a "store" to hold the list's rows
- the Row data type to hold `text`, `detail`, `updatesCount` and other values, used to populate the list
- a ContentView with a List, and a `.refreshable` modifier applied for the pull-to-refresh behaviour

The components' behaviour:

- the store object is part of the ContentView's state. Changing it will trigger a ContentView redraw
- when pulling-to-refresh the List, the store simulates a background update using a detached Task
	- one random row in the store is mutated by incrementing its `updatesCount` property
	- the store's rows are updated on the main dispatch queue (or UI thread, where UI updates must happen)

State changes (like the store) are monitored by the ContentView and trigger a redraw. During the redraw, the system presumably compares the previous and new states and decides which parts of the view hierarchy to replace, and which parts to skip. 

### Making it work

To animate a row's `"Updates: <count>"` Text view, several additions need to be made:

1. add the `.animation` modifier to the List, specifying `store.rows` as the `value` that should be monitored for changes
2. add the `.transition` modifier to the view that will change (in this case one of the row's Text views), specifying the kind of transition that should happen, e.g. a scale & opacity transition
3. set the `.id` modifier on the view in question, so that SwiftUI can identify where in the view hierarchy the transition should happen

#### Notes on `.id`

The docs for the `.id` modifier mention the following: 

```
id - Binds a view’s identity to the given proxy value.

When the proxy value specified by the id parameter changes, the identity of the view - for example, its state — is reset.
```

The value used with the Text view's `id` modifier should be:
- unique among the list's rows
- distinct for the removed and the inserted rows

One value that satisfies both requirements above could be `row.text + row.detail`. When a row gets updated, the `id` for the removed Text view is `BicycleUpdates: 0`, and `BicycleUpdates: 1` for the new Text view. The system will replace one view with another, animating the transition.

View identity is important in a SwiftUI view hierarchy because if it's not reset, the system can skip redrawing that part of the hierarchy, which means less work and faster UI updates.

Note: `row.id` is not sufficient to animate the transition because it is constant for a row with its `updatesCount` incremented.

### Updating insertion/removal/moving of entire rows

This is simpler to do than animating a certain subview of a row, because the system can identify rows directly. For this purpose, it's sufficient for the Row struct to adopt the `Identifiable` protocol. Any change in the `rows` array will be reflected in the List.

These changes are also necessary in the playground app:

- to see the animated removal of a row, comment out the insertion: 
	- `// rows.insert(randomRow, at: index)` 
- to see the animated insertion, comment out the removal:
	-  `// rows.remove(at: index)`
	- note that the console will show errors because multiple rows now have the same `id`. To fix this, set a new `id` for the `randomRow`:
		- `randomRow.id = UUID().uuidString`
		
### Other resources

- [List](https://developer.apple.com/documentation/swiftui/list)
- [Stack Overflow](https://stackoverflow.com/a/60136737)
- [Delaying an asynchronous Swift Task](https://www.swiftbysundell.com/articles/delaying-an-async-swift-task/)