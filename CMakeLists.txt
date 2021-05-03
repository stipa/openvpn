cmake_minimum_required(VERSION 3.3)
project(openvpn)

# This CMake file implements building OpenVPN with CMAKE
#
# Note that this is *NOT* the official way to build openvpn on anything
# other than Windows despite working on other platforms too.
#
# This cmake file also makes a few assertions like lzo, lz4 being
# and OpenSSL having version 1.1.1+


# AddressSanitize
set(CMAKE_C_FLAGS_ASAN
        "-fsanitize=address -fno-optimize-sibling-calls -fsanitize-address-use-after-scope -fno-omit-frame-pointer -g -O1"
        CACHE STRING "Flags used by the C compiler during AddressSanitizer builds."
        FORCE)
set(CMAKE_CXX_FLAGS_ASAN
        "-fsanitize=address -fno-optimize-sibling-calls -fsanitize-address-use-after-scope -fno-omit-frame-pointer -g -O1"
        CACHE STRING "Flags used by the C++ compiler during AddressSanitizer builds."
        FORCE)


find_package(PkgConfig)
INCLUDE(CheckSymbolExists)
INCLUDE(CheckIncludeFiles)
INCLUDE(CheckTypeSize)
INCLUDE(CheckStructHasMember)

set(OPENVPN_VERSION_MAJOR 2)
set(OPENVPN_VERSION_MINOR 6)
set(OPENVPN_VERSION_PATCH _git)

if(NOT WIN32)
    add_definitions(-DPLUGIN_LIBDIR=\"/usr/local/lib/openvpn/plugins\"
            -DHAVE_CONFIG_H
            -DWC_NO_HARDEN )
endif()


# TODO remove later when msvc-config.h is removed
add_definitions(-DHAVE_CONFIG_H)

option(MBED "BUILD with mbed" OFF)
option(OLDOSSL "BUILD with old OpenSSL" OFF)
option(WOLFSSL "BUILD with wolfSSL" OFF)

set(MASTER YES)
if (EXISTS src/openvpn/networking_linuxdco.c)
    set(DCO YES)
else()
    set(DCO YES)
endif()

if(NOT ${DCO})
    add_definitions(-DENABLE_DEF_AUTH)
endif()

# Set the various defines for config.h.cmake.in
if (${CMAKE_SYSTEM_NAME} STREQUAL "Linux")
    set(LINUX TRUE)
else()
    set(LINUX FALSE)
endif ()

if(APPLE)
    set(TARGET_DARWIN YES)
endif()

if (${LINUX} OR WIN32)
    set(ENABLE_DCO YES)
endif()

if (${LINUX})
    set(ENABLE_ASYNC_PUSH YES)
    set(ENABLE_LINUXDCO YES)
    set(ENABLE_SITNL YES)
    set(HAVE_DECL_SO_MARK YES)
    set(HAVE_CMSGHDR YES)
    set(TARGET_LINUX 1)
    set(ENABLE_FEATURE_TUN_PERSIST 1)
    set(HAVE_LINUX_TYPES_H 1)
endif()

if(UNIX)
    set(PATH_SEPARATOR /)
    set(ENABLE_PORT_SHARE YES)
    set(HAVE_SA_FAMILY_T YES)
elseif(WIN32)
    set(PATH_SEPARATOR \\\\)
    set(TARGET_WIN32 YES)
    set(ENABLE_WINDCO YES)
endif()

CHECK_SYMBOL_EXISTS(chroot unistd.h HAVE_CHROOT)
CHECK_SYMBOL_EXISTS(chdir unistd.h HAVE_CHDIR)
CHECK_SYMBOL_EXISTS(dup unistd.h HAVE_DUP)
CHECK_SYMBOL_EXISTS(dup2 unistd.h HAVE_DUP2)
CHECK_SYMBOL_EXISTS(fork unistd.h HAVE_FORK)
CHECK_SYMBOL_EXISTS(execve unistd.h HAVE_EXECVE)
CHECK_SYMBOL_EXISTS(ftruncate unistd.h HAVE_FTRUNCATE)
CHECK_SYMBOL_EXISTS(setgid unistd.h HAVE_SETGID)
CHECK_SYMBOL_EXISTS(setuid unistd.h HAVE_SETUID)
CHECK_SYMBOL_EXISTS(getpeereid unistd.h HAVE_GETPEEREID)

