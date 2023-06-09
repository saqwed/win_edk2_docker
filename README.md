# Windows docker images for EDK2 build environment

## Install docker desktop

- Goto [https://www.docker.com/products/docker-desktop/](https://www.docker.com/products/docker-desktop/) and download docker-desktop
  - After install completed, make sure change to use windows container

## Build the docker image

- Clone this repository
- Enter the dicrecotry in repository
- Run `docker build . -t win_edk2_docker`
- During adjust Dockerfile, you may use `docker system prune --force` to clean up unused images.

## Push docker image to ghcr.io

```batch
rem Create classic personal token first, and then use your github user name and token to login ghcr.io
docker login ghcr.io
docker tag win_edk2_docker ghcr.io/saqwed/win_edk2_docker/win_edk2_docker:latest
docker push ghcr.io/saqwed/win_edk2_docker/win_edk2_docker:latest
```

## Pull image from ghcr.io

```bash
docker pull ghcr.io/saqwed/win_edk2_docker/win_edk2_docker:latest
```

## How to run the docker image

- `docker run -it win_edk2_docker c:\windows\system32\cmd.exe`
- If you already clone the tianocore/edk2 repository, use below command to map the directory into container
  - `docker run -it -v c:\edk2:c:\edk2 win_edk2_docker c:\windows\system32\cmd.exe`

## How to setup build environment in the docker container

- Build BaseTools

```batch
rem "C:\ProgramData\Microsoft\Windows\Start Menu\Programs\Visual Studio 2019\Visual Studio Tools\VC\x86 Native Tools Command Prompt for VS 2019.lnk"
call "c:\BuildTools\VC\Auxiliary\Build\vcvars32.bat"
cd c:\edk2
edksetup VS2019
cd c:\edk2\BaseTools
nmake
```

- Build Packages

```batch
rem "C:\ProgramData\Microsoft\Windows\Start Menu\Programs\Visual Studio 2019\Visual Studio Tools\VC\x86_x64 Cross Tools Command Prompt for VS 2019.lnk"
call "c:\BuildTools\VC\Auxiliary\Build\vcvarsx86_amd64.bat"
cd c:\edk2
edksetup.bat VS2019
rem build -a X64 -t VS2019 -p ShellPkg/ShellPkg.dsc -b RELEASE
build -a IA32 -a X64 -t VS2019 -p OvmfPkg/OvmfPkgIa32X64.dsc -b RELEASE
```

## Use vswhere to get Visual Studio install directory

Cannot run directly, have to put into a batch file and run it.


```batch
rem x86

for /f "tokens=*" %%i in ('"%ProgramFiles(x86)%\Microsoft Visual Studio\Installer\vswhere.exe" -latest -products * -requires Microsoft.VisualStudio.Product.BuildTools -property installationPath') do call %%i\VC\Auxiliary\Build\vcvars32.bat
rem for /f "tokens=*" %%i in ('"%ProgramFiles(x86)%\Microsoft Visual Studio\Installer\vswhere.exe" -latest -products * -requires Microsoft.VisualStudio.Component.VC.Tools.x86.x64 -property installationPath') do call %%i\VC\Auxiliary\Build\vcvars32.bat

rem x86/x64

for /f "tokens=*" %%i in ('"%ProgramFiles(x86)%\Microsoft Visual Studio\Installer\vswhere.exe" -latest -products * -requires Microsoft.VisualStudio.Product.BuildTools -property installationPath') do call %%i\VC\Auxiliary\Build\vcvarsx86_amd64.bat
rem for /f "tokens=*" %%i in ('"%ProgramFiles(x86)%\Microsoft Visual Studio\Installer\vswhere.exe" -latest -products * -requires Microsoft.VisualStudio.Component.VC.Tools.x86.x64 -property installationPath') do call %%i\VC\Auxiliary\Build\vcvarsx86_amd64.bat
```

Reference from: [Find-VC](https://github.com/microsoft/vswhere/wiki/Find-VC)
