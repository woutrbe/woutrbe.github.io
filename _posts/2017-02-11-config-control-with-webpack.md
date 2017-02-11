---
layout: post
title: Configuration control with Webpack
comments: true
---

Most developers who use node are aware of [node-config](https://github.com/lorenwest/node-config), it allows you to define different configurations for different environments.
That means you could have a `development.json` and a `production.json` configuration, `node-config` will pick the correct configration based on the environment.

A typical setup would look like this:

```
├──config
|   ├──default.json           * Default parameters
|   ├──development.json       * Development parameters
|   ├──staging.json           * Staging parameters
|   ├──production.json        * Production parameters
```

## Flexible front-end configurations

While working on a front-end app, you often want to have different configurations as well, your api endpoint will usually be different during development, staging and production.
There might be some other parameters that are changed during your production builds.

[Webpack](https://webpack.github.io) has a configuration option called `resolve.alias`, this allows you to replace modules with other modules or paths.
With that in mind, we can simply do:

```javascript
module.exports = {
	resolve: {
		alias: {
			// For development
			'config': path.resolve(config.src, 'config', 'development.js')
			// For production
			'config': path.resolve(config.src, 'config', 'production.js')
		}
	}
}
```

In any JavaScript file, we can get our config as follows:

```javascript
import config from 'config';
console.log(config);
```