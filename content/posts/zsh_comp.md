+++
title = "zsh comp"
date= "2022-05-11"
[taxonomies]
categories = ["programming notes"]
tags = ["clap", "rust","command line utilities"]
+++


# introduction to add completion to zsh

<!-- more -->
1. add a path for your completion scriptes: `mkdir ~/.zfunc`
2. generate the file for command `your_command`: `your_command -g=zsh >> ~/.zfunc/_your_command`
3. restart your shell and try

