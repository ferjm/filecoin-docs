---
title: 'Lotus: install and setup'
description: 'This guide covers how to install the Lotus applications and launch a Lotus Node.'
breadcrumb: 'Install and setup'
---

# {{ $frontmatter.title }}

{{ $frontmatter.description }}. This guide covers installing `lotus`, `lotus-miner` and `lotus-worker` to your computer, and then runs through setting up a Lotus node. For information on running the miner, check the [Lotus Miner documentation](../../mine/lotus/README.md).

[[TOC]]

## Minimal requirements

To run a Lotus node, your computer must have:

- macOS or Linux installed. Windows is not yet supported.
- a quad-core CPU. Models with support for _Intel SHA Extensions_ (AMD since Zen microarchitecture, or Intel since Ice Lake) will significantly speed things up.
- 8 GiB RAM
- Enough space to store the current Lotus chain (preferably on an SSD storage medium). The chain grows at approximately 12 GiB per week.

:::warning
These are the minimal requirements to run a Lotus node. [Hardware requirements for Miners](../../mine/hardware-requirements.md) are different.
:::

## Linux

The following instructions are specific to Linux installations. Head to the [macOS](#macos) section if you want to install Lotus on a Mac.

### Software dependencies

You will need the following software installed to install and run Lotus.

#### System-specific

Building Lotus requires some system dependencies, usually provided by your distribution.

| Linux distribution | Dependency install command                                                                                                                               |
| ------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Arch Linux         | `sudo pacman -Syu opencl-icd-loader gcc git bzr jq pkg-config opencl-icd-loader opencl-headers`                                                          |
| Ubuntu/Debian      | `sudo apt update && sudo apt install mesa-opencl-icd ocl-icd-opencl-dev gcc git bzr jq pkg-config curl -y && sudo apt upgrade -y`                        |
| Fedora             | `sudo dnf -y update && sudo dnf -y install gcc git bzr jq pkgconfig mesa-libOpenCL mesa-libOpenCL-devel opencl-headers ocl-icd ocl-icd-devel clang llvm` |
| OpenSUSE           | `sudo zypper in gcc git jq make libOpenCL1 opencl-headers ocl-icd-devel clang llvm && sudo ln -s /usr/lib64/libOpenCL.so.1 /usr/lib64/libOpenCL.so`      |

#### Rustup

Lotus needs [rustup](https://rustup.rs). The easiest way to install it is:

```sh
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

:::tip
Make sure your `$PATH` variable is correctly configured after the rustup installation so that `cargo` and `rustc` are found in their rustup-configured locations.
:::

#### Go

To build Lotus, you need a working installation of [Go 1.14 or higher](https://golang.org/dl/):

```sh
wget -c https://dl.google.com/go/go1.14.7.linux-amd64.tar.gz -O - | sudo tar -xz -C /usr/local
```

Make sure that `/usr/local/go/bin` is in your `PATH`. If you are running into problems, check the [official Go installation instructions](https://golang.org/doc/install) for your operating system.

### Build and install Lotus

Once all the dependencies are installed, you can build and install the Lotus suite (`lotus`, `lotus-miner`, and `lotus-worker`).

1. Clone the repository:

   ```sh
   git clone https://github.com/filecoin-project/lotus.git
   cd lotus/
   ```

1. Checkout the branch corresponding to the [network you want to join](./switch-networks.md#rebuild-and-install-lotus-on-the-right-branch). You can look up the correct branch or tag for the network you want to join in the [networks dashboard](https://networks.filecoin.io):

   ```sh
   git checkout <branch_or_tag>
   ```

   Currently, the _master_ branch corresponds to **testnet**.

1. If you are in China, check out the specific [tips](tips-running-in-china.md).
1. If you have an AMD Zen or Intel Ice Lake CPU (or later), enable the use of SHA extensions by adding these two environment variables:

   ```sh
   export RUSTFLAGS="-C target-cpu=native -g"
   export FFI_BUILD_FROM_SOURCE=1
   ```

   See the [Native Filecoin FFI section](#native-filecoin-ffi) for more details about this process.

1. Build Lotus:

   ```sh
   make clean all
   sudo make install
   ```

   This will put `lotus`, `lotus-miner` and `lotus-worker` in `/usr/local/bin`.

   `lotus` will use the `$HOME/.lotus` folder by default for storage (configuration, chain data, wallets, etc). See [advanced options](configuration-and-advanced-usage.md) for information on how to customize the Lotus folder.

1. Lotus should now be installed on your computer.

#### Native Filecoin FFI

Some newer CPU architectures like AMD's Zen and Intel's Ice Lake, have support for SHA extensions. Having these extensions enabled significantly speeds up your Lotus node. To make full use of your processor's capabilities, make sure you set the following variables **before building from source**:

```sh
export RUSTFLAGS="-C target-cpu=native -g"
export FFI_BUILD_FROM_SOURCE=1
```

This method of building does not produce portable binaries. Make sure you run the binary on the same computer as you built it.

### Systemd service files

Lotus provides Systemd service files. They can be installed with:

```sh
make install-daemon-service
make install-miner-service
```

Once installed, you should be able to control Lotus using `systemctl [start|stop] lotus-daemon`.

::: tip
By default, the `lotus-daemon` service file redirects the logging output to `/var/log/lotus/daemon.log`, so `journalctl` displays nothing.
:::

## macOS

These instructions are specific to macOS. If you are installing Lotus on a Linux distribution, head over to the [Linux section](#linux).

### XCode Command Line Tools

Lotus requires that X-Code CLI tools be installed before building the Lotus binaries.

1. Check if you already have the XCode Command Line Tools installed via the CLI, run:

   ```sh
   xcode-select -p
   ```

   If this command returns a path, you can move on to the [next step](#install-homebrew). Otherwise, to install via the CLI, run:

   ```sh
   xcode-select --install
   ```

1. To update, run:

   ```sh
   sudo rm -rf /Library/Developer/CommandLineTools
   xcode-select --install
   ```

### Install Homebrew

We recommend that MacOS users use [Homebrew](https://brew.sh) to install each of the necessary packages.

1. Use the command `brew install` to install the following packages:

   ```sh
   brew install go bzr jq pkg-config rustup
   ```

1. Clone the repository:

   ```sh
   git clone https://github.com/filecoin-project/lotus.git
   cd lotus/
   ```

1. Checkout the branch corresponding to the [network you want to join](./switch-networks.md#rebuild-and-install-lotus-on-the-right-branch). You can look up the correct branch or tag for the network you want to join in the [networks dashboard](https://networks.filecoin.io):

   ```sh
   git checkout <branch_or_tag>
   ```

   Currently, the _master_ branch corresponds to **testnet**.

1. If you are in China, check out the specific [tips](tips-running-in-china.md).
1. Build Lotus:

   ```sh
   make clean && make all
   sudo make install
   ```

1. You should now have Lotus installed.

## Start the Lotus daemon

The `lotus` application runs as a daemon and a client to control and interact with that daemon. A daemon is a long-running program that is usually run in the background.

To start the Lotus daemon run:

```sh
lotus daemon
## When running with systemd do:
# systemctl start lotus-daemon
```

During the first run, Lotus will:

- Setup its data folder at `~/.lotus`.
- Download the necessary proof parameters. This is a few gigabytes of data that is downloaded once.
- Start syncing the Lotus chain.

The daemon will start producing lots of log messages right away.

:::tip
Do not be concerned by the number of warnings and sometimes errors showing in the logs. They are usually part of the usual functioning of the daemon as part of a distributed network.
:::

## Chain sync

As in many other blockchains, the Lotus Node will need to sync to the _tip_ of the chain after learning about this. We call this process _chain sync_. First, the headers for every block will be synced from tip to bottom. Afterward, blocks will be fetched and verified from bottom to top. To inspect the current sync status run:

```sh
lotus sync status
```

You can also interactively wait for the chain to be fully synced:

```
lotus sync wait
```

:::tip
Syncing the Filecoin chain is a _very_ slow process, and the chain grows at a very fast pace. A network 100 days old will take about 25 days to sync from scratch.

As a workaround, we are offering chain state snapshots that can be downloaded and used to initialize the daemon. This workaround is temporary until the syncing mechanism is improved. You can [download the latest state snapshot here](https://very-temporary-spacerace-chain-snapshot.s3-us-west-2.amazonaws.com/Spacerace_stateroots_snapshot_latest.car)

Then start your lotus daemon with:

```sh
lotus daemon --import-snapshot <snapshot>.car
```

For more information about chain snapshots, [see the Chain snapshots section](./chain-snapshots.md).
:::

To check how far behind you are when syncing the chain, run the following command:

```sh
date -d @$(./lotus chain getblock $(./lotus chain head) | jq .Timestamp)
```

## Interact with the daemon

The `lotus` command allows you to interact with a _running_ Lotus daemon. The `lotus-miner` and `lotus-worker` commands work in the same way.

Lotus comes with built-in CLI documentation:

```sh
# Show general help
lotus --help
# Show specific help for the "client" subcommand
lotus client --help
```

For example, after your Lotus daemon has been running for a few minutes, use `lotus` to check the number of other peers that it is connected to in the Filecoin network:

```sh
lotus net peers
```
