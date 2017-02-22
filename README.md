# angular2-template-loader
Chain-to loader for webpack that inlines all html and style's in angular2 components.

[![Build Status](https://travis-ci.org/TheLarkInn/angular2-template-loader.svg?branch=master)](https://travis-ci.org/TheLarkInn/angular2-template-loader)
[![Coverage](https://codecov.io/gh/TheLarkInn/angular2-template-loader/branch/master/graph/badge.svg)](https://codecov.io/gh/TheLarkInn/angular2-template-loader)
[![Taylor Swift](https://img.shields.io/badge/secured%20by-taylor%20swift-brightgreen.svg)](https://twitter.com/SwiftOnSecurity)

### Quick Links
- [Installation](#installation)
- [Requirements](#requirements)
- [Example usage](#example-usage)
- [How does it work](#how-does-it-work)
- [Common Issues](#common-issues)

### Installation
Install the webpack loader from [npm](https://www.npmjs.com/package/angular2-template-loader).
- `npm install angular2-template-loader --save-dev`

Chain the `angular2-template-loader` to your currently used typescript loader.

```js
loaders: ['awesome-typescript-loader', 'angular2-template-loader'],
```

### Requirements
To be able to use the template loader you must have a loader registered, which can handle `.html` and `.css` files.
> The most recommended loader is [`raw-loader`](https://github.com/webpack/raw-loader)

This loader allows you to decouple templates from the component file and maintain AoT compilation. This is particularly useful  when building complex components that have large templates.

### Example Usage

#### Webpack
Here is an example markup of the `webpack.config.js`, which chains the `angular2-template-loader` to the `tsloader`

```js
module: {
  loaders: [
    {
      test: /\.ts$/,
      loaders: ['awesome-typescript-loader', 'angular2-template-loader?keepUrl=true'],
      exclude: [/\.(spec|e2e)\.ts$/]
    },
    /* Embed files. */
    { 
      test: /\.(html|css)$/, 
      loader: 'raw-loader',
      exclude: /\.async\.(html|css)$/
    },
    /* Async loading. */
    {
      test: /\.async\.(html|css)$/, 
      loaders: ['file?name=[name].[hash].[ext]', 'extract']
    }
  ]
}
```

#### Before
```js
@Component({
  selector: 'awesome-button',
  template: 'button.template.html',
  styles: ['button.style.css']
})
export class AwesomeButtonComponent { }
```

#### After (before it is bundled into your webpack'd application)
```js
@Component({
  selector: 'awesome-button',
  template: require('./button.template.html'),
  styles: [require('./button.style.css')]
})
export class AwesomeButtonComponent { }
```

### How does it work
The `angular2-template-loader` searches for `templateUrl` and `styleUrls` declarations inside of the Angular 2 Component metadata and replaces the paths with the corresponding `require` statement.
If `keepUrl=true` is added to the loader's query string, `templateUrl` and `styleUrls` will not be replaced by `template` and `style` respectively so you can use a loader like `file-loader`.

The generated `require` statements will be handled by the given loader for `.html` and `.js` files.

### Common Issues
In some cases the webpack compilation will fail due to unknown `require` statements in the source.<br/>
This is caused by the [way the template loader works](#how-does-it-work). 

> The Typescript transpiler doesn't have any typings for the `require` method, which was generated by the loader.

We recommend the installation of type defintions, which contain a declaration of the `require` method.
- [@types/node](https://www.npmjs.com/package/@types/node) | [dt~node](https://github.com/DefinitelyTyped/DefinitelyTyped/blob/master/node/node.d.ts)
- [@types/requirejs](https://www.npmjs.com/package/@types/requirejs) | [dt~requirejs](https://github.com/DefinitelyTyped/DefinitelyTyped/tree/master/requirejs)
