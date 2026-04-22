# Shell Zsh Sync

This document records the effective shell configuration that was applied on the server when switching all login users to `zsh` and keeping `bash` only as a fallback that immediately jumps into `zsh`.

## Effective State

- Login-capable users:
  - `root` -> `/usr/bin/zsh`
  - `admin` -> `/usr/bin/zsh`
- New user default shell:
  - `/etc/default/useradd`: `SHELL=/usr/bin/zsh`
  - `/etc/adduser.conf`: `DSHELL=/usr/bin/zsh`
- Interactive `bash` fallback:
  - `root`, `admin`, and `/etc/skel/.bashrc` all contain the same early jump block:

```bash
if [ -z "${ZSH_VERSION-}" ] && command -v zsh >/dev/null 2>&1; then
    exec zsh -l
fi
```

## root

### `~/.bashrc`

```bash
# ~/.bashrc: executed by bash(1) for non-login shells.
# see /usr/share/doc/bash/examples/startup-files (in the package bash-doc)
# for examples

# If not running interactively, don't do anything
case $- in
    *i*) ;;
      *) return;;
esac

if [ -z "${ZSH_VERSION-}" ] && command -v zsh >/dev/null 2>&1; then
    exec zsh -l
fi

# don't put duplicate lines or lines starting with space in the history.
# See bash(1) for more options
HISTCONTROL=ignoreboth

# append to the history file, don't overwrite it
shopt -s histappend

# for setting history length see HISTSIZE and HISTFILESIZE in bash(1)
HISTSIZE=1000
HISTFILESIZE=2000

# check the window size after each command and, if necessary,
# update the values of LINES and COLUMNS.
shopt -s checkwinsize

# If set, the pattern "**" used in a pathname expansion context will
# match all files and zero or more directories and subdirectories.
#shopt -s globstar

# make less more friendly for non-text input files, see lesspipe(1)
[ -x /usr/bin/lesspipe ] && eval "$(SHELL=/bin/sh lesspipe)"

# set variable identifying the chroot you work in (used in the prompt below)
if [ -z "${debian_chroot:-}" ] && [ -r /etc/debian_chroot ]; then
    debian_chroot=$(cat /etc/debian_chroot)
fi

# set a fancy prompt (non-color, unless we know we "want" color)
case "$TERM" in
    xterm-color|*-256color) color_prompt=yes;;
esac

# uncomment for a colored prompt, if the terminal has the capability; turned
# off by default to not distract the user: the focus in a terminal window
# should be on the output of commands, not on the prompt
#force_color_prompt=yes

if [ -n "$force_color_prompt" ]; then
    if [ -x /usr/bin/tput ] && tput setaf 1 >&/dev/null; then
        color_prompt=yes
    else
        color_prompt=
    fi
fi

if [ "$color_prompt" = yes ]; then
    PS1='${debian_chroot:+($debian_chroot)}\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w\[\033[00m\]\$ '
else
    PS1='${debian_chroot:+($debian_chroot)}\u@\h:\w\$ '
fi
unset color_prompt force_color_prompt

# If this is an xterm set the title to user@host:dir
case "$TERM" in
xterm*|rxvt*)
    PS1="\[\e]0;${debian_chroot:+($debian_chroot)}\u@\h: \w\a\]$PS1"
    ;;
*)
    ;;
esac

# enable color support of ls and also add handy aliases
if [ -x /usr/bin/dircolors ]; then
    test -r ~/.dircolors && eval "$(dircolors -b ~/.dircolors)" || eval "$(dircolors -b)"
    alias ls='ls --color=auto'
    #alias dir='dir --color=auto'
    #alias vdir='vdir --color=auto'

    alias grep='grep --color=auto'
    alias fgrep='fgrep --color=auto'
    alias egrep='egrep --color=auto'
fi

# some more ls aliases
alias ll='ls -alF'
alias la='ls -A'
alias l='ls -CF'

# Alias definitions.
if [ -f ~/.bash_aliases ]; then
    . ~/.bash_aliases
fi

# enable programmable completion features
if ! shopt -oq posix; then
  if [ -f /usr/share/bash-completion/bash_completion ]; then
    . /usr/share/bash-completion/bash_completion
  elif [ -f /etc/bash_completion ]; then
    . /etc/bash_completion
  fi
fi

[ -x "$HOME/.local/bin/starship" ] && eval "$("$HOME/.local/bin/starship" init bash)"
```

### `~/.bash_aliases`