CHECK_SYMBOL_EXISTS(epoll_create sys/epoll.h HAVE_EPOLL_CREATE)

CHECK_SYMBOL_EXISTS(gettimeofday sys/time.h HAVE_GETTIMEOFDAY)
CHECK_SYMBOL_EXISTS(basename libgen.h HAVE_BASENAME)
CHECK_SYMBOL_EXISTS(chsize io.h HAVE_CHSIZE)
CHECK_SYMBOL_EXISTS(daemon stdlib.h HAVE_DAEMON)
CHECK_SYMBOL_EXISTS(dirname libgen.h HAVE_DIRNAME)
CHECK_SYMBOL_EXISTS(getrlimit sys/resource.h HAVE_GETRLIMIT)
CHECK_SYMBOL_EXISTS(mlockall sys/mman.h HAVE_MLOCKALL)

CHECK_SYMBOL_EXISTS(sendmsg sys/socket.h HAVE_SENDMSG)
CHECK_SYMBOL_EXISTS(recvmsg sys/socket.h HAVE_RECVMSG)
CHECK_SYMBOL_EXISTS(openlog syslog.h HAVE_OPENLOG)
CHECK_SYMBOL_EXISTS(syslog syslog.h HAVE_SYSLOG)
CHECK_SYMBOL_EXISTS(getgrnam grp.h HAVE_GETGRNAM)
CHECK_SYMBOL_EXISTS(getpwnam pwd.h HAVE_GETPWNAM)
CHECK_SYMBOL_EXISTS(getsockname sys/socket.h HAVE_GETSOCKNAME)


CHECK_INCLUDE_FILES(linux/if_tun.h HAVE_LINUX_IF_TUN_H)
CHECK_INCLUDE_FILES(linux/sockios.h HAVE_LINUX_SOCKIOS_H)
CHECK_INCLUDE_FILES(dlfcn.h HAVE_DLFCN_H)
CHECK_INCLUDE_FILES(fcntl.h HAVE_FCNTL_H)
CHECK_INCLUDE_FILES(dmalloc.h HAVE_DMALLOC_H)
CHECK_INCLUDE_FILES(err.h HAVE_ERR_H)
CHECK_INCLUDE_FILES(sys/epoll.h HAVE_SYS_EPOLL_H)
CHECK_INCLUDE_FILES(poll.h HAVE_POLL_H)
CHECK_INCLUDE_FILES(sys/socket.h HAVE_SYS_SOCKET_H)
CHECK_INCLUDE_FILES(sys/time.h HAVE_SYS_TIME_H)
CHECK_INCLUDE_FILES(netdb.h HAVE_NETDB_H)
CHECK_INCLUDE_FILES(unistd.h HAVE_UNISTD_H)
CHECK_INCLUDE_FILES(sys/un.h HAVE_SYS_UN_H)
CHECK_INCLUDE_FILES(libgen.h HAVE_LIBGEN_H)
CHECK_INCLUDE_FILES(net/if.h HAVE_NET_IF_H)
CHECK_INCLUDE_FILES(netinet/ip.h HAVE_NETINET_IP_H)
CHECK_INCLUDE_FILES(net/if_tun.h HAVE_NET_IF_TUN_H)
CHECK_INCLUDE_FILES(netinet/in.h HAVE_NETINET_IN_H)
CHECK_INCLUDE_FILES(arpa/inet.h HAVE_ARPA_INET_H)
CHECK_INCLUDE_FILES(net/if_utun.h HAVE_NET_UTUN_H)
CHECK_INCLUDE_FILES(sys/ioctl.h HAVE_SYS_IOCTL_H)
CHECK_INCLUDE_FILES(resolv.h HAVE_RESOLV_H)
CHECK_INCLUDE_FILES(sys/inotify.h HAVE_SYS_INOTIFY_H)
CHECK_INCLUDE_FILES(sys/uio.h HAVE_SYS_UIO_H)
CHECK_INCLUDE_FILES(syslog.h HAVE_SYSLOG_H)
CHECK_INCLUDE_FILES(sys/wait.h HAVE_SYS_WAIT_H)
CHECK_INCLUDE_FILES(grp.h HAVE_GRP_H)
CHECK_INCLUDE_FILES(pwd.h HAVE_PWD_H)
CHECK_INCLUDE_FILES(sys/mman.h HAVE_SYS_MMAN_H)

