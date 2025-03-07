# Use scripts to perform actions

## Understand how scripts work

chezmoi supports scripts, which are executed when you run `chezmoi apply`. The
scripts can either run every time you run `chezmoi apply`, or only when their
contents have changed.

In verbose mode, the script's contents will be printed before executing it. In
dry-run mode, the script is not executed.

Scripts are any file in the source directory with the prefix `run_`, and are
executed in alphabetical order. Scripts that should only be run if they have
not been run before have the prefix `run_once_`. Scripts that should be run
whenever their contents change have the `run_onchange_` prefix.

Scripts break chezmoi's declarative approach, and as such should be used
sparingly. Any script should be idempotent, even `run_once_` and
`run_onchange_` scripts.

Scripts are normally run while chezmoi updates your dotfiles. To configure
scripts to run before or after your dotfiles are updated use the `before_` and
`after_` attributes respectively, e.g.
`run_once_before_install-password-manager.sh`.

Scripts must be created manually in the source directory, typically by running
`chezmoi cd` and then creating a file with a `run_` prefix. There is no need to
set the executable bit on the script, as chezmoi will set the executable bit
before executing the script.

Scripts with the suffix `.tmpl` are treated as templates, with the usual
template variables available. If, after executing the template, the result is
only whitespace or an empty string, then the script is not executed. This is
useful for disabling scripts.

When chezmoi executes a script, it first generates the script contents in a
file in a temporary directory with the executable bit set, and then executes
the contents with `exec(3)`. Consequently, the script's contents must either
include a `#!` line or be an executable binary.

!!! note

    By default, `chezmoi diff` will print the contents of scripts that would be
    run by `chezmoi apply`. To exclude scripts from the output of `chezmoi
    diff`, set `diff.exclude` in your configuration file, for example:

    ```toml title="~/.config/chezmoi/chezmoi.toml"
    [diff]
        exclude = ["scripts"]
    ```

    Similarly, `chezmoi status` will print the names of the scripts that it
    will execute with the status `R`. This can similarly disabled by setting
    `status.exclude` to `["scripts"]` in your configuration file.

## Install packages with scripts

Change to the source directory and create a file called
`run_once_install-packages.sh`:

```console
$ chezmoi cd
$ $EDITOR run_once_install-packages.sh
```

In this file create your package installation script, e.g.

```sh
#!/bin/sh
sudo apt install ripgrep
```

The next time you run `chezmoi apply` or `chezmoi update` this script will be
run. As it has the `run_once_` prefix, it will not be run again unless its
contents change, for example if you add more packages to be installed.

This script can also be a template. For example, if you create
`run_once_install-packages.sh.tmpl` with the contents:

``` title="~/.local/share/chezmoi/run_once_install-packages.sh.tmpl"
{{ if eq .chezmoi.os "linux" -}}
#!/bin/sh
sudo apt install ripgrep
{{ else if eq .chezmoi.os "darwin" -}}
#!/bin/sh
brew install ripgrep
{{ end -}}
```

This will install `ripgrep` on both Debian/Ubuntu Linux systems and macOS.

## Run a script when the contents of another file changes

chezmoi's `run_` scripts are run every time you run `chezmoi apply`, whereas
`run_once_` scripts are run only when their contents have changed, after
executing them as templates. You can use this to cause a `run_once_` script to
run when the contents of another file has changed by including a checksum of
the other file's contents in the script.

For example, if your [dconf](https://wiki.gnome.org/Projects/dconf) settings
are stored in `dconf.ini` in your source directory then you can make `chezmoi
apply` only load them when the contents of `dconf.ini` has changed by adding
the following script as `run_once_dconf-load.sh.tmpl`:

``` title="~/.local/share/chezmoi/run_once_dconf-load.sh.tmpl"
#!/bin/bash

# dconf.ini hash: {{ include "dconf.ini" | sha256sum }}
dconf load / < {{ joinPath .chezmoi.sourceDir "dconf.ini" | quote }}
```

As the SHA256 sum of `dconf.ini` is included in a comment in the script, the
contents of the script will change whenever the contents of `dconf.ini` are
changed, so chezmoi will re-run the script whenever the contents of `dconf.ini`
change.

In this example you should also add `dconf.ini` to `.chezmoiignore` so chezmoi
does not create `dconf.ini` in your home directory.

## Clear the state of `run_once_` scripts

chezmoi stores whether and when `run_once_` scripts have been run in the
`scriptState` bucket of its persistent state. To clear the state of `run_once_`
scripts, run:

```console
$ chezmoi state delete-bucket --bucket=scriptState
```
