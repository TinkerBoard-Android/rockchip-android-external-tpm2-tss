# SPDX-License-Identifier: BSD-2-Clause
# Copyright (c) 2015 - 2020 Intel Corporation
# Copyright (c) 2018 - 2020 Fraunhofer SIT sponsored by Infineon Technologies AG
# All rights reserved.

AC_INIT([tpm2-tss],
        [2.4.0],
        [https://github.com/tpm2-software/tpm2-tss/issues],
        [],
        [https://github.com/tpm2-software/tpm2-tss])
AC_CONFIG_MACRO_DIR([m4])
AM_INIT_AUTOMAKE([foreign
                  subdir-objects])
m4_ifdef([AM_SILENT_RULES], [AM_SILENT_RULES([yes])]) #Backward compatible setting of "silent-rules"

AC_CONFIG_HEADERS([config.h])

AC_CONFIG_FILES([Makefile Doxyfile lib/tss2-sys.pc lib/tss2-esys.pc lib/tss2-mu.pc lib/tss2-tcti-device.pc lib/tss2-tcti-mssim.pc lib/tss2-rc.pc lib/tss2-tctildr.pc lib/tss2-fapi.pc])

# propagate configure arguments to distcheck
AC_SUBST([DISTCHECK_CONFIGURE_FLAGS],[$ac_configure_args])

AC_CANONICAL_HOST

AX_IS_RELEASE(dash-version)
AX_CHECK_ENABLE_DEBUG([info])

AC_PROG_CC
AC_PROG_CXX
AC_PROG_LN_S
AC_USE_SYSTEM_EXTENSIONS
LT_INIT()
LT_LIB_DLLOAD
PKG_INSTALLDIR()

# Check OS and set library and compile flags accordingly
case "${host_os}" in
    *nto-qnx*)
        HOSTOS='QNX'
        ADD_COMPILER_FLAG([-D_QNX_SOURCE])
        LIBSOCKET_LDFLAGS="-lsocket"
        ;;
    *bsd* | *BSD*)
        HOSTOS='BSD'
        LIBSOCKET_LDFLAGS=""
        ;;
    *)
        #Assume linux
        HOSTOS='Linux'
        LIBSOCKET_LDFLAGS=""
        ;;
esac
AC_SUBST([LIBSOCKET_LDFLAGS])
AC_SUBST([HOSTOS])

# runstatedir is only defined in the yet unreleased Autoconf 2.70
AC_ARG_WITH([runstatedir],
            AS_HELP_STRING([--with-runstatedir=<dir>],
                           [replacement for --runstatedir in Autoconf <2.70]),
            [runstatedir="$withval"])
AS_IF([test -z "$runstatedir"], [runstatedir="${localstatedir}/run"])
AC_SUBST([runstatedir])

AX_RECURSIVE_EVAL([$sysconfdir], [SYSCONFDIR])
AC_DEFINE_UNQUOTED([SYSCONFDIR], ["$SYSCONFDIR"], [System config dir])

AC_ARG_WITH([userstatedir],
            [AS_HELP_STRING([--with-userstatedir=<dir>],
                            [sets the relative path in the user's home (default is ".local/share")])],,
            [with_userstatedir=.local/share])
AC_SUBST(userstatedir, $with_userstatedir)

AC_ARG_WITH([sysusersdir],
            [AS_HELP_STRING([--with-sysusersdir=<dir>],
                            [sets the sysusers.d directory (default is "${sysconfdir}/sysusers.d")])],,
            [with_sysusersdir="${sysconfdir}/sysusers.d"])
AC_SUBST(sysusersdir, $with_sysusersdir)

AC_ARG_WITH([tmpfilesdir],
            [AS_HELP_STRING([--with-tmpfilesdir=<dir>],
                            [sets the tmpfiles.d directory (default is "${sysconfdir}/tmpfiles.d")])],,
            [with_tmpfilesdir="${sysconfdir}/tmpfiles.d"])
AC_SUBST(tmpfilesdir, $with_tmpfilesdir)

AC_ARG_ENABLE([unit],
            [AS_HELP_STRING([--enable-unit],
                            [build cmocka unit tests])],,
            [enable_unit=no])
m4_define([cmocka_min_version], [1.0])
m4_define([cmocka_err], [Unit test enabled, but cmocka missing or version requirements not met. cmocka version must be >= cmocka_min_version])
AS_IF([test "x$enable_unit" != xno],
      [PKG_CHECK_MODULES([CMOCKA],
                         [cmocka >= cmocka_min_version],,
                         [AC_MSG_ERROR([cmocka_err])])])
AM_CONDITIONAL([UNIT], [test "x$enable_unit" != xno])

AC_ARG_ENABLE([esapi],
            [AS_HELP_STRING([--disable-esapi],
                            [don't build the esapi layer])],,
            [enable_esapi=yes])

AM_CONDITIONAL(ESAPI, test "x$enable_esapi" = "xyes")

AC_CHECK_FUNC([strndup],[],[AC_MSG_ERROR([strndup function not found])])
AC_CHECK_FUNCS([reallocarray])
AC_ARG_ENABLE([fapi],
            [AS_HELP_STRING([--enable-fapi],
                            [build the fapi layer (default is yes)])],
            [enable_fapi=$enableval],
            [enable_fapi=yes])

AM_CONDITIONAL(FAPI, test "x$enable_fapi" = "xyes")

AC_ARG_ENABLE([fapi-async-tests],
    AS_HELP_STRING([--enable-fapi-async-tests],
               [Force fapi to spin asynchronously. (NOT FOR PRODUCTION!)]),,
    [enable_fapi_async_tests=no])
AS_IF([test "x$enable_fapi_async_tests" = "xyes"],
    AC_DEFINE([TEST_FAPI_ASYNC], [1], [FAPI forced async spinning]))
AS_IF([test "x$enable_fapi_async_tests" = "xyes"],
    AC_MSG_WARN("FAPI compiled with asynchronous spinning testing. NOT FOR PRODUCTION!"))

AC_ARG_ENABLE([tcti-device-async],
    AS_HELP_STRING([--enable-tcti-device-async],
	           [Enable asynchronus operation on TCTI device
		    (note: This needs to be supported by the kernel driver).]),,
    [enable_tcti_device_async=no])
AS_IF([test "x$enable_tcti_device_async" = "xyes"],
	AC_DEFINE([TCTI_ASYNC],[1], [TCTI ASYNC MODE]))

AC_ARG_ENABLE([tcti-partial-reads],
    AS_HELP_STRING([--enable-tcti-partial-reads],
	           [Enable partial reads for TCTI device
		    (note: This needs to be supported by the kernel driver).]),,
    [enable_tcti_partial_reads=no])
AS_IF([test "x$enable_tcti_partial_reads" = "xyes"],
	AC_DEFINE([TCTI_PARTIAL_READ],[1], [TCTI PARTIAL READ MODE]))

AC_ARG_WITH([crypto],
            [AS_HELP_STRING([--with-crypto={gcrypt,ossl}],
                            [sets the ESAPI crypto backend (default is OpenSSL)])],,
            [with_crypto=ossl])

AM_CONDITIONAL(ESYS_OSSL, test "x$with_crypto" = "xossl")
AM_CONDITIONAL(ESYS_GCRYPT, test "x$with_crypto" = "xgcrypt")

AS_IF([test "x$enable_esapi" = xyes],
      [AS_IF([test "x$with_crypto" = xgcrypt], [
           AM_PATH_LIBGCRYPT([1.6.0], [], [AC_MSG_ERROR([Missing required gcrypt library])])
           TSS2_ESYS_CFLAGS_CRYPTO="$LIBGCRYPT_CFLAGS"
           TSS2_ESYS_LDFLAGS_CRYPTO="$LIBGCRYPT_LIBS"
       ], [test "x$with_crypto" = xossl], [
           PKG_CHECK_MODULES([LIBCRYPTO], [libcrypto])
           AC_DEFINE([OSSL], [1], [OpenSSL cryptographic backend])
           TSS2_ESYS_CFLAGS_CRYPTO="$LIBCRYPTO_CFLAGS"
           TSS2_ESYS_LDFLAGS_CRYPTO="$LIBCRYPTO_LIBS"
       ], AC_MSG_ERROR([Bad value for --with-crypto $with_crypto]))])
AC_SUBST([TSS2_ESYS_CFLAGS_CRYPTO])
AC_SUBST([TSS2_ESYS_LDFLAGS_CRYPTO])

AS_IF([test "x$enable_fapi" != xno -a "x$enable_esapi" = "xno"],
    AC_MSG_ERROR([ESAPI has to be enabled to compile FAPI.]))

AS_IF([test "x$enable_fapi" != xno -a "x$with_crypto" != "xossl"],
    AC_MSG_ERROR([FAPI has to be compiled with OpenSSL]))

AS_IF([test "x$enable_fapi" = xyes ],
    PKG_CHECK_MODULES([JSON_C], [json-c]))

AS_IF([test "x$enable_fapi" = xyes ],
    PKG_CHECK_MODULES([CURL], [libcurl]))

AC_ARG_WITH([tctidefaultmodule],
            [AS_HELP_STRING([--with-tctidefaultmodule],
[The default TCTI module for ESAPI. (Default: libtss2-tcti-default.so)])],
            [AC_DEFINE_UNQUOTED([ESYS_TCTI_DEFAULT_MODULE],
                                ["$with_tctidefaultmodule"],
                                ["The default TCTI library file"])],
	    [with_tctidefaultmodule=libtss2-tcti-default.so])

AC_ARG_WITH([tctidefaultconfig],
            [AS_HELP_STRING([--with-tctidefaultconfig],
                            [The default tcti module's configuration.])],
            [AC_DEFINE_UNQUOTED([ESYS_TCTI_DEFAULT_CONFIG],
                                ["$with_tctidefaultconfig"],
                                ["The default TCTIs configuration string"])])

AC_ARG_ENABLE([tcti-device],
            [AS_HELP_STRING([--disable-tcti-device],
                            [don't build the tcti-device module])],,
            [enable_tcti_device=yes])
AM_CONDITIONAL([ENABLE_TCTI_DEVICE], [test "x$enable_tcti_device" != xno])
AS_IF([test "x$enable_tcti_device" = "xyes"],
	AC_DEFINE([TCTI_DEVICE],[1], [TCTI FOR DEV TPM]))

AC_ARG_ENABLE([tcti-mssim],
            [AS_HELP_STRING([--disable-tcti-mssim],
                            [don't build the tcti-mssim module])],,
            [enable_tcti_mssim=yes])
AM_CONDITIONAL([ENABLE_TCTI_MSSIM], [test "x$enable_tcti_mssim" != xno])
AS_IF([test "x$enable_tcti_mssim" = "xyes"],
	AC_DEFINE([TCTI_MSSIM],[1], [TCTI FOR MS SIMULATOR]))

AC_ARG_ENABLE([tcti-fuzzing],
            [AS_HELP_STRING([--enable-tcti-fuzzing],
                            [build the tcti-fuzzing module])],,
            [enable_tcti_fuzzing=no])
AM_CONDITIONAL([ENABLE_TCTI_FUZZING], [test "x$enable_tcti_fuzzing" != xno])
AS_IF([test "x$enable_tcti_fuzzing" = "xyes"],
	AC_DEFINE([TCTI_FUZZING],[1], [TCTI FOR FUZZING]))

AC_ARG_ENABLE([nodl],
              [AS_HELP_STRING([--enable-nodl],
                              [link against TCTIs directly, do not use dlopen])],
              [],
              [enable_nodl=no])
AM_CONDITIONAL([NO_DL], [test "x$enable_nodl" = "xyes"])
AS_IF([test "x$enable_nodl" = "xyes"],
      [AC_DEFINE([NO_DL],[1], [disable use of dlopen])])

#
# udev
#
AC_ARG_WITH([udevrulesdir],
            [AS_HELP_STRING([--with-udevrulesdir=DIR],[udev rules directory])],,
            [with_udevrulesdir=${libdir}/udev/rules.d])
AX_NORMALIZE_PATH([with_udevrulesdir])
AC_SUBST([udevrulesdir], [$with_udevrulesdir])
AC_ARG_WITH([udevrulesprefix],
            [AS_HELP_STRING([--with-udevrulesprefix=XY],[prefix for udev rules file])],
            [AC_SUBST([udevrulesprefix],[$with_udevrulesprefix])])
AM_CONDITIONAL(WITH_UDEVRULESPREFIX, [test -n "$with_udevrulesprefix"])
#
# enable integration tests and check for simulator binary
#
AC_ARG_ENABLE([integration],
    [AS_HELP_STRING([--enable-integration],
        [build and execute integration tests])],,
    [enable_integration=no])
AS_IF([test "x$enable_integration" = "xyes" -a "$HOSTOS" = "Linux"],
      [ERROR_IF_NO_PROG([tpm_server])
       ERROR_IF_NO_PROG([ss])
       ERROR_IF_NO_PROG([ps])
       ERROR_IF_NO_PROG([echo])
       ERROR_IF_NO_PROG([kill])
       ERROR_IF_NO_PROG([stdbuf])
       ERROR_IF_NO_PROG([sleep])
       ERROR_IF_NO_PROG([cat])
       ERROR_IF_NO_PROG([realpath])
       ERROR_IF_NO_PROG([dirname])
       ERROR_IF_NO_PROG([basename])
       ERROR_IF_NO_PROG([mktemp])
       ERROR_IF_NO_PROG([od])
       ERROR_IF_NO_PROG([awk])
       ERROR_IF_NO_PROG([expr])
       ERROR_IF_NO_PROG([grep])
       ERROR_IF_NO_PROG([env])
       ERROR_IF_NO_PROG([rm])
       AS_IF([test "x$with_crypto" != xossl -o "x$enable_esapi" != xyes],
           PKG_CHECK_MODULES([LIBCRYPTO],[libcrypto]))
       AC_CHECK_HEADER(uthash.h, [], [AC_MSG_ERROR([Can not find uthash.h. Please install uthash-dev])])
       AS_IF([test "x$enable_tcti_mssim" = xno],
              AC_MSG_ERROR([Integration tests can not be enabled without the TCTI_MSSIM module]))
       AC_SUBST([ENABLE_INTEGRATION], [$enable_integration])])
AM_CONDITIONAL([ENABLE_INTEGRATION],[test "x$enable_integration" = "xyes"])
#
# sanitizer compiler flags
#
AC_ARG_WITH([sanitizer],
            [AS_HELP_STRING([--with-sanitizer={none,address,undefined}],
                            [build with the given sanitizer])],,
            [with_sanitizer=none])
AS_CASE(["x$with_sanitizer"],
        ["xnone"],
        [],
        ["xaddress"],
        [
            SANITIZER_CFLAGS="-fsanitize=address -fno-omit-frame-pointer"
            SANITIZER_LDFLAGS="-lasan"
        ],
        ["xundefined"],
        [
            SANITIZER_CFLAGS="-fsanitize=undefined"
            SANITIZER_LDFLAGS="-lubsan"
        ],
        [AC_MSG_ERROR([Bad value for --with-sanitizer])])
AC_SUBST([SANITIZER_CFLAGS])
AC_SUBST([SANITIZER_LDFLAGS])
#
# fuzz testing
#
AC_ARG_WITH([fuzzing],
            [AS_HELP_STRING([--with-fuzzing={none,libfuzzer,ossfuzz}],
                            [fuzzing to build with (default is none)])],,
            [with_fuzzing=none])
AS_CASE(["x$with_fuzzing"],
        ["xnone"],
        [],
        ["xlibfuzzer"],
        [ADD_FUZZING_FLAG([-fsanitize=fuzzer])],
        ["xossfuzz"],
        [AS_IF([test "x$LIB_FUZZING_ENGINE" = "x"],
               AC_MSG_ERROR([OSS Fuzz testing requires LIB_FUZZING_ENGINE environment variable be set]))
         ADD_FUZZING_FLAG([-lFuzzingEngine])],
        [AC_MSG_ERROR([Bad value for --with-fuzzing])])
AM_CONDITIONAL([ENABLE_FUZZING],[test "x$with_fuzzing" != "xnone"])
AS_IF([test "x$with_fuzzing" != "xnone"],
      [AS_IF([test "x$enable_tcti_fuzzing" = xno],
             AC_MSG_ERROR([Fuzz tests can not be enabled without the TCTI_FUZZING module]))
       AS_IF([test "x$GEN_FUZZ" != "x1"],
             AC_MSG_ERROR([Fuzz tests can not be enabled without "GEN_FUZZ=1" variable]))])

AX_VALGRIND_CHECK

gl_LD_VERSION_SCRIPT

AC_ARG_WITH([maxloglevel],
            [AS_HELP_STRING([--with-maxloglevel={none,error,warning,info,debug,trace}],
                            [sets the maximum log level (default is trace)])],,
            [with_maxloglevel=trace])
AS_CASE(["x$with_maxloglevel"],
        ["xnone"],
        [AC_DEFINE_UNQUOTED([MAXLOGLEVEL], [0], ["Logging disabled"])],
        ["xerror"],
        [AC_DEFINE_UNQUOTED([MAXLOGLEVEL], [2], ["Error log level"])],
        ["xwarning"],
        [AC_DEFINE_UNQUOTED([MAXLOGLEVEL], [3], ["Warning log level"])],
        ["xinfo"],
        [AC_DEFINE_UNQUOTED([MAXLOGLEVEL], [4], ["Info log level"])],
        ["xdebug"],
        [AC_DEFINE_UNQUOTED([MAXLOGLEVEL], [5], ["Debug log level"])],
        ["xtrace"],
        [AC_DEFINE_UNQUOTED([MAXLOGLEVEL], [6], ["Trace log level"])],
        [AC_MSG_ERROR([Bad value for --with-maxloglevel])])

AC_ARG_ENABLE([defaultflags],
              [AS_HELP_STRING([--disable-defaultflags],
                              [Disable default preprocessor, compiler, and linker flags.])],,
              [enable_defaultflags=yes])
AS_IF([test "x$enable_defaultflags" = "xyes"],
      [
      AS_IF([test "x$enable_debug" = "xno"],
            [ADD_COMPILER_FLAG([-O2])
             AX_ADD_FORTIFY_SOURCE])
      ADD_COMPILER_FLAG([-std=c99])
      ADD_COMPILER_FLAG([-Wall])
      ADD_COMPILER_FLAG([-Wextra])
      ADD_COMPILER_FLAG([-Wformat-security])
      AS_IF([test "x$ax_is_release" = "xno"],
          [ADD_COMPILER_FLAG([-Werror])])
      ADD_COMPILER_FLAG([-fstack-protector-all])
      ADD_COMPILER_FLAG([-fpic])
      ADD_COMPILER_FLAG([-fPIC])
      # work around GCC bug #53119
      #   https://gcc.gnu.org/bugzilla/show_bug.cgi?id=53119
      ADD_COMPILER_FLAG([-Wno-missing-braces])
      ADD_COMPILER_FLAG([-Wstrict-overflow=5])
      ADD_LINK_FLAG([-Wl,--no-undefined])
      ADD_LINK_FLAG([-Wl,-z,noexecstack])
      ADD_LINK_FLAG([-Wl,-z,now])
      ADD_LINK_FLAG([-Wl,-z,relro])
      ])

AC_ARG_ENABLE([weakcrypto],
    [AS_HELP_STRING([--disable-weakcrypto],
	           [Disable crypto algorithms considered weak])],,
    [enable_weakcrypto=no])
AS_IF([test "x$enable_weakcrypto" = "xyes"],
	AC_DEFINE([DISABLE_WEAK_CRYPTO],[1],[DISABLE WEAK CRYPTO ALGORITHMS]))

AC_SUBST([PATH])

dnl ---------  Physical TPM device -----------------------

AC_ARG_WITH([ptpm],
            [AS_HELP_STRING([--with-ptpm=<device>],[TPM device])],
            [AS_IF([test \( -w "$with_ptpm" \)  -a \( -r "$with_ptpm" \)],
                   [AC_MSG_RESULT([success])
                    AC_SUBST([PTPM],[$with_ptpm])
                    AX_NORMALIZE_PATH([with_ptpm])
                    with_ptpm_set=yes],
                   [AC_MSG_ERROR([TPM device provided does not exist or is not writable])])],
            [with_ptpm_set=no])
AM_CONDITIONAL([PTPM],[test "x$with_ptpm_set" = "xyes"])

AC_ARG_WITH([ptpmtests],
            [AS_HELP_STRING([--with-ptpmtests=<case>],[Comma-separated values of possible tests: destructive,mandatory,optional] default is mandatory)],
            [AS_IF([test "x" =  x$(echo $with_ptpmtests | sed 's/destructive//g'  | sed 's/mandatory//g'  | sed 's/optional//g' | sed 's/,//g') ],
                   [AC_MSG_RESULT([success])
                    with_ptpmtests_set=yes],
                   [AC_MSG_ERROR([Illegal test type for pTPM tests.])])],
            [with_ptpmtests="mandatory"])

if echo $with_ptpmtests | grep destructive > /dev/null; then
    enable_ptpm_destructive="yes"
fi
AM_CONDITIONAL([PTPMDESTRUCTIVE],[test "x$enable_ptpm_destructive" = "xyes"])

if echo $with_ptpmtests | grep optional > /dev/null; then
    enable_ptpm_optional="yes"
fi
AM_CONDITIONAL([PTPMOPTIONAL],[test "x$enable_ptpm_optional" = "xyes"])

if echo $with_ptpmtests | grep mandatory > /dev/null; then
    enable_ptpm_mandatory="yes"
fi
AM_CONDITIONAL([PTPMMANDATORY],[test "x$enable_ptpm_mandatory" = "xyes"])

AM_CONDITIONAL([TESTPTPM],[test "x$with_ptpm_set" = xyes])

dnl --------- Doxy Gen -----------------------
DX_DOXYGEN_FEATURE(ON)
DX_DOT_FEATURE(OFF)
DX_HTML_FEATURE(ON)
DX_CHM_FEATURE(OFF)
DX_CHI_FEATURE(OFF)
DX_MAN_FEATURE(ON)
DX_RTF_FEATURE(ON)
DX_XML_FEATURE(OFF)
DX_PDF_FEATURE(OFF)
DX_PS_FEATURE(OFF)
DX_INIT_DOXYGEN($PACKAGE_NAME, [Doxyfile], [doxygen-doc])
AM_CONDITIONAL(DOXYMAN, [test $DX_FLAG_man -eq 1])

AS_IF([test "x$enable_doxygen_doc" != xno],
      [ERROR_IF_NO_PROG([doxygen])])

AX_CODE_COVERAGE
m4_ifdef([_AX_CODE_COVERAGE_RULES],
         [AM_CONDITIONAL(AUTOCONF_CODE_COVERAGE_2019_01_06, [true])],
         [AM_CONDITIONAL(AUTOCONF_CODE_COVERAGE_2019_01_06, [false])])
AX_ADD_AM_MACRO_STATIC([])

AM_COND_IF([ENABLE_TCTI_DEVICE], [],
           [AM_COND_IF([ENABLE_TCTI_MSSIM], [],
                       [AM_COND_IF([ENABLE_TCTI_FUZZING], [],
                                   [AC_MSG_WARN("No build-in TCTI module enabled")])])])

AM_COND_IF([ENABLE_TCTI_FUZZING], [
            AM_COND_IF([ENABLE_TCTI_DEVICE],
                       AC_MSG_ERROR([Fuzzing TCTI is meant to be built as the only TCTI: use --disable-tcti-device --disable-tcti-mssim]))
            AM_COND_IF([ENABLE_TCTI_MSSIM],
                       AC_MSG_ERROR([Fuzzing TCTI is meant to be built as the only TCTI: use --disable-tcti-device --disable-tcti-mssim]))
            AS_IF([test "x$CC" != "xclang"],
                       AC_MSG_ERROR("Fuzzing TCTI can only be used with clang"), [])
            ], [])

AC_OUTPUT

AC_MSG_RESULT([
    $PACKAGE_NAME $VERSION
    esapi:              $enable_esapi
    fapi:               $enable_fapi
    tctidefaultmodule:  $with_tctidefaultmodule
    tctidefaultconfig:  $with_tctidefaultconfig
    unit:               $enable_unit
    fuzzing:            $with_fuzzing
    debug:              $enable_debug
    maxloglevel:        $with_maxloglevel
    doxygen:            $DX_FLAG_doc $enable_doxygen_doc
    tcti-device-async:  $enable_tcti_device_async
    tcti-partial-read:  $enable_tcti_partial_reads
    crypto backend:     $with_crypto
    sysconfdir:         $sysconfdir
    localstatedir:      $localstatedir
    runstatedir:        $runstatedir
    sysusersdir:        $sysusersdir
    tmpfilesdir:        $tmpfilesdir
    userstatedir:       [\$HOME/]$with_userstatedir
])