# Is this obscure header needed anywhere?!
CHECK_INCLUDE_FILES(Netinet/in_systm.h HAVE_NETINET_IN_SYSTM_H)


SET(CMAKE_EXTRA_INCLUDE_FILES netinet/ip.h)
CHECK_TYPE_SIZE("struct in_pktinfo" IN_PKTINFO)
CHECK_STRUCT_HAS_MEMBER("struct in_pktinfo" ipi_spec_dst netinet/ip.h HAVE_IPI_SPEC_DST)
CHECK_TYPE_SIZE("struct msghdr" MSGHDR)
SET(CMAKE_EXTRA_INCLUDE_FILES)

find_program(IFCONFIG_PATH ifconfig)
find_program(IPROUTE_PATH ip)
find_program(ROUTE_PATH route)


function(add_library_deps target)
    if (${MBED})
        target_include_directories(${target} PRIVATE $ENV{HOME}/oss/mbedtls/include)
        message("Building ${target} for mbed TLS")
        target_link_libraries(openvpn -L$ENV{HOME}/oss/mbedtls/library -L/usr/local/opt/lzo/lib  -llzo2 -lmbedtls -lmbedx509 -lmbedcrypto)

    elseif(${OLDOSSL})
        set(OSSLDIR /usr/local/Cellar/openssl/1.0.2t)
        target_include_directories(${target} PRIVATE ${OSSLDIR}/include/)
        find_library(CRYPTO NAMES crypto PATH ${OSSLDIR})
        find_library(OPENSSL NAMES ssl PATH ${OSSLDIR})

        message("Building ${target} for old OpenSSL")

    elseif(${WOLFSSL})

        pkg_search_module(wolfssl wolfssl REQUIRED)
        target_link_libraries(${target} ${wolfssl_LINK_LIBRARIES})
        target_include_directories(${target} PRIVATE  ${wolfssl_INCLUDE_DIRS}/wolfssl)
        message("Building ${target} for WolfSSL: ${wolfssl_LINK_LIBRARIES} ${wolfssl_INCLUDE_DIRS}/wolfsll" )
    else ()
        set(ENABLE_X509ALTUSERNAME YES)

        find_library(CRYPTO NAMES crypto PATH ${OSSLDIR})
        find_library(OPENSSL NAMES ssl PATH ${OSSLDIR})

        set(OSSLDIR /usr/local/opt/openssl@1.1/)
        target_include_directories(${target} PRIVATE ${OSSLDIR}/include/)
        target_link_libraries(${target} -L${OSSLDIR}/lib -lcrypto -lssl)

        message("Building ${target} for default OpenSSL")
    endif ()

    if (WIN32)
        find_path(OVPN_DCO_WIN_INCLUDE_DIRS "ovpn-dco-win/uapi/ovpn-dco.h")
        target_include_directories(${target} PRIVATE "${OVPN_DCO_WIN_INCLUDE_DIRS}/ovpn-dco-win/")
    endif()
endfunction()

if (${MBED})
    set(ENABLE_CRYPTO_MBEDTLS YES)
    set(BUILDDIR "$ENV{HOME}/oss/openvpn-mbed/")
elseif(${OLDOSSL})
    set(BUILDDIR "$ENV{HOME}/oss/openvpn-ossl-old/")
elseif(${WOLFSSL})
    set(ENABLE_CRYPTO_OPENSSL YES)
    set(ENABLE_CRYPTO_WOLFSSL YES)
    set(ENABLE_X509ALTUSERNAME YES)
    set(BUILDDIR "$ENV{HOME}/oss/openvpn-wolf/")
