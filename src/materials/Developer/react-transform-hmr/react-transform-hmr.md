---
path: '/materials/react-transform-hmr'
type: 'GitHub'
img: './screenshot.png'
material:
  title: 'react-transform-hmr'
  url: 'https://github.com/gaearon/react-transform-hmr'
  github_url: 'https://github.com/gaearon/react-transform-hmr'
  subscribers_count: '13'
  stargazers_count: '780'
  tags: ['']
  subtitle: 'A React Transform that enables hot reloading React classes using Hot Module Replacement API'
  clone_url: 'https://github.com/gaearon/react-transform-hmr.git'
  ssh_url: 'git@github.com:gaearon/react-transform-hmr.git'
  pushed_at: '2018-03-21T10:47:25Z'
  updated_at: '2019-02-02T09:04:29Z'
  author:
    name: 'gaearon'
    avatar: 'https://avatars0.githubusercontent.com/u/810438?v=4'
    github_url: 'https://github.com/gaearon'
  latestRelease:
    tag_name: 'v1.0.4'
    name: 'v1.0.4'
    url: 'https://github.com/gaearon/react-transform-hmr/releases/tag/v1.0.4'
    created_at: '2016-03-06T18:03:36Z'
---
>## This Project Is Deprecated

>React Hot Loader 3 is [on the horizon](https://github.com/gaearon/react-hot-loader/pull/240), and you can try it today ([boilerplate branch](https://github.com/gaearon/react-hot-boilerplate/pull/61), [upgrade example](https://github.com/gaearon/redux-devtools/commit/64f58b7010a1b2a71ad16716eb37ac1031f93915)). It fixes some [long-standing issues](https://twitter.com/dan_abramov/status/722040946075045888) with both React Hot Loader and React Transform, and is intended as a replacement for both. The docs are not there yet, but they will be added before the final release. For now, [this commit](https://github.com/gaearon/redux-devtools/commit/64f58b7010a1b2a71ad16716eb37ac1031f93915) is a good reference.


# react-transform-hmr

[![react-transform channel on discord](https://img.shields.io/badge/discord-react--transform%40reactiflux-61DAFB.svg?style=flat-square)](http://www.reactiflux.com)


A [React Transform](https://github.com/gaearon/babel-plugin-react-transform) that enables hot reloading React classes using Hot Module Replacement API. Hot module replacement is [supported natively by Webpack](http://webpack.github.io/docs/hot-module-replacement-with-webpack.html) and available in Browserify with [browserify-hmr](https://github.com/AgentME/browserify-hmr).

## 🚧🚧🚧🚧🚧

This is **highly experimental tech**. If you’re enthusiastic about hot reloading, by all means, give it a try, but don’t bet your project on it. Either of the technologies it relies upon may change drastically or get deprecated any day. You’ve been warned 😉 .

**This technology exists to prototype next-generation React developer experience**. Please don’t use it blindly if you don’t know the underlying technologies well. Otherwise you are likely to get disillusioned with JavaScript tooling.

**No effort went into making this user-friendly yet. The goal is to eventually kill this technology** in favor of less hacky technologies baked into React. These projects are not long term.

## Installation

First, install the [Babel plugin](https://github.com/gaearon/babel-plugin-react-transform):

```
npm install --save-dev babel-plugin-react-transform
```

Then, install the transform:

```
npm install --save-dev react-transform-hmr
```

### React

Edit your `.babelrc` to include a plugin configuration for `react-transform`. It contains array of the transforms you want to use:

```js
{
  'presets': ['es2015', 'stage-0'],
  'env': {
    // only enable it when process.env.NODE_ENV is 'development' or undefined
    'development': {
      'plugins': [['react-transform', {
        'transforms': [{
          'transform': 'react-transform-hmr',
          // if you use React Native, pass 'react-native' instead:
          'imports': ['react'],
          // this is important for Webpack HMR:
          'locals': ['module']
        }]
        // note: you can put more transforms into array
        // this is just one of them!
      }]]
    }
  }
}
```

Make sure you process files with `babel-loader`, and that you *don’t* use React Hot Loader (it’s not needed with this transform).

**It is up to you to ensure that the transform is not enabled when you compile the app in production mode.** The easiest way to do this is to put React Transform configuration inside `env.development` in `.babelrc` and ensure you’re calling `babel` with `NODE_ENV=production`. See [babelrc documentation](https://babeljs.io/docs/usage/babelrc/#env-option) for more details about using `env` option.

**Warning!** This doesn't currently work for stateless functional components that were introduced in [React 0.14](https://facebook.github.io/react/blog/2015/10/07/react-v0.14.html#stateless-functional-components)!

### React Native

This transform enables hot reloading when used together with [React Native Webpack Server](https://github.com/mjohnston/react-native-webpack-server). **However note that you should not use `.babelrc` to configure it with React Native.** Otherwise you’ll get [`Uncaught SyntaxError: Unexpected reserved word` in `ActivityIndicatorIOS.ios.js`](https://github.com/mjohnston/react-native-webpack-server/issues/57#issuecomment-141487449).

There are two problems why `.babelrc` doesn’t work well in React Native:

* Changes in it [aren’t picked up by packager’s aggressive caching](https://github.com/mjohnston/react-native-webpack-server/issues/63).
* Another unknown problem causes `import` generated by `babel-plugin-react-transform` to not be compiled into a `require` call.

Until we have better `.babelrc` support in React Native, **you should configure React Transform together with `babel-loader`**:

```js
var fs = require('fs');
var path = require('path');
var webpack = require('webpack');

var config = {
  debug: true,

  devtool: 'source-map',

  entry: {
    'index.ios': ['./src/main.js'],
  },

  output: {
    path: path.resolve(__dirname, 'build'),
    filename: '[name].js',
  },

  module: {
    loaders: [{
      test: /\.js$/,
      exclude: /node_modules/,
      loader: 'babel',
      query: {
        stage: 0,
        plugins: []
      }
    }]
  },

  plugins: []
};

// Hot mode
if (process.env.HOT) {
  config.devtool = 'eval';
  config.entry['index.ios'].unshift('react-native-webpack-server/hot/entry');
  config.entry['index.ios'].unshift('webpack/hot/only-dev-server');
  config.entry['index.ios'].unshift('webpack-dev-server/client?http://localhost:8082');
  config.output.publicPath = 'http://localhost:8082/';
  config.plugins.unshift(new webpack.HotModuleReplacementPlugin());

  // Note: enabling React Transform and React Transform HMR:
  config.module.loaders[0].query.plugins.push([
    'react-transform', {
      transforms: [{
        transform : 'react-transform-hmr',
        imports   : ['react'],
        locals    : ['module']
      }]
    }
  ]);
}

if (process.env.NODE_ENV === 'production') {
  config.plugins.push(new webpack.optimize.OccurrenceOrderPlugin());
  config.plugins.push(new webpack.optimize.UglifyJsPlugin());
}

module.exports = config;
```

See [React Native Webpack Server examples](https://github.com/mjohnston/react-native-webpack-server/tree/master/Examples/) for details.


## License

MIT