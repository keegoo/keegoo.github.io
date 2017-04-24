---
layout: post
title:  "es6 + react/redux minimute env setup"
date:   2017-04-24 15:00:00 +0800
categories: jekyll update
---

This tutorial serves more as a renference than a detailed howto.

It's a minimum setup for:

  - [ES6 + ReactJS](#reactjs)
  - [ES6 + Redux](#redux)

## ReactJS

Let's say our project folder is called `react_demo`. 

> Following commands are all supposed to be executed under `react-demo` folder !

#### Step 1 - create folder structure

```bash
~$ mkdir src
~$ touch src/App.jsx
~$ mkdir build
~$ touch webpack.config.js
~$ touch index.html
# your react_demo folder structure should be:
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
npm install --save-dev babel-core babel-preset-es2015 babel-loader babel-preset-react
# babel-core:           name tells it.
# babel-preset-es2015:  compiles ES2015 to ES5.
# babel-loader:         transpiling JavaScript files using Babel and webpack.
# babel-preset-react:   strip flow types and transform JSX into createElement calls.
npm install --save-dev react react-dom
# react and react-dom been seperated from version 0.14.
npm install --save-dev webpack
```

#### Step 4 - edit webpack.config.js

```js
var webpack = require('webpack');
 
module.exports = {
  entry: './src/App.jsx',
  output: {
    filename: 'bundle.js',
    path: __dirname + '/build'
  },
  module: {
    loaders: [
      {
        test: /\.jsx?$/,
        loader: "babel-loader",
        query: {
          presets: ['es2015', 'react'],
        }
      }
    ]
  }
};
// entry:     entry point tells webpack where to start.
// output:    tells webpack where to output bundled codes.
// test:      a regular expression that tests what kind of files to run through this loader
// query:     loader options. Here 'es2015' and 'react' are two presets for babel-loader.
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

class App extends React.Component {
  render(){
    return(
      <h1>React APP!</h1>
    )
  }
}

ReactDOM.render(<App />, document.getElementById("main"))
```

#### Final - bundle

```shell
~$ ./node_modules/.bin/webpack -d
```

#### Now open index.html and take a look!

## Redux