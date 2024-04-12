# Federated Learning Platform Documentation

This repository contains the official documentation of the Federated Learning platform.
The whole documentation is written in markdown and rendered with [MkDocs](https://www.mkdocs.org) and its [material theme](https://squidfunk.github.io/mkdocs-material).

The Federated Learning (FL) platform is serving as a proof of concept for the [Catena-X](https://catena-x.net/en) project.
The FL platform aims to demonstrate the potential of federated learning in a practical, real-world context.

A complete list of all repositories relevant to the FL platform can be found [here](https://dlr-ki.github.io/fl-documentation#repositories).

## Get Started

This README.md is primarily intended for developers and contributors, providing necessary information for setup, installation, and contribution guidelines.
If you're interested in using or testing this project, we recommend starting with the [GitHub pages](https://dlr-ki.github.io/fl-documentation) which serves this documentation.
They offer a more user-friendly interface and comprehensive guides to get you started.

## Requirements

- python 3.7 or later  
  `which python`
- virtualenv or venv  
  `pip install -U virtualenv`

## Install

```bash
# create virtual environment
virtualenv -p $(which python3.7) .venv
# or
# python -m venv .venv

# activate our virtual environment
source .venv/bin/activate

# update pip (optional)
python -m pip install -U pip

# install
./dev install -U -e ".[all]"

# start the server (optional)
./dev start 
```

## Helpers

```txt
$ ./dev --help
usage: ./dev <action>

positional arguments:
  {clean,doc,doc-build,help,install,licenses,licenses-check,lint,lint-doc,lint-scripts,safety-check,start,test,version,versions}
                        Available sub commands
    help                Show this help message and exit
    start               Start documentation server
    test                Run all tests
    lint                Run all linter
    lint-doc            Run documentation linter
    lint-scripts        Run bash script linter
    doc                 Start documentation server
    doc-build           Build documentation
    licenses            Generate licenses
    licenses-check      Check licenses
    safety-check        Check dependencies for known security vulnerabilities
    install             Install package
    clean               Clean up local files
    version             Show package version
    versions            Show versions
```
