---
title: Distribute Package to PyPi
icon: simple/pypi
slug: distribute-package-to-pypi
tags:
  - development
  - python
  - package
publish: true
---

_How to distribute a Python package through PyPI._

## Setup Package

Create a `.pypirc` file under `%userprofile%`. Add your API key from PyPi.

``` ini title=".pypirc"
[distutils]
  index-servers = pypi

[pypi]
  repository: https://upload.pypi.org/legacy/
  username = __token__
  password = <API KEY>
```

Inside your Python package, create a `setup.py` file at the root. This file is taken from the [fxgui](https://github.com/healkeiser/fxgui) package.

``` python title="setup.py"
"""PyPI setup script."""

# Built-in
from pathlib import Path
from setuptools import setup, find_packages
import sys

# Metadata
__author__ = "Valentin Beaumont"
__email__ = "valentin.onze@gmail.com"


# Add `README.md` as project long description
this_directory = Path(__file__).parent
long_description = (this_directory / "README.md").read_text()

# Add required dependencies
install_requires = ["qtpy", "QtAwesome"]

setup(
    name="fxgui",
    version="2.3.1",
    long_description=long_description,
    long_description_content_type="text/markdown",
    description="Custom Python classes and utilities tailored for Qt built UI, in VFX-oriented DCC applications.",
    url="https://github.com/healkeiser/fxgui",
    author="Valentin Beaumont",
    author_email="valentin.onze@gmail.com",
    license="MIT",
    keywords="Qt PySide2 VFX DCC UI",
    packages=find_packages(),
    install_requires=install_requires,
    include_package_data=True,
)
```

> [!warning] 
> The above script works if you plan on **manually** release the package to PyPI. In the case where you have a GitHub action configured, you need to replace some lines as explained in the [Link Package Version to Git Tag](#Link%20Package%20Version%20to%20Git%20Tag) section.

You can also add a `MANIFEST.in` file, which specify additional files to include in the source distribution (sdist) that are not automatically included by default.

``` ini title="MANIFEST.in"
recursive-include fxgui *
global-exclude __pycache__/*
```

## Distribute Package

<font color=4287ff><b>Install <code>twine</code> library</b></font>

``` shell
python -m pip install twine
```

<font color=4287ff><b>Build package</b></font>

``` shell
python setup.py sdist bdist_wheel
```

<font color=4287ff><b>Upload package</b></font>

``` shell
python -m twine upload dist/* --skip-existing
```

<font color=4287ff><b>Build package locally</b></font>

``` shell
python -m pip install -e .
```

## CI/CD With GitHub Actions

Go to GitHub and add two repository secrets:

- `PYPI_USERNAME`: Your PyPI username.
- `PYPI_PASSWORD`: Your PyPI password or API token.

> [!warning]
> If using an API key, `PYPI_USERNAME` should be `__token__`.

Inside your local repository, Create a `.github/workflows` directory, and add a `publish.yml` inside.

``` yaml title="publish.yml"
name: Publish Python Package

on:
  push:
    tags:
      - 'v*.*.*'  # Trigger the workflow on version tags

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v2

    - name: Set up Python
      uses: actions/setup-python@v2
      with:
        python-version: '3.x'

    - name: Install dependencies
      run: |
        python -m pip install --upgrade pip
        pip install setuptools wheel twine

    - name: Build package
      run: |
        python setup.py sdist bdist_wheel

    - name: Publish package to PyPI
      env:
        TWINE_USERNAME: ${{ secrets.PYPI_USERNAME }}
        TWINE_PASSWORD: ${{ secrets.PYPI_PASSWORD }}
      run: |
        twine upload dist/*
```

You can now tag the release, which will trigger the package release on PyPI.

``` shell
git add .
git commit -m "Initial release"
git tag v1.0.0
git push origin v1.0.0
```

### Link Package Version to Git Tag

You can link the PyPI package version to the Git tag by installing `setuptools_scm`:

``` shell
python -m pip install setuptools_scm
```

And modify your `setup.py` accordingly:

``` python title="setup.py"
setup(
    name="fxgui",
    use_scm_version=True,
    setup_requires=["setuptools_scm"],
	...
```