# Patrician 3 Insights

This repository contains the source files for the [Patrician 3 Insights](https://p3modding.github.io) book.

## Build instructions

To build this book locally, we assume you already have [Rust and Cargo](https://www.rust-lang.org/tools/install) installed on your system.

Next you need to install `mdbook` via `cargo`:
```bash
cargo install mdbook
```

Building the book:
```bash
mdbook build
```

Serving the book locally:
```bash
mdbook serve
```
Then open [http://localhost:3000](http://localhost:3000) in your browser. The server watches for changes and rebuilds automatically.
