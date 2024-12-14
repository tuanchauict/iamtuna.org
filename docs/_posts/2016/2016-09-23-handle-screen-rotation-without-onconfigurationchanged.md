---
layout: post
title:  "Handle screen rotation without onConfiguration&shy;Changed"
author: "Tuna"
comments: false
category: Android
tags: android screen-orientation
excerpt: |
  Learn how to handle screen rotation in Android without using `onConfigurationChanged`, by leveraging `OrientationEventListener` to create responsive views that adapt to orientation changes smoothly.
excerpt_separator: <!--more-->
sticky: false
hidden: false
draft: false
---

Especially for camera views, we need to implement a view that can adapt to the screen’s orientation without relying on the native Android screen rotation handler. To achieve this, we create an instance of [OrientationEventListener](http://developer.android.com/reference/android/view/OrientationEventListener.html) with an abstract method `onOrientationChanged`. This method takes a parameter representing the screen's degree from _**0**_ to _**359**_, and _**-1**_ for screen flipping.

Example:

```java
// MainActivity OrientationEventListener mListener;
void onCreate(Bundle savedInstanceState){
    ...

    mListener = new OrientationEventListener(this){
        public void onOrientationChanged(int orientation){
            System.out.println(orientation);
        }
    };
}
```

From this point, I created two shortcut classes based on `OrientationEventListener`.

The first class is `SimpleOrientationEventListener`, which calculates the current screen orientation and triggers the `onChanged` abstract method in `onOrientationChanged`. Below is the method.

```java
@Override
public final void onOrientationChanged(int orientation) {
    if (orientation < 0) {
        return; // Flip screen, Not take account
    }

    int curOrientation;

    if (orientation <= 45) {
        curOrientation = ORIENTATION_PORTRAIT;
    } else if (orientation <= 135) {
        curOrientation = ORIENTATION_LANDSCAPE_REVERSE;
    } else if (orientation <= 225) {
        curOrientation = ORIENTATION_PORTRAIT_REVERSE;
    } else if (orientation <= 315) {
        curOrientation = ORIENTATION_LANDSCAPE;
    } else {
        curOrientation = ORIENTATION_PORTRAIT;
    }
    if (curOrientation != lastOrientation) {
        onChanged(lastOrientation, curOrientation);
        lastOrientation = curOrientation;
    }
}
```
The second class is `RotateOrientationEventListener`, which calculates the appropriate rotation degrees for smooth view rotation. This class extends `SimpleOrientationEventListener` and uses the `onChanged` method to determine the rotation, then triggers the `onRotateChanged` abstract method.

```java
@Override
public final void onChanged(int lastOrientation, int orientation) {
    int startDeg = lastOrientation == 0
            ? orientation == 3 ? 360 : 0
            : lastOrientation == 1 ? 90
            : lastOrientation == 2 ? 180
            : 270; // don't know how, but it works
    int endDeg = orientation == 0
            ? lastOrientation == 1 ? 0 : 360
            : orientation == 1 ? 90
            : orientation == 2 ? 180
            : 270; // don't know how, but it works

    onRotateChanged(startDeg, endDeg);
}
```
You can find the [full code for both classes here](https://gist.github.com/tuanchauict/6a885779c0940a012b81).

To test the code, set `screenOrientation` to `landscape` or `portrait` in `AndroidManifest.xml` or use `setRequestedOrientation` in the activity.

> *Bonus: This is an example of how to apply the startDeg and endDeg above to views:*

```java
private void rotateView(View view, int startDeg, int endDeg) {
    view.setRotation(startDeg);
    view.animate().rotation(endDeg).start();
}
```