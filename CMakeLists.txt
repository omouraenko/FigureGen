cmake_minimum_required(VERSION 3.10)
project(fig_gen LANGUAGES Fortran)
enable_language(Fortran)

option(USE_MPI "Build Parallel Version of Figuregen" OFF)
option(USE_NETCDF "Build with NetCDF" OFF)
option(GMT_PRE_5 "Version of GMT is older than 5.0" OFF)

set(CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake/")

# define default flags for compilers
if (CMAKE_Fortran_COMPILER_ID STREQUAL "Intel")
    # using Intel
    if (NOT DEFINED CMAKE_Fortran_FLAGS)
        set(CMAKE_Fortran_FLAGS "-O3")
    endif ()
else()
    # using GNU
    if (NOT DEFINED CMAKE_Fortran_FLAGS)
        set(CMAKE_Fortran_FLAGS "-Wall -Wextra -O3")
    endif ()
endif()

add_executable(figuregen src/FigureGen.F90)

if (${USE_MPI})
    find_package(MPI REQUIRED Fortran)
    set(CMAKE_Fortran_COMPILER "${MPI_Fortran_COMPILER}")
    target_include_directories(figuregen PUBLIC SYSTEM ${MPI_Fortran_INCLUDE_PATH})
    target_link_libraries(figuregen PUBLIC ${MPI_Fortran_LIBRARIES})
    target_compile_definitions(figuregen PUBLIC CMPI)
    set_target_properties(figuregen PROPERTIES OUTPUT_NAME pfiguregen)
endif()

if (${USE_NETCDF})
    set(NETCDF_F90 "YES")
    find_package(NetCDF REQUIRED)
    find_package(HDF5 REQUIRED COMPONENTS Fortran)
    target_include_directories(figuregen PUBLIC SYSTEM ${NETCDF_INCLUDES} ${HDF5_INCLUDE_DIRS})
    target_link_libraries(figuregen PUBLIC ${NETCDF_LIBRARIES} ${HDF5_LIBRARIES})
    target_compile_definitions(figuregen PUBLIC NETCDF)
endif()

target_compile_definitions(figuregen PUBLIC VARFILLVAL)

if (${GMT_PRE_5})
    target_compile_definitions(figuregen PUBLIC GMTPRE5)
endif()
