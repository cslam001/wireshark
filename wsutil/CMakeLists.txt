# CMakeLists.txt
#
# Wireshark - Network traffic analyzer
# By Gerald Combs <gerald@wireshark.org>
# Copyright 1998 Gerald Combs
#
# This program is free software; you can redistribute it and/or
# modify it under the terms of the GNU General Public License
# as published by the Free Software Foundation; either version 2
# of the License, or (at your option) any later version.
#
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.	See the
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with this program; if not, write to the Free Software
# Foundation, Inc., 51 Franklin Street, Fifth Floor, Boston, MA 02110-1301 USA.
#

include(UseABICheck)

if(WIN32)
	set(WSUTIL_PLATFORM_FILES
		file_util.c
		inet_ntop.c
		inet_pton.c
	)
ENDIF(WIN32)

set(WSUTIL_FILES
	adler32.c
	aes.c
	airpdcap_wep.c
	base64.c
	bitswap.c
	buffer.c
	cfutils.c
	clopts_common.c
	cmdarg_err.c
	compiler_info.c
	copyright_info.c
	cpu_info.c
	crash_info.c
	crc10.c
	crc16.c
	crc16-plain.c
	crc32.c
	crc6.c
	crc7.c
	crc8.c
	crc11.c
	des.c
	eax.c
	filesystem.c
	g711.c
	glib_version_info.c
	md4.c
	md5.c
	mem_info.c
	mpeg-audio.c
	nstime.c
	os_version_info.c
	plugins.c
	privileges.c
	sha1.c
	sober128.c
	strnatcmp.c
	str_util.c
	rc4.c
	report_err.c
	tempfile.c
	time_util.c
	type_util.c
	u3.c
	unicode-utils.c
	ws_mempbrk.c
	ws_mempbrk_sse42.c
	ws_version_info.c
	${WSUTIL_PLATFORM_FILES}
)

if(HAVE_SSE4_2)
	set(WSUTIL_FILES ${WSUTIL_FILES} ws_mempbrk_sse42.c)
endif()

if(NOT HAVE_GETOPT_LONG)
	set(WSUTIL_FILES ${WSUTIL_FILES} wsgetopt.c)
endif()

if(NOT HAVE_INET_ATON)
	set(WSUTIL_FILES ${WSUTIL_FILES} inet_aton.c)
endif()

if(NOT HAVE_POPCOUNT)
	set(WSUTIL_FILES ${WSUTIL_FILES} popcount.c)
endif()

if(NOT HAVE_STRPTIME)
	set(WSUTIL_FILES ${WSUTIL_FILES} strptime.c)
endif()

if(APPLE)
	#
	# We assume that APPLE means OS X so that we have the OS X
	# frameworks.
	#
	FIND_LIBRARY (APPLE_CORE_FOUNDATION_LIBRARY CoreFoundation)
endif()

set(wsutil_LIBS
	${APPLE_CORE_FOUNDATION_LIBRARY}
	${GMODULE2_LIBRARIES}
	${GLIB2_LIBRARIES}
	${GCRYPT_LIBRARIES}
	${WIN_WSOCK32_LIBRARY}
)
IF(WIN32)
	set(wsutil_LIBS ${wsutil_LIBS} "ws2_32.lib")
ENDIF(WIN32)

set(CLEAN_FILES
	${WSUTIL_FILES}
)

if (WERROR)
	set_source_files_properties(
		${CLEAN_FILES}
		PROPERTIES
		COMPILE_FLAGS -Werror
	)
endif()
if (HAVE_SSE4_2)
	#
	# XXX - we're assuming MSVC supports the SSE 4.2 intrinsics and
	# that other C compilers support them iff they support the
	# -msse4.2 flag.
	#
	# Perhaps we should check whether we can compile something
	# that uses them, instead, and do something else to figure
	# out what compiler flag, if any, we need to pass to the
	# compiler to compile code that uses them.
	#
	if(NOT CMAKE_C_COMPILER_ID MATCHES "MSVC")
		set_source_files_properties(
			${WSUTIL_SSE42_FILES}
			PROPERTIES
			COMPILE_FLAGS -msse4.2
		)
	endif()
endif()

add_library(wsutil ${LINK_MODE_LIB}
	${WSUTIL_FILES}
	../image/libwsutil.rc
)

add_dependencies(wsutil gitversion)

set(FULL_SO_VERSION "0.0.0")

set_target_properties(wsutil PROPERTIES
	PREFIX "lib"
	COMPILE_DEFINITIONS "WS_BUILD_DLL"
	LINK_FLAGS "${WS_LINK_FLAGS}"
	VERSION ${FULL_SO_VERSION} SOVERSION 0
	FOLDER "DLLs")

if(ENABLE_APPLICATION_BUNDLE)
	set_target_properties(wsutil PROPERTIES
		LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/run/Wireshark.app/Contents/Frameworks
	)
endif()

ABICHECK(libwsutil)

add_custom_command(OUTPUT libwsutil.abi.tar.gz
	COMMAND cp ../config.h ${ABICHECK_TMPDIR}/
	COMMAND ${ABICHECK_COMMAND}
	COMMAND cp ${CMAKE_CURRENT_BINARY_DIR}/abi_dumps/libwsutil/libwsutil_* ${CMAKE_CURRENT_BINARY_DIR}/libwsutil.abi.tar.gz
	COMMAND rm -rf ${ABICHECK_TMPDIR}/* ${CMAKE_CURRENT_BINARY_DIR}/abi_dumps
	DEPENDS ${HEADERS} wsutil)

target_link_libraries(wsutil ${wsutil_LIBS})

if(NOT ${ENABLE_STATIC})
	install(TARGETS wsutil
		LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
		RUNTIME DESTINATION ${CMAKE_INSTALL_LIBDIR}
		ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
	)
endif()

add_definitions( -DTOP_SRCDIR=\"${CMAKE_SOURCE_DIR}\" )

#
# Editor modelines  -  http://www.wireshark.org/tools/modelines.html
#
# Local variables:
# c-basic-offset: 8
# tab-width: 8
# indent-tabs-mode: t
# End:
#
# vi: set shiftwidth=8 tabstop=8 noexpandtab:
# :indentSize=8:tabSize=8:noTabs=false:
#
