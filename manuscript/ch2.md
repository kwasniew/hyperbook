# Chapter 2: View as a function of state

## Getting started

When learning a new framework, we like to understand every single step we take and every single line of code we write.
Therefore, instead of generating boilerplate, we'll be writing everything form scratch.

With more Hyperapp experience, you may formalize the setup into your own starter kit.
However, you may also realize the starter kit is no longer necessary with certain sources of complexity eliminated.


In your project root directory create **index.html** and **src/App.js**. This book convention is to 
uppercase the first letter of the JS file names.

**index.html**
```html
<!DOCTYPE html>
<html lang="en">
<head>
   <meta charset="UTF-8">
   <meta name="viewport" content="width=device-width, initial-scale=1" />
   <title>HyperPosts</title>
   <link rel="stylesheet" href="https://unpkg.com/sakura.css/css/sakura.css" type="text/css">
   <script type="module" src="src/App.js"></script>
</head>
<body>
   <main>
       <div id="app"></div>
   </main>
</body>
</html>
```
You'll be using a prototype friendly [sakura](https://oxal.org/projects/sakura/) classless CSS framework to get
out-of-the-box styling for semantic HTML.
Our HTML uses **src/App.js** as ES6 module (`type="module"`). Therefore you can use ES6 imports without a build tool.
Finally, Hyperapp will render its contents into the  `<div id="app"></main>`.


**src/App.js**
```js
import {h, text, app} from "https://unpkg.com/hyperapp?module";

const state = {text: "Welcome to Hyperapp!"};

app({
    init: state,
    view: state => h("h1", {id: "my-header"}, text(state.text)),
    node: document.getElementById("app")
});
```
To start experimenting, import Hyperapp as ES6 module directly from CDN (e.g. unpkg.com).
The exported module provides three functions: `h` , `text` and `app`.

The **app** function is your main integration point with the framework.
We’re passing an object with three attributes:
* **init** - initial state of your application
* **view** - view function rendering current state
* **node** - DOM node to mount the application to

Serve your **src** directory with any static HTTP server. We're using https://www.npmjs.com/package/http-server.
If you already have Node.js and npm on your machine then you can install it with:
`
npm i http-server -G
`
After that, run in the root of the project:
```
http-server .
```

By default, the `http-server` starts on http://localhost:8080.

Check if your browser renders the following HTML:

![Figure: Getting started HTML](images/getting-started.png)

## Understanding view function

![View as a function of state](images/view.png)

In the functional approach to UI development, the view is a pure function of the state.
Hyperapp ```view``` function takes ```state``` object as an input and returns a data structure describing future DOM tree to build.
The returned data structure is known as the **Virtual DOM**. The framework can translate it into very efficient low-level DOM updates.
With Hyperapp, you never work directly with the DOM API in your application code.
Instead of making imperative calls such as ```document.createElement```, ```element.insertBefore``` or ```element.removeChild``` you declare
what the view should look like and let the framework figure out the details. 

## Analyzing view rendering options

View function builds a Virtual DOM data structure. You have at least 3 options to choose from:
* `h` - low level Virtual DOM builder
* `@hyperapp/html` - syntactic sugar on top of the `h` function
* `hyperlit` - JSX like library translating to `h` at runtime or build time


### h

Currently, your application uses a built-in **h** function to create Virtual DOM nodes.
Change your ```view``` function to wrap the text in a ```span``` element:
```js
state => h("h1", {id: "my-header"}, [h("span", {}, text(state.text))])
```
Check the generated HTML:
```html
<h1 id="my-header"><span>Welcome to Hyperapp!</span></h1>
```

What about creating something more complicated?
How much effort would it take to translate the following snippet into the `h` function calls?
```html
<div>
   <h1>Recent Posts</h1>
   <ul>
       <li>
           <strong>@js_developers</strong>
           <span>Modern JS frameworks are too complicated</span>
       </li>
       <li>
           <strong>@jorgebucaran</strong>
           <span>There, I fixed it for you!</span>
       </li>
   </ul>
</div>
```
Translating between HTML and the `h` function calls can get tiresome for nested HTML.
Even if you automate the process, you still have to mentally switch between JS representation and HTML representation you see in DevTools.
However, if you write everything from scratch and prefer JS-driven templating, directly calling ```h``` function is a solid option.

### hyperlit

[hyperlit](https://github.com/zaceno/hyperlit) is a tiny library with HTML-like syntax and no build tool requirement.

Change you **App.js** code to use `hyperlit`:
```js
import {app} from "https://unpkg.com/hyperapp?module";
import html from "https://unpkg.com/hyperlit?module";

const state = {text: "Welcome to Hyperapp!"};

app({
    init: state,
    view: state => html`<h1 id="my-header"><span>${state.text}</span></h1>`,
    node: document.getElementById("app")
});
```
Write your HTML inside `html` tagged template literals. Under the hood `hyperlit` translates your HTML to the low-level `h` function calls.

Note: If you want to replace `hyperlit` with ```h``` calls in production, you can use `babel-plugin-hyperlit`. The plugin will
eliminate the runtime cost of the HTML parser.

