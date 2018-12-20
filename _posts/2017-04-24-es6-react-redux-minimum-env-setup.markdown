---
layout: post
title:  "es6 + react/redux minimum env setup"
date:   2017-04-24 15:00:00 +0800
categories: jekyll update
---

> updated 2018-01-20: don't need to compile ES6 to ES5 as major browsers already support ES6.

> updated 2018-07-23: webpack 4

> updated 2018-12-20: babel 7

This tutorial serves more as a renference than a detailed howto.

It's a minimum setup for:

  - [ES6 + ReactJS](#reactjs)
  - [ES6 + ReactJS + transform-class-properties](#es6-reactjs-transform-class-properties)
  - [ES6 + Redux](#redux)

## ReactJS

Let's say our project folder is called `react_demo`.

> Following commands are all supposed to be executed under `react-demo` folder!

#### Step 1 - create folder structure

```bash
~$ mkdir src
~$ touch src/App.jsx
~$ mkdir build
~$ touch webpack.config.js
~$ touch index.html
# by far, your react_demo folder structure should be:
# ├── build
# ├── src
# │   └── App.jsx
# ├── index.html
# └── webpack.config.js
```

#### Step 2 - create package.json

```shell
npm init --yes
```

#### Step 3 - install dependencies and tools

```shell
npm install --save-dev @babel/core babel-loader @babel/preset-react
# babel-core:           name tells it.
# babel-loader:         transpiling JavaScript files using Babel and Webpack 4.
# babel-preset-react:   strip flow types and transform JSX into createElement calls.
npm install --save react react-dom
# react and react-dom been seperated from version 0.14.
npm install --save-dev webpack webpack-cli
```

#### Step 4 - edit webpack.config.js

```js
module.exports = {
  entry: './src/App.jsx',
  output: {
    filename: 'bundle.js',
    path: __dirname + '/build'
  },
  module: {
    rules: [
      {
        test: /\.jsx$/,
        exclude: /node_modules/,
        use: {
          loader: 'babel-loader',
          options: {
            presets: ['@babel/react'],
          }
        }
      }
    ]
  }
}
// entry:     entry point tells webpack where to start.
// output:    tells webpack where to output bundled codes.
// test:      a regular expression that tests what kind of files to run through this loader
// options:   babel-loader options.
```

#### Step 5 - edit index.html

```html
<html>
  <body>
    <div id="main"></div>
    <script type="text/javascript" src="build/bundle.js" charset="utf-8"></script>
  </body>
</html>
```

#### Step 6 - edit src/App.jsx

```javascript
import React from 'react'
import ReactDOM from 'react-dom'

const App = () => {
  return <h1>React APP!</h1>
}

ReactDOM.render(<App />, document.getElementById("main"))
```

#### Final - bundle

```shell
./node_modules/.bin/webpack -d
```

#### Test

You would find a list of [one-line-servers](https://gist.github.com/willurd/5720255) implemented in different languages.

For me, I use

    ruby -run -ehttpd . -p8000

Open `http://127.0.0.1:8000/` and take a look!


## ES6 ReactJS transform-class-properties

#### Explain

With `transform-class-properties` plugin we could do `static in class` and `property initialize`. This ie.g:

```javascript
class App extends React.Component {
  // ...
  static propTypes = {
    disabled: PropTypes.bool
  }
  // ...
  state = {
    shouldShowBox: true
  }
  // ...
  toggleBox = () => {
    this.setState({
      shouldShowBox: !this.state.shouldShowBox
    })
  }
}
```

#### Adjusted Steps

for `Step 3`, we need to add:

```shell
npm install --save-dev babel-plugin-transform-class-properties
# babel-plugin-transform-class-properties   as explained class properties will be transformed.
```

for `Step 4`, we need to add:

```javascript
// ...
    loaders: [
      {
        test: /\.jsx?$/,
        loader: "babel-loader",
        query: {
          // +++++++++++++++++++++++++++++++++++++
          // add transform-class-properties plugin
          plugins: ['transform-class-properties'],
          // +++++++++++++++++++++++++++++++++++++
          presets: ['react'],
        }
      }
// ...
```

That's all!



## Redux

Let's say our project folder is called `redux_demo`.

> Following commands are all supposed to be executed under `redux-demo` folder!

#### Step 1 - create package.json

```shell
npm init --yes
```

#### Step 2 - install redux and webpack

```shell
npm install --save-dev webpack
npm install --save redux
```

#### Step 3 - create webpack.config.js

```javascript
var webpack = require('webpack')

module.exports = {
  entry: './voting.js',
  output: {
    filename: 'bundle.js'
  }
}
// babel not used at all!
```

#### Step 4 - create voting.js

```javascript
// `import` probably will be shipped with Node 9.0.
const createStore = require('redux').createStore

const counter = (state = 0, action) => {
  switch (action.type) {
    case 'INCREMENT':
      return state + 1
    case 'DESCREMENT':
      return state - 1
    default:
      return state
  }
}

const store = createStore(counter)
// ---------------------------
// output counter when action been dispatched.
const render = () => {
  console.log('current state is: ' + store.getState())
}
store.subscribe(render)
// ---------------------------
store.dispatch({type: 'INCREMENT'})
```

#### Final - bundle

```shell
./node_modules/.bin/webpack -d
```

#### Test

```shell
node bundle.js     # => current state is: 1
```