---
title: Multiple Versions on Windows
icon: material/microsoft-windows
slug: multiple-versions-on-windows
tags:
  - development
  - windows
  - python
publish: false
---

_How to manage multiple Python versions on a Windows system._

## Using Pyenv for Windows

We can manage multiple Python versions on a system using [pyenv for Windows](https://github.com/pyenv-win/pyenv-win), which brings the same capabilities than on an Unix system. See [[Multiple versions on macOS]].

List the available Python installations

```shell 
pyenv install --list
```

Install a Python version

```shell
pyenv install 3.9.13
```

Set the global Python version

```shell
pyenv global 3.9.13
```

Set the Python version only for the current shell

```shell
pyenv shell 3.9.13
```

---

> [!error] Deprecated
> The following methods are deprecated in favor of [pyenv](#Using%20Pyenv%20for%20Windows).

## How-to

If you want to handle multiple versions, here's an example on how to use PIP to install the `mkdocs` plugin `exclude` for Python 3.7 and 3.9.

```powershell
& %LOCALAPPDATA%\programs\python\python37\python.exe -m pip install mkdocs-exclude
```

```powershell
& %LOCALAPPDATA%\programs\python\python39\python.exe -m pip install mkdocs-exclude
```

> [!note] Note
>  All you have to do is call the right `.exe` matching the right Python version.
>  The `&` is used to expand the environment variable `$USERNAME` in PowerShell.

### To Copy-paste

```powershell
& %LOCALAPPDATA%\programs\python\python27\python.exe
```

```powershell
& %LOCALAPPDATA%\programs\python\python37\python.exe
```

```powershell
& %LOCALAPPDATA%\programs\python\python39\python.exe
```

```powershell
& %LOCALAPPDATA%\programs\python\python311\python.exe
```

## Alternative Method

We can also define some batch scripts in a given directory. In the case we're using [Cloud VFX Server](https://github.com/healkeiser/cloud_vfx_server), we can rely on `$ENVIRONMENT_ROOT` (`C:\Users\valen\OneDrive\.config\environment`).

In `$ENVIRONMENT_ROOT`, create a `python` folder: `C:\Users\valen\OneDrive\.config\environment\python`.

You can now create a batch script for each version of python you want to use in this folder:

### `python39`

```batch
@ECHO OFF
setlocal
set "PYTHONHOME=%LOCALAPPDATA%\programs\python\python39"
set "PYTHONPATH=%PYTHONHOME%\lib;%PYTHONPATH%"
"%PYTHONHOME%\python.exe" %*
endlocal
```

> [!note] Note
>  We might be able to leverage other variables such as `$USERPROFILE` or `$APPDATA`.

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



