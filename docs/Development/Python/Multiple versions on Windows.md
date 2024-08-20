---
title: Multiple Versions on Windows
icon: material/microsoft-windows
slug: multiple-versions-on-windows
tags:
  - development
  - windows
  - python
publish: true
---

_How to manage multiple Python versions on a Windows system._

## Using Pyenv for Windows

We can manage multiple Python versions on a system using [pyenv for Windows](https://github.com/pyenv-win/pyenv-win), which brings the same capabilities than on an Unix system. See [[Multiple versions on macOS]].

<font color=4287ff><b>List the available Python installations</b></font>

```shell 
pyenv install --list
```

<font color=4287ff><b>Install a Python version</b></font>

```shell
pyenv install 3.9.13
```

<font color=4287ff><b>Set the global Python version</b></font>

```shell
pyenv global 3.9.13
```

<font color=4287ff><b>Set the Python version only for the current shell </b></font>

```shell
pyenv shell 3.9.13
```

---

> [!error] Deprecated
> The following method is deprecated in favor of [pyenv](#Using%20Pyenv%20for%20Windows).

## How-to

We can define some batch scripts in a given directory. In the case we're using [Cloud VFX Server](https://github.com/healkeiser/cloud_vfx_server), we can rely on `$ENVIRONMENT_ROOT` (which expand to `C:\Users\valen\OneDrive\.config\environment` in my case).

In `$ENVIRONMENT_ROOT`, create a `python` folder: `C:\Users\valen\OneDrive\.config\environment\python`.

You can now create a batch script for each version of python you want to use in this folder:

 `python39.bat`

```batch
@ECHO OFF
setlocal
set "PYTHONHOME=%LOCALAPPDATA%\programs\python\python39"
set "PYTHONPATH=%PYTHONHOME%\lib;%PYTHONPATH%"
"%PYTHONHOME%\python.exe" %*
endlocal
```

Finally, edit your `$PATH` environment variable, adding as a **first entry** the `%ENVIRONMENT_ROOT%\python` folder, so the batch files are accessible inside a terminal.

### Pip

You can now use `pip` using this method, by typing:

```shell
python39 -m pip <command>
```

```shell
python39 -m pip install PySide2
```



