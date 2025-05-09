dnl Process this file with autoconf to produce a configure script.
AC_PREREQ(2.59)
AC_INIT(alsa-utils, 1.2.13)
AC_CONFIG_SRCDIR([aplay/aplay.c])
AC_PREFIX_DEFAULT(/usr)
AM_INIT_AUTOMAKE([subdir-objects])

AM_MAINTAINER_MODE([enable])

AM_GNU_GETTEXT([external])
AM_GNU_GETTEXT_VERSION([0.22.5])

dnl Checks for programs.

AC_PROG_CC
dnl AC_PROG_CXX
AC_PROG_INSTALL
AC_PROG_MKDIR_P
AC_PROG_LN_S
AC_PROG_SED
AC_DISABLE_STATIC
AM_PROG_LIBTOOL
PKG_PROG_PKG_CONFIG
AM_PATH_ALSA(1.2.13)
if test "x$enable_alsatest" = "xyes"; then
AC_CHECK_FUNC([snd_ctl_elem_add_enumerated],
	      , [AC_ERROR([No user enum control support in alsa-lib])])
fi


AC_CHECK_HEADERS([dlfcn.h malloc.h])

dnl Check components
AC_CHECK_HEADERS([alsa/pcm.h], [have_pcm="yes"], [have_pcm="no"],
  [#include <alsa/asoundlib.h>])
AC_CHECK_HEADERS([alsa/mixer.h], [have_mixer="yes"], [have_mixer="no"],
  [#include <alsa/asoundlib.h>])
AC_CHECK_HEADERS([alsa/rawmidi.h], [have_rawmidi="yes"], [have_rawmidi="no"],
  [#include <alsa/asoundlib.h>])
AC_CHECK_HEADERS([alsa/seq.h], [have_seq="yes"], [have_seq="no"],
  [#include <alsa/asoundlib.h>])
AC_CHECK_HEADERS([alsa/use-case.h], [have_ucm="yes"], [have_ucm="no"],
  [#include <alsa/asoundlib.h>])
AC_CHECK_HEADERS([alsa/topology.h], [have_topology="yes"], [have_topology="no"],
  [#include <alsa/asoundlib.h>])
AC_CHECK_HEADERS([samplerate.h], [have_samplerate="yes"], [have_samplerate="no"],
  [#include <samplerate.h>])

AC_CHECK_LIB([asound], [snd_seq_client_info_get_card], [HAVE_SEQ_CLIENT_INFO_GET_CARD="yes"])
AC_CHECK_LIB([atopology], [snd_tplg_save], [have_topology="yes"], [have_topology="no"])

#
# NOTE: The library 'libffado' (at least v2.4.1) executes ctor/dtor of instances
# for some objects in startup/finish routines of C runtime. As a result, it
# outputs some superfluos messages. Furthermore, it brings much memory leak
# internally. Totally, libffado support is not recommended at all in usual
# purposes except for technical preview.
#
AC_CHECK_LIB([ffado], [ffado_streaming_init], [have_ffado="yes"], [have_ffado="no"])
AS_IF([test x"$have_ffado" = xyes],
      [AC_DEFINE([WITH_FFADO], [1], [Define if FFADO library is available])])

# Test programs for axfer use shm by memfd_create(2). If not supported, open(2) is used alternatively.
AC_CHECK_FUNC([memfd_create], [have_memfd_create="yes"], [have_memfd_create="no"])
AS_IF([test x$have_memfd_create = xyes],
      [AC_DEFINE([HAVE_MEMFD_CREATE], [1], [Define if Linux kernel supports memfd_create system call])])

AM_CONDITIONAL(HAVE_PCM, test "$have_pcm" = "yes")
AM_CONDITIONAL(HAVE_MIXER, test "$have_mixer" = "yes")
AM_CONDITIONAL(HAVE_RAWMIDI, test "$have_rawmidi" = "yes")
AM_CONDITIONAL(HAVE_SEQ, test "$have_seq" = "yes")
AM_CONDITIONAL(HAVE_UCM, test "$have_ucm" = "yes")
AM_CONDITIONAL(HAVE_TOPOLOGY, test "$have_topology" = "yes" -a "$ac_cv_header_dlfcn_h" = "yes")
AM_CONDITIONAL(HAVE_SAMPLERATE, test "$have_samplerate" = "yes")
AM_CONDITIONAL(HAVE_FFADO, test "$have_ffado" = "yes")

dnl Use tinyalsa
alsabat_backend_tiny=
AC_ARG_ENABLE(alsabat_backend_tiny,
    AS_HELP_STRING([--enable-alsabat-backend-tiny], [Use tinyalsa for alsabat backend]),
    [case "${enableval}" in
      yes) alsabat_backend_tiny=true ;;
      no) alsabat_backend_tiny=false ;;
      *) AC_MSG_ERROR(bad value ${enableval} for --enable-alsabat-backend-tiny) ;;
    esac],[alsabat_backend_tiny=false])

dnl Disable bat
bat=
if test "$have_pcm" = "yes"; then
AC_ARG_ENABLE(bat,
     AS_HELP_STRING([--disable-bat], [Disable bat compilation]),
     [case "${enableval}" in
       yes) bat=true ;;
       no)  bat=false ;;
       *) AC_MSG_ERROR(bad value ${enableval} for --enable-bat) ;;
     esac],[bat=true])
fi
AM_CONDITIONAL(BAT, test x$bat = xtrue)

if test x$bat = xtrue; then

  saved_CFLAGS="$CFLAGS"
  saved_LDFLAGS="$LDFLAGS"
  saved_LIBS="$LIBS"
  FFTW_INC=""
  FFTW_LIB=""
  FFTW_CFLAGS=""
  dnl Check for libfftw3
  have_libfftw3="yes"
  AC_CHECK_LIB([fftw3f], [fftwf_malloc], , [have_libfftw3="no"])
  dnl Check for libtinyalsa
  have_libtinyalsa=
  if test x$alsabat_backend_tiny = xtrue; then
    have_libtinyalsa="yes"
    AC_CHECK_LIB([tinyalsa], [pcm_open], , [have_libtinyalsa="no"])
  fi
  AC_CHECK_LIB([m], [sqrtf], , [AC_MSG_ERROR([Error: Need sqrtf])])
  AC_CHECK_LIB([pthread], [pthread_create], , [AC_MSG_ERROR([Error: need PTHREAD library])])
  FFTW_CFLAGS="$CFLAGS"
  FFTW_LIB="$LIBS"
  CFLAGS="$saved_CFLAGS"
  LDFLAGS="$saved_LDFLAGS"
  LIBS="$saved_LIBS"
  AC_SUBST(FFTW_INC)
  AC_SUBST(FFTW_LIB)
  AC_SUBST(FFTW_CFLAGS)

fi
AM_CONDITIONAL(HAVE_LIBFFTW3, test "$have_libfftw3" = "yes")
AM_CONDITIONAL(HAVE_LIBTINYALSA, test "$have_libtinyalsa" = "yes")

dnl Check for librt
LIBRT=""
AC_MSG_CHECKING(for librt)
AC_ARG_WITH(librt,
  AS_HELP_STRING([--with-librt], [Use librt for monotonic clock (default = yes)]),
  [ have_librt="$withval" ], [ have_librt="yes" ])
if test "$have_librt" = "yes"; then
  AC_CHECK_LIB([rt], [clock_gettime], [HAVE_LIBRT="yes"])
  if test "$HAVE_LIBRT" = "yes" ; then
    LIBRT="-lrt"
    AC_DEFINE([HAVE_LIBRT], 1, [Have librt])
    AC_DEFINE([HAVE_CLOCK_GETTIME], 1, [Have clock gettime])
  fi
else
  AC_MSG_RESULT(no)
fi

dnl Disable alsamixer
CURSESINC=""
CURSESLIB=""
CURSES_CFLAGS=""
alsamixer=
if test "$have_mixer" = "yes"; then
AC_ARG_ENABLE(alsamixer,
     AS_HELP_STRING([--disable-alsamixer], [Disable alsamixer compilation]),
     [case "${enableval}" in
       yes) alsamixer=true ;;
       no)  alsamixer=false ;;
       *) AC_MSG_ERROR(bad value ${enableval} for --enable-alsamixer) ;;
     esac],[alsamixer=true])
fi
AM_CONDITIONAL(ALSAMIXER, test x$alsamixer = xtrue)

dnl Disable alsaconf
AC_ARG_ENABLE(alsaconf,
     AS_HELP_STRING([--disable-alsaconf], [Disable alsaconf packaging]),
     [case "${enableval}" in
       yes) alsaconf=true ;;
       no)  alsaconf=false ;;
       *) AC_MSG_ERROR(bad value ${enableval} for --enable-alsaconf) ;;
     esac],[alsaconf=true])
AM_CONDITIONAL(ALSACONF, test x$alsaconf = xtrue)

dnl Disable alsaloop
AC_ARG_ENABLE(alsaloop,
     AS_HELP_STRING([--disable-alsaloop], [Disable alsaloop packaging]),
     [case "${enableval}" in
       yes) alsaloop=true ;;
       no)  alsaloop=false ;;
       *) AC_MSG_ERROR(bad value ${enableval} for --enable-alsaloop) ;;
     esac],[alsaloop=true])