else ()
    set(ENABLE_CRYPTO_OPENSSL YES)
    set(ENABLE_X509ALTUSERNAME YES)
    set(BUILDDIR "$ENV{HOME}/oss/openvpn-ossl/")
endif ()

include_directories(${CMAKE_CURRENT_SOURCE_DIR} src/compat /usr/local/include include)

if (WIN32)
    #include_directories("C:\\Users\\User\\source\\repos\\tap-windows6\\src")
    #include_directories("C:\\Users\\User\\source\\repos\\ovpn-dco-win")


    find_package(OpenSSL REQUIRED)


    link_libraries(OpenSSL::SSL OpenSSL::Crypto ws2_32.lib Crypt32.lib fwpuclnt.lib iphlpapi.lib
            wininet.lib setupapi.lib rpcrt4.lib wtsapi32.lib Ncrypt.lib )
    add_definitions(-D_CRT_SECURE_NO_WARNINGS -D_CRT_NONSTDC_NO_DEPRECATE)
endif()


#CMAKE_SHARED_LINKER_FLAGS=-llzo
configure_file(config.h.cmake.in config.h)
configure_file(include/openvpn-plugin.h.in openvpn-plugin.h)

include_directories(${CMAKE_CURRENT_BINARY_DIR})

message(${PROJECT_SOURCE_DIR}/config.h.cmake.in config.h)

set(SOURCE_FILES
        ${CMAKE_CURRENT_BINARY_DIR}/config.h
        ${CMAKE_CURRENT_BINARY_DIR}/openvpn-plugin.h

        include/openvpn-plugin.h
        src/compat/compat-basename.c
        src/compat/compat-daemon.c
        src/compat/compat-dirname.c
        src/compat/compat-gettimeofday.c
        src/compat/compat-strsep.c
        src/compat/compat-versionhelpers.h
        src/openvpn/argv.c
        src/openvpn/argv.h
        src/openvpn/base64.c
        src/openvpn/base64.h
        src/openvpn/basic.h
        src/openvpn/block_dns.h
        src/openvpn/block_dns.c
        src/openvpn/buffer.c
        src/openvpn/buffer.h
        src/openvpn/circ_list.h
        src/openvpn/clinat.c
        src/openvpn/clinat.h
        src/openvpn/common.h
        src/openvpn/comp-lz4.c
        src/openvpn/comp-lz4.h
        src/openvpn/comp.c
        src/openvpn/comp.h
        src/openvpn/compstub.c
        src/openvpn/console.c
        src/openvpn/console_builtin.c
        src/openvpn/console.h
        src/openvpn/crypto.c
        src/openvpn/crypto.h
        src/openvpn/crypto_backend.h
        src/openvpn/crypto_openssl.c
        src/openvpn/crypto_openssl.h
        src/openvpn/crypto_mbedtls.c
        src/openvpn/crypto_mbedtls.h
        src/openvpn/cryptoapi.c
        src/openvpn/cryptoapi.h
        src/openvpn/dhcp.c
        src/openvpn/dhcp.h
        src/openvpn/errlevel.h
        src/openvpn/error.c
        src/openvpn/error.h
        src/openvpn/event.c
        src/openvpn/event.h
        src/openvpn/fdmisc.c
        src/openvpn/fdmisc.h
        src/openvpn/forward.c
        src/openvpn/forward.h
        src/openvpn/fragment.c
        src/openvpn/fragment.h
        src/openvpn/gremlin.c
        src/openvpn/gremlin.h
        src/openvpn/helper.c
        src/openvpn/helper.h
        src/openvpn/httpdigest.c
        src/openvpn/httpdigest.h
        src/openvpn/init.c
        src/openvpn/init.h
        src/openvpn/integer.h
        src/openvpn/interval.c
        src/openvpn/interval.h
        src/openvpn/list.c
        src/openvpn/list.h
        src/openvpn/lladdr.c
        src/openvpn/lladdr.h
        src/openvpn/lzo.c
        src/openvpn/lzo.h
        src/openvpn/manage.c
        src/openvpn/manage.h
        src/openvpn/mbuf.c
        src/openvpn/mbuf.h
        src/openvpn/memdbg.h
        src/openvpn/misc.c
        src/openvpn/misc.h
        src/openvpn/mroute.c
        src/openvpn/mroute.h
        src/openvpn/mss.c
        src/openvpn/mss.h
        src/openvpn/mstats.c
        src/openvpn/mstats.h
        src/openvpn/mtcp.c
        src/openvpn/mtcp.h
        src/openvpn/mtu.c
        src/openvpn/mtu.h
        src/openvpn/mudp.c
        src/openvpn/mudp.h
        src/openvpn/multi.c
        src/openvpn/multi.h
        src/openvpn/ntlm.c
        src/openvpn/ntlm.h
        src/openvpn/occ.c
        src/openvpn/occ.h
        src/openvpn/openvpn.c
        src/openvpn/openvpn.h
        src/openvpn/options.c
        src/openvpn/options.h
        src/openvpn/otime.c
        src/openvpn/otime.h
        src/openvpn/packet_id.c
        src/openvpn/packet_id.h
        src/openvpn/perf.c
        src/openvpn/perf.h
        src/openvpn/pf.c
        src/openvpn/pf.h
        src/openvpn/ping.c
        src/openvpn/ping.h
        src/openvpn/pkcs11.c
        src/openvpn/pkcs11.h
        src/openvpn/pkcs11_backend.h
        src/openvpn/pkcs11_openssl.c
        src/openvpn/pkcs11_mbedtls.c
        src/openvpn/platform.c
        src/openvpn/platform.h
        src/openvpn/plugin.c
        src/openvpn/plugin.h
        src/openvpn/pool.c
        src/openvpn/pool.h
        src/openvpn/proto.c
        src/openvpn/proto.h
        src/openvpn/proxy.c
        src/openvpn/proxy.h
        src/openvpn/ps.c
        src/openvpn/ps.h
        src/openvpn/push.c
        src/openvpn/push.h
        src/openvpn/pushlist.h
        src/openvpn/reliable.c
        src/openvpn/reliable.h
        src/openvpn/route.c
        src/openvpn/route.h
        src/openvpn/schedule.c
        src/openvpn/schedule.h
        src/openvpn/session_id.c
        src/openvpn/session_id.h
        src/openvpn/shaper.c
        src/openvpn/shaper.h
        src/openvpn/sig.c
        src/openvpn/sig.h
        src/openvpn/socket.c
        src/openvpn/socket.h
        src/openvpn/socks.c
        src/openvpn/socks.h
        src/openvpn/ssl.c
        src/openvpn/ssl.h
        src/openvpn/ssl_backend.h
        src/openvpn/ssl_common.h
        src/openvpn/ssl_openssl.c
        src/openvpn/ssl_openssl.h
        src/openvpn/ssl_mbedtls.c
        src/openvpn/ssl_mbedtls.h
        src/openvpn/ssl_verify.c
        src/openvpn/ssl_verify.h
        src/openvpn/ssl_verify_backend.h
        src/openvpn/ssl_verify_openssl.c
        src/openvpn/ssl_verify_openssl.h
        src/openvpn/ssl_verify_mbedtls.c
        src/openvpn/ssl_verify_mbedtls.h
        src/openvpn/status.c
        src/openvpn/status.h
        src/openvpn/syshead.h
        src/openvpn/tls_crypt.c
        src/openvpn/tun.c
        src/openvpn/tun.h
        src/openvpn/win32.c
        src/openvpn/win32.h
        src/openvpn/networking_sitnl.c
        src/openvpn/auth_token.c
        src/openvpn/auth_token.h
        src/openvpn/ssl_ncp.c
        src/openvpn/ssl_ncp.h
        src/openvpn/env_set.c
        src/openvpn/env_set.h
        src/openvpn/run_command.c
        src/openvpn/run_command.h
        src/openvpn/vlan.c
        src/openvpn/vlan.h
        )

