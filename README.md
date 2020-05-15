# Complete Intro to Hyperapp

This tutorial gives you full coverage of Hyperapp - a tiny framework for building web interfaces.

## How to read it

To benefit the most, read sections in order. I will explain every single concept with code, images and words. 
Later sections will build upon the previous ones, so don't skip anything.

## How long does it take

You will need about 6 hours.

## Prerequisites

* HTML, CSS, JS
* Basics of Functional Programming in JS:
    * pure functions and programming without side effects
    * higher-order functions
    * currying
    * immutability
* Some familiarity with browser development tooling (HTML elements tab, network tab, performance tab)  will be beneficial   
* Some familiarity with other JS frameworks will be beneficial    

No prior experience with Hyperapp is needed.

## Why do we need another framework?

<figure>
    <img src="images/rube_goldberg_machine.jpg" width="650" alt="Rube Goldberg Machine" align="center">
    <figcaption><em>Figure: Rube Goldberg Machine - a metaphore for accidental complexity</em></figcaption>
    <br><br>
</figure>


> We have lost our ability of achieving more with less. We do more with more. Or, in certain cases, we do less with more.

Modern frontend development is too complicated:
* frameworks with 10000k+ LOC, impenetrable to non-core developers
* tools with huge API surface area
* more and more JS "the bad parts" to learn
* complex tooling to make everything work
* layers upon layers of abstraction on top of the browser
* elaborate performance tricks to improve performance
* code mixing side-effects and application logic 
* testing techniques encouraging monkey patching the language

## What is your elevator pitch?

<figure>
    <img src="images/elevator_pitch.jpg" width="650" alt="Hyperapp Elevator Pitch" align="center">
    <figcaption><em>Figure: Hyperapp Elevator Pitch</em></figcaption>
    <br><br>
</figure>


```
For JS developers. 
Who need to build highly interactive Web interfaces  
Hyperapp is a functional view and state management framework
That allows for writing easy to understand, performant and testable code
Unlike other JS frameworks
Hyperapp fits in one 500 LOC file with zero dependencies and facilitates programming in a tiny subset of JS "the very best parts"
```
In this tutorial I'll try to back up this bold statement. 

## Learning outcomes

By the end of this tutorial you will learn to:
* develop frontend code using mostly pure functions and object literals 
* setup build tool free development workflow to avoid tooling fatigue
* render views as simple functions, not stateful components
* model application state to make impossible states impossible
* change application state with pure functions
* talk to HTTP APIs and stream server events without callbacks, promises, async/await, observables in the user space
* describe all side effects (e.g. random number generation) as data structures
* optimize load time and runtime performance 
* test your application at the unit and integration level
* deploy your code to production
* render frontend views on the server
* route between different pages
* integrate with 3rd party libraries
* appreciate the benefits of minimalist approach to software development

## Introducing the problem

You will be building a messaging/chat application called **HyperPosts**. I decided to choose a boring and familiar domain purposefully. 
To ease the learning process you will only focus on the framework concepts instead of learning new and unfamiliar domain.

You will find a deployed version of this app on netlify: https://hyperposts.netlify.app/

You will find source code on github: https://github.com/kwasniew/hyperbook-tutorial

The following two figures show 2 screens you'll be building:
* main screen with a post submit form and a list of posts streamed from the server
* login screen to set your username

<figure>
    <img src="images/main.png" width="650" alt="Hyperposts main screen" align="center">
    <figcaption><em>Figure: Hyperposts main screen</em></figcaption>
    <br><br>
</figure>

<figure>
    <img src="images/login.png" width="650" alt="Hyperposts login screen" align="center">
    <figcaption><em>Figure: Hyperposts login screen</em></figcaption>
    <br><br>
</figure>


## Getting started

When learning a new framework it's important to understand every single step you take and every single line of code you write. 
Instead of generating boilerplate, you'll write everything yourself. 

With more Hyperapp experience, you may formalize the setup into your own starter kit. 
However, you may also realize the starter kit is no longer necessary with certain sources of complexity eliminated.

Create empty **src** directory with **index.html** and **App.js**. You will name JS files with first uppercase letter. 

**index.html**
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <title>HyperPosts</title>
    <link rel="stylesheet" href="https://andybrewer.github.io/mvp/mvp.css">
    <script type="module" src="App.js"></script>
</head>
<body>
    <main>
        <div id="app"></div>
    </main>
</body>
</html>
```
HTML links to **App.js** as ES6 module (```type="module"```), therefore you can use ES6 imports in JS code. 
Hyperapp will render its content into ```<div id="app"></main>```.


**App.js**
```javascript
import {h, app} from "https://unpkg.com/hyperapp?module";

const state = {text: "Welcome to Hyperapp!"};

app({
    init: state,
    view: state => h("h1", {id: "my-header"}, state.text),
    node: document.getElementById("app")
});
```
To start experimenting, you can fetch Hyperapp directly from CDN (e.g. unpkg.com). 
The exported module provides two functions: ```h``` and ```app```.

The **app** function is your main integration point with the framework. 
Pass an object with 3 parameters:
* **init** - initial state of your application
* **view** - view function rendering current state
* **node** - DOM node to mount the application to

Open your project directory using any static HTTP server. I'm using https://www.npmjs.com/package/http-server 
```
npm i http-server -G
http-server src
```

By default ```http-server``` starts on http://127.0.0.1:8080

Check if you browser renders same HTML as in the following figure:
<figure>
    <img src="images/getting_started.png" width="650" alt="Getting started result" align="center">
    <figcaption><em>Figure: Getting started HTML</em></figcaption>
    <br><br>
</figure>

## Understanding view function

<figure>
    <img src="images/view.jpg" width="650" alt="View as a function of state" align="center">
    <figcaption><em>Figure: View as a function of state</em></figcaption>
    <br><br>
</figure>

In the functional approach to UI development, view is a pure function of state. 
Hyperapp ```view``` function takes ```state``` object as an input and returns a data structure describing future DOM tree to build. 
The returned data structure is known as the Virtual DOM. The framework can translate it into very efficient low-level DOM updates. 
The important point is that you never work directly with DOM API in your application code. 
Instead of making imperative calls such as ```document.createElement```, ```element.insertBefore``` or ```element.removeChild``` you declare
what the view should look like and call it a day.  

## Analyzing view rendering options

View function needs to build a Virtual DOM data structure. You have at least 3 options to choose from:
* ```h```
* ```JSX``` translating to ```h``` at build time
* ```htm``` translating to ```h``` at runtime or build time


### h

Currently your application uses built-in **h** function to make Virtual DOM nodes.
Change your view function to wrap the text in a ```span``` element:
```javascript
state => h("h1", {id: "my-header"}, [h("span", {}, state.text)])
```
Check the generated HTML:
```html
<h1 id="my-header"><span>Welcome to Hyperapp!</span></h1>
```

What about something more complicated? How much effort would it take to translate the following snippet into ```h``` function calls?
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
Translating between HTML and ```h``` function calls can get tiresome for nested HTML. 
Even if you automate the process, you still have to mentally switch between JS representation and HTML representation you inspect in DevTools.
On the other hand if you write everything from scratch and prefer JS-driven templating, calling ```h``` function directly is a solid option. 

### JSX 

[JSX](http://facebook.github.io/jsx/) is a language extension that originated in the React circles. It allows to write JS code that looks like HTML:
```jsx
view: state => <h1 id="my-header"><span>{state.text}</span></h1>
```

To make JSX work, you need to run a transpiler from JSX to ```h``` function calls. 
If adding a build step to your development process is not your thing, we have one more option.

### htm

[htm](https://github.com/developit/htm) is a tiny library with HTML-like syntax and no build tool requirement. 

Change you **App.js** code to use ```htm```:
```javascript
import {h, app} from "https://unpkg.com/hyperapp?module";
import htm from 'https://unpkg.com/htm?module';

const html = htm.bind(h);

const state = {text: "Welcome to Hyperapp!"};

app({
    init: state,
    view: state => html`<h1 id="my-header"><span>${state.text}</span></h1>`,
    node: document.getElementById("app")
});
```
```htm``` connects to Hyperapp via ```bind``` function. Write your HTML inside ```html``` tagged template. Under the hood ```htm``` translates everything to the low-level ```h``` function calls.

```htm``` works with any Virtual DOM framework matching the signature:
```javascript
function buildVirtualNode(type, props, ...children) {}
```
```h``` function happens to match the signature.

## Using Hyperapp from npm

Using modules directly from CDN is convenient for simple experiments. 
However, for the regular development you want to have local version of all dependencies. 
Why? Because sometimes CDNs:
* go down
* are slow to respond
* have security breaches
* go out of business

Also having your local dependencies allows to work offline.

Create **package.json** in your root directory:
```json
{
  "dependencies": {
    "htm": "3.0.4",
    "hyperapp": "2.0.4"
  }
}
```
Put the same versions of dependencies as this tutorial to avoid surprises.

Install dependencies:
```
npm i
```
On quick inspection of **node_modules** you'll find no transitive dependencies. Both ```htm``` and ```hyperapp``` bring no extra guests
to the party.

You can try to reference npm dependencies from **App.js**:
```javascript
import {h, app} from "hyperapp";
import htm from "htm";
```
But unfortunately browsers can't resolve those dependencies.

Since both ```hyperapp``` and ```htm``` are zero-dependency libraries you can load them from **node_modules**:
```javascript
import {h, app} from "../node_modules/hyperapp/src/index.js";
import htm from "../node_modules/htm/dist/htm.mjs";
```
It certainly works, but I had to inspect the contents of both libraries to provide correct paths.

## Integrating Hyperapp with Snowpack 

[Snowpack](https://www.snowpack.dev/) is a tool to translate selected ```node_modules``` into browser friendly bundles at dependency installation time.
In essence it makes bundling JS optional at development time.

Update **package.json** with this ```snowpack``` setup:
```json
{
  "scripts": {
    "snowpack": "snowpack install --dest=src/web_modules",
    "postinstall": "npm run snowpack"
  },
  "dependencies": {
    "htm": "3.0.4",
    "hyperapp": "2.0.4"
  },
  "devDependencies": {
    "snowpack": "2.0.0-beta.20"
  }
}
```
Snowpack is our development dependency. It provides ```snowpack install``` command that you will run after ```npm i```. 
You tell snowpack to put the browser friendly bundles in ```src/web_modules```. By default it would put everything in the root-level
```web_modules```.

Rewrite your imports to use ```web_module```:
```
import {h, app} from "./web_modules/hyperapp.js";
import htm from "./web_modules/htm.js";
```
 
Run the installation command:
```npm i```

At this point Snowpack will inspect your code and translate required modules from ```node_modules``` to ```web_modules```.

If you track your code in git add **src/web_modules** to **.gitignore**.

## Formatting code with prettier 

[Prettier](https://prettier.io/) is an opinionated code formatter saving your code review time for things that really matter. 
The days of spaces vs tabs wars are over.

Add ```format``` command and ```prettier``` ```devDependency``` to **package.json**:
```json
{
  "scripts": {
    "snowpack": "snowpack install --dest=src/web_modules",
    "postinstall": "npm run snowpack",
    "format": "prettier --write '**/!(web_modules)/*.js'"
  },
  "dependencies": {
    "htm": "3.0.4",
    "hyperapp": "2.0.4"
  },
  "devDependencies": {
    "prettier": "2.0.5",
    "snowpack": "2.0.0-beta.20"
  }
}
```
```format``` command willl format your JS files except from the ```web_modules``` and ```node_modules``` (excluded by default).
And ```--write``` option will re-write the formatted files in place.


Copy this malformed code to **App.js**:
```javascript
import { h, app } from "./web_modules/hyperapp.js";
import htm from "./web_modules/htm.js";

