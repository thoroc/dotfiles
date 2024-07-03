# Chezmoi dotfiles

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->

- [Prerequisite](#prerequisite)
- [Setup](#setup)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

This repository uses [chezmoi](https://www.chezmoi.io/)

**target**: MacOS with webdev emphasis

Inspiration:

- <https://github.com/politician/dotfiles/>
- <https://github.com/federicober/dotfiles>
- <https://github.com/m0lson84/dotfiles>
- <https://macos-defaults.com/>

## Prerequisite

install the following if not already present on the machine

- xcode: `xcode-select --install`
- git
- brew

## Setup

Run:

```sh
sh -c "$(curl -fsLS get.chezmoi.io)" -- init --apply thoroc
```
