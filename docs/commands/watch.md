# cargo lambda watch

The watch subcommand emulates the AWS Lambda control plane API. Run this command at the root of a Rust workspace and cargo-lambda will use cargo-watch to hot compile changes in your Lambda functions. Use flag `--no-reload` to avoid hot compilation.

::: warning
This command works best with the **[Lambda Runtime version 0.5.1](https://crates.io/crates/lambda_runtime/0.5.1)**. Previous versions of the runtime are likely to crash with serialization errors.
:::

```
cargo lambda watch
```

The function is not compiled until the first time that you try to execute it. See the [invoke](/commands/invoke) command to learn how to execute a function. Cargo will run the command `cargo run --bin FUNCTION_NAME` to try to compile the function. `FUNCTION_NAME` can be either the name of the package if the package has only one binary, or the binary name in the `[[bin]]` section if the package includes more than one binary.

## Environment variables

If you need to set environment variables for your function to run, you can specify them in the metadata section of your Cargo.toml file.

Use the section `package.metadata.lambda.env` to set global variables that will applied to all functions in your package:

```toml
[package]
name = "basic-lambda"

[package.metadata.lambda.env]
RUST_LOG = "debug"
MY_CUSTOM_ENV_VARIABLE = "custom value"
```

If you have more than one function in the same package, and you want to set specific variables for each one of them, you can use a section named after each one of the binaries in your package, `package.metadata.lambda.bin.BINARY_NAME`:

```toml
[package]
name = "lambda-project"

[package.metadata.lambda.env]
RUST_LOG = "debug"

[package.metadata.lambda.bin.get-product.env]
GET_PRODUCT_ENV_VARIABLE = "custom value"

[package.metadata.lambda.bin.add-product.env]
ADD_PRODUCT_ENV_VARIABLE = "custom value"

[[bin]]
name = "get-product"
path = "src/bin/get-product.rs"

[[bin]]
name = "add-product"
path = "src/bin/add-product.rs"
```

You can also set environment variables on a workspace
```toml
[workspace.metadata.lambda.env]
RUST_LOG = "debug"

[workspace.metadata.lambda.bin.get-product.env]
GET_PRODUCT_ENV_VARIABLE = "custom value"
```
These behave in the same way, package environment variables will override workspace settings, the order of precedence is:

1) Package Binary
2) Package Global
3) Workspace Binary
4) Workspace Global

## Function URLs

The emulator server includes support for [Lambda function URLs](https://docs.aws.amazon.com/lambda/latest/dg/lambda-urls.html) out of the box. Since we're working locally, these URLs are under the `/lambda-url` path instead of under a subdomain. The function that you're trying to access through a URL must respond to Request events using [lambda_http](https://crates.io/crates/lambda_http/), or raw `ApiGatewayV2httpRequest` events.

You can create functions compatible with this feature by running `cargo lambda new --http FUNCTION_NAME`.

To access a function via its HTTP endpoint, start the watch subcommand `cargo lambda watch`, then send requests to the endpoint `http://localhost:9000/lambda-url/FUNCTION_NAME`. You can add any additional path after the function name, or any query parameters.

## Extra arguments

You can pass any extra arguments to `cargo lambda watch` that `cargo watch` supports after two trailing dashes. This allows you to send extra options to the command spawned to serve request, like features flags:

```
cargo lambda watch -- --features my-own-features
```

See the available options in [the cargo-watch manual page](https://github.com/watchexec/cargo-watch/blob/main/cargo-watch.1.ronn).
