AC_PREREQ([2.63])
AC_INIT([nm-applet],
        [0.9.8.4],
        [https://bugzilla.gnome.org/enter_bug.cgi?product=NetworkManager],
        [network-manager-applet])

AC_CONFIG_HEADERS([config.h])
AC_CONFIG_MACRO_DIR([m4])

AM_INIT_AUTOMAKE([1.10 subdir-objects no-dist-gzip dist-bzip2 -Wno-portability])
AM_MAINTAINER_MODE([enable])

# Support silent build rules, requires at least automake-1.11. Disable
# by either passing --disable-silent-rules to configure or passing V=1
# to make
m4_ifdef([AM_SILENT_RULES], [AM_SILENT_RULES([no])])

dnl
dnl Require programs
dnl
AC_PROG_CC
AM_PROG_CC_C_O
AC_PROG_INSTALL
PKG_PROG_PKG_CONFIG

dnl Define _GNU_SOURCE for various things like strcasestr()
AC_GNU_SOURCE

dnl Initialize libtool
LT_PREREQ([2.2.6])
LT_INIT

dnl
dnl Required headers
dnl
AC_CHECK_HEADERS(fcntl.h paths.h sys/ioctl.h sys/time.h syslog.h unistd.h)

dnl
dnl Checks for typedefs, structures, and compiler characteristics.
dnl
AC_TYPE_MODE_T
AC_TYPE_PID_T
AC_HEADER_TIME

dnl
dnl Checks for library functions.
dnl
AC_PROG_GCC_TRADITIONAL
AC_FUNC_MEMCMP
AC_CHECK_FUNCS(select socket uname)

dnl
dnl translation support
dnl
IT_PROG_INTLTOOL([0.40.0])

GETTEXT_PACKAGE=nm-applet
AC_SUBST(GETTEXT_PACKAGE)
AC_DEFINE_UNQUOTED(GETTEXT_PACKAGE,"$GETTEXT_PACKAGE", [Gettext package])
AM_GLIB_GNU_GETTEXT

# Check for iso-codes for country names translation
AC_MSG_CHECKING([whether to disable iso-codes at build-time])
AC_ARG_ENABLE([iso-codes],
    AS_HELP_STRING([--disable-iso-codes],[do not check for iso-codes at build-time]),
    [],[disable_iso_codes_check=no])
if test x$disable_iso_codes_check = xno ; then
    AC_MSG_RESULT([no])
else
    AC_MSG_RESULT([yes])
fi

if test x$disable_iso_codes_check = "xno" ; then
    AC_MSG_CHECKING([whether iso-codes has iso_3166 domain])
    if $PKG_CONFIG --variable=domains iso-codes | grep iso_3166 >/dev/null ; then
        AC_MSG_RESULT([yes])
    else
        AC_MSG_RESULT([no])
    fi
    AC_DEFINE_UNQUOTED([ISO_CODES_PREFIX],["`$PKG_CONFIG --variable=prefix iso-codes`"],[ISO codes prefix])
    PKG_CHECK_MODULES(ISO_CODES, [iso-codes],
        [],
        [echo -e "\n$ISO_CODES_PKG_ERRORS.\n"
         echo "Consider installing the package or adjusting the PKG_CONFIG_PATH environment variable."
         echo "You can also disable build-time check for 'iso-codes' via --disable-iso-codes";
         exit 1;])
else
    AC_DEFINE_UNQUOTED([ISO_CODES_PREFIX],["$prefix"],[ISO codes prefix])
fi

dnl
dnl Make sha1.c happy on big endian systems
dnl
AC_C_BIGENDIAN

PKG_CHECK_MODULES(GOBJECT, gobject-2.0)

PKG_CHECK_MODULES(NMA,
		[dbus-glib-1 >= 0.74
		 gio-2.0 >= 2.26
		 NetworkManager >= 0.9.8
		 libnm-glib >= 0.9.8
		 libnm-util >= 0.9.8
		 libnm-glib-vpn >= 0.9.8
		 gmodule-export-2.0])

# With recent glib, defining GLIB_VERSION_MIN_REQUIRED avoids
# deprecation warnings for recently-deprecated functions (eg,
# GValueArray stuff). We say GLIB_VERSION_2_26 because there
# aren't macros for any older versions.
NMA_CFLAGS="$NMA_CFLAGS -DGLIB_VERSION_MIN_REQUIRED=GLIB_VERSION_2_26"
# Likewise for Gtk; we don't want to port to GtkGrid, etc, until
# we can drop gtk2 support
NMA_CFLAGS="$NMA_CFLAGS -DGDK_VERSION_MIN_REQUIRED=GDK_VERSION_3_0"

AC_MSG_CHECKING([whether to build nm-applet-migration-tool])
AC_ARG_ENABLE([migration],
	      [AS_HELP_STRING([--disable-migration], [Don't build migration tool for NM <= 0.8 settings])],
	      [enable_migration="$enableval"],
	      [if test "$UNDER_JHBUILD" = "true"; then
	           enable_migration=no
	       else
	           enable_migration=yes
	       fi])
AC_MSG_RESULT([$enable_migration])
if test "$enable_migration" = "yes"; then
	PKG_CHECK_MODULES(MIGRATION, [gconf-2.0 gnome-keyring-1], :,
			  [AC_MSG_ERROR([
Could not find devel files needed for nm-applet-migration-tool:

$MIGRATION_PKG_ERRORS

You can configure with --disable-migration if you want to build
without the code to migrate GConf settings from older (< 0.9)
nm-applet releases.])])

	AC_SUBST(MIGRATION_CFLAGS)
	AC_SUBST(MIGRATION_LIBS)

	AC_DEFINE(BUILD_MIGRATION_TOOL, 1, [Define when building nm-applet-migration-tool])
fi
AM_CONDITIONAL(BUILD_MIGRATION_TOOL, test "$enable_migration" = "yes")

PKG_CHECK_MODULES(LIBSECRET, [libsecret-unstable])
AC_SUBST(LIBSECRET_CFLAGS)
AC_SUBST(LIBSECRET_LIBS)

# Check for libnotify >= 0.7
PKG_CHECK_MODULES(LIBNOTIFY_07, [libnotify >= 0.7], [have_libnotify_07=yes],[have_libnotify_07=no])
if test x"$have_libnotify_07" = "xyes"; then
	AC_DEFINE(HAVE_LIBNOTIFY_07, 1, [Define if you have libnotify 0.7 or later])
fi
PKG_CHECK_MODULES(NOTIFY, [libnotify >= 0.4.3])
AC_SUBST(NOTIFY_CFLAGS)
AC_SUBST(NOTIFY_LIBS)

gtk2_req=2.20
gtk3_req=3.0
AC_ARG_WITH([gtkver], AS_HELP_STRING([--with-gtkver], [The major version of GTK+ to build with]),
            with_gtkver="$withval",with_gtkver=0)
case "${with_gtkver}" in
       0) PKG_CHECK_MODULES(GTK, gtk+-3.0 > $gtk3_req, ,
                            [PKG_CHECK_MODULES(GTK, gtk+-2.0 > $gtk2_req)])
	  ;;
       2) PKG_CHECK_MODULES(GTK, gtk+-2.0 >= $gtk2_req)
          ;;
       3) PKG_CHECK_MODULES(GTK, gtk+-3.0 >= $gtk3_req)
          ;;
       *) AC_MSG_ERROR(unknown GTK+ version $with_gtkver!)
          ;;
esac
AC_SUBST(GTK_CFLAGS)
AC_SUBST(GTK_LIBS)

# Check for dbus-1.2.6 or later for deny-by-default rules
PKG_CHECK_MODULES(DBUS_126, [dbus-1 >= 1.2.6], [have_dbus_126=yes],[have_dbus_126=no])
AM_CONDITIONAL(HAVE_DBUS_126, test x"$have_dbus_126" = "xyes")

AC_ARG_WITH(dbus-sys, AS_HELP_STRING([--with-dbus-sys=DIR], [where D-BUS system.d directory is]))

if ! test -z "$with_dbus_sys" ; then
    DBUS_SYS_DIR="$with_dbus_sys"
else
    DBUS_SYS_DIR="${sysconfdir}/dbus-1/system.d"
fi
AC_SUBST(DBUS_SYS_DIR)

dnl Check for gnome-bluetooth
AC_ARG_WITH([bluetooth],
	    AS_HELP_STRING([--with-bluetooth|--without-bluetooth], [Enable Bluetooth support]),
	    with_bluetooth="$withval",with_bluetooth=yes)
have_gbt=no
case "${with_bluetooth}" in
	no)	AC_MSG_NOTICE(Bluetooth support disabled)
		;;
	*)
		AC_MSG_CHECKING(for gnome-bluetooth)
		PKG_CHECK_MODULES(GNOME_BLUETOOTH,
				  gnome-bluetooth-1.0 >= 2.27.6
				  libnm-util >= 0.9.4
				  libnm-glib >= 0.9.4,
				  have_gbt=yes, have_gbt=no)
		;;
