---
title: Custom Schemas
icon: material/toy-brick
slug: custom-schemas
publish: true
---

_How-to create a USD custom schema._

> [!quote] Important Links
> - [USD Survival Guide - Schemas](https://lucascheller.github.io/VFX-UsdSurvivalGuide/core/plugins/schemas.html)
> - [USD - Generating New Schema Classes](https://openusd.org/release/tut_generating_new_schema.html)

> [!note]
> You can find the **usdFxquinox** plugin on [GitHub](https://github.com/healkeiser/fxquinox/tree/main/plugins/usd/usdFxquinox).

The easiest way to create a custom schema is to create what’s called a codeless schema (uncompiled version). This will create a new schema **but won’t provide the C++/Python bindings** from the freshly created schema class.

## Directory Structure

Let’s start by creating a directory and a `schema.usda` file:

```
.
└── 📁 usdFxquinox/
    └── 📁 resources/
	    └── 📄 schema.usda
```

Which corresponds to:

```
.
└── 📁 <plugin name>/
    └── 📁 resources/
	    └── 📄 schema.usda
```

## Edit `schema.usda`

Let’s now edit our `schema.usda` file. This will contain the **IsA schemas** or **API schemas** you want to create.

``` c++ title="schema.usda"
#usda 1.0
(
    subLayers = [
        @usd/schema.usda@
    ]
)

over "GLOBAL" (
    customData = {
        string libraryName       = "usdFxquinox"
        string libraryPath       = "."
        string libraryPrefix     = "usdFxquinox"
        bool skipCodeGeneration = true
    }
)
{
}

class FxquinoxContextInfo "FxquinoxContextInfo" (
    doc = """Holder for fxquinox-specific context information."""
    inherits = </Typed>
    customData = {
        string className = "FxquinoxContextInfo"
        }
    )
{
    string fxquinox:project (
        doc = """The project name."""
    )

    asset fxquinox:projectRoot (
        doc = """The project root path."""
    )

    string fxquinox:entity (
        doc = """The current context entity (asset or shot)."""
    )

    string fxquinox:assetType (
        doc = """If the entity is an asset, the type of asset."""
    )

    string fxquinox:asset (
        doc = """If the entity is an asset, the asset."""
    )

    string fxquinox:sequence (
        doc = """If the entity is a shot, the sequence."""
    )

    string fxquinox:shot (
        doc = """If the entity is a shot, the shot."""
    )

    string fxquinox:step (
        doc = """The context pipeline step."""
    )

    string fxquinox:task (
        doc = """The context pipeline task."""
    )

    string fxquinox:user (
        doc = """The current username."""
    )

    string fxquinox:hostname (
        doc = """The current hostname (machine name)."""
    )
}
```

## Run usdGenSchema

Once you’re happy with your new schema, we now need to run **usdGenSchema.** The easiest way is to run the one shipping with your DCC compiled USD.

I made this batch script to run it on Windows:

``` shell title=".run_usdGenSchema.bat"
@echo off

REM Set paths to Houdini programs, and the USD file
set HOUDINI_VERSION=20.0.547
set USD_PLUGIN_NAME=usdFxquinox
set HYTHON_PATH="C:\Program Files\Side Effects Software\Houdini %HOUDINI_VERSION%\bin\hython.exe"
set USDGENSCHEMA_PATH="C:\Program Files\Side Effects Software\Houdini %HOUDINI_VERSION%\bin\usdGenSchema"
set SCHEMA_PATH=%~dp0resources\schema.usda
set DESTINATION_PATH=%~dp0

echo Running usdGenSchema for %USD_PLUGIN_NAME%:
echo           Hython: %HYTHON_PATH%
echo       USD plugin: %USD_PLUGIN_NAME%
echo     UsdGenSchema: %USDGENSCHEMA_PATH%
echo           Schema: %SCHEMA_PATH%
echo             Dest: %DESTINATION_PATH%
echo.

REM Run the command `hython usdGenSchema 'path/to/schema.usda' 'destination/path'`
%HYTHON_PATH% %USDGENSCHEMA_PATH% %SCHEMA_PATH% %DESTINATION_PATH%

pause
```

> [!note]
> Think about changing your `HOUDINI_VERSION` and `USD_PLUGIN_NAME` accordingly.

> [!warning]
> `%~dp0` is the Batch variable that expands to the current directory. Only use it if your .bat file is placed under the plugin folder: `usdFxquinox/.run_usdGenSchema.bat`

Let the tool run. Once it’s done, you should have 2 new files:

``` hl_lines="5 6"
.
└── 📁 usdFxquinox/
    ├── 📁 resources/
	│   └── 📄 schema.usda
    ├── 📄 generatedSchema.usda    🟢
    └── 📄 plugInfo.json           🟢
```

You’ll need to modify the `plugInfo.json` file, since usdGenSchema leaves in the cmake `@...@` string replacements:

``` json title="plugInfo.json" hl_lines="2 4 5"
...
    "LibraryPath": "@PLUG_INFO_LIBRARY_PATH@",
    "Name": "usdFxquinox",
    "ResourcePath": "@PLUG_INFO_RESOURCE_PATH@",
    "Root": "@PLUG_INFO_ROOT@",
    "Type": "resource"
...
```

To:

``` json title="plugInfo.json" hl_lines="2 4 5"
...
	"LibraryPath": "",
	"Name": "usdFxquinox",
	"ResourcePath": ".",
	"Root": ".",
	"Type": "resource"
...
```

This change only need to be made once, as indicated in `plugInfo.json`:

```
# Edits will survive regeneration except for comments and
# changes to types with autoGenerated=true.
```

## Edit Environment

The last step is to add the folder containing `plugInfo.json` to the `PXR_PLUGINPATH_NAME` environment variable. In our instance:

```shell
set PXR_PLUGINPATH_NAME=%SERVER_ROOT%/Projects/Code/fxquinox/plugins/usd/usdFxquinox;%PXR_PLUGINPATH_NAME%
```

