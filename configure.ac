dnl                                               -*- Autoconf -*-
dnl
dnl Process this file with autoconf to produce the 'configure' script:
dnl
dnl autoreconf -i
dnl
dnl You must have recent versions of Autoconf and Automake installed.

############################################################

AC_PREREQ(2.61)
AC_INIT([Specfem 2D], [7.0.0], [see the wiki], [Specfem2D])
AC_CONFIG_SRCDIR([README.md])

AC_CONFIG_HEADER([setup/config.h])

AC_CONFIG_MACRO_DIR([m4])

############################################################

AS_BOX([setting up compilation flags])
AC_CANONICAL_HOST

# 'configure' options

###
### precision
###

AC_ARG_ENABLE([double-precision],
    [AC_HELP_STRING([--enable-double-precision],
        [solver in double precision @<:@default=no@:>@])],
    [want_double_precision="$enableval"],
    [want_double_precision=no])
if test "$want_double_precision" = no; then
    CUSTOM_REAL=SIZE_REAL
    CUSTOM_MPI_TYPE=MPI_REAL
else
    CUSTOM_REAL=SIZE_DOUBLE
    CUSTOM_MPI_TYPE=MPI_DOUBLE_PRECISION
fi
AC_SUBST([CUSTOM_REAL])
AC_SUBST([CUSTOM_MPI_TYPE])

###
### debugging
###

AC_ARG_ENABLE([debug],
    [AC_HELP_STRING([--enable-debug],
        [build with debugging options enabled @<:@default=no@:>@])],
    [want_debug="$enableval"],
    [want_debug=no])
AM_CONDITIONAL([COND_DEBUG], [test x"$want_debug" != xno])
export COND_DEBUG_FALSE
export COND_DEBUG_TRUE

###
### FORCE_VECTORIZATION
###

AC_ARG_ENABLE([vectorization],
    [AC_HELP_STRING([--enable-vectorization],
        [build FORCE_VECTORIZATION enabled version @<:@default=auto@:>@])],
    [want_vec="$enableval"],
    [want_vec=no])
if test x"$want_vec" == xauto; then
    if test x"$want_debug" != xno; then
        want_vec=no
    else
        want_vec=yes
    fi
else
    if test x"$want_vec" != xno && test x"$want_debug" != xno; then
        AC_MSG_ERROR([--enable-debug and --enable-vectorization cannot be specified simultaneously.])
    fi
fi
AM_CONDITIONAL([COND_VECTORIZATION], [test x"$want_vec" != xno])

###
### MPI
###

AC_ARG_WITH([mpi],
    [AC_HELP_STRING([--with-mpi],
        [build parallel version @<:@default=no@:>@])],
    [want_mpi="$withval"],
    [want_mpi=no])
AM_CONDITIONAL([COND_MPI], [test "$want_mpi" = yes])

###
### CUDA
###

AC_ARG_WITH([cuda],
    [AS_HELP_STRING([--with-cuda],
        [build CUDA GPU enabled version @<:@default=no@:>@])],
    [want_cuda="$withval"],
    [want_cuda=no])
AM_CONDITIONAL([COND_CUDA], [test "$want_cuda" != no])
AM_CONDITIONAL([COND_CUDA5], [test "$want_cuda" = cuda5])
AM_CONDITIONAL([COND_CUDA6], [test "$want_cuda" = cuda6])
AM_CONDITIONAL([COND_CUDA7], [test "$want_cuda" = cuda7])
AM_CONDITIONAL([COND_CUDA8], [test "$want_cuda" = cuda8])
# cuda linking for cuda 5x and 6x and 7x and 8x
AM_CONDITIONAL([COND_CUDA_PLUS], [test "$want_cuda" = cuda5 -o "$want_cuda" = cuda6 -o "$want_cuda" = cuda7 -o "$want_cuda" = cuda8])

############################################################

# Checks for programs.

# a courtesy to the installed base of users
if test x"$FC" = x && test x"$F90" != x; then
    FC="$F90"
fi
if test x"$MPIFC" = x && test x"$MPIF90" != x; then
    MPIFC="$MPIF90"
fi
# note: if not set, default FCFLAGS (and CFLAGS) will be set by autoconf/configure script (mostly -g -O2)
if test x"$FCFLAGS" = x && test x"$FLAGS_CHECK" != x ; then
    FCFLAGS=""
fi

AC_PROG_FC
export FC
export MPIFC
F77="$FC"
FFLAGS="$FCFLAGS"

AC_PROVIDE([AC_PROG_F77])
AC_SUBST([FCENV])

flags_guess="$SHELL $srcdir/flags.guess"
AC_MSG_NOTICE([running $flags_guess])
flags=`$flags_guess` ||
    AC_MSG_ERROR([$flags_guess failed])
eval $flags

AC_FC_WRAPPERS
AC_LANG(Fortran)
AC_FC_SRCEXT(f90)
AC_FC_FREEFORM()

