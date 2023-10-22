#/python #/development 

## How-to

If you want to handle multiple versions, here's an example on how to use PIP to intall the `mkdocs` plugin `exclude` for Python 3.7 and 3.9.

```powershell
& c:\users\$env:USERNAME\appdata\local\programs\python\python37\python.exe -m pip install mkdocs-exclude
```

```powershell
& c:\users\$env:USERNAME\appdata\local\programs\python\python39\python.exe -m pip install mkdocs-exclude
```

> [!note] Note 
>  All you have to do is call the right `.exe` matching the right Python version.
>  The `&` is used to expand the environment variable `$USERNAME` in PowerShell.

### To Copy-paste
```powershell
& c:\users\$env:USERNAME\appdata\local\programs\python\python27\python.exe
```

```powershell
& c:\users\$env:USERNAME\appdata\local\programs\python\python37\python.exe
```

```powershell
& c:\users\$env:USERNAME\appdata\local\programs\python\python39\python.exe
```

```powershell
& c:\users\$env:USERNAME\appdata\local\programs\python\python311\python.exe
```

## Clean Method

We can also define some batch scripts in a given directory. In the case we're using [Cloud VFX Server](https://github.com/healkeiser/cloud_vfx_server), we can rely on `$ENVIRONMENT_ROOT` (`C:\Users\valen\OneDrive\.config\environment`).

In `$ENVIRONMENT_ROOT`, create a `python` folder: `C:\Users\valen\OneDrive\.config\environment\python`. 

You can now create a batch script for each version of python you want to use in this folder:
### `python39`

```batch
@ECHO OFF
setlocal
set "PYTHONHOME=C:\Users\%USERNAME%\appdata\local\programs\python\python39"
set "PYTHONPATH=%PYTHONHOME%\lib;%PYTHONPATH%"
"%PYTHONHOME%\python.exe" %*
endlocal
```

Finally, edit your `$PATH` environment variable, adding as a **first entry** the `%ENVIRONMENT_ROOT%\python` folder, so the batch files are accessible inside a terminal.

You can now test to type `python39` in your terminal to see if everything works as expected: 

![[terminal_python_001.png]]

You can also use those batch scripts to execute a Python program: 

![[terminal_python_002.png]]

Here's the result using both Python 3.9 and 3.11:

![[terminal_python_003.png]]