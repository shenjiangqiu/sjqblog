+++
title = "clap"
description = "clap"
date= "2022-05-11"
+++

clap is a tool for generating a complete clap project.

<!-- more -->

# clap
## links
- crates: [crates](https://crates.io/crates/clap)
- doc: [doc](https://docs.rs/clap/latest/clap/)
- github: [git](https://github.com/clap-rs/clap)

## related crates:
- [clap_mangen](https://crates.io/crates/clap_mangen)
- [clap_complete](@/posts/clap_complete.md)
## example code
- basic usage
```rust
use clap::{AppSettings, Parser};

#[derive(Parser)]
#[clap(author, version, about, long_about = None)]
#[clap(args_override_self = true)]
#[clap(allow_negative_numbers = true)]
#[clap(global_setting(AppSettings::DeriveDisplayOrder))]
struct Cli {
    #[clap(long)]
    two: String,
    #[clap(long)]
    one: String,
}

fn main() {
    let cli = Cli::parse();

    println!("two: {:?}", cli.two);
    println!("one: {:?}", cli.one);
}
```
- prositional:
 ```rust
 use clap::Parser;

#[derive(Parser)]
#[clap(author, version, about, long_about = None)]
struct Cli {
    name: Option<String>,
}

fn main() {
    let cli = Cli::parse();

    println!("name: {:?}", cli.name.as_deref());
}
 ```
- subcommands:
```rust
use clap::{Parser, Subcommand};

#[derive(Parser)]
#[clap(author, version, about, long_about = None)]
#[clap(propagate_version = true)]
struct Cli {
    #[clap(subcommand)]
    command: Commands,
}

#[derive(Subcommand)]
enum Commands {
    /// Adds files to myapp
    Add { name: Option<String> },
}
```
- defaults
```rust
use clap::Parser;

#[derive(Parser)]
#[clap(author, version, about, long_about = None)]
struct Cli {
    #[clap(default_value_t = String::from("alice"))]
    name: String,
}

fn main() {
    let cli = Cli::parse();

    println!("name: {:?}", cli.name);
}
```
- enums
```rust
use clap::{ArgEnum, Parser};

#[derive(Parser)]
#[clap(author, version, about, long_about = None)]
struct Cli {
    /// What mode to run the program in
    #[clap(arg_enum)]
    mode: Mode,
}

#[derive(Copy, Clone, PartialEq, Eq, PartialOrd, Ord, ArgEnum)]
enum Mode {
    Fast,
    Slow,
}
```
- parse
```rust
use clap::Parser;

#[derive(Parser)]
#[clap(author, version, about, long_about = None)]
struct Cli {
    /// Network port to use
    #[clap(parse(try_from_str))]
    port: usize,
}
```
- validate
```rust
use std::ops::RangeInclusive;

use clap::Parser;

#[derive(Parser)]
#[clap(author, version, about, long_about = None)]
struct Cli {
    /// Network port to use
    #[clap(parse(try_from_str=port_in_range))]
    port: usize,
}

fn main() {
    let cli = Cli::parse();

    println!("PORT = {}", cli.port);
}

const PORT_RANGE: RangeInclusive<usize> = 1..=65535;

fn port_in_range(s: &str) -> Result<usize, String> {
    let port: usize = s
        .parse()
        .map_err(|_| format!("`{}` isn't a port number", s))?;
    if PORT_RANGE.contains(&port) {
        Ok(port)
    } else {
        Err(format!(
            "Port not in range {}-{}",
            PORT_RANGE.start(),
            PORT_RANGE.end()
        ))
    }
}
```
- relations
```rust
use clap::{ArgGroup, Parser};

#[derive(Parser)]
#[clap(author, version, about, long_about = None)]
#[clap(group(
            ArgGroup::new("vers")
                .required(true)
                .args(&["set-ver", "major", "minor", "patch"]),
        ))]
struct Cli {
    /// set version manually
    #[clap(long, value_name = "VER")]
    set_ver: Option<String>,

    /// auto inc major
    #[clap(long)]
    major: bool,

    /// auto inc minor
    #[clap(long)]
    minor: bool,

    /// auto inc patch
    #[clap(long)]
    patch: bool,

    /// some regular input
    #[clap(group = "input")]
    input_file: Option<String>,

    /// some special input argument
    #[clap(long, group = "input")]
    spec_in: Option<String>,

    #[clap(short, requires = "input")]
    config: Option<String>,
}

fn main() {
    let cli = Cli::parse();

    // Let's assume the old version 1.2.3
    let mut major = 1;
    let mut minor = 2;
    let mut patch = 3;

    // See if --set-ver was used to set the version manually
    let version = if let Some(ver) = cli.set_ver.as_deref() {
        ver.to_string()
    } else {
        // Increment the one requested (in a real program, we'd reset the lower numbers)
        let (maj, min, pat) = (cli.major, cli.minor, cli.patch);
        match (maj, min, pat) {
            (true, _, _) => major += 1,
            (_, true, _) => minor += 1,
            (_, _, true) => patch += 1,
            _ => unreachable!(),
        };
        format!("{}.{}.{}", major, minor, patch)
    };

    println!("Version: {}", version);

    // Check for usage of -c
    if let Some(config) = cli.config.as_deref() {
        let input = cli
            .input_file
            .as_deref()
            .unwrap_or_else(|| cli.spec_in.as_deref().unwrap());
        println!("Doing work using input {} and config {}", input, config);
    }
}
```
- custom
```rust
use clap::{CommandFactory, ErrorKind, Parser};

#[derive(Parser)]
#[clap(author, version, about, long_about = None)]
struct Cli {
    /// set version manually
    #[clap(long, value_name = "VER")]
    set_ver: Option<String>,

    /// auto inc major
    #[clap(long)]
    major: bool,

    /// auto inc minor
    #[clap(long)]
    minor: bool,

    /// auto inc patch
    #[clap(long)]
    patch: bool,

    /// some regular input
    input_file: Option<String>,

    /// some special input argument
    #[clap(long)]
    spec_in: Option<String>,

    #[clap(short)]
    config: Option<String>,
}

fn main() {
    let cli = Cli::parse();

    // Let's assume the old version 1.2.3
    let mut major = 1;
    let mut minor = 2;
    let mut patch = 3;

    // See if --set-ver was used to set the version manually
    let version = if let Some(ver) = cli.set_ver.as_deref() {
        if cli.major || cli.minor || cli.patch {
            let mut cmd = Cli::command();
            cmd.error(
                ErrorKind::ArgumentConflict,
                "Can't do relative and absolute version change",
            )
            .exit();
        }
        ver.to_string()
    } else {
        // Increment the one requested (in a real program, we'd reset the lower numbers)
        let (maj, min, pat) = (cli.major, cli.minor, cli.patch);
        match (maj, min, pat) {
            (true, false, false) => major += 1,
            (false, true, false) => minor += 1,
            (false, false, true) => patch += 1,
            _ => {
                let mut cmd = Cli::command();
                cmd.error(
                    ErrorKind::ArgumentConflict,
                    "Can only modify one version field",
                )
                .exit();
            }
        };
        format!("{}.{}.{}", major, minor, patch)
    };

    println!("Version: {}", version);

    // Check for usage of -c
    if let Some(config) = cli.config.as_deref() {
        // todo: remove `#[allow(clippy::or_fun_call)]` lint when MSRV is bumped.
        #[allow(clippy::or_fun_call)]
        let input = cli
            .input_file
            .as_deref()
            // 'or' is preferred to 'or_else' here since `Option::as_deref` is 'const'
            .or(cli.spec_in.as_deref())
            .unwrap_or_else(|| {
                let mut cmd = Cli::command();
                cmd.error(
                    ErrorKind::MissingRequiredArgument,
                    "INPUT_FILE or --spec-in is required when using --config",
                )
                .exit()
            });
        println!("Doing work using input {} and config {}", input, config);
    }
}
```
- asserts
```rust
use clap::Parser;

#[derive(Parser)]
#[clap(author, version, about, long_about = None)]
struct Cli {
    /// Network port to use
    #[clap(parse(try_from_str))]
    port: usize,
}

fn main() {
    let cli = Cli::parse();

    println!("PORT = {}", cli.port);
}

#[test]
fn verify_app() {
    use clap::CommandFactory;
    Cli::command().debug_assert()
}
```