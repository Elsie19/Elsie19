---
layout: post
title: Making a pacscript
subtitle: A quick mini-tutorial on making one
gh-repo: pacstall/pacstall-programs
gh-badge: [star, fork, follow]
tags: [pacstall]
comments: true
mathjax: false
author: Elsie
---

## Picking a Package

So first, I'm going to pick a project to package. If you don't know what to package, consider looking at our [package requests](https://github.com/pacstall/pacstall-programs/issues?q=sort%3Aupdated-desc+is%3Aissue+state%3Aopen+label%3A%22package+request%22) in our repository. There are *always* open requests who have at least one person interested in seeing it exist!

For this post, I will *not* be packaging one of these. There are a couple reasons, but the main one is simply that I don't want to package something I'm not interested in maintaining.

I'm going to package [soar](https://github.com/pkgforge/soar), which from the tin says that it is:

> A fast, modern package manager for Static Binaries, Portable Formats \(AppImage\|AppBundle\|FlatImage\|Runimage\) & More

## Prerequisites

### Figuring out versions

First of all, I notice that it's a rust package, so the very first thing I'm going to do is clone the repository and try to probe the minimum required cargo/rustc version that this project can be built on. For this, I use the [`cargo-msrv`](https://gribnau.dev/cargo-msrv/) tool. I cloned soar at commit `37e92bb61afb2ae90a89eb9fe90d449150bcfdce`, so for future reference, you might want to `checkout` that commit.

```bash
git clone https://github.com/pkgforge/soar && cd soar
cargo msrv find
```

This will take some time, but here are the results:

```
Result:
   Considered (min … max):   Rust 0.11.0 … Rust 1.85.0
   Search method:            bisect
   MSRV:                     1.82.0
   Target:                   x86_64-unknown-linux-gnu
```

Note that MSRV line, because that is what we will use later to "version clamp" our pacscript.

### Finding useful information

The next step is to snoop around in the repo and figure out some other important metadata that we can fill our pacscript with later. Usually that is figuring out the license, runtime dependencies, and maintainer specified build instructions.

#### License

This is fairly easy, since soar is written in Rust, I can look in the `Cargo.toml` and lo and behold, [we've figured out](https://github.com/pkgforge/soar/blob/42cf13f8375895121bb8d295a8d8a1fb0b568b28/Cargo.toml#L10) that the license is [MIT](https://en.wikipedia.org/wiki/MIT_License).

#### Runtime dependencies

From the README, I found the a link to their [installation docs](https://soar.qaidvoid.dev/installation), which leads to an install script, and upon reading it, it appears there aren't any runtime dependencies.

#### Maintainer build instructions

Many projects have specific build instructions that they expect users and package maintainers to adhere to, but I have found no such instructions, which means we are fine to package it however we like[^1].

## Creating our pacscript

### Template

Next, I will setup a template for our pacscript, which I grabbed from [pacscript 101](https://github.com/pacstall/pacstall/wiki/Pacscript-101), which looks like this when filled out:

```bash
# file: soar.pacscript
pkgname="soar"
arch=("any")
pkgver="0.5.9"
license=("MIT")
url='https://soar.qaidvoid.dev/'
pkgdesc='Fast, modern package manager for Static Binaries, Portable Formats (AppImage|AppBundle|FlatImage|Runimage) & More'
source=("https://github.com/pkgforge/soar/archive/refs/tags/v${pkgver}.tar.gz")
sha256sums=()
makedepends=("cargo>=1.82.0")
maintainer=("Elsie19 <hwengerstickel@pm.me>")
external_connection=true
```

If any of these seem confusing or out of place, please look at the [variables](https://github.com/pacstall/pacstall/wiki/101.1-Variables) section of our official tutorial and look at each one individually.

### Figuring out hashsums

Next, we will want to figure out the hashsum of our source, so what I usually do is copy the `pkgver` line in a bash shell, and then copy the `source` link and download it:

```
[~] elsie@elsie $ pkgver="0.5.9"
[~] elsie@elsie $ source=("https://github.com/pkgforge/soar/archive/refs/tags/v${pkgver}.tar.gz")
[~] elsie@elsie $ wget -q --show-progress --progress=bar:force ${source[0]} 2>&1
v0.5.9.tar.gz                 [  <=>                                ]  97.37K   425KB/s    in 0.2s
[~] elsie@elsie $ sha256sum v0.5.9.tar.gz
70be79806608cfb8482bc9d81e2238117cd31a60734939907031c25f76ed266b  v0.5.9.tar.gz
```

Now that we have that hash, we will put it into the `sha256sums` array.

### Time to build

Now that we have all our variables in place, we can work on actually building soar. I will be using our [official rust packaging guidelines](https://github.com/pacstall/pacstall/wiki/Different-Package-Build-Systems#cargo) for this.

Once that has been done, you should have something resembling:

```bash
pkgname="soar"
arch=("any")
pkgver="0.5.9"
license=("MIT")
url='https://soar.qaidvoid.dev/'
pkgdesc='Fast, modern package manager for Static Binaries, Portable Formats (AppImage|AppBundle|FlatImage|Runimage) & More'
source=("https://github.com/pkgforge/soar/archive/refs/tags/v${pkgver}.tar.gz")
sha256sums=("70be79806608cfb8482bc9d81e2238117cd31a60734939907031c25f76ed266b")
makedepends=("cargo>=1.82.0")
maintainer=("Elsie19 <hwengerstickel@pm.me>")
external_connection=true

build() {
  cd soar/
  cargo build -j"${NCPU}" --release --locked
}

package() {
  cd soar/
  install -Dm755 "target/release/soar" -t "${pkgdir}/usr/bin"
}
```

## Testing the pacscript

At this point, I have not actually tested this pacscript from start to finish; only certain parts, so now we can begin to test:

```
~ % ››› pacstall -I soar.pacscript
(soar) Do you want to view/edit the pacscript? [y/N] n
[+] INFO: Sourcing pacscript
⟡ sudo ~>
[*] WARNING: This package will connect to the internet during its build process.
[+] INFO: Checking build dependencies
	[>] gzip ✓ [installed]
	[>] tar ✓ [installed]
	[>] cargo>=1.82.0 ✗ [required]
	[!] ERROR: cargo>=1.82.0 version cannot be satisfied
[+] INFO: Cleaning up
```

Uh-oh! I don't have the right cargo version! But wait a minute? Didn't I just check it up in [the MSRV finding stage](#Figuring-out-versions)? Yes, I did. But as it turns out, my cargo binary and the apt installed one are different:

```
~ % ››› which cargo
/home/elsie/.cargo/bin/cargo
~ % ››› ./.cargo/bin/cargo --version
cargo 1.84.0 (66221abde 2024-11-19)
~ % ››› dpkg --listfiles cargo
/.
/usr
/usr/bin
/usr/share
/usr/share/bash-completion
/usr/share/bash-completion/completions
/usr/share/cargo
/usr/share/cargo/bin
/usr/share/doc
/usr/share/doc/cargo
/usr/share/doc/cargo/copyright
/usr/share/zsh
/usr/share/zsh/vendor-completions
/usr/bin/cargo
/usr/share/bash-completion/completions/cargo
/usr/share/cargo/bin/cargo
/usr/share/cargo/scripts
/usr/share/doc/cargo/changelog.gz
/usr/share/zsh/vendor-completions/_cargo
~ % ››› /usr/share/cargo/bin/cargo --version
cargo 1.80.1 (376290515 2024-07-16)
```

Even worse, even if I removed the version constraints we added earlier, it still would not work, because pacstall builds packages in a sandbox, where my version of cargo is not in `PATH`!

Luckily, we have tools for this, specifically [pacstall-docker-builder](https://github.com/pacstall/docker), which we can use to generate a docker image for us to test.

#### Getting our docker image

We will need install the program and decide on some important flags:

```bash
pacstall -I pacstall-docker-builder-git
```

We are going to want to generate an Ubuntu release with a apt version of cargo that satisfies soar's constraints (`ubuntu:devel` or `ubuntu:25.04`), so the command should look something like:

```bash
./pacstall-docker-builder --distro ubuntu:devel --version master --test
```

In a separate terminal window, we're going to copy our pacscript to the docker to test, so let's figure out what the container is:

```bash
~ % ››› docker ps
CONTAINER ID   IMAGE                          COMMAND   CREATED         STATUS         PORTS     NAMES
5afa8902c6cc   pacstall/ubuntu-devel:master   "bash"    6 seconds ago   Up 5 seconds             great_burnell
```

With that ID (`5afa8902c6cc`), you can run:

```bash
docker cp soar.pacscript 5afa8902c6cc:/home/pacstall/
```

Now we can begin testing.

#### Actually testing

Inside the docker container, we can try to test it again:

```
pacstall@elsie:~$ pacstall -I soar.pacscript
.....
[+] INFO: Downloading v0.5.9.tar.gz
v0.5.9.tar.gz             [  <=>                  ]  97.37K   316KB/s    in 0.3s
	[>] Checking hash 70be7980[...]
	[>] Extracting v0.5.9.tar.gz
[+] INFO: Running functions
	[>] Running build
/tmp/pacstall/bwrapenv.3dIvd3bgTe: line 120: cd: soar/: No such file or directory
error: could not find `Cargo.toml` in `/tmp/pacstall/soar~0.5.9` or any parent directory
	[!] ERROR: Could not build soar properly
[+] INFO: Cleaning up
```

Well that was a bust. Remember when I downloaded the archive to get the hash? When I inspect it, I see that inside the archive, the next directory inside is `soar-0.5.9`, which is just `soar-${pkgver}`, so let's adjust the pacscript and send it back to the docker to re-test:

```bash
pkgname="soar"
arch=("any")
pkgver="0.5.9"
license=("MIT")
url='https://soar.qaidvoid.dev/'
pkgdesc='Fast, modern package manager for Static Binaries, Portable Formats (AppImage|AppBundle|FlatImage|Runimage) & More'
source=("https://github.com/pkgforge/soar/archive/refs/tags/v${pkgver}.tar.gz")
sha256sums=("70be79806608cfb8482bc9d81e2238117cd31a60734939907031c25f76ed266b")
makedepends=("cargo>=1.82.0")
maintainer=("Elsie19 <hwengerstickel@pm.me>")
external_connection=true

build() {
  cd "soar-${pkgver}"
  cargo build -j"${NCPU}" --release --locked
}

package() {
  cd "soar-${pkgver}"
  install -Dm755 "target/release/soar" -t "${pkgdir}/usr/bin"
}
```

And now it starts to build as usual:

```
pacstall@elsie:~$ pacstall -I soar.pacscript
.....
[+] INFO: Packaging soar
	[>] Packing control.tar
	[>] Packing data.tar
	[>] Compressing
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
Note, selecting 'soar' instead of '/usr/src/pacstall/soar_0.5.9-pacstall1_amd64.deb'
The following NEW packages will be installed:
  soar
0 upgraded, 1 newly installed, 0 to remove and 0 not upgraded.
Need to get 0 B/3650 kB of archives.
After this operation, 7805 kB of additional disk space will be used.
Get:1 /usr/src/pacstall/soar_0.5.9-pacstall1_amd64.deb soar amd64 0.5.9-pacstall1 [3650 kB]
Selecting previously unselected package soar.
(Reading database ... 16945 files and directories currently installed.)
Preparing to unpack .../soar_0.5.9-pacstall1_amd64.deb ...
Unpacking soar (0.5.9-pacstall1) ...
Setting up soar (0.5.9-pacstall1) ...
[+] INFO: Performing post install operations
	[>] Storing pacscript
[+] INFO: Done installing soar
[+] INFO: Cleaning up
```

And now when I run `soar`, I get the command as intended:

```
pacstall@elsie:~$ soar
soar-cli 0.5.9
Rabindra Dhakal <contact@qaidvoid.dev>
A modern package manager for Linux

Usage: soar [OPTIONS] <COMMAND>

Commands:
  config     Print the configuration file to stdout
  install    Install packages [aliases: i, add]
  search     Search package [aliases: s, find]
  query      Query package info [aliases: Q]
  remove     Remove packages [aliases: r, del]
  sync       Sync with remote metadata [aliases: S, fetch]
  update     Update packages [aliases: u, upgrade]
  info       Show info about installed packages [aliases: list-installed]
  list       List all available packages [aliases: ls]
  log        Inspect package build log
  inspect    Inspect package build script
  run        Run packages without installing to PATH [aliases: exec, execute]
  use        Use package from different family
  download   Download arbitrary files [aliases: dl]
  health     Health check
  defconfig  Generate default config
  env        View env
  clean      Garbage collection
  self       Modify the soar installation
  help       Print this message or the help of the given subcommand(s)

Options:
  -v, --verbose...
  -q, --quiet
  -j, --json
      --no-color
  -p, --profile <PROFILE>
  -c, --config <CONFIG>
  -h, --help               Print help
  -V, --version            Print version
```

## Making a PR

### Setting up the development environment

Now that we've established our pacscript can build, it's time to make our PR.

First you need to [fork](https://github.com/pacstall/pacstall/fork) our repository and clone it:

```bash
git clone git@github.com:$GITHUB_USER/pacstall-programs.git "${GITHUB_USER}-programs" && cd "${GITHUB_USER}-programs"
```

Next, you will want to make a branch for the program:

```bash
git checkout -b add/soar
```

Now, you should setup the development environment, which is laid out in [our readme](https://github.com/pacstall/pacstall-programs#how-to-setup-the-environment-for-pacscript-development).

### Creating the package structure

Next, we'll take the name of the package (soar) and run the following commands

```bash
mkdir -p packages/soar/ && cp path/to/soar.pacscript packages/soar/
```

Now, you should run the following commands:

```
~/Elsie19-programs % ››› git add .
~/Elsie19-programs % ››› git commit -m 'add: `soar`'
trim trailing whitespace.................................................Passed
fix end of files.........................................................Passed
check yaml...........................................(no files to check)Skipped
check for added large files..............................................Passed
mixed line ending........................................................Passed
shellcheck...............................................................Passed
shfmt....................................................................Passed
update .SRCINFO data.....................................................Passed
check for .SRCINFO.......................................................Failed
- hook id: check-srcinfo
- exit code: 1

[ERR]: no .SRCINFO tracked by git for soar.pacscript
  [>]: run git add packages/soar/.SRCINFO

update packagelist.......................................................Failed
- hook id: update-pkglist
- files were modified by this hook
update srclist...........................................................Failed
- hook id: update-srclist
- files were modified by this hook
```

We've run into a very common issue (at least on my machine), but no worry, we just have to rerun both commands:

```
~/Elsie19-programs % ››› git add .
~/Elsie19-programs % ››› git commit -m 'add: `soar`'
trim trailing whitespace.................................................Passed
fix end of files.........................................................Passed
check yaml...........................................(no files to check)Skipped
check for added large files..............................................Passed
mixed line ending........................................................Passed
shellcheck...............................................................Passed
shfmt....................................................................Passed
update .SRCINFO data.....................................................Passed
check for .SRCINFO.......................................................Passed
update packagelist.......................................................Passed
update srclist...........................................................Passed
[add/soar 30c9c08c] add: `soar`
 4 files changed, 47 insertions(+)
 create mode 100644 packages/soar/.SRCINFO
 create mode 100644 packages/soar/soar.pacscript
```

### Pushing

Just run:

```
~/Elsie19-programs % ››› git push origin add/soar
Enumerating objects: 53, done.
Counting objects: 100% (53/53), done.
Delta compression using up to 6 threads
Compressing objects: 100% (40/40), done.
Writing objects: 100% (40/40), 10.71 KiB | 1.53 MiB/s, done.
Total 40 (delta 24), reused 0 (delta 0), pack-reused 0 (from 0)
remote: Resolving deltas: 100% (24/24), completed with 9 local objects.
remote:
remote: Create a pull request for 'add/soar' on GitHub by visiting:
remote:      https://github.com/Elsie19/pacstall-programs/pull/new/add/soar
remote:
To github.com:Elsie19/pacstall-programs.git
 * [new branch]        add/soar -> add/soar
```

And click that link that GitHub has so helpfully provided to us in the remote git hooks and make the PR. Do not modify the title, but if you want to add a little message to the description you are free to do so.

## Dealing with reviewers

So now that I've made the PR, I will wait for a reviewer to come and look at it. You'll notice that some checks run during this period, which simulate essentially what we already tested, but on a multitude of different distros, versions, architectures, and pacstall versions. Don't worry if some fail: pacstall reviewers are more leniant about failing tests if version constraints are in place.

### First review

Speaking of which, I just got a notification for a review from [oklopfer](https://github.com/oklopfer)! Let's take a look at [the review](https://github.com/pacstall/pacstall-programs/pull/7112#discussion_r1974700877).

He has requested that we change our description to something:

> short, sweet, and straight to the point

So let's do that now. I will edit the pacscript in the branch and push the new change.

Now that I've [pushed the new change](https://github.com/pacstall/pacstall-programs/pull/7112/commits/bdbdcf2bd968849fe5428a2ab6b65bf8b73bb349), I will wait for another review.

### Second review

[Another notification](https://github.com/pacstall/pacstall-programs/pull/7112#issuecomment-2689640665)! This time it's about repology integration, which will help us keep this package up-to-date once this package is merged! As oklopfer already gave us a useable key, we can just use that, but for future reference, you can check <https://repology.org/projects/> and search for the project key.

Once again, [I have pushed](https://github.com/pacstall/pacstall-programs/pull/7112/commits/ea119d068fd07bc0c9a80db469ec89978e1e5082) and will await further reviews.

### Merge

Oklopfer has now merged the PR and now our package is in the repo and is now installable by running `pacstall -I soar` In the next post I will discuss how to deal with package updates.

---

[^1]: We still have standards for packaging, but I mean that there isn't anything extra we must do to package soar.
