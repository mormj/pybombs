# PyBOMBS

Website: https://www.gnuradio.org/blog/pybombs-the-what-the-how-and-the-why

Minimum Python version: 2.7

[![Build Status](https://travis-ci.org/gnuradio/pybombs.svg?branch=master)](https://travis-ci.org/gnuradio/pybombs)

## License

Copyright 2015 Free Software Foundation, Inc.

This file is part of PyBOMBS

PyBOMBS is free software; you can redistribute it and/or modify
it under the terms of the GNU General Public License as published by
the Free Software Foundation; either version 3, or (at your option)
any later version.

PyBOMBS is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
GNU General Public License for more details.

You should have received a copy of the GNU General Public License
along with PyBOMBS; see the file COPYING.  If not, write to
the Free Software Foundation, Inc., 51 Franklin Street,
Boston, MA 02110-1301, USA.

## Installation

### Through pip

You don't have to clone this repository if you don't want to contribute to PyBOMBS itself.
In that case, simply run

    $ [sudo] pip install PyBOMBS

and it will download and install PyBOMBS for you. Note that this usually doesn't install the latest HEAD, but only the latest version that was submitted to PyPI, so not every bugfix is automatically always immediately propagated that way.

If you do want to install the latest version from git, but still use pip, you
run the following command:

    $ [sudo] pip install [--upgrade] git+https://github.com/gnuradio/pybombs.git

### From source using Python's setuptools

PyBOMBS can be installed using Python's setuptools. From the top
level of the source code repository, run

    $ python setup.py build

or

    $ sudo python setup.py install

This will install PyBOMBS and all required dependencies. See

    $ python setup.py build --help
    $ python setup.py install --help

for additional settings.

pip also provides a `-e` switch for installing PyBOMBS in 'editable' mode.

### Install it all manually

If you want to install PyBOMBS yourself, you need to make sure the `pybombs` module is in the PYTHONPATH. To run PyBOMBS in this case, execute `main.py`. You can symlink or alias that to `pybombs` (e.g. `ln -s /path/to/pybombs/main.py ~/bin/pybombs`). If you don't know what any of this means, please use one of the methods explained further up.

## Quickstart

For the impatient:

1. Install PyBOMBS as per the previous section
2. Apply a configuration:

        $ pybombs auto-config

3. Add a list of recipes, e.g., the default recipes:

        $ pybombs recipes add-defaults

4. Install GNU Radio, gr-osmosdr and some other goodies into your home directory `~/prefix`:

        $ pybombs prefix init ~/prefix -a myprefix -R gnuradio-default

   All commands after this will use `myprefix` as the default prefix. You can change the
   default prefix later by running `pybombs config default_prefix NEWPREFIX`

5. Run GNU Radio Companion from your new prefix:

        $ source ~/prefix/setup_env.sh
        $ gnuradio-companion

   or execute it without changing the current environment:

        $ pybombs run gnuradio-companion

## Platform-specific instructions

Some platforms have their own idiosyncrasies. Platforms that require special
attention are listed here:

- [CentOS](centos.md)
- [Debian](debian.md)
- [OpenSUSE](opensuse.md)

### Testing specific platforms

For testing distributions, PyBOMBS uses Docker containers. To make the
statement "PyBOMBS is fully functional on platform XYZ", a specific container
for said platform is run using the test framework. For example, to test
Fedora 26, the following command can be run:

    $ ./run-tests.sh --skip-pylint --container=fedora26

The Dockerfiles are stored in `tests/docker/*`. It is easy to add containers
to include more distributions, and submitting new Dockerfiles is heavily
encouraged. Note that distributions can have subtle differences, and the
Dockerfiles can be used as a reference on how exactly to set up PyBOMBS on a
specific distribution.

If the tests pass, a distribution is usually considered functional. This does
not mean that everything will automatically work on someone's computer running
this distribution, but that's usually because local settings weren't taken into
account.

## Prefixes

A prefix is a directory into which packages are installed.

The prefix may be `~/prefix` as in the example above, and typically, the
prefix resides inside your home directory so you can modify or delete prefixes
easily without admin access. This is the recommended way of running PyBOMBS.
It can also be `/usr/local/` for system-wide installation of packages.
Any directory may be a prefix, but it is highly recommended
to choose a dedicated directory for this purpose.

Many developers have multiple prefixes. Instead if installing to `~/prefix`, a
common way is to have multiple prefixes, e.g., `~/prefix/default_prefix`,
`~/prefix/dev_prefix`, etc.

Prefixes require a configuration directory to function properly.
Typically, it is called `.pybombs/` and is a subdirectory of the prefix.
So, if your prefix is `~/prefix`, there will be a directory called
`~/prefix/.pybombs/` containing special files. The two most important
files are the inventory file (inventory.yml) and the prefix-local
configuration file (config.yml), but it can also contain recipe files
that are specific to this prefix.

There is no limit to the number of prefixes. Indeed, it may make sense
to have many prefixes, e.g. one for system-wide installation, one for
a user-specific installation, and one for cross-compiling to a different
platform.

When running PyBOMBS, you select the desired prefix using the `-p` switch.
You can set a default prefix with the following command:

    $ pybombs config default_prefix PREFIXNAME

The first time you run `pybombs prefix init`, it will set this value for you.

### Aliases

In order to make prefix selection more easy, it is possible to assign names
to prefixes by adding a `[prefix_aliases]` section to a configuration file.
The format is `alias=/path/to/prefix`. Instead of providing the entire path
every time, the alias can be used instead. When running `pybombs prefix init`,
you can use the `--alias` argument to set this automatically.

### Prefix Selection

Prefixes are selected by the following rules, in this order:

1. Whatever is provided by the `-p` or `--prefix` command line switch
2. The current directory
3. The default prefix as defined by the `default_prefix` config switch

If no prefix can be found, most PyBOMBS operations will not be possible,
but some will still work (for example, you can install all dependencies for a
package from binary sources).

### Initializing Prefixes

Any directory can function as a prefix, and PyBOMBS will make sure all the
required files and directories are created. However, PyBOMBS provides a way
to initialize a directory to be a full PyBOMBS prefix:

    $ pybombs prefix init /path/to/prefix [-a alias]

This is similar to `git init`. The optional alias allows you to access the
prefix with the alias instead of the full path. A typical value for the default
prefix is `~/prefix/default`, and then other prefixes also reside in `~/prefix`
alongside the default prefix.

After initializing a prefix, you can start to install to this prefix using the
install command:

    $ pybombs -p <alias> install <package>

PyBOMBS provides a way to not only initialize a raw prefix, but also configure
it and install packages through a *prefix recipe*. These are selected using
the `-R` switch on the command line:

    $ pybombs prefix init /path/to/prefix [-a alias] [-R prefix-recipe]


### Configuring a prefix' environment (e.g. for cross-compiling)

#### Setting environment variables directly:

For a quick setup of environment variables, you can use the `pybombs config`
command:

    $ pybombs config --env CC clang
    $ pybombs prefix env
    # ...lots of output...
    CC=clang
    # ...lots of output...

This will, by default, set an environment variable for *all* prefixes. You
might want to set it for a specific one, in that case, specify the prefix:

    $ pybombs -p default config --env CC clang

You can also edit the config files directly.  In any config file that is read,
a `env:` section can be added. This will set environment variables for any
command  (configure, build, make...) that is run within PyBOMBS.

Note that this will still use the regular system environment as well, but
it will overwrite existing variables. Variable expansion can be used, so
this will keep the original setting:

```yaml
env:
    LD_LIBRARY_PATH: ${LD_LIBRARY_PATH}:/path/to/more/libs
```

Note: Because this is a YAML file, remember to separate key/value pairs with
colon (:), not an equals sign, as you would in a shell script.

In all cases, the environment variable `PYBOMBS_PREFIX` is set to the
current prefix, and `PYBOMBS_PREFIX_SRC` is set to the source directory.

Use `pybombs prefix env` to show all environment variables as they would appear
when commands are run inside the prefix.

#### Using an external script to set the environment

Inside the config section, a shell script can be defined that sets up an
environment, which will then be used for commands running inside this prefix.

Example:

```yaml
config:
    # Other vars
    setup_env: /path/to/environment-setup-armv7ahf-vfp-neon-oe-linux-gnueabi
```

In this case, the environment from the calling shell session is *not* inherited.

### Python version

PyBOMBS itself is developed to work with both Python 2 and 3. However, the
Python version is not a one-dimensional problem: The prefix could be running a
different version of Python than the system. A common use case for this is when
the system's default version is Python 3, but a prefix was created to test
software with Python 2 (or vice versa). In this case, the PyBOMBS executable
will be executed by a different Python interpreter than will be used to run
installed scripts.

To define the Python version of the prefix, there are multiple options:
- If the prefix is also a Python virtualenv, the Python version is autodetected.
- Use the `python_ver` prefix configuration option.

The Python version of the prefix will default to the interpreter version running
the PyBOMBS script.

## Installing packages

When you run a command such as

    $ pybombs install gnuradio

PyBOMBS will initiate an installation procedure for the package. Since PyBOMBS
can interact both with the system's package manager (e.g., apt, dnf, brew...)
and install source packages, there need to be clearly defined rules about the
order of operations. PyBOMBS will attempt to install packages following these
rules:

1. Is the package flagged explicitly for source installation? If so, put it
   into a queue for source package builds. Otherwise, attempt to install it
   from the system's package manager. If that fails, also put it into the queue
   for source package builds.
2. Any package that is flagged for building from source is analysed to find the
   source build dependencies. For all of those packages, the same procedure is
   applied.
3. Eventually, all binary installs are complete and the source installs are
   left. The source packages are put into a tree, so they can be installed
   starting at the lowest dependee.

Example: In the command above, the package gnuradio has two dependencies, `uhd`
and `boost`. In a configuration file, uhd and gnuradio are flagged for building
from source. PyBOMBS will put gnuradio and uhd into the source build queue, and
then invoke the system's package manager to install boost.
When the package boost is installed, PyBOMBS generates a source build tree,
which in this case is a very simple tree: `gnuradio <- uhd`, meaning that the
package gnuradio depends on the package uhd, so uhd needs to be installed first.
PyBOMBS then executes a source build of uhd, then gnuradio (in that order) into
the prefix.

(Note: The actual dependency structure for those packages is more complex and
was simplified for this document).

## Recipes

### Recipe Format

Recipes files are in YAML format. To write new recipes, the easiest way is to
use other recipes as examples.

Important keys in the recipe files include:

- `inherit`: This will load the values from a template file (`*.lwt`) before
  using the values from the recipe, to set up suitable defaults.
- `category`: Can technically be anything, but certain categories carry certain
  meanings. In most cases, choose 'common'.

Example:


```yaml
# This means the build/install works like any other cmake project:
inherit: cmake
# These dependencies are only for source builds:
depends:
- boost
- fftw
- cppunit
# There's more dependencies in the real repo, skipping for this example
description: Free and open source toolkit for software defined radio
category: common
# Its good practice to add one of these for all the installers we have:
satisfy:
  deb: gnuradio-dev
  pacman: gnuradio
  port: gnuradio
  portage: net-wireless/gnuradio
source: git+https://github.com/gnuradio/gnuradio.git
# master is the default branch, but you can choose a different branch or tag here:
gitbranch: master
# Instead of a branch, you can also specify any commit:
#gitrev: 012345abc
# Another way to specify a commit is to append a rev, tag, or commit has to the
# source URL (git+https://.../gnuradio.git@abcd1234)
# Only when cloning the source code is this used, in that case, these args are
# appended to the git command that does the clone:
gitargs: --recursive
# Variables defined here can be used in various places in this recipe:
vars:
  config_opt: " -DENABLE_DOXYGEN=$builddocs "
# For static builds, we need to override the defaults from the cmake.lwt recipe:
configure_static: cmake .. -DCMAKE_BUILD_TYPE=$cmakebuildtype -DCMAKE_INSTALL_PREFIX=$prefix -DENABLE_STATIC_LIBS=True $config_opt
```

### Recipe Management

Recipes can be stored in multiple locations, which easily allows to store
separate recipe lists for specific projects.

If the same recipe can be found in more than one location, it will be
chosen from the most specific. The precise order is (from more to less
specific):
- Recipe locations specified on the command line (Using the `-r` switch)
- From the environment variable `PYBOMBS_RECIPE_DIR`
- The current prefix (if available)
- Global recipe locations

The command

    $ pybombs recipes list-repos

will show the recipe locations in the order they're used (it will pick a recipe
from the top line before it'll pick it from the bottom line).

This mechanism can be used to override recipes for certain prefixes. For
example, the `gnuradio.lwr` file could be copied and adapted to use a
different branch than the default recipe does. (Note that specific parts
of recipes can also be overridden in the config.yml file, in the [packages]
section).

Recipe management can be mostly done through the command line using
the `pybombs recipes` command -- editing configuration files is possible,
but often not necessary. Run

    $ pybombs help recipes

for further information on the `pybombs recipes` command.

#### Remote and Local Recipe Locations

Recipe locations can be either local directories (in this case, PyBOMBS will
simply read any .lwr file from this directory, *without* traversing into
subdirectories), or a remote location.
Remote locations can be:
- git repositories
- Remotely stored .tar.gz archives

Remote locations are copied into a local directory, so PyBOMBS can read the .lwr
files locally. During normal operations, PyBOMBS will not try to read the remote
location, so offline usage is still possible.
This local cache of recipes is stored in the same directory as the location of
the corresponding config file (e.g., if `~/.pybombs/config.yml` declares a
recipe called 'myrecipes', the local cache will be in
`~/.pybombs/recipes/myrecipes`).

## Configuration Files

Typically, there are four ways to configure PyBOMBS:

1. The global configuration file (e.g. `/etc/pybombs/config.yml`)
2. The user-local configuration file (e.g. `~/.pybombs/config.yml`)
3. The prefix-local configuration file (e.g. `~/src/prefix/.pybombs/config.yml`)
4. By using the `--config` switch on the command line

Higher numbers mean higher priority. Conflicting options are resolved by
choosing option values with higher priority.

### `config.yml` File Format

The config.yml files are in YAML format. A typical file looks like this:

```yaml
# All configuration options:
# (Run `pybombs config` to learn which options are recognized)
# You can edit these with `pybombs config` too
config:
    default_prefix: default
    makewidth: 8 # Run on 8 cores
    # ... more options

# Prefix aliases:
prefix_aliases:
    default: /home/user/src/pb-prefix/
    sys: /usr/local
    # pybombs prefix init -a <alias> will add one automatically here

# Prefix configuration directories:
prefix_config_dir:
    sys: /home/user/pb-default/
    # Typically, you don't need this, because the prefix configuration
    # directory is in <PREFIX>/.pybombs

# Recipe locations:
recipes:
    myrecipes: /usr/local/share/recipes
    morerecipes: /home/user/pb-recipes
    remoterecipes: git+git://url/to/repo
    # You wouldn't usually hand-edit this section, but use 'pybombs recipes' to
    # manipulate it

# Package flags:
packages:
    gnuradio:
        forcebuild: True  # This will skip any packagers for this package
                          # and use a source build
        forceinstalled: False  # 'True' will always assume this package is
                               # installed and skip installing it
        # Any other option here will override whatever's in the
        # corresponding recipe (in this case, gnuradio.lwr)
        # You can set these with pybombs config --package gnuradio forcebuild False

# Like package flags, but applies flags to all packages
# in a certain category. 'common' is all OOTs.
categories:
    common:
        forcebuild: True  # This would force source builds for any package in the
                          # `common` category
        # Still works via pybombs config --category common forcebuild True

# Environment variables
env:
    LD_LIBRARY_PATH: ${LD_LIBRARY_PATH}:/path/to/more/libs
    # You can also do pybombs config --env CC clang
```

## git

Many packages specify git source repositories. Because there's a lot of git
interaction, pybombs has some tools to make your life working with git easier.

### git cache (reference repository)

If you use PyBOMBS a lot, and have many prefixes with similar content, you'll
be cloning the same repos over and over again. You can set up a git cache, or
reference repository, to locally store objects and hence reduce clone times.

The simplest way to set this up is to run

    $ pybombs git make-ref

It will create the reference repository (which will then be used in subsequent)
clones, and configure your PyBOMBS accordingly. See

    $ pybombs git make-ref --help

for more use cases.

If you already have a reference repository elsewhere, simply point PyBOMBS to
it:

    $ pybombs config git-cache /path/to/ref

