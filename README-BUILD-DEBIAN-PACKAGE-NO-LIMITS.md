# System Base

Ubuntu 14.04 minimal was choosen (Any other distro which supports docker should also be fine).

## System preparation
```
sudo -i
# Enter user password

apt-get update
apt install -y git docker.io
ln -sf /usr/bin/docker.io /usr/local/bin/docker
sed -i '$acomplete -F _docker docker' /etc/bash_completion.d/docker

service docker status
# docker.io start/running, process 14394
```

## Fetch special branch

```
git clone \
    --depth=1 \
    --recursive \
    --branch v6.3.1.37-btactic \
    https://github.com/btactic/build_tools.git \
    /build_tools
cd /build_tools
mkdir out
docker build --tag onlyoffice-document-editors-builder .
docker run -v $(pwd)/out:/build_tools/out onlyoffice-document-editors-builder
```

## Package is built

Package `onlyoffice-documentserver_6.3.0-111~btactic1_amd64.deb` should be found at `/build_tools/out/package/` directory.
Yes, the tag is v6.3.1.37-btactic and the package filename has 6.3.0-111~btactic1 version in it which it's different. That's the way it is.

# Usage

Use documentation on how to install OnlyOffice package on Ubuntu such as [https://helpcenter.onlyoffice.com/installation/docs-community-install-ubuntu.aspx](https://helpcenter.onlyoffice.com/installation/docs-community-install-ubuntu.aspx). Do not add OnlyOffice repo.

When asked to install onlyoffice-documentserver package do instead:

```
sudo apt-get install /path/to/onlyoffice-documentserver_6.3.0-111~btactic1_amd64.deb
```

# Developer notes

- Use tags to make sure to use your exact version
- This build not only depends on btactic organisation repos but on ONLYOFFICE ones. TODO: Create btactic-onlyoffice org and clone all the needed repos there.
- Even then the build depends on other software downloads so you could not perform it offline (accessing offline local repos).
- Hopefully they manage to remove the need of building everything instead of only server because building everything makes us to build qt which we don't need and it takes too long.
- It would be nice for them to also open their Debian package system as they say in https://github.com/ONLYOFFICE/build_tools/pull/338#pullrequestreview-691809988 .
- Find new upstream packages at [https://download.onlyoffice.com/](https://download.onlyoffice.com/)
- Compare files found at md5sums file (sorted) from control.tar.gz from new upstream packages to learn if changes are needed to our alternate package.

## Work on built container

If you have just built an container and you want to do Debian package manually (in order to avoid waiting for 9 hours for it to build again) you can do like this:

Once:
```
docker run -v $(pwd)/out:/build_tools/out onlyoffice-document-editors-builder
```
has finished we should have:

```
CONTAINER ID        IMAGE                                        COMMAND                CREATED             STATUS              PORTS               NAMES
035367486ecd        onlyoffice-document-editors-builder:latest   "/bin/sh -c 'cd tool   39 minutes ago      Exited (0) 3 minutes ago                           loving_pare
```

Now we can do a commit to freeze that status with:
```
docker commit 035367486ecd onlyoffice-just-built
```

and then we can enter that image without triggering the default build process:


```
cd /build_tools
docker run -it --entrypoint /bin/bash -v $(pwd)/out:/build_tools/out onlyoffice-just-built
```
.

Now we can visit: `/build_tools/out/package/onlyoffice` inside the docker image to build manually our package thanks to the `dpkg-buildpackage -b -us -uc` command.


## Branches

- debian-package : Bare minimum to build the package
- 6.3.1.37-btactic : Improvements over debian-package including btactic1 version and removing connection limits
- v6.3.1.37-btactic : Tag for the first release

# Warning

This is not an official onlyoffice build. Do not seek for help on OnlyOffice issues/forums unless you replicate it on original source code or original binaries from them.