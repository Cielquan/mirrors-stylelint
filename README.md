# stylelint mirror


Mirror of stylelint package for pre-commit.

For pre-commit: see https://github.com/pre-commit/pre-commit

For stylelint: see https://github.com/stylelint/stylelint


### Using stylelint with pre-commit

Add this to your `.pre-commit-config.yaml`:

    -   repo: https://github.com/Cielquan/mirrors-stylelint
        rev: ''  # Use the sha / tag you want to point at
        hooks:
        -   id: stylelint

### Plugins and shareable configs

List them in `additional_dependencies` together with `stylelint` itself, because your list replaces the one this hook sets:

    -   repo: https://github.com/Cielquan/mirrors-stylelint
        rev: ''  # Use the sha / tag you want to point at
        hooks:
        -   id: stylelint
            additional_dependencies:
            -   stylelint@17.16.0
            -   stylelint-config-standard-scss@17.0.0

From v17, Stylelint can't find these packages when your config refers to them by name in `extends`, `plugins` or `customSyntax`, and every file fails with:

    ConfigurationError: Could not find "stylelint-config-standard-scss". Do you need to install the package or use the "configBasedir" option?

pre-commit installs `additional_dependencies` into the hook's own environment and puts its `node_modules` folder on `NODE_PATH`. Stylelint 16 searched `NODE_PATH` as a fallback, but Stylelint 17 dropped that fallback ([stylelint/stylelint#8886](https://github.com/stylelint/stylelint/pull/8886)) and only looks in `node_modules` folders above your config file.

You can work around this with a JavaScript config that resolves the package paths itself, since Node's `require.resolve` still reads `NODE_PATH`:

```js
// stylelint.config.mjs
import fs from "node:fs";
import { createRequire } from "node:module";
import path from "node:path";

const require = createRequire(import.meta.url);

function resolve(name) {
  try {
    return require.resolve(name);
  } catch {
    // ESM-only packages such as stylelint-config-standard-scss can't be
    // resolved by require, so point Stylelint at their index.js instead.
    return (
      require.resolve
        .paths(name)
        ?.map((dir) => path.join(dir, name, "index.js"))
        .find((file) => fs.existsSync(file)) ?? name
    );
  }
}

export default {
  extends: [resolve("stylelint-config-standard-scss")],
};
```

This still uses your project's `node_modules` first when the packages are installed there, so running `npx stylelint` and editor integrations keep working. It won't find an ESM-only package whose entry point is somewhere other than `index.js` at the package root.
