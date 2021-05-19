# Development Container Scripts
This folder contains a set of scripts that can be referenced by Dockerfiles in development container "definitions".  Taken from https://github.com/microsoft/vscode-dev-containers/tree/master/script-library

## Using a script

*This section will outline some general tips for alternate ways to reference the scripts in your Dockerfile.*

### Copying the script to .devcontainer/library-scripts

The easiest way to use a script is to simply copy it into a `.devcontainers/library-scripts` folder. From here you can then use the script as follows in your `Dockerfile`:

**Debian/Ubuntu**

```Dockerfile
COPY library-scripts/*.sh /tmp/library-scripts/
RUN bash /tmp/library-scripts/common-debian.sh
```

Generally it's also good to clean up after running a script in the same `RUN` statement to keep the "layer" small.

```Dockerfile
COPY library-scripts/*.sh /tmp/library-scripts/
RUN bash /tmp/library-scripts/common-debian.sh
    && apt-get clean -y && rm -rf /var/lib/apt/lists/* /tmp/library-scripts
```

**Alpine**

```Dockerfile
COPY library-scripts/*.sh /tmp/library-scripts/
RUN ash /tmp/library-scripts/common-alpine.sh \
    && rm -rf /tmp/library-scripts
```

**CentOS/RedHat/Oracle Linux**

```Dockerfile
COPY library-scripts/*.sh /tmp/library-scripts/
RUN bash /tmp/library-scripts/common-redhat.sh \
    && yum clean all && rm -rf /tmp/library-scripts
```

Note that the CI process for this repository will automatically keep scripts in the `.devcontainers/library-scripts` folder up to date for each definition in the `containers` folder.

### Downloading the script with curl / wget instead

If you prefer, you can download the script using `curl` or `wget` and execute it instead. This can convenient to do with your own `Dockerfile`, but is generally avoided for definitions in this repository. To avoid unexpected issues, you should reference a release specific version of the script, rather than using master. For example:

```Dockerfile
RUN bash -c "$(curl -fsSL "https://raw.githubusercontent.com/microsoft/vscode-dev-containers/master/script-library/common-debian.sh")" \
    && apt-get clean -y && rm -rf /var/lib/apt/lists/*
```

Or if you're not sure if `curl` is installed:

```Dockerfile
RUN apt-get update && export DEBIAN_FRONTEND=noninteractive  \
    && apt-get -y install --no-install-recommends curl ca-certificates \
    && bash -c "$(curl -fsSL "https://raw.githubusercontent.com/microsoft/vscode-dev-containers/master/script-library/common-debian.sh")" \
    && apt-get clean -y && rm -rf /var/lib/apt/lists/*
```

As before, the last line is technically optional, but minimizes the size of the layer by removing temporary contents.

You can also use `wget`:

```Dockerfile
RUN apt-get update && export DEBIAN_FRONTEND=noninteractive  \
    && apt-get -y install --no-install-recommends wget ca-certificates \
    && bash -c "$(wget -qO- "https://raw.githubusercontent.com/microsoft/vscode-dev-containers/master/script-library/common-debian.sh")" \
    && apt-get clean -y && rm -rf /var/lib/apt/lists/*
```

### Arguments

Some scripts include arguments that you can allow developers to set by using `ARG` in your `Dockerfile`.

#### Using arguments with scripts from the .devcontainers/library-scripts folder

In this case, you can simply pass in the arguments to the script.

```Dockerfile
# Options for script
ARG INSTALL_ZSH="true"

COPY library-scripts/*.sh /tmp/library-scripts/
RUN /bin/bash /tmp/library-scripts/common-debian.sh "${INSTALL_ZSH}" "vscode" "1000" "1000" "true" "true" "true"\
    && apt-get clean -y && rm -rf /var/lib/apt/lists/* /tmp/library-scripts
```

#### Using arguments when downloading with curl

The trick here is to use the double-dashes (`--`) after the `bash -c` command and then listing the arguments.

```Dockerfile
# Options for script
ARG INSTALL_ZSH="true"

# Download script and run it with the option above
RUN apt-get update && export DEBIAN_FRONTEND=noninteractive  \
    && apt-get -y install --no-install-recommends curl ca-certificates \
    && bash -c "$(curl -fsSL "https://raw.githubusercontent.com/microsoft/vscode-dev-containers/master/script-library/common-debian.sh")" -- "${INSTALL_ZSH}" "vscode" "1000" "1000" "true" \
    && apt-get clean -y && rm -rf /var/lib/apt/lists/*