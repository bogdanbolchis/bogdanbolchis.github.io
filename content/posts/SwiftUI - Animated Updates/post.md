---
title: "SwiftUI - Animated Updates of List Row Subviews"
date: 2022-08-28 16:47:00 +0000
draft: false
---

There are a few things to note when trying to apply animation to specific subviews in a List's row. I've created a [playground app](https://github.com/bogdanbolchis/ListRowUpdates) for experimenting with this, and i've put my learnings in this post.

{{< rawhtml >}}
<video controls muted playsinline style="max-width: 33%">
	<source src="/animating-list-row-subviews.mp4" type="video/mp4">
</video>
{{< /rawhtml >}}

## Components

The playground app contains the following components:

- a "store" to hold the list's rows
- the Row data type to hold `text`, `detail`, `updatesCount` and other values. These are used to populate the list
- a ContentView that contains a List, with a `.refreshable` modifier applied for the pull-to-refresh behaviour

The components' behaviour:

- the store object is part of the ContentView's state. Thus, changing it will trigger a ContentView update
- when pulling-to-refresh the List, the store simulates a background update in a detached Task
	- one random row in the store is mutated by incrementing its `updatesCount` property
	- the store's rows are updated on the main queue (where UI updates must happen)
	- an animated transition is triggered, for the Text view corresponding to the row's `updatesCount` property

## Making it work

To animate a row's `"Updates: <count>"` Text view, several changes need to be made:

1. add the `.animation` modifier to the List, passing the store's rows as the `value` that should be observed for changes
2. add the `transition` modifier to the view that will change (in this case one of the row's Text views), passing the kind of transition that should happen, e.g. a scale & opacity transition
3. set the `id` modifier on the view in question, so that SwiftUI can identify where in the view hierarchy the transition should happen

### Notes on `.id`

The docs for the `.id` modifier mention the following: 

```
id - Binds a view’s identity to the given proxy value.

When the proxy value specified by the id parameter changes, the identity of the view - for example, its state — is reset.
```

The value used for the Text view's `id` should be unique for the list's rows: `row.text + row.detail`. This value is bound to the view, and when a different value is set, the identity of the view is reset.

View identity is important in a SwiftUI view hierarchy because if it's not reset, the system can skip redrawing that part of the hierarchy, which means less work and faster UI updates.

The fact that the `id` changes (because `row.detail` changes), causes the Text element with the old `id` to be replaced by a new Text element (with the new `id`). This change in the view hierarchy is what gets animated.