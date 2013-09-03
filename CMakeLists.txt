cmake_minimum_required (VERSION 2.6)
project (libgadu C)

set (GG_LIBGADU_VERSION "1.12.0-pre")
option (libgadu_build_examples "Build examples" ON)
option (libgadu_build_tests "Build automatic tests" ON)

find_program (protoc_c_EXECUTABLE protoc-c)
if (NOT protoc_c_EXECUTABLE)
	message (FATAL_ERROR "Unable to find protoc-c compiler.")
endif ()
find_library (protoc_LIBRARY protobuf-c)
if (NOT protoc_LIBRARY)
	message (FATAL_ERROR "Unable to find protobuf-c library.")
endif ()

# Platform checks

# Check if libgadu was compiled for bigendian machine. */
include (TestBigEndian)
test_big_endian (GG_CONFIG_BIGENDIAN)

# Check if this machine has gethostbyname_r(). */
include (CheckFunctionExists)
check_function_exists (gethostbyname_r GG_CONFIG_HAVE_GETHOSTBYNAME_R)

# Check if libgadu was compiled and linked with fork support. */
check_function_exists (fork GG_CONFIG_HAVE_FORK)

# Check if libgadu was compiled and linked with pthread support. */
find_package (Threads)
if (CMAKE_USE_PTHREADS_INIT)
	set (GG_CONFIG_HAVE_PTHREAD)
endif ()

# Check if pthread resolver is the default one. */
if (CMAKE_USE_PTHREADS_INIT)
	set (GG_CONFIG_PTHREAD_DEFAULT)
endif ()

# Check if this machine has C99-compiliant vsnprintf(). */
INCLUDE (CheckCSourceCompiles)
check_c_source_compiles ("#include <stdio.h>
	int main()
	{
		char tmp;
		return (snprintf(&tmp, sizeof(tmp), \"test\") != 4);
	}" GG_CONFIG_HAVE_C99_VSNPRINTF)

# Check if this machine has va_copy(). */
check_c_source_compiles ("#include <stdarg.h> 
	int main()
	{
		va_list a, b;
		va_copy(a, b);
	}" GG_CONFIG_HAVE_VA_COPY)

# Check if this machine has __va_copy(). */
check_c_source_compiles ("#include <stdarg.h>
	int main()
	{
		va_list a, b;
		__va_copy(a, b);
	}" GG_CONFIG_HAVE___VA_COPY)

# Check if this machine supports long long. */
check_c_source_compiles ("
	int main()
	{
		long long a = 1LL;
		unsigned long long b = 1LL;
	}" GG_CONFIG_HAVE_LONG_LONG)

# Check if libgadu was compiled and linked with GnuTLS support. */
# GG_CONFIG_HAVE_GNUTLS

# Check if libgadu was compiled and linked with OpenSSL support. */
# GG_CONFIG_HAVE_OPENSSL

# Check if libgadu was compiled and linked with zlib support. */
find_package (ZLIB)
if (ZLIB_FOUND)
	set (GG_CONFIG_HAVE_ZLIB TRUE)
	list (APPEND libgadu_INCLUDE_DIRS ${ZLIB_INCLUDE_DIRS})
	list (APPEND libgadu_LIBRARIES ${ZLIB_LIBRARIES})
endif ()

# Check if uintX_t types are defined in <stdint.h>. */
include (CheckIncludeFiles)
check_include_files (stdint.h GG_CONFIG_HAVE_STDINT_H)

# Check if uintX_t types are defined in <inttypes.h>. */
check_include_files (inttypes.h GG_CONFIG_HAVE_INTTYPES_H)

# Check if uintX_t types are defined in <sys/inttypes.h>. */
check_include_files (sys/inttypes.h GG_CONFIG_HAVE_SYS_INTTYPES_H)

# Check if uintX_t types are defined in <sys/int_types.h>. */
check_include_files (sys/int_types.h GG_CONFIG_HAVE_SYS_INT_TYPES_H)

# Check if uintX_t types are defined in <sys/types.h>. */
check_include_files (sys/types.h GG_CONFIG_HAVE_SYS_TYPES_H)

# Check if this machine has uint64_t. */
INCLUDE (CheckSymbolExists)
check_symbol_exists (uint64_t "stdint.h" GG_CONFIG_HAVE_UINT64_T)

# Generate protobuf source file
set (packets_src ${CMAKE_CURRENT_BINARY_DIR}/packets.pb-c.c)
add_custom_command (OUTPUT ${packets_src}
	COMMAND ${protoc_c_EXECUTABLE}
		--c_out=${CMAKE_CURRENT_BINARY_DIR}
		-I ${CMAKE_CURRENT_SOURCE_DIR}
		${CMAKE_CURRENT_SOURCE_DIR}/packets.proto)

# Generate empty config.h
add_custom_command (OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/config.h
	COMMAND ${CMAKE_COMMAND} -E touch ${CMAKE_CURRENT_BINARY_DIR}/config.h)

# All sources
set (libgadu_la_SOURCES
	src/common.c
	src/dcc.c
	src/dcc7.c
	src/debug.c
	src/deflate.c
	src/encoding.c
	src/endian.c
	src/events.c
	src/handlers.c
	src/http.c
	src/libgadu.c
	src/message.c
	src/network.c
	src/obsolete.c
	src/protobuf.c
	src/pubdir.c
	src/pubdir50.c
	src/resolver.c
	src/sha1.c
	src/tvbuff.c
	src/tvbuilder.c
	${packets_src}
	${CMAKE_CURRENT_BINARY_DIR}/config.h)

# Configure libgadu.h.in
configure_file (
	include/libgadu.h.in
	${CMAKE_CURRENT_BINARY_DIR}/libgadu.h)

include_directories (
	include/
	${CMAKE_CURRENT_BINARY_DIR}/
	${libgadu_INCLUDE_DIRS})

add_definitions (-DGG_IGNORE_DEPRECATED)
add_library (gadu SHARED
	${libgadu_la_SOURCES})
target_link_libraries (gadu
	${protoc_LIBRARY}
	${libgadu_LIBRARIES})
set_target_properties(gadu PROPERTIES
	VERSION 3.13
	SOVERSION 3.13)
if (libgadu_build_examples)
	add_subdirectory (examples)
endif ()
if (libgadu_build_tests)
	enable_testing ()
	add_subdirectory (test)
endif ()