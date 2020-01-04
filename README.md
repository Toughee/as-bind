# as-bind

<!-- Badges -->

[![Build Status](https://travis-ci.org/torch2424/as-bind.svg?branch=master)](https://travis-ci.org/torch2424/as-bind)
![npm bundle size (minified)](https://img.shields.io/bundlephobia/min/as-bind.svg)
![npm](https://img.shields.io/npm/dt/as-bind.svg)
![npm version](https://img.shields.io/npm/v/as-bind.svg)
![GitHub](https://img.shields.io/github/license/torch2424/as-bind.svg)

Isomorphic library to handle passing high-level data structures between AssemblyScript and JavaScript. 🤝🚀

![Asbind Markdown Parser Demo Gif](./assets/asbind.gif)

## Features

- The library is Isomorphic. Meaning it supports both the Browser, and Node! And has ESM, CommonJS, and IIFE bundles! 🌐
- Wraps around the [AssemblyScript Loader](https://github.com/AssemblyScript/assemblyscript/tree/master/lib/loader). The loader handles all the heavy-lifting of passing data into WebAssembly linear memory. 💪
- Wraps around imported JavaScript functions, and exported AssemblyScript functions of the AssemblyScript Wasm Module. This allows high-level data types to be passed to AssemblyScript functions, without having to touch Wasm memory! 🤯
- The library works at runtime, so no generated code that you have to maintain and try to get to work in your environment. 🏃
- Maintains great performance (relative to generating the corresponding JavaScript code), by using [Speculative Execution](https://en.wikipedia.org/wiki/Speculative_execution), and caching types passed between functions. 🤔
- The library is [< 5KB (minified and gzip'd)](https://bundlephobia.com/result?p=as-bind) and tree-shakeable! 📦🌲
- This library is currently the [wasm-bindgen](https://github.com/rustwasm/wasm-bindgen) in the Rust/Wasm ecosystem, for AssemblyScript. 😀

## Installation

You can install as-bind in your project by running the following:

`npm install --save as-bind`

## Quick Start

**1. In your Assemblyscript**

Export everything from the asbind assemblyscript library:

```typescript
// This should be in your entry point file for your Assemblyscript project

// '../node_modules/as-bind/*' should be the relative path to this directory in your project
export * from "../node_modules/as-bind/lib/assembly/asbind.ts";
```

After this, let's export an example function we can try:

```typescript
export function myExportedFunctionThatTakesAString(value: string): string {
  return "AsBind: " + value;
}
```

**2. In your Javascript**

In the browser using ESM Syntax:

```javascript
import { AsBind } from "as-bind";

const wasm = fetch("./path-to-my-wasm.wasm");

const asyncTask = async () => {
  const asBindInstance = await AsBind.instantiate(wasm);

  // You can now use your wasm / asbind instance!
  const response = asBindInstance.exports.myExportedFunctionThatTakesAString(
    "Hello World!"
  );
  console.log(response); // AsBind: Hello World!
};
asyncTask();
```

Or we can also use Node:

```javascript
const { AsBind } = require("as-bind");
const fs = require("fs");

const wasm = fs.readFileSync("./path-to-my-wasm.wasm");

const asyncTask = async () => {
  const asBindInstance = await AsBind.instantiate(wasm);

  // You can now use your wasm / asbind instance!
  const response = asBindInstance.exports.myExportedFunctionThatTakesAString(
    "Hello World!"
  );
  console.log(response); // AsBind: Hello World!
};
asyncTask();
```

## Motivation

This library was inspired by several chats I had with some awesome buddies of mine in the WebAssembly Communitty:

- Till Schneidereit and I had a chat about [WasmBoy](https://github.com/torch2424/wasmboy), and about how I had a really good experience writing the emulator, even though I had to do my own memory management. But they helped me realize, building something low level isn't that bad with manual memory management, but building something like a markdown parser would be very tedious since you have to manually write the string back and forth. Which then inspired this library, and it's [markdown parser demo](https://torch2424.github.io/as-bind/).

- While I was building [WasmByExample](https://wasmbyexample.dev/?programmingLanguage=assemblyscript) I wanted to start building the "High Level Data Structures" section. I then realized how much work it would be to maintain code for passing data between WebAssembly Linear memory would be for each data type, and how much work it would be to created each individual example. Then, my buddy [Ashley Williams](https://twitter.com/ag_dubs) helped me realize, if your docs are becoming too complex, it may be a good idea to write a tool. That way you have less docs to write, and users will have an easier time using your stuff!

Thus, this library was made to help AssemblyScript/JavaScript users build awesome things! 😄🎉

## Supported Data Types

**TL;DR:** Currently Numbers, Strings, and Typed Arrays are supported. Returning a high-level data type from an imported JavaScript function, and passing AssemblyScript Classes will be coming later.

<!-- Generated from: https://www.tablesgenerator.com/markdown_tables# -->

| Function & Direction                        | Number (Integers and Floats) | Strings | Int8Array | Uint8Array | Int16Array | UInt16Array | Int32Array | Uint32Array | Float32Array | Float64Array | AssemblyScript Classes |
| ------------------------------------------- | ---------------------------- | ------- | --------- | ---------- | ---------- | ----------- | ---------- | ----------- | ------------ | ------------ | ---------------------- |
| Exported AssemblyScript Function Parameters | ✔️                           | ✔️      | ✔️        | ✔️         | ✔️         | ✔️          | ✔️         | ✔️          | ✔️           | ✔️           | ❌                     |
| Exported AssemblyScript Function Return     | ✔️                           | ✔️      | ✔️        | ✔️         | ✔️         | ✔️          | ✔️         | ✔️          | ✔️           | ✔️           | ❌                     |
| Imported JavaScript Function Paramters      | ✔️                           | ✔️      | ✔️        | ✔️         | ✔️         | ✔️          | ✔️         | ✔️          | ✔️           | ✔️           | ❌                     |
| Imported JavaScript Function Return         | ✔️                           | ❌      | ❌        | ❌         | ❌         | ❌          | ❌         | ❌          | ❌           | ❌           | ❌                     |

## Supported AssemblyScript Runtime Variants

as-bind only supports AssemblyScript modules compiled with the `--runtime full` (default), and `--runtime stub` flags. These should be the only supported modes, because these runtime variants specify that you would like types / objects to be created externally as well as internally. Other runtime variants would mean that you DO NOT want anything externally created for your wasm module.

Please see the [AssemblyScript Docs on runtime variants](https://docs.assemblyscript.org/details/runtime) for more information.

## Performance

as-bind does all of it's data passing at runtime. Meaning this will be slower than a code generated bindings generator, such as something like [wasm-bindgen](https://github.com/rustwasm/wasm-bindgen). This is because, as-bind needs to cycle through every supported type on every paremeter or return value for each function, whenever the function is called. However, this is mitigated due to the Speculative execution that the library implements. Which. in this case it means, that the library by default will assume the type of value being passed to, or returned by a function will not change, and will only have to cycle through the params once, and then after that, it would be as fast as a code generated solution (in theory). This speculative execution can be turned off as specified in the Reference API.

If your project is doing one-off processing using a high level data type, this project should have a very small impact on performance of your project. However, if you project is doing it's processing in a very time constrained loop (such as a game running at 60fps), you may want to be more considerate when choosing this library. The speculative execution should greatly help in the amount of time to pass high level data types, but if your game is already not running as fast as you would like, you may want to avoid this library, or even not using high level data types, for passing memory to your WebAssembly module.

Eventually for the most performant option, we would want to do some JavaScript code generation in the AssemblyScript compiler itself, as part of an `as-bindgen` project for the most performant data passing.

**TL;DR** This library should be fast, but depending on your project you may want some more careful consideration.

## Reference API

### AsBind

The default exported ESM class of `as-bind`, also available as `import { AsBind } from "as-bind"` / `const { AsBind } = require('as-bind')`.

#### Class Properties

The `AsBind` class is meant to vaugely act as the [WebAssembly](https://developer.mozilla.org/en-US/docs/WebAssembly) Object exposed to JavaScript environments.

##### version

`AsBind.version: string`

Value that is the current version of your imported AsBind.

##### instantiate

```typescript
AsBind.instantiate: (
  moduleOrBuffer: (
    WebAssembly.Module |
    BufferSource |
    Response |
    PromiseLike<WebAssembly.Module |
    BufferSource |
    Response
  ),
  imports?: WasmImports
) => Promise<AsBindInstance>`
```

This function is the equivalent to the [AssemblyScript Loader instantiate](https://github.com/AssemblyScript/assemblyscript/tree/master/lib/loader#api) function, which is similar to the [WebAssembly.instantiateStreaming](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_objects/WebAssembly/instantiateStreaming) function. It essentially takes as it's parameters:

- Any type of object that can be (resolved) and instantied into a WebAssembly instance. Which in our case would be an AsBindInstance.

- A [WebAssembly importObject](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_objects/WebAssembly/instantiateStreaming), which would have all of your imported functions that can be called from within your AssemblyScript module.

#### Instance Properties

An AsBindInstance is vaugley similar to a [WebAssembly instance](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/WebAssembly/Instance).

TODO Make fields private like importObject and instantiate

##### exports

##### unboundExports

##### enableExportFunctionTypeCaching

##### disableExportFunctionTypeCaching

##### enableImportFunctionTypeCaching

##### disableExportFunctionTypeCaching

## License

[MIT](https://oss.ninja/mit/torch2424).
