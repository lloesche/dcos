# DC/OS Tools, Packages, and Installers for various platforms

Tools for building DC/OS and launching a cluster with it in the hardware of a customer's choice.

  - *docker/* Locally defined docker containers packages are built in
  - *docs/* Documentation
  - *ext/dcos-installer/* Backend for Web, SSH, and some bits of the Advanced installer. To be merged into the top codebase once the code is cleaned up
  - *gen/* Python library for rendering yaml config files for various platforms into packages, with utilities to do things like make "late binding" config set by CloudFormation
  - *gen/installer/* Code to take a build and transform it into a particular platform installer (Bash / command line, AWS, Azure, etc.)
  - *packages/* Packages which make up DC/OS (Mesos, Marathon, AdminRouter, etc). These packages are built by pkgpanda, and combined into a "bootstrap" tarball for deployment.
  - *pkgpanda/* DC/OS baseline/host package management system. Tools for building, deploying, upgrading, and bundling packages together which live on the root filesystem of a machine / underneath Mesos.
  - *pytest/* Misc. tests. Should be moved to live next to the appropriate code
  - *release/* Release tools for DC/OS. (Building releases, building installers for releases, promoting between channels)
  - *ssh/* AsyncIO based parallel ssh library used by the installer
  - *test_util/* various scripts, utilities to help with integration testing

All code in this repository is Python 3

# Doing a local build

## Dependencies
  1. Linux distribution:
    - Docker doesn't have all the features needed on OS X or Windows
    - `tar` needs to be GNU tar for the set of flags used
  1. [tox](https://tox.readthedocs.org/en/latest/)
  1. git
  1. Docker
    - [Install Instructions for varios distributions](https://docs.docker.com/engine/installation/). Docker needs to be configued so your user can run docker containers. The command `docker run alpine  /bin/echo 'Hello, World!'` when run at a new terminal as your user should just print `"Hello, World!"`. If it says something like "Unable to find image 'alpine:latest' locally" then re-run and the message should go away.
  1. Python 3.4
    - Arch Linux: `sudo pacman -S python`
    - Fedora 23 Workstation: Already installed by default / no steps
  1. Over 10GB of free disk space
  1. _Optional_ pxz (speeds up package and bootstrap compression)
    - ArchLinux: [pxz-git in the AUR](https://aur.archlinux.org/packages/pxz-git). The pxz package corrupts tarballs fairly frequently.
    - Fedora 23: `sudo dnf install pxz`

## Running local code quality tests
```
tox
```

[Tox](https://tox.readthedocs.io/en/latest/) is used to run the codebase unit tests, as well as coding standard checks. The config is in `tox.ini`.

## Running a DC/OS Build

```
./build_local.sh
```

That will run a simple local build, and output the resulting DC/OS installers to $HOME/dcos-artifacts. You can run the created `dcos_generate_config.sh like so:

NOTE: Building a release from scratch the first time on a modern dev machine (4 cores / 8 hyper threads, SSD, reasonable interent bandwidth) takes about 1 hour.

```
$ $HOME/dcos-artifacts/testing/`whoami`/dcos_generate_config.sh
```

## What's happening under the covers

If you look inside of the bash script `build_local.sh` there are the commands with discriptions of each.

The general flow is to:
1. Check the environment is reasonable
2. Write a `release` tool configuration if one doesn't exist
3. Setup a python virtualenv where we can install the DC/OS python tools to in order to run them
4. Install the DC/OS python tools to the virtualenv
5. Build the release using the `release` tool

These steps can all be done by hand and customized / tweaked like standard python projects. You can hand create a virtualenvironment, and then do an editable pip install (`pip install -e`) to have a "live" working environment (as you change code you can run the tool and see the results).

## Release Tool Configuration

This release tool always loads the config in `dcos-release.config.yaml` in the current directory.

The config is [YAML](http://yaml.org/). Inside it has two main sections. `storage` which contains a dictionary of different storage providers which the built artifacts should be sent to, and `options` which sets general DC/OS build configuration options.

Config values can either be specified directly, or you may use $ prefixed environment variables (the env variable must set the whole value).

### Storage Providers
All the available storage providers are in [release/storage](./release/storage/). The configuration is a dictionary of a reference name for the storage provider (local, aws, my_azure), to the configuration.

Each storage provider (ex: aws.py) is an available kind prefix. The dictionary `factories` defines the suffix for a particular kind. For instance `kind: aws_s3` would map to the S3StorageProvider.

The configuration options for a storage provider are the storage provider's constructor parameters.

Sample config storage that will save to my home directory (/home/cmaloney):
```yaml
storage:
  local:
    kind: local_path
    path: /home/cmaloney/dcos-artifacts
```

Sample config that will store to a local archive path as wll as AWS S3. Environment variables AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY would need to be set to use the config (And something like a CI system could provide them so they don't have to be committed to a code repository).
```yaml
storage:
  aws:
    kind: aws_s3
    bucket: downloads.dcos.io
    object_prefix: dcos
    download_url: https://downloads.dcos.io/dcos/
    access_key_id: $AWS_ACCESS_KEY_ID
    secret_access_key: $AWS_SECRET_ACCESS_KEY
    region_name: us-west-2
  local:
    kind: local_path
    path: /mnt/big_artifact_store/dcos/
```

# TODO

Lots of docs are still being written. If you have immediate questions please ask the [DC/OS Community](https://dcos.io/community/). Someone else probably has exactly the same question.

 - Add getting started on common distros / dependencies
 - Add overview of what is in here, how it works
 - Add general theory of stuff that goes in here.
 - PR (guidelines, testing)
 - How to make different sorts of common changes
