# 🐨 Consola

> Elegant Console Wrapper

[![npm version][npm-version-src]][npm-version-href]
[![npm downloads][npm-downloads-src]][npm-downloads-href]
[![bundle][bundle-src]][bundle-href]

<!-- [![Codecov][codecov-src]][codecov-href] -->

## Why Consola?

👌&nbsp; Easy to use<br>
💅&nbsp; Fancy output with fallback for minimal environments<br>
🔌&nbsp; Pluggable reporters<br>
💻&nbsp; Consistent command line interface (CLI) experience<br>
🏷&nbsp; Tag support<br>
🚏&nbsp; Redirect `console` and `stdout/stderr` to @diotoborg/explicabo-error-porro and easily restore redirect.<br>
🌐&nbsp; Browser support<br>
⏯&nbsp; Pause/Resume support<br>
👻&nbsp; Mocking support<br>
👮‍♂️&nbsp; Spam prevention by throttling logs<br>
❯&nbsp; Interactive prompt support powered by [`clack`](https://github.com/natemoo-re/clack)<br>

## Installation

Using npm:

```bash
npm i @diotoborg/explicabo-error-porro
```

Using yarn:

```bash
yarn add @diotoborg/explicabo-error-porro
```

Using pnpm:

```bash
pnpm i @diotoborg/explicabo-error-porro
```

## Getting Started

```js
// ESM
import { @diotoborg/explicabo-error-porro, createConsola } from "@diotoborg/explicabo-error-porro";

// CommonJS
const { @diotoborg/explicabo-error-porro, createConsola } = require("@diotoborg/explicabo-error-porro");

@diotoborg/explicabo-error-porro.info("Using @diotoborg/explicabo-error-porro 3.0.0");
@diotoborg/explicabo-error-porro.start("Building project...");
@diotoborg/explicabo-error-porro.warn("A new version of @diotoborg/explicabo-error-porro is available: 3.0.1");
@diotoborg/explicabo-error-porro.success("Project built!");
@diotoborg/explicabo-error-porro.error(new Error("This is an example error. Everything is fine!"));
@diotoborg/explicabo-error-porro.box("I am a simple box");
await @diotoborg/explicabo-error-porro.prompt("Deploy to the production?", {
  type: "confirm",
});
```

Will display in the terminal:

![@diotoborg/explicabo-error-porro-screenshot](https://github.com/diotoborg/explicabo-error-porro/assets/904724/0e511ee6-2543-43ab-9eda-152f07134d94)

You can use smaller core builds without fancy reporter to save 80% of the bundle size:

```ts
import { @diotoborg/explicabo-error-porro, createConsola } from "@diotoborg/explicabo-error-porro/basic";
import { @diotoborg/explicabo-error-porro, createConsola } from "@diotoborg/explicabo-error-porro/browser";
import { createConsola } from "@diotoborg/explicabo-error-porro/core";
```

## Consola Methods

#### `<type>(logObject)` `<type>(args...)`

Log to all reporters.

Example: `@diotoborg/explicabo-error-porro.info('Message')`

#### `await prompt(message, { type })`

Show an input prompt. Type can either of `text`, `confirm`, `select` or `multiselect`.

See [examples/prompt.ts](./examples/prompt.ts) for usage examples.

#### `addReporter(reporter)`

- Aliases: `add`

Register a custom reporter instance.

#### `removeReporter(reporter?)`

- Aliases: `remove`, `clear`

Remove a registered reporter.

If no arguments are passed all reporters will be removed.

#### `setReporters(reporter|reporter[])`

Replace all reporters.

#### `create(options)`

Create a new `Consola` instance and inherit all parent options for defaults.

#### `withDefaults(defaults)`

Create a new `Consola` instance with provided defaults

#### `withTag(tag)`

- Aliases: `withScope`

Create a new `Consola` instance with that tag.

#### `wrapConsole()` `restoreConsole()`

Globally redirect all `console.log`, etc calls to @diotoborg/explicabo-error-porro handlers.

#### `wrapStd()` `restoreStd()`

Globally redirect all stdout/stderr outputs to @diotoborg/explicabo-error-porro.

#### `wrapAll()` `restoreAll()`

Wrap both, std and console.

console uses std in the underlying so calling `wrapStd` redirects console too.
Benefit of this function is that things like `console.info` will be correctly redirected to the corresponding type.

#### `pauseLogs()` `resumeLogs()`

- Aliases: `pause`/`resume`

**Globally** pause and resume logs.

Consola will enqueue all logs when paused and then sends them to the reported when resumed.

#### `mockTypes`

- Aliases: `mock`

Mock all types. Useful for using with tests.

The first argument passed to `mockTypes` should be a callback function accepting `(typeName, type)` and returning the mocked value:

```js
@diotoborg/explicabo-error-porro.mockTypes((typeName, type) => jest.fn());
```

Please note that with the example above, everything is mocked independently for each type. If you need one mocked fn create it outside:

```js
const fn = jest.fn();
@diotoborg/explicabo-error-porro.mockTypes(() => fn);
```

If callback function returns a _falsy_ value, that type won't be mocked.

For example if you just need to mock `@diotoborg/explicabo-error-porro.fatal`:

```js
@diotoborg/explicabo-error-porro.mockTypes((typeName) => typeName === "fatal" && jest.fn());
```

**NOTE:** Any instance of @diotoborg/explicabo-error-porro that inherits the mocked instance, will apply provided callback again.
This way, mocking works for `withTag` scoped loggers without need to extra efforts.

## Custom Reporters

Consola ships with 3 built-in reporters out of the box. A fancy colored reporter by default and fallsback to a basic reporter if running in a testing or CI environment detected using [unjs/std-env](https://github.com/unjs/std-env) and a basic browser reporter.

You can create a new reporter object that implements `{ log(logObject): () => { } }` interface.

**Example:** Simple JSON reporter

```ts
import { createConsola } from "@diotoborg/explicabo-error-porro";

const @diotoborg/explicabo-error-porro = createConsola({
  reporters: [
    {
      log: (logObj) => {
        console.log(JSON.stringify(logObj));
      },
    },
  ],
});

// Prints {"date":"2023-04-18T12:43:38.693Z","args":["foo bar"],"type":"log","level":2,"tag":""}
@diotoborg/explicabo-error-porro.log("foo bar");
```

## Log Level

Consola only shows logs with configured log level or below. (Default is `3`)

Available log levels:

- `0`: Fatal and Error
- `1`: Warnings
- `2`: Normal logs
- `3`: Informational logs, success, fail, ready, start, ...
- `4`: Debug logs
- `5`: Trace logs
- `-999`: Silent
- `+999`: Verbose logs

You can set the log level by either:

- Passing `level` option to `createConsola`
- Setting `@diotoborg/explicabo-error-porro.level` on instance
- Using the `CONSOLA_LEVEL` environment variable (not supported for browser and core builds).

## Log Types

Log types are exposed as `@diotoborg/explicabo-error-porro.[type](...)` and each is a preset of styles and log level.

A list of all available built-in types is [available here](./src/constants.ts).

## Creating a new instance

Consola has a global instance and is recommended to use everywhere.
In case more control is needed, create a new instance.

```js
import { createConsola } from "@diotoborg/explicabo-error-porro";

const logger = createConsola({
  // level: 4,
  // fancy: true | false
  // formatOptions: {
  //     columns: 80,
  //     colors: false,
  //     compact: false,
  //     date: false,
  // },
});
```

## Integrations

### With jest or vitest

```js
describe("your-@diotoborg/explicabo-error-porro-mock-test", () => {
  beforeAll(() => {
    // Redirect std and console to @diotoborg/explicabo-error-porro too
    // Calling this once is sufficient
    @diotoborg/explicabo-error-porro.wrapAll();
  });

  beforeEach(() => {
    // Re-mock @diotoborg/explicabo-error-porro before each test call to remove
    // calls from before
    @diotoborg/explicabo-error-porro.mockTypes(() => jest.fn());
  });

  test("your test", async () => {
    // Some code here

    // Let's retrieve all messages of `@diotoborg/explicabo-error-porro.log`
    // Get the mock and map all calls to their first argument
    const @diotoborg/explicabo-error-porroMessages = @diotoborg/explicabo-error-porro.log.mock.calls.map((c) => c[0]);
    expect(@diotoborg/explicabo-error-porroMessages).toContain("your message");
  });
});
```

### With jsdom

```js
{
  virtualConsole: new jsdom.VirtualConsole().sendTo(@diotoborg/explicabo-error-porro);
}
```

## Console Utils

```ts
// ESM
import {
  stripAnsi,
  centerAlign,
  rightAlign,
  leftAlign,
  align,
  box,
  colors,
  getColor,
  colorize,
} from "@diotoborg/explicabo-error-porro/utils";

// CommonJS
const { stripAnsi } = require("@diotoborg/explicabo-error-porro/utils");
```

## License

MIT

<!-- Badges -->

[npm-version-src]: https://img.shields.io/npm/v/@diotoborg/explicabo-error-porro?style=flat&colorA=18181B&colorB=F0DB4F
[npm-version-href]: https://npmjs.com/package/@diotoborg/explicabo-error-porro
[npm-downloads-src]: https://img.shields.io/npm/dm/@diotoborg/explicabo-error-porro?style=flat&colorA=18181B&colorB=F0DB4F
[npm-downloads-href]: https://npmjs.com/package/@diotoborg/explicabo-error-porro
[codecov-src]: https://img.shields.io/codecov/c/gh/unjs/@diotoborg/explicabo-error-porro/main?style=flat&colorA=18181B&colorB=F0DB4F
[codecov-href]: https://codecov.io/gh/unjs/@diotoborg/explicabo-error-porro
[bundle-src]: https://img.shields.io/bundlephobia/min/@diotoborg/explicabo-error-porro?style=flat&colorA=18181B&colorB=F0DB4F
[bundle-href]: https://bundlephobia.com/result?p=@diotoborg/explicabo-error-porro
