---
layout: post
title: "Text and font rendering on Android"
subtitle: "Some notes about text rendering of Android"
author: "tuna"
category: Android
tags: android text-rendering
excerpt: |
  An in-depth exploration of text and font rendering techniques on Android, including font size scaling, fit font size algorithms, text drawing, and handling shadows.
excerpt_separator: <!--more-->
---
<!--more-->

## Font size scaling
Text rendering in Android does have a scale ratio. For example, *font size = 1,000 is 5 times as big as font size 200*, and 10 times as big as font size 100. The abnormal situation we have seen when getting the width and height of text through `Paint.getTextBounds()` comes from the integer properties of a Rect. There are 4 int properties of a Rect: `left`, `right`, `top`, and `bottom`. `left` and `top` are initialized by `Math.floor` while `right` and `bottom` are initialized by `Math.ceil`. As a result, we lose information on small font sizes.

After analyzing 622 fonts on my computer, I found that only 12 out of 622 fonts have a width difference of more than 20 pixels (the highest being 27 pixels) when scaling `textWidth * 5` from a font size of `1,000,000` to `Paint.getTextBounds()` with a font size of `5,000,000`.

```
               CREATURE.TTF
                 FUTRFW.TTF
               FUTRFW_0.TTF
            LaTribuneCP.ttf
              Matchbook.ttf
                monof55.ttf
           Mr Sheffield.ttf
                 Sketch.ttf
                  today.ttf
           VCR_OSD_MONO.ttf
   VNF-STRANGELOVE-TEXT.TTF
```

## Fit Font Size Finding Algorithm
The algorithm is designed to work with a large base font size. In this example, the base font size is set to 1,000,000 pixels. This large size helps to minimize the impact of rounding errors when calculating the text bounds using `Paint.getTextBounds()`. By scaling down from this large size, we can achieve more accurate results for the desired font size that fits within the specified bounds.

```java
float getFitFontSize(
    float boundWidth, float boundHeight, 
    Paint paint, Typeface typeface, String text
){
    baseTextSize = 1000000f;
    paint.setTextSize(baseTextSize);
    paint.setTypeface(typeface);
    Rect rect = new Rect();

    paint.getTextBounds(text, 0, text.length(), rect);
    float sizeW = boundWidth  * baseTextSize / (float) (rect.width());
    float sizeH = boundHeight * baseTextSize / (float) (rect.height());

    return sizeW < sizeH ? sizeW : sizeH;
}
```

## Text Drawing
Text is drawn at the baseline, which is equal to `-textBoundsRect.top` (since `top` is always negative). Additionally, we should consider `textBoundsRect.left` because the width of the text is calculated by `right - left`, and for some fonts and font sizes, `left` may not be equal to zero.

So, the drawing text should be:

```java
canvas.drawText(
    text, 
    leftMost - textBoundsRect.left, 
    topMost - textBoundsRect.top, paint
);
```

## Shadow
```java
paint.setShadowLayer(blur, offsetX, offsetY, color);
```
When shadow is turned on, we need to adjust the font size finding algorithm and the drawing method slightly.

The new algorithm should be:

```java
float getFitFontSize(
    float boundWidth, float boundHeight, 
    float offsetX, float offsetY, 
    Paint paint, Typeface typeface, String text
){
    boundWidth  -= offsetX;
    boundHeight -= offsetY;
    baseTextSize = 1000000f;
    paint.setTextSize(baseTextSize);
    paint.setTypeface(typeface);
    Rect rect = new Rect();

    paint.getTextBounds(text, 0, text.length(), rect);
    float sizeW = boundWidth  * baseTextSize / (float) (rect.width());
    float sizeH = boundHeight * baseTextSize / (float) (rect.height());

    return sizeW < sizeH ? sizeW : sizeH;
}
```

And the text drawing should be:

```java
paint.setShadowLayer(blur, offsetX, offsetY, colour);
float dx = offsetX < 0 ? -offsetX : 0;
float dy = offsetY < 0 ? -offsetY : 0;
canvas.drawText(
    text, 
    leftMost - textBoundsRect.left + dx, 
    topMost - rect.top + dy, paint
);
```

**Note:** The above code is not correct in case of `blur > 1`.

