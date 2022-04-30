# IoC Context Modules

**Dynamically loadable [IoC] context modules conforming to [@proc7ts/context-values] API**

[![NPM][npm-image]][npm-url]
[![Build Status][build-status-img]][build-status-link]
[![Code Quality][quality-img]][quality-link]
[![Coverage][coverage-img]][coverage-link]
[![GitHub Project][github-image]][github-url]
[![API Documentation][api-docs-image]][api-docs-url]

Context module can be used to provide multiple assets for [IoC] context at once. Unloading the module revokes all assets
provided by it.

[npm-image]: https://img.shields.io/npm/v/@proc7ts/context-modules.svg?logo=npm
[npm-url]: https://www.npmjs.com/package/@proc7ts/context-modules
[build-status-img]: https://github.com/proc7ts/context-modules/workflows/Build/badge.svg
[build-status-link]: https://github.com/proc7ts/context-modules/actions?query=workflow:Build
[quality-img]: https://app.codacy.com/project/badge/Grade/2d46bc304056475881239a28a739fc93
[quality-link]: https://www.codacy.com/gh/proc7ts/context-modules/dashboard?utm_source=github.com&utm_medium=referral&utm_content=proc7ts/context-modules&utm_campaign=Badge_Grade
[coverage-img]: https://app.codacy.com/project/badge/Coverage/2d46bc304056475881239a28a739fc93
[coverage-link]: https://www.codacy.com/gh/proc7ts/context-modules/dashboard?utm_source=github.com&utm_medium=referral&utm_content=proc7ts/context-modules&utm_campaign=Badge_Coverage
[github-image]: https://img.shields.io/static/v1?logo=github&label=GitHub&message=project&color=informational
[github-url]: https://github.com/proc7ts/context-modules
[api-docs-image]: https://img.shields.io/static/v1?logo=typescript&label=API&message=docs&color=informational
[api-docs-url]: https://proc7ts.github.io/context-modules/
[ioc]: https://en.wikipedia.org/wiki/Inversion_of_control
[@proc7ts/context-values]: https://www.npmjs.com/package/@proc7ts/context-values

Usage example:

```typescript
import { CxModule } from '@proc7ts/context-modules';

// Construct new module.
const myModule = new CxModule('my module', {
  setup(setup) {
    // Provide assets for `Foo` entry.
    setup.provide(cxConstAsset(Foo, 'foo'));
  },
});

// Add the module to context.
const myModuleSupply = cxBuilder.provide(myModule);

// Start using the module.
// The first usage of the module loads it.
const myModuleUse = await context.get(myModule).use();

// Await for the module to load.
await myModuleUse.whenReady;

// Access the value provided by module.
console.log(context.get(Foo)); // 'foo'

// Stop using the module.
// Once tha last usage stopped, the module is unloaded.
myModuleUse.supply.off();

// Revoke the module.
myModuleSupply.off();
```

Context module's constructor accepts a human-readable module name, and options object.

The following options supported:

- `needs` - A module or modules the constructed one requires.

  The listed modules will be loaded prior to loading the constructed one.

- `has` - A module or modules the constructed one provides.

  When specified, the constructed module will be loaded _instead_ of the listed ones.

- `setup()` - A method that sets up constructed module.

  May be synchronous or asynchronous.

  Accepts a `CxModule.Setup` instance with the following properties:

  - `get()` - For accessing context values.

    Inherited from `ContextValues` interface.

  - `provide()` - For providing context entry assets.

    The same as in `CxBuilder`.

    All assets provided by this method will be revoked once the module unloaded.

  - `initBy()` - For registering module initializers.

To use a module:

1. Create its implementation.
2. Provide it by `CxBuilder.provide(module)` call.
3. Obtain its handle from context by calling `CxValues.get(module)`.
4. Start using it by calling the `.use()` method of obtained module handle.
5. Wait for the module to load and become ready to use: `await use.whenReady`.
6. When no longer needed, cut off the module use supply: `use.supply.off()`.

   The module will be unloaded once no longer uses remain.

   Note that the module can be in use indirectly when required by another one.
