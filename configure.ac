# Use v2.61 for backwards compat with Ubuntu 12.04 LTS
AC_PREREQ([2.61])
AC_INIT(Finit, 3.0-beta1, https://github.com/troglobit/finit/issues, finit, http://troglobit.com/finit.html)
AM_INIT_AUTOMAKE([1.11 foreign no-dist-gzip dist-xz subdir-objects])
AM_SILENT_RULES([yes])

AC_CONFIG_SRCDIR([finit.c])
AC_CONFIG_HEADER([config.h])
AC_CONFIG_FILES([Makefile plugins/Makefile])
AC_CONFIG_FILES([libite/Makefile])
AC_CONFIG_FILES([libuev/Makefile libuev/examples/Makefile])

# Older versions of autoconf (<2.58) do not have AC_CONFIG_MACRO_DIR()
#m4_include([m4/misc.m4])
#m4_include([m4/mroute.m4])
AC_CONFIG_MACRO_DIR([m4])

# Change default prefix, we're PID 1 after all ...
AC_PREFIX_DEFAULT([/])
datarootdir=/usr/share
AC_SUBST([datarootdir])

# Default plugins
#plugins_all="alsa-utils bootmisc dbus hwclock initctl netlink pidfile resolvconf time tty urandom x11-common"

# Checks for programs.
AC_PROG_CC
AC_PROG_LN_S
AC_PROG_INSTALL

# Configuration.
AC_HEADER_STDC
AC_CHECK_FUNCS(strstr getopt)

# Check for uint[8,16,32]_t
AC_TYPE_UINT8_T
AC_TYPE_UINT16_T
AC_TYPE_UINT32_T

# Needed for the libraries
AM_PROG_AR
LT_INIT([shared disable-static dlopen])
#LT_INIT([dlopen])
AM_CONDITIONAL([ENABLE_EXAMPLES], [test "$enable_examples" = yes])

# Check for configured Finit features
AC_ARG_ENABLE(embedded,
        AS_HELP_STRING([--enable-embedded], [Embedded defaults, BusBox getty, mdev etc.]))

AC_ARG_ENABLE(rw-rootfs,
        AS_HELP_STRING([--enable-rw-rootfs], [Remount / as read-write at bootstrap, not for embedded]))

AC_ARG_ENABLE(quiet,
        AS_HELP_STRING([--enable-quiet], [Quiet mode, reduce screen output to absolute minimum]))

AC_ARG_ENABLE(inetd,
        AS_HELP_STRING([--disable-inetd], [Disable inetd support if not needed, default enabled]))

AC_ARG_ENABLE(kernel-quiet,
        AS_HELP_STRING([--disable-kernel-quiet], [Disable kernel cmdline quiet, default enabled]))

AC_ARG_WITH(fifo,
        AS_HELP_STRING([--with-fifo=FIFO], [Old Finit initctl FIFO, default /dev/initctl]),
	[fifo=$withval], [fifo=/dev/initctl])

AC_ARG_WITH(config,
        AS_HELP_STRING([--with-config=CONFIG], [Finit CONFIG file, default $sysconfdir/finit.conf]),
	[conf=$withval], [conf='$(sysconfdir)/finit.conf'])

AC_ARG_WITH(heading,
        AS_HELP_STRING([--with-heading=GREET], [Display GREET instead of Finit version at boot]),
	[heading=$withval], [heading='$(PACKAGE_NAME) v$(PACKAGE_VERSION)'])

AC_ARG_WITH(plugindir,
        AS_HELP_STRING([--with-plugindir=DIR], [Finit plugins directory, default $libdir/finit/plugins]),
	[pkglibdir=$withval], [pkglibdir='$(libdir)/finit/plugins'])
AC_SUBST([pkglibdir])

#AC_ARG_WITH(plugins,
#        AS_HELP_STRING([--with-plugins=LIST], [Finit plugins to build, default $plugins_all]),
#	[plugins=$withval], [plugins=$plugins_all])
#plugins_src=""
#plugins_lib=""
#for plug in $plugins; do
#    plugins_src="${plugins_src} ${plug}.c"
#    plugins_lib="${plugins_lib} ${plug}.la"
#done    
#AC_SUBST([plugins_src])
#AC_SUBST([plugins_lib])

AC_ARG_WITH(rcsd,
        AS_HELP_STRING([--with-rcsd=DIR], [Finit rc.d/ directory, default $sysconfdir/finit.d]),
	[rcsd=$withval], [rcsd='$(sysconfdir)/finit.d'])
AC_SUBST(rcsd)

AC_ARG_WITH(console,
        AS_HELP_STRING([--with-console=DEV], [Console to use, default: /dev/console]),
	[console=$withval], [console=/dev/console])

AC_ARG_WITH(shell,
        AS_HELP_STRING([--with-shell=PATH], [Fallback shell to use, default: /bin/sh]),
	[shell=$withval], [shell=/bin/sh])

AC_ARG_WITH(username,
        AS_HELP_STRING([--with-username=USER], [User for 'startx' .conf option, default: root]),
	[username=$withval], [username=root])

AC_ARG_WITH(hostname,
        AS_HELP_STRING([--with-hostname=NAME], [Hostname, if /etc/hostname is missing, default: noname]),
	[hostname=$withval], [hostname=noname])

AC_ARG_WITH(runlevel,
        AS_HELP_STRING([--with-runlevel=N], [Runlevel to switch to after bootstrap, default: 2]),
	[runlevel=$withval], [runlevel=2])

### Enable features ###########################################################################

# Create config.h from selected features and fallback defautls
AS_IF([test "x$enable_embedded" = "xyes"],[
	AC_DEFINE(EMBEDDED_SYSTEM, 1,                [Target an embedded system with BusyBox])
	AC_DEFINE(SETUP_DEVFS,     "/sbin/mdev -s",  [Command to populate /dev])
	AC_DEFINE(GETTY,           "/sbin/getty -L", [Command to start getty on login prompt])
        AC_DEFINE(GETTY_BUSYBOX,   1,                [Use BusyBox getty call syntax])
        AC_DEFINE(BAUDRATE,        115200,           [Default getty baudrate])
	],[
        AC_DEFINE(SETUP_DEVFS,     "/sbin/udevd --daemon", [Command to populate /dev])
	AC_DEFINE(GETTY,           "/sbin/getty -8", [Command to start getty on login prompt])
        AC_DEFINE(GETTY_AGETTY,    1,                [Use standard agetty call syntax])
        AC_DEFINE(BAUDRATE,        38400,            [Default getty baudrate])])

AS_IF([test "x$enable_rw-rootfs" = "xyes"],[
        AC_DEFINE(REMOUNT_ROOTFS,  1, [Remount / as rw at boot])])

AS_IF([test "x$enable_quiet" = "xyes"],[
        AC_DEFINE(VERBOSE_MODE,  0, [Quiet Finit])
	],[
	AC_DEFINE(VERBOSE_MODE,  1, [Verbose output, default])])

### Disable features ###########################################################################
AS_IF([test "x$enable_inetd" != "xno"],enable_inetd="yes",[
	AC_DEFINE(INETD_DISABLED,  1, [Disable built-in inetd])])

AS_IF([test "x$enable_kernel-quiet" != "xno"],[
        AC_DEFINE(KERNEL_QUIET,  0, [Will print stopping/starting/etc messages])
	],[
	AC_DEFINE(KERNEL_QUIET,  1, [Silent after initial boot, default])
	])

### With features ##############################################################################
AS_IF([test "x$with_fifo" != "xno"],[
	AS_IF([test "x$fifo" = "xyes"],[
		AC_MSG_ERROR([Must supply argument])])
	],[
	fifo='/dev/initctl'])
AC_SUBST(fifo)

AS_IF([test "x$with_confing" != "xno"],[
	AS_IF([test "x$config" = "xyes"],[
		AC_MSG_ERROR([Must supply argument])])
	],[
	conf='$(sysconfdir)/finit.conf'])
AC_SUBST(conf)

AS_IF([test "x$with_heading" != "xno"],[
	AS_IF([test "x$heading" = "xyes"],[
		AC_MSG_ERROR([Must supply argument])])
	],[
	heading='$(libdir)/finit'])
AC_DEFINE_UNQUOTED(INIT_HEADING, "$heading", [Displayed by Finit at boot])

#AS_IF([test "x$with_plugindir" != "xno"],[
#	AS_IF([test "x$plugindir" = "xyes"],[
#		AC_MSG_ERROR([Must supply argument])])
#	],[
#	plugindir='$(libdir)/finit'])
#AC_SUBST(plugindir)

#AS_IF([test "x$with_rcsd" != "xno"],[
#	AS_IF([test "x$rcsd" = "xyes"],[
#		AC_MSG_ERROR([Must supply argument])])
#	],[
#	rcsd='$(sysconfdir)/finit.d'])

AS_IF([test "x$with_console" != "xno"],[
	AS_IF([test "x$console" = "xyes"],[
		AC_MSG_ERROR([Must supply argument])])
	],[
	console=/dev/console])
AC_DEFINE_UNQUOTED(CONSOLE, "$console", [Default console to use])

AS_IF([test "x$with_shell" != "xno"],[
	AS_IF([test "x$shell" = "xyes"],[
		AC_MSG_ERROR([Must supply argument])])
	],[
	shell=/bin/sh])
AC_DEFINE_UNQUOTED(FALLBACK_SHELL, "$shell", [Used if no login getty])

AS_IF([test "x$with_username" != "xno"],[
	AS_IF([test "x$username" = "xyes"],[
		AC_MSG_ERROR([Must supply argument])])
	],[
	username=root])
AC_DEFINE_UNQUOTED(DEFUSER, "$username", [Used for startx .conf option])

AS_IF([test "x$with_hostname" != "xno"],[
	AS_IF([test "x$hostname" = "xyes"],[
		AC_MSG_ERROR([Must supply argument])])
	],[
	hostname=noname])
AC_DEFINE_UNQUOTED(DEFHOST, "$hostname", [For /etc/hostname])

AS_IF([test "x$with_runlevel" != "xno"],[
	AS_IF([test "x$runlevel" = "xyes"],[
		AC_MSG_ERROR([Runlevel must be a number, 1-9, but not 6 (reboot)])])
	AC_DEFINE_UNQUOTED(RUNLEVEL, $runlevel, [Runlevel to start after S])
	],[
	AC_DEFINE(RUNLEVEL, 2, [Default runlevel to start after S])])

# Check if user has downloaded a GitHub ZIP file and attempt to fix it.
AC_MSG_CHECKING([status of GIT submodules])
if test ! -e libite/lite.h; then
    AC_MSG_RESULT([Missing!])
    AC_MSG_CHECKING([if we can update all submodules])
    run=`git submodule update --init`
    if test $? -ne 0; then
       AC_MSG_RESULT([Failed!  No Internet access?])
       exit 1
    fi
    AC_MSG_RESULT([Yes, all fixed!])
else
    AC_MSG_RESULT([OK])
fi

# Control build with automake flags
AM_CONDITIONAL(STATIC, [test "x$enable_static" = "xyes"])
AM_CONDITIONAL(INETD, [test "x$enable_inetd" = "xyes"])

# Generate all files
AC_OUTPUT
