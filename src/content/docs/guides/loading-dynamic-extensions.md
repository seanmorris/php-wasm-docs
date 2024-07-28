---
title: Loading Dynamic Extensions
description: Loading Dynamic Extensions as JS Modules
---

### Loading Dynamic Extensions as JS Modules

Dynamic extensions can be loaded as modules: So long as the main file of the module defines the `getLibs` and `getFiles` methods, extensions may be loaded like so:

```javascript
new PhpNode({sharedLibs:[ await import('php-wasm-intl') ]})
```

Dynamic extensions can also be loaded as modules from any static HTTP server with an ESM directory structure.

```javascript
// This will load both sqlite.so & php8.x-sqlite.so:
const php = new PhpWeb({sharedLibs: [ await import('https://cdn.jsdelivr.net/npm/php-wasm-sqlite') ]});
```

Sadly, this notation is not available for Service Workers, since they do not yet support dynamic `imports()`. Hopefully this will change soon.

### Compiling extensions

Extensions may be compiled as `dynamic`, `shared`, or `static`. See `Custom Builds` for more information on compiling php-wasm.

* dynamic - these extensions may be loaded selectively at runtime.
* shared - these extensions will always be loaded at startup and can be cached and reused.
* static - these extensions will be built directly into the main wasm binary (may cause a huge filesize).

## 📦 Loading Files

### Loading single files at runtime

When spawning a new instance of PHP, a `files` array can be provided to be loaded into the filesystem. For example, the `php-intl` extension requires us to load `icudt72l.dat` into the  `/preload` directory.

```javascript
const sharedLibs = [`https://unpkg.com/php-wasm-intl/php\${PHP_VERSION}-intl.so`];

const files = [
    {
        name: 'icudt72l.dat',
        parent: '/preload/',
        url: 'https://unpkg.com/php-wasm-intl/icudt72l.dat'
    }
];

const php = new PhpWeb({sharedLibs, files});
```
### Preloaded FS

Use the `PRELOAD_ASSETS` key in your `.php-wasm-rc` file to define a list of files and directories to include by default.

The files and directories will be collected into a single directory. Individual files & directories will appear in the top level, while directories will maintain their internal structure.

These files & directories will be available under `/preload` in the final package, packaged into the `.data` file that is built along with the `.wasm` file.

```bash
PRELOAD_ASSETS='/path/to/file.txt /some/directory /path/to/other_file.txt /some/other/directory'
```

### locateFile

You can provide the `locateFile` option to php-wasm as a callback to map the names of files to URLs where they're loaded from. `undefined` can be returned as a fallback to default.

You can use this if your static assets are served from a different directory than your javascript.

This applies to `.wasm` files, shared libraries, single files and preloaded FS packages in `.data` files.

```javascript
const php = new PhpWeb({locateFile: filename => `/my/static/path/${filename}`});
```