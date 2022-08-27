---
title: "The Correct Way to Align UILabel on iOS"
date: 2022-08-27T19:52:57+12:00
draft: true
---

> This post focuses on center aligning UILabels by setting their frame and may not be useful for other options such as constraint based layouts

In mobile apps, we commonly implement user interfaces where text needs to be center aligned to another view, such as an image, label or shape. You may have used a naive solution similar to the code below, but something feels off. Let's investigate this problem and see how it can be fixed.

```
// 1. Align the top of the label to the top of the image
// 2. Center align the label to the image using the frames height
labelView.origin.y = imageView.frame.minY + (imageView.frame.height - labelView.frame.height) / 2
```

## The problem
![Pasted image 20220411214501.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1650605357170/rcLHaX-it.png)

If we inspect a UILabel using the View Hierarchy Debugger, we can see that the label's bounding box's height extends well past the height of the text itself. This is because the bounding box's height is determined by the font's characters with the greatest **ascender** and **descender** values, regardless of whether those characters are actually used.

We can demonstrate this by drawing lines whose origin is determined by the font's ascender (red) and descender (green) values.

![Pasted image 20220421215534.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1650605440828/b6ZC4upEh.png)

But, what is an **ascender** and **descender**? Let's take a look at a font's anatomy next.

## Anatomy of a font
![IMG_40E72CCB08D0-1.jpeg](https://cdn.hashnode.com/res/hashnode/image/upload/v1650605498566/2HYbojKIk.jpeg)
*Original image sourced from [here](https://material.io/design/typography/understanding-typography.html#type-properties)*

The image above shows the different components that make up a typical typeface. The components circled in green are ones that we can access as properties on a UIFont. 

> On a UILabel, the font can be accessed with the font property.

Apple's documentation (with some adjustments from myself) define these values as such:
- `ascender` - The top y-coordinate, offset from the baseline, of the font’s longest ascender. Note that on a UIFont, the ascender is actually the distance from the baseline, not the median.
- `descender` - The bottom y-coordinate, offset from the baseline, of the font’s longest descender. Note that descender is a **negative** value since it is offset **below** the baseline.
- `capHeight` - The height of the tallest capital character.
* `xHeight` - The height of the tallest lowercase character.
* `lineHeight` - The distance, in points, between the baseline of two lines of text.

Although it's not accessible as a property, `baseline` is also an important component of a font and can be thought of as the invisible line that characters sit on. It is the reference point for both the ascender and descender.

## The solution
With our newfound knowledge of fonts and UIFont, we can come up with a better way to align text with views.

Instead of centering with the label's default size that we get from `sizeThatFits`, which is determined by characters that may or may not be used, let's go for something more consistent. If you take a look at the majority of text used in user interfaces, you will notice that the height of lowercase characters is relatively uniform. The text that you're reading now being an example of this. Further, lowercase characters typically make up the majority of characters used in text. Therefore, we will use the font's `xHeight`. 

> If your label uses mostly capital characters, you can substitute `font.xHeight` with `font.capHeight`

```
extension CGRect {
	public mutating func centerAlignY(
		with rect: CGRect, 
		left: CGFloat, 
		font: UIFont, 
		precomputedSize: CGSize = .zero) {
	
	    var y: CGFloat = .zero
	
	    // 1. Align the top of the fonts xHeight with the top of the view
	    y -= font.lineHeight + font.descender - font.xHeight
	    // 2. Center align with view using xHeight
	    y += (rect.height - font.xHeight) / 2
	
	    self = CGRect(
			origin: CGPoint(x: left, y: y),
			size: precomputedSize)
	}
}
```

1 - With the naive solution, we began by aligning the top of the label's CGRect to the top of the view. We need to do this with the xHeight as well. The following snippet calculates an offset that we can use to align the top of lowercase letters with the top of the view.

```
font.lineHeight + font.descender - font.xHeight
```

![Pasted image 20220421220017.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1650605540713/PeoQvonIX.png)
2 - Next, we can perform the center align function from the naive implementation. But this time, we will use the font's `xHeight` rather than the label's height

![Pasted image 20220421220120.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1650605572359/gFdM7Ub6s.png)

We can then use this function like so...

```
labelView.frame.centerAlignY(
	with: circleView.frame,
	left: left,
	font: labelView.font,
	precomputedSize: labelSize)
```

And that's it! A simple 2 step function that correctly aligns a label's `xHeight` to another view. It's a subtle difference for sure, but potentially worth the addition of a simple function? 

Finally, a quick comparison between the two solutions. The top is the naive solution and the bottom uses the font's xHeight for alignment.

![Pasted image 20220421231202.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1650605614549/UF0KYq3lD.png)

You can access the repository for the demo project [here](https://github.com/Jessenw/center-align-label)

Thanks for reading my first ever blog post. This whole writing/blogging thing is pretty new to me, so if you have any thoughts or feedback, reach out on [Twitter](https://twitter.com/jessenw3)
