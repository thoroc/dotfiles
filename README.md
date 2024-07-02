# Chezmoi dotfiles

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->

- [Prerequisite](#prerequisite)
- [Setup](#setup)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

**target**: MacOS with webdev emphasis

Inspiration:

- <https://github.com/politician/dotfiles/>
- <https://github.com/federicober/dotfiles>

## Prerequisite

install the following if not already present on the machine

- xcode: `xcode-select --install`
- git
- brew
- run `brew bundle install --file=~/.local/share/chezmoi/Brewfile`

## Setup

Run `chezmoi init`. That should prompt you to inform all the variables in `chezmoi.toml.tmpl` and save a new config file
under: `~/.config/chezmoi.toml`.
