# Build Lua on Wine

## License

The same license of Lua ([https://lua.org/license.html](https://lua.org/license.html)).

## Overview

Motivated by the goal to provide a guide to build Lua on Windows in a POSIX environment, under the `amd64` architecture, we are going to show how it is possible to build Lua (5.4.8) on Wine ([https://www.winehq.org/](https://www.winehq.org/)). For this task, we are going to use either MinGW or MinGW-w64 binaries.

In the guides, you can choose to setup either the legacy 32-bit MinGW ([https://sourceforge.net/projects/mingw/](https://sourceforge.net/projects/mingw/)) or MinGW-w64 ([https://sourceforge.net/projects/mingw-w64/](https://sourceforge.net/projects/mingw-w64/)) on Wine.

If you want compatibility to the legacy MinGW project, then follow the guide [Setup MinGW to build Lua on Wine](./docs/Setup-MinGW.md). Otherwise, in case you want compatibility to the standard MinGW-w64 adopted nowadays on Windows, then look at [Setup MinGW-w64 to build Lua on Wine](./docs/Setup-MinGW-w64.md).

## Legacy 32-bit MinGW

Follow the tutorial: [Setup MinGW to build Lua on Wine](./docs/Setup-MinGW.md).

## Standard MinGW-w64

Currently, there are many projects providing MinGW-w64 binaries (see [https://www.mingw-w64.org/downloads/](https://www.mingw-w64.org/downloads/)). As a matter of choice that better suits our needs on Wine, we are going to use the binaries provided by [https://github.com/niXman/mingw-builds-binaries](https://github.com/niXman/mingw-builds-binaries).

> [!NOTE]
> 
> At the moment (2025-09-01), the MinGW-w64 binaries provided by [https://github.com/niXman/mingw-builds-binaries](https://github.com/niXman/mingw-builds-binaries) are deployed on GitHub Actions machines (see [here](https://github.com/actions/runner-images/blob/bfd23df81da886b5f5bb97936ea0ad02240e5b40/images/windows/scripts/build/Install-Mingw64.ps1#L54)).

Follow the guide: [Setup MinGW-w64 to build Lua on Wine](./docs/Setup-MinGW-w64.md).