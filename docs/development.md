# Development

## Building the images

To build an image run `./build.ps1 -build -name <ImageFolderName>`.

### Example

For example to build Ubuntu 16.04/xenial, which is in `./release/stable/ubuntu16.04`:

```sh
PS /powershell-docker> ./build.ps1 -Build -Name ubuntu16.04
VERBOSE: lauching build with fromTag: trusty-20180531 Tag: 6.0.2-ubuntu-16.04 PSversion: 6.0.2
VERBOSE: lauching build with fromTag: trusty-20180531 Tag: 6.0.2-ubuntu-trusty PSversion: 6.0.2
VERBOSE: lauching build with fromTag: trusty-20180531 Tag: 6.0.2-ubuntu-trusty-20180531 PSversion: 6.0.2
VERBOSE: lauching build with fromTag: trusty-20180531 Tag: 6.1.0-preview.2-ubuntu-16.04 PSversion: 6.1.0~preview.2
VERBOSE: lauching build with fromTag: trusty-20180531 Tag: 6.1.0-preview.2-ubuntu-trusty PSversion: 6.1.0~preview.2
VERBOSE: lauching build with fromTag: trusty-20180531 Tag: 6.1.0-preview.2-ubuntu-trusty-20180531 PSversion: 6.1.0~preview.2
VERBOSE: image name: powershell.local:6.0.2-ubuntu-16.04
VERBOSE: image name: powershell.local:6.0.2-ubuntu-trusty
VERBOSE: image name: powershell.local:6.0.2-ubuntu-trusty-20180531
VERBOSE: image name: powershell.local:6.1.0-preview.2-ubuntu-16.04
VERBOSE: image name: powershell.local:6.1.0-preview.2-ubuntu-trusty
VERBOSE: image name: powershell.local:6.1.0-preview.2-ubuntu-trusty-20180531
```

### Run the Docker image you built

```sh
PS /powershell-docker> docker run -it --rm powershell.local:6.1.0-preview.2-ubuntu-16.04 pwsh -c '$psversiontable'

Name                           Value
----                           -----
PSVersion                      6.0.2
PSEdition                      Core
GitCommitId                    v6.0.2
OS                             Linux 4.9.87-linuxkit-aufs #1 SMP Wed Mar 14 15:12:16 UTC 2018
Platform                       Unix
PSCompatibleVersions           {1.0, 2.0, 3.0, 4.0...}
PSRemotingProtocolVersion      2.3
SerializationVersion           1.1.0.1
WSManStackVersion              3.0
```

## Adding new Docker image

### Folder structure

The top level folder with the `Dockerfile`s is `release`.
This should only have folders under it.
The three folders are:

- `stable` - Images for the current stable release of PowerShell Core.
- `preview` - Images for the current preview release of PowerShell Core.
- `community-stable` - Images for the current release of PowerShell Core that are not officially supported.

Under each of these, will be a folder for each image.
The name of the folder will be the name of the image in the build system, but does not translate into anything in docker.
For example, the `stable` Ubuntu 16.04 image is in `release/stable/ubuntu16.04`.
In this folder, there are 4 items:

- `docker` - A folder containing the `Dockerfile` to build the image and any other files needed in the Docker build context.
- `test-deps` (official images only) - Directory for a sub-image. See the [`test-deps` image purpose](./index.md#test-dep-images).
- `dependabot` (optional) - in this directory you can put a dummy `Dockerfile` for [Dependabot](https://dependabot.com) to auto-bump the version. See [Dependabot](#dependabot).
- `meta.json` - See [this section](#metadata-files) later.

### `Dockerfile` Standards

All `Dockerfile`s should follow certain standards:

- The following comments should be applied at the beginning of the `Dockerfile`:

  - Copyright notice
  - Software license
  - A brief description should be applied after a new line.

   For example:

```dockerfile
# Copyright (c) Microsoft Corporation.
# Licensed under the MIT License.
#
# Docker image file that describes a brief description of the image to describe
# this image
```

- All arguments should be defaulted if needed to successfully build without specifying the argument

- The `FROM` statement should use an argument.

  For example:

```dockerfile
ARG fromTag=16.04

FROM ubuntu:${fromTag}
```

- A `PS_VERSION` argument should be defined, and used wherever the version is needed.

  For example:

```dockerfile
ARG PS_VERSION=6.0.4
```

- An `IMAGE_NAME` argument should be defined, and used in the labels where the image name is needed.

  For example:

```dockerfile
ARG IMAGE_NAME=mcr.microsoft.com/powershell:ubuntu16.04
```

## Testing

You should not have to write any specific tests for your image,
but you should consider if it needs to be added to the CI system.

The CI definition is here at `vsts-ci.yml`.

### Template

TODO: Update this

## Tags

Tags are a JSON array that describes the tags the image should have.

### Supported Tags

Tags you can use:

- `#psversion#` is replaced by the `<Major>.<Minor>` form of the version of PowerShell used to build the image.
- `#shorttag#` is replaced by short tags generated from meta.json.

### Example Tag Template

```json
"tagTemplates": [
    "#psversion#-windowsservercore-#tag#",
    "windowsservercore-#tag#"
]
```

## Metadata Files

This file *is **required*** for all containers. Here is the bare minimum:

```json
{
    "IsLinux" : true
}
```

### Host architecture

You can specify the host architecture.
Currently, `amd64` is supported everywhere and
`arm64` is supported on Linux.
Linux `arm64` can build both `arm32` and `arm64`.

```json
{
    "Architecture" : "amd64"
}
```

You should also add [tags](#tags) as a field.

## Dependabot

This repository has [Dependabot](https://dependabot.com) enabled on it.

The PRs opened for automatic base-image-version bumps will be closed, but the version will most likely get increased.

### Adding to a new Image

You will need to put a `Dockerfile` in the `dependabot` directory of your image, simply containing:

```dockerfile
FROM my-base-image:1.0.0
```

You will also need to add an entry in the `/.dependabot/config.yml` file. Here is a template for that:

```yaml
- package_manager: "docker"
    directory: "/release/theChannelHere/theImageHere/dependabot"
    update_schedule: "daily"
```

> **Do not use `latest` as the base**, as this makes the whole purpose invalid!
