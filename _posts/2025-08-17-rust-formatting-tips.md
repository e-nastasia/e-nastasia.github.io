## Rust formatting tips

Formatting syntax in rust allows for different convenience features.

## Pre-requisites

You can print types that implement either `Display` or `Debug` -- these are the traits responsible for representation of values in the text form.
Their behavior is similar to other languages: for example, a type implementing `Display` trait in rust is similar to a type having the `__str__` function in python: both provide a readable way to output the type. And same as with `__str__` in python, rust doesn't try to implement this one by default and instead relies heavily on `Debug` (analogue of python's `__repr__)`, that provides the unambigious way to output an object.

All built-in rust types already implement both traits, and for custom types it's usually very easy to auto-implement the `Debug` trait -- in most cases you can just slap `#[derive(Debug)]` on top of them and let the code generation do all the work (that is, assuming all your field types also implement `Debug`). `Display` can't be auto-implemented due to it's potentially failing nature, so that's something you have to implement manually for the new types if there's such a need. In practice this is rarely needed.

## Debug vs Display formatting

Rust syntax allows you to explicitly select the trait to be used when converting something to a text form:

1. `Display` formatting is defined as `{}`. For example, you can do `println!("Hello {}", "world");` to use built-in `Display` for the string value `"world"`
2. `Debug` formatting is defined as `{:?}`. For example, the same string value can also be printed using `Debug` by doing `println!("Hello {:?}", "world");`. In this case result will be the same because it's a string, but hopefully you get the idea.

## Pretty debug formatting

I love that this is a built-in feature that requires no additional imports (`pprint`, I'm looking at you). 

Use the format `{:#?}` to signify that your value should be pretty printed when using `Debug` trait.

For example, given this struct:

```rust
use uuid::Uuid;
use std::collections::HashMap;

#[derive(Debug)]
pub struct MyStruct {
    pub num_id: i64,
    pub values: HashMap<String, Uuid>
}

fn main() {
    let example = MyStruct {
        num_id: 42,
        values: HashMap::from_iter(vec![("type1".to_string(), Uuid::new_v4()),("type2".to_string(), Uuid::new_v4())])
    };
    println!("MyStruct's Debug representation: {:?}\n", example);
    println!("MyStruct's pretty Debug representation: {:#?}", example);
}
```

We'll see the following output:

```rust
MyStruct's Debug representation: MyStruct { num_id: 42, values: {"type1": 99f39a6e-1324-4c1b-8b0b-3a85c01232b4, "type2": 86a7534d-c247-4afa-b56c-59a516ffdb57} }

MyStruct's pretty Debug representation: MyStruct {
    num_id: 42,
    values: {
        "type1": 99f39a6e-1324-4c1b-8b0b-3a85c01232b4,
        "type2": 86a7534d-c247-4afa-b56c-59a516ffdb57,
    },
}
```

