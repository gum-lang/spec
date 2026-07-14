![acronym](./assets/acronym.png)

`gum` is a markup language for configuring software inspired by `TOML`, `Nix`, and `JSON`. Take a look at the [specification](./SPEC.md) to see how it works, or head over the the [website]() for broader documentation.

## Usage

There are a number of standardized [parsers]() for whatever language you're into at the moment:

- [Python](https://github.com/gum-lang/python)
- ~~[TypeScript]()~~
- ~~[Java]()~~
- ~~[Kotlin]()~~
- ~~[C#]()~~
- ~~[Go]()~~
- ~~[Rust]()~~
- ~~[Odin]()~~
- ~~[C]()~~
- ~~[Zig]()~~

There is also a [CLI]() for validation, formatting, and conversion, and an [LSP]() with several editor plugins available:

- ~~[VSCode]()~~
- ~~[Neovim]()~~
- ~~[Zed]()~~

## Interface

APIs are standardized to mirror Go's [encoding/json](https://pkg.go.dev/encoding/json) package.

To turn data into `gum`:

```psuedo
fn Marshal(GUMData) -> (string, error)
```

To turn `gum` into data:

```psuedo
fn Unmarshal(string, GUMData) -> (error)
```

The exact API of a given parser will change as best fits a given platform/language, but at minimum you should expect them to implement the above two functions.

## License

[Apache-2.0](./LICENSE)

![elaine](./assets/elaine.png)
