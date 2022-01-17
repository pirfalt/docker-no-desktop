# Docker no Desktop

https://www.docker.com/legal/docker-subscription-service-agreement.

Replacing the software may be easier than complying with the service agreement.

Therefor, let's follow this guide. https://dhwaneetbhatt.com/blog/run-docker-without-docker-desktop-on-macos.

## Installation

Extracted from the blog, the "one step installation".

```sh
# Install hyperkit and minikube
brew install hyperkit
brew install minikube

# Install Docker CLI
brew install docker
brew install docker-compose

# Start minikube
minikube start

# Tell Docker CLI to talk to minikube's VM
eval $(minikube docker-env)

# Save IP to a hostname
echo "$(minikube ip) docker.local" | sudo tee -a /etc/hosts > /dev/null

# Test
docker run hello-world
```

## Notes

Problems encountered and solutions. From uninstalling `docker desktop` to working on my project working again. This is only tested on _my_ project and only on mac os. Not every project, or every os, so you may find other issues.

### Docker cli configuration in `~/.profile`

The part where you configures the docker cli only sets up your current terminal. And needs to be done in every terminal you open, before you use docker.

```sh
# Tell Docker CLI to talk to minikube's VM
eval $(minikube docker-env)
```

Put it in your `.profile`/`.zshrc`.

```sh
echo '
##
## Docker
##
# Configure docker cli to use minikube
eval "$(minikube docker-env)"
' >> ~/.profile
```

### Docker credentials store

Since we uninstalled docker-desktop we no longer have `docker-credential-desktop`. Issue first showed up when running `docker-compose`.

The docker config still exists. And it's incorrect.
https://github.com/docker/for-mac/issues/3785

```sh
$ code ~/.docker/config.json
```

Disable the credential store by setting and empty string. This is fine, since the credential store is only used to simplify `docker login`.
https://docs.docker.com/engine/reference/commandline/login/

```diff
{
-   "credsStore" : "desktop"
+   "credsStore" : ""
}
```

If you have another credential provider, you can use that.

```diff
{
-   "credsStore" : "desktop"
+   "credsStore" : "<YOUR_CREDENTIAL_PROVIDER>"
}
```

### Docker compose "caveats" during installation

No problem found yet, but saving the warning in case I need to fix it later.

```
$ brew install docker-compose
==> Downloading https://ghcr.io/v2/homebrew/core/docker-compose/manifests/2.2.3
######################################################################## 100.0%
==> Downloading https://ghcr.io/v2/homebrew/core/docker-compose/blobs/sha256:af736ed840766
==> Downloading from https://pkg-containers.githubusercontent.com/ghcr1/blobs/sha256:af736
######################################################################## 100.0%
==> Pouring docker-compose--2.2.3.big_sur.bottle.tar.gz
==> Caveats
Compose is now a Docker plugin. For Docker to find this plugin, symlink it:
  mkdir -p ~/.docker/cli-plugins
  ln -sfn /usr/local/opt/docker-compose/bin/docker-compose ~/.docker/cli-plugins/docker-compose
==> Summary
🍺  /usr/local/Cellar/docker-compose/2.2.3: 6 files, 26.1MB
==> Running `brew cleanup docker-compose`...
Disable this behaviour by setting HOMEBREW_NO_INSTALL_CLEANUP.
Hide these hints with HOMEBREW_NO_ENV_HINTS (see `man brew`).
```
