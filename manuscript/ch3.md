# Chapter 3: Development workflow

## Using Hyperapp from npm

Using modules directly from CDN is convenient for simple experiments. 
However, for the regular development, you want to have a local version of your dependencies. 
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
    "hyperapp": "2.0.8",
    "hyperlit": "0.3.5"
  }
}
```
Use the same versions of dependencies as this book to avoid surprises.

Install dependencies:
```
npm i
```
Look inside of **node_modules** directory. You'll find no transitive dependencies. Both `hyperlit` and `hyperapp` bring no extra guests to the party.

You can't reference npm dependencies from **App.js** with just a name:
```js
import {app} from "hyperapp";
import html from "hyperlit";
```
Because browsers can't resolve those.

Since both `hyperapp` and `hyperlit` are zero-dependency libraries you can load them using **node_modules** paths.
You have to look for the main JavaScript files. If you've used recommended versions then you can use:
```js
import {app} from "../node_modules/hyperapp/index.js";
import html from "../node_modules/hyperlit/dist.js";
```

Start HTTP server from the root:
`http-server .`

Open http://localhost:8080/src and test your app.

It certainly works, but you had to inspect the contents of both libraries to provide correct paths. We can automate 
this process.

## Integrating Hyperapp with Snowpack 

[Snowpack](https://www.snowpack.dev/) is a tool to translate selected `node_modules` into browser friendly bundles at dependency installation time.
It puts all dependencies as single file bundles in a predictable location called `web_modules`. 
It also has experimental support for deep imports so all transitive dependencies are resolved as single browser-friendly files. 
Deep imports work for libraries using `require` and [Node.js ESM modules](https://nodejs.org/api/esm.html). The latter means your dependency needs
to use `.js` extension in the import statements.  
In essence Snowpack makes bundling JS optional at development time.

Update **package.json** with this `snowpack` setup:
```json
{
  "scripts": {
    "snowpack": "snowpack install --dest=src/web_modules",
    "postinstall": "npm run snowpack"
  },
  "dependencies": {
    "hyperapp": "2.0.8",
    "hyperlit": "0.3.5"
  },
  "devDependencies": {
    "snowpack": "2.6.4"
  }
}
```
Snowpack is our development dependency. It provides `snowpack install` command that will run after ```npm i``` from the ```postinstall``` script. 
You tell Snowpack to put the browser-friendly bundles in `src/web_modules`, because we want to HTTP serve the whole `src` directory. Without `--dest`  it would put everything in the root-level `web_modules`.

Rewrite your imports to use `web_modules`:
```
import {app} from "./web_modules/hyperapp.js";
import html from "./web_modules/hyperlit.js";
```
 
Run the installation command:
```npm i```

Snowpack will inspect your code and translate required modules from `node_modules` to `web_modules`.

If you use git then add **src/web_modules** to **.gitignore**.

## Formatting code with prettier 

[Prettier](https://prettier.io/) is an opinionated code formatter saving your code review time for things that really matter. 
The days of spaces vs tabs wars are over.

Add `format` command and `prettier` dev dependency to **package.json**:
```json
{
  "scripts": {
    "snowpack": "snowpack install --dest=src/web_modules",
    "postinstall": "npm run snowpack",
    "format": "prettier --write '**/!(web_modules)/*.js'"
  },
  "dependencies": {
    "hyperapp": "2.0.8",
    "hyperlit": "0.3.5"
  },
  "devDependencies": {
    "prettier": "2.0.5",
    "snowpack": "2.6.4"
  }
}
```
`format` command will format your JS files except from the `web_modules` (excluded explicitly in the format script) and `node_modules` (excluded by default).
With the `--write` option `prettier` will re-write the formatted files in place.

Install prettier by running:
```npm i```

To test if `prettier` is working paste this malformed code into **App.js**:
```js
import { app } from "./web_modules/hyperapp.js";
import html from "./web_modules/hyperlit.js";

const state = { text: "Welcome to Hyperapp!" };

app({
  init: state,
  view: (state) => html`
<div>
    <h4>HyperPosts</h4>
    <ul>
      <li>
        <strong>@js_developers</strong>
        <span> Modern JS frameworks are too complicated</span>
      </li>
      <li>
        <strong>@jorgebucaran</strong>
        <span> There, I fixed it for you!</span>
      </li>
    </ul>
</div>
`,
  node: document.getElementById("app"),
});
```
Notice that the opening `div` is not aligned correctly.

Now run:
`
npm run format
`
The `view` code should get nicely aligned.

You can connect `prettier` to your IDE/text editor to format on save, but it's beyond the scope of this book.
Go to the [prettier website](https://prettier.io/) if you want to find out the configuration details.

You took a detour to learn about some tools that play nicely with Hyperapp:
* `hyperlit` for HTML-like syntactic sugar
* `snowpack` for browser friendly dependencies
* `prettier` for consistent code formatting

In the next chapter, we're back to your app.