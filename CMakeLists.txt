PROJECT(librdns C)

SET(RSPAMD_VERSION_MAJOR 0)
SET(RSPAMD_VERSION_MINOR 0)
SET(RSPAMD_VERSION_PATCH 0)

CMAKE_MINIMUM_REQUIRED(VERSION 2.6.0 FATAL_ERROR)

OPTION(ENABLE_CURVE "Enable DNSCurve plugin" OFF)

INCLUDE(CheckFunctionExists)
INCLUDE(CheckSymbolExists)

IF(NOT SLAVE_BUILD)
	CHECK_SYMBOL_EXISTS(IPV6_V6ONLY "sys/socket.h;netinet/in.h" HAVE_IPV6_V6ONLY)

	CHECK_SYMBOL_EXISTS(getaddrinfo "sys/types.h;sys/socket.h;netdb.h" HAVE_GETADDRINFO)
	IF(NOT HAVE_GETADDRINFO)
		MESSAGE(FATAL_ERROR "Your system does not support getaddrinfo call, please consider upgrading it to run librdns")
	ENDIF(NOT HAVE_GETADDRINFO)

	SET(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -g")
ENDIF(NOT SLAVE_BUILD)


IF(ENABLE_CURVE MATCHES "ON" AND NOT LIBSODIUM)
	FIND_LIBRARY(LIBSODIUM NAMES sodium PATH_SUFFIXES lib64 lib
	                  PATHS
	                      ~/Library/Frameworks
	                      /Library/Frameworks
	                      /usr/local
	                      /usr
	                      /sw
	                      /opt/local
	                      /opt/csw
	                      /opt
	                 DOC "Path where the libsodium library can be found")
	IF(NOT LIBSODIUM)
		MESSAGE(FATAL_ERROR "libsodium is required for DNSCurve support")
	ENDIF(NOT LIBSODIUM)

	FIND_PATH(LIBSODIUM_INCLUDE sodium.h PATHS 	/opt/include
											/usr/include
											/usr/local/include
											DOC "Path where the libsodium header files can be found")
	IF(NOT LIBSODIUM_INCLUDE)
		MESSAGE(FATAL_ERROR "libsodium header is required for DNSCurve support")
	ENDIF(NOT LIBSODIUM_INCLUDE)
	GET_FILENAME_COMPONENT(LIBSODIUM_PATH "${LIBSODIUM}" PATH)
	INCLUDE_DIRECTORIES("${LIBSODIUM_INCLUDE}")
	LINK_DIRECTORIES("${LIBSODIUM_PATH}" /usr/local/lib/)
	SET(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -DHAVE_SODIUM")
ENDIF(ENABLE_CURVE MATCHES "ON" AND NOT LIBSODIUM)


SET(LIBRDNSSRC			src/util.c
						src/logger.c
						src/compression.c
						src/punycode.c
						src/curve.c
						src/parse.c
						src/packet.c
						src/resolver.c)

INCLUDE_DIRECTORIES("${CMAKE_CURRENT_SOURCE_DIR}/src"
					"${CMAKE_CURRENT_SOURCE_DIR}/include"
					"${CMAKE_BINARY_DIR}/src"
					"${CMAKE_BINARY_DIR}/include"
					"${CMAKE_CURRENT_SOURCE_DIR}/contrib/uthash"
					"${CMAKE_CURRENT_SOURCE_DIR}/contrib/libottery")

SET(CMAKE_INSTALL_RPATH "/usr/local/lib/")

IF(NOT SLAVE_BUILD)
	ADD_SUBDIRECTORY(contrib/libottery)
ENDIF(NOT SLAVE_BUILD)
ADD_SUBDIRECTORY(test)

ADD_LIBRARY(rdns SHARED ${LIBRDNSSRC})
TARGET_LINK_LIBRARIES(rdns ottery)

IF(ENABLE_CURVE MATCHES "ON")
	TARGET_LINK_LIBRARIES(rdns sodium)
ENDIF(ENABLE_CURVE MATCHES "ON")

install(TARGETS rdns
     LIBRARY DESTINATION lib COMPONENT libraries
     ARCHIVE DESTINATION lib/static COMPONENT libraries)

set(RDNS_HEADERS include/rdns_curve.h include/rdns_event.h include/rdns_ev.h include/rdns.h)
install(FILES ${RDNS_HEADERS} DESTINATION include/rdns)