const html = htm.bind(h);

const state = { text: "Welcome to Hyperapp!" };

app({
  init: state,
  view: (state) => html`
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
`,
  node: document.getElementById("app"),
});
```
The opening ```div``` is not aligned properly.

After running:
```
npm run format
```
The ```view``` code should get nicely aligned.

You can connect prettier to your IDE/text editor but it's beyond the scope of this tutorial.

You took a detour to learn about some tools that play nicely with Hyperapp:
* ```htm``` for HTML-like syntactic sugar
* ```snowpack``` for browser friendly dependencies
* ```prettier``` for consistent code formatting

In the next section we're back to your app.

## Splitting view into smaller functions

In the previous section you rendered static HTML independent of the application state. 
The next code snippet shows how to iterate over a list of items in a view function:

```javascript
import { h, app } from "./web_modules/hyperapp.js";
import htm from "./web_modules/htm.js";

const html = htm.bind(h);

const state = {
  posts: [
    {
      username: "js_developers",
      body: "Modern JS frameworks are too complicated",
    },
    { username: "js_developers", body: "Modern JS frameworks are too heavy" },
    { username: "jorgebucaran", body: "There, I fixed it for you!" },
  ],
};

const listItem = (post) => html`
  <li>
    <strong>@${post.username}</strong>
    <span> ${post.body}</span>
  </li>
`;

const view = (state) => html`
  <div>
    <h1>Recent Posts</h1>
    <ul>
      ${state.posts.map(listItem)}
    </ul>
  </div>
`;

app({
  init: state,
  view,
  node: document.getElementById("app"),
});
```
View is extracted into a separate function.
Inside the ```view``` you map over a list of ```posts``` and render each of them using ```listItem``` view fragment. 
As a rule of thumb, if your view gets too big, split it into **smaller view fragments**. 
Pass as much state as needed. For example: ```listItem``` only needs a single ```post``` parameter.

At the end of this section your view should look like this:
<figure>
    <img src="images/splitting_view.png" width="650" alt="Displaying a list of posts" align="center">
    <figcaption><em>Figure: Displaying a list of posts</em></figcaption>
    <br><br>
</figure>

## Changing state with actions

**Actions** bring interactivity to your application. As users click buttons or type some text, you want to react to those events.

First, add a button just below the ```h1``` element:
```javascript
<h1>Recent Posts</h1>
<button onclick=${AddPost}>Add Post</button>
```
The ```onclick``` translates to DOM API click event. 
More precisely, Hyperapp translates the ```onclick``` into ```button.addEventListener('click')```. 
Everything you know about DOM API is still relevant and transferable. 
There's no extra framework-specific events to learn.

Add the action itself. Put it between the state and view declarations:
```javascript
const AddPost = (state) => {
  const newPost = { username: "fixed user", body: "fixed text" };
  return { ...state, posts: [newPost, ...state.posts] };
};
```
```AddPost``` is a pure function mapping previous state to the new state. 
When you click a button, Hyperapp automatically passes previous state to your action. 
```newPost``` is created and added to the beginning of the posts list. 
A common pattern is to destructure previous state and only update those properties that change. 
Our current state has no other properties, but the code is future proofed. 

The following figure shows the same action in a visual format:
<figure>
    <img src="images/action.jpg" width="650" alt="Action is a pure function of state" align="center">
    <figcaption><em>Figure: Action is a pure function of state</em></figcaption>
    <br><br>
</figure>


Test your app in the browser and click the "Add Post" button several times. New items should be added to the list.

<figure>
    <img src="images/add_post_action.png" width="650" alt="AddPost action adding new items to the list" align="center">
    <figcaption><em>Figure: AddPost action adding new items to the list</em></figcaption>
    <br><br>
</figure>

## Understanding functional data flow

Hyperapp **data flow** is inspired by the [Elm Architecture](https://guide.elm-lang.org/architecture/):
* view **V** interaction (e.g. click) triggers some action **A** 
* action **A** changes state **S**
* state **S** change triggers re-render of the view **V**

<figure>
    <img src="images/data-flow.jpg" width="650" alt="Functional data flow" align="center">
    <figcaption><em>Figure: Functional data flow</em></figcaption>
    <br><br>
</figure>

As a Hyperapp user you declare all the views, actions and the initial state. 
Hyperapp connects the circles and takes care of:
* handling events
* dispatching actions
* re-rendering the view

This approach makes your code very declarative as you never have to perform fine-grain view updates. 
At any given time, your view is the HTML/DOM projection of your current state.
And state is the ultimate source of truth. 
In other words, state is not spread across many JS components or even worse, in the DOM itself.

Note: with Hyperapp there's no need to use classes extending from a framework superclass or to decorate your code with framework specific annotations. 
View and actions are pure functions and state is a plain JS object. 
Therefore, cognitive overhead from unnecessary language features is minimal.

## Modelling state

You already implemented a button click action. It always adds a post with the same text. 

Add an input field to change the text:
```javascript
<h1>Recent Posts</h1>
<input type="text" autofocus />
<button onclick=${AddPost}>Add Post</button>
```

When you start typing some text, your state and view will get out of sync. What you're typing is not
reflected in the state change.
One of the tenets of functional UI architecture is continuous synchronization of state and view. 
View reacting to state changes, and state changes reacting to view actions. 
To make this work, you need some part of your state to model the contents of the input field. 

Create a new state property named ```currentPostText```:
```javascript
const state = {
  currentPostText: "type your text",
  posts: [...]
};
```
Read the new property in your view:
```javascript
<input type="text" value=${state.currentPostText} autofocus />
```
DOM attribute called ```value``` sets the text of the input field to the ```currentPostText```.

## Accessing DOM events

Input text reflects ```currentPostText``` from the state object. You want to close the circle with DOM events changing the state.

Add DOM ```oninput``` attribute to trigger ```UpdatePostText``` action on input changes:
```javascript
<input type="text" oninput=${UpdatePostText} value=${state.currentPostText} autofocus />
```

Add a new action next to the ```AddPost``` action:
```javascript
const UpdatePostText = (state, event) => ({
    ...state,
    currentPostText: event.target.value
});
```
Compare ```UpdatePostText``` signature with ```AddPost``` signature.

```
(oldState) => newState
(oldState, event) => newState
```
Hyperapp actions accept either ```(oldState)``` or ```(oldState, event)```. 
With a second attribute provided, Hyperapp will inject both sources of information to your action.
The ```event``` is a regular DOM event, therefore we can access ```event.target.value``` from DOM Event API. 
As mentioned before, it's all about transferable skills. 

The following figure shows updated conceptual model of Hyperapp actions with an extra event parameter:
<figure>
    <img src="images/action_with_event.jpg" width="650" alt="Action is a pure function of state and event" align="center">
    <figcaption><em>Figure: Action is a pure function of state and event</em></figcaption>
    <br><br>
</figure>

Try to add a new post with some text. It should still not work. You need to copy the ```currentPostText``` to the newly added post.

```javascript
const AddPost = (state) => {
  const newPost = { username: "fixed user", body: state.currentPostText };
  return { ...state, posts: [newPost, ...state.posts] };
};
```

With this change, you can start adding custom messages to the list.

<figure>
    <img src="images/custom_messages.png" width="650" alt="Adding custom messages to the list" align="center">
    <figcaption><em>Figure: Adding custom messages to the list</em></figcaption>
    <br><br>
</figure>

## Extracting repetitive event data

All event based actions will follow similar pattern:
```javascript
(oldState, event) => {
    const userData = event.target.value;
    ....
}
```
Action code would be cleaner if it didn't know about DOM Event API.

Create a **selector function** to extract part of the event you care about:
```javascript
const targetValue = event => event.target.value;
```
Eventually, you'll move this code to a library but for now put it somewhere above your view declarations.

Switch ```UpdatePostTest``` to use the new function:
```javascript
const UpdatePostText = (state, event) => ({
    ...state,
    currentPostText: targetValue(event)
});
```
The code is still dependent on the ```targetValue``` function.

Ideally, you'd like the action to accept only the data it needs:
```javascript
const UpdatePostText = (state, currentPostText) => ({
    ...state,
    currentPostText
});
```
Shape the second argument of your action inside the input handler. 
A two argument array with an action and a selector applies the event selector before the action is invoked. 
In our case ```targetValue``` is applied to DOM event before invoking ```UpdatePostText```.
```javascript
<input type="text" oninput=${[UpdatePostText, targetValue]} value=${state.currentPostText} autofocus />
```

If you keep using the ```[action, selector]``` array over and over, consider creating an alias:
```javascript
const UpdatePostTestAction = [UpdatePostText, targetValue];
```
As you don't have a second usage of this pattern yet, withhold this decision for now. 

## Exercise: cleaning text input

According to [modern reaserch](https://en.wikipedia.org/wiki/Desirable_difficulty), testing your knowledge is essential for learning. 
If you want to get the most out of this tutorial please do the exercises. They are not optional.

Your application doesn't clear the input text after adding a new post.
Modify the ```AddPost``` action to reset ```currentPostText```. 
Also make sure the initial text is empty.
When you're done compare with the solution below.
But first, try to do it on your own. 

<details>
    <summary id="cleaning_text_input">Solution</summary>

```javascript
const AddPost = (state) => {
  const newPost = { username: "fixed user", body: state.currentPostText };
  return { ...state, currentPostText: "", posts: [newPost, ...state.posts] };
};
```

</details>

## Exercise: checking empty input

After we clean the input, users may be tempted to submit empty text. Your task is to prevent them from doing so.
Application should ignore **Add Post** clicks when the text is empty.

<details>
    <summary id="checking_empty_input">Solution</summary>

```javascript
const AddPost = (state) => {
  if(state.currentPostText.trim()) {
      const newPost = { username: "fixed user", body: state.currentPostText };
      return { ...state, currentPostText: "", posts: [newPost, ...state.posts] };
  }  else {
      return state;
  }
};
```

</details>

## Understanding "effects as data"

All actions you've seen so far were simple state transitions from one data structure to the other. 
However in real-world scenarios your application will probably have to deal with side effect e.g. making HTTP calls to some API. 
A common functional approach to side-effects is to move them to the edges of the system. 

Imagine the following hypothetical code you could write:
```javascript
const SetPosts = (state, posts) => ({
  ...state,
  posts
});
const LoadLatestPosts = (state) => fetch("https://hyperapp-api.herokuapp.com/api/post").then(SetPosts);
```
```LoadLatestPosts``` uses browser fetch API to get data from the server. 
When data arrives it invokes simple state transition function to set the posts in a local state. 
Since ```fetch``` causes side effects (going over the wire with HTTP) it makes your entire program side-effectful. 
It only takes one innocent ```fetch``` call to make the code impure.

Another thought experiment is to represent the effect as a data structure:
```javascript
const SetPosts = (state, posts) => ({
  ...state,
  posts
});
const LoadLatestPosts = {
  url: "https://hyperapp-api.herokuapp.com/api/post",
  action: SetPosts
};
```
```LoadLatesPosts``` is an object with API ```url``` and follow-up ```action``` to invoke after the fetch completes. 
Ideally we'd like to pass this object to Hyperapp and let it get the posts from the API on our behalf. 
We don't want to fetch the data ourselves in the userland code. 
This is the essence of moving impure code to the edges of the system. Framework does the impure part, while your code stays very declarative. 

But how should Hyperapp know how to interpret this object? 

<figure>
    <img src="images/arbitrary_effect.jpg" width="650" alt="Hyperapp can't interpret arbitrary data" align="center">
    <figcaption><em>Figure: Hyperapp can't interpret arbitrary data</em></figcaption>
    <br><br>
</figure>


There's no way it can possibly translate arbitrary JS objects to every single side-effect you can imagine. 
That's why it doesn't even try. Instead, you must pass side-effect definition and the effect data as a two-argument array.

```javascript
const LoadLatestPosts = [effectDefinition, {
  url: "https://hyperapp-api.herokuapp.com/api/post",
  action: SetPosts
}];
```

Side-effects in Hyperapp are made of effect definition and effect data:
```javascript
[effectDefinition, data]
```

The effect definition will hide the ```fetch``` call or some other impure call, but you will never invoke it in the userland code.
It's something you must return to the framework, so it can handle the impure parts.

<figure>
    <img src="images/effect_definition.jpg" width="650" alt="Hyperapp can handle effect definitions from the userland" align="center">
    <figcaption><em>Figure: Hyperapp can handle effect definitions from the userland</em></figcaption>
    <br><br>
</figure>

## Implementing "effects as data"

In this section you'll use an open source library [hyperapp-fx](https://github.com/okwolf/hyperapp-fx) that implements the most common effects. 
In the next section we'll peek under the hood and build your own effects.

In **App.js** add ```LoadLatestPosts``` effect that invokes ```SetPost``` action on successful response:
```javascript
import { Http } from "./web_modules/hyperapp-fx.js";

