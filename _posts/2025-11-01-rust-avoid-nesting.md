## Rust: avoiding nested code

There are some ways to write code in rust that would be correct and compliliable, but they would also make the code a bit harder to read and/or work with. This post contains some tips on how to avoid that.

### Options

This is a relatively common move I see in some rust code written by people in the beginning of their language learning journey. `Option` type does offer explicit methods for checking its inner value, `is_some()` and `is_none()`, but those are usually reserved for cases where you want to check the state of this value without having to use it.

When you do actually need to use the optional value, instead of using these checks like this:

```
if my_optional_value.is_some() {
    do_something(my_optional_value.unwrap())
}

```

you can do this:

```
if let Some(val) = my_optional_value {
    do_something(val)
}
```

And you can make it even shorter by doing this if you don't care about doing anything when the value is `None`:

```
let Some(val) = my_optional_value;
```


### Results and errors: using syntax sugar

Let's consider this example function that returns us a `Result` value:

```
fn get_user_data() -> Result<String, std::io::Error> {
    std::fs::read_to_string("user.json")
}

fn handler() -> Result<(), std::io::Error> {
    // TODO: handle result of get_user_data
    Ok(())
}
```

When we want to do anything with this result, we'll need to handle both `Ok(String)` and `Err(std::io::Error)` states that it has.

The most verbose way to do this looks like this. It allows us to clearly handle both cases and provides visual separation in the code for two different branches:

```
fn handler() -> Result<(), std::io::Error> {
    match get_user_data() {
        Ok(data) => {
            do_something(data);
        }
        Err(e) => {
            log::error!("Error reading user data: {}", e);
        }
    }

    Ok(())
}
```

Sometimes the code will be structured such that handling an `Ok()` value takes much longer than handling an error, and 



However, since our outer function `handler` also returns a `Result`, we could use the built-in syntax sugar `?` for propagating the errors. This will mean that an error happening in `get_user_data` will bubble up and become an error returned by the `handler` function:

```
fn handler() -> Result<(), std::io::Error> {
    let data = get_user_data()?;
    do_something(data);

    Ok(())
}
```

This does allow us to shorten the code snippet, but by refactoring this code we lost the bit where we log the error that happened: `log::error!("Error reading user data: {}", e)`.

To fix this, we can use the [`inspect_err` method](https://doc.rust-lang.org/std/result/enum.Result.html) to add context to our error before it bubbles up from the `handler` function. Note that call to `inspect_err` happens before the `?` operator: the method will be called on any error first, and only then will it propagate further:

```
fn handler() -> Result<(), std::io::Error> {
    let data = get_user_data()
        .inspect_err(|e| log::error!("Error reading user data: {}", e))?;
    do_something(data);

    Ok(())
}
```

### Results and errors: converting the types

The example above is great, but in practice you will often have functions that have different error types, and you will actually get compliation errors if you try to propagate one error type from a function that returns another.

A "proper" solution to that would be ensuring that error types can be converted into each other. But! Rust typing system only allows you to do `impl` blocks for types that you own (as in, types that you have defined in your codebase), so you won't be able to do that as easily if both of your error types come from external sources (like other crates, etc).

For cases like this, you can make a small change to the code above to fix the problem: use `map_err` instead of `inspect_err`. This would allow you to write any error handling logic you want and then manually convert your original error into the one you'll be propagating.

```
fn handler() -> Result<(), serde_json::Error> {
     let data = get_user_data().map_err(|e| {
        log::error!("Error reading user data: {}", e);
        serde_json::Error::custom(format!("Error reading user data: {}", e))
    })?;
    do_something(data);

    Ok(())
}
```