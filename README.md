# @vxna/optimize-three-webpack-plugin

[![Build Status](https://travis-ci.com/vxna/optimize-three-webpack-plugin.svg)](https://travis-ci.com/vxna/optimize-three-webpack-plugin) [![npm](https://img.shields.io/npm/v/@vxna/optimize-three-webpack-plugin.svg)](https://www.npmjs.com/package/@vxna/optimize-three-webpack-plugin)

A compat layer that enables tree shaking and human-readable imports.

## Warning

1. `webpack@>=4.0.0` and `three@>=0.112.0` are hard requirements.

2. `three@0.109.0` [introduced](https://github.com/mrdoob/three.js/pull/17276) ES6 classes in core. If you have to [support older browsers](https://caniuse.com/#feat=es6-class), you must [transpile it](#older-browsers).

3. I am not sure if it works with TypeScript and I have no resources to support it on my own.

4. Examples ESM conversion seems finished, so this package is in the maintenance mode.

## Usage

Default webpack configuration:

```js
const OptimizeThreePlugin = require('@vxna/optimize-three-webpack-plugin')

module.exports = {
  plugins: [new OptimizeThreePlugin()]
}
```

Your code:

```js
// core imports
import { WebGLRenderer } from 'three'
// becomes now
import { WebGLRenderer } from '@three/core'

// examples imports
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader'
// becomes now
import { GLTFLoader } from '@three/loaders/GLTFLoader'
```

## Custom module aliases

In the past, one of possible ways to tree shake `three` was to use `"sideEffects": false` flag in `webpack` and alias `three` to `src/Three.js` instead of default `build/three.module.js`. Since `three@0.103.0`, ESM [support for examples](https://threejs.org/docs/#manual/en/introduction/Import-via-modules) landed in `three` package. This change allows us to import loaders and other things from the examples folder with ease.

The problem was that tree shaking method we've used isn't compatible with ESM examples. I am not sure if using custom module aliases is actually the best solution for everyone, but at least it works for me and gives desired results.

At the time of writing all available ESM examples are [supported](https://github.com/vxna/optimize-three-webpack-plugin/blob/master/src/aliases.js).

## Rationale

Using basic https://threejs.org/examples/#webgl_geometry_cube example, results are:

Before:

```
> ls build -ghs
total 569K
500K -rw-r--r-- 1 None 497K Dec 15 09:13 0.ceabf32b276fff4b1659.js
 68K -rw-r--r-- 1 None  67K Dec 15 09:13 crate.da499b8537.gif
1.0K -rw-r--r-- 1 None  295 Dec 15 09:13 index.html

> gzip-size build\0.ceabf32b276fff4b1659.js
126 kB
```

After:

```
> ls build -ghs
total 381K
312K -rw-r--r-- 1 None 312K Dec 15 09:15 0.8e10d7ecdf3c9cb6f57f.js
 68K -rw-r--r-- 1 None  67K Dec 15 09:15 crate.da499b8537.gif
1.0K -rw-r--r-- 1 None  295 Dec 15 09:15 index.html

> gzip-size build\0.8e10d7ecdf3c9cb6f57f.js
78.7 kB
```

## Older browsers

Assuming that you're using [Babel](https://github.com/babel/babel-loader), this is possible configuration to make `three@>=0.109.0` work with older browsers:

```js
const OptimizeThreePlugin = require('@vxna/optimize-three-webpack-plugin')

module.exports = {
  module: {
    rules: [
      {
        include: /three/,
        loader: 'babel-loader',
        options: {
          presets: ['@babel/preset-env']
        }
      }
    ]
  },
  plugins: [new OptimizeThreePlugin()]
}
```

## License

[MIT](./LICENSE)
