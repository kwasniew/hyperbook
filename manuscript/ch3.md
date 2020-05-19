# Chapter 3: Development workflow

## Using Hyperapp from npm

Using modules directly from CDN is convenient for simple experiments. 
However, for the regular development you want to have local version of your dependencies. 
Why? Because sometimes CDNs:
* go down
* are slow to respond
* have security breaches
* go out of business

Additionally, local dependencies allow for offline work.

Create **package.json** in your root directory:
```json
{
  "dependencies": {
    "htm": "3.0.4",
    "hyperapp": "2.0.4"
  }
}
```
Use the same versions of dependencies as this book to avoid surprises.

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
But unfortunately browsers can't resolve those.

Since both ```hyperapp``` and ```htm``` are zero-dependency libraries you can load them using **node_modules** path:
```javascript
import {h, app} from "../node_modules/hyperapp/src/index.js";
import htm from "../node_modules/htm/dist/htm.mjs";
```

Start HTTP server from the root:
```http-server .```

Open ```http://127.0.0.1:8080/src``` and test your app.

It certainly works, but I had to inspect the contents of both libraries to provide correct paths.

## Integrating Hyperapp with Snowpack 

[Snowpack](https://www.snowpack.dev/) is a tool to translate selected ```node_modules``` into browser friendly bundles at dependency installation time.
It puts all dependencies as single file bundles in a predictable location called ```web_modules```. 
It also has experimental support for deep imports so all transitive dependencies are resolved as a single browser-friendly file. 
Deep imports work for libraries using ```require``` and [Node.js ESM modules](https://nodejs.org/api/esm.html). The latter means your dependency needs
to use ```.js``` extension in import statements.  
In essence Snowpack makes bundling JS optional at development time.

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
Snowpack is our development dependency. It provides ```snowpack install``` command that will run after ```npm i``` from the ```postinstall``` script. 
You tell snowpack to put the browser friendly bundles in ```src/web_modules```. Without ```--dest```  it would put everything in the root-level
```web_modules```. Since we serve the whole ```src``` directory everything needs to be inside.

Rewrite your imports to use ```web_modules```:
```
import {h, app} from "./web_modules/hyperapp.js";
import htm from "./web_modules/htm.js";
```
 
Run the installation command:
```npm i```

Snowpack will inspect your code and translate required modules from ```node_modules``` to ```web_modules```.

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
```format``` command willl format your JS files except from the ```web_modules``` (excluded explicitly) and ```node_modules``` (excluded by default).
With the ```--write``` option it will re-write the formatted files in place.


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

You can connect prettier to your IDE/text editor to format on save, but it's beyond the scope of this book.

You took a detour to learn about some tools that play nicely with Hyperapp:
* ```htm``` for HTML-like syntactic sugar
* ```snowpack``` for browser friendly dependencies
* ```prettier``` for consistent code formatting

In the next chapter we're back to your app.