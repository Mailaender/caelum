######################################################################
# Caelum BUILD SYSTEM
# Welcome to the CMake build system for Caelum
# This is the main file where we prepare the general build environment
# and provide build configuration options.
######################################################################
# cmake system for Caelum updated on the 22th of March 2011 by thomas{at}thomasfischer{DOT}biz

cmake_minimum_required(VERSION 2.4)
# loose if - else constructs
SET(CMAKE_ALLOW_LOOSE_LOOP_CONSTRUCTS TRUE)
if(COMMAND cmake_policy)
  cmake_policy(SET CMP0003 NEW)
endif(COMMAND cmake_policy)


# add some functions we use that are shipped with cmake
INCLUDE(CheckLibraryExists)
INCLUDE(CheckIncludeFile)
INCLUDE(CheckIncludeFileCXX)
INCLUDE(CheckCCompilerFlag)
INCLUDE(CheckCSourceCompiles)


# define the project
project(Caelum)

# find all dependencies
include(CMakeDependenciesConfig.txt)

# build static libs by default
SET(BUILD_SHARED_LIBS OFF)

# setup paths
SET(RUNTIME_OUTPUT_DIRECTORY "${Caelum_SOURCE_DIR}/bin/")
SET(LIBRARY_OUTPUT_DIRECTORY "${Caelum_SOURCE_DIR}/lib/")
SET(ARCHIVE_OUTPUT_DIRECTORY "${Caelum_SOURCE_DIR}/lib/")
SET(EXECUTABLE_OUTPUT_PATH   ${RUNTIME_OUTPUT_DIRECTORY})
SET(LIBRARY_OUTPUT_PATH      ${LIBRARY_OUTPUT_DIRECTORY})

# some optimization flags
if(WIN32)
  # add multi processor compilation flags
  set(CMAKE_CXX_FLAGS_RELEASE        "${CMAKE_CXX_FLAGS_RELEASE}        /MP /GL /Ox /Ob2 /Oi /Ot /Oy /fp:fast /GS- /MP /Zi")
  set(CMAKE_CXX_FLAGS_MINSIZEREL     "${CMAKE_CXX_FLAGS_MINSIZEREL}     /MP /Zi")
  set(CMAKE_CXX_FLAGS_RELWITHDEBINFO "${CMAKE_CXX_FLAGS_RELWITHDEBINFO} /MP /Od /Zi")
  set(CMAKE_CXX_FLAGS_DEBUG          "${CMAKE_CXX_FLAGS_DEBUG}          /MP /Od /Zi")
  # some general flags
  add_definitions("-D_CRT_SECURE_NO_WARNINGS")
  include_directories(${DirectX_INCLUDE_DIR})

  set(CMAKE_EXE_LINKER_FLAGS_DEBUG          "${CMAKE_EXE_LINKER_FLAGS_DEBUG}          /DEBUG /SUBSYSTEM:WINDOWS")
  set(CMAKE_EXE_LINKER_FLAGS_RELEASE        "${CMAKE_EXE_LINKER_FLAGS_RELEASE}        /DEBUG /SUBSYSTEM:WINDOWS /LTCG /OPT:REF")
  set(CMAKE_EXE_LINKER_FLAGS_MINSIZEREL     "${CMAKE_EXE_LINKER_FLAGS_MINSIZEREL}     /DEBUG /SUBSYSTEM:WINDOWS /LTCG /OPT:REF")
  set(CMAKE_EXE_LINKER_FLAGS_RELWITHDEBINFO "${CMAKE_EXE_LINKER_FLAGS_RELWITHDEBINFO} /DEBUG /SUBSYSTEM:WINDOWS /LTCG /OPT:REF")
ELSEIF(UNIX)
  set(CMAKE_EXE_LINKER_FLAGS_RELEASE      "${CMAKE_EXE_LINKER_FLAGS_RELEASE}      -mfpmath=sse -msse2 -mmmx -msse -msse3 -m3dnow -O2 -fomit-frame-pointer -fstrict-aliasing -ffast-math -ftracer")
  set(CMAKE_EXE_LINKER_FLAGS_RelWithDebug "${CMAKE_EXE_LINKER_FLAGS_RelWithDebug} -O0")
endif(WIN32)

# some PG build options
set(Caelum_BUILD_SAMPLES         "TRUE"  CACHE BOOL "build the examples")
#set(Caelum_ALTERNATE_COORDSYSTEM "FALSE" CACHE BOOL "alternate coordinate system, do not use unless you are very sure about it")
#set(Caelum_USE_OGRE_RANDOM       "FALSE" CACHE BOOL "fallback to Ogre's PRNG instead of using our own (not recommended)")
#set(Caelum_USER_DATA             "FALSE" CACHE BOOL "ability to attach user data to entities")

# some versioning things
SET(LIB_MAJOR_VERSION "0")
SET(LIB_MINOR_VERSION "6")
SET(LIB_BUILD_VERSION "1")
SET(LIB_VERSION "${LIB_MAJOR_VERSION}.${LIB_MINOR_VERSION}.${LIB_BUILD_VERSION}")
IF(NOT DEFINED LIB_INSTALL_DIR)
  SET(LIB_INSTALL_DIR "lib${LIB_SUFFIX}")
ENDIF(NOT DEFINED LIB_INSTALL_DIR)

