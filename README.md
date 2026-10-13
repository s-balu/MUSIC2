[![CMake on multiple platforms](https://github.com/cosmo-sims/MUSIC/actions/workflows/cmake-multi-platform.yml/badge.svg)](https://github.com/cosmo-sims/MUSIC/actions/workflows/cmake-multi-platform.yml)

MUSIC v2 - multi-scale cosmological initial conditions
======================================================

MUSIC is a computer program to generate nested grid initial conditions for high-resolution "zoom" cosmological simulations. A detailed description of some of the algorithms can be found in [Hahn & Abel (2011)](http://arxiv.org/abs/1103.6031), but version 2.0 is much improved and has a much lower memory footprint. 

*Be aware that v1 and v2 do not produce entirely consistent zooms, due to the changes in the way the noise hierarchy is composed. So do not upgrade if you need to exactly reproduce a zoom run generated with v1, there is currently no backwards compatibility mode.*

To learn how to use MUSIC, please [read the Wiki](https://github.com/cosmo-sims/music/wiki). 

Current MUSIC key features include:

- Supports output for RAMSES, ENZO, Arepo, Swift, Gadget-2/3, ART, Pkdgrav/Gasoline, NyX, and GAMER-2 via plugins. New codes can be added.
- Support for first (1LPT) and second order (2LPT) Lagrangian perturbation theory.
- Full integration of the [PANPHASIA](https://arxiv.org/abs/1306.5968) hierarchical Gaussian random fields
- Pluggable transfer functions, currently CAMB, Eisenstein&Hu, BBKS, Warm Dark Matter variants. Distinct baryon+CDM fields.
- Minimum bounding ellipsoid and convex hull shaped high-res regions supported with most codes, supports refinement mask generation for RAMSES.
- Parallelized with OpenMP
- Requires FFTW3, GSL (and HDF5 for output for some codes)

## Building MUSIC
While we still supply the old Makefile, using CMake is now the preferred way of building. CMake use out-of-source build, i.e. you create a build directory, and then configure the code using CMake. Inside the `music` directory, do
```
  mkdir build
  cd build
  ccmake ..
  make -j
```
to configure the code (you will se a menu), and then start a parallel compilation. If CMake has trouble finding your FFTW or HDF5 installation, you can add hints as follows
```
  FFTW3_ROOT=<path> HDF5_ROOT=<path> ccmake ..
```
If you want to build on macOS, then it is strongly recommended to use GNU (or Intel) compilers instead of Apple's Clang. Install them e.g. via homebrew and then configure cmake to use them instead of the macOS default compiler via
```
  CC=gcc-13 CXX=g++-13 ccmake ..
```
This is necessary since Apple's compilers haven't supported OpenMP for years.

### Nyx output with AMReX

Enable the Nyx output plugin with `-DENABLE_NYX=ON`. It requires a separate
3D, double-precision, CPU-only AMReX library built without MPI. Nyx itself
can still use MPI and GPUs. A GNU Make build of Nyx/AMReX does not supply
the CMake package required by MUSIC.

From the MUSIC source directory, build and install AMReX using the same
C++ compiler as MUSIC (replace `/path/to/amrex` with your AMReX source path):

```sh
cmake -S /path/to/amrex -B build/amrex \
  -DCMAKE_BUILD_TYPE=Release \
  -DCMAKE_INSTALL_PREFIX="$PWD/build/amrex-install" \
  -DAMReX_SPACEDIM=3 -DAMReX_PRECISION=DOUBLE \
  -DAMReX_MPI=OFF -DAMReX_GPU_BACKEND=NONE \
  -DAMReX_FORTRAN=OFF -DAMReX_PARTICLES=OFF \
  -DAMReX_AMRLEVEL=OFF -DAMReX_LINEAR_SOLVERS=OFF
cmake --build build/amrex -j4
cmake --install build/amrex

cmake -S . -B build/music-nyx \
  -DENABLE_NYX=ON -DCMAKE_BUILD_TYPE=Release \
  -DCMAKE_PREFIX_PATH="$PWD/build/amrex-install"
cmake --build build/music-nyx -j4
```

If AMReX is already installed, pass its installation prefix in
`CMAKE_PREFIX_PATH`, or set `AMReX_DIR` to the directory containing
`AMReXConfig.cmake`. FFTW, GSL and OpenMP are still required by MUSIC.
The default MUSIC build also enables CLASS and PANPHASIA; disable them with
`-DENABLE_CLASS=OFF -DENABLE_PANPHASIA=OFF` if they are not needed.

In your MUSIC parameter file, set `format = nyx` and `filename = init` in
`[output]`, and add `blocking_factor = 8` in `[setup]`. The plugin writes
`init/Level_*/Cell`, a header, a grids file, and sample Nyx `inputs`/`probin`
files in the working directory. Use Nyx's MUSIC reader with
`nyx.particle_init_type = Cosmological`, `cosmo.ic-source = MUSIC` and
`cosmo.initDirName = init`. The sample inputs need the runtime settings
appropriate to your Nyx build.


## Running
There is an example parameter file 'example.conf' in the main directory. Possible options are explained in it, it can be run as a simple argument, e.g. from within the build directory:
```
  ./MUSIC ../ics_example.conf
```


## Disclaimer
This program is distributed in the hope that it will be useful, but WITHOUT ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. By downloading and using MUSIC, you agree to the LICENSE, distributed with the source code in a text file of the same name.


