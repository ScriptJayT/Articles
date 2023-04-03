# JS Errors in the Console, with a twist

> Oh no! My site, it's broken!
> What ever shall I do? Better open the console to see what's wrong...

## It happens

Errors happen, so we have to handdle them. And since javascript has the console-object, most if not all js-developers use it.
Yet having console errors littered around on your site is not great, (I could be wrong but) it might even influence your SEO score in search engines, so some delegating is in order.

## Better not flood the user's console

Flooding the console of a regular user with errors and exceptions is rather inefficient and unnecessary. They don't know how to open the developer tools, never mind navigate to the console.
The only thing website-users need to know is whether or not all interactive functions on the site work, that's it. And that can be done by showing just a _"Something has gone wrong"_ message in a nice and well designed pop-up.

## But a flood for them is a sip for me.

Yes, developers do use and need the console for first line debugging, users do not.
So let's hide all messages from the console unless a developer visits the site. A dumbed-down developer-mode if you wish.

First, convert all loud errors into silent ones. Use the try-catch block to exit the function early when something goes wrong and show a simple error message to the user. That's better than crashing the whole thread, wouldn't you say?

Great! But now the developer can't assess any errors though. To remedy, let's create our own console function:

```js
const dev_log = () => {};
```

This function is going to pass a message to the console if and only if a condition is met. Otherwise it will exit the function early with `return`.

```js
const dev_log = (_error) => {
	if (!condition) return;
	console.log(_error);
};
```

But what condition? You could use any: login, ip-address, local host... Or whatever floats your boat.
I like checking the hash in the url, you know: `yoursite.com#hash`.
It's the easiest to set up and has no real implications for the website, since the hash is mostly used to highlight articles, or navigate one-pagers.
So let's do that. You can access that hash from the location-object.

```js
const dev_log = (_error) => {
	const is_debugging = location.hash == "dev";
	if (!is_debugging) return;
	console.log(_error);
};
```

And that's it. Now the only thing left is to call this function every time we try and catch errors;

```js
try {
	foo();
} catch (error) {
	show_error_to_user();
	dev_log(error);
}
```

Ofcourse you could expand the function to support multiple types of console functions like `console.error`, `console.group` or `console.time` even, make it fit your needs.

So now only visitors using `yoursite.com#dev` will be able to inspect any errors you logged to the console.
What do you think?
