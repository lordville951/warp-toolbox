# warp-toolbox Guide

This document explains how to install, use, and remove the
**warp-toolbox** environment.

The toolbox provides a container with common cloud and Kubernetes tools
that run through **Cloudflare WARP**.

Tools included:

-   Azure CLI
-   kubectl
-   Helm
-   k9s
-   yq
-   git / jq

The container runs as **the same user as the host** and mounts your real
home directory.

Example:

Host: vick

Container: vick

Home directory inside container: /home/vick

------------------------------------------------------------------------

# Project Layout

warp-toolbox/ ├── compose.yaml ├── README.md ├── bin/ │ ├── up │ ├──
down │ └── toolbox ├── toolbox/ │ ├── Dockerfile │ └── entrypoint.sh ├──
config/ │ ├── azure/ │ └── kube/ ├── data/

Folders:

config/azure -- Azure CLI configuration (separate from host) config/kube
-- Kubernetes configuration (separate from host) data -- WARP runtime
state

------------------------------------------------------------------------

# Why Azure and Kubernetes configs are separated

Even though the container mounts your **real home directory**, the
configs are intentionally kept separate.

This prevents:

-   accidentally modifying host kube contexts
-   overwriting Azure CLI login sessions
-   polluting your normal development environment

Inside the container:

/home/`<user>`{=html}/.azure → ./config/azure
/home/`<user>`{=html}/.kube → ./config/kube

------------------------------------------------------------------------

# Installation

Clone the repository:

git clone `<repo>`{=html} cd warp-toolbox

Make scripts executable:

chmod +x bin/\*

------------------------------------------------------------------------

# Optional: Install global commands (symlinks)

To run the toolbox from anywhere, create global symlinks:

sudo ln -sf \~/projects/warp-toolbox/bin/up /usr/local/bin/warp-up sudo
ln -sf \~/projects/warp-toolbox/bin/toolbox /usr/local/bin/warp-toolbox
sudo ln -sf \~/projects/warp-toolbox/bin/down /usr/local/bin/warp-down

Now the commands can be used from any directory.

------------------------------------------------------------------------

# Starting the toolbox

Start the containers:

warp-up

What happens:

1.  Your host user is detected
2.  The toolbox image is built
3.  The same user is created inside the container
4.  WARP container starts
5.  Toolbox container starts

------------------------------------------------------------------------

# Opening the toolbox shell

Open an interactive shell:

warp-toolbox

Expected prompt example:

vick@container:/home/vick\$

Verify:

whoami echo \$HOME

Expected output:

vick /home/vick

------------------------------------------------------------------------

# Azure login

Inside the toolbox container run:

az login

Credentials are stored in:

config/azure/

------------------------------------------------------------------------

# Kubernetes configuration

If needed copy your kubeconfig:

cp \~/.kube/config config/kube/config

Inside the container:

kubectl get nodes

------------------------------------------------------------------------

# Stopping the environment

Stop the containers:

warp-down

------------------------------------------------------------------------

# Removing global commands

If you installed global symlinks remove them with:

sudo rm /usr/local/bin/warp-up sudo rm /usr/local/bin/warp-toolbox sudo
rm /usr/local/bin/warp-down

------------------------------------------------------------------------

# Daily workflow

Typical usage:

warp-up warp-toolbox warp-down

------------------------------------------------------------------------

# Summary

The toolbox provides:

-   identical host user inside container
-   mounted real home directory
-   isolated Azure and Kubernetes configs
-   WARP network routing
-   simple global commands

