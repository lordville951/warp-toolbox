# warp-toolbox Guide

This document explains how to install, use, and remove the
**warp-toolbox** environment.

The toolbox provides a container with common cloud and Kubernetes tools
that run through **Cloudflare WARP**.

## Tools included

-   Azure CLI
-   kubectl
-   Helm
-   k9s
-   yq
-   git
-   jq

The container runs as **the same user as the host** and mounts your real
home directory.

Example:

    Host user: vick
    Container user: vick
    Home inside container: /home/vick

------------------------------------------------------------------------

## Project layout

    warp-toolbox/
    ├── compose.yaml
    ├── README.md
    ├── bin/
    │   ├── up
    │   ├── down
    │   └── toolbox
    ├── toolbox/
    │   ├── Dockerfile
    │   └── entrypoint.sh
    ├── config/
    │   ├── azure/
    │   └── kube/
    └── data/

### Folders

-   `config/azure/` --- optional repo-local Azure CLI config
-   `config/kube/` --- optional repo-local Kubernetes config
-   `data/` --- WARP runtime state

------------------------------------------------------------------------

## Important note about Azure and Kubernetes config

Even though this repository contains `config/azure/` and `config/kube/`,
the current setup does **not automatically isolate** Azure CLI and
kubeconfig from your host home directory.

In the current configuration, the container mounts your real home
directory, so these paths inside the container still point to your host
data:

    /home/<user>/.azure
    /home/<user>/.kube

That means:

-   your normal Azure CLI session may be reused
-   your normal kubeconfig may be reused
-   changes in the container can affect your host config

So unless you explicitly remap those paths in `compose.yaml`, Azure and
Kubernetes config are **not actually separated**.

------------------------------------------------------------------------

## Installation

Clone the repository:

``` bash
git clone <repo>
cd warp-toolbox
```

Make scripts executable:

``` bash
chmod +x bin/*
```

------------------------------------------------------------------------

## Optional: install global commands

To run the toolbox from anywhere, create global symlinks:

``` bash
sudo ln -sf ~/projects/warp-toolbox/bin/up /usr/local/bin/warp-up
sudo ln -sf ~/projects/warp-toolbox/bin/toolbox /usr/local/bin/warp-toolbox
sudo ln -sf ~/projects/warp-toolbox/bin/down /usr/local/bin/warp-down
```

Now the commands can be used from any directory.

------------------------------------------------------------------------

## Starting the toolbox

Start the containers:

``` bash
warp-up
```

### What happens

1.  Your host user is detected
2.  The toolbox image is built
3.  The same user is created inside the container
4.  The WARP container starts
5.  The toolbox container starts

------------------------------------------------------------------------

## Opening the toolbox shell

Open an interactive shell:

``` bash
warp-toolbox
```

Expected prompt example:

    vick@container:/home/vick$

Verify:

``` bash
whoami
echo $HOME
```

Expected output:

    vick
    /home/vick

------------------------------------------------------------------------

## Azure login

Inside the toolbox container, run:

``` bash
az login
```

If your current Compose setup uses your mounted home directory directly,
credentials are stored in your normal host Azure config:

    ~/.azure/

If you later remap Azure config to a repo-local folder, then it could
instead be stored in:

    config/azure/

------------------------------------------------------------------------

## Kubernetes configuration

If needed, verify what kubeconfig is being used:

``` bash
echo $KUBECONFIG
```

If the current setup points to your mounted home directory, the
container is likely using:

    ~/.kube/config

You can test with:

``` bash
kubectl get nodes
```

If you want real separation, update `compose.yaml` so the container uses
`./config/kube/config` instead.

------------------------------------------------------------------------

## Stopping the environment

Stop the containers:

``` bash
warp-down
```

------------------------------------------------------------------------

## Removing global commands

If you installed global symlinks, remove them with:

``` bash
sudo rm /usr/local/bin/warp-up
sudo rm /usr/local/bin/warp-toolbox
sudo rm /usr/local/bin/warp-down
```

------------------------------------------------------------------------

## Daily workflow

Typical usage:

``` bash
warp-up
warp-toolbox
warp-down
```

------------------------------------------------------------------------

## Summary

The toolbox provides:

-   the same user inside the container as on the host
-   access to your mounted home directory
-   Cloudflare WARP network routing
-   common cloud and Kubernetes tooling
-   simple helper commands

> Note: Azure and Kubernetes configs are only isolated if you explicitly
> map them that way in `compose.yaml`.