esac

AM_CONDITIONAL(HAVE_GBT, test x"$have_gbt" = "xyes")

dnl ModemManager1 with libmm-glib
AC_ARG_WITH(modem-manager-1, AS_HELP_STRING([--with-modem-manager-1], [Enable new ModemManager1 interface support]),,[with_modem_manager_1=auto])
if (test "${with_modem_manager_1}" != "no"); then
    PKG_CHECK_MODULES(MM_GLIB,
                      [mm-glib],
                      [have_libmm_glib=yes],
                      [have_libmm_glib=no])
    AC_SUBST(MM_GLIB_CFLAGS)
    AC_SUBST(MM_GLIB_LIBS)

    if (test "${have_libmm_glib}" = "no"); then
        if (test "${with_modem_manager_1}" = "yes"); then
            AC_MSG_ERROR([Couldn't find libmm-glib])
        fi
    else
        with_modem_manager_1="yes"
    fi
fi

if (test "${with_modem_manager_1}" = "yes"); then
    AC_DEFINE(WITH_MODEM_MANAGER_1, 1, [Define if you have ModemManager1 support])
else
	AC_DEFINE(WITH_MODEM_MANAGER_1, 0, [Define if you have ModemManager1 support])
fi
AM_CONDITIONAL(WITH_MODEM_MANAGER_1, test "${with_modem_manager_1}" = "yes")

dnl Check for gudev
PKG_CHECK_MODULES(GUDEV, gudev-1.0 >= 147)
AC_SUBST(GUDEV_CFLAGS)
AC_SUBST(GUDEV_LIBS)

dnl Check for gobject introspection
GOBJECT_INTROSPECTION_CHECK([0.9.6])

GLIB_CONFIG_NMA

dnl
dnl Compiler flags
dnl
NM_COMPILER_WARNINGS
# Use --enable-maintainer-mode to disabled deprecated symbols
GNOME_MAINTAINER_MODE_DEFINES


AC_CONFIG_FILES([
Makefile
src/Makefile
src/libnm-gtk/Makefile
src/libnm-gtk/tests/Makefile
src/libnm-gtk/examples/Makefile
src/libnm-gtk/libnm-gtk.pc
src/marshallers/Makefile
src/utils/Makefile
src/utils/tests/Makefile
src/gconf-helpers/Makefile
src/gconf-helpers/tests/Makefile
src/wireless-security/Makefile
src/connection-editor/Makefile
src/gnome-bluetooth/Makefile
icons/Makefile
icons/16/Makefile
icons/22/Makefile
icons/32/Makefile
icons/48/Makefile
icons/scalable/Makefile
po/Makefile.in
man/Makefile
man/nm-applet.1
man/nm-connection-editor.1
])
AC_OUTPUT
