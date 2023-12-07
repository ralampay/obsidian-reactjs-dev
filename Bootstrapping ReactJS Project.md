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

## Build Script

We'll create a `build.js` file with the following contents:

### Important Libraries

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