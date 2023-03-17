---
layout: post
title: "Text and font rendering on Android"
subtitle: "Some notes about text rendering of Android"
author: "tuna"
category: Android
tags: android text-rendering
excerpt_separator: <!--more-->
---
> *Some notes about text rendering of Android*

<!--more-->

## Font size scaling
Text rendering in Android does have scale ratio. For example, *font size = 1,000 is 5 times as big as font size 200*, and 10 times as big as font size 100. The abnormal situation we have seen when get width and height of text through `Paint.getTextBounds()` comes from the integer. There are 4 int properties of a Rect: `left, right, top, bottom`. `left` and `top` are initiated by `Math.floor` while right and bottom are initiated by Math.ceil. As the result, we lost information on small font size.

After doing stats with 622 fonts on my computer, I experience that only 12/622 fonts have the width value different more than 20 pixels (the highest is 27 pixels) on scaling `textWidth * 5` of font size `1,000,000` to `Paint.getTextBounds()` of font size `5,000,000`.

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

The algorithm is created based on the big font size. Currently, I set font size 1,000,000 pixels.

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
Text is drawn at the baseline which is equal to the `-textBoundsRect.top` (top is always negative). Besides, we should take care of the `textBoundsRect.left` because the width of text is calculated by `right — left`, and with some font and font size, `left` may not be equal to `zero`.

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
When shadow is turned on, we need to change a little bit about the finding font size algorithm and drawing.

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

The above code is not correct in case of `blur > 1`.

