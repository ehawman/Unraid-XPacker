# Unraid-XPacker

## Introduction

Aims to be a streamlined package creator for dmacias72/unRAID-NerdPack

Goals:

- Take minimal human input:
  - SlackBuild URL
  - Source Code URL
  - Authorizing GitHub maybe?
  - Optional: Specify which version(s) of Unraid to build packages for
- Follow instructions below:
  - Create/prepare Unraid VM(s)
  - Download/unzip/modify files from slackbuilds.org and source
  - Run SlackBuild script
    - Log the error
    - If the first error, present error and ask user if they'd like to continue or break
    - If continue, continue (through all subsequent errors)
    - At the end of all VMs, cache VMs that didn't work.
    - On next run, ask user if they'd _just_ like to run these. Clear cache on fully successful run.
  - Prep local fork, create branch, move files from VM to local (`txz` and Source dir)
  - Repeat above for each version of Unraid
  - Create pull request

## Dependencies

- [Rust w/ rustup](https://www.rust-lang.org/tools/install)
- Git
- A Git manager configured to work with GitHub
  - GitKraken
  - GitHub CLI
  - VSCode
  - Many, many _many_ more.
- Docker

Written in Rust? Python? I think this app can live on the local machine, change files which are mounted into a Docker container, and use `docker exec` to issue commands to the container directly. Point being, don't have to worry about trying to get Rust to work on Unraid, or installing NerdPack to get Python working on Unraid.

Never worked with Rust but hey let's give it a shot.

## Human-readable instructions

Environment Setup:

1. Setup an Unraid VM
   - Create a copy of your drive and install it as a VM onto your Unraid server. # TODO: Make full writeup of this step
   - If you have already created a drive image for a desired Unraid version, use it here. (You'd have multiples. Please read below)
2. Setup SSH # TODO: Make a full writeup of this step
3. Install DevPack
4. Create a build directory
   - `/boot/package_builds/`

Build Package:

1. Download the x64 Source to your build dir.
   - [Source Code micro-2.0.10-linux64.tar.gz](https://github.com/zyedidia/micro/releases/download/v2.0.10/micro-2.0.10-linux64.tar.gz)
   - 📂package_builds  
     ┗ 📂micro  
     ┃ ┗ 📦micro-2.0.10-linux64.tar.gz
2. Find desired package on slackbuilds.org, download the "SlackBuild", and extract it into the your build dir.
   - [slackbuilds.org micro.tar.gz](https://slackbuilds.org/slackbuilds/15.0/development/micro.tar.gz)
   - 📂package_builds  
     ┗ 📂micro  
     ┃ ┣ 📦micro-2.0.10-linux64.tar.gz  
     ┃ ┣ 📜doinst.sh  
     ┃ ┣ 📜micro.info  
     ┃ ┣ 📜micro.SlackBuild  
     ┃ ┣ 📜README  
     ┃ ┗ 📜slack-desc
3. In {package_name}.SlackBuild, change `PKGTYPE=${PKGTYPE:-tgz}` to `PKGTYPE=${PKGTYPE:-txz}`
4. If the version of the source code is different than the version listed in the SlackBuild file, update the SlackBuild to match.
   - `VERSION=${VERSION:-2.0.10}`
5. Access VM console via SSH or webterminal and cd into your build folder
   - `cd /boot/package_builds/micro`
6. Run the SlackBuild file. If it fails, attempt to navigate the errors, rerun, and repeat until successful.
   - `./micro.SlackBuild`
7. Copy resulting package back to build folder
   - `cp -r /tmp/micro-2.0.10.txz /boot/package_builds/micro-2.0.10.tzx`

Prepare Git Pull Request:

1. Either fork [dmacias72/unRAID-NerdPack](https://github.com/dmacias72/unRAID-NerdPack), or pull/fast-forward your existing fork's `Main` branch.
2. Create a branch for the package you want to create
   - `micro-2.10.0`
3. Checkout the branch to your local machine (**not** the VM)
4. Copy `yourpackage.txz` from the Unraid VM to the fork on your local machine, in `/packages/[version-of-your-Unraid-VM]/`
   - `scp root@yourunraidserver:/domains/unraid_build_vm_example_volume/boot/package_builds/micro-2.0.10.tzx /path/to/git/folder/unRAID-NerdPack/packages/6.10/micro-2.0.10.tzx`
5. Copy the Source folder from the Unraid VM to the fork on your local machine, in `/source/SlackBuild/{your package name}/
   - `scp -r root@yourunraidserver:/domains/unraid_build_vm_example_volume/boot/package_builds/micro/ /path/to/git/folder/unRAID-NerdPack/source/SlackBuild/micro/`
6. To create packages for multiple Unraid versions, save a copy of the VM drive as the Unraid version number for future use, then go all the way back to Step 1 and restart with a different Unraid version as your VM. Eventually you'll have a nice stable of Unraid images to build packages for, and this process won't suck nearly as much.
7. Create pull request. Be sure to specify which Unraid versions you've built packages for.
8. Once your pull request has been accepted, feel free to delete your branch (on both origin and local).
