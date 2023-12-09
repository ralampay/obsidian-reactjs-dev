This explains the different steps involved in getting [Bootstrap CSS Framework](https://github.com/twbs/bootstrap) integrated with your ReactJS project.

1. Install the necessary packages

```bash
npm install bootstrap
```

2. In `src/styles/index.scss`, import bootstrap's scss version:

```js
@import "../../node_modules/bootstrap/scss/bootstrap";
```

This should do for a vanilla bootstrap installation. ESBuild will compile everything and include bootstrap as part of its css bundle as stored in `./assets/styles/index.css`. This stylesheet is already linked and referenced in `public/index.html`. After refreshing your app, you should be able to notice that the Bootstrap default styles are now currently being followed.