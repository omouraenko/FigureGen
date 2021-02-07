# Using the FigureGen Containers

Contact: Georgia Stuart, georgia.stuart@austin.utexas.edu  
7 February 2021

## Input File Setup

For both Singularity and Docker container use, the path option for `ghostscript` and `gmt` 
**must be left blank**. See `autotest/docker-test/BathyFilledCPT.inp` for example.

## Singularity Containers

[Singularity](https://sylabs.io/docs/) is a container platform designed for use with HPC systems.
We provide three Singularity definition files:
- `figuregen.def` provides MPI-enabled FigureGen for use on local Linux/Unix machines.
- `figuregen-serial.def` provides serial (not MPI-enabled) FigureGen for use on local Linux/Unix machines.
- `figuregen-tacc.def` is compatible with TACC or TACC-like HPC systems.

In addition, we maintain a [FigureGen folder](https://cloud.sylabs.io/library/_collection/602041171e573cd09be5c019) at the [Singularity Container Library](https://cloud.sylabs.io/library).

### Using pre-built Singularity containers from the library

Running singularity containers assumes **your current working directory is where your ADCIRC datafiles are**. 
By default, singularity mounts your current working directory and `$HOME` so we have direct file access.

To download the image from the cloud library (e.g., to the root `FigureGen` folder of this repository, or to some other 
convenient location for singularity images), run the command:

```bash
singularity pull figuregen.sif library://georgiastuart/figuregen/figuregen
```

This command will pull down the latest `figuregen` image located at the cloud library and save it locally 
to `figuregen.sif`. Similarly, replace `figuregen` with `figuregen-serial` or `figuregen-tacc` to run the
serial or TACC versions of `figuregen`, respectively.

```bash
singularity pull figuregen-serial.sif library://georgiastuart/figuregen/figuregen-serial
singularity pull figuregen-tacc.sif library://georgiastuart/figuregen/figuregen-tacc
```

To run the tests in this repository, `cd` to `autotest/TestFiles` and run the following 
command (for example):

##### MPI Example
```bash
singularity exec <path to figuregen.sif> mpirun -np <num processes> figuregen -I ../Tests/BathyFilledCPT.inp
```

##### Serial Exmaple

```bash
singularity exec <path to figuregen-serial.sif> figuregen -I ../Tests/BathyFilledCPT.inp
```

##### TACC/HPC Example (e.g. on Stampede2. DO NOT RUN ON LOGIN NODE.)

```bash
prun singularity exec <path to figuregen-tacc.sif> figuregen -I ../Tests/BathyFilledCPT.inp
```
### Building Singularity Containers

To build a Singularity container yourself, from the root `FigureGen` directory of this repository,
use this command *on a machine where you have sudo privileges*:

```
sudo singularity build figuregen.sif container-files/singularity-def-files/figuregen.def
```

Replace `figuregen.def` with `figuregen-serial.def` or `figuregen-tacc.def` as appropriate.

## Docker Containers

For use on MacOS and Windows, we also provide Docker containers. **All docker container instructions
rely on your data files and input files being in the same directory** (e.g., the `autotest/docker-test` directory in this 
repository).