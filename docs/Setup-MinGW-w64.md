# Setup MinGW-w64 to build Lua on Wine

In a machine running Debian (current stable, `trixie`), this guide shows how to setup MinGW ([https://sourceforge.net/projects/mingw/](https://sourceforge.net/projects/mingw/)) on Wine to build Lua from source.

> [!IMPORTANT]
> 
> This tutorial was written in a system with `amd64` architecture. Other architectures were not tested.

Currently, there are many projects providing MinGW-w64 binaries (see [https://www.mingw-w64.org/downloads/](https://www.mingw-w64.org/downloads/)). As a matter of choice that better suits our needs on Wine, we are going to use the binaries provided by [https://github.com/niXman/mingw-builds-binaries](https://github.com/niXman/mingw-builds-binaries).

> [!NOTE]
> 
> At the moment (2025-09-01), the MinGW-w64 binaries provided by [https://github.com/niXman/mingw-builds-binaries](https://github.com/niXman/mingw-builds-binaries) are deployed on GitHub Actions machines (see [here](https://github.com/actions/runner-images/blob/bfd23df81da886b5f5bb97936ea0ad02240e5b40/images/windows/scripts/build/Install-Mingw64.ps1#L54)).

## Table of Contents

* [Prerequisites](#prerequisites)
* [Install Wine](#install-wine)
* [Configure Wine for Lua and MinGW-w64](#configure-wine-for-lua-and-mingw-w64)
    * [Initial Wine configuration](#initial-wine-configuration)
    * [Create a symbolic link pointing to Lua development directory](#create-a-symbolic-link-pointing-to-lua-development-directory)
    * [Set permanent helper environment variables on Wine](#set-permanent-helper-environment-variables-on-wine)
* [Setup MinGW-w64 on Wine](#setup-mingw-w64-on-wine)
* [Build and test Lua on Wine](#build-and-test-lua-on-wine)

## Prerequisites

To follow this guide, we need:

* basic skills to use the terminal;
* internet connection.

Moreover, we are going to employ well-known softwares:

* `wget` to download `MinGW-w64`;
* `p7zip` to extract `MinGW-w64` archives compressed in `7z` format.

Although `wget` comes preinstalled on a wide range of Unix-like distributions, `p7zip` does not. Thus, install them:

```bash
sudo apt install -y wget p7zip
```

## Install Wine

1. To run 32-bit applications, Wine needs `i386` packages. Then, add the `i386` architecture:

    ```bash
    sudo dpkg --add-architecture i386
    ```

2. Update list of available packages:

    ```bash
    sudo apt update
    ```

3. Finally, install the packages:

    ```bash
    sudo apt install -y wine wine32 wine64
    ```

    <img width="1036" height="653" alt="01-install-wine" src="https://github.com/user-attachments/assets/93de7486-1c63-4cf1-bca7-3d8c4ac951a2" />

## Configure Wine for Lua and MinGW-w64

### Initial Wine configuration

> [!NOTE]
> 
> This guide assumes that the reader has an entry-level knowledge about Windows. Thus, we install Wine in a separate directory under `$HOME` customized to have `MinGW-w64` tools available in the `PATH` environment variable, not interfering with other toolchains. Feel free to adapt this guide to your skill level.

1. Since we are going to install the *whole* Wine-MinGW-w64 tools in a separate directory, I highly recommend you to set a permanent shell variable on your system to hold the full path for such directory (*you may need to replace `$HOME/.bashrc` below by the shell file, in case it is not `bash`*):

    ```bash
    echo 'export WINE_MINGW_W64=$HOME/wine-MinGW-w64' >> $HOME/.bashrc
    ```

2. Reload the variables on your current shell:

    ```bash
    source $HOME/.bashrc
    ```

3. Test the assigned variable:

    ```bash
    echo $WINE_MINGW_W64
    ```

    <img width="1036" height="653" alt="02-initial-wine-config" src="https://github.com/user-attachments/assets/37df5c44-cbda-49a0-a7f0-1094d51d8c77" />

4. Initialize Wine in the directory pointed by `$WINE_MINGW_W64` using Windows 10 as target version:

    ```bash
    WINEPREFIX=$WINE_MINGW_W64 winecfg /v win10
    ```

    <img width="1036" height="653" alt="03-wine-initialization" src="https://github.com/user-attachments/assets/53dc8454-9e5e-465f-be65-d731ff1c9853" />

> [!TIP]
> 
> The errors shown in the command execution are not relevant.

5. List the contents of `$WINE_MINGW_W64` directory:

    ```bash
    ls -la $WINE_MINGW_W64
    ```

    <img width="1036" height="653" alt="04-wine-mingw-w64-contents" src="https://github.com/user-attachments/assets/561f023a-b109-4ac0-abd4-d9cba1c3d598" />

    In the image, it is worth to note a folder named `drive_c` - equivalent to the root `C:\` on Windows - and the file `user.reg` used to store registry entries (*Windows uses the registry to store configuration info about the system*).

### Create a symbolic link pointing to Lua development directory

1. For this step, we are going to place a symbolic link on `$WINE_MINGW_W64/drive_c/lua` pointing to Lua development directory. This has the same effect to have the development directory at `C:\lua` on Windows. I'll assume the development directory is stored at `$HOME/lua-5.4.8` (**CHANGE** this to the real path on your system):

    ```bash
    ln -s $HOME/lua-5.4.8 $WINE_MINGW_W64/drive_c/lua
    ```

2. List the contents of Lua development directory on Wine:

    ```bash
    WINEPREFIX=$WINE_MINGW_W64 wine cmd /C dir C:\\lua
    ```

    <img width="1036" height="653" alt="05-lua-dev-dir-contents" src="https://github.com/user-attachments/assets/7bccd674-40b5-40a4-9406-7ed7c4fa4c1f" />

> [!NOTE]
> 
> The last command is equivalent to run `dir C:\lua` on `cmd`. Since arguments are passed on our Unix shell, we have to use double backslash (`\\`) to forward a single backslash to `cmd`.

### Set permanent helper environment variables on Wine

We are going to set two helper environment variables for the current user on Wine:

* adjust `PATH` to have the directory of MinGW-w64 binaries (`gcc`, `mingw32-make`, etc);
* set `LUA_DIR` to hold Lua development directory.

To do that, we have to:

1. Open the file `$WINE_MINGW_W64/user.reg` on your text editor of choice. I'll use the `nano` editor for this task:

    ```bash
    nano $WINE_MINGW_W64/user.reg
    ```

2. Scroll down to find `[Environment]`:

    <img width="1036" height="653" alt="06-env-section" src="https://github.com/user-attachments/assets/882f860b-95e4-40f3-8763-ca1dba20f21b" />

3. In the `[Environment]` section, add the lines below:

    ```bash
    "PATH"="C:\\MinGW-w64\\mingw64\\bin"
    "LUA_DIR"="C:\\lua"
    ```

    <img width="1036" height="653" alt="07-changed-env-section" src="https://github.com/user-attachments/assets/3e732ef0-88ad-4f37-8ed1-d40325ae02ca" />

4. Save the changes;

5. Test the assigned environment variables:

    ```bash
    WINEPREFIX=$WINE_MINGW_W64 wine cmd /C echo %PATH%
    WINEPREFIX=$WINE_MINGW_W64 wine cmd /C echo %LUA_DIR%
    WINEPREFIX=$WINE_MINGW_W64 wine cmd /C dir %LUA_DIR%
    ```

    <img width="1036" height="653" alt="08-test-env-vars" src="https://github.com/user-attachments/assets/786133cf-9316-4094-a0eb-f1c533561ffa" />

> [!TIP]
> 
> In the output of `PATH` environment variable, it is different compared to what we set, because `PATH` is a special environment variable and Windows merges the values of system paths with user paths.

## Setup MinGW-w64 on Wine

1. We are going to install MinGW-w64 binaries provided by [https://github.com/niXman/mingw-builds-binaries](https://github.com/niXman/mingw-builds-binaries) on Wine through a shell script. Save the following script to a file named `setup-MinGW-w64.sh`:

    ```bash
    target_dir="$1"

    if ! [ -d "${target_dir}" ]; then
        mkdir -p "${target_dir}";

        if ! [ $? = 0 ]; then
            echo "Failed to create directory ${target_dir}";
            exit 1;
        fi
    fi

    base_url="https://github.com/niXman/mingw-builds-binaries/releases/download/15.2.0-rt_v13-rev0"
    
    for file in \
        x86_64-15.2.0-release-posix-seh-msvcrt-rt_v13-rev0.7z \
        ; do

        target_file="${target_dir}/${file}";

        if ! [ -e "${target_file}" ]; then
            wget "-P${target_dir}" "${base_url}/${file}";

            if ! [ $? = 0 ]; then
                echo "Failed to download ${file}";
                exit 1;
            fi
        fi

        /usr/bin/env -C "${target_dir}" \
        p7zip --decompress --force --keep "${file}";

        if ! [ $? = 0 ]; then
            echo "Failed to extract ${file}";
            exit 1;
        fi
    done
    ```

2. Run the script to download `MinGW-w64` and extract it at `$WINE_MINGW_W64/drive_c/MinGW-w64`, equivalent to `C:\MinGW-w64` on Windows:

    ```bash
    sh setup-MinGW-w64.sh $WINE_MINGW_W64/drive_c/MinGW-w64
    ```

    <img width="1036" height="653" alt="09-download-extract-mingw-w64" src="https://github.com/user-attachments/assets/bd6039df-db80-499d-b1a6-03dd60854885" />

3. Test `gcc` and `mingw32-make`:

    ```bash
    WINEPREFIX=$WINE_MINGW_W64 wine cmd /C x86_64-w64-mingw32-gcc --version
    WINEPREFIX=$WINE_MINGW_W64 wine cmd /C mingw32-make --version
    ```

    <img width="1036" height="653" alt="10-test-gcc-and-make" src="https://github.com/user-attachments/assets/8cbae4a1-7823-45e9-ad44-0c212b88c756" />

## Build and test Lua on Wine

> [!NOTE]
> 
> All the previous steps are part of setup operations. Thus, they only need to be done once.

1. Finally, build Lua on Wine with `gcc` through `mingw32-make`:

    ```bash
    WINEPREFIX=$WINE_MINGW_W64 wine cmd /C mingw32-make -C %LUA_DIR% mingw
    ```

    <img width="1036" height="653" alt="11-build-lua-on-wine" src="https://github.com/user-attachments/assets/b95feaf3-00f9-4c36-a6a0-43a67f78c808" />

2. Test the built interpreter:

    ```bash
    WINEPREFIX=$WINE_MINGW_W64 wine cmd /C %LUA_DIR%\\src\\lua.exe -v
    ```

    <img width="1036" height="653" alt="12-test-lua-interpreter" src="https://github.com/user-attachments/assets/adc24679-166e-43f8-a5bb-6dcab35c52c0" />

---

[Back to TOC](#table-of-contents) | [Back to home](../README.md)