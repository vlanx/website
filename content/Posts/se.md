---
title: I wish I could bring my aliases and scripts with me when I ssh...
description: Carry over your local aliases and scripts that you're accustomed to to any remote host via ssh.
tags:
  - ssh
  - zsh
  - devops
aliases:
  - /Posts/se
---

I can though, and you can too if you don't mind a tiny wrapper function around your `ssh` command.

The way it works is: 

- Have a `.serc` file with your favorite aliases and scripts
- Before `ssh`, base64 encode it to get rid of any formatting issues.
- Pass the b64 decoded config to bash via `--rcfile`, using process substitution <(...) so bash reads it as a path that only exists in memory, making sure no file gets left behind on the remote.
- You're in.

> It needs to have `bash` installed on the remote but the login shell can be anything.

This comes about because I was looking at this QoL improvement and wasn't finding anything to my liking on the web. My hard requirement was:

- Not bring any files whatsoever and have almost no overhead.

There is [xxh](https://github.com/xxh/xxh) but that brings your whole shell to whatever remote machine you `ssh` into. I didn't want that compromise, even though the only cost is the very first connection, it felt like a bad crutch to move and install a custom shell on remote environments. [sshrc](https://github.com/cdown/sshrc) is abandoned (though one could argue feature-complete) and it requires also bringing files around permanently; however, `sshrc` is a good alternative and a fair compromise all things considered and I might try it more thoroughly in the future.

### Wrapper

Really simple as you can see. Throw it in your `.zshrc` or `.bashrc` and then just `se HOST`. 

```bash
# Your portable shell config lives in ~/.serc (aliases, exports, functions).
# Requires bash to exist on the remote; the remote's login shell can be anything.
se() {
  local b64
  b64=$( {
    cat ~/.serc 2>/dev/null
  } | base64 | tr -d '\n' )
  ssh -t "$@" "bash -c 'exec bash --rcfile <(echo $b64 | base64 -d) -i'"
}

compdef se=ssh   # reuse ssh's host completion (needs compinit loaded; put this after it)
```

The last line (`compdef`) is `zsh` specific and is there so that we have the same tab completion as we'd have by using standalone `ssh`.

Be mindful to add the following two `source` lines to the `.serc` file:

```bash
source /etc/profile
source ~/.bashrc
# Your personal aliases and others
export EDITOR='vi'
alias k=kubectl
alias 'p'="python3"
alias 'v'="vim"
alias 'de'="deactivate"
alias 'c'="clear"
alias 'lh'="ls -ahl"
alias df='df -h'
alias diff="diff --color -u"
alias sr="source .venv/bin/activate"
alias sht="history | grep "
alias rr='sudo $(fc -ln -1)' #rerun last command as sudo
alias 'dc'='docker-compose'
alias 'dk'='docker'
alias nd='vim -d' # start a vim diff session
alias ss='ss -nltup' #
...
```

The first two lines are what make sure we don't completely override the host's `.bashrc` and run into PATH and other problems.

### Test results

> TL;DR: There's a `~14ms` of added overhead which is negligible and not noticeable. In fact, it's almost the same value as the standard deviation (σ).

Since I wanted to make sure I wasn't adding any meaningful overhead, I used [hyperfine](https://github.com/sharkdp/hyperfine) to measure my wrapper against the default `ssh` connection.

Both scripts were tested in a "warm" state, that is, the `ssh` connection control path was reused for every subsequent connection to avoid the key exchange and authentication.

`.ssh/config`: 

```bash
ControlMaster     auto
ControlPath       ~/.ssh/control-%C
ControlPersist    yes
```

> Helper script for the wrapper. It's basically the wrapper we use anyway, we just add the `-ic exit` so that the connection terminates and doesn't hang.

```bash
#!/usr/bin/env bash
b64=$(cat ~/.serc 2>/dev/null | base64 | tr -d '\n')
ssh -t "$1" "bash -c 'exec bash --rcfile <(echo $b64 | base64 -d) -ic exit'"
```

##### Testing: 

```bash
hyperfine --warmup 3 --runs 100 --shell=none \
  'ssh -t muttley exit' \
  '/tmp/se-bench muttley'
```


The `--warmup 3` makes sure we've setup the connection before the timing of the tests so that the session we created is being reused on the actual tests, resulting in a "warm" state.

The `muttley` host is one that is on my job's infra and I'm connecting to it via my home network and through an OpenVPN connection; fairly standard.

##### Results:

![Results](assets/se.png)

You can see that it's barely noticeable. The extra System time is the cost of the extra process and pipe the wrapper sets up
Very small price to pay in my opinion.
