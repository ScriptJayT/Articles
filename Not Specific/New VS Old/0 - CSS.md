# CSS

> CSS is more than picking a background color.

An overview for new features and old css-tricks not to be used anymore.

- Old
	- Float Layouts
	- Padding Aspect Ratio
	- Pixel Values
	- Break Out of the Parent
- New
	- :is(), :where()
	- :has()
	- Container Queries
	- Layers
	- Nesting
	- Use of Pre-processors
- Tips & Tricks
	- Min, Max, Clamp
	- Where to put Media Queries
	- Performant Transitions & Animations
	- Declaritive / Semantic CSS
- Better UX
	- Media Prefers Reduced Motion
	- Media Color Contrast
	- Readability

## Float Layouts
You're probably familiar with this:

```css
.el {
	float: left; /*left, right*/
}
.after {
	clear: left; /*left, right, both*/
}
```

In the old days this could be used to align two containers next to each other. It was more a hack than an actual way to do it, due to the effects it had on the parent.
The element got pulled out of the flow (in a way), which meant that the parent would collapse to 0 if there were no other children to give it a height, or it wasn't given a fixed height.
The clear-method was a way to tell the parent not to do that; by giving the element after the floated element the `clear` property.

This way of layout whas rather fragile and with the new layout methods like `flex` and `grid`; `float` quickly became obsolete and outdated.

**Refactor your old code to use `flex` or `grid` instead of `float`**

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

But now we still have the collapsing problem, which is not really noticable as long as the text is long enough. But why take chances?
To fix this, we could use the clear-method again; but I prefer using the parent. It's a cleaner way to do this.

```css
.parent {
	display: flow-root;
}
```

A `display: flow-root` changes the parent's context, in this case, it means that it will grow to fit the content, even if the content is floated.

**Use `float` with `display: flow-root` to curve text around an image**

## Padding Aspect Ratio

A short one; before we had the `aspect-ratio` property, they would use a different method to achief this effect:

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
Even though both values are 100%, they are not the same size. Because percentages refer to the size of the parent element.

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

But this did not work if their element had children elements like text.
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

Lukily those days are over.

**Use the `aspect-ratio` property to keep a constant ratio between with and height that is responsive**


## Pixels values

## Break-Out of the Parent

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

Where the `max-width` constrains the width of its children to let's say 75rem; and the `full-width` wants to span the whole site width of let's say 100vw.

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
   overflow: vissible;
}
.full-width {
   position: relative;
   left: -50vw; /*pull out*/
   transform: translateX(-50%); /* push back in*/
   width: 100vw;
}
```

You would fix the overflow issue by removing scrollbars on the `main` width `overflow: visible` and counteract the overflow-issues on the `body` with `overflow: hidden`. 
<small>(Which most do by default anyway, so no biggy there. 
And yes, you're better of specifying `overflow-x: hidden` in production)</small>
Then you would position the article in the center by pulling it back by half the viewport width and pushing it back by half its own length. 

And this would work, were it not for scrollbars.
The scrollbar would push the content further back by, idk, 20 pixels on Chrome, 30px on Safari and 0px on browsers with floating scrollbars. 
<small>(No, I did not measure the scrollbar thickness of those browsers, fyi, these values are random)</small>

There is no way of counteracting these offsets, due to there being no standards about scrollbars and their width. Not even with JavaScript, or at least not reliably.
This means that in best case: the content does not line up with the rest; or in worst case: overflow issues.

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

You will have to ensure the HTML is properly wrapped; by a div, or a proper semanticaly correct element.

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

**Don't use `width:100vw` to break out of a container parent; rethink how to structure your HTML / how to use your CSS-classes if you can, and curse if you can't**