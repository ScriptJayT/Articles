# The Electron Saga 2️⃣: Bridge building

> Hi there; a short message before we jump into the code:
>
> With this being the third article of this journey, I thought: why not open a [Github Repo](https://github.com/ScriptJayT/Checks-out-Electron) for this project? So here you go.
> For those who would like access to the source code. The latest version of this project can be downloaded/pulled from this repo.
> Note that some of the older commit-messages may be a bit wonky. I am terrible at naming things sometimes.
>
> Now let's jump back into the project.

Last time we build the backbone and the start of a body. Now, these two can't communicate with each other yet; and sometimes we sort of do need that.
For now we will transfer a couple of variables.

If you remember, last time we had a variable which could tell us if the app was in production or not:

```js
const is_dev = process.env.NODE !== "production";
```

Well, wouldn't it be usefull to have this constant in our body as well? I would think so, so let's take a look.

## Build the bridge

Remembering back; we told electron to preload a _preload.js_ file.

<small>/main.js</small>

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

<small>So if you didn't create an empty _preload.js_ file in the root folder and ran into some trouble last time... Now you know why. 😉</small>

This _preload.js_ acts like a bridge between our backbone and our body.

Let's build some containers to bridge over.

<small>/preload.js</small>

```js
const { contextBridge } = require("electron");

contextBridge.exposeInMainWorld("E_system", {
	platform: process.platform,
	mode: process.env.NODE,
});
```

The `contextBridge.exposeInMainWorld` function takes in two params, the first on is the object-name you wish to use. The second is the object with key-value pairs of the actual data.

In this case we save the users system (mac, linux, windows) in the `E_system.platform` variable; and the production mode, (live, in production) in the `E_system.mode` variable.

Important to note is that these variables are read-only.

Great, now we have access to backbone-data. Let's see it.

## Cross the bridge

<small>/front/index.html</small>

```html
<html lang="en" spellcheck="false">
	<head>
		<meta charset="UTF-8" />
		<meta http-equiv="Content-Security-Policy" content="script-src 'self' 'unsafe-inline';" />

		<meta name="viewport" content="width=device-width, initial-scale=1.0" />

		<title>My Lab</title>
	</head>

	<body>
		<h1>Hello World</h1>

		<script>
			console.log(E_system.mode);
			console.log(E_system.platform);
		</script>
	</body>
</html>
```

Let's fire up Electron `npm start` and check the console:

```
C:\Program Files\nodejs\node.exe

win32
```

Ha, 🤔
The platform variable logs something we expect, the `win32` is indeed the machine I'm currently working on...
But the mode does not say anything about dev-mode or production.

Well, take a look at the `is_dev` variable declaration

```js
const is_dev = process.env.NODE !== "production";
```

If we check `is_dev` variable against our `E_system.mode`, we would get true. `C:\Program Files\nodejs\node.exe` is not equal to `production`.

So let's change the code a bit so we get a variable we could use without getting a weird string.

<small>/preload.js</small>

```js
contextBridge.exposeInMainWorld("E_system", {
	platform: process.platform,
	is_dev_mode: process.env.NODE !== "production",
});
```

<small>/front/index.html</small>

```html
<body>
	<h1>Hello World</h1>
	<script>
		console.log(E_system.is_dev_mode);
		console.log(E_system.platform);
	</script>
</body>
```

Now let's refresh the Electron app. (The shortcut `ctrl+r` is set up in our app-menu to reload the frontend of our app).

We get the values we expected logged out.

```
true

win32
```

Great! We build a bridge and successfully crossed it.
Let's stop here for now.
Will see you in the next one! 👋
