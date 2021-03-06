AC_INIT([Open Dylan], [2013.2])
AC_PREREQ(2.50)

# Directory for config.guess etc.
AC_CONFIG_AUX_DIR(build-aux)

#---------------------
# Make srcdir absolute
#---------------------

srcdir=`(cd $srcdir; pwd)`

SUPPORT_GC_MPS=no
SUPPORT_GC_BOEHM=yes
GC_USE_STATIC_BOEHM=false
GC_LIB=gc
ADDITIONAL_RUNTIME_TARGET_PLATFORMS=

AC_CANONICAL_TARGET

MULTIARCH=$host
case $host in
  i*86-*-linux*)
      OPEN_DYLAN_TARGET_PLATFORM=x86-linux;
      SUPPORT_GC_MPS=yes;
      MULTIARCH=i386-linux-gnu
    ;;
  i*86-*-freebsd*)
      OPEN_DYLAN_TARGET_PLATFORM=x86-freebsd;
      GC_LIB=gc-threaded
      SUPPORT_GC_MPS=yes;
    ;;
  i*86-*-darwin*)
      OPEN_DYLAN_TARGET_PLATFORM=x86-darwin;
      ADDITIONAL_RUNTIME_TARGET_PLATFORMS="x86_64-darwin";
      GC_USE_STATIC_BOEHM=true
    ;;
  x86_64-*-darwin*)
      OPEN_DYLAN_TARGET_PLATFORM=x86-darwin;
      ADDITIONAL_RUNTIME_TARGET_PLATFORMS="x86_64-darwin";
      GC_USE_STATIC_BOEHM=true
    ;;
  amd64-*-freebsd* | x86_64-*-freebsd*)
      OPEN_DYLAN_TARGET_PLATFORM=x86_64-freebsd;
      GC_LIB=gc-threaded
    ;;
  x86_64-*-linux*)
      OPEN_DYLAN_TARGET_PLATFORM=x86_64-linux;
      MULTIARCH=x86_64-linux-gnu
    ;;
  arm-*-linux*)
      OPEN_DYLAN_TARGET_PLATFORM=arm-linux;
    ;;
  *)
      AC_MSG_ERROR([The $host platform is not supported.])
    ;;
esac
AC_SUBST(OPEN_DYLAN_TARGET_PLATFORM)
AC_SUBST(ADDITIONAL_RUNTIME_TARGET_PLATFORMS)

AM_INIT_AUTOMAKE

AC_PROG_INSTALL
AC_PROG_LN_S

AC_PATH_PROG(PERL, perl, no)
test "$PERL" = no && AC_MSG_ERROR([
perl is required to build Open Dylan])
AC_SUBST(PERL)

PERL_MODULES="File::Spec Getopt::Long"
for module in $PERL_MODULES; do
  AC_MSG_CHECKING(for Perl module $module)
  $PERL -M$module -e 'exit 0' || AC_MSG_ERROR([
$module is required to build Open Dylan])
  AC_MSG_RESULT(yes)
done

