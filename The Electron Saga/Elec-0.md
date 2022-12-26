# The Electron Saga 0: My First App

I've always wanted to make a desktop app. ✨
Ever since I started my webdev journey.
But I currently have other projects to finish, and a responsible dev would not start one project before all others are done, right?
Right.

But I am not a responsible dev, so I am building a desktop app now.

## ~Let's Start: What to make?

Something small, (my other projects are giving me the stink eye already 👀). But not too small, it still has to be usefull.
An app I could use every day, for work; I thought.
But what functions does it need to have? Let's start simple, A todo-list. I have written a webcomponent for todo-lists in Typescript before, should make things a bit easier.
So that is our goal, an app with a todo-function.

### What to use?

My knowledge on app-developing is limited to non-existent. I have only worked with web-technologies before, like HTML, CSS, Javascript... So learning a complete new language is not ideal. At least not for a quick sideproject like this, I should say. So I was looking for frameworks that use web-technologies for app-developing; and I found two: **_Electron_** and **_Tauri_**.

**_Electron_** is bigger, slower, but more consistent on different devices, since it carries its own machine to run on (chromium, I believe).
**_Tauri_** on the other hand is said to be smaller and faster, because it doesn't carry its own machine and uses one present on the OS. This however has the drawback of it not beeing as consistent.

In the end I did choose Electron, since it has more documentation to work off. Given this is my first project, it's the safer choice. But after this I'll most certainly give Tauri a chance.

So I went in search of a tutorial to start off of and familiarize myself with the framework. I found this one, it's pretty great.
{% embed https://www.youtube.com/watch?v=ML743nrkMHw %}
<small>Video by Traversy Media</small>

## ~Setup

> A dev is set up to encounter errors.

I cloned the quick-start directory from Electron's [github](https://github.com/electron/electron-quick-start) and spun up my node-js.
No problems there. 🦾

I followed the tutorial to set up the Main frame of the app and everything worked as expected; but after a while this error message appeared in my terminal:

```
[9532:1010/142156.863:ERROR:gpu_init.cc(521)] Passthrough is not supported, GL is disabled, ANGLE is
```

Odd, never seen this one. 🧐
When the interwebs didn't return more than a void function would, even after an hour of searching, I gave up. It didn't seem to impact my workflow, so... ¯\_(ツ)\_/¯.
I did find some other people who got the same error message, all using a different setup and even a different OS; so no fix there.
If anyone knows what might cause this error and how to mend it, by all means, leave a comment.

### The Main Frame

I got back to assembling the main frame and got it running without any issue. I could open the app, close the app... But I could not yet inspect the innards of the app.

Since Electron is web-based it is possible to use devtools like you would on a website, which is great for debugging. So I implemented the function to automaticly open the devtools when launching the app:

```js
my_window.webContents.openDevTools();
```

And it worked... until it didn't.🙀 Once I closed the app and reopened it, the devtools would not show again.

I got stuck. I could not figure out what was going on. From one moment to the next the tools stopped working.
Traversy Media did not seem to get the same issue. So, just me?
"Perhaps Electronmon is acting up, so try disabling that."
"No that isn't it. It doesn't even work just running Electron."
I deleted and restarted the whole project three times, only to get the same issue again and again. The first devtools launch worked, every following failed.

## ~Got there in the end. 😸

So, sleep helps. After taking a break for the weekend I came back to this function and went straight to the github issues of Electron and found that it was a recurring problem.
This time the interwebs did return something! A workaround:

```js
const dev = new BrowserWindow();
my_window.webContents.setDevToolsWebContents(dev.webContents);
my_window.webContents.openDevTools({ mode: "detach" });
```

I can't really explain why this works while the other one didn't but it is working now, so let's not touch it.
It's time to set up the front-end of our app...
