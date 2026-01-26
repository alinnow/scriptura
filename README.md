# scriptura

A fish version of [ianthehenry/sd](https://github.com/ianthehenry/sd). It provides a convenient way to organise and execute your shell scripts from anywhere in your terminal.

Just like the original `sd`, you can organise your scripts in a logical directory hierarchy, and then execute them with a much shorter command.

## Differences from original sd

This fish version has a few differences from the original `sd`:

- Flags for `scriptura` must be placed before any arguments, unlike the original where flags could come after arguments
- `--really` flag has been removed (the above constraint makes it no longer necessary)
- `--` can be used to separate arguments intended for `scriptura` itself from those intended for the executed command
- [Choose your own short alias](#shorter-command-name)

## Motivation

The command name `sd` conflicts with [chmln/sd](https://github.com/chmln/sd) (a nicer version of `sed`), which I've been using for quite some time.

`sd` doesn't include completions for fish and the linked gist seems to have been removed.

I also thought that a fish implementation would be cleaner.

## Installation

### Using Nix

If you're using Nix, `scriptura` is available as a flake, intended for use with home-manager and the script-directory module:

```nix
inputs.scriptura.url = "git+https://codeberg.org/alinnow/scriptura";
```

Add the overlay:

```nix
overlays = [
  scriptura.overlays.default
]
```

And then configure the script-directory module:

```nix
programs.script-directory = {
  enable = true;
  package = pkgs.scriptura
  settings = {
    SD_ROOT = "${config.home.homeDirectory}/.sd";
    SD_EDITOR = "nvim";
    SD_CAT = "lolcat"; 
  };
};
```

Any `SD_*` variables will be respected as long as their `SCRIPTURA_*` counterpart is not set.

### Manual Installation

1. Clone this repository:
   ```fish
   git clone https://codeberg.org/alinnow/scriptura
   ```

2. Add the function to your fish functions directory:
   ```fish
   ln -s $PWD/functions/scriptura.fish ~/.config/fish/functions/scriptura.fish
   ```

3. Add the completion to your fish completions directory:
   ```fish
   ln -s $PWD/completions/scriptura.fish ~/.config/fish/completions/scriptura.fish
   ```

### Shorter command name

Who wants to type out the name `scriptura` every time? Not me. 

I find `sf` to be nice to type and it doesn't clash with anything, but you can choose. Aliases in fish inherit the completion of the parent.

```fish
alias --save sf scriptura
```

In home-manager you can do this with

```nix
home.shellAliases.sf = "scriptura";
```

## Usage

The default behaviour for `scriptura foo bar` is:
- Assume scripts are under `~/sd`.
- If `~/sd/foo` is an executable file, execute `~/sd/foo bar`.
- If `~/sd/foo/bar` is an executable file, execute it with no arguments.
- If `~/sd/foo/bar` is a directory, it prints usage information.
- If `~/sd/foo/bar` is a non-executable regular file, it just prints the file out.

### Special Flags

`scriptura` supports several special flags that override normal execution:

- `--help`: Print help information
- `--new`: Create a new script
- `--edit`: Open the script in an editor
- `--cat`: Print the contents of the script
- `--which`: Print the path of the script

## Configuration

The following environment variables can be used to configure `scriptura`:

- `SCRIPTURA_ROOT`: location of the script directory. Defaults to `$HOME/sd`.
- `SCRIPTURA_EDITOR`: used by `scriptura foo --edit` and `scriptura foo --new`. Defaults to `$VISUAL`, then `$EDITOR`, then, if neither of those are set, tries `vim`,`nano`, and finally, `vi`.
- `SCRIPTURA_CAT`: program used when printing files, in case you want to use something like `bat`. Defaults to `cat`.

If not defined, each variable will try the `SD_` equivalent before using default values.
