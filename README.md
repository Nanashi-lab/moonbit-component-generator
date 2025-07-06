# moonbit-component-generator

Rust library embedding the MoonBit compiler (compiled to WASM, executed on V8) and the MoonBit core library and various
tools from the WASM ecosystem to programmatically generate MoonBit source code and compile it to WebAssembly Components.

The crate is fully self-contained, does not require any external dependencies.
Some example use cases are exposed by crate features:

- `get-script`: generates a WASM component with a single exported function `get-script` that returns an arbitrary string
  embedded in the component. An example use case for this can be to compose JavaScript scripts with a precompiled WASM
  JS engine.
- `typed-config`: takes a simple WIT interface with functions getting "typed configuration", and generates a WASM
  component that implements this interface using either the WASI environment or the WASI config APIs.

## Use cases
The crate implements the general machinery for generating WASM components from MoonBit source, and it also exports a few example use cases:

### "Get Script" component
The simplest example generates a WASM component that exports a single function `get-script` which returns a string. 
This can be used to attach dynamic content to a statically compiled WASM component via composition, 
for example providing user-defined JavaScript code to a precompiled WASM JavaScript engine. 

To use this generator, just provide the script contents and the target WASM path:

```rust
use moonbit_component_generator::get_script::generate_get_script_component;

fn main() {
    let script = "console.log('Hello, world!');";
    let wasm_path = "output/get_script.wasm";
    generate_get_script_component(script, wasm_path).expect("Failed to generate get-script component");
}
```

### "Typed Config" component
The "Typed Config" component generator inspects a given WIT package and generates a WASM component that implements it
using either the WASI environment API or the WASI config APIs. This can be used to have a statically typed configuration
API provided for WASM components of any language.

The current implementation only supports string configuration values, but it can be extended to support more types including
records and variants, as the code generator has access to the resolved WIT interface.

To use this generator, provide the WIT package path and the target WASM path:

```rust
use moonbit_component_generator::typed_config::generate_typed_config_component;

fn main() {
  let wit = r#"
                package example:typed-config;

                interface config {
                    username: func() -> string;
                    password: func() -> string;
                    server: func() -> string;
                }

                world example-config {
                    export config;
                }
            "#;
  let wasm_path = "output/typed_config.wasm";
  generate_typed_config_component(wit, wasm_path).expect("Failed to generate typed config component");
}

```

## Development

- `moonc_wasm` is a cloned and patched version of https://github.com/moonbitlang/moonc_wasm to make it library crate.
- `core` is a git submodule containing the MoonBit core library (https://github.com/moonbitlang/core)

To update and build the MoonBit core library:

```
git submodule update --recursive
moon bundle --target wasm
```

The bundled core library is included in the compiled crate using the `include_dir!` macro.

Running the tests require `wasmtime-cli` to be installed with the following features enabled:
- `component-model`
- `wasi-config`
