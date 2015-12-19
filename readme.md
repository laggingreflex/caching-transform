# caching-transform [![Build Status](https://travis-ci.org/jamestalmage/caching-transform.svg?branch=master)](https://travis-ci.org/jamestalmage/caching-transform) [![Coverage Status](https://coveralls.io/repos/jamestalmage/caching-transform/badge.svg?branch=master&service=github)](https://coveralls.io/github/jamestalmage/caching-transform?branch=master)

> Wraps a transform and provides caching.

Caching transform results can greatly improve performance. `nyc` saw [dramatic performance increases](https://github.com/bcoe/nyc/pull/101#issuecomment-165716069) when we implemented caching. 


## Install

```
$ npm install --save caching-transform
```


## Usage

```js
const cachingTransform = require('caching-transform');

cachingTransform({
  cacheDir: '/path/to/cache/directory',
  salt: 'hash-salt',
  transform: (input, additionalData, hash) => {
    // ...
    return transformedResult;
  }
});
```



## API

### cachingTransform(options)

Returns a transform callback that takes two arguments:

 - `input` a string to be transformed
 - `additionalData` an arbitrary data object.

Both arguments are passed to the wrapped transform. Results are cached in the cache directory using an `md5` hash of `input` and an optional `salt` value. If a cache entry already exist for `input`, the wrapped transform function will never be called.

#### options
                 
##### salt

Type: `string`
Default: `empty string`

A string that uniquely identifies your transform, a typical salt value might be the concatenation of the module name of your transform and its version':

```js
  const pkg = require('my-transform/package.json');
  const salt = pkg.name + ':' + pkg.version;
```

Including the package version in the salt ensures existing cache entries will be automatically invalidated when you bump the version of your transform. If your transform relies on additional dependencies, and the transform output might change as those dependencies update, then your salt should incorporate the versions of those dependencies as well.

##### transform

Type: `Function(input: string, additionalData: *, hash: string): string`  

 - `input`: The string to be transformed. passed through from the wrapper.
 - `additionalData`: An arbitrary data object passed through from the wrapper. A typical value might be a string filename.
 - `hash`: The salted hash of `input`. Useful if you intend to create additional cache entries beyond the transform result (i.e. `nyc` also creates cache entries for source-map data).

The transform function should return a `string` containing the result of transforming `input`.

##### factory

Type: `Function(cacheDir: string): transformFunction`

If the `transform` function is expensive to create, and it is reasonable to expect that it may never be called during the life of the process, you may supply a `factory` function that will be used to create the `transform` function the first time it is needed.

##### cacheDir

Type: `string`

The directory where cached transform results will be stored. The directory is automatically created with [`mkdirp`](https://www.npmjs.com/package/mkdirp). You can set `options.createCacheDir = false` if you are certain the directory already exists. 

##### ext

Type: `string`
Default: `empty string`

An extension that will be appended to the salted hash to create the filename inside your cache directory. It is not required, but recommended if you know the file type. Appending the extension allows you to easily inspect the contents of the cache directory with your file browser.

## License

MIT © [James Talmage](http://github.com/jamestalmage)
