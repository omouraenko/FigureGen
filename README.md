# FigureGen

This is an updated version of [FigureGen](https://ccht.ccee.ncsu.edu/figuregen-v-49/). It was modified to work with GMT version 6.0 or higher.

## Rerequesites

FigureGen requires the following software:

1. **Generic Mapping Tools (GMT)** is the software package that plots the contours, contour lines, etc. and produces a PostScript file containing the figure. This version of FigureGen requires GMT 6.0 or higher. Install with command:

    ```text
    yum install gmt
    ```

1. **GhostScript** is the software package that converts PostScript files into different formats, including the raster format used by FigureGen. Install with command:

    ```text
    yum install ghostscript
    ```

1. **ImageMagick** is a software package for the manipulation and conversion of images. This software is optional. It is only required when FigureGen adds background images to the plots. Install with command:

    ```text
    yum install ImageMagick
    ```

1. **Zip** is a compression and file packaging uility. Install with command:

    ```text
    yum install zip
    ```

1. **Sed** is a string processing utility. Install with command:

    ```text
    yum install sed
    ```

### Installation of GMT on CentOS 8.5

Installation of GMT package on CentOS 8.5 may fail as package with the specific version of `libjson-c` library is not avaialble from the EPEL repository. The error may look like this (note, that `gdal` package is required by the `gmt` package):

```text
[root@node ~]# yum install gdal
Last metadata expiration check: 0:48:50 ago on Thu 11 Apr 2024 01:20:18 PM CDT.
Error:
Problem: package gdal-3.0.4-12.el8.x86_64 requires libgdal.so.26()(64bit), but none of the providers can be installed
  - package gdal-3.0.4-12.el8.x86_64 requires gdal-libs(x86-64) = 3.0.4-12.el8, but none of the providers can be installed
  - conflicting requests
  - nothing provides libjson-c.so.4(JSONC_0.14)(64bit) needed by gdal-libs-3.0.4-12.el8.x86_64
(try to add '--skip-broken' to skip uninstallable packages or '--nobest' to use not only best candidate packages)
```

The latest version of `libjson-c` library from the EPEL reposotiry is 0.13.1-2. However, it does not include the required `libjson-c.so.4(JSONC_0.14)` library:

```text
[root@node ~]# rpm -q --provides json-c
json-c = 0.13.1-2.el8
json-c(x86-64) = 0.13.1-2.el8
libjson-c.so.4()(64bit)
```

The later version 0.13.1-3 includes the required library and can be downloaded separately. For example, from [rpmfind.net](https://rpmfind.net) download [json-c-0.13.1-3.el8.x86_64.rpm](https://rpmfind.net/linux/centos/8-stream/BaseOS/x86_64/os/Packages/json-c-0.13.1-3.el8.x86_64.rpm) and [json-c-devel-0.13.1-3.el8.x86_64.rpm](https://rpmfind.net/linux/centos/8-stream/AppStream/x86_64/os/Packages/json-c-devel-0.13.1-3.el8.x86_64.rpm).

Then install the packages and verify:

```text
[root@node ~]# yum install json-c-0.13.1-3.el8.x86_64.rpm json-c-devel-0.13.1-3.el8.x86_64.rpm
[root@node ~]# rpm -q --provides json-c
json-c = 0.13.1-3.el8
json-c(x86-64) = 0.13.1-3.el8
libjson-c.so.4()(64bit)
libjson-c.so.4(JSONC_0.14)(64bit)
libjson-c.so.4(JSONC_0.15)(64bit)
```

After that, the GMT package should install without issues.

## Compilation

Below is an example of compilation steps using the Intel compiler. Intel MPI and NetCDF libraries should be available.

```bash
cmake -B build \
        -DUSE_MPI=ON \
        -DUSE_NETCDF=ON \
        -DCMAKE_Fortran_COMPILER=ifort \
        -DNETCDF_INCLUDES="$(nc-config --includedir)" \
        -DNETCDF_LIBRARIES="$(nf-config --flibs)" \
        -DNETCDF_LIBRARIES_F90="$(nc-config --libdir)"

cmake --build build
```

The parallel version executable name is `pfiguregen`. The serial version executable name is `figuregen`. The executables are created under `build` folder.

## Input file

Note, that in the input file, names of `gmt` and `gs` executables or full paths to executables (if they are not in the PATH) need to be specified:

```text
gmt                                                ! Path to GMT executable.
gs                                                 ! Path to GhostScript executable.
```

Or

```text
/usr/bin/gmt                                       ! Path to GMT executable.
/usr/bin/gs                                        ! Path to GhostScript executable.
```

## Example script

Below is an example script to run parallel version of FigureGen:

```bash
#!/bin/bash

# select executable
exe=pfiguregen

# input file name
inpFile=new.inp

# set environment (load mpi, netcdf, and set path to binaries)
module load mpi
module load netcdf
module load figuregen

# configure gmt.conf, check defaults with 'gmt defaults' command
# '-D' option for FORMAT_GEO_* not working, use '"-D"' instead of -D
gmt set FORMAT_GEO_MAP D
gmt set FORMAT_GEO_OUT D
gmt set FORMAT_FLOAT_OUT %lg
gmt set MAP_FRAME_TYPE fancy
gmt set GMT_HISTORY FALSE
gmt set PS_MEDIA letter

mpirun -n 2 $exe -I $inpFile > ${exe}.log
```