AM_CONDITIONAL(ALSALOOP, test x$alsaloop = xtrue)

dnl Disable nhlt
AC_ARG_ENABLE(nhlt,
     AS_HELP_STRING([--disable-nhlt], [Disable nhlt packaging]),
     [case "${enableval}" in
       yes) nhlt=true ;;
       no)  nhlt=false ;;
       *) AC_MSG_ERROR(bad value ${enableval} for --enable-nhlt) ;;
     esac],[nhlt=true])
AM_CONDITIONAL(NHLT, test x$nhlt = xtrue)

xmlto_available=""
AC_ARG_ENABLE(xmlto,
 AS_HELP_STRING([--disable-xmlto], [Disable man page creation via xmlto]),
 xmlto="$enableval", xmlto="yes")
if test "$xmlto" = "yes"; then
  AC_CHECK_PROG([xmlto_available], [xmlto], [yes])
fi
AM_CONDITIONAL(USE_XMLTO, test x"$xmlto_available" = xyes)

rst2man_available=""
AC_ARG_ENABLE(rst2man,
 AS_HELP_STRING([--disable-rst2man], [Disable man page creation via rst2man]),
 rst2man="$enableval", rst2man="yes")
if test "$rst2man" = "yes"; then
  AC_CHECK_PROG([rst2man_available], [rst2man], [yes])
