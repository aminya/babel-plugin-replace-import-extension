# @aminya/babel-plugin-replace-import-extension

Note: this is a temporary fork until [this pull request](https://github.com/silane/babel-plugin-replace-import-extension/pull/9) is merged.

Babel plugin to replace extension of file name written in import statement and
dynamic import.

## Installation
```shell
npm install --save-dev @aminya/babel-plugin-replace-import-extension
```

## Example
With the option:
```json
{ "extMapping": { ".js": ".mjs" }}
```

### In
```javascript
import { foo } from './module1.js';
export { bar } from './module2.js'; // Works for re-exporting
const promise = import('./module3' + '.js'); // Also works for dynamic import!
```

### Out
```javascript
import { foo } from './module1.mjs';
export { bar } from './module2.mjs';

// In dynamic import, function to replace extension is inserted.
// Note the actual code is not exactly the same.
const promise = import(__transformExtension('./module3' + '.js'));
```

## Why We Need This Plugin?
When you develop a npm package that includes both ESModule and CommonJS version
of the code, there is two ways to tell Node which file is which version.

- Distinguish files by their extension, `mjs` for ESModule and `cjs` for
  CommonJS.
- If two versions are located in separate directories, put a `package.json`
  with a `type` field specified to the directory.

If you choose the former and you write your code in ESModule and transpile it
to CommonJS, you have to change the extension of the files while transpiling.

In Babel CLI, extension of the output file name can be changed with
`--out-file-extension` option. But the file name referenced inside the code
is not changed. In this case, this plugin comes into play.

Note that the conversion is performed only on relative file name
(starts with `./` or `../`), because built-in packages or packages importing
from `node_modules` should not be converted.

## Usage
If project root `package.json` has `type` field of `module`, Babel config of
```json
{
  "plugins": [
    ["replace-import-extension", { "extMapping": { ".js": ".cjs" }}],
    ["@babel/transform-modules-commonjs"]
  ]
}
```
will convert the file extension from `.js` to `.cjs` and convert ESModule to
CommonJS, allowing both version's code exist together while Node can handle
each versions correctly. (`@babel/plugin-transform-modules-commonjs` must be
installed.) Or if you also need other translations, `@babel/env` preset can be
used together like,
```json
{
  "presets": [["@babel/env"]],
  "plugins": [
    ["replace-import-extension", { "extMapping": { ".js": ".cjs" }}]
  ]
}
```


If project root `package.json` has no `type` field or has `type` field of
`cjs`, ESModule files must be explicitly marked by `mjs` extension, which can
be done by Babel config of
```json
{
  "plugins": [
    ["replace-import-extension", { "extMapping": { ".js": ".mjs" }}]
  ]
}
```
Once again, `--out-file-extension` option must be used together to change the
output file extension.

## Supporting both `mjs` and `cjs` in the same package

If you are using `.mjs` for your source files, you can use babel to generate `.cjs` files for backwards compatibility:

```json
{
  "presets": [["@babel/env"]],
  "plugins": [
    ["replace-import-extension", { "extMapping": { ".mjs": ".cjs" }}]
  ]
}
```

In your `package.json` specify the entries accordingly:

```json
{
  "main": "dist/index.cjs",
  "module": "src/index.mjs",
  "source": "src/index.mjs",
  "exports": {
    ".": {
      "require": "dist/index.cjs",
      "import": "src/index.mjs"
    },
    "src/index.mjs": {
      "import": "src/index.mjs"
    },
    "dist/index.cjs": {
      "require": "dist/index.cjs",
      "import": "dist/index.cjs"
    }
  },
  "scripts": {
    "build.cjs": "babel -d dist/ src/ --out-file-extension .cjs",
    "prepare": "npm run build.cjs"
  }
}
```

## Options
### `extMapping`
`Object`, defaults to `{}`.

Mapping of original extension to converted extension.
Leading `.` is mandatory.

Both the original and the converted extensions can be empty string `''`, which means
no extension. You can use this feature to add or remove extension.
