# webview2-rs
Rust bindings for the WebView2 COM APIs

## Crates in this repo
The root of this repo defines a virtual workspace in [Cargo.toml](./Cargo.toml) which includes three crates:
- [webview2-com](./crates/webview2-com/README.md): The main crate, which depends on `webview2-com-macros` and `webview2-com-sys`.
- [webview2-com-macros](./crates/callback-macros/README.md): Macros used to build boilerplate implementations of all the `ICoreWebView2...Handler` interfaces declared in `WebView2.h`.
- [webview2-com-sys](./crates/bindings/README.md): Unsafe bindings built with the [Windows](https://github.com/microsoft/windows-rs) crate.
- [update-bindings](./crates/update-bindings/README.md): Utility to regenerate the bindings in `webview2-com-sys` from a new `winmd` file.

## Windows Metadata
The Windows crate requires a Windows Metadata (`winmd`) file describing the API. The one used in this repo was generated with the [webview2-win32md](https://github.com/wravery/webview2-win32md) project.

## Getting started
Run all the tests from the root of the repo:
```cmd
> cargo test
```

Or run the sample app from anywhere in the repo:
```cmd
> cargo run --example sample
```

See the [README.md](./crates/webview2-com/README.md) in `webview2-com` for more details about using that crate in your own project.

## Cross-compilation
The [update-bindings](./crates/update-bindings/src/main.rs) tool automatically downloads and extracts the NuGet package for the SDK and links against the libraries from that package. If you build on a Windows machine, you probably won't need to do anything special to enable this, but if you are building on a Linux or macOS machine, you need to have [mono](https://www.mono-project.com/) installed and on your `$PATH` to execute the `nuget.exe` CLI tool.

By default this crate uses the `WebView2LoaderStatic.lib` static library and does not need to redistribute `WebView2Loader.dll`. However, this doesn't work with the `*-pc-windows-gnu` targets instead of `*-pc-windows-msvc` (for example, when cross-compiling). For the `gnu` targets, it will link against the import lib for `WebView2Loader.dll` instead, and you will need to place this DLL next to your executable or in your `$PATH` so it can find the DLL at runtime.

## Updating the WebView2 SDK
You can tell the `update-bindings` tool to use a different version by updating `WEBVIEW2_VERSION` in `main.rs`:
```rust
    const WEBVIEW2_VERSION: &str = "1.0.2045.28";
```
It will also regenerate [callback_interfaces.rs](./crates/bindings/src/callback_interfaces.rs) if they change in a new version. This file is used in `webview2-com`, and in particular, the tests in [callback.rs](./crates/webview2-com/src/callback.rs) verify that all of the interfaces listed in `callback_interfaces.rs` are implemented. If a new version of the SDK declared additional callback interfaces, you will need to add those interfaces to `callback.rs` using the `#[completed_callback]` (for `ICoreWebView2...CompletedHandler` interfaces) and `#[event_callback]` (for `ICoreWebView2...EventHandler` interfaces) macros.

It does not regenerate the `winmd` file automatically because that would depend on having the `dotnet` CLI installed. New versions of the SDK should be backwards compatible, but you may want to regenerate the `Microsoft.Web.WebView2.Win32.winmd` file using `webview2-win32md` if you need functionality which was added in a new version. You should then copy the file to `./crates/bindings/winmd/Microsoft.Web.WebView2.Win32.winmd`, which is where the bindings build script looks for it.
