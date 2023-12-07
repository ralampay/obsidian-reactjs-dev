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