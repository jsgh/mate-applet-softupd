AC_PREREQ([2.66])
AC_INIT([mate-applet-softupd], [0.4.7], [assen.totin@gmail.com])
AC_CONFIG_HEADERS([config.h])

AM_INIT_AUTOMAKE([-Wall])

# prefix
if test "x$prefix" = "xNONE" ; then
	prefix="/usr/local"
fi
if test "x$exec_prefix" = "xNONE" ; then
	exec_prefix=$prefix
fi
PREFIX=$prefix
AC_SUBST([PREFIX])
BINDIR=$exec_prefix/bin
AC_SUBST([BINDIR])
SBINDIR=$exec_prefix/sbin
AC_SUBST([SBINDIR])
DATAROOTDIR=$prefix/share
AC_SUBST([DATAROOTDIR])
LIBEXECDIR=$prefix/libexec
if test "$libexecdir" != '${exec_prefix}/libexec'; then
        LIBEXECDIR=$libexecdir
fi
AC_SUBST([LIBEXECDIR])
PIXMAPDIR=$prefix/share/pixmaps
AC_SUBST([PIXMAPDIR])
ICONSDIR=$prefix/share/icons
AC_SUBST([ICONSDIR])
LOCALEDIR=$prefix/share/locale
AC_SUBST([LOCALEDIR])
AC_DEFINE_UNQUOTED([LOCALEDIR],["$LOCALEDIR"],[Locale files])
TOPLEVELDIR=`pwd`
AC_SUBST([TOPLEVELDIR])

# gettext
AM_GNU_GETTEXT_VERSION(0.18.1)
AM_GNU_GETTEXT([external])
if test $USE_NLS != "yes" ; then
        echo "*** gettext not found. You're losing internationalization."
fi
AC_DEFINE_UNQUOTED([MATELOCALEDIR], ["$LOCALEDIR"], ["Locale Dir MATE"])

# Checks for programs
AC_PROG_CC
MAKEFILE_CC=`which gcc`
AC_SUBST([MAKEFILE_CC])

AC_CHECK_PROG([FOUND_MSGFMT],[msgfmt],["yes"],["no"])
if test $FOUND_MSGFMT = "no"; then
	echo "*** msgfmt not found. Cannot continue."
	exit
else
	MSGFMT=`which msgfmt`
	AC_SUBST([MSGFMT])
fi

# Early check for libmatepanelapplet: results can be used to auto-select the
# GTK version.
PKG_CHECK_MODULES([LIBMATEPANELAPPLET_4], [libmatepanelapplet-4.0], [FOUND_LMPA_4="yes"], [FOUND_LMPA_4="no"])
PKG_CHECK_MODULES([LIBMATEPANELAPPLET_3], [libmatepanelapplet-3.0], [FOUND_LMPA_3="yes"], [FOUND_LMPA_3="no"])
if test $FOUND_LMPA_4 = "yes" ; then
	LIBMATEPANELAPPLET="libmatepanelapplet-4.0"
elif test $FOUND_LMPA_3 = "yes" ; then
	LIBMATEPANELAPPLET="libmatepanelapplet-3.0"
else
	AC_MSG_ERROR(["*** libmatepanelapplet not found."])
fi

# Check for GTK and determine version
FORCE_GTK="no"
RPM_REQUIRE_GTK=" "
USE_GTK="0"
AC_ARG_ENABLE([gtk],[AS_HELP_STRING([--enable-gtk],[compile only for the specified GTK version, 2 or 3 (default: check both, use GTK-3, if not found, use GTK-2))])],[FORCE_GTK=$enableval])

PKG_CHECK_MODULES([PKG_GTK2], [gtk+-2.0], [FOUND_GTK2="1"], [FOUND_GTK2="0"])
PKG_CHECK_MODULES([PKG_GTK3], [gtk+-3.0], [FOUND_GTK3="1"], [FOUND_GTK3="0"])

if test $FORCE_GTK = "3" ; then
	if test $FOUND_GTK3 = "1" ; then
		AM_PATH_GTK_3_0([3.0.0], [USE_GTK="3"], [AC_MSG_ERROR(["*** GTK3 forced, but not found."])])
	else
		AC_MSG_ERROR(["*** GTK3 forced, but not found."])
	fi
elif test $FORCE_GTK = "2" ; then
	if test $FOUND_GTK2 = "1" ; then
		AM_PATH_GTK_2_0([2.24.0], [USE_GTK="2"], [AC_MSG_ERROR(["*** GTK2 forced, but not found."])])
	else
		AC_MSG_ERROR(["*** GTK2 forced, but not found."])
	fi
else
	# Try to match the GTK version used by mate-panel.
	PANEL_GTK=`$PKG_CONFIG --print-requires "$LIBMATEPANELAPPLET" 2>/dev/null |
		grep -m 1 '^gtk+-'`
	case "$PANEL_GTK" in
	gtk+-3*)
		AM_PATH_GTK_3_0([3.14.0], [USE_GTK="3"], [AC_MSG_ERROR(["*** GTK3 used by mate-panel, but no suitable version found."])]);;
	gtk+-2*)
		AM_PATH_GTK_2_0([2.24.0], [USE_GTK="2"], [AC_MSG_ERROR(["*** GTK2 used by mate-panel, but no suitable version found."])]);;
	esac

	# Only accept GTK3 if at least 3.14; else resort to GTK2
	if test $FOUND_GTK3 = "1" -a $USE_GTK = "0"; then
		AM_PATH_GTK_3_0([3.14.0], [USE_GTK="3"], [USE_GTK="0"])
	fi
	if test $FOUND_GTK2 = "1" -a $USE_GTK = "0" ; then
		AM_PATH_GTK_2_0([2.24.0], [USE_GTK="2"], [USE_GTK="0"])
	fi
fi

if test $USE_GTK = "3" ; then
	AC_DEFINE([HAVE_GTK3], [1], ["Define GTK3"])
	AC_DEFINE([GETTEXT_PACKAGE], ["gtk30"], ["GTK gettext"])
	RPM_REQUIRE_GTK="Requires: gtk3 >= 3.14.0"
	RPM_BUILDREQUIRE_GTK="Requires: gtk3 >= 3.14.0"
	SELECTED_GTK="GTK-3"
elif test $USE_GTK = "2" ; then
	AC_DEFINE([HAVE_GTK2], [1], ["Define GTK2"])
	AC_DEFINE([GETTEXT_PACKAGE], ["gtk20"], ["GTK gettext"])
	RPM_REQUIRE_GTK="Requires: gtk2 >= 2.24.0"
	RPM_BUILDREQUIRE_GTK="Requires: gtk2-devel >= 2.24.0"
	SELECTED_GTK="GTK-2"
else
	AC_MSG_ERROR(["*** GTK not found."])
fi

AC_SUBST([RPM_REQUIRE_GTK])
AC_SUBST([RPM_BUILDREQUIRE_GTK])

# Check for backend tools
FOUND_BACKEND="no"
FORCE_BACKEND="no"
ADDITIONAL_PACKAGES=""
AC_ARG_ENABLE([backend],[AS_HELP_STRING([--enable-backend],[use the specified backend: package-kit, yum, yum-updatesd, apt-check (default: pick up the first available from the list)])],[FORCE_BACKEND=$enableval])

## Check for backend: packagekit
AC_CHECK_PROG([CHECK_PACKAGEKIT],[pkcon],[yes],[no])
if test $CHECK_PACKAGEKIT = "no" ; then
	echo "*** package-kit not found."
else
	FOUND_BACKEND="yes"
fi

## Check for backend: yum-updatesd
AC_CHECK_PROG([CHECK_YUMUPDATESD],[yum-updatesd],[yes],[no])
if test $CHECK_YUMUPDATESD = "no" ; then
	echo "*** yum-updatesd not found."
else
	FOUND_BACKEND="yes"
fi

## Check for backend: dnf
AC_CHECK_PROG([CHECK_DNF],[dnf],[yes],[no])
if test $CHECK_DNF = "no" ; then
	echo "*** dnf not found."
else
	FOUND_BACKEND="yes"
	DNF_VIEWER=`which dnf`
fi

## Check for backend: yum
AC_CHECK_PROG([CHECK_YUM],[yum],[yes],[no])
if test $CHECK_YUM = "no" ; then
	echo "*** yum not found."
else
	FOUND_BACKEND="yes"
	YUM_VIEWER=`which yum`
fi

## Check for backend: apt-check
AC_PATH_PROG([APTCHECK_PROG],[apt-check],[no],[/usr/lib/update-notifier:/var/lib/update-notifier:/usr/lib64/update-notifier:/var/lib64/update-notifier])
if test $APTCHECK_PROG = "no" ; then
	echo "*** apt-check not found."
else
	FOUND_BACKEND="yes"
fi

## Select which backend to use
SELECTED_BACKEND="none"
EXIT_NOW="no"
EXIT_MSG="none"
RPM_REQUIRES_BACKEND=""
RPM_BUILDREQUIRES_BACKEND=""
if test $FOUND_BACKEND = "no" ; then
	EXIT_NOW="yes"
	EXIT_MSG="*** No suitable backend found. Cannot continue."
elif test $FORCE_BACKEND = "package-kit" ; then
	if test $CHECK_PACKAGEKIT = "no" ; then
		EXIT_NOW="yes"
		EXIT_MSG="Forced backend $FORCE_BACKEND not found. Cannot continue."
	else
		SELECTED_BACKEND=$FORCE_BACKEND
		RPM_REQUIRES_BACKEND="Requires: PackageKit"
		RPM_BUILDREQUIRES_BACKEND="BuildRequires: PackageKit-glib-devel, PackageKit"
		AC_DEFINE([HAVE_PACKAGEKIT],[1],[Define if package-kit was found])
		ADDITIONAL_PACKAGES="packagekit-glib2"
	fi
elif test $FORCE_BACKEND = "yum-updatesd" ; then
	if test $CHECK_YUMUPDATESD = "no" ; then
		EXIT_NOW="yes"
		EXIT_MSG="Forced backend $FORCE_BACKEND not found. Cannot continue."
	else
		SELECTED_BACKEND=$FORCE_BACKEND
		RPM_REQUIRES_BACKEND="Requires: yum-updatesd"
		AC_DEFINE([HAVE_YUMUPDATESD],[1],[Define if yum-updatesd was found])
		ADDITIONAL_PACKAGES="dbus-1 dbus-glib-1"
	fi
elif test $FORCE_BACKEND = "dnf" ; then
	if test $CHECK_DNF = "no" ; then
		EXIT_NOW="yes"
		EXIT_MSG="Forced backend $FORCE_BACKEND not found. Cannot continue."
	else
		SELECTED_BACKEND=$FORCE_BACKEND
		AC_DEFINE([HAVE_DNF],[1],[Define if dnf was found])
		AC_DEFINE_UNQUOTED([DNF_BINARY],["$DNF_VIEWER"],[Define where is the yum binary])
		RPM_REQUIRES_BACKEND="Requires: dnf"
	fi
elif test $FORCE_BACKEND = "yum" ; then
	if test $CHECK_YUM = "no" ; then
		EXIT_NOW="yes"
		EXIT_MSG="Forced backend $FORCE_BACKEND not found. Cannot continue."
	else
		SELECTED_BACKEND=$FORCE_BACKEND
		AC_DEFINE([HAVE_YUM],[1],[Define if yum was found])
		AC_DEFINE_UNQUOTED([YUM_BINARY],["$YUM_VIEWER"],[Define where is the yum binary])
		RPM_REQUIRES_BACKEND="Requires: yum"
	fi
elif test $FORCE_BACKEND = "apt-check" ; then
	if test $APTCHECK_PROG = "no" ; then
		EXIT_NOW="yes"
		EXIT_MSG="Forced backend $FORCE_BACKEND not found. Cannot continue."
	else
		SELECTED_BACKEND=$FORCE_BACKEND
	fi
else
	echo "Chosing backend wisely..."
	if test $CHECK_PACKAGEKIT != "no" ; then
		SELECTED_BACKEND="package-kit"
		AC_DEFINE([HAVE_PACKAGEKIT],[1],[Define if package-kit was found])
		ADDITIONAL_PACKAGES="packagekit-glib2"
		RPM_REQUIRES_BACKEND="Requires: PackageKit"
		RPM_BUILDREQUIRES_BACKEND="BuildRequires: PackageKit-glib-devel, PackageKit"
	elif test $CHECK_YUMUPDATESD != "no" ; then
		SELECTED_BACKEND="yum-updatesd"
		AC_DEFINE([HAVE_YUMUPDATESD],[1],[Define if yum-updatesd was found])
		ADDITIONAL_PACKAGES="dbus-1 dbus-glib-1"
		RPM_REQUIRES_BACKEND="Requires: yum-updatesd"
	elif test $CHECK_DNF != "no" ; then
		SELECTED_BACKEND="dnf"
		AC_DEFINE([HAVE_DNF],[1],[Define if dnf was found])
		AC_DEFINE_UNQUOTED([DNF_BINARY],["$DNF_VIEWER"],[Define where is the yum binary])
		RPM_REQUIRES_BACKEND="Requires: dnf"
	elif test $CHECK_YUM != "no" ; then
		SELECTED_BACKEND="yum"
		AC_DEFINE([HAVE_YUM],[1],[Define if yum was found])
		AC_DEFINE_UNQUOTED([YUM_BINARY],["$YUM_VIEWER"],[Define where is the yum binary])
		RPM_REQUIRES_BACKEND="Requires: yum"
	elif test $APTCHECK_PROG != "no" ; then
		SELECTED_BACKEND="apt-check"
		AC_DEFINE([HAVE_APTCHECK],[1],[Define if apt-check was found])
		AC_DEFINE_UNQUOTED([APTCHECK_BINARY],["$APTCHECK_PROG"],[Define where is the apt-check binary])
	fi
fi

AM_CONDITIONAL([WITH_PACKAGEKIT], [test $SELECTED_BACKEND = "package-kit"])
AM_CONDITIONAL([WITH_YUMUPDATESD], [test $SELECTED_BACKEND = "yum-updatesd"])
AM_CONDITIONAL([WITH_DNF], [test $SELECTED_BACKEND = "dnf"])
AM_CONDITIONAL([WITH_YUM], [test $SELECTED_BACKEND = "yum"])
AM_CONDITIONAL([WITH_APTCHECK], [test $SELECTED_BACKEND = "apt-check"])
AC_SUBST([RPM_REQUIRES_BACKEND])
AC_SUBST([RPM_BUILDREQUIRES_BACKEND])

# Check which software installer to use
FOUND_INSTALLER="no"
FORCE_INSTALLER="no"
AC_ARG_ENABLE([installer],[AS_HELP_STRING([--enable-installer],[use the specified backend: yumex-dnf, yumex, gpk-update-viewer, update-manager (default: pick up the first available from the list)])],[FORCE_INSTALLER=$enableval])

# Check for yumex
AC_CHECK_PROG([CHECK_YUMEXVIEWER],[yumex],[yes],[no])
if test $CHECK_YUMEXVIEWER = "no" ; then
        echo "*** yumex not found."
else
        FOUND_INSTALLER="yes"
        YUMEXVIEWER_INSTALLER=`which yumex`
fi

# Check for yumex-dnf
AC_CHECK_PROG([CHECK_YUMEXDNFVIEWER],[yumex-dnf],[yes],[no])
if test $CHECK_YUMEXDNFVIEWER = "no" ; then
        echo "*** yumex-dnf not found."
else
        FOUND_INSTALLER="yes"
        YUMEXDNFVIEWER_INSTALLER=`which yumex-dnf`
fi

# Check for gpk-update-viewer
AC_CHECK_PROG([CHECK_GPKUPDATEVIEWER],[gpk-update-viewer],[yes],[no])
if test $CHECK_GPKUPDATEVIEWER = "no" ; then
        echo "*** gpk-update-viewer not found."
else
        FOUND_INSTALLER="yes"
        GPKUPDATEVIEWER_INSTALLER=`which gpk-update-viewer`
fi

# Check for update-manager
AC_CHECK_PROG([CHECK_UPDATEMANAGER],[update-manager],[yes],[no])
if test $CHECK_UPDATEMANAGER = "no" ; then
        echo "*** update-manager not found."
else
        FOUND_INSTALLER="yes"
        UPDATEMANAGER_INSTALLER=`which update-manager`
fi

# Check for dnfdragora
AC_CHECK_PROG([CHECK_DNFDRAGORA],[dnfdragora],[yes],[no])
if test $CHECK_DNFDRAGORA = "no" ; then
        echo "*** dnfdragora not found."
else
        FOUND_INSTALLER="yes"
        DNFDRAGORA_INSTALLER=`which dnfdragora`
fi

# Select which software installer to use
SELECTED_INSTALLER="none"
if test $FORCE_INSTALLER = "yumex" ; then
	if test $CHECK_YUMEXVIEWER = "yes" ; then
		SELECTED_INSTALLER="yumex"
	else 
                EXIT_NOW="yes"
                EXIT_MSG="Forced installer $FORCE_INSTALLER not found. Cannot continue."	
	fi
elif test $FORCE_INSTALLER = "yumex-dnf" ; then
	if test $CHECK_YUMEXDNFVIEWER = "yes" ; then
		SELECTED_INSTALLER="yumex-dnf"
	else 
                EXIT_NOW="yes"
                EXIT_MSG="Forced installer $FORCE_INSTALLER not found. Cannot continue."	
	fi

elif test $FORCE_INSTALLER = "gpk-update-viewer" ; then
        if test $CHECK_GPKUPDATEVIEWER = "yes" ; then
                SELECTED_INSTALLER="gpk-update-viewer"
        else
                EXIT_NOW="yes"
                EXIT_MSG="Forced installer $FORCE_INSTALLER not found. Cannot continue." 
	fi
elif test $FORCE_INSTALLER = "update-manager" ; then
        if test $CHECK_UPDATEMANAGER = "yes" ; then
                SELECTED_INSTALLER="update-manager"
        else
                EXIT_NOW="yes"
                EXIT_MSG="Forced installer $FORCE_INSTALLER not found. Cannot continue." 
        fi
elif test $FORCE_INSTALLER = "dnfdragora" ; then
        if test $CHECK_DNFDRAGORA = "yes" ; then
                SELECTED_INSTALLER="dnfdragora"
        else
                EXIT_NOW="yes"
                EXIT_MSG="Forced installer $FORCE_INSTALLER not found. Cannot continue." 
        fi
else
	echo "Chosing installer wisely..."
	if test $CHECK_DNFDRAGORA = "yes" ; then
        	SELECTED_INSTALLER="dnfdragora"
	elif test $CHECK_YUMEXDNFVIEWER = "yes" ; then
        	SELECTED_INSTALLER="yumex-dnf"
	elif test $CHECK_YUMEXVIEWER = "yes" ; then
        	SELECTED_INSTALLER="yumex"
	elif test $CHECK_GPKUPDATEVIEWER = "yes" ; then
        	SELECTED_INSTALLER="gpk-update-viewer"
	elif test $CHECK_UPDATEMANAGER = "yes" ; then
        	SELECTED_INSTALLER="update-manager"
	fi
fi

RPM_REQUIRES_INSTALLER=""
RPM_BUILDREQUIRES_INSTALLER=""
AC_DEFINE_UNQUOTED([SELECTED_INSTALLER],["$SELECTED_INSTALLER"],[Define which installer was selected])
if test $SELECTED_INSTALLER = "yumex" ; then
        AC_DEFINE_UNQUOTED([INSTALLER_BINARY],["$YUMEXVIEWER_INSTALLER"],[Define where is the installer binary])
	RPM_REQUIRES_INSTALLER="Requires: yumex"
	RPM_BUILDREQUIRES_INSTALLER="BuildRequires:yumex"
elif test $SELECTED_INSTALLER = "yumex-dnf" ; then
        AC_DEFINE_UNQUOTED([INSTALLER_BINARY],["$YUMEXDNFVIEWER_INSTALLER"],[Define where is the installer binary])
	RPM_REQUIRES_INSTALLER="Requires: yumex-dnf"
	RPM_BUILDREQUIRES_INSTALLER="BuildRequires:yumex-dnf"
elif test $SELECTED_INSTALLER = "gpk-update-viewer" ; then
        AC_DEFINE_UNQUOTED([INSTALLER_BINARY],["$GPKUPDATEVIEWER_INSTALLER"],[Define where is the installer binary])
	RPM_REQUIRES_INSTALLER="Requires: gnome-packagekit"
elif test $SELECTED_INSTALLER = "update-manager" ; then
        AC_DEFINE_UNQUOTED([INSTALLER_BINARY],["$UPDATEMANAGER_INSTALLER"],[Define where is the installer binary])
elif test $SELECTED_INSTALLER = "dnfdragora" ; then
        AC_DEFINE_UNQUOTED([INSTALLER_BINARY],["$DNFDRAGORA_INSTALLER"],[Define where is the installer binary])
else
	echo "*** No suitable installer found. You will have to update the system manually."
fi
AC_SUBST([RPM_REQUIRES_INSTALLER])
AC_SUBST([RPM_BUILDREQUIRES_INSTALLER])

# libmatenotify or libnotify?
RPM_BUILDREQUIRES_NOTIFY=""
RPM_REQUIRES_NOTIFY=""
FORCE_NOTIFY="no"
AC_ARG_ENABLE([notify],[AS_HELP_STRING([--enable-notify],[use the specified notificatin library: libnotify, libmatenotify (default: pick up the first available from the list)])],[FORCE_NOTIFY=$enableval])
PKG_CHECK_MODULES([LIBMATENOTIFY], [libmatenotify], [FOUND_LMN="yes"], [FOUND_LMN="no"])
PKG_CHECK_MODULES([LIBNOTIFY], [libnotify], [FOUND_LN="yes"], [FOUND_LN="no"])
if test $FORCE_NOTIFY = "libmatenotify" ; then
        if test $FOUND_LMN = "yes" ; then
                SELECTED_NOTIFY="libmatenotify"
		AC_DEFINE([HAVE_LIBMATENOTIFY],[1],[Define if libmatenotify was found])
		RPM_BUILDREQUIRES_NOTIFY="BuildRequires: libmatenotify-devel >= 1.2.0"
		RPM_REQUIRES_NOTIFY="Requires: libmatenotify >= 1.2.0"
        else
                EXIT_NOW="yes"
                EXIT_MSG="Forced notification library $FORCE_NOTIFY not found. Cannot continue."
        fi
elif test $FORCE_NOTIFY = "libnotify" ; then
        if test $FOUND_LN = "yes" ; then
                SELECTED_NOTIFY="libnotify"
		AC_DEFINE([HAVE_LIBNOTIFY],[1],[Define if libnotify was found])
                RPM_BUILDREQUIRES_NOTIFY="BuildRequires: libnotify-devel >= 0.7.0"
                RPM_REQUIRES_NOTIFY="Requires: libnotify >= 0.7.0"
        else
                EXIT_NOW="yes"
                EXIT_MSG="Forced notification library $FORCE_NOTIFY not found. Cannot continue."
        fi
else
	if test $FOUND_LN = "yes" ; then
                SELECTED_NOTIFY="libnotify"
		AC_DEFINE([HAVE_LIBNOTIFY],[1],[Define if libnotify was found])	
                RPM_BUILDREQUIRES_NOTIFY="BuildRequires: libnotify-devel >= 0.7.0"
                RPM_REQUIRES_NOTIFY="Requires: libnotify >= 0.7.0"
	elif test $FOUND_LMN = "yes" ; then
		SELECTED_NOTIFY="libmatenotify"
		AC_DEFINE([HAVE_LIBMATENOTIFY],[1],[Define if libmatenotify was found])
                RPM_BUILDREQUIRES_NOTIFY="BuildRequires: libmatenotify-devel >= 1.2.0"
                RPM_REQUIRES_NOTIFY="Requires: libmatenotify >= 1.2.0"
	else
		EXIT_NOW="yes"
		EXIT_MSG="Neither libnotify nor libmatenotify found. Cannot continue."
	fi
fi

AC_SUBST([RPM_BUILDREQUIRES_NOTIFY])
AC_SUBST([RPM_REQUIRES_NOTIFY])

if test $EXIT_NOW = "yes" ; then
        echo
        echo $EXIT_MSG
        echo
        exit
fi

# Checks for libraries.

# Check for packages
PKG_CHECK_MODULES([GLIB], [glib-2.0])
if test $SELECTED_BACKEND = "package-kit" ; then
	PKG_CHECK_MODULES([PKGKIT_BLIB], [packagekit-glib2])
fi

APPLET_FLAGS=`$PKG_CONFIG --cflags-only-I glib-2.0 $SELECTED_NOTIFY $LIBMATEPANELAPPLET $ADDITIONAL_PACKAGES`
APPLET_LIBS=`$PKG_CONFIG --libs glib-2.0 $SELECTED_NOTIFY $LIBMATEPANELAPPLET $ADDITIONAL_PACKAGES`
AC_SUBST([APPLET_FLAGS])
AC_SUBST([APPLET_LIBS])

# Makefiles
AC_CONFIG_FILES([Makefile src/Makefile po/Makefile.in po/Makefile images/Makefile])

# Service files
AC_CONFIG_FILES([src/org.mate.applets.SoftupdApplet.mate-panel-applet src/org.mate.panel.applet.SoftupdApplet.service mate-applet-softupd.spec])

# Icons
AC_DEFINE_UNQUOTED([APPLET_ICON_PATH], ["$PIXMAPDIR"], [Define icons location])

# Grand finale
AC_OUTPUT

echo
echo "Configuration complete:"
echo " * Selected GTK version: $SELECTED_GTK"
echo " * Selected backend: $SELECTED_BACKEND"
echo " * Selected installer: $SELECTED_INSTALLER"
echo " * Selected notification library: $SELECTED_NOTIFY"
echo
echo "Now, run make."
echo 