```bash
alias mosh='mosh --no-init'
alias zj='zellij'
alias zl='zellij ls'
alias zk='zellij kill-all-sessions'
alias zd='zellij delete-all-sessions -y'

za() {
    local target sessions

    if [ $# -gt 0 ]; then
        command zellij attach -c "$1"
        return
    fi

    sessions="$(
        command zellij ls 2>/dev/null |
            sed -r 's/\x1B\[[0-9;]*[mK]//g' |
            awk 'NF { print $1 }'
    )"

    if [ -z "$sessions" ]; then
        command zellij attach -c main
        return
    fi

    target="$(
        printf '%s\n' "$sessions" |
            fzf --prompt='zellij session> ' --height=40% --reverse --border --exit-0
    )"

    [ -n "$target" ] && command zellij attach -c "$target"
}

zks() {
    command zellij kill-session "$1"
}
```

### `~/.zshrc`

```zsh
# ~/.zshrc

[[ -o interactive ]] || return

HISTFILE=$HOME/.zhistory
HISTSIZE=1000
SAVEHIST=2000

setopt APPEND_HISTORY
setopt HIST_IGNORE_DUPS
setopt HIST_IGNORE_SPACE
setopt SHARE_HISTORY
setopt AUTO_CD

autoload -Uz compinit
compinit

if [[ -x /usr/bin/dircolors ]]; then
  eval "$(/usr/bin/dircolors -b)"
  alias ls='ls --color=auto'
  alias grep='grep --color=auto'
  alias fgrep='fgrep --color=auto'
  alias egrep='egrep --color=auto'
fi

alias ll='ls -alF'
alias la='ls -A'
alias l='ls -CF'

[[ -f "$HOME/.bash_aliases" ]] && source "$HOME/.bash_aliases"
[[ -x /usr/local/bin/starship ]] && eval "$(/usr/local/bin/starship init zsh)"
```

### `~/.zprofile`

```zsh
# ~/.zprofile

[[ -d "$HOME/bin" ]] && export PATH="$HOME/bin:$PATH"
[[ -d "$HOME/.local/bin" ]] && export PATH="$HOME/.local/bin:$PATH"
```

## admin

### `~/.bash_aliases`

```bash
alias mosh='mosh --no-init'
alias zj='zellij'
alias zl='zellij ls'
alias zk='zellij kill-all-sessions'
alias zd='zellij delete-all-sessions -y'

za() {
    local target sessions

    if [ $# -gt 0 ]; then
        command zellij attach -c "$1"
        return
    fi

    sessions="$(
        command zellij ls 2>/dev/null |
            sed -r 's/\x1B\[[0-9;]*[mK]//g' |
            awk 'NF { print $1 }'
    )"

    if [ -z "$sessions" ]; then
        command zellij attach -c main
        return
    fi

    target="$(
        printf '%s\n' "$sessions" |
            fzf --prompt='zellij session> ' --height=40% --reverse --border --exit-0
    )"

    [ -n "$target" ] && command zellij attach -c "$target"
}

zks() {
    command zellij kill-session "$1"
}
```

### `~/.zshrc`

```zsh
# ~/.zshrc

[[ -o interactive ]] || return

HISTFILE=$HOME/.zhistory
HISTSIZE=1000
SAVEHIST=2000

setopt APPEND_HISTORY
setopt HIST_IGNORE_DUPS
setopt HIST_IGNORE_SPACE
setopt SHARE_HISTORY
setopt AUTO_CD

autoload -Uz compinit
compinit

if [[ -x /usr/bin/dircolors ]]; then
  eval "$(/usr/bin/dircolors -b)"
  alias ls='ls --color=auto'
  alias grep='grep --color=auto'
  alias fgrep='fgrep --color=auto'
  alias egrep='egrep --color=auto'
fi

alias ll='ls -alF'
alias la='ls -A'
alias l='ls -CF'

[[ -f "$HOME/.bash_aliases" ]] && source "$HOME/.bash_aliases"
[[ -x /usr/local/bin/starship ]] && eval "$(/usr/local/bin/starship init zsh)"
```

### `~/.zprofile`

```zsh
# ~/.zprofile

[[ -d "$HOME/bin" ]] && export PATH="$HOME/bin:$PATH"
[[ -d "$HOME/.local/bin" ]] && export PATH="$HOME/.local/bin:$PATH"
```

## System Defaults

### `/etc/default/useradd`

```ini
SHELL=/usr/bin/zsh
```

### `/etc/adduser.conf`

```ini
DSHELL=/usr/bin/zsh
```