# Needed for Caelum.pc.in
SET(prefix ${CMAKE_INSTALL_PREFIX})
SET(exec_prefix "\${prefix}")
SET(libdir "\${exec_prefix}/${LIB_INSTALL_DIR}")
SET(bindir "\${exec_prefix}/bin")
SET(includedir "\${prefix}/include")
SET(PACKAGE_NAME "Caelum")
SET(PACKAGE_VERSION "${LIB_VERSION}")

# configuration of the config.h and PkgConfig
CONFIGURE_FILE(
    "${Caelum_SOURCE_DIR}/main/include/CaelumConfig.h.in"
    "${Caelum_BINARY_DIR}/main/include/CaelumConfig.h")
CONFIGURE_FILE(
    "${Caelum_SOURCE_DIR}/Caelum.pc.in"
    "${Caelum_BINARY_DIR}/Caelum.pc"
    @ONLY)
# install the PkgConfig file
INSTALL(FILES "${Caelum_BINARY_DIR}/Caelum.pc" DESTINATION "${LIB_INSTALL_DIR}/pkgconfig")

# some additional compiler flags
IF(NOT WIN32)
	ADD_DEFINITIONS(-Wall)
	CHECK_C_COMPILER_FLAG(-Wextra HAVE_W_EXTRA)
	IF(HAVE_W_EXTRA)
		ADD_DEFINITIONS(-Wextra)
	ENDIF()
endif()

# Set visibility options if available
#IF(NOT WIN32)
#	CHECK_C_SOURCE_COMPILES("int foo() __attribute__((destructor));
#								int main() {return 0;}" HAVE_GCC_DESTRUCTOR)
#
#	CHECK_C_COMPILER_FLAG(-fvisibility=hidden HAVE_VISIBILITY_SWITCH)
#	IF(HAVE_VISIBILITY_SWITCH)
#		CHECK_C_SOURCE_COMPILES("int foo() __attribute__((visibility(\"default\")));
#									int main() {return 0;}" HAVE_GCC_VISIBILITY)
#		IF(HAVE_GCC_VISIBILITY)
#			ADD_DEFINITIONS(-fvisibility=hidden -DHAVE_GCC_VISIBILITY)
#		ENDIF()
#	ENDIF()
#ENDIF()


# now add the directories
add_subdirectory(main)

#if(Caelum_BUILD_SAMPLES)
#	add_subdirectory(samples)
#endif(Caelum_BUILD_SAMPLES)



# cpack
set(CPACK_PACKAGE_DESCRIPTION "Caelum")
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "Caelum is an add-on for the 3D graphics rendering engine OGRE, aimed to render atmospheric effects.")
set(CPACK_PACKAGE_NAME "caelum")
set(CPACK_DEBIAN_PACKAGE_DEPENDS "ois")
set(CPACK_PACKAGE_CONTACT "support@rigsofrods.com")
set(CPACK_PACKAGE_VENDOR "support@rigsofrods.com")
set(CPACK_RESOURCE_FILE_LICENSE "${CMAKE_CURRENT_SOURCE_DIR}/lgpl.txt")

SET(CPACK_GENERATOR ZIP)
IF(UNIX)
  SET(CPACK_GENERATOR ${CPACK_GENERATOR};STGZ;TGZ)
ENDIF(UNIX)
IF(LINUX)
SET(CPACK_GENERATOR ${CPACK_GENERATOR};DEB;RPM)
ENDIF(LINUX)
IF(MSVC)
  SET(CPACK_GENERATOR ${CPACK_GENERATOR};NSIS)
ENDIF(MSVC)
IF(APPLE)
  SET(CPACK_GENERATOR ${CPACK_GENERATOR};PackageMaker)
ENDIF(APPLE)
set(CPACK_PACKAGE_FILE_NAME "${CPACK_PACKAGE_NAME}-${LIB_VERSION}-${CMAKE_SYSTEM_PROCESSOR}")
include(CPack)

# doxygen stuff
find_package(Doxygen)
if (DOXYGEN_FOUND)
	message("found doxygen, generating documentation")
	# prepare doxygen configuration file
	configure_file(${CMAKE_CURRENT_SOURCE_DIR}/Doxyfile.in ${CMAKE_CURRENT_BINARY_DIR}/Doxyfile)
	# add doxygen as target
	add_custom_target(doxygen ${DOXYGEN_EXECUTABLE} ${CMAKE_CURRENT_BINARY_DIR}/Doxyfile)

	# cleanup $build/api-doc on "make clean"
	set_property(DIRECTORY APPEND PROPERTY ADDITIONAL_MAKE_CLEAN_FILES api-doc)

	# add doxygen as dependency to doc-target
	get_target_property(DOC_TARGET doc TYPE)
	if(NOT DOC_TARGET)
		add_custom_target(doc)
	endif()
	add_dependencies(doc doxygen)
	#	install(DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/doc/html/ DESTINATION doc/api)
	# install man pages into packages, scope is now project root..
	#install(DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/api-doc/man/man3 DESTINATION share/man/man3/ )
endif (DOXYGEN_FOUND)

# other doc files
set(DOC_FILES Contributors.txt gpl.txt lgpl.txt ReadMe.txt)
install(FILES ${DOC_FILES} DESTINATION doc/)
