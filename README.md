# Node "exports" field explanation

Node doesn't allow using `.js` extension for both `esm` and `commonjs` files in the same project when importing files _from_ that project. This is why this `exports` field is **BAD**:

```json
{
  "exports": {
    ".": {
      "import": "./index.esm.js",
      "require": "./index.cjs.js"
    }
  }
}
```

> Run `pnpm install` before running examples

## Prerequisite

We have 3 packages:

- **test-cjs** is a commonjs package that has incorrect `exports` field for "import" files (has `.js` extension)
- **test-esm** is a esm pckage that has incorrect `exports` field for "require" files (has `.js` extension)
- **test-mixed** is commonjs package that has correct `exports` field for "import" and "require" files (has `.mjs` and `.cjs` extensions)

## Examples

When running this package, you will get an error, depending on it's `type` field:

- run `node cjs/test-esm.js` to get `require() of ES modules is not supported` error (cjs `requires` cjs-like file inside esm package)

  - which means you can only use `import` syntax with this module in Node
  - example: when running `node esm/test-esm.mjs`, you will not get an error (esm `imports` esm)

- run `node esm/test-cjs.mjs` to get `The requested module 'test-cjs' is a CommonJS module` error (esm `imports` esm-like file inside cjs package)

  - which means you can only use `require` syntax with this module in Node
  - example: when running `node cjs/test-cjs.js`, you will not get an error (cjs `requires` cjs)

- if you run `node cjs/test-mixed.js` or `node esm/test-mixed.mjs` you will see no errors because it has correct `exports` field.

To fix this, bundle your files with appropriate extensions:

1. If your package doesn't have `"type"` in your `package.json`, use `.cjs`/`.js` extensions, when files have `commonjs` syntax, and `.mjs` extension for files with `esm` syntax

```jsonc
{
  "exports": {
    ".": {
      "import": "./index.mjs",
      "require": "./index.js"  // or "./index.cjs"
    }
  }
}
```

2. If your package has `"type": "commonjs"` in your `package.json`, use `.cjs`/`.js` extensions, when files have `commonjs` syntax, and `.mjs` extension for files with `esm` syntax

```jsonc
{
  "type": "commonjs",
  "exports": {
    ".": {
      "import": "./index.mjs",
      "require": "./index.js" // or "./index.cjs"
    }
  }
}
```

3. If your package has `"type": "module"` in your `package.json`, use `.cjs` extension, when files have `commonjs` syntax, and `.mjs`/`.js` extension for files with `esm` syntax


```jsonc
{
  "type": "module",
  "exports": {
    ".": {
      "import": "./index.js", // or "./index.mjs"
      "require": "./index.cjs"
    }
  }
}
```

> Warning: These examples work with Webpack/Rollup/Vite and other bundler's pipelines because they don't _run_ these files, they only read and analyze them, but they will **FAIL** when run inside Node by tools like Vitest or manually.

To build you project correctly, use one of these configs:

## rollup.config.js

```js
export default {
  input: "src/index.ts",
  output: [
    {
      file: "dist/index.cjs",
      format: "cjs",
    },
    {
      file: "dist/index.mjs",
      format: "esm",
    },
  ],
};
```

## webpack.config.js

> TODO

## vite.config.js

```js
export default {
  build: {
    lib: {
      entry: path.resolve(__dirname, 'src/index.js'),
      fileName: (format) => `index.${format == 'es' ? 'mjs' : 'js'}`
    }
  }
}
```

## tsup.config.js

```js
export default {
    entry: ['index.js'],
    format: ['esm', 'cjs']
}
```

# See also
- [Dual CommonJS/ES module packages in official Node.js documentation](https://nodejs.org/api/packages.html#dual-commonjses-module-packages)
- [Publish ESM and CJS in a single package](https://antfu.me/posts/publish-esm-and-cjs) by [Anthony Fu](https://github.com/antfu)
