# Level - 03 🚀

## 01 - Create the project

```bash
npm init rust-webpack markdown-rust
```

## 02 - Check the project generated


## 03 - Add the dependencies

```toml
comrak = "0.6"
```

## 04 - Lets write some Rust

```rust
// src/lib.rs
use comrak::{markdown_to_html, ComrakOptions};
use wasm_bindgen::prelude::*;

#[wasm_bindgen]
pub fn parse(input: &str) -> String {
    markdown_to_html(&input.to_string(), &ComrakOptions::default())
}
```

## 05 - Some JavaScript into the mix

Replace the contents of `js/index.js`.

```javascript
// js/index.js
const rust = import('../pkg/index.js');

rust.then(module => {
    console.log(module.parse('# some markdown content')); 
});
```

## 06 - Let us run it

We can produce our WebAssembly binary and the JavaScript glue code with the following command:

```bash
$ npm run start
```

## Interested to explore further...

https://dev.to/sendilkumarn/increase-rust-and-webassembly-performance-382h

https://dev.to/sendilkumarn/reduce-your-webassembly-binaries-72-from-56kb-to-26kb-to-16kb-40mi

## Reflections

* Walk through the code. 

## Exercises

* Go ahead and publish your libraries.

## Bonus level 🦄
* Check [this](https://github.com/sendilkumarn/rust-wasm-workshop/tree/master/going-further) for game-boy with Rust and WebAssembly

