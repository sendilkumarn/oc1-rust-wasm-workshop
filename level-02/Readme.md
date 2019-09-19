# Level - 02 🚘

## 01 - Create the project

Create a Rust project with `create-rust-webpack`.

```bash
$ npm init rust-webpack offline-canvas
```

## 02 - Check the project generated

The Rust side and the JavaScript side.

## 03 - Add the dependencies

We will remove the existing feature

```toml
// Cargo.toml
[dependencies.web-sys]
version = "0.3.22"
features = [
    'console'
]
```

and replace it with the following:

```toml
//Cargo.toml

[dependencies.web-sys]
version = "0.3.22"
features = [
  'CanvasRenderingContext2d',
  'CssStyleDeclaration',
  'Document',
  'Element',
  'EventTarget',
  'HtmlCanvasElement',
  'HtmlElement',
  'MouseEvent',
  'Node',
  'Window',
]
```

## 04 - Lets write some Rust

Open the `src/lib.rs` replace the contents with the following:

```rust
use std::cell::Cell;
use std::rc::Rc;
use wasm_bindgen::prelude::*;
use wasm_bindgen::JsCast;


// When the `wee_alloc` feature is enabled, this uses `wee_alloc` as the global
// allocator.
//
// If you don't want to use `wee_alloc`, you can safely delete this.
#[cfg(feature = "wee_alloc")]
#[global_allocator]
static ALLOC: wee_alloc::WeeAlloc = wee_alloc::WeeAlloc::INIT;


#[wasm_bindgen(start)]
pub fn start() -> Result<(), JsValue> {
    // List of colors that we will use. Feel free to add your own favorite hex codes here.
    let colors = vec!["#F4908E", "#F2F097", "#88B0DC", "#F7B5D1", "#53C4AF", "#FDE38C"];
    
    // Calling the window object from the global
    let window = web_sys::window().expect("should have a window in this context");
    // Getting the document object from the Window.
    let document = window.document().expect("window should have a document");
    
    // Create the Canvas element.
    let canvas = document
        .create_element("canvas")?
        .dyn_into::<web_sys::HtmlCanvasElement>()?;

    // Append it to the document's body. Simple HTML 101.
    document.body().unwrap().append_child(&canvas)?;

    canvas.set_width(640);
    canvas.set_height(480);
    canvas.style().set_property("border", "solid")?;

    let context = canvas
        .get_context("2d")?
        .unwrap()
        .dyn_into::<web_sys::CanvasRenderingContext2d>()?;

    let context = Rc::new(context);
    let pressed = Rc::new(Cell::new(false));

    // Event handlers callback.
    { mouse_down(&context, &pressed, &canvas); }
    { mouse_move(&context, &pressed, &canvas); }
    { mouse_up(&context, &pressed, &canvas); }

    // Create divs for color picker
    for c in colors {
        let div = document
            .create_element("div")?
            .dyn_into::<web_sys::HtmlElement>()?;
        div.set_class_name("color");
        {
            click(&context, &div, c.clone());
        }
        
        div.style().set_property("background-color", c);
        let div = div.dyn_into::<web_sys::Node>()?;
        document.body().unwrap().append_child(&div)?;
    }

    Ok(())
}

fn click(context: &std::rc::Rc<web_sys::CanvasRenderingContext2d>, div: &web_sys::HtmlElement, c: &str) {
    let context = context.clone();
    let c = JsValue::from(String::from(c));
    let closure = Closure::wrap(Box::new(move || {
        context.set_stroke_style(&c);            
    }) as Box<dyn FnMut()>);

    div.set_onclick(Some(closure.as_ref().unchecked_ref()));
    closure.forget();
}


fn mouse_up(context: &std::rc::Rc<web_sys::CanvasRenderingContext2d>, pressed: &std::rc::Rc<std::cell::Cell<bool>>, canvas: &web_sys::HtmlCanvasElement) {
    let context = context.clone();
    let pressed = pressed.clone();
    let closure = Closure::wrap(Box::new(move |event: web_sys::MouseEvent| {
        pressed.set(false);
        context.line_to(event.offset_x() as f64, event.offset_y() as f64);
        context.stroke();
    }) as Box<dyn FnMut(_)>);
    canvas.add_event_listener_with_callback("mouseup", closure.as_ref().unchecked_ref()).unwrap();
    closure.forget();
}

fn mouse_move(context: &std::rc::Rc<web_sys::CanvasRenderingContext2d>, pressed: &std::rc::Rc<std::cell::Cell<bool>>, canvas: &web_sys::HtmlCanvasElement){
    let context = context.clone();
    let pressed = pressed.clone();
    let closure = Closure::wrap(Box::new(move |event: web_sys::MouseEvent| {
        if pressed.get() {
            context.line_to(event.offset_x() as f64, event.offset_y() as f64);
            context.stroke();
            context.begin_path();
            context.move_to(event.offset_x() as f64, event.offset_y() as f64);
        }
    }) as Box<dyn FnMut(_)>);
    canvas.add_event_listener_with_callback("mousemove", closure.as_ref().unchecked_ref()).unwrap();
    closure.forget();
}

fn mouse_down(context: &std::rc::Rc<web_sys::CanvasRenderingContext2d>, pressed: &std::rc::Rc<std::cell::Cell<bool>>, canvas: &web_sys::HtmlCanvasElement){
    let context = context.clone();
    let pressed = pressed.clone();

    let closure = Closure::wrap(Box::new(move |event: web_sys::MouseEvent| {
        context.begin_path();
        context.set_line_width(5.0);
        context.move_to(event.offset_x() as f64, event.offset_y() as f64);
        pressed.set(true);
    }) as Box<dyn FnMut(_)>);
    canvas.add_event_listener_with_callback("mousedown", closure.as_ref().unchecked_ref()).unwrap();
    closure.forget();
}
```

## 05 - Create a view 

Replace the contents of `static/index.html` with the following:

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8">
    <title>My Rust + Webpack - some canvas adventure!</title>
    <style>
        .color {
            display: inline-block;
            width: 50px;
            height: 50px;
            border-radius: 50%;
            cursor: pointer;
            margin: 10px;
        }
    </style>
  </head>
  <body>
    <script src="index.js"></script>
  </body>
</html>
```

## 06 - Let us run it

We can produce our WebAssembly binary and the JavaScript glue code with the following command:

```bash
$ npm run start
```

## Reflections

* Walk through the Rust code. 
* Play with the canvas

## Exercises

* Learn more about [web-sys](https://crates.io/crates/web-sys) and [js-sys](https://crates.io/crates/js-sys).

