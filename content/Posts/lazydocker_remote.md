---
title: Using lazydocker with remote Docker contexts.
---

I'm an immense fan of `lazygit` and I recommend you to [check it out](https://github.com/jesseduffield/lazygit). The very same author that did that wonderful tool also created `lazydocker`, which in the same vein as `lazygit`, provides a useful TUI to manage and observe more ergonomically your Docker containers and deployments.

This works out of the box for your local Docker installation and context, however, I don't usually use my own Docker context and I rely almost entirely on remote contexts.

## What are remote contexts? 

In Docker, a **context** is a named configuration that tells the CLI which Docker daemon to talk to.

By default, you’re using the `default` context, which points to your local machine. A **remote context** is just one that points to a Docker daemon running somewhere else, usually accessed over SSH (at least I prefer to connect to these remote daemons via `ssh`).

For example, you might create one like this:

```bash
docker context create remote-context \
  --docker "host=ssh://ubuntu@10.255.0.1"
```

Then switch to it:

```bash
docker context use remote-context
```

> This assumes you can connect to this remote machine via `ssh`.

From that point on, commands like `docker ps` run against that remote server instead of your local machine.

The point of this is that your workflow stays the same and you just change the target environment by switching contexts. This allows you to keep code and deployment files in your personal machine and you don't need to go back and forth `ssh`ing to machines just to perform deployments.

## Problems!

There's an open [issue](https://github.com/jesseduffield/lazydocker/issues/559) and [pull request](https://github.com/jesseduffield/lazydocker/pull/577) informing the author that using `lazydocker` with remote contexts is not working properly.

Nonetheless, I bring you here what I do to circumvent this and add a tiny bit of Quality of Life to running `lazydocker` with remote contexts (spoiler: it's `ssh` autocompletion)

### The fix:

Simply put, you have to specify `DOCKER_HOST` right before starting `lazydocker`. Just doing this was not enough for me; Assigning `DOCKER_HOST=ssh://ubuntu@10.255.0.1` was still not enough to make it work, it still complained:

```bash
2026/04/04 18:56:15 tunnel ssh docker host: ssh tunneled socket never became available: context deadline exceeded
```

What fixed it was using `ssh` aliases. You just have to add an entry to your `ssh` config file under `~/.ssh/config` like so:

```sshconfig
Host remote-context
  Hostname 10.255.0.1
  User ubuntu
```

Then you can do `DOCKER_HOST=ssh://remote-context lazydocker`; This will use your aliases described in the config file.

### QoL

To make it so that I don't have to type all that, I simply wrapped it around a zsh shell function and added (via `zsh`) the `ssh` autocompletion plugin so that I could more ergonomically connect to my previously created docker contexts (for convenience, I use the same names as the `ssh` aliases in my config), getting tab completion in the process.

```bash
lzd() {
  local host_alias="$1"

  if [ -z "$host_alias" ]; then
    echo "usage: lzd <ssh-host-alias>"
    return 1
  fi

  DOCKER_HOST="ssh://$host_alias" lazydocker
}

compdef lzd=ssh
```

Now, launching `lazydocker` with a remote context is as simple as:

```bash
lzd remote-context
```

Notice that the line `compdef lzd=ssh` is what actually gives us the autocomplete engine from `ssh`.

So very simple and ergonomic now.
