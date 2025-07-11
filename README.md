# ⚡️Znap!
**Znap** is a fast, light-weight set of tools to ease the use of Zsh plugins &
Git repos and reduce your shell's startup time.

> Enjoy using this software? [Become a sponsor!](https://github.com/sponsors/marlonrichert)

## Requirements
Tested with:
* Zsh 5.8.1
* Git 2.39.1

## Installation
Put this in your `.zshrc` file (replacing `~/Repos` with wherever you want to
keep your Zsh plugins and/or Git repos):
```sh
# Download Znap, if it's not there yet.
[[ -r ~/Repos/znap/znap.zsh ]] ||
    git clone --depth 1 -- \
        https://github.com/marlonrichert/zsh-snap.git ~/Repos/znap
source ~/Repos/znap/znap.zsh  # Start Znap
```
Then restart your shell.

To uninstall, simply remove the above from your `.zshrc` file and remove Znap's repo.

Znap will automatically manage the repos found in its parent directory.  To change the directory it should manage, add
the following to your `.zshrc` file:
```sh
zstyle ':znap:*' repos-dir <path>
```

### Updating
To update Znap and all of your plugins/repos simultaneously, run
```sh
% znap pull
```

Note, that if you told Znap not to manage its parent directory (see the previous section), then it will not update
itself with this.  You will have to manually `cd` to its directory and run `git pull`.

If there are repos that you do not want to be included by `znap pull`, add the following to your `.zshrc` file:
```sh
zstyle ':znap:pull:*' exclude <repo> ...
```

To run `znap pull` on specific repos only, including ones you have set to be excluded, pass them as an arguments:
```sh
% znap pull <repo> ...
```

## Named dirs
Znap gives you handy shortcuts to your repos in the form of [named
directories](http://zsh.sourceforge.net/Doc/Release/Expansion.html#Filename-Expansion).

You can use these both on the command line:
```sh
% cd ~[zsh-snap]               # `cd` to a repo
% ls ~[asdf/asdf]/completions  # `ls` to a subdir in a repo
% rm -rf ~[zsh-users/zsh]      # Remove a repo completely.
% print ~[zsh-snap]            # Print the absolute path of a repo.
```

And in your dotfiles:
```sh
znap clone tuist/xcbeautify zsh-users/zsh-completions  # Make sure we have the repos we need.
 path=( ~[tuist/xcbeautify]/tools         $path )      # Add commands.
fpath=( ~[zsh-users/zsh-completions]/src $fpath )      # Add completions.
```

Additionally, Znap gives you a shortcut for moving to the root directory of the current repo:
```sh
% cd ~[zsh-autocomplete]/Functions/Util
% cd ~[]  # Moves you all the way up to ~[zsh-autocomplete].
```

## `.zshrc` optimization
Using Znap to optimize your Zsh config can be as simple as this:
```sh
[[ -r ~/Repos/znap/znap.zsh ]] ||
    git clone --depth 1 -- https://github.com/marlonrichert/zsh-snap.git ~/Repos/znap
source ~/Repos/znap/znap.zsh

# `znap prompt` makes your prompt visible in just 15-40ms!
znap prompt sindresorhus/pure

# `znap source` starts plugins.
znap source marlonrichert/zsh-autocomplete

# `znap eval` makes evaluating generated command output up to 10 times faster.
znap eval iterm2 'curl -fsSL https://iterm2.com/shell_integration/zsh'

# `znap function` lets you lazy-load features you don't always need.
znap function _pyenv pyenv "znap eval pyenv 'pyenv init - --no-rehash'"
compctl -K    _pyenv pyenv
```

For more examples of what Znap can do for your dotfiles, please see [the included `.zshrc`
file](.zshrc).

Additionaly, Znap makes it so that you actually need to have _less_ in your `.zshrc` file, by
automating several tasks for you.

### Faster `eval`
Use `znap eval ... <command>` to cache the output of `<command>`, compile it, and then `source` it (instead of `eval` it):
```sh
znap eval <name> '<command>'
```
This can be up 10 times faster than a regular `eval "$( <command> )"` statement!  If you pass a repo as the first
argument, then Znap will `eval` the command output inside the given repo and will invalidate the cache whenever the repo
is update.  Otherwise, the cache will be invalidated whenever `<command>` changes.  Caches are stored in
`${XDG_CACHE_HOME:-$HOME/.cache}/zsh-snap/eval`.

### Automatic `compinit` and `bashcompinit`
Note that the above example does not include any call to
[`complist`](http://zsh.sourceforge.net/Doc/Release/Zsh-Modules.html#The-zsh_002fcomplist-Module),
[`compinit`, or
`bashcompinit`](http://zsh.sourceforge.net/Doc/Release/Completion-System.html#Initialization) in
the `.zshrc` file. That is because Znap will run these for you as needed.

Znap also regenerates your [comp dump
file](http://zsh.sourceforge.net/Doc/Release/Completion-System.html#Use-of-compinit) automatically whenever you update a
repo, install a repo, or change your `.zshrc` file.

If necessary, you can let Znap pass arguments to `compinit` as follows:
```sh
zstyle '*:compinit' arguments -D -i -u -C -w
```

### Asynchronous compilation
Znap compiles your scripts and functions in the background. This way, your shell will start up even
faster next time!

Should you not want this feature, you can disable it with
```sh
zstyle ':znap:*' auto-compile no
```

In any case, you can compile sources manually at any time with
`znap compile [ <dir> | <file> ] ...`.

## Command-Line Usage
Znap also makes life on the command line easier.  For a full list of available commands, run
```sh
% znap
```
For more help on a particular command, run
```sh
% znap help <command>
```
Exhaustive tab completion is available, too.  For examples of the most important command-line features, see below.

> Note:
> * The examples below you should run on the command line, not add to your `.zshrc` file!
> * `%` represents the prompt.  You shouldn't type that part.  🙂

### Check Git status of all repos
To check the Git status of all repos managed by Znap, run
```sh
% znap status
```

If there are repos that you do not want to be included by `znap status`, add the following to your `.zshrc` file:
```sh
zstyle ':znap:status:*' exclude <repo> ...
```

To run `znap status` on specific repos only, including ones you have set to be excluded, pass them as an arguments:
```sh
% znap status <repo> ...
```

### Install generated functions
Some commands generate output that should be loaded as a function.  You can install these generated functions with
`znap fpath <function> '<command>'`.  For example:
```sh
% znap fpath _kubectl 'kubectl completion  zsh'
% znap fpath _rustup  'rustup  completions zsh'
% znap fpath _cargo   'rustup  completions zsh cargo'
```

This will save them to `${XDG_DATA_HOME:-$HOME/.local/share}/zsh/site-functions`.

## Author
© 2020-2021 [Marlon Richert](https://github.com/marlonrichert)

## License
This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.