const SetPosts = (state, posts) => ({
  ...state,
  posts
});

const LoadLatestPosts = Http({
  url: "https://hyperapp-api.herokuapp.com/api/post",
  action: SetPosts
});
```
```Http``` function takes your effect data and builds a two argument array with ```[httpEffectDefinition, effectData]```.

Add ```hyperapp-fx``` and let Snowpack bundle it for the browser:
```
{
  "dependencies": {
    "htm": "3.0.4",
    "hyperapp": "2.0.4",
    "hyperapp-fx": "2.0.0-beta.1"
  }
}
```

```npm i```

## Triggering effects on application startup

With HTTP effect defined you must decide when to invoke it. For now, you'll do it on application startup to start fetching posts early.

Modify ```init``` to invoke ```LoadLatestPosts```:
```javascript
app({
  init: [state, LoadLatestPosts],
  ...  
});
```
```init``` has overloaded signature. In addition to the initial state you can pass one or more actions to invoke on startup.

With those changes in place test your application. A list of posts from the server should arrive and replace the hardcoded posts. 
You may observe a content flip as Hyperapp replaces initial state with server posts.

<figure>
    <img src="images/initial_posts.png" width="650" alt="Loading initial posts" align="center">
    <figcaption><em>Figure: Loading initial posts</em></figcaption>
    <br><br>
</figure>

If everything works fine, replace the initial posts with an empty array:
```javascript
const state = {
  posts: []
};
```
The initial content flip should be gone.

## Writing your own effects

Most of the time you don't need to write your own effects. However, to better understand the underlying concepts implement
Http effect yourself.

Comment out this line of code:
```javascript
// import { Http } from "./web_modules/hyperapp-fx.js";
```

Our own implementation of the effect should build an array with the effect definition and data.
```javascript
const Http = data => [httpEffect, data];
```
Hyperapp expects 2 parameter signature in the effect definition:
```javascript
const httpEffect = (dispatch, data) => {};
```

A simple implementation of the HTTP effect may look like this:
```javascript
const httpEffect = (dispatch, data) => {
  return fetch(data.url)
      .then(response => response.json())
      .then(json => dispatch(data.action, json));
};
```
As an effect library author you translate the side-effectful API call (e.g. fetch) into the dispatch call. Since you never call this function yourself you don't need to care about the internals of the dispatch function. Hyperapp will pass it for you when it handles the effect. 

Our original post-fetch action definition looked like this:
```javascript
const SetPosts = (state, posts) => ({
  ...state,
  posts
});
```
Dispatch call will replace the second parameter with the actual JSON data from the API. The first parameter will be a regular state object that we used before.

Test your own implementation of the Http effect. If everything works uncomment the original Http effect from the library:

```javascript
import { Http } from "./web_modules/hyperapp-fx.js";
```

## Understanding effectful actions

LoadLatestPosts Http effect is invoked on application startup. What if we wanted to trigger the effects from regular actions?

Create SavePost effect:
```javascript
const SavePost = (post) =>
    Http({
      url: "https://hyperapp-api.herokuapp.com/api/post",
      options: {
        method: "post",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify(post)
      },
      action: (state, data) => state
    });
```
This effect wraps HTTP POST to the API. The action to be triggered on successful response is not doing anything.

The signature of effectful actions looks like this:
```javascript
const EffectfulActions = oldState => [newState, Effect];
```
If you have more than one effect wrap them in an array:
```javascript
const EffectfulActions = oldState => [newState, [Effect1, Effect2]];
```
Hyperapp applies the new state and schedules the effect almost instantly. The action inside the effect will trigger eventually e.g. when the HTTP response arrives.

## Exercise: making effectful action

Change ```AddPost``` action to trigger SavePost effect when post is added to the local state.

<details>
    <summary id="making_effectful_action">Solution</summary>

```javascript
const AddPost = state => {
  if (state.currentPostText.trim()) {
    const newPost = { username: "fixed", body: state.currentPostText };
    const newState = { ...state, currentPostText: "", posts: [newPost, ...state.posts] };
    return [newState, SavePost(newPost)];
  } else {
    return state;
  }
};
```

</details>

## Understanding long running effects (subscriptions)

The HTTP effect you've been using so far was **short-lived**. A request is sent, a response arrives and the effect is over.
Not all effects fit this pattern. E.g. if you open a connection to a WebSocket it will be running for a long time, unlike the short-lived HTTP request-response model. 

In Hyperapp you use so-called **subscriptions** to handle those **long-lived** effects. 

To build intuition about subscription take a look at some other even sources that fit the model:
* setInterval
* mouse moves
* keybord key presses
* history/URL changes 

What they have in common is a long-lived nature of the underlying event source. 

Note: From now one, we'll refer to short-live effects as just effects and to long-lived effects as subscriptions.

## Implementing subscriptions

In this section you will subscribe to the WebSocket stream with post updates.

Import subscription definition:
```javascript
import { WebSocketListen } from "./web_modules/hyperapp-fx.js";
```
hyperapp-fx uses a convention ```*Listen``` to name the functions for creating subscriptions.

Write the action for handling WebSocket events:
```javascript
const SetPost = (state, event) => {
    try {
        const post = JSON.parse(event.data);
        return {
            ...state,
            posts: [post, ...state.posts]
        }
    } catch(e) {
        return state;
    }
};
```
The event is the underlying ```MessageEvent``` from the WebSocket API. You parse the ```data``` property of the event. If the data is valid JSON you add the post to the beginning of the post list. In case of a parsing error you don't change the state of the application.

Plug the subscription and the action into the application:
```javascript
app({
  ...
  subscriptions: state => [WebSocketListen({action: SetPost, url: 'ws://hyperapp-api.herokuapp.com'})],
  ...
});
```
```WebSocketListen``` function expects an object with ```action``` and ```url```.
Since WebSockets is a protocol other than HTTP we changed the URL scheme to ```ws://```.

