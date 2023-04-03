# The Electron Saga 1️⃣: Setup

It's been a while, hasn't it? But let's not dwell and dive back into the project.

The goal for this chapter is to try give a bit of a clearer image of what is going on. Rather than tell you about mishaps I encountered, I'll try and show you how it works.

## Folder Structure

```
 - root
	- front
	- node_modules
	- .gitignore
	- main.js
	- package.json
	- package-lock.json
	- preload.js
```

### First: background noice \(mostly\)

.gitignore
: if you are familiar with Git you know what this one does.
It allows us to exclude files and folders from the git repo. In our case it removes the _node_modules_ folder and the _package-lock.json_. You don't need it if you don't use Git.

package.json
: this file is generated when we setup everything we need, including Electron; and contains some functions to symplify workflow. We will take a look at this one a bit down below.

package-lock.json
: magic happens here. It is a file that is generated after a Node project is initialized.

### Next up: Electron

main.js
: this is the core of our project. It contains the code which allows us to 'communicate' with Electron.
You could say it is the backbone of our project and it also acts like a key to the user system.

preload.js
: this file is the bridge between our backbone and our future front-end. Any user-system data we want access to in our front-end will be copied here and passed on.

### Front-end

The folder _front_ contains any code, styles, markup the actual project consists out of.
I devided this folder further like this:

```
- front
	- logic
	- media
	- sheets
	- styles
	- index.html
```

Note that you could rename any folder in here to your liking.
The _logic_ folder contains any javascript; _media_ contains any images and svg-icons; _styles_ contains css-files; and _sheets_... html-files. What for is not for now.

## Setup Node

To setup Node you need it installed on your system. Then open a terminal in the root folder of the project.
First of, initialize Node with following command: `npm init`
Second, open the generated _package.json_ and add/edit the following:

```js
	"main": "main.js",
	"dependencies": {
		"electron": "^21.1.0"
	}
```

If you want to install some more packages here like Sass or Typescript, that is possible ofcourse.
With this set up, let's install Electron: `npm install`.

If that succeeded we could start with the next setup, but first I like to add a script to the package to simplify the workflow a bit.

```js
	"scripts": {
		"start": "electron ."
	},
```

Now, the command `npm start` would launch Electron. But before this works, we need a body, meaning a backbone and a front.

## Setup main.js

As said before, the main.js is like the core of the project. Without, no Electron.
Let's get it set up.

### Variables

Firstly the _app_, _BrowserWindow_ and _Menu_ from _electron_. These give us the objects necessary to create the app itself.
The _path_ gives us access to the user's directory folders.
Lastly is the _process_. This object gives us access to the user's system info. And the _env_ object can be used to check if we are in develop-mode.

```js
const { app, BrowserWindow, Menu } = require("electron");
const path = require("path");

const is_mac = process.platform == "darwin";
const is_windows = process.platform == "win32";
const is_linux = process.platform == "linux";

const is_dev = process.env.NODE !== "production";
```

Now, let's create an object containing the menu of our app. There are some default functions we can use, to use those, use the _role_ key.
The _accelerator_ key indicates a keyboard shortcut.
Here we install a `reload` functionality, to quickly reload to see new changes in the front-end; and a `quit-app` functionality, well to fully close the app.

```js
const my_main_menu = [
	{
		label: "App",
		submenu: [
			{
				label: "Quit",
				role: "quit",
				accelerator: is_mac ? "Cmd+W" : "Ctrl+W",
			},
			{
				label: "Reload",
				role: "forceReload",
				accelerator: is_mac ? "Cmd+R" : "Ctrl+R",
			},
		],
	},
];
```

### Functions:

The `create_window` function sets up an app window. It takes the name of the file to render there.
As you can see it creates a window with given width and height and title. It is also preloading the _preload.js_ here.
It also checks whether or not we are in develop-mode and opens a chrome devtools if so. The way we create the devtools here is a fix, as we saw in [Electron Saga 0](https://dev.to/scriptjayt/the-electron-saga-0-my-first-app-45jg)".

```js
function create_window(_name) {
	const my_window = new BrowserWindow({
		title: "My Lab",
		width: 1000,
		height: 750,
		webPreferences: {
			contextIsolation: true,
			nodeIntegration: true,
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

The `set_menu` function takes an object and creates a menu.

```js
function set_menu(_menu) {
	const M = Menu.buildFromTemplate(_menu);
	Menu.setApplicationMenu(M);
}
```

### Events:

These events trigger when opening the app and closing the app.
For a deepdive, I suggest looking at the documentation.

```js
app.on("ready", () => {
	create_window("index");

	set_menu(my_main_menu);

	app.on("activate", () => {
		if (BrowserWindow.getAllWindows().length === 0) create_window("index");
	});
});

app.on("window-all-closed", () => {
	if (!is_mac) {
		app.quit();
	}
});
```

## First Load

Lets edit the _index.html_ in our _front_ folder to see something; some may notice a new meta-tag; the "http-equiv" is needed for applications.

And to prevent red squibly lines when the chrome-engine doesn't recognize a word, the `spellcheck="false"` on the `<html>` tag does wonders.

```html
<!DOCTYPE html>
<html lang="en" spellcheck="false">
	<head>
		<meta charset="UTF-8" />
		<meta http-equiv="Content-Security-Policy" content="script-src 'self' 'unsafe-inline';" />

		<meta name="viewport" content="width=device-width, initial-scale=1.0" />

		<title>My Lab</title>
	</head>

	<body>
		<h1>Hello World</h1>
	</body>
</html>
```

Now we run the command `npm start` and a new window should open and it should greet us with the Hello World message we set.

Finaly, caught up.
There still is some stuff like the preload.js we need to take a look at, and even further down the line what the _sheets_ folder is for.