#if (WIN32)
#    LIST(APPEND SOURCE_FILES
#            config-msvc.h
#            config-msvc-version.h
#            )
#else()
#    LIST(APPEND SOURCE_FILES
#        ${BUILDDIR}/config-version.h
#        ${BUILDDIR}/config.h)
#endif()

if (${MASTER})
    LIST(APPEND SOURCE_FILES
            src/openvpn/ssl_util.c
            src/openvpn/ssl_util.h
            )
endif()

if (${DCO})
    LIST(APPEND SOURCE_FILES

            src/openvpn/networking_linuxdco.c
            src/openvpn/networking_linuxdco.h
            src/openvpn/networking_windco.c
            src/openvpn/networking_windco.h
            src/openvpn/dco.c
            src/openvpn/dco.h
            src/openvpn/dco.h
            )
endif ()


add_executable(openvpn ${SOURCE_FILES})

find_library(lz4 lz4)
target_link_libraries(openvpn ${LZO_LIBRARY} ${lz4})

add_library_deps(openvpn)

pkg_search_module(lzo2 lzo2 REQUIRED)
target_link_libraries(openvpn ${lzo2_LINK_LIBRARIES})
target_include_directories(openvpn PRIVATE ${LZO2_INCLUDE_DIRS})

if (NOT WIN32)
    target_link_libraries(openvpn -lresolv)
endif()

if (${LINUX})
    target_link_libraries(openvpn -ldl)

    find_package(PkgConfig)
    pkg_search_module(LIBNL REQUIRED libnl-genl-3.0)

    set(OVPN_DCO_SOURCE "$ENV{HOME}/ovpn-dco" CACHE PATH "ovpn-dco kernel module source tree")

    target_include_directories(openvpn PRIVATE ${OVPN_DCO_SOURCE}/include/uapi ${LIBNL_INCLUDE_DIRS})
    target_link_libraries(openvpn ${LIBNL_LIBRARIES})

endif ()


find_package(cmocka CONFIG REQUIRED)

SET(unit_tests "test_packet_id" "test_crypto" "test_ncp" "test_tls_crypt" "test_auth_token")

if (${DCO})
    LIST(APPEND unit_tests "test_misc")
endif()

if (${CMAKE_SYSTEM_NAME} STREQUAL "Linux")
    LIST(APPEND unit_tests "test_networking")
endif()

if(NOT WIN32)
FOREACH (test_name ${unit_tests})
    ADD_EXECUTABLE(${test_name}
            tests/unit_tests/openvpn/${test_name}.c
            tests/unit_tests/openvpn/mock_msg.c
            src/openvpn/buffer.c
            src/openvpn/platform.c
            src/openvpn/crypto_mbedtls.c
            src/openvpn/crypto_openssl.c
            src/openvpn/crypto.c
            src/openvpn/otime.c
            src/openvpn/packet_id.c
            src/openvpn/base64.c
            src/openvpn/mtu.c
            src/openvpn/networking_sitnl.c
            )

    if (${DCO})
        target_sources(${test_name} PRIVATE src/openvpn/ssl_util.c)
    endif()

    add_library_deps(${test_name})
    target_link_libraries(${test_name} ${CMOCKA_LIBRARY}  -L/usr/local/opt/lzo/lib -llzo2 ${lz4})

    target_include_directories(${test_name} PRIVATE src/openvpn)
    if (${test_name} STREQUAL "test_networking" OR ${test_name} STREQUAL "test_tls_crypt")
        target_link_options(${test_name} PRIVATE -Wl,--wrap=parse_line)
    endif()
    if (${test_name} STREQUAL "test_tls_crypt")
        target_sources(${test_name} PRIVATE
                src/openvpn/argv.c
                src/openvpn/env_set.c
                src/openvpn/run_command.c)
    endif()
    if (${CMAKE_SYSTEM_NAME} STREQUAL "Linux")

        target_include_directories(${test_name} PRIVATE ${OVPN_DCO_SOURCE}/include/uapi ${LIBNL_INCLUDE_DIRS})
        target_link_libraries(${test_name} ${LIBNL_LIBRARIES})
    endif()
ENDFOREACH ()
endif()