# yisangsay-rs

Yisangsay is a CLI program like cowsay, but instead of a talking cow, it's Yi Sang from Limbus Company!

## Preview

Preview of Yi Sang animation.

![Preview](./yi-sang-limbus-company.gif)

## Features

- Written in Rust!
- Talking ASCII art of Yi Sang
- Animated ASCII art of Yi Sang
- Freestyle changing animation of Yi Sang

## Installation

### Cargo

```sh
cargo install yisangsay
```

### Nix

Quick usage (one-off)

```sh
nix run github:VectorSophie/yisangsay-rs -- say "To be is to change."
```

Install with flake

```nix
{
  inputs = {
    yisang-say.url  = "github:VectorSophie/yisangsay-rs";
  };
}
```


```nix
{ pkgs, ... }: {
  home.packages = [ inputs.yisang-say.packages.${pkgs.system}.yisangSay ];
}
```

### Manually Build

```sh
git clone https://github.com/VectorSophie/yisangsay-rs.git
cd yisangsay-rs
cargo build -r
```

## Usage

```
Usage: yisangsay <COMMAND>

Commands:
  say        Display Yi Sang saying the provided text
  animate    Display an animated Yi Sang (variant 1 or 2)
  freestyle  Display Yi Sang in freestyle mode. Pretty cool for ricing btw
  help       Print this message or the help of the given subcommand(s)

Options:
  -h, --help     Print help
  -V, --version  Print version
```