# The Electron Saga 3️⃣: All beginings have an end.

And we are back for our latest part of the Electron Sage. As this series is more about the setup I will only go so far, as building the app itself is not really linked to the workings of, and working with Electron.

## Frameworks?

When I first started with Electron, I did not think about using a JavaScript framework; most of the time I use PHP for my web development, so it did not come to mind that such a framework could be useful.

The app I wanted to build would be able to do a bit more than just add items to a list I could edit. It needed to be expandable; so I build my first page with a nice sidebar and a main... and came to the realisation that I would need to copy the sidebar each time I would add a new page.
When using PHP, you would build a page *index.php*, which would contain the sidebar and an empty main that would be populated with a `require`- or an `include` function. What file you would include would depend on the current location (the URI).

```php
<html lang="en">

<head>
   <meta charset="UTF-8">
   <meta name="viewport" content="width=device-width, initial-scale=1.0">

   <title>My Site</title>
</head>

<body>
	<header>
   		<h1>Hello World</h1>
	</header>

	<aside>
		<nav>
			<ul>
				<li>navitem</li>
				<li>navitem</li>
				<li>navitem</li>
				<li>navitem</li>
			</ul>
		</nav>
	</aside>

	<main>
		<?include 'path_of_file'?>
	</main>
</body>

</html>
```

But no PHP, so no HTML-include.
I shrugged and instead of looking for a NPM package that could help me, or to look into a framework like Svelte; I decided to try importing with vanilla JavaScript. It worked, kind of.

## Layout

First of; creating a new folder *classes* in */front/logic* and a new file *app.js*.
Then linking the *app.js* file as a module.

<small>/front/index.html</small>

```html
<head>
	<meta charset="UTF-8">
	<meta http-equiv="Content-Security-Policy" content="script-src 'self' 'unsafe-inline';" />

	<meta name="viewport" content="width=device-width, initial-scale=1.0">

	<title>My App</title>
	<link rel="stylesheet" href="styles/reset.css">
	<link rel="stylesheet" href="styles/main.css">

	<!-- right here  -->
	<script type="module" src="logic/app.js"></script>
</head>
```

Great, check if the connection is working by logging the values from last time: 

<small>/front/logic/app.js</small>

```js
console.log(E_system.mode);
console.log(E_system.platform);
```

No breakage? Great, let's see, how do we want it to work?
Well, when I click a button in the nav; the content should be switched out.
I would like to be able to include content in a header, main and footer separately as to not break my main layout; so let's build up a nav and include some slots in our body

<small>/front/index.html</small>

```html
<body>
	<nav id="feature-nav" class="c-wrapper">
		<ul>
			<li>
				<button sheet="home">
					<span>Home</span>
				</button>
			</li>
			<li>
				<button sheet="todo">
					<span>Todo</span>
				</button>
			</li>
			<li>
				<button sheet="tracker">
					<span>Schema</span>
				</button>
			</li>
		</ul>
	</nav>
	<div id="app-interface">
		<header>
			<content-slot></content-slot>
		</header>
		<main>
			<content-slot></content-slot>
		</main>
		<footer>
			<content-slot></content-slot>
		</footer>
	</div>
</body>
```

You can see I added an attribute I called `sheet` to all buttons. I also created a folder */front/sheets* and added some HTML-files with names that correspond with those attribute values.

It is certainly possible to add aria-labels like `aria-current` and `aria-controls` to the buttons; or add a skip-to-content button above the `ul`, but those features are a bit out of scope for this checking-out series.

I want the sheet-files to be plain old HTML-file; I want three containers that I can target with the JS dom-api and just write content as I would normally.

<small>/front/sheets/home.html</small>
```html
<header-content>
	<h1>Home</h1>
</header-content>

<main-content>
	my content
</main-content>

<footer-content>
	my footer
</footer-content>
```

## JS

The class is called `page_loader`, and it just gets imported and called. The `is_ready` method checks to see if our index.html has all the required elements to work; and the `activate()` method enables the button click.

```js
import { D$ } from "./fn/dev.js";
import { page_loader } from "./classes/loader.js";

console.log(E_system.mode);
console.log(E_system.platform);

//? Load Pager
{
	const page = new page_loader();
	if (page.is_ready()) {
		D$(console.log, "Page Loader Ready");
		page.activate();
	} else D$(console.warn, "Page Loader not Ready");
}
```

It works by using the fetch-api and a DOMParser object. 
When a button is clicked, the class checks which sheet it links to and fetches the file with, well, `Fetch`.
A plain text-string is returned; which is passed on to a DOMParser. This JS-object is able to transform our plain string back to HTML.
Then we use the good old DOM-api to query our content-blocks 
(`<header-content>` `<main-content>` `<footer-content>`)
At last it's just a matter of removing the old content from our index.html and pasting in the new content to have our final result.

### Helper functions;

I had made some helper functions and saved them in the files */front/logic/fn/dev.js* and */front/logic/fn/dom.js*.

`$D()`
: Is a function that fires the passed through function only when the app is in development. It uses the `is_dev` variable we bridged over before.

`$S()`
: An abbreviation for querySelector

