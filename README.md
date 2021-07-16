# plumber

*A script that keeps dev dependencies at arm's length.*

`plumber` is a solution to the problem of developer cruft. Installing
toolchains, runtimes, and development dependencies directly onto a host system
can lead to rigidity, fear of experimentation, and broken development
environments that waste time and generate frustration. We know that
containerization is a good solution to this problem because it enables
repeatable environment setup and disposal, but for most of us the overhead cost
of dealing with Docker is too high to justify. `plumber` lowers that cost by
handling the container stuff with sane defaults so you can get back to your
actual work.

`plumber` doesn't cover all use cases and [really shouldn't anyway][0]. Its
main purpose is to reduce the amount of time developers spend standing up their
environments for projects with low or moderate complexity. If your project
depends on a few packages from `apt` or one or two language toolchains, then
`plumber` will probably work for you.

`plumber` doesn't do anything that you can't already do with Docker on your
own. It just makes all that stuff easier by giving it a nice UI.

[0]: https://en.wikipedia.org/wiki/Unix_philosophy

## Features

 - Runs the target software in a container so cruft doesn't accumulate
 - Runs on any system with Docker installed
 - Still lets you run your editor, version control, and other common tools on
   your host system
 - Shares your project's directory as a container volume, letting you operate
   on your project's files from inside
 - Automatically runs as UID 1000, so permission issues with volume files
   aren't a problem; the "you" on the outside is the "you" on the inside, too
 - To move between machines, just move `.plumber/` into your new working
   directory
 - To reclaim disk (and sanity), `docker image rm` the stuff you don't need
   anymore
 - Doesn't hide any Docker features, only provides affordances for the most
   common actions

## Installation and Usage

### Quickstart

To install `plumber`, you must first install [Docker][1]. Then, copy `plumber`
to a directory on your shell's PATH.

To get started quickly, the commands you need are:

 1. In your project directory, `plumber init <IMAGE NAME>` where `<IMAGE NAME>`
    is a Docker image (e.g. `debian:latest`).
 2. (optional) Run `plumber edit` to make changes to the Dockerfile, such as to
    install additional packages from `apt`. Save and run `plumber build`.
 3. Run `plumber start` to get a shell in the container. `/project` in the
    container is mapped to the working directory.
 4. Do whatever you need to do with your toolchain (e.g. build your code, run a
    dev server, etc.).
 5. Exit the container with CTRL+d, or get more shells with `plumber attach` in
    other terminals. Use `plumber attach root` if you need that.
 6. Stop the container with `plumber stop`.

[1]: https://docs.docker.com/engine/install/

### Usage Example

Let's say I want to work on `rust-bcrypt` but don't want to install Rust on my
host system. To use `plumber` with `rust-bcrypt`, start by cloning the repo on
your host machine as usual.

```bash
bradley@hostmachine ~/code> git clone https://github.com/Keats/rust-bcrypt.git
...
bradley@hostmachine ~/code> cd rust-bcrypt/
```

Now, set `plumber` up in this directory with `plumber init`. We need to tell
`plumber` what base image to start with. In this case, we want `rust:latest` so
we can use the Rust toolchain inside the container. You may specify any Docker
image here, either on [Docker Hub](https://hub.docker.com/) or any other
accessible image repository.

```bash
bradley@hostmachine ~/c/rust-bcrypt> plumber init rust:latest
```

`plumber` will download the image, set it up, and write a `.plumber/`
directory, which has its own `.gitignore` by default.

Now that plumber is set up, we just need to start the container.

```bash
bradley@hostmachine ~/c/rust-bcrypt> plumber start
```

Notice that your username stays the same, but your hostname changes to the
container's name, which is just the name of the parent directory. The
`/project` directory inside the container points to the working directory
outside the container. User permissions are the same between the container and
the host, so any changes you make inside the container look just like they
would if you made them on the host.

```bash
bradley@rust-bcrypt /code> ls -lha
total 56K
drwxr-xr-x 9 bradley bradley 4.0K Jul 16 21:41 ./
drwxr-xr-x 1 root    root    4.0K Jul 16 21:42 ../
-rw-r--r-- 1 bradley bradley  188 Jul 16 21:41 .editorconfig
drwxr-xr-x 8 bradley bradley 4.0K Jul 16 21:41 .git/
drwxr-xr-x 3 bradley bradley 4.0K Jul 16 21:41 .github/
-rw-r--r-- 1 bradley bradley   25 Jul 16 21:41 .gitignore
drwxr-xr-x 3 bradley bradley 4.0K Jul 16 21:41 .plumber/
-rw-r--r-- 1 bradley bradley  776 Jul 16 21:41 Cargo.toml
-rw-r--r-- 1 bradley bradley 1.1K Jul 16 21:41 LICENSE
-rw-r--r-- 1 bradley bradley 2.8K Jul 16 21:41 README.md
drwxr-xr-x 2 bradley bradley 4.0K Jul 16 21:41 benches/
drwxr-xr-x 2 bradley bradley 4.0K Jul 16 21:41 examples/
drwxr-xr-x 3 bradley bradley 4.0K Jul 16 21:41 fuzz/
drwxr-xr-x 2 bradley bradley 4.0K Jul 16 21:41 src/
```

Now we can do anything we like that involves Rust, and it will appear to our
host system as if we did everything in the usual way.

```bash
bradley@rust-bcrypt /code> cargo build
...
```

And you can still use your editor of choice on the host to make changes. `git`
still works fine, too. Only the toolchain is containerized, but that's enough
to prevent a lot of annoying cruft.

Press CTRL+d to detach from the container, or attach more terminals with
`plumber attach`. You can also attach as root with `plumber attach root`.

To stop the container, run `plumber stop`. You can also check to see whether or
not the container is running with `plumber status`.

Because `plumber` is just a wrapper around Docker, you can use normal Docker
tools to operate on `plumber` containers and images, including to clean them up
if you want some disk back.

## A Note on Stability

I've been using `plumber` on my own machine for a while now, and it doesn't
give me much trouble. Proceed with caution, though. I've only ever tested it on
Debian 10, so I don't know what it might do on other systems. If you run
`plumber` successfully on something interesting, let me know so I can brag
about it! In principle, it should run anywhere as long as Bash and Docker are
available.

## Why `fish` and not `bash`?

`plumber` is my personal project, and I use `fish`, so that's what `plumber`
uses. In the future I might try to add shell auto-detection, or at least some
sort of configuration option. It shouldn't be too hard to set up support for
other shells if you need it.

## License

Copyright 2021  Bradley Gannon

This file is part of `plumber`.

`plumber` is free software: you can redistribute it and/or modify
it under the terms of the GNU General Public License as published by
the Free Software Foundation, either version 3 of the License, or
(at your option) any later version.

`plumber` is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the
GNU General Public License for more details.

You should have received a copy of the GNU General Public License
along with `plumber`. If not, see <https://www.gnu.org/licenses/>.
