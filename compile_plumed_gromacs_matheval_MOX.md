# Compile Plumed with Matheval Library
```
0.0) Add these aliases in your .login file (Then log out, log back in)
          alias icpc="icpc -std=c++11"
          alias icc="icc -std=c++11"
0.1) module load icc_17-impi_2017 #or newest compiler if downloading a new version
```
### 2) Compile Matheval (without GUILE)
2a) unpack libmatheval (we use the 2013 version)
[libmatheval](http://hg.savannah.gnu.org/hgweb/libmatheval/)

2b) Create Directory m4
```bash
cd libmatheval
mkdir m4
```
2c) Edit Configure.in Files to remove Guile dependency
vi configure.in.

Comment out the following lines (#)

```bash
GUILE FLAGS
dnl Additional Guile feature checks.
AC_CHECK_TYPE([scm_t_bits], [AC_DEFINE([HAVE_SCM_T_BITS], [1], [Define to 1 if you have the `scm_t_bits` type.])], [], [#include <libguile.h>])
AC_CHECK_LIB([guile], [scm_c_define_gsubr], [AC_DEFINE([HAVE_SCM_C_DEFINE_GSUBR], [1], [Define to 1 if you have the `scm_c_define_gsubr` function.])], [], [$GUILE_LDFLAGS])
AC_CHECK_LIB([guile], [scm_make_gsubr], [AC_DEFINE([HAVE_SCM_MAKE_GSUBR], [1], [Define to 1 if you have the `scm_make_gsubr` function.])], [], [$GUILE_LDFLAGS])
AC_CHECK_LIB([guile], [scm_num2dbl], [AC_DEFINE([HAVE_SCM_NUM2DBL], [1], [Define to 1 if you have the `scm_num2dbl` function.])], [], [$GUILE_LDFLAGS])
 ```

2d) Edit tests/Makefile.am Files to remove Guile dependency
Comment out the following lines (#)

vi tests/Makefile.am

```bash
noinst_PROGRAMS = matheval
matheval_SOURCES = matheval.c
matheval_CFLAGS = $(GUILE_CFLAGS)
matheval_LDADD = $(top_builddir)/lib/libmatheval.la
matheval_LDFLAGS = $(GUILE_LDFLAGS)
```

2e) edit doc/libmatheval.texi
remove brackets from the following lines 

```
@title GNU manual
@subtitle{Manual edition @value{EDITION}}
@subtitle{For GNU @code{libmatheval} version @value{VERSION}}
@subtitle{Last updated @value{UPDATED}}
@author{Aleksandar Samardzic}
```

for example change:

```
@subtitle{Manual edition @value{EDITION}}
```

to 

```
@subtitle Manual edition @value{EDITION}
```


2f) Configure libmatheval

```bash
autoreconf -fi
./configure
```
2g) Disable yywrap in flex in MakeFile

Change: LEX=flex  in Makefile to:

LEX=flex --noyywrap

Apply this to the Makefile in libmatheval and the libmatheval/lib

2h) Make libmatheval

```bash

make

```

### 3) Compile PLUMED 2

3a) Unpack Plumed
```
tar -xzvf plumed-2.4.2.tgz
```

3b) Configure Plumed

```bash
cd plumed-2.4.2
./configure --prefix=***path_to_plumed_folder***  --enable-modules=all #there are other modules you can enable (i.e. adjmat,ves, etc)
```

Enabling adjmat allows for sprint coordinates

3c) Adjust paths to libmatheval in the Makefile.conf

Add the following flags to the lines (append to lines that already exist):
- DYNAMIC_LIBS= -lmatheval -L/***path_to_libmatheval***/libmatheval/lib/.libs
- CPPFLAGS= -D__PLUMED_HAS_MATHEVAL=1 -I/***path_to_libmatheval***/libmatheval/lib
- LDFLAGS=-rdynamic -L/***path_to_libmatheval***/libmatheval/lib/.libs

***Change for your path***

3d) add the following line to sourceme.sh (do not delete the other `export LD_LIBRARY_PATH=` line - add a new one)

export LD_LIBRARY_PATH="/***path_to_libmatheval***/libmatheval/lib/.libs/:$LD_LIBRARY_PATH"


***Change for your path***

3e) source sourceme.sh

3f)Make Plumed
```bash
make
make install
```
3g) Note commands for setting up PLUMED environment (should be given at end of make)

Example:

export PATH=$PATH:/gscratch/pfaendtner/codes/plumed2-BayesBias/bin/bin

export INCLUDE=$INCLUDE:/gscratch/pfaendtner/codes/plumed2-BayesBias/bin/include

export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/gscratch/pfaendtner/codes/plumed2-BayesBias/bin/lib/.libs
### 4) Install GROMACS

4a) Unpack GROMACS and change to it Directory
```bash
tar -xvzf gromacs-2018.3.tar.gz
cd gromacs-2018.3
```
4b) Patch Plumed and select version of GROMACS
```bash
plumed patch -p --runtime
```

4c) Setup the build for X-core node you are currently working with.

```bash
mkdir bin build_gromacs
cd bin
export GMXINST=`pwd`
cd ../build_gromacs/
module load cmake_3.8 #must use 3.4.3 or higher 
cmake .. -DGMX_MPI=ON -DCMAKE_INSTALL_PREFIX=$GMXINST # (-DGMX_FFT_LIBRARY=mkl default fftw)
```
For serial change last line to
```bash
cmake .. -DGMX_MPI=OFF -DCMAKE_INSTALL_PREFIX=$GMXINST  -DGMX_DEFAULT_SUFFIX=OFF -DGMX_BINARY_SUFFIX=_serial -DGMX_LIBS_SUFFIX=_serial -DGMX_FFT_LIBRARY=mkl
```
Choice of suffix is up to you.

4d) make -j 16  install

Add the following line to plumed environment script:

source /***path_to_GROMACS***/gromacs-5.1.2/bin/bin/GMXRC

note that to run GROMACS the commands are now gmxsuffix followed by the operation
where suffix is what you specified in the build step (see underscore was included in suffix)
if suffix not set and -DGMX_MPI=ON, then default suffix is _mpi (gmx_mpi is therefore the command to call GROMACS)

Ex:
gmx_mpi grompp -c start.gro -p topol.top -f nvt.mdp

to run a 4 replica multiple walkers job:

mpiexec.hydra -np 8 gmx_mpi mdrun -plumed plumed.dat -multi 4