AC_FC_PP_DEFINE()
AC_SUBST([FC_DEFINE])
AC_FC_PP_SRCEXT(F90)dnl Because AC_FC_PP_DEFINE messes with things.

AC_PROG_CC

AC_LANG_PUSH(C)
AC_CHECK_HEADER(emmintrin.h,AC_DEFINE([HAVE_EMMINTRIN],[1],[Define if emmintrin.h]))
AC_CHECK_HEADER(xmmintrin.h,AC_DEFINE([HAVE_XMMINTRIN],[1],[Define if xmmintrin.h]))
# AIX doesn't have err.h so we need to check
AC_CHECK_HEADER(err.h,      AC_DEFINE([HAVE_ERR],      [1],[Define if err.h]))
AC_LANG_POP(C)


############################################################

#checks for Scotch
AS_BOX([SCOTCH])

export SCOTCH_DIR
export SCOTCH_LIBDIR
export SCOTCH_INCLUDEDIR
export USE_BUNDLED_SCOTCH

AC_ARG_WITH([scotch-dir],
AS_HELP_STRING([--with-scotch-dir=DIR],[define the root path to Scotch (e.g. /opt/scotch/)]),
[
 ac_scotch_dir="$withval";
])

AC_ARG_WITH([scotch-includedir],
AS_HELP_STRING([--with-scotch-includedir=DIR],[define the path to the Scotch headers (e.g. /opt/scotch/include)]),
[
 ac_scotch_include_dir="$withval";
])

AC_ARG_WITH([scotch-libdir],
AS_HELP_STRING([--with-scotch-libdir=DIR],[define the path to the Scotch libraries (e.g. /opt/scotch/lib)]),
[
 ac_scotch_lib_dir="$withval";
])

if test "${USE_BUNDLED_SCOTCH}" != "1"; then

   if test -z "${ac_scotch_lib_dir}"; then
      if test -n "${ac_scotch_dir}"; then
         ac_scotch_lib_dir="${ac_scotch_dir}/lib";
      else
        ac_scotch_lib_dir="/usr/lib";
      fi
   fi

  if test -z "${ac_scotch_include_dir}"; then
    if test -n "${ac_scotch_dir}"; then
      ac_scotch_include_dir="${ac_scotch_dir}/include";
    else
      ac_scotch_include_dir="/usr/include/scotch";
    fi
  fi

  scotch_lib=""
  ac_save_ldflags=${LDFLAGS}
  if test -n "${ac_scotch_lib_dir}"; then
    LDFLAGS="${LDFLAGS} -L${ac_scotch_lib_dir}";
  fi
  AC_CHECK_LIB(scotch,scotchfarchinit ,[scotch_lib="yes";],[scotch_lib="no";LDFLAGS=${ac_save_ldflags}], -lscotcherr)

  scotch_include=""
  ac_save_cppflags=${CPPFLAGS}
  # scotch only needed for fortran code
  #if test -n "${ac_scotch_include_dir}"; then
  #  CPPFLAGS="${CPPFLAGS} -I${ac_scotch_include_dir}";
  #fi

dnl This does not work because we are in Fortran mode, which does not
dnl handle headers.
dnl
dnl AC_CHECK_HEADER(scotchf.h,[scotch_include="yes";],[scotch_include="no";CPPFLAGS=${ac_save_cppflags}])

  scotch_usable="${scotch_lib}"

fi

AC_MSG_CHECKING([whether Scotch is usable])
if test "x${scotch_usable}" = "xyes"; then
 AC_DEFINE([HAVE_SCOTCH],[1],[defined if Scotch is installed])
 want_scotch=yes
 USE_BUNDLED_SCOTCH=0
 SCOTCH_DIR="${ac_scotch_dir}"
 SCOTCH_LIBDIR="${ac_scotch_lib_dir}"
 SCOTCH_INCLUDEDIR="${ac_scotch_include_dir}"
 AC_MSG_RESULT([yes])
else
 AC_DEFINE([HAVE_SCOTCH],[1],[defined if Scotch is installed])
 AC_MSG_RESULT([no, using bundled scotch instead])
 AC_PROG_LEX
 if test -z "$LEX" || test "X$LEX" = "Xno"; then
   AC_MSG_ERROR([No suitable lex found])
 fi
 AC_PROG_YACC
 if test -z "$YACC" || test "X$YACC" = "Xno"; then
   AC_MSG_ERROR([No suitable yacc or bison found])
 fi
 ACX_PTHREAD(AC_MSG_RESULT([pthread found]), AC_MSG_ERROR([pthread not found]))

 # scotch only needed with mpi support
 if test "$want_mpi" = yes; then
  want_scotch=yes
  #daniel: scotch bundle
  # uses bundled scotch: current version pointed to by symbolic link scotch/
  USE_BUNDLED_SCOTCH=1

  SCOTCH_DIR="$srcdir/src/meshfem2D/scotch"
  SCOTCH_LIBDIR="${SCOTCH_DIR}/lib"
  SCOTCH_INCLUDEDIR="${SCOTCH_DIR}/include"
 else
  # no mpi support, scotch not needed
  want_scotch=no
  USE_BUNDLED_SCOTCH=0
 fi # want_mpi