fi
AM_CONDITIONAL(USE_RST2MAN, test x"$rst2man_available" = xyes)

AC_ARG_WITH(
        [udev-rules-dir],
        AS_HELP_STRING([--with-udev-rules-dir=DIR],[Directory where to install udev rules to (default=auto)]),
        [udevrulesdir="$withval"],
        [udevdir=$($PKG_CONFIG udev --variable=udevdir)
        if test "x$udevdir" = "x"; then
            udevrulesdir="/lib/udev/rules.d"
        else
            udevrulesdir="$udevdir/rules.d"
        fi])
AC_SUBST(udevrulesdir)

dnl Checks for header files.
AC_HEADER_STDC
if test x$alsamixer = xtrue; then
  AC_ARG_WITH(curses,
    AS_HELP_STRING([--with-curses=libname], [Specify the curses library to use (default=auto)]),
    curseslib="$withval",
    curseslib="auto")
  CURSESLIBDIR=""
  NCURSESLIBSUFFIX=""
  CURSES_NLS="no"
  if test "$curseslib" = "ncursesw" -o \( "$curseslib" = "auto" -a "$USE_NLS" = "yes" \); then
    dnl First try out pkg-config, then fall back to old config scripts.
    PKG_CHECK_MODULES([NCURSESW], [ncursesw], [
        CURSESINC="<ncurses.h>"
        CURSESLIB="${NCURSESW_LIBS}"
        CURSESLIBDIR=
        CURSES_CFLAGS="${NCURSESW_CFLAGS}"
        curseslib="ncursesw"
      ], [
        AC_CHECK_PROG([ncursesw5_config], [ncursesw5-config], [yes])
        if test "$ncursesw5_config" = "yes"; then
          CURSESINC="<ncurses.h>"
          CURSESLIB=`ncursesw5-config --libs`
          CURSESLIBDIR=`ncursesw5-config --libdir`
          CURSES_CFLAGS=`ncursesw5-config --cflags`
          curseslib="ncursesw"
        else
          AC_CHECK_LIB(ncursesw, initscr,
                     [ CURSESINC='<ncurses.h>'; CURSESLIB='-lncursesw'; curseslib="ncursesw"])
        fi
      ])
    if test -n "$CURSESINC"; then
      NCURSESLIBSUFFIX="w"
      CURSES_NLS="yes"
    fi
  fi
  if test "$curseslib" = "ncurses" -o "$curseslib" = "auto"; then
    dnl First try out pkg-config, then fall back to old config scripts.
    PKG_CHECK_MODULES([NCURSES], [ncurses], [
        CURSESINC="<ncurses.h>"
        CURSESLIB="${NCURSES_LIBS}"
        CURSESLIBDIR=
        CURSES_CFLAGS="${NCURSES_CFLAGS}"
        curseslib="ncurses"
      ], [
        AC_CHECK_PROG([ncurses5_config], [ncurses5-config], [yes])
        if test "$ncurses5_config" = "yes"; then
          CURSESINC="<ncurses.h>"
          CURSESLIB=`ncurses5-config --libs`
          CURSESLIBDIR=`ncurses5-config --libdir`
          CURSES_CFLAGS=`ncurses5-config --cflags`
          curseslib="ncurses"
        else
          AC_CHECK_LIB(ncurses, initscr,
                     [ CURSESINC='<ncurses.h>'; CURSESLIB='-lncurses'; curseslib="ncurses"])
        fi
      ])
  fi
  if test "$curseslib" = "curses" -o "$curseslib" = "auto"; then
    AC_CHECK_LIB(curses, initscr,
                 [ CURSESINC='<curses.h>'; CURSESLIB='-lcurses'; curseslib="curses"])
  fi
  if test -z "$CURSESINC"; then
     AC_MSG_ERROR(this packages requires a curses library)
  fi

  AC_MSG_CHECKING([for curses library])
  AC_MSG_RESULT([$curseslib])
  AC_MSG_CHECKING([for curses header name])
  AC_MSG_RESULT([$CURSESINC])
  AC_MSG_CHECKING([for curses compiler flags])
  AC_MSG_RESULT([$CURSES_CFLAGS])

  dnl CURSESLIBS might have the library path at the beginning.  If so, we cut it
  dnl off so that we can insert the other curses libraries before the ncurses
  dnl library but after the library path (which is later again prepended below).
  if test -n "$CURSESLIBDIR"; then
    if test "-L$CURSESLIBDIR " = "$(echo $CURSESLIB | cut -c-$((${#CURSESLIBDIR}+3)) )"; then
      CURSESLIB="$(echo $CURSESLIB | cut -c$((${#CURSESLIBDIR}+4))-)"
    fi
  fi

  saved_CFLAGS="$CFLAGS"
  saved_LDFLAGS="$LDFLAGS"
  saved_LIBS="$LIBS"
  CFLAGS="$CFLAGS $CURSES_CFLAGS"
  if test -n "$CURSESLIBDIR"; then
    LDFLAGS="$LDFLAGS -L$CURSESLIBDIR"
  fi
  LIBS="$CURSESLIB $LIBS"

  AC_TRY_LINK([#include <panel.h>], [set_escdelay(100);],[HAVE_CURSES_ESCDELAY="yes"])
  if test "$HAVE_CURSES_ESCDELAY" = "yes"; then
    AC_DEFINE([HAVE_CURSES_ESCDELAY], 1, [Have curses set_escdelay])
  fi

  if test "$USE_NLS" = "yes"; then
    AC_MSG_CHECKING([for curses NLS support])
    dnl In theory, a single-byte curses works just fine in ISO 8859-* locales.
    dnl In practice, however, everybody uses UTF-8 nowadays, so we'd better
    dnl check for wide-character support.
    dnl For ncurses/ncursesw, CURSES_NLS was already set above.
    if test "$curseslib" = "curses"; then
      AC_TRY_LINK([
          #define _XOPEN_SOURCE 1
          #define _XOPEN_SOURCE_EXTENDED 1
          #include <curses.h>
        ], [
          cchar_t wc;
          setcchar(&wc, L"x", A_NORMAL, 0, 0);
        ],
        [CURSES_NLS="yes"])
    fi
    AC_MSG_RESULT([$CURSES_NLS])
    if test "$CURSES_NLS" = "yes"; then
      AC_DEFINE([ENABLE_NLS_IN_CURSES], [1],
                [Define if curses-based programs can show translated messages.])
    fi
  fi

  AC_CHECK_HEADERS([panel.h menu.h form.h], [],
                   [AC_MSG_ERROR([required curses helper header not found])])
  AC_CHECK_LIB([panel$NCURSESLIBSUFFIX], [new_panel],
               [CURSESLIB="-lpanel$NCURSESLIBSUFFIX $CURSESLIB"],
               [AC_MSG_ERROR([panel$NCURSESLIBSUFFIX library not found])])
  AC_CHECK_LIB([menu$NCURSESLIBSUFFIX], [new_menu],
               [CURSESLIB="-lmenu$NCURSESLIBSUFFIX $CURSESLIB"],
               [AC_MSG_ERROR([menu$NCURSESLIBSUFFIX library not found])])
  AC_CHECK_LIB([form$NCURSESLIBSUFFIX], [new_form],
               [CURSESLIB="-lform$NCURSESLIBSUFFIX $CURSESLIB"],
               [AC_MSG_ERROR([form$NCURSESLIBSUFFIX library not found])])

  CFLAGS="$saved_CFLAGS"
  LDFLAGS="$saved_LDFLAGS"
  LIBS="$saved_LIBS"

  if test -n "$CURSESLIBDIR"; then
    CURSESLIB="-L$CURSESLIBDIR $CURSESLIB"
  fi

  AC_MSG_CHECKING([for curses linker flags])
  AC_MSG_RESULT([$CURSESLIB])
fi

AC_SUBST(CURSESINC)
AC_SUBST(CURSESLIB)
AC_SUBST(CURSES_CFLAGS)

test "x$prefix" = xNONE && prefix=$ac_default_prefix
test "x$exec_prefix" = xNONE && exec_prefix=$prefix

eval dir="$datadir"
case "$dir" in
/*) ;;
*) dir="$prefix/share"
esac

soundsdir="$dir/sounds/alsa"
AC_DEFINE_UNQUOTED(SOUNDSDIR, "$soundsdir", [directory containing sample data])

mydatadir="$dir/alsa"
AC_DEFINE_UNQUOTED(DATADIR, "$mydatadir", [directory containing alsa configuration])
AC_SUBST(mydatadir)

AC_ARG_WITH(testsound,
  AS_HELP_STRING([--with-testsound=file], [give the path of test sound file for alsaconf]),
  TESTSOUND="$withval",
  TESTSOUND="$dir/test.wav")
AC_SUBST(TESTSOUND)

AC_CONFIG_HEADERS(include/aconfig.h)

dnl Checks for typedefs, structures, and compiler characteristics.
AC_C_CONST
AC_C_INLINE
AC_HEADER_TIME

dnl Checks for library functions.
AC_PROG_GCC_TRADITIONAL

dnl Enable largefile support
AC_SYS_LARGEFILE

SAVE_UTIL_VERSION

AC_SUBST(LIBRT)

dnl Check for systemd
PKG_CHECK_MODULES(SYSTEMD, [systemd >= 18],
        [have_min_systemd="yes"],
        [have_min_systemd="no"])
AC_ARG_WITH([systemdsystemunitdir],
        AS_HELP_STRING([--with-systemdsystemunitdir=DIR], [Directory for systemd service files]),
        [], [with_systemdsystemunitdir=$($PKG_CONFIG --variable=systemdsystemunitdir systemd)])
if test "x$with_systemdsystemunitdir" != xno; then
        AC_SUBST([systemdsystemunitdir], [$with_systemdsystemunitdir])
fi
AM_CONDITIONAL(HAVE_SYSTEMD, [test "$have_min_systemd" = "yes" \
        -a -n "$with_systemdsystemunitdir" -a "x$with_systemdsystemunitdir" != xno ])

AC_ARG_WITH([asound-state-dir],
        AS_HELP_STRING([--with-asound-state-dir=DIR], [Directory to place asound.state file in]),
        [ASOUND_STATE_DIR="$withval"],
        [ASOUND_STATE_DIR="/var/lib/alsa"])
AC_SUBST(ASOUND_STATE_DIR)

AC_ARG_WITH([alsactl-lock-dir],
        AS_HELP_STRING([--with-alsactl-lock-dir=DIR], [Directory to place lock files in]),
        [ASOUND_LOCK_DIR="$withval"],
        [ASOUND_LOCK_DIR="/var/lock"])
AC_SUBST(ASOUND_LOCK_DIR)

AC_ARG_WITH([alsactl-pidfile-dir],
        AS_HELP_STRING([--with-alsactl-pidfile-dir=DIR], [Directory to place alsactl.pid file in]),
        [ALSACTL_PIDFILE_DIR="$withval"],
        [ALSACTL_PIDFILE_DIR="/var/run"])
AC_SUBST(ALSACTL_PIDFILE_DIR)

AC_ARG_WITH([alsactl-daemonswitch],
        AS_HELP_STRING([--with-alsactl-daemonswitch=FILE], [File to test for the daemon mode]),
        [ALSACTL_DAEMONSWITCH="$withval"],
        [ALSACTL_DAEMONSWITCH="/etc/alsa/state-daemon.conf"])
AC_SUBST(ALSACTL_DAEMONSWITCH)

AC_ARG_WITH([alsactl-udev-extra-test],
        AS_HELP_STRING([--with-alsactl-udev-extra-test=TEST], [Extra udev tests]),
        [ALSACTL_UDEV_EXTRATEST="$withval"],
        [ALSACTL_UDEV_EXTRATEST="default"])
if test "$ALSACTL_UDEV_EXTRATEST" = "default"; then
  ALSACTL_UDEV_EXTRATEST="TEST==\"${sbindir}\", TEST==\"${mydatadir}\","
fi
AC_SUBST(ALSACTL_UDEV_EXTRATEST)

AC_ARG_WITH([alsactl-udev-args],
        AS_HELP_STRING([--with-alsactl-udev-args=ARGS], [Extra alsactl arguments (udev rules)]),
        [ALSACTL_UDEV_ARGS="$withval"],
        [ALSACTL_UDEV_ARGS=""])
AC_SUBST(ALSACTL_UDEV_ARGS)

dnl pre-process plugin directory
AC_ARG_WITH(plugindir,
    AS_HELP_STRING([--with-plugindir=dir],
	[path where pre-process plugin files are stored]),
    plugindir="$withval", plugindir="")
if test -z "$plugindir"; then
    eval dir="$libdir"
    echo ${exec_prefix}
    echo $libdir
    echo $dir
    case "$dir" in
    /*) ;;
    *) dir="$dir"
    esac
    plugindir="$dir/alsa-topology"
fi
AC_DEFINE_UNQUOTED(ALSA_TOPOLOGY_PLUGIN_DIR, "$plugindir", [directory containing ALSA topology pre-process plugins])
ALSA_TOPOLOGY_PLUGIN_DIR="$plugindir"
AC_SUBST(ALSA_TOPOLOGY_PLUGIN_DIR)

AC_OUTPUT(Makefile alsactl/Makefile alsactl/init/Makefile \
	  alsamixer/Makefile amidi/Makefile amixer/Makefile \
	  m4/Makefile po/Makefile.in \
	  alsaconf/alsaconf alsaconf/Makefile \
	  alsaconf/po/Makefile \
	  alsaucm/Makefile topology/Makefile topology/nhlt/Makefile \
	  bat/Makefile bat/tests/Makefile bat/tests/asound_state/Makefile \
	  aplay/Makefile include/Makefile iecset/Makefile utils/Makefile \
	  utils/alsa-utils.spec seq/Makefile seq/aconnect/Makefile \
	  seq/aplaymidi/Makefile seq/aplaymidi2/Makefile seq/aseqdump/Makefile seq/aseqnet/Makefile \
	  seq/aseqsend/Makefile speaker-test/Makefile speaker-test/samples/Makefile \
	  alsaloop/Makefile alsa-info/Makefile \
	  axfer/Makefile axfer/test/Makefile \
	  nhlt/Makefile)