`$SA()`
: An abbreviation for querySelectorAll

### page_loader class

First; some more helper functions; these do most of the work; they fetch and convert data.
And we import the previously mentioned helpers as well.

<small>/front/logic/classes/loader.js</small>
```js
import { D$ } from "../fn/dev.js";
import { S$, SA$ } from "../fn/dom.js";

/**
 * @description get the sheet-attribute off of buttons
 * @param {HTMLButtonElement} _btn
 * @returns {string} - Sheet
 */
const get_sheet = (_btn) => {
	return _btn.getAttribute("sheet") || "";
};

/**
 * @description fetch sheet and return its data
 * @param {string} _sheet
 * @returns {response} - string
 */
async function find_sheet(_sheet) {
	const path = `./sheets/${_sheet}.html`;
	const F = await fetch(path, {
		method: "GET",
	})
		.then((response) => {
			// D$(console.log, response);
			return response.text();
		})
		.then((data) => {
			return data;
		})
		.catch((error) => {
			D$(console.log, error);
			return "";
		});
	return F;
}

/**
 * @description convert string to html of header-main-footer
 * @param {*} _html
 * @returns {array} - object of header_data, main_data, footer_data
 */
const data_to_html = (_html) => {
	var parser = new DOMParser();
	var doc = parser.parseFromString(_html, "text/html");
	const header_data = S$("header-content", doc)?.innerHTML.trim() || "";
	const main_data = S$("main-content", doc)?.innerHTML.trim() || "";
	const footer_data = S$("footer-content", doc)?.innerHTML.trim() || "";
	return [header_data, main_data, footer_data];
};
```

And the main class; it does little more than querying all nav-buttons with the `sheets` attribute and the `<content-slot>` tags and attaches a click-event to those nav-buttons.
There are more functions you could add to the class; for example, automatically load the home.html sheet on startup. Or the sheet last used.
And some navigation functionality like showing what the current sheet is.

```js
export class page_loader {
	#header_slot;
	#main_slot;
	#footer_slot;

	#all_nav_btns;

	constructor() {
		this.#get_targets();
		this.#get_nav();
	}

	// query native elements
	#get_targets() {
		this.#header_slot = S$("#app-header content-slot");
		this.#main_slot = S$("#app-content content-slot");
		this.#footer_slot = S$("#app-footer content-slot");
	}
	#get_nav() {
		this.#all_nav_btns = SA$("#feature-nav [sheet]");
	}

	/**
	 * Fetch given sheet and render the data in the app
	 * @param {string} _sheet - sheet of data to fetch
	 * @param {HTMLButtonElement} _btn - button to turn on in the nav
	 * @returns {void}
	 */
	async #render(_sheet, _btn) {
		const data = await find_sheet(_sheet);
		if (!data) return;
		const [header_data, main_data, footer_data] = data_to_html(data);

		this.#header_slot.innerHTML = header_data;
		this.#main_slot.innerHTML = main_data;
		this.#footer_slot.innerHTML = footer_data;
	}

	/**
	 * @description checks to see if the loader has succesfully retrieved all nav btns and the content slots
	 * @returns {boolean}
	 */
	is_ready() {
		if (
			this.#all_nav_btns &&
			this.#header_slot &&
			this.#main_slot &&
			this.#footer_slot
		)
			return true;
		return false;
	}

	/**
	 * @description function to activate click events on nav-btns
	 */
	activate() {
		this.#all_nav_btns.forEach((btn) => {
			btn.addEventListener("click", (e) => {
				const sheet = get_sheet(btn);
				this.#render(sheet, btn);
			});
		});
	}
}
```

## It works but...

What I noticed when using this class is, that when I write JavaScript in a `<script>` tags in a sheet.html; it does not get triggered in the index.html; which does not mean it is safe to allow any scripts to infiltrate those files though. My guess is that the script gets run when the fetch-request reads through the file and not when the content gets 'pasted' into our index.html.

But what if you want to import some JS anyway? Well; I noticed that custom-elements get rendered properly, (if they are defined in our app.js file); so you could use a custom-element to autoload JS code.

Notice: I can't really speak about the security of this method; I would say not too bad, but if those HTML files got corrupted anyhow, the class would have no way of knowing, never mind handeling.

Conclusion: use a framework 😉

## Series Conclusion

In these 4 articles, I set to lay out my first steps into starting with Electron and desktop-app making. I must say I was surprised how little extra setup it is compared to web development, for this example at least. I assume that if I where to dive deeper into this world, it would require some more setup. 
I found that it is certainly possible to work with Electron without a framework (I fully build my app before realizing I could have used one); but it would have improved my dev-experience a bit.
I like how close it is to web-development I'm used to, and working with it feels quite nice, but I can't really compare how it holds up against other app-dev frameworks.
However, the unsolvable errors at the start of my journey should be mentioned [(The Electron Saga 0)](https://dev.to/scriptjayt/the-electron-saga-0-my-first-app-45jg). They did cause a bit of confusion and worry.

Overall, it was fun mocking around this new environment, away from the projects lurking over my shoulder.
Speaking of which, I should return to those. I hope you got something out of this series.
See ya! 👋