# react-scripts

This package includes scripts and configuration used by [Create React App](https://github.com/facebookincubator/create-react-app).
Please refer to its documentation:

* [Getting Started](https://github.com/facebookincubator/create-react-app/blob/master/README.md#getting-started) – How to create a new app.
* [User Guide](https://github.com/facebookincubator/create-react-app/blob/master/packages/react-scripts/template/README.md) – How to develop apps bootstrapped with Create React App.

# Changes in this fork

I added the possibility of patching webpack configs. So you can switch on css-modules or add sass-loader if you want.

## How to use

1. Set version of `react-scripts` in your package.json to `https://github.com/ezhlobo/react-scripts.git#use-custom-config`.

2. Run `npm update react-scripts`.

2. Update scripts in package.json:
  ```json
"scripts": {
  "start": "WEBPACK_CONFIG_PATH=./config/webpack.config.dev.js react-scripts start",
  "build": "WEBPACK_CONFIG_PATH=./config/webpack.config.prod.js react-scripts build",
```

3. Create your config files in `./config`.
  Here's an example how to do patches:
  ```js
const generalConfig = require('react-scripts/config/webpack.config.dev');

// Switch on css-modules
generalConfig.module.loaders[1].loader = 'style!css?module!postcss';

module.exports = generalConfig;
  ```
  Take a look at default configs here: [ezhlobo/react-scripts/tree/use-custom-config/config](https://github.com/ezhlobo/react-scripts/tree/use-custom-config/config).

It's an advanced guide. Feel free to ask about it by sending an email to ezhlobo@gmail.com or by creating an issue.

# How to keep relevant version with original repository

This fork will be updated when original react-scripts will be changed. I'll do it manually for now so we can get a delay. Feel free to email me (ezhlobo@gmail.com) or create an issue. I'll respond in two days.

# How I use it in production

I want to provide an example how I use this fork for my projects.

1. Firstly I created `./config/patcher.js` to do patches for `dev` and `prod` configs by the same way.
  ```js
  class Patcher {
    static isCssLoader(handler) {
      return handler.test.toString().indexOf('\\.css') > -1;
    }

    constructor(config) {
      this.config = config;
      return this;
    }

    patchLoader(test, patch) {
      this.config.module.loaders.forEach(handler => {
        if (test(handler)) {
          patch(handler);
        }
      })

      return this;
    }

    patchCssLoader(patch) {
      return this.patchLoader(Patcher.isCssLoader, patch);
    }
  }

  module.exports = Patcher;
```

2. I patch `dev` config in `./config/webpack.config.dev.js`:
  ```js
  const Patcher = require('./patcher');
  const generalConfig = require('react-scripts/config/webpack.config.dev');

  module.exports = new Patcher(generalConfig)
    .patchCssLoader(handler => {
      handler.loader = 'style!css?module!postcss';
    })
    .config;
```

3. I patch `prod` config in `./config/webpack.config.prod.js`:
  ```js
  const Patcher = require('./patcher');
  const generalConfig = require('react-scripts/config/webpack.config.prod');
  const ExtractTextPlugin = require('extract-text-webpack-plugin');

  module.exports = new Patcher(generalConfig)
    .patchCssLoader(handler => {
      handler.loader = ExtractTextPlugin.extract('style', 'css?-autoprefixer&module!postcss');
    })
    .config;
```