Test your application. Add a new post. The post should be added to the list twice. Directly from the local state update and from the WebSocket. You'll fix this behavior in the next exercise. For now, test your WebSocket connection in two different browser windows. See if the messages are propagated correctly.

Diagnosing problems with WebSockets:
* make sure the HTTP protocol was switched to WebSockets
![Switching Protocols](https://i.imgur.com/ykeYaPv.png)
* click on the switching protocols raw and find the actual message
![Sending Messages](https://i.imgur.com/XC1V2n1.png)

## Exercise: avoiding duplicate posts

Your task is to change the code, so that it only adds a post from the WebSocket. Modify AddPost action and stop adding newPost before we receive a confirmation from the server.

<details>
    <summary id="avoiding_suplicate_posts">Solution</summary>

Inside AddPost actions change this line:
```javascript
const newState = { ...state, currentPostText: "", posts: [newPost, ...state.posts] };
```
To this:
```
const newState = { ...state, currentPostText: "" };
```

</details>

## Writing your own subscription

In this section you'll write your own subscription for [Server-Sent Events](https://www.smashingmagazine.com/2018/02/sse-websockets-data-flow-http2/).

Server-Sent Events (SSE) is a lesser known, but much simpler HTTP-native alternative to WebSockets. SSE also handles network failures more gracefully than plain WebSockets.

In the [Writing your own effects](#writing-your-own-effects) section you used the following signature for effect definition:
```
const httpEffect = (dispatch, data) => {};
```
Start with the same signature for the subscription definition:
```javascript
const eventSourceSubscription = (dispatch, data) => {

};
```
The Web API to connect to SSE looks looks like this:
```javascript
const es = new EventSource("https://hyperapp-api.herokuapp.com/api/event/post");
es.addEventListener("message", event => /* handle event with a data field */)
```
The browser API for SSE is called EventSource and it is a regular event emitter similar e.g. to a button with click handlers.

Wrap the API into our subscription definition:
```javascript
const eventSourceSubscription = (dispatch, data) => {
    const es = new EventSource(data.url);
    es.addEventListener("message", event => dispatch(data.action, event));
};
```
```data``` parameter will hold two configuration options: url and action. It is the same convention that was used in the WebSockets implementation. When the event arrives dispatch action and pass the server event.

In the [Writing your own effects](#writing-your-own-effects) section you used the actual effect signature:
```javascript
const Http = data => [httpEffect, data];
```

Following the same convention create your own subscription:
```
const EventSourceListen = data => [eventSourceSubscription, data];
```


Start using the subscription in your application:
```javascript
app({
  subscriptions: state => [EventSourceListen({action: SetPost, url: 'https://hyperapp-api.herokuapp.com/api/event/post', event: 'post'})]
});
```
Because you followed the same nameing convention for action and url it should be just a matter of switching WebSocketListen to EventSourceListen.

Test your application. It should work the same as the WebSocket version, but without a need for a different protocol. 

Diagnosing problems with WebSockets:
* make sure the eventsource type was sent over HTTP
![Eventsource Type](https://i.imgur.com/ehHPEG3.png)
* click on the eventsource raw and find the actual message
![Sending Messages](https://i.imgur.com/hR5esEk.png)

Note: if your browser doesn't support SSE use a polyfill:
```html
<script src="https://polyfill.io/v3/polyfill.min.js?features=fetch%2CEventSource%2Cdefault" defer></script>
```

## Understanding differences between init effect and subscription

Looking at our subscription signature it's not much different from any short-live effect. 
You could event plug the subscription into the application:
```javascript
app({
    init: [state, LoadLatestPosts, EventSourceListen({action: SetPost, url: 'https://hyperapp-api.herokuapp.com/api/event/post'})],
    ...
});
```
Both the short-lived ```LoadLatestPosts``` action and long-lived ```EventSourceListen``` subscription are invoked on the application init.

If you never need to stop listening to the long-running event source, the subscription is effectively the same as the init action.
The moment you need to stop listening to the event source they start to differ.

## Unsubscribing from subscriptions

Subscriptions are long-lived effects you can unsubscribe from. The code to unsubscribe lives in the return function of the subscription definition.
```javascript
const eventSourceSubscription = (dispatch, data) => {
   return () => {
      // unsubscribe here
   };
};
```

Fill in this template with your EventSource implementation:
```javascript
const eventSourceSubscription = (dispatch, data) => {
    const es = new EventSource(data.url);
    const listener = event => dispatch(data.action, event);
    es.addEventListener("message", listener);

    return () => {
        es.removeEventListener("message", listener);
    };
};
```
The unsubscribe function removes a listener from the event source. ```addEventListener``` and ```removeEventListener``` need a reference to the same listener. Therefore, put the listener in a shared variable.

## Controlling subscription status

You will add a capability to enable/disable live updates through the UI.

Introduce intial state field for liveUpdate control set to true:
```javascript
const state = {
  ...
  liveUpdate: true
};
```
By default we'll be listening to SSE notifications.

Add actions to toggle live update settings:
```javascript
const ToggleLiveUpdate = state => ({...state, liveUpdate: !state.liveUpdate});
```

Add those two lines just below the Add Post button.
```javascript
 <input type="checkbox" id="liveUpdate" onchange=${ToggleLiveUpdate} checked=${state.liveUpdate}/>
 <label for="liveUpdate">Live Update</label>
```
The checkbox reflects the liveUpdate state. Every time the checkbox changes we toggle the live update setting.
Label for the input field conveniently allows to click on the "Live Update" text to change the setting.

React to liveUpdate setting:
```javascript
app({
    ...
    subscriptions: state => [state.liveUpdate && EventSourceListen({action: SetPost, url: 'https://hyperapp-api.herokuapp.com/api/event/post'})],
    ...
});
```
```app.subscription``` function allows to control the subscription status based on the current state. When ```state.liveUpdate``` is true we create a new subscription. When ```state.liveUpdate``` is false we usubscribe from the subscription.

## Handling slow API

Switch SavePost to a new url of the slow API.
```javascript
const SavePost = (post) =>
  Http({
    url: "https://hyperapp-api.herokuapp.com/slow-api/post",
    ...
  });
```
When you test the app ```SavePost``` should take about 3 seconds before a notification arrives. While waiting for the response you can send more requests without getting confirmation that the previous ones succeeded. Assume you have a requriement to disable the "Add Post" button while the post is saving.

Enhance initial state with the isSaving property.
```javascript
const state = {
  ...
  isSaving: false
};
```

Map the state property to the button disable property:
```javascript
<button onclick=${AddPost} disabled="${state.isSaving}">Add Post</button>
```
From now on the button reflects the current state of the saving operation.

AddPost action disables the button:
```javascript
const AddPost = (state) => {
  ...
    const newState = { ...state, currentPostText: "", isSaving: true };
  ...
};
```

```SavePost``` effectful action enables the button on successful response with a help of a new ```PostSaved``` action:
```javascript
const PostSaved = state => ({...state, isSaving: false});

const SavePost = (post) =>
  Http({
    ...
    action: PostSaved,
  });
```

## Handling API errors

Switch SavePost to a new url of the error API.
```javascript
const SavePost = (post) =>
  Http({
    url: "https://hyperapp-api.herokuapp.com/error-api/post",
    ...
  });
```
The new API is not only slow, but also returns 500 errors.

Enhance initial state with the error property.
```javascript
const state = {
  ...
  error: ""
};
```
Eventually you will populate this field with an error value.

Map the error property to the error text in the UI:
```javascript
<div>${state.error}</div>
<button onclick=${AddPost} disabled="${state.isSaving}">Add Post</button>
```
You should add the error just above the "Add Post" button.

Add ```PostError``` action that will be triggered on error. hyperapp-fx Http effect has a special error field for the error handling action.
```javascript
const PostSaved = state => ({...state, isSaving: false});
const PostError = state => ({...state, isSaving: false, error: "Post cannot be saved. Please try again."});

const SavePost = (post) =>
  Http({
    ...
    action: PostSaved,
    error: PostError,
  });
```
```PostError``` should enable the "Add Post" button and set the error message. 

Test your application. After the post submission the error is displayed. When you start typing a new text the error is still there. I'd expect the error to dissapear at this point. You can change ```SetPost``` action to remove the error message, but in the next section we'll see a better way to model state.

## Modeling only valid states

Take a look at the last 2 fields you added to the state:
```javascript
const state = {
  ...
  isSaving: false,
  error: ""
};
```

There's 4 possible combinations that can be triggered:
* ```isSaving: false``` and empty error (request is idle, user is typing a new message)
* ```isSaving: false``` and non-empty error (request error after form submission)
* ```isSaving: true``` and empty error (request is pending)
* ```isSaving: true``` and non-empty error (should never be possible)

The last combination should be impossible. But the way we modelled our state makes it possible. Of course you can write tests to verify the combination is never triggered. But you can also model your state to make the unwanted state impossible.

Think of a concept of the request status.
The request status can be in one of the states:
* ```{status: "idle"}```
* ```{status: "pending"}```
* ```{status: "error", message: "Post cannot be saved. Please try again."}```

In the next section you'll implement it.

## Implementing only valid states

Introduce 3 valid states we defined in the modelling exercise and set the idle state as the initial one.
```javascript
const idle = {status: "idle"};
const saving = {status: "saving"};
const error = {status: "error", message: "Post cannot be saved. Please try again."};
const state = {
  ...
  requestState: idle
};
```
A strategy to scale state is to split one big object into smaller objects and combine them.

Find all the places where you were setting ```isSaving``` and ```error``` properties. 

```AddPost``` sets the state to saving:
```javascript
const AddPost = (state) => {
  ...
    const newState = { ...state, currentPostText: "", requestState: saving };
  ...
};
```

```PostSaved``` sets the state to idle:
```javascript
const PostSaved = state => ({...state, requestState: idle});
```

```PostError``` sets the state to error:
```javascript
const PostError = state => ({...state, requestState: error});
```

Map the request state to the error message view fragment:
```javascript
const errorMessage = (requestState) => {
  if(requestState.status === "error") {
    return html`
        <div>${requestState.message}</div>
  `;
  }
  return "";
};
```

Map the request state to the correct button disabled status:
```javascript
const addPostButton = (requestState) => html`
        <button onclick=${AddPost} disabled=${requestState.status === "saving"}>Add Post</button>
  `;
```

Delete those two lines:
```javascript
<div>${state.error}</div>
<button onclick=${AddPost} disabled="${state.isSaving}">Add Post</button>
```
And replace them with our new view functions:
```javascript
${errorMessage(state.requestState)}
${addPostButton(state.requestState)}
```
A strategy to scale view functions is to split them into smaller view fragments.

After this part revert your API to: https://hyperapp-api.herokuapp.com/api/post

## Exercise: removing error when typing a new post

Modify ```UpdatePostText``` action to remove the error when a user starts typing a new post.

<details>
    <summary id="cleaning_text_input">Solution</summary>

```javascript
const UpdatePostText = (state, currentPostText) => ({
  ...state,
  requestState: idle,
  currentPostText,
});
```

</details>

## Breaking from purity

Effects/subscriptions as data taken to the extreme means no side-effects in the userland code. setTimeout becomes an effect, setInterval becomes a subscription. HTTP calls become effects, SSE become subscriptions. 
What about things like console.log or Math.random()? We can wrap them inside effects but sometimes it's more convenient to just use them directly in your code. 

Put this guid function based on Math.random() into your codebase:
```javascript
const guid = () => {
    return 'xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx'.replace(/[xy]/g, function (c) {
        var r = Math.random() * 16 | 0, v = c == 'x' ? r : (r & 0x3 | 0x8);
        return v.toString(16);
    });
}
```

Modify AddPost action to generate id for all new posts:
```javascript
const newPost = { id: guid(), username: "fixed", body: state.currentPostText };
```

## Optimising lists of items with keys

Our application keeps adding new posts to the top of the list. When Hyperapp sees a new item in the new list and compares it with the first item in the old list they differ. Same with the second and third and all the other items. In other words all the old items got shifted by one. We know it, but the algorithm for the Virtual DOM diffing doesn't. To maintain a stable list item identity between the renders add a **key** attribute.

![key attribute](https://camo.githubusercontent.com/b64d4e13c9eb4e3a1ac3fc6fcc569ca7bbf5563a/68747470733a2f2f692e696d6775722e636f6d2f655766385469582e706e67)
With the extra hint from the key attribute Hyperapp avoids re-rendering the items that got shifted by one.

Modify listItem view function:
```javascript
const listItem = (post) => html`
  <li key=${post.id} class="list-group-item" data-testid="item">
    ...
  </li>
`;
```
Usually the best candidate for the key value is a stable identifier like the one we used. Don't use post content because it may not be unique. Don't use array index as it's not stable over re-renders.

Since key attribute is not visible in the generated DOM you can also add data-key attribute for debugging purposes:
```javascript
const listItem = (post) => html`
  <li key=${post.id} data-key=${post.id} class="list-group-item" data-testid="item">
    ...
  </li>
`;
```
Hyperapp internally uses key and you can use data-key for insepection.

## Finding runtime performance bottlenecks

Switch ```LoadLatestPosts``` to fetch 1000 items when the application starts:
```javascript
const LoadLatestPosts = Http({
  url: "https://hyperapp-api.herokuapp.com/api/post?limit=1000",
  action: SetPosts
});
```

Start typing text into the text field. Every time you put a new character into the text field Hyperapp has to do a Virtual DOM diffing of the entire page it controls. If you have a fast machine the delay after typing a character may not be even noticeable. Hyperapp is often fast enough without any optimisations. 

Go to Devtools and slow down your CPU to make the impact of large DOM tree updates noticeable.

![Slow CPU](https://i.imgur.com/PoWNBjD.png)

Record CPU profile while typing the text. It should show significant time spent on JS execution:

![Impact of slow CPU](https://i.imgur.com/V45f8Pi.png)

Zoom in on the slow parts:

![Zoom in](https://i.imgur.com/ZCYDY4H.png)

render function seems to be the bottleck. But the render function belongs to Hyperapp so keep looking for the code that you wrote. Just below the render function you should have view function callling listItem mutliple times. The source of our bottlenect is listItem function invoked multiple times when we type the text.

## Optimising large DOM trees with memoization

You want to avoid the unnecessary computation of the post list items view when typing a new post text. 

Extract postList view fragment:
```javascript
const postList = ({posts}) => html`
  <ul>
    ${posts.map(listItem)}
  </ul>
`;
```
Use it in your main view:
```javascript
${postList({posts: state.posts})}
```

Import Lazy function from Hyperapp core:
```javascript
import { h, app, Lazy } from "./web_modules/hyperapp.js";
```
Decorate postList call with Lazy:
```javascript
const lazyPostList = ({posts}) => Lazy({view: postList, posts});
```
Lazy expects a view to be decorated and other properties that will be passed to the original view function.

Replace postList call with postListView call:
```javascript
${lazyPostList({posts: state.posts})}
```
lazyPostList uses so-called memoization. It remembers the input and output of the previous invocation. If you call it again with the same input the function doesn't compute anything and returns previously saved result.

Verify performance profile again.

Most key presses should generate a pattern with much shorter JS execution times:

![optimized](https://i.imgur.com/YZBLxBj.png)

## Understanding testable architecture

In the object-oriented programming circles people often talk about creating testable architecture that goes by different names:
* functional core, imperative shell
* ports and adapters architecture
* and many others (onion, hexagonal, clean etc.)

Lengthy books are written how to achieve the holy grail and much effort is required. 

Functional architecture imposed by Hyperapp makes the holy grail a default. As Mark Seemann noted in his [blog](https://blog.ploeh.dk/2016/03/18/functional-architecture-is-ports-and-adapters/) "functional architecture tends to fall into a pit of success".

You can visualize your app as state in the center. Actions returning new state sitting around it. And effects/subscriptions at the edges.
![Functional Architecture](https://d82.intsig.net/sync/download_resize_jpg?user_id=1609921833&_t=1589277758&sid=D3A958EEF4EE4D39MK7yC8S6&folder_name=CamScanner_Page&file_name=tXT167Y7a7LT62S0dHT3LK1g.jpg&pixel=1024)

Your application is as a functional core with pure view functions, pure actions and immutable state.  
The framework is an imperative shell sitting at the edges, interpreting effectful actions and handling side-effects.

The functional core makes decisions and defines the shape of your UI, the shell converts the decisions into side effects, gathers the inputs and renders physical DOM nodes.

Your effect signature is a port to the external world. 
The actual effect definition invoked by the framework is an adapter.

In terms of testability functional core allows for simple output-based unit testing. Invoke a function. Assert on the output. No mocks required.

## Testing simple actions

```npm i mocha -D```

Why mocha and what are the tradeoffs I made choosing it?
* (+) as of this writing it has native EcmaScript Modules (ESM) support so you don't need transpiler in testing. Less tooling is always good. I don't want my test framework to run my code through babel if it doesn't need to.
* (+) it doesn't enourage questionable and magical testing practices like overwirting imports for testability. Relying on a test framework to mock imports is a dead end testing strategy. Your code can't be tested in other test runners. 
* (+) fast startup time that allows for subsecond tests with clean start (without watchers). This is super important if you want to get into the flow.
* (-) my main reservation about mocha is that it can't run plain Node.js files as tests. 

Make sure you have Node 14 installed as it ships native ESM support.

In package.json set Node.js to use ESM by default and set the test script:
```json
{
    "type": "module",
    ...
    "scripts": {
        "test": "mocha test/*Test.js"
    }
}
```

Write your first test in test/appTest.js
```javascript
import assert from "assert";
import {UpdatePostText} from "../src/app.js";

describe("App", () => {
    it("can update post test", () => {
        const initState = { currentPostText: "", requestState: {status: "idle"}};
        
        const newState = UpdatePostText(initState, "text");
        
        assert.deepStrictEqual(newState, { currentPostText: "text", requestState: {status: "idle"}});
    });
});
```
We use Node.js built-in assert module. Import statemets work without transpilation because latest Node.js versions have native ESM support (with ```"type": "module"``` set). The test prepares initial state. Then you invoke an action with a new post text. Finally you verify if the state update succeeds.

Run the test:
```npm test```

## Exercise: Testing simple actions

Write a unit test that verifies that ```UpdatePostText``` resets error request state to idle.

<details>
    <summary id="testing_actions">Solution</summary>

```javascript
    it("update resets request state to idle", () => {
        const initState = { currentPostText: "", requestState: {status: "error", error: "oh nooo"}};

        const newState = UpdatePostText(initState, "text");

        assert.deepStrictEqual(newState, { currentPostText: "text", requestState: {status: "idle"}});
    });
```

</details>

## Making actions more testable

```AddPost``` action is difficult to test because it relies on Math.random() for guid generation. 

Our desired signature should take pregenerated id as input parameter:
```javascript
const AddPost = (state, id) => {
  ...
};
```

Find where you use ```AddPost``` and replace it with:
```javascript
const addPostButton = (requestState) => html`
        <button onclick=${WithGuid(AddPost)} disabled=${requestState.status === "saving"}>Add Post</button>
  `;
```
```WithGuid``` doesn't exist yet but we're sketching our ideal API in code.

```WithGuid``` should ask Hyperapp to generate a new id and pass the id to a testable action:
```javascript
const WithGuid = action => state => [state, Guid(action)];
const Guid = action => [(dispatch, action) => {
  dispatch(action, guid());
}, action];
```
Previously you were passing configuration objects to your Http effects. But the Guid effect is simple and it only accepts the action to invoke. 

## Testing effectful action

Now you can test ```AddPost``` action.
```javascript
    it("add post", () => {
        const initState = { currentPostText: "text", requestState: {status: "idle"}, post: []};

        const [newState, [savePostEffect, savePostData]] = AddPost(initState, "1234");

        assert.deepStrictEqual(newState, { currentPostText: "", requestState: {status: "saving"}, post: []});
        assert.deepStrictEqual(savePostData.url, "https://hyperapp-api.herokuapp.com/slow-api/post");
    });
```
This action returns new state and the effect that we can destructure to conveniently assert on the test output. 
Action should clear the text, mark the request as saving and nothing should be added to the post list yet.
For the effect part we ignore savePostEffect as it's for the framework. On the other hand we can verify if we passed correct data to the effect. In our case we only check the url. 

To emphasize that we ignore the effect definition in the test we can name it with underscore or leave a blank in the destructured array.
```javascript
const [newState, [_, savePostData]] = AddPost(initState, "1234");
const [newState, [, savePostData]] = AddPost(initState, "1234");
```

## Separating application code from library code

Before you start testing effects and subscriptions separate them in code.
Create src/lib directory and move SSE and Guid related code there. Remember to export appropriate functions.

src/lib/eventsource/EventSource.js
```javascript
const eventSourceSubscription = (dispatch, data) => {
    const es = new EventSource(data.url);
    const listener = (event) => dispatch(data.action, event);
    es.addEventListener("message", listener);

    return () => {
        es.removeEventListener("message", listener);
    };
};
export const EventSourceListen = (data) => [eventSourceSubscription, data];
```

src/lib/guid/Guid.js
```javascript
const guid = () => {
    return "xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx".replace(/[xy]/g, function (c) {
        var r = (Math.random() * 16) | 0,
            v = c == "x" ? r : (r & 0x3) | 0x8;
        return v.toString(16);
    });
};
export const WithGuid = action => state => [state, Guid(action)];
export const Guid = action => [(dispatch, action) => {
    dispatch(action, guid());
}, action];
```

In app.js you should now have:
```javascript
import {EventSourceListen} from "./lib/eventsource/EventSource.js";
import {WithGuid} from "./lib/guid/Guid.js";
```
## Testing effects and subscriptions

Effects and subscriptions live at the edges of the system and need to talk to global APIs you don't control e.g. DOM API or fetch API. Therefore, effects and subscriptions are more difficult to unit test and it's left to the library authors providing those effects. 

```javascript
import assert from "assert";
import {EventSourceListen} from "../src/lib/eventsource/EventSource.js";

const runFx = ([effect, data]) => {
    const dispatch = (action, event) => dispatch.invokedWith = [action, event];
    const unsubscribe = effect(dispatch, data);
    return { dispatch, unsubscribe }
};

const eventServer = url => {
    const listeners = {};
    const emit = (event) => Object.values(listeners).forEach(listener => {
        listener(event);
    });

    class EventSource {
        constructor(url) {
            this.url = url;
        }
        addEventListener(name, listener) {
            if(this.url === url) {
                listeners[name] = listener;
            }
        }
        removeEventListener(name, listener) {
            if(this.url === url) {
                const registeredListener = listeners[name];
                if(registeredListener === listener) {
                    delete listeners[name];
                }
            }
        }
    }
    return {EventSource, emit};
};

const action = "action";

describe("Event source subscription", () => {
    const defaultEventSource = global.EventSource;
    afterEach(() => {
        global.EventSource = defaultEventSource;
    });
    it("dispatch message events", () => {
        const {emit, EventSource} = eventServer("http://example.com");
        global.EventSource = EventSource;
        const {dispatch, unsubscribe} = runFx(EventSourceListen({url: "http://example.com", action}));

        emit({data: "event data"});

        assert.deepStrictEqual(dispatch.invokedWith, ["action", {data: "event data"}])
    });
});
```
In the test we provide a fake version of the event server that will emit events. This server also gives you fake EventSource implementation you can put into a global object. ```runFx``` function simulates Hyperapp setting up a subscription. Then you emit a test event and verify if dispatch was called with the correct data. ```dispatch``` function is a manual mock with a custom invokedWith field to verify the intended intecraction. After the test runs you should revert original EventSource in the global namespace if there was one. 

The code and tests for effects/subscriptions is not as easy to reason about as the rest of the code. It's the essential complexity of the Web platform that you work with. And because this code is not very convenient to work with Hyperapp keeps it at the edges and doesn't allow your application code to get polluted. All those callback event listeners, eagerly resolving promises (e.g. fetch) are hidden away from your application logic.

## Exercise: Testing effects and subscriptions

Write a second test that verifies if emitting an event after unsubscribe triggers no actions.

<details>
    <summary id="testing_effects">Solution</summary>

```javascript
    it("unsubscribe from messages", () => {
        const {emit, EventSource} = eventServer("http://example.com");
        global.EventSource = EventSource;
        const {dispatch, unsubscribe} = runFx(EventSourceListen({url: "http://example.com", action}));

        unsubscribe();
        emit({data: "event data"});

        assert.deepStrictEqual(dispatch.invokedWith, undefined);
    });
```

</details>

## Comparing integration test options

If you find yourself struggling with excessive mocking and hard to maintain tests consider integration testing your application. Some JS developer found their integrations tests give them more leverage than unit tests and subverted a traditional test pyramid.

First option for integration tests is to emulate browser environment in Node.js with jsdom and polyfills for things like fetch, localStorage, SSE etc. Personally, I've spent much time trying to match browser environment in this setup and I'm not sure if it's worth the effort. Your mileage may vary though. Also, with jsdom you're not integration testing against a real browser. 

jsdom based setup tradeoffs:
* (+) easy to run from CLI without extra tooling
* (+) faster tests in CI server than browser tests
* (-) slow startup time for the first test which makes subsecond testing impossible
* (-) can't find browser discrepancies

Another option is to run your tests in a browser where you can inspect everything after a failing tests. Mocha test runner can be run in Node.js and in a browser which is not a case for all the other test runners. 

Browser based setup tradeoffs:
* (+) testing real browser
* (+) easy to inspect the environment after a failing tests
* (+) all APIs just work
* (+) with an open browser first test starts instantly in development 
* (-) cleaning the environment between tests is cumbersome
* (-) requires complex tooling to run in CI server
* (-) with many tests it's slower than jsdom because of the rendering overhead

## Testing in a browser

Create test/index.html with mocha browser test boilerplate:
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8" />
    <title>Mocha Tests</title>
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <link rel="stylesheet" href="https://unpkg.com/mocha/mocha.css" />
</head>
<body>
<div id="mocha">
    <div id="app"></div>
</div>

<script src="https://unpkg.com/chai/chai.js"></script>
<script src="https://unpkg.com/mocha/mocha.js"></script>

<script class="mocha-init">
    mocha.setup('bdd');
</script>
<script type="module" src="browser.test.js"></script>
<script type="module" class="mocha-exec">
    mocha.run();
</script>
</body>
</html>
```
This setup loads mocha and chai assertion library since browsers don't provide any assertion library as Node.js does.
```<div id="app"></div>``` is a placeholder where our app will mount.

Split app.js into app.js and init.js:

src/init.js is a new file you'll reference in index.html 
```javascript
import { init } from "./app.js";

init();
```

src/app.js should export init function.
```javascript
export const init = () =>
  app({
    ...
  });
```
The reason for this split is the ability to initialize a new instance of a new app per test.

Integration testing heavily relies on DOM elements rendering asynchronously. Install a library to facilitate wating for those elements. 
```
  "webDependencies": {
    "@testing-library/dom": "7.5.1"
  },
```
```
npm run snowpack
```
Even though we don't use testing-library in production code we still need it available in browser tests. So we run it through snowpack as any other browser dependency.

Finally create a test in test/browser.test.js
```javascript
const {assert} = chai;
import {init} from "../src/app.js";
import {getAllByTestId, waitFor} from "../src/web_modules/@testing-library/dom.js";

const container = () => document.getElementById("app");

describe("App", () => {
    beforeEach(function () {
        container().innerHTML = "";
    });

    it("Load initial posts", async () => {
        init();
        await waitFor(() => {
            assert.strictEqual(getAllByTestId(container(), "item").length, 10);
        });
    });

});
```
The code cleans up the app container before every test. Inside a test, init a new instance of the app. Then wait for the 10 items to show up. ```waitFor``` runs until the assertion doesn't throw any errors or until it times out or mocha times out.
```getAllByTestId``` is a utility querying for ```data-testid="item```. 

Add the test data attribute to list item view fragment.
```javascript
const listItem = (post) => html`
  <li
    ...
    data-testid="item"
  >
    ...
  </li>
`;
```

Open test.html in your browser. The test should be green.

## Testing more advanced scenario in a browser

Add a test for input submission and waiting for SSE notification:
```javascript
import {findByTestId, findByText, fireEvent, getAllByTestId, waitFor} from "../src/web_modules/@testing-library/dom.js";

    it("Add a post", async () => {
        init();
        const input = await findByTestId(container(), "input");
        const newMessage = `new message ${new Date().toJSON()}`;
        input.value = newMessage;
        fireEvent.input(input);
        const button = await findByText(container(), "Add Post");
        button.click();
        await waitFor(() => {
            assert.strictEqual(getAllByTestId(container(), "item")[0].textContent, `@fixed ${newMessage}`);
        });
    })
```
Init the app again so that we don't have leftovers from the previous test. Find input by test data attribute. You will add it in a second. Fill in the text input with a new message and simulate "Add Post" button click. Wait until your message show up at the top of the message list.

Put a test data attribute in your only input:
```javascript
<input
          data-testid="input"
          ...
        />
```

## Making integration tests faster

You integration test executon time and correctness is heavily dependent on the API response time and availability. 
There are at least two options you have to make it faster and more reliable:
* record all HTTP traffic with a library like [PollyJS](https://netflix.github.io/pollyjs/#/). The first time you run your tests it intercepts all network calls and saves them to localStorage/REST API etc. Afterwards it can replay traffic much faster than the original API.
* inject fake implementation of all effects and subscriptions you want to replace. 

The trick is to move all effects and subscriptions out of the app.js file.
In production you can init your app with real dependencies
```javascript
import { init } from "./app.js";
import { Http } from "./web_modules/hyperapp-fx.js";
import { WebSocketListen } from "./web_modules/hyperapp-fx.js";
import {EventSourceListen} from "./lib/eventsource/EventSource.js";
import {WithGuid} from "./lib/guid/Guid.js";

init({Http, WebSocketListen, EventSourceListen, WithGuid});
```
And in your tests your provide fake implementation of those APIs. This is essentialy what some people call a dependency injection, which is a fancy name for passig arguments to functions. 

Since those techniques independent of a framework or even  a programming language we leave it to the reader to experiment with them.

## Preparing code for production

Before you ship your code to production you may need to:
* translate modern ES6+ code to ES5 so older browsers can understand it
* generate a single or a few JS bundles so we don't serve too many files

In this tutorial we use Parcel - a zero-configuration bundler.

```
npm i parcel@next -D
```

Add a script to build your production code:
```json
  "scripts": {
    "build": "parcel build src/index.html"
  }
```

Parcel will generate a ```dist``` directory with the production optimized index.html and JS bundle.

Verify your production distribution locally:
```
http-server dist
```

Here's a deployed version of our application: TODO

TODO: Hyperapp + application is smaller than most framework code. No matter how much code splitting you do in React.
TODO: remove htm

## Rendering view and state into a string

So far you've been using Hyperapp to translate views into DOM nodes. 

You can also translate Hyperapp view to HTML string with a help of a library called [hyperapp-render](https://github.com/kriasoft/hyperapp-render).

Create server.js in a root directory of your project:
```javascript
import render from "hyperapp-render";
import {state, view} from "./app.js";

const html = render.renderToString(view(state));

console.log(html);
```
Make sure your src/app.js exports the main view function and initial state. ```hyperapp-render``` should serialize your view with state into HTML string.

Run it in Node.js:
```node src/server.js```

## Rendering view and state from HTTP server

You've just seen how to turn your Hyperapp views and state into HTML. You can serve this HTML from your HTTP server.

Install a popular and minimal Node.js Web application server framework express:
```
npm i express
```

Expose your Hyperapp view as a resource with HTML representation:
```javascript
import render from "hyperapp-render";
import express from "express";
import {state, view} from "./src/app.js";

const app = express();

app.get("/", (req, res) => {
    const html = render.renderToString(view(state));
    res.send(html);
});
app.listen(3000, () => {
    console.log("Listening on 3000");
});
```

Run your server:
```
node src/server.js
```

Open ```http://localhost:3000```.

It should serve unstyled HTML with a form and empty list of posts.

## Fetching data on the server

In this section you'll add a list of posts to the server rendered HTML. 
When fetching data from 3rd party APIs server and client significantly differ.

Client fetches data in the background. In our client side app ```LoadLatestPosts``` effect starts when you open a browser and data is fetched in the bacground. When the data arrives Hyperapp re-renders a view with a newly fetched response.

Server has to wait for the data and only then render the response. So the wait before render behavior is a major difference. 

In this tutorial you won't be trying to create universal data fetching effects that can be shared between a client and a server. Instead we'll use axios and handle data fetching separately. 
```
npm i axios
```

```javascript
import render from "hyperapp-render";
import express from "express";
import {state, view, SetPosts} from "./src/app.js";
import axios from "axios";

const app = express();

app.get("/", async (req, res) => {
    const response = await axios.get("https://hyperapp-api.herokuapp.com/api/post");
    const posts = response.data;
    const stateWithPosts = SetPosts(state, posts);
    const html = render.renderToString(view(stateWithPosts));
    res.send(html);
});
app.listen(3000, () => {
    console.log("Listening on 3000");
});
```
Make sure ```SetPosts``` action is exposed. We can reuse the action on the server. 

Once you open your app in the browser a list of posts should be rendered.

As you've just seen Hyperapp can be used as a server-side template engine with a help of ```hyperapp-render```.

## Hydrating server side code on the client side

Hydration is a fancy name for taking control over server-side rendered content on the client side. For large applications server-side rendering with or without hydration always improves the time to visible content. However when it comes to time to interactivity hydration always performs worse than server-side rendering or client side-rendering alone. In other words with hydration you pay the rendering tax twice. Since Hyperapp is really small you may not notice the hydration penalty.

Move index.html into server.js and put it into a template string:
```javascript
const htmlTemplate = (content) => /*HTML*/ `
    <!DOCTYPE html>
    <html>
    <head>
        <link rel="stylesheet" href="https://andybrewer.github.io/mvp/mvp.css">
        <script type="module" src="init.js"></script>
    </head>
    <body>
    <main>
        <div id="app">${content}</div>
    </main>
    </body>
    </html>
`;
```
You'll be replacing ```content``` placeholder with dynamically rendered content.
Note: ````/*HTML*/``` comment is for prettier to auto-format the string as HTML just the same way it works with ```html`````.

Update server.js to serve static files in src directory. Put it after GET "/" handler so that index.html has lower precedence over our root resource. Also, replace the ```content``` placeholder with the actual content.
```javascript
app.get("/", async (req, res) => {
    ...
    res.send(htmlTemplate(content));
});
app.use(express.static("src"));
```

```
node server.js
```

## Passing server state to the client

Current version of your application fetches the post list twice. First on the server, second on the client in ```LoadLatestPosts``` effect. You can skip the second rendering. 

In app.js change the init property value:
```javascript
export const init = () =>
  app({
    init: window.initialState ? window.initialState : [state, LoadLatestPosts],
    ...
  });
```
Backend and frontend will use window.intialState to pass the state. If the state is provided we skip the initial empty state setting and post loading effect.

In server.js change htmlTemplate to accept not only content, but also state. Put the state into ```window.initialState``` where frontend code will find it.
```javascript
const htmlTemplate = (content, state) => /*HTML*/ `
    <!DOCTYPE html>
    <html>
    <head>
        <link rel="stylesheet" href="https://andybrewer.github.io/mvp/mvp.css">
        <script type="module" src="init.js"></script>
        <script>
            window.initialState = ${state};
        </script>
    </head>
    <body>
    <main>
        <div id="app">${content}</div>
    </main>
    </body>
    </html>
`;
```

Make sure to serialize the object into string
```javascript
import serialize from "serialize-javascript";

app.get("/", async (req, res) => {
    ...
    res.send(htmlTemplate(content, serialize(stateWithPosts)));
});
```

Test your app with JS enabled and disabled. Both versions should work. 

## Decorating view with layout and navigation

First write the ideal signature for the layout decorator. You're designing layout decorator API by using it before it's implemented.
```javascript
  app({
    view: layout(view),
  });
```
```layout``` should wrap the original view function and return a new function accepting state as a parameter. 

Create src/layout.js with a function matching the specification.
```javascript
export const layout = (content) => (state) => html``;
```

Fill in the gaps:
```javascript
import { html } from "./html.js";

const nav = html`
  <nav>
    <ul>
      <li><a href="/" class="href">Posts</a></li>
      <li><a href="/login" class="href">Login</a></li>
    </ul>
  </nav>
`;

export const layout = (content) => (state) => html`
  <div>
    <header>
      <h1>HyperPosts</h1>
      ${nav}
    </header>
    <main>
      ${content(state)}
    </main>
  </div>
`;
```
```layout``` provides a common header with navigation links. It invokes the ```content``` view function with the current state you get from the framework.

Import the layout and test your app:
```javascript
import { layout } from "./layout.js";
```

## Analyzing routing options

In the next section you'll start adding a new login page. With more than a single page our system requires some kind of routing.
There are two routing options:
* server-side
* client-side

With **server-side** routing you create one app per page. Each page:
* can evolve independently
* can use a different version of a framework
* can even use a different framework
* can use only HTML/CSS
To integrate pages together you use hypermedia: links and forms. To explore this topic go to Self-Contained Systems architecture [website](https://scs-architecture.org/).

With **client-side** routing you manage page transitions in JS and you need a client-side router. 
Most JS routers:
* hijack links and forms to prevent default browser behavior
* call ```history.pushState``` to update browser URL bar
* listen to ```popstate``` events to make the back button work

## Routing between HTML pages

To facilitate server side routing you'll create a second HTML page src/login.html.

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <title>HyperPosts</title>
    <link rel="stylesheet" href="https://andybrewer.github.io/mvp/mvp.css">
    <script type="module" src="login.js"></script>
</head>
<body>
<main>
    <div id="app"></div>
</main>
</body>
</html>
```
This is almost the same HTML you wrote for the main page. The only difference is a script it references (login.js).

Create src/login.js:
```javascript
import { app } from "./web_modules/hyperapp.js";
import { layout } from "./layout.js";
import { html } from "./html.js";
import {WriteToStorage} from "./web_modules/hyperapp-fx.js";

const state = {
  login: "",
};

const targetValue = (event) => event.target.value;

const ChangeLogin = (state, login) => [{...state, login}, WriteToStorage({key: "hyperposts", value: login})];

const view = (state) => html`
  <form method="get" action="/">
    <input onchange=${[ChangeLogin, targetValue]} value=${state.login} />
    <button>Login</button>
  </form>
`;

app({
  init: [state],
  view: layout(view),
  node: document.getElementById("app"),
});
```
This is another Hyperapp application. In this tutorial we use exactly the same Hyperapp version for the login page, but it doesn't have to be. Server-side routing allows for independent evolution of your pages. Each of them can upgrade its framework version independently of the others.

The code for the login page should be familiar to you by now. The only new thing is ```WriteToStorage``` effect from the ```hyperapp-fx``` library. As the user types the login we save it to the ```localStorage```. 

```WriteToStorage({key: "hyperposts", value: login})``` basically translates to ```localStorage.setItem("hyperposts", login)```.

When our user clicks the "Login" button a regular HTML form should take her to the main page. As you want to let the server handle navigation between the pages, don't hijack the form and let the browser do what it's good at - handling hypermedia.

## Exercise: reading from local storage

On the main posts page your task will be to read the login and set the current username in the state object. When the user submits a post use the username instead of "anonymous" username we used so far.

To make it easier here's a starter code for reading from storage:
```javascript
import {ReadFromStorage} from "./web_modules/hyperapp-fx.js";

const ReadLogin = ReadFromStorage({key: "hyperposts", action: ({value}) => ...})
```

<details>
    <summary id="reading_local_storage">Solution</summary>

```javascript
const SetUsername = (state, {value}) => value ? ({...state, username: value}) : state;
const ReadLogin = ReadFromStorage({key: "hyperposts", action: SetUsername})

export const init = () =>
  app({
    init: window.initialState ? window.initialState : [state, LoadLatestPosts, ReadLogin],
    ...
  });
```

</details>

## Preparing pages for client side routing

Server-side routing is easier than client-side routing. You let the browser handle links, forms and history when clicking back and forward buttons. However in certain situation you need to take over the browser job and handle routing/navigation in JS. It also means you need to keep a global state across the pages which makes state management more difficult. 


First, expose action for page initialization on each of your pages.

src/posts.js
```javascript
export const InitPage = (state) => [{...state, ...initState}, [LoadLatestPosts, ReadLogin]];
```
You're merging new page initState with the whole app state. If there's any field in the previous ```state``` and the ```initState``` the latter will overwrite the former. We could create a nested state for each page but it would complicate state updates.
```InitPage``` not only sets the initial state, but also fires all effects.

Do the same thing in src/login.js
```javascript
export const InitPage = (state) => [{...state, ...initialState}];
```
There's not effects to trigger on this page so only state merging happens. Make sure to remove the ```app()``` call on this page. We'll be moving to one centralized app setup.

## Setting up the main app

In src/app.js you'll setup the whole SPA.
```javascript
import {layout} from "./layout.js";
import {EventSourceListen} from "./lib/eventsource/EventSource.js";
import {SetPost} from "./posts.js";
import {view as postsView, InitPage as InitPostsPage} from "./posts.js";
import {view as loginView, InitPage as InitLoginPage} from "./login.js";
import {app} from "./web_modules/hyperapp.js";

const views = {
    "/": postsView,
    "/login": loginView
};
const routeInitActions = {
    "/": InitPostsPage,
    "/login": InitLoginPage
};

const view = state => {
    const routeView = views[state.location];
    return routeView ? routeView(state): "Loading...";
};

export const init = () =>
    app({
        init: {},
        view: layout(view),
        subscriptions: (state) => [
            state.liveUpdate &&
            EventSourceListen({
                action: SetPost,
                url: "https://hyperapp-api.herokuapp.com/api/event/post",
            })
        ],
        node: document.getElementById("app"),
    });
```
You start with mapping each URL path to a corresponding view and init action.
The main view function will look up the actual view based on the ```state.location``` property that you'll add next.

## Integrating with 3rd party libraries

In this section you'll integrate our app with a client-side router [page.js](https://github.com/visionmedia/page.js/). 
I use it to demo how Hyperapp can work with any library outside of its ecosystem. When wrapping 3rd party libraries you normally put them inside subscriptions and effects.

Add page.js to your web dependencies:
```json
  "webDependencies": {
    ...
    "page": "1.11.6",
    ...
  },
```

Looking at page.js documentation I came up with the following API calls you may need:
```
import page from "./web_modules/page.js";

page("/", fn); // register a route and call fn when a user navigated to the url
page("/login", fn);

page.start(); // start client-side routing
page.stop(); // stop client-side routing
```

Wrap this code into a subscription src/router.js:
```javascript
import page from "./web_modules/page.js";

const routeSubscription = (dispatch, data) => {
    page("/", () => {
    
    });
    page("/login", () => {
    
    });

    page.start();

    return () => {
        page.stop();
    };
};
```
Start the router when the subscription is created. Stop the router when subscription is unsubscribed from.

## Mapping 3rd party calls to Hyperapp actions

Fill in the page URL change handlers with some dispatch calls.
```javascript
import page from "./web_modules/page.js";

const SetLocation = (state, location) => ({location});

const routeSubscription = (dispatch, data) => {
    page("/", () => {
        dispatch(SetLocation, "/");
        dispatch(InitPostsPage);
    });
    page("/login", () => {
        dispatch(SetLocation, "/login");
        dispatch(InitLoginPage);
    });

    ...
};
```
When a user navigates to a URL you need to store the location for the correct view display. Our router has a private action ```SetLocation```. Then you need to initialise a current page. 
This code can be more generic. You can move the app spacific configuration to the caller. That's our next section.

## Driving router design from the outside

Start from a file where you'll use the router.
```javascript
import {RouteListen} from "./router.js";

const routeInitActions = {
    "/": InitPostsPage,
    "/login": InitLoginPage
};

export const init = () =>
    app({
        ...
        subscriptions: (state) => [
            ...
            RouteListen(routeInitActions)
        ]
    });
```
Ideally, we want to have ```RouteListen``` subscription with the route init actions config.

Implement the ideal API in router.js:
```javascript
const routeSubscription = (dispatch, data) => {
    Object.entries(data).map(([location, init]) => {
        page(location, (context) => {
            setTimeout(() => {
                dispatch(SetLocation, location);
                dispatch(init, context.params);
            });
        });
    });
    ...
};

export const RouteListen = data => [routeSubscription, data];
```
```RouteListen``` is parametrized with data. In the subscription definition iterate over the config object and register a page for each of them. You should also wrap the dispatch calls in setTimeout. Current Hyperapp implementation expects asynchronous calls to dispatch, otherwise it goes into the infinite loop. It may change in future version of the framework. 

Test the nevigation between our two pages. All anchor tags should be handles client-side. ```page.js``` hijack browser links so you don't need to create custom link tags. One are where you still need to create a custom action is the form submission. When you login and go to the posts page a full reload is performed. ```page.js``` doesn't handle forms, only links.

## Wrapping 3rd party library intro effects

You can wrap 3rd party libraries not only into subscription, but also into one-off effects and effectful actions.

Add Navigate action and effect to router.js:
```javascript
const navigateEffect = location => [(_, location) => {
    page(location);
}, location];
export const Navigate = location => (state) => [state, navigateEffect(location)];
```
```page(location)``` is a way of telling ```page.js``` to navigate to a given url. You wrapped the call inside the effect.

Use it in login.js:
```javascript
import {Navigate} from "./router.js";

export const view = (state) => html`
  <form method="get" action="/" onsubmit=${Navigate("/")}>
    ...
  </form>
`;
```

When you test the code it still doesn't work. Default browser submit event still fires. You need to prevnet it.

## Preventing browser events

Add a little helper library ```@hyperapp/events```
```json
  "webDependencies": {
    "@hyperapp/events": "0.0.4"
  }
```

Use it in login.js
```javascript
import {preventDefault} from "./web_modules/@hyperapp/events.js";

export const view = (state) => html`
  <form method="get" action="/" onsubmit=${preventDefault(Navigate("/"))}>
    ...
  </form>
`;
```
```preventDefault``` wraps any action and returns a new action. The new action calls ```event.preventDefault()``` and delegates everything else to the original action. 

With this change your client-side navigation should work.

## Summary

You started this tutorial with the elevator pitch:
```
For JS developers. 
Who need to build highly interactive Web interfaces  
Hyperapp is a functional view and state management framework
That allows for writing easy to understand, performant and testable code
Unlike other JS frameworks
Hyperapp fits in one 500 LOC file with zero dependencies and facilitates programming in a tiny subset of JS "the very best parts"
```
To summarize everything you've learned I'd like to expand on the elevator pitch and provide more details.
For me personally Hyperapp has 3 main benefits:
* teachability
* performance
* correctness

### Teacheability

* Tiny size: with some effort I can understand Hyperapp source code without being a core contributor. 
On the other hand, most other frameworks have tens/hundreds of thousands lines of code. I am at the mercy of their documentation and people
finding time to solve my problems. With Hyperapp I just look at the code.
* Tiny API surface area: only 5 core concepts to learn. State, views, actions, effects and subscriptions. The last 4 are just functions.  
* Small subset of JS: pure functions and object literals powered by the Elm-inspired functional architecture. 
No need for this, new, classes. No need to extend from framework components or annotate code with @Component annotations.
Those things are like dandruff for your application code. 
Also, no need for hooks and special linting rules. No need for nonstandard language extensions.
* ESM native: you can use Hyperapp directly in your browser without any extra tooling in development.
* Transferable skills: Hyperapp uses core DOM Events API. The onlick, oninput, event.target.value you learned 10 years ago are still relevant.

### Performance

* Fast load time: with one file and zero dependencies Hyperapp loads fast by default (only 2kB). It doesn't require sophisticated performance tricks to load faster.
* No compromise in runtime performance: https://krausest.github.io/js-framework-benchmark/current.html 

### Correctness

* Easy to unit test: everything is a pure function. Pass the input, verify the output. 
* Side effects at the edges: framework executes side effects, your own code is only pure functions.
* Centralized immutable state: easy to track state transitions from the old state to the new state. 


## Comparison to other technologies

You may be wondering how Hyperapp compares to other frontend solutions. Below you'll find my subjective comparison.

My choice of technology is filtered through those principles:
* Understand things from principles. 
* Understand one layer of abstraction below your normal abstraction.
* Compose solutions from simple tools solving narrow problems aka Unix Philosophy.
* Prioritize tools that give you fast feedback (fast build, fast subsecond tests etc.).  

If your principles are different probably you'll make different choice.

### Elm

Type-Driven Development in Elm makes for a very nice development experience. Compiler taking care of runtime exceptions is very helpful.
Refactoring is much easier than in the JS/TS world. The tradeoff is that Elm requires build tooling in development. You can't use a browser as your REPL.
Also some APIs that haven't been ported to Elm are harder to work with than in JS. 

### React

Because of the code size, I can't understand it at the source code level. It has many parts I don't care about (classes, lifecycle method etc.).

### Preact

Smaller than React. Can read the sources and understand them. Has many parts I don't care about. To emulate functional architecture requires extra libraries.

### Angular

Totally opposite approach to Hyperapp. Layers upon layers of abstraction. Too far away from the browser to my taste. Too OO centric 
and too much magic. Way too big to understand it at the source code level. 

### Vue

Uses language features I tend to avoid (new, classes, this). Too big to understand at the source code level. 

### Svelte

Nice idea with code shrinking at runtime. But it's not really JS, but a compiler from JS superset. If I have to pay the price
of another language with compiler enabling dead code elimination I'd go for Elm.



