---
layout: post
title:  "Event delegation with multiple sources and multiple handlers"
author: "Tuna"
comments: false
category: Leetcode
tags: leetcode python optimization
excerpt_separator: <!--more-->
sticky: false
hidden: false
draft: true
---

> A simple note on strategy for delegating events from UI to handlers.

<!--more-->

For client app development, regardless whether mobile, desktop, or web app, there are lots of times we need to handle users' inputs or interactions. The flow looks like this:
1. The user interacts with a UI component (eg. button, text field, scrollable container, etc.)
2. The UI component fires a UI Event
3. The UI event is captured by the view holder (the view holder belongs to View layer in all MVC, MVVM, MVP, etc.)
4. The view holder notifies the event to lower level layer such as View Model in MVVM, Controller in MVC, Presenter in MVP, and so on
5. The View Model/Controller/Presenter might handle the event or keep delegate the event to deeper layer, such as Data controller, Model, etc.

TODO: Draw diagram

When the UI component only fires only one or two or, maybe, three kinds of UI events, or the events can be handled by ther intermediate layer such as within the View Holder, View Controller, the event handling approach is similar to this:

```kotlin
// ViewHolder
init {
    openFooScreenButton.setOnClickListener {
        openFooScreen()
    }
    confirmButton.setOnClickListener {
        markConfirmation()
    }
}

private fun openFooScreen() {
    // snippet
}

private fun markConfirmation() {
    // snippet
}
```

