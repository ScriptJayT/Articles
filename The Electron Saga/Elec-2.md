# The Electron Saga 2: Bridge

Last time we build the backbone and the start of a body. Now, these two can't communicate with each other yet; the backbone can't send variables to the body. And sometimes we don't need that, but often we do.

If you remember, last time we had a variable which could tell us if the app was in production or not:

```js
const is_dev = process.env.NODE !== "production";
```

Well, wouldn't it be usefull to have this constant in our body as well?
So let's set this up.

## preload.js

Remembering back; we told electron to preload a _preload.js_ file.

```js
function create_window(_name) {
	const my_window = new BrowserWindow({
		title: "My Lab",
		width: 1000,
		height: 750,
		webPreferences: {
			contextIsolation: true,
			nodeIntegration: true,
			// We are telling that here!
			preload: path.join(__dirname, "preload.js"),
		},
	});

	my_window.loadFile(path.join(__dirname, `./front/${_name}.html`));

	if (!is_dev) return;
	const dev = new BrowserWindow();
	my_window.webContents.setDevToolsWebContents(dev.webContents);
	my_window.webContents.openDevTools({ mode: "detach" });
}
```

<small>So if you didn't create an empty _preload.js_ file in the root folder and run into some trouble last time... Now you know why. 😉</small>

This \_preload.js acts like a bridge between our backbone and our body.
