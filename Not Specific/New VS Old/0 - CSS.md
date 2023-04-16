> CSS is more than simply picking a background color

Let's take a look at how things were before and how not to do those things today.

## Table of contents
| |
| --- |
| [Float Layouts](#float-layouts) |
| [Padding Aspect Ratio](#padding-aspect-ratio) |
| [Pixel Values](#pixels-values) |
| [Break Out of the Parent](#break-out-of-the-parent) |

## Float Layouts 

[🔝](#table-of-contents)

You're probably familiar with this:

```css
.el {
	float: left; /*left, right*/
}
.after {
	clear: left; /*left, right, both*/
}
```

In the old days this could be used to align two containers next to each other. It was more a hack than an actual way to do it, due to the effects it had on the parent... Except it was the way to do it.
The element got pulled out of the flow (in a way), which meant that the parent would collapse to 0 if there were no other children to give it a height, or it didn't have a height of its own.
The clear-method was a way to tell the parent not to do that; by giving the element after the floated element the `clear` property.

This way of layout whas rather fragile and with the new layout methods like `flex` and `grid`; `float` quickly became obsolete and outdated.

**☝ Refactor old code to use `flex` or `grid` instead of `float`**

But `float` still has its usecases. When you want text to follow the curve of an image, `float` is the perfect way to do it.

```html
<div class="parent">
	<img src="your_img.png" />
	<p>lorem500</p>
	<p>lorem200</p>
	<p>lorem100</p>
</div>
```

First of all, you would want the image to be the first element. If it's the last one, it won't work. And give it the `float` property.

```css
img {
	float: left;
	/*
	I curve the image by using border radius; 
	using the shape-outside property works as well
	*/
	border-radius: 100%;
	widht: 150px;
	height: 150px;
}
```

But now we still have the collapsing problem, which is not really noticeable as long as the text is long enough. But why take chances?
To fix this, we could use the clear-method again; but I prefer using the parent. It's a cleaner way to do this.

```css
.parent {
	display: flow-root;
}
```

A `display: flow-root` changes the parent's context, in this case, it means that it will grow to fit the content, even if the content is floated.

**☝ Use `float` with `display: flow-root` to curve text around an image**

## Padding Aspect Ratio 

[🔝](#table-of-contents)


A short one; before we had the `aspect-ratio` property, they would use a different method to achieve this effect:

```css
.element {
	/* 1:1 AR */
	width: 50px;
	height: 50px;
}
```

Great! But not responsive, because this would not work:

```css
.element {
	width: 100%;
	height: 100%;
}
```
Even though both values are 100%, they are not the same size. Because percentages for sizes refer to the sizes of the parent element.

So they found this hack:

```css
.element {
	padding-top: 100%; 
	/* 1:1 AR: 100% */
	/* 16:9 AR: 56.25% */
	/* 4: AR: 75% */
	/* 3:2 AR: 66.66% */
}
```

But this did not work if their element had child elements like text. The "aspect ratio" would be added to the height of the elements.
So they did following:

```css
.parent {
	position: relative;
	padding-top: 100%; 
}
.child {
	position: absolute;
	top: 0;
	left: 0;
	right: 0;
	bottom: 0;
}
```

But this still is not responsive to the content. The content of the child could still overflow.

Luckily those days are over.

**☝ Use the `aspect-ratio` property to keep a constant ratio between with and height that is responsive**


## Pixels Values 

[🔝](#table-of-contents)

A pixel. A small, square speck of which your screen is build out of. But in CSS-land, a pixel is a unit measurement, one of the first, if not the first.

If you want a square of 50 by 50 pixels, you would use the following:
```css
.square {
	width: 50px;
	height: 50px;
}
```
Nothing ground-breaking. But nowadays we have a lot more units that we can use like `rem`, `em`, `ch`, `vh`, `vw`, `dvh`, `dvw`...
Each unit has their own use case, their own reference point; and their own perks that should be taken account for.
For this section, I will only take al look at the first three of that list.

`Rem`
: A unit that has the roots font-size as reference point; by default 16px. 
<small>(You can change this value by changing the roots font-size; but you should NOT do this.)</small>

`Em`
: A unit that has the elements font-size as reference point. So if you change the font-size, the em-value changes as well.

`Ch`
: A unit that has the elements 0 character's width of the current font and font-size. So if you change the font-size, the ch-value changes as well.

When should you use each? And when not?

### Rem

If you would use pixels, use rem instead. That is a good start in my opinion. So, font-size, width, height...
The `rem` unit is more accessible than the pixel unit. Because a browser can easily increase the rem default value when a user wishes to use larger fonts for example. If you use pixels instead, the fonts on site might not be enlarged, which is bad UX.
Yes, it might create some weird values for some regular used pixel-values due to the base 16 our rem unit uses, but I find myself using CSS custom properties in most cases anyway, so it doesn't matter much.

### Em

If you want a unit that sizes with your font-size for a certain component, use `em`. Let's say we have a button with `font-size: 0.875rem` and we want a padding on all sizes of 0.4375rem. 
Now later you might want the same button but larger, let's say `font-size: 1.25rem`; but you want to keep the same ratios, you want to "scale" the button up. In that case, you might want to change the padding value to 0.5em instead. So now the only value you need to update is the font-size.

```css
/* old */
.btn {
	font-size: 0.875rem; /*14px*/
	padding: 0.4375rem; /*7px*/
}
.btn[large] {
	font-size: 1.25rem; /*20px*/
	padding: 0.6125rem; /*10px*/
}
/* new */
.btn {
	font-size: 0.875rem;
	padding: 0.5em;
}
.btn[large] {
	font-size: 1.25rem;
}
```

It makes the code a lot cleaner; as you can see, using a value like 0.4375rem is a bit cluttering, for sure if you don't know what that value translates to in pixels, it might seem a random value.
But by using the 0.5em, now we simply know: half of the font-size.
And it beautifully scales the button up.
Many use this trick to scale box-shadows as well.

### Ch

The `ch` unit makes it very easy to constrict a paragraph to a certain number of characters, heck, I would not be surprised if that was the exact reason it was introduced.
It works best with monospace fonts in that way, due to all characters being the same width. It does work for other fonts too, but the result is less satisfying in my opinion.

Similar to the `em` unit, `ch` is linked to the font-size. They are not the same though, I have noticed that for some fonts 1em and 2ch are similar in size, but not exactly so. This is due to the `em` unit being linked to the fonts height, where `ch` is character width, I believe. The wording in their descriptions are a bit fuzzy to say for sure.

According to some UX articles, a length of 45 to 90 characters is the best length for paragraphs to ensure legibility, and yeah, that seems to work. 
Personally I shoot for max-widths of 70 to 75ch, which fits neatly in that range. 
This unit makes it really easy to implement that theory in our web designs.

<small>Articles I quickly scanned through for that character-range:</small>
[<small>Practical Typography: Line Length</small>](https://practicaltypography.com/line-length.html) 
[<small>Webflow: Ch unit</small>](https://webflow.com/feature/control-width-of-text-elements-by-character-count-using-ch-unit)

## Break out of the Parent 

[🔝](#table-of-contents)


You might find yourself in this situation:

```html
<main class="max-width">
	<article>
		<p>lorem500</p>
		<p>lorem200</p>
   	</article>
	<div class="full-width">
   		<article>
			<p>lorem500</p>
     	 		<p>lorem200</p>
   		</article>
	</div>
	<article>
		<p>lorem500</p>
		<p>lorem200</p>
	</article>
</main>
```

Where the `max-width` constrains the width of its children to, let's pick, 75rem; and the `full-width` wants to span the whole site width of, let's say, 100vw.

```css
.max-width {
   max-width: 75rem;
   /*to center the container horizontally*/
   margin-inline: auto; 
}
.full-width {
   width: 100vw;
}
```

Well, this is not going to work; unless a messy overflow was your goal. So I often see following to fix this:

<small>(for those thinking: `width: 100%`; that would mean the same width as the parent, which is not the desired effect as it would not break out)</small>

```css
body {
   overflow: hidden;
}
.max-width {
   max-width: 75rem;
   margin-inline: auto;
   overflow: visible;
}
.full-width {
   position: relative;
   left: -50vw; /*pull out*/
   transform: translateX(-50%); /* push back in*/
   width: 100vw;
}
```

You would fix the overflow issue by removing scrollbars on the `main` width `overflow: visible` and counteract the overflow-issues on the `body` with `overflow: hidden`. 
<small>(Yes, you're better of specifying `overflow-x: hidden` in production)</small>
Then you would position the article in the center by pulling it back by half the viewport width and pushing it back by half its own length. 

And this would work, were it not for scrollbars.
The scrollbar would push the content further back by, idk, 20 pixels on Chrome, 30px on Safari and 0px on browsers with floating scrollbars. 
<small>(No, I did not measure the scrollbar thickness of those browsers, fyi, these values are random)</small>

There is no way of counteracting these offsets, due to there being no standards about scrollbars and their width. Not even with JavaScript, or at least not reliably.
This means that for a best case: the content does not line up with the rest; or a worst case: overflow issues that completely muck up your design.

So, how do you break out? 
In my opinion, you don't. I would put the max-width class only on the blocks that actually need it. 
But in the case you can't add classes to those elements except the full-width class, you could do something like this:

```css
.max-width > :not(.full-width) {
   max-width: 75rem;
   margin-inline: auto;
}
```

There is a catch though, and that is when the children of `.max-width` are not containers themselves.

```html
<main class="max-width">
	<!-- these children -->
	<h1>lorem5</h1>
	<p>lorem500</p>
	<!--  -->
	<article>
		<p>lorem500</p>
		<p>lorem200</p>
   	</article>
	<div class="full-width">
   		<article>
			<p>lorem500</p>
     	 		<p>lorem200</p>
   		</article>
	</div>
	<article>
		<p>lorem500</p>
		<p>lorem200</p>
	</article>
</main>
```

Let's say that I want all `p` tags to be a maximum of 60 characters long; the `max-width` can be overwritten to be `60ch`, that is not the issue; but the text will not be able to align left with the rest of the content, due to the `margin-inline: auto`. And removing the `margin-inline` of those elements would mean that the `p` would stick to the edge of the screen instead. You could try to hard-code the `margin-left` so it lines up again, but I don't recommend doing that.

You will have to ensure the HTML is properly wrapped; by a div, or a proper semantically correct element.

```html
<main class="max-width">
	<div>
		<h1>lorem5</h1>
		<p>lorem500</p>
	</div>
	<article>
		<p>lorem500</p>
		<p>lorem200</p>
   	</article>
	<div class="full-width">
   		<article>
			<p>lorem500</p>
     	 		<p>lorem200</p>
   		</article>
	</div>
	<article>
		<p>lorem500</p>
		<p>lorem200</p>
	</article>
</main>
```

Something to think about which method you prefer. Use verbose classes on the elements that need it, or only add a class to divergent items.

**☝ Don't use `width:100vw` to break out of a container parent; rethink how to structure your HTML / how to use your CSS-classes** 
if you can, and curse if you can't.

In case you only need a background that spans the whole site-width, I recommend this short by Kevin Powell: [Bleeding Background](https://www.youtube.com/shorts/81pnuZFarRw) as they do a way better job explaining how to than I ever could. 😉

See you in the next one! 👋