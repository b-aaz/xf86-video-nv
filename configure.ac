#  Copyright 2005 Adam Jackson.
#
#  Permission is hereby granted, free of charge, to any person obtaining a
#  copy of this software and associated documentation files (the "Software"),
#  to deal in the Software without restriction, including without limitation
#  on the rights to use, copy, modify, merge, publish, distribute, sub
#  license, and/or sell copies of the Software, and to permit persons to whom
#  the Software is furnished to do so, subject to the following conditions:
#
#  The above copyright notice and this permission notice (including the next
#  paragraph) shall be included in all copies or substantial portions of the
#  Software.
#
#  THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
#  IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
#  FITNESS FOR A PARTICULAR PURPOSE AND NON-INFRINGEMENT.  IN NO EVENT SHALL
#  ADAM JACKSON BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER
#  IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN
#  CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
#
# Process this file with autoconf to produce a configure script

# Initialize Autoconf
AC_PREREQ([2.60])
AC_INIT([xf86-video-nv],
        [2.1.23],
        [https://gitlab.freedesktop.org/xorg/driver/xf86-video-nv/-/issues],
        [xf86-video-nv])
AC_CONFIG_SRCDIR([Makefile.am])
AC_CONFIG_HEADERS([config.h])

# Require X.Org macros 1.8 or later for MAN_SUBSTS set by XORG_MANPAGE_SECTIONS
m4_ifndef([XORG_MACROS_VERSION],
          [m4_fatal([must install xorg-macros 1.8 or later before running autoconf/autogen])])
XORG_MACROS_VERSION(1.8)
XORG_DEFAULT_OPTIONS

AC_CONFIG_AUX_DIR(.)

# Initialize Automake
AM_INIT_AUTOMAKE([foreign dist-xz])

# Initialize libtool
LT_INIT([disable-static])

AH_TOP([#include "xorg-server.h"])

# Options
# Define a configure option for an alternate module directory
AC_ARG_WITH(xorg-module-dir,
            AS_HELP_STRING([--with-xorg-module-dir=DIR],
                           [Default xorg module directory [[default=$libdir/xorg/modules]]]),
            [moduledir="$withval"],
            [moduledir="$libdir/xorg/modules"])

# Store the list of server defined optional extensions in REQUIRED_MODULES
XORG_DRIVER_CHECK_EXT(RANDR, randrproto)
XORG_DRIVER_CHECK_EXT(RENDER, renderproto)
XORG_DRIVER_CHECK_EXT(XV, videoproto)
XORG_DRIVER_CHECK_EXT(DPMSExtension, xextproto)

# Obtain compiler/linker options for the driver dependencies
PKG_CHECK_MODULES(XORG, [xorg-server >= 1.3 xproto fontsproto $REQUIRED_MODULES])
PKG_CHECK_MODULES(XEXT, [xextproto >= 7.0.99.1],
                  HAVE_XEXTPROTO_71="yes"; AC_DEFINE(HAVE_XEXTPROTO_71, 1, [xextproto 7.1 available]),
                  HAVE_XEXTPROTO_71="no")
AM_CONDITIONAL(HAVE_XEXTPROTO_71, [ test "$HAVE_XEXTPROTO_71" = "yes" ])

# Checks for libraries.


save_CFLAGS="$CFLAGS"
CFLAGS="$XORG_CFLAGS"

AC_CHECK_HEADERS(dev/wscons/wsconsio.h)

# PCI rework
AC_CHECK_DECL(XSERVER_LIBPCIACCESS,
              [XSERVER_LIBPCIACCESS=yes],[XSERVER_LIBPCIACCESS=no],
              [#include "xorg-server.h"])
if test "x$XSERVER_LIBPCIACCESS" = "xyes"; then
    PKG_CHECK_MODULES(PCIACCESS, [pciaccess >= 0.10.7])
fi

CFLAGS="$save_CFLAGS"

AC_ARG_ENABLE(xaa,
              AS_HELP_STRING([--enable-xaa],
                             [Enable legacy X Acceleration Architecture (XAA) [default=auto]]),
              [XAA="$enableval"],
              [XAA=auto])
if test "x$XAA" != xno; then
        save_CFLAGS=$CFLAGS
        save_CPPFLAGS=$CPPFLAGS
        CFLAGS=$XORG_CFLAGS
        CPPFLAGS="$XORG_CFLAGS"
        AC_CHECK_HEADERS([xaa.h], XAA=yes, XAA=no)
        CFLAGS=$save_CFLAGS
        CPPFLAGS=$save_CPPFLAGS
fi
AC_MSG_CHECKING([whether to include XAA support])
AM_CONDITIONAL(XAA, test "x$XAA" = xyes)
AC_MSG_RESULT([$XAA])

# Substitutions
AC_SUBST([moduledir])

# Checks for library functions
# We don't use strlcat, strlcpy, or timingsafe_memcmp, but check to quiet
# -Wredundant-decls warning from xorg/os.h which will otherwise redefine them.
AC_CHECK_FUNCS([strlcat strlcpy timingsafe_memcmp])

DRIVER_NAME=nv
AC_SUBST([DRIVER_NAME])

AC_CONFIG_FILES([
                Makefile
                src/Makefile
                man/Makefile
])
AC_OUTPUT
