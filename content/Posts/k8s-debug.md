# Are there no minimal Kubernetes debug pods?

Or am I just not looking in the right places? Anyway I've created my own.

Obviously I stumbled into [netshoot](https://github.com/nicolaka/netshoot), [network-multitool](https://github.com/Praqma/Network-MultiTool)(Outdated!) which now is from [WBITT](https://github.com/wbitt/Network-MultiTool) and also [doks-debug](https://github.com/digitalocean/doks-debug) from Digital Ocean.

```bash
╭─  in  ~/Github/website/content/Posts on   main ?1 at   devenv
╰─❯ docker images

IMAGE                                             ID             DISK USAGE   CONTENT SIZE   EXTRA
alpine:3.23                                       25109184c71b       13.1MB         3.95MB
ghcr.io/digitalocean-packages/doks-debug:latest   14ca91dd9567       1.55GB          367MB    U
ghcr.io/vlanx/k8s-debug:latest                    df4ab98ee6ae       39.7MB         11.8MB
nicolaka/netshoot:latest                          47b907d662d1        874MB          213MB    U
wbitt/network-multitool:latest                    db2810fe2c8d        433MB         96.7MB
```

But just look at the size of those things. I mean, network is fast and all but I don't like bloat just because. 

Plus, it created an opportunity for me to create one tailored to my needs and made me mess with scheduled Github Actions Workflows, which I didn't even knew existed.
