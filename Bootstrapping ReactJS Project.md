Create a ReactJS webapp project from scratch using `esbuild` as compiler.
## Project Setup

In a project directory, establish an empty `npm` project:

```bash
npm init
```

Setup the top level directories:
* `src`: Contains all our js and css code
* `public`: Contains `index.html` as our single page container

### `src` subdirectories
* `js`: Javascript source code
* `styles`: CSS source code

We then need to install the relevant packages:

```bash
npm install \
	autoprefixer \
	axios \
	bootstrap \
	bootstrap-icons \
	dotenv \
	esbuild \
	esbuild-envfile-plugin \
	esbuild-sass-plugin \
	jwt-decode \
	postcss \
	react \
	react-bootstrap \
	react-dom \
	react-router-dom \
	sass
```
## Build Script

We'll create a `build.js` file with the following contents:

### Import Libraries

```js
import esbuild from 'esbuild';
import {sassPlugin} from 'esbuild-sass-plugin';
import postcss from 'postcss';
import autoprefixer from 'autoprefixer';
import esbuildEnvfilePlugin from 'esbuild-envfile-plugin';
```

### Build Parameters

```js
const port = 8000;
const args = process.argv.slice(2);
const watch = args.includes("--watch");
const dev = args.includes("--dev");
```

### Plugin for Compiling on Every Code Change

```js
const watchPlugin = {
	name: 'watch-plugin',
	setup(build) {
		build.onStart(() => {
			console.log(`Build starting: ${new Date(Date.now()).toLocaleString()}`);
		});

		build.onEnd(() => {
			if (result.errors.length > 0) {
				console.log(`Build finished with errors ${new Date(Date.now()).toLocaleString()}`);
				console.log(errors);
			} else {
				console.log(`Build finished successfully: ${new Date(Date.now()).toLocaleString()}`);
			}
		})
	}
}
```

### Common Settings

```js
let commonSettings = {        
  logLevel: 'debug',           
  metafile: true,            
  entryPoints: [
    'src/styles/index.scss',          
    'src/index.js'                        
  ],  
  outdir: 'public/assets',             
  bundle: true,                 
  loader: {           
    '.js': 'jsx',      
    ".png": "dataurl",     
    ".woff": "dataurl",    
    ".woff2": "dataurl",      
    ".eot": "dataurl",      
    ".ttf": "dataurl",      
    ".svg": "dataurl",                        
  },
  plugins: [   
    esbuildEnvfilePlugin,            
    sassPlugin({
      async transform(source) {
        const { css } = await postcss([autoprefixer]).process(            
          source                  
        );         
        return css;                    
      },                     
    }),             
    watchPlugin
  ],
}
```

### Build Process

```js
let debugSettings = {}
let productionSettings = {}

if(watch ||  dev) {
  // override settings here for debugSettings
  debugSettings = {
    ...commonSettings,
    logLevel: "debug",
    sourcemap: "linked"
  }

  let debugMode = await esbuild.context(debugSettings)
  console.log("Watching for changes...");

  // watch and dev
  if (watch) {
    console.log("Watching for changes...");
    debugMode.watch();
  }

  if(dev) {
    console.log("Debug Mode with" , debugSettings);
    debugMode.serve({
      servedir: 'public',
      port: port
    });
  }
} else {
  productionSettings = {
    ...commonSettings,
    // add settings for production.
  }
  console.log("Building with" , productionSettings);
  esbuild.build(productionSettings).catch((err) => {
    console.error(err);
    process.exit(1);
  });
  console.log("Deployment build completed.");
}
```

## Configuring `package.json`

We need to make the `type` of this application a `module`. Make sure we have the following attributes in `package.json`:

```js
"type": "module"
```

We can then define a `scripts` section that will call our `build.js` for dev or production environments:

```js
"scripts": {
	"build": "node build.js",
	"start": "node build.js --watch --dev"
}
```

Where:
* `build`: Builds the app for production
* `start`: Runs the build process with watch plugin under dev and runs a development server

## Setup Single Page Container

In the file `src/index.html`, we'd have the following:

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <title>
        Envelop
    </title>
    <script src="./assets/index.js" async defer></script>
    <link rel="stylesheet" href="./assets/styles/index.css">
  </head>
  <body id="bootstrap-overrides">
    <noscript>You need to enable JavaScript to run this app.</noscript>
    <div id="root"></div>
  </body>
</html>
```

This assumes the following:
* Main js bundle is found in`./assets/index.js`
* Main css bundle is found in `./assets/styles/index.css`
* Our main reactjs container is in `div` with id `root`
## Setup the `index.js` Entry Point

The entry point of our webapp will be in `src/index.js` when ESBuild compiles our application and stores the product in `public/assets/index.js`. This can be configured in our `build.js` file under `entryPoints` (which also contains the entry point for css) and `outdir` under `commonSettings`. The entry point `index.js` will create our React Root element  in a div with `id="root"` (see `src/index.html`) and render our top level component called `App` as defined in the file `src/js/App.js`.

### `src/js/App.js`

```js
import React from "react";

export default App = () => {
	return (
		<div>
			My App
		</div>
	);
}
```

### `src/index.js`

```js
import React from "react";
import ReactDOM from "react-dom/client";
import App from "./js/App";

const container = document.getElementById("root");
const root = ReactDOM.createRoot(container);

root.render(
	<App/>
);
```

Before compiling, since we specified that part of the entry point is our stylesheet file found in `src/styles/index.scss`, we have to make sure that the file is present (even if it has no contents).

## Testing things Out

To compile, run and watch for changes, run the following command:

```bash
npm run start
```

To compile for production run the following command:

```bash
npm run build
```