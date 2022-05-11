+++
title = "clap complete"
date= "2022-05-11"
description = "clap complete"
[taxonomies]
categories = ["programming notes"]
tags = ["clap", "rust","command line utilities"]
+++

a tool for generating a complete clap project

<!-- more -->

# clap complete
## links
- [clap_complete](https://crates.io/crates/clap_complete)
- [zsh completion function]( @/posts/zsh_comp.md )

## example

- args:
```rust
use std::path::PathBuf;

use clap::{Parser, ValueHint};
use clap_complete::Shell;

#[derive(Parser, Debug)]
#[clap(author="Jiangqiu Shen",version,about="a spmm pim simulator",long_about=None,trailing_var_arg=true)]
pub struct Args {
    /// Generate completion for the given shell
    #[clap(long = "generate", short = 'g', arg_enum)]
    pub generator: Option<Shell>,

    /// the path of config file, default is "default.toml"
    #[clap(parse(from_os_str),value_hint=ValueHint::FilePath)]
    pub config_file: Vec<PathBuf>,
}

```

- main
```rust
use clap::{Command, IntoApp, Parser};
use clap_complete::Generator;
fn main() -> Result<()>{
    if let Some(generator) = args.generator {
        let mut cmd = Args::command();
        eprintln!("Generating completion file for {:?}...", generator);
        print_completions(generator, &mut cmd);
        return Ok(());
    }
}

fn print_completions<G: Generator>(gen: G, cmd: &mut Command) {
    clap_complete::generate(gen, cmd, cmd.get_name().to_string(), &mut io::stdout());
}
```