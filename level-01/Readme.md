# Level - 01 🐎

## 01 - Create the project

Create a Rust project with `Cargo`.

> What is Cargo? -- Cargo is package manager for Rust similar to NPM for JavaScript.

```bash
$ cargo new add --lib
```

## 02 - Create the function

Once we have the project let's add our code to it. We'll add just a simple function named `add`:

```rust
// src/lib.rs

pub fn add(a: usize, b: usize) -> usize {
    a + b
}
```

## 03 - Annotate the function

To make this callable from JavaScript through WebAssembly we'll need some glue code. 
Luckily the [wasm-bindgen](https://crates.io/crates/wasm-bindgen) can do all of this for us. 
Let's add wasm-bindgen to our crate as a dependency and then add the `wasm-bindgen` attribute to our add function.

```rust
use wasm_bindgen::prelude::*;

#[wasm_bindgen]
pub fn add(a: usize, b: usize) -> usize {
    a + b
}
```

## 04 - Add the dependencies

Add the wasm-bindgen dependency and make the library as a dynamic library. 

```toml
// Cargo.toml

[lib]
crate-type = ["cdylib", "rlib"]

[dependencies]
wasm-bindgen="0.2.50"
```

## 05 - wasm-pack

While we could compile this to a WebAssembly binary using cargo, this won't give us the JavaScript glue code we need. 
For that we could use the `wasm-bindgen` cli tool that reads the WebAssembly binary and produces JavaScript glue code that we can use to call the WebAssembly code. 
However, this is a multistep process, and we can have a tool do all these steps for us: `wasm-pack`.

To install `wasm-pack` simply run the following:

```bash
$ cargo install wasm-pack
```

## 06 - Let us build it

We can produce our WebAssembly binary and the JavaScript glue code with the following command:

```bash
$ wasm-pack build --target web --release
```

We're running the command with the option `--target web` which will allow us to run inside a normal `<script>` tag instead of the default of being optimized for running from a JavaScript bundler.

This will run cargo and the `wasm-bindgen` CLI tool to create the WebAssembly binary and the JavaScript glue code and place it inside of `pkg` folder. wasm-pack also produces TypeScript definitions for us as well if we choose to use TypeScript.

## 07 - Let us run it

We can then hook it all up by creating an `index.html` file that simply references the generated JavaScript glue code. We'll need to call the generated `init` function and then we can all the `add` function with no issue:

```javascript
  <script type="module">
    import init, { add } from './pkg/add.js';

    init().then(() => {
      const result = add(1, 2);
      console.log(`1 + 2 = ${result}`);
    });
  </script>
```

If we try to open the `index.html` file in a browser, we'll find an error in the console. This is because we're importing a module at a relative path which will effectively be a cross-origin request which isn't allowed.

We'll need to serve our example from a webserver so that all requests go to the same domain (i.e., localhost). Feel free to use your favorite simple http server. 

Some options are:
* Brian Anderson's [`basic-http-server`](https://github.com/brson/basic-http-server).
* If you have Python 2 - `python -m SimpleHTTPServer` 
* Else with Python 3 - `python3 -m http.server`

If we access the `index.html` file through the web server, we'll finally see our message printed out to the console.

## Reflections

* Walk through the generated JavaScript binding files. 
* Let us expand with `cargo expand`

## Exercises

* Play with passing more complex types like Strings, Vectors and references to such types to the function. 
Inspect the JavaScript and see if you can make sense of what's going on. 

Check out [here](https://rustwasm.github.io/docs/wasm-bindgen/reference/types/string.html) for more samples.