fi
LDFLAGS=${ac_save_ldflags}

AM_CONDITIONAL([COND_SCOTCH], [test "$want_scotch" = yes])
AS_IF([test "$want_scotch" = yes],[
  AC_MSG_RESULT([SCOTCH is enabled])
],[
  AC_MSG_RESULT([SCOTCH is not enabled])
])


############################################################
# influential environment variables

AC_ARG_VAR(USE_BUNDLED_SCOTCH, [Set to 1 to always use the bundled Scotch library])
AC_ARG_VAR(SCOTCH_DIR, [Directory where Scotch is installed])
AC_ARG_VAR(SCOTCH_INCLUDEDIR, [Directory where Scotch headers are installed])
AC_ARG_VAR(SCOTCH_LIBDIR, [Directory where Scotch libraries are installed])

AC_ARG_VAR(MPIFC, [MPI Fortran compiler command])
AC_ARG_VAR(MPILIBS, [extra libraries for linking MPI programs])
AC_ARG_VAR(MPICC, [MPI C compiler command])

AC_ARG_VAR(FLAGS_CHECK, [Fortran compiler flags])

# scratch disks
AC_ARG_VAR(LOCAL_PATH_IS_ALSO_GLOBAL, [files on a local path on each node are also seen as global with same path @<:@default=true@:>@])

############################################################

# tests
AS_IF([test x"$MPIFC" = x],[
    MPIFC=mpif90
])
AS_IF([test x"$MPICC" = x],[
    MPICC=mpicc
])
AS_IF([test x"$LOCAL_PATH_IS_ALSO_GLOBAL" = x],[
    LOCAL_PATH_IS_ALSO_GLOBAL=true
])

# Checks for typedefs, structures, and compiler characteristics.

# check fortran modules flag

AS_BOX([module extensions])

AC_FC_MODULE_EXTENSION
AC_SUBST([FC_MODEXT])
AC_FC_MODULE_FLAG([], [])
AC_FC_MODULE_OUTPUT_FLAG([
	FCFLAGS_f90="$FC_MODOUT./obj $FC_MODINC./obj $FC_MODINC. $FCFLAGS_f90"
	FC_MODDIR=./obj
], [
	FC_MODDIR=.
])
AC_SUBST([FC_MODDIR])


# Checks for libraries.

###
### MPI
###

AS_IF([test "$want_mpi" = yes], [
    AS_BOX([MPI])
    CIT_MPI_INCDIR([$MPIFC])
])

###
### CUDA
###

AS_IF([test x"$want_cuda" != xno], [
    AS_BOX([CUDA])
    CIT_CUDA_CONFIG
])

###
### FORCE_VECTORIZATION
###

AS_IF([test x"$want_vec" != xno],[
  AS_BOX([FORCE VECTORIZATION])
  AC_MSG_NOTICE([FORCE_VECTORIZATION is enabled])

  FLAGS_CHECK="${FLAGS_CHECK} ${FC_DEFINE}FORCE_VECTORIZATION"
  CPPFLAGS="${CPPFLAGS} -DFORCE_VECTORIZATION"
])

############################################################

AS_BOX([setting up default simulation setup])

# Output results.
AC_CONFIG_FILES([
  Makefile
  setup/constants.h
  setup/constants_tomography.h
  setup/precision.h
  setup/config.fh
])

if test "$USE_BUNDLED_SCOTCH" = 1; then
  AC_CONFIG_FILES([
	  ${SCOTCH_DIR}/src/Makefile.inc
  ])
fi

AC_CONFIG_COMMANDS([bin], [AS_MKDIR_P(bin)])
AC_CONFIG_COMMANDS([obj], [AS_MKDIR_P(obj)])
AC_CONFIG_COMMANDS([DATA], [AS_MKDIR_P(DATA)])
AC_CONFIG_COMMANDS([OUTPUT_FILES], [AS_MKDIR_P(OUTPUT_FILES)])

AC_OUTPUT

dnl FYI, this is not a "result", but AS_ECHO is not available in 2.61, and it
dnl basically works like a portable echo.

AC_MSG_RESULT([])
AS_BOX([$PACKAGE_NAME $PACKAGE_VERSION])
AC_MSG_RESULT([])
AC_MSG_RESULT([$0 has completed and set up a default configuration to build.])
AC_MSG_RESULT([])
AC_MSG_RESULT([You may wish to modify the following files before running a simulation:])
AC_MSG_RESULT([AS_HELP_STRING([DATA/Par_file],
                              [Set parameters affecting the simulation.])])
AC_MSG_RESULT([AS_HELP_STRING([DATA/SOURCE],
                              [Set the source parameters before running the solver.])])
AC_MSG_RESULT([])

dnl end of configure.ac
