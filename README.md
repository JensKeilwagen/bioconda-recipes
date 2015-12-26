# The bioconda channel

[![Build Status](https://travis-ci.org/bioconda/bioconda-recipes.svg?branch=master)](https://travis-ci.org/bioconda/bioconda-recipes)

[Conda](http://anaconda.org) is a platform- and language-independent package
manager that sports easy distribution, installation and version management of
software.  The [bioconda channel](https://anaconda.org/bioconda) is a Conda
channel providing bioinformatics related packages.  This repository hosts the
corresponding recipes.

## User guide

Please visit https://bioconda.github.io for details.

## Developer guide

If you want to contribute new packages to Bioconda, you are invited to join the
Bioconda team.  Please post in the [team thread on
GitHub](https://github.com/bioconda/recipes/issues/1) to ask for permission.

We build Linux packages inside a CentOS 5 docker container to maintain
compatibility across multiple systems. OSX packages are built using the [OSX
build environment](https://docs.travis-ci.com/user/osx-ci-environment/) on
[Travis CI](https://travis-ci.org/).

The steps below describe how to contribute a new package. The following
prerequisites are assumed:

- The [`conda`](http://anaconda.org) command line tool. This comes with the
  full [Anaconda scientific Python stack](https://www.continuum.io/downloads)
  installation, or the slimmed-down
  [Miniconda](http://conda.pydata.org/miniconda.html). The Python 3 version is
  recommended.
- [`docker`](https://www.docker.com/)
- [`git`](https://git-scm.com/)

### Step 1: Create a new recipe

Fork this repository or create a new branch to work in. Within the new branch,
[create a recipe](http://conda.pydata.org/docs/building/build.html)
(`your_package` in this example) in the `recipes` directory. See the "bioconda
build system" section below for further details.

### Step 2: Test the recipe

When the recipe is ready, first test it with your local conda installation via

    conda build recipes/your_package

If the recipe has dependencies in the bioconda channel (this is often the
case), you will need to add `--channel bioconda` to the command. If the recipe
is an R package, you will also need to add `--channel r`. For example many
Bioconductor packages will be built using:

    conda build recipes/your_package --channel bioconda --channel r

Then, you can test it in the docker container with:

    docker run -v `pwd`:/bioconda-recipes bioconda/bioconda-builder --packages your_package

To optionally build for a specific Python version, provide the `CONDA_PY`
environmental variable. For example, to build specifically for Python 3.4:

    docker run -e CONDA_PY=34 -v `pwd`:/bioconda-recipes bioconda/bioconda-builder --packages your_package

To optionally build and test all packages (if they don't already exist), leave off the
package name:

    docker run -v `pwd`:/tmp/conda-recipes bioconda/bioconda-builder

If rebuilding a previously-built package and the version number hasn't changed,
be sure to increment the build number in `meta.yaml` (the default build number
is 0):

    build:
      number: 1

### Step 3: Submit a pull request

Once these local tests pass, submit a [pull
request](https://help.github.com/articles/using-pull-requests) to this
repository. The [travis-ci](https://travis-ci.org) continuous
integration service will automatically test the pull request.

When the PR tests pass, the PR can be merged into the master branch.

Travis-CI will again run tests. However, when testing the master branch, new,
successfully-built packages will be uploaded to the `bioconda` conda channel.
Once these tests pass, your new package can now be installed from anywhere
using:

    conda install -c bioconda your_package

### Step 4:

If you want to promote the Bioconda installation of your package, we recommend
to add the following badge to your homepage:

    [![bioconda-badge](https://img.shields.io/badge/install%20with-bioconda-brightgreen.svg?style=flat-square)](http://bioconda.github.io)

This will display as
[![bioconda-badge](https://img.shields.io/badge/install%20with-bioconda-brightgreen.svg?style=flat-square)](https://bioconda.github.io).
For other styles, replace ``?style=flat-square`` with ``?style=flat`` or
``?style=plastic``.

### Building packages for Mac OSX

If you want your package to be built for Mac OSX as well, add the recipe name
to the ``osx-whitelist.txt`` file in the root of this repository.

To set up a local machine for building and testing OSX recipes, run
``scripts/travis-setup.sh``. Several commands in this script will prompt for
sudo privileges, but the script itself should be run as a regular user. This
script will set up a conda environment in ``/anaconda`` and install necessary
prerequisites.

To test all recipes in the ``osx-whitelist``, use:

```
scripts/build-packages.py --repository . --packages `cat osx-whitelist.txt`
```


### Managing multiple versions of a package

If there is interest to keep multiple versions of a package or to explicitly
build an older version of a package, you can store those versions in
subdirectories of the corresponding recipe, e.g.:

```
java-jdk/
├── 7.0.91
│   ├── build.sh
│   └── meta.yaml
├── build.sh
└── meta.yaml
```

There should always be a primary in the root directory of a package that is
updated when new releases are made.

### Other notes

We use a pre-built CentOS 5 image with compilers installed as part of the
standard build. To build this yourself, you can do:

    docker login
    cd scripts && docker build -t bicoonda/bioconda-builder .

### Creating Bioconductor recipes

See [`scripts/bioconductor/README.md`](scripts/bioconductor/README.md) for
details on creating and updating Bioconductor recipes.

### Creating and updating UCSC tools

See [`scripts/ucsc/README.md`](scripts/ucsc/README.md) for details on creating
and updating recipes for UCSC programs.
