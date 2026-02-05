# scriptura

A fish version of [ianthehenry/sd](https://github.com/ianthehenry/sd). It provides a convenient way to organise and execute your shell scripts from anywhere in your terminal.

Just like the original `sd`, you can organise your scripts in a logical directory hierarchy, and then execute them with a much shorter command.

## Differences from original sd

This fish version has a few differences from the original `sd`:

- Uses subcommands instead of flags (`scriptura new` vs `sd -new`)
- `--` can be used to separate arguments intended for `scriptura` itself from those intended for the executed command
- [Choose your own short alias](#shorter-command-name)
- A new `cmd` subcommand for running external commands on scripts (e.g. `scriptura cmd foobar -- rm`)
- [Add your own subcommands](#custom-subcommands) by placing them under `$SCRIPTURA_ROOT/scriptura`

## Motivation

The command name `sd` conflicts with [chmln/sd](https://github.com/chmln/sd) (a nicer version of `sed`), which I've been using for quite some time.

`sd` doesn't include completions for fish and the linked gist seems to have been removed.

I also thought that a fish implementation would be cleaner. 

## Installation

Currently, installation via Nix is the only known method. Installation via a fish plugin manager will probably work, but is untested. Feedback on this would be appreciated.

### Using Nix

If you're using Nix, `scriptura` is available as a flake, intended for use with home-manager:

```nix
inputs.scriptura.url = "git+https://codeberg.org/alinnow/scriptura";
```

Add the overlay:

```nix
overlays = [
  scriptura.overlays.default
]
```

And then configure the scriptura module:

```nix
programs.scriptura = {
  enable = true;
  package = pkgs.scriptura;
  settings = {
    ROOT = "${config.home.homeDirectory}/.sd";
    EDITOR = "nvim";
    CAT = "lolcat";
  };
};
```

The `settings` attribute accepts environment variable suffixes (e.g., `ROOT` becomes `SCRIPTURA_ROOT`). For a list of available options, see the [configuration documentation](#configuration).

### Shorter command name

Who wants to type out the name `scriptura` every time? Not me. 

I find `sf` (*s*cript *f*older) to be nice to type and it doesn't clash with anything, but you can choose. 

```fish
alias --save sf "scriptura run"
alias --save sfm "scriptura"
```

Aliases in fish automatically inherit the completion of the parent, including subcommands. So with these aliases, `sf <tab>` will only show scripts to be run, whereas `sfm <tab>` will show all subcommands.

In home-manager you can do this with

```nix
programs.scriptura.shellAliases = {
  sf = "scriptura run";
  sfm = "scriptura";
};
```

(This is just another way to set `home.shellAliases`).

## Usage

The default behaviour for `scriptura run foo bar` is:

- Assume scripts are under `~/sd`.
- If `~/sd/foo` is an executable file, execute `~/sd/foo bar`.
- If `~/sd/foo/bar` is an executable file, execute it with no arguments.
- If `~/sd/foo/bar` is a directory, it prints usage information.
- If `~/sd/foo/bar` is a non-executable regular file, it just prints the file out.

### Subcommands

`scriptura` accepts the following subcommands:

- `run`: Run the script
- `help`: Print help information
- `new`: Create a new script
- `edit`: Open the script in an editor
- `cat`: Print the contents of the script
- `which`: Print the path of the script

#### Help

The `help` subcommand prints the content of a help file or comments inside a script.

For directories, the content of a file called `help` will be printed, if it exists. `scriptura help nix` would print the contents of `~/sd/nix/help`.

Given a script at `~/sd/foo/bar`, `scriptura help foo bar` will print the contents of a corresponding file with the `.help` extension, in this case `~/sd/foo/bar.help`.
If such a file does not exist, any comments in the script (excluding shebang) are printed instead.

#### Custom subcommands

You can add your own subcommands by placing them under `$SCRIPTURA_ROOT/scriptura`. Adding a tree subcommand would look like this:

```fish
scriptura new scriptura tree -- tree
```

Then you can run the new command like so (note: no `run` subcommand)

```fish
scriptura tree
```

```
.
└── scriptura
    └── tree

2 directories, 1 file
```

Flags and arguments can be passed to the subcommand by adding them after the subcommand name: `scriptura tree -L1 foo`

## Configuration

The following environment variables can be used to configure `scriptura`:

- `SCRIPTURA_ROOT`: location of the script directory. Defaults to `$HOME/sd`.
- `SCRIPTURA_EDITOR`: used by `scriptura edit foo` and `scriptura new foo`. Defaults to `$VISUAL`, then `$EDITOR`, then, if neither of those are set, tries `vim`,`nano`, and finally, `vi`.
- `SCRIPTURA_CAT`: program used when printing files, in case you want to use something like `bat`. Defaults to `cat`.

If not defined, each variable will try the `SD_` equivalent before using default values.

In home-manager, use the `settings` option. Prefixes are optional (added by module; this won't overwrite `$EDITOR`)

```nix
programs.scriptura.settings = {
  ROOT = "${config.home.homeDirectory}/.sd";
  EDITOR = "nvim";
  CAT = "lolcat";
};
```

Or using full variable names:

```nix
programs.scriptura.settings = {
  SCRIPTURA_ROOT = "${config.home.homeDirectory}/.sd";
  SD_EDITOR = "nvim";  # also works for easy migration
};
```

The `SD_` prefixed variables are automatically converted to their `SCRIPTURA_` equivalents. If both prefixes are defined for the same variable, `SCRIPTURA_` takes precedence.