AC_CHECK_PROGS(DYLANCOMPILER, dylan-compiler opendylan minimal-console-compiler, no)
test "$DYLANCOMPILER" = no && AC_MSG_ERROR([
dylan-compiler, opendylan or minimal-console-compiler is required to bootstrap Open Dylan. Please download and install a build from http://opendylan.org/download/index.html])
# In 2014.1 and later we need -verbose so fdmake.pl gets warning count summaries.
AC_MSG_CHECKING(if $DYLANCOMPILER supports the -verbose flag)
if $DYLANCOMPILER -version -verbose >/dev/null 2>&1 ; then
  DYLANCOMPILER="$DYLANCOMPILER -verbose -build"
  AC_MSG_RESULT(yes)
else
  test "$DYLANCOMPILER" = dylan-compiler && DYLANCOMPILER="$DYLANCOMPILER -build"
  test "$DYLANCOMPILER" = opendylan && DYLANCOMPILER="$DYLANCOMPILER -build"
  test "$DYLANCOMPILER" = minimal-console-compiler && DYLANCOMPILER="$DYLANCOMPILER -build"
  AC_MSG_RESULT(no)
fi
AC_SUBST(DYLANCOMPILER)

# TODO: figure out the bootstrap type given the fdcompile version
#       as we may be able to just do a 1-stage-bootstrap. But for now,
#       default to a 3-stage-bootstrap to be safe.
bootstrap_target=3-stage-bootstrap
AC_SUBST(bootstrap_target)

AC_PROG_CC([clang gcc cc])

AC_CHECK_PROGS([MAKE],[gnumake gmake make],[AC_MSG_ERROR([Unable to find a make program.])])
if ! $MAKE --version | grep -q "GNU" 2> /dev/null; then
  AC_MSG_ERROR([OpenDylan requires GNU make to build.])
fi

AC_ARG_WITH(gc-path,
            AS_HELP_STRING([--with-gc-path=PATH],
                           [Path to GC distribution and build.]))

AC_ARG_WITH(gc,
            AS_HELP_STRING([--with-gc=mps|boehm],
                           [Which GC (MPS or Boehm) to use.]))

AS_IF([test x${with_gc} = x],
      [
        with_gc=boehm
      ])

AC_ARG_WITH(mps,
           AS_HELP_STRING([--with-mps=PATH],
                          [Deprecated option. Use --with-gc and --with-gc-path instead.]))
AS_IF([test x${with_mps} != x],
      [
        AC_MSG_WARN([You are using a deprecated configure flag. Please use --with-gc and --with-gc-path instead.])
        with_gc=mps
        with_gc_path=${with_mps}
      ])

AC_MSG_CHECKING([which GC to use])
AC_MSG_RESULT(${with_gc})
AC_MSG_CHECKING([which GC path to use])
AS_IF([test x${with_gc_path} = x],
      [
        AC_MSG_RESULT([none specified])
      ],
      [
        AC_MSG_RESULT(${with_gc_path})
      ])

if test "$cross_compiling" != "yes"; then
    AS_IF([test x${GC_USE_STATIC_BOEHM} = xtrue],
          [
            GC_LIB_NAME=libgc.a
          ],
          [
            AS_IF([test x${OPEN_DYLAN_TARGET_PLATFORM} = xx86-darwin -o x${OPEN_DYLAN_TARGET_PLATFORM} = xx86_64-darwin],
                  [
                    GC_LIB_NAME=libgc.dylib
                  ],
                  [
                    GC_LIB_NAME=libgc.so
                  ])
          ]
    )

    AS_IF([test x${with_gc} = xmalloc],
          [
            GC_CFLAGS="-DGC_USE_MALLOC"
            GC_LFLAGS=""
            GC_LIBRARY=""
            GC_USE_STATIC_BOEHM=false
            GC_CHOICE="malloc"
          ]
    )
    AS_IF([test x${with_gc} = xmps],
          [
            AS_IF([test x${SUPPORT_GC_MPS} = xno],
              [
                AC_MSG_ERROR([The platform that you are building for requires the Boehm GC.])
              ]
            )
            GC_CFLAGS="-DGC_USE_MPS -I${with_gc_path}/code"
            GC_LFLAGS=""
            GC_LIBRARY=""
            GC_USE_STATIC_BOEHM=false
            GC_CHOICE="mps"
            old_CPPFLAGS=$CPPFLAGS
            CPPFLAGS="-I${with_gc_path}/code"
            AC_CHECK_HEADER([mps.h],[],[AC_MSG_ERROR([Incorrect path to MPS specified. Please make sure you specified an absolute path to the root MPS directory.])])
            CPPFLAGS=$old_CPPFLAGS
            # This is to help catch people who haven't updated to the new MPS yet.
            AC_CHECK_FILE([${with_gc_path}/code/lii4gc.gmk],[AC_MSG_ERROR([You have an old version of MPS and need to update.])],[])
          ]
    )
    AS_IF([test x$with_gc = xboehm],
          [
            AS_IF([test x${SUPPORT_GC_BOEHM} = xno],
                  [
                    AC_MSG_ERROR([The platform that you are building for requires the MPS GC.])
                  ])
            GC_CFLAGS="-DGC_USE_BOEHM -DGC_THREADS"
            GC_LFLAGS="-l${GC_LIB}"
            AS_IF([test x${with_gc_path} != x],
                  [
                    GC_CFLAGS="${GC_CFLAGS} -I${with_gc_path}/include"
                    GC_LFLAGS="-L${with_gc_path}/lib ${GC_LFLAGS}"
                  ])
            old_LDFLAGS=$LDFLAGS
            LDFLAGS="${LDFLAGS} ${GC_LFLAGS}"
            AC_CHECK_LIB([${GC_LIB}], [GC_malloc])
            LDFLAGS=$old_LDFLAGS
            AS_IF([test x${GC_USE_STATIC_BOEHM} = xtrue],
                  [
                    AS_IF([test x${with_gc_path} != x],
                          [
                             AC_CHECK_FILE([${with_gc_path}/lib/${GC_LIB_NAME}],
                                           [
                                             GC_LIBRARY="${with_gc_path}/lib/${GC_LIB_NAME}"
                                           ],
                                           [])
                          ],
                          [
                            LIB_DIRECTORIES="/usr/lib /usr/lib64 /usr/local/lib /usr/local/lib64 /usr/lib/${MULTIARCH}"
                            for lib_directory in ${LIB_DIRECTORIES};
                            do
                              AS_IF([test x${GC_LIBRARY} = x],
                                    [
                                      AC_CHECK_FILE([${lib_directory}/${GC_LIB_NAME}],
                                                    [
                                                      GC_LIBRARY="${lib_directory}/${GC_LIB_NAME}"
                                                    ])
                                    ])
                            done
                          ])
                  ],
                  [
                    # We aren't using static boehm, so we don't need a GC_LIBRARY set.
                    GC_LIBRARY=""
                  ])
            GC_CHOICE="boehm"
          ])
fi

AS_IF([test x${GC_CHOICE}x${GC_USE_STATIC_BOEHM}x${GC_LIBRARY} = xboehmxtruex],
      [
        AC_MSG_ERROR([Could not find ${GC_LIB_NAME}. Please specify the path to the Boehm GC installation via --with-gc-path, or omit --with-gc-path to search the default directories.])
      ])
AS_IF([test x${GC_CHOICE} = x],
      [
        AC_MSG_ERROR([Invalid GC choice. Please select either 'boehm' or 'mps'.])
      ])
AC_SUBST(GC_CFLAGS)
AC_SUBST(GC_LFLAGS)
AC_SUBST(GC_LIBRARY)
AC_SUBST(GC_CHOICE)
AC_SUBST(GC_USE_STATIC_BOEHM)

# We don't want to build generated code with warnings enabled usually.
# The exception to this is if the person running configure has passed
# in CFLAGS with -W options ...
AC_MSG_CHECKING(if we should disable C compiler warnings for generated code)
AS_IF([echo "$CFLAGS" | grep -q -- "-W" 2> /dev/null],
      [
        DISABLE_WARNINGS_CFLAGS=
        AC_MSG_RESULT(no)
      ],[
        DISABLE_WARNINGS_CFLAGS=-w
        AC_MSG_RESULT(yes)
      ]
)
AC_SUBST(DISABLE_WARNINGS_CFLAGS)

AC_CONFIG_FILES(Makefile
                sources/jamfiles/Makefile
                sources/jamfiles/config.jam
                sources/lib/run-time/Makefile)

AC_OUTPUT
