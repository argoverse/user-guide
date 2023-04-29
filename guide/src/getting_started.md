# Getting Started

~~~admonish important
If you would like to take advantage of the most recent features, please install via `conda/install.sh`.
~~~

## Table of Contents

<!-- toc -->

## Overview

This section covers the following:

1. Installing `av2`.
2. Downloading the Argoverse 2 and TbV datasets.

## Setup

We _highly_ recommend using `conda` with the `conda-forge` channel for package management.

### Installing via `conda` (recommended)

You will need to install `conda` on your machine. We recommend installing [miniforge](https://github.com/conda-forge/miniforge#install):

```bash
wget -O Miniforge3.sh "https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-$(uname)-$(uname -m).sh"
bash Miniforge3.sh -bp "${HOME}/conda"
```

Then, install `av2`:

```bash
bash conda/install.sh && conda activate av2
```

~~~admonish note
You may need to run a post-install step to initialize `conda`:

```terminal
$(which conda) init $SHELL
```

If `conda` is not found, you will need to add the binary to your `PATH` environment variable.
~~~

### Installing via pip

Installation via PyPI requires manually installing Rust via [rustup](https://rustup.rs/).

Run the following and select the _default_ installation:

```terminal
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

Make sure to adjust your `PATH` as:
```terminal
export PATH=$HOME/.cargo/bin:$PATH
```

We use the `nightly` release of Rust for SIMD support. Default to it:
```terminal
rustup default nightly
```

Then, install `av2`:

```bash
pip install git+https://github.com/argoverse/av2-api#egg=av2
```

## Downloading the data

Our datasets are available for download from [AWS S3](https://aws.amazon.com/s3/).

For the best experience, we highly recommend using the open-source [s5cmd](https://github.com/peak/s5cmd) tool to transfer the data to your local filesystem. Please note that an AWS account is not required to download the datasets.

```admonish note
Additional info can be found at [https://aws.amazon.com/blogs/opensource/parallelizing-s3-workloads-s5cmd/]().
```

## Installing `s5cmd`

### Conda Installation (Recommended)

The easiest way to install `s5cmd` is through `conda` using the `conda-forge` channel:

```terminal
conda install s5cmd -c conda-forge
```

### Manual Installation

`s5cmd` can also be installed with the following script:

```bash
#!/usr/bin/env bash

export INSTALL_DIR=$HOME/.local/bin
export PATH=$PATH:$INSTALL_DIR
export S5CMD_URI=https://github.com/peak/s5cmd/releases/download/v2.0.0/s5cmd_2.0.0_$(uname | sed 's/Darwin/macOS/g')-64bit.tar.gz

mkdir -p $INSTALL_DIR
curl -sL $S5CMD_URI | tar -C $INSTALL_DIR -xvzf - s5cmd
```

Note that it will install `s5cmd` in your local bin directory. You can always change the path if you prefer installing it in another directory.

# Download the Datasets

Run the following command to download the one or more of the datasets:

```bash
#!/usr/bin/env bash

# Dataset URIs
# s3://argoverse/datasets/av2/sensor/ 
# s3://argoverse/datasets/av2/lidar/
# s3://argoverse/datasets/av2/motion-forecasting/
# s3://argoverse/datasets/av2/tbv/

export DATASET_NAME="sensor"  # sensor, lidar, motion_forecasting or tbv.
export TARGET_DIR="$HOME/data/datasets"  # Target directory on your machine.

s5cmd --no-sign-request cp "s3://argoverse/datasets/av2/$DATASET_NAME/*" $TARGET_DIR
```

The command will all data for `$DATASET_NAME` to `$TARGET_DIR`. Given the size of the dataset, it might take a couple of hours depending on the network connectivity.

When the download is finished, the dataset is ready to use!

## FAQ

> Why manage dependencies in `conda` instead of `pip`?

`conda` enables package management outside of the `python` ecosystem. This enables us to specify all necessary dependencies in `environment.yml`. Further, gpu-based packages (e.g., `torch`) are handled better through `conda`.

> Why `conda-forge`?

`conda-forge` is a community-driven channel of conda recipes. It includes a large number of packages which can all be properly tracked in the `conda` resolver allowing for consistent environments without conflicts.
