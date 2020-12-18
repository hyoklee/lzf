cmake_minimum_required (VERSION 3.1.0)
PROJECT (LZF C)

#-----------------------------------------------------------------------------
# Basic LZF stuff here
#-----------------------------------------------------------------------------

#-----------------------------------------------------------------------------
# Define some CMake variables for use later in the project
#-----------------------------------------------------------------------------
set (LZF_RESOURCES_DIR          ${LZF_SOURCE_DIR}/config/cmake)
set (LZF_SRC_DIR                ${LZF_SOURCE_DIR})
set (LZF_EXAMPLES_SOURCE_DIR    ${LZF_SOURCE_DIR}/examples)

#-----------------------------------------------------------------------------
# Set the core names of all the libraries
#-----------------------------------------------------------------------------
set (LZF_LIB_CORENAME           "lzf")

#-----------------------------------------------------------------------------
# Set the true names of all the libraries if customized by external project
#-----------------------------------------------------------------------------
set (LZF_LIB_NAME              "${LZF_EXTERNAL_LIB_PREFIX}${LZF_LIB_CORENAME}")

#-----------------------------------------------------------------------------
# Set the target names of all the libraries
#-----------------------------------------------------------------------------
set (LZF_LIB_TARGET             "${LZF_LIB_CORENAME}-static")
set (LZF_LIBSH_TARGET           "${LZF_LIB_CORENAME}-shared")

if (NOT LZF_INSTALL_BIN_DIR)
  set (LZF_INSTALL_BIN_DIR bin)
endif ()
if (NOT LZF_INSTALL_LIB_DIR)
  set (LZF_INSTALL_LIB_DIR lib)
endif ()
if (NOT LZF_INSTALL_INCLUDE_DIR)
  set (LZF_INSTALL_INCLUDE_DIR include)
endif ()
if (NOT LZF_INSTALL_DATA_DIR)
  if (NOT WIN32)
    if (APPLE)
      option (LZF_BUILD_FRAMEWORKS "Create a Mac OSX Framework" OFF)
      set (LZF_INSTALL_FWRK_DIR ${CMAKE_INSTALL_FRAMEWORK_PREFIX})
    endif ()
    set (LZF_INSTALL_DATA_DIR share)
    set (LZF_INSTALL_CMAKE_DIR share/cmake)
  else ()
    set (LZF_INSTALL_DATA_DIR ".")
    set (LZF_INSTALL_CMAKE_DIR cmake)
  endif ()
endif ()

#-----------------------------------------------------------------------------
# parse the full version number from lzf.h and include in LZF_VERS_INFO
#-----------------------------------------------------------------------------
#file (READ ${LZF_SRC_DIR}/lzf.h _lzf_h_contents)
set (LZF_VERS_MAJOR "3")
set (LZF_VERS_MINOR "6")
set (LZF_VERS_RELEASE "0")
set (LZF_VERS_SUBRELEASE "")
#message (STATUS "VERSION: ${LZF_VERS_MAJOR}.${LZF_VERS_MINOR}.${LZF_VERS_RELEASE}-${LZF_VERS_SUBRELEASE}")
#string (REGEX REPLACE ".*#define[ \t]+LZF_VERSION[ \t]+\"([0-9A-Za-z.]+)\".*"
#    "\\1" LZF_FULL_VERSION ${_lzf_h_contents})
#message (STATUS "VERSION: ${LZF_FULL_VERSION}")

#-----------------------------------------------------------------------------
set (LZF_PACKAGE "lzf")
set (LZF_PACKAGE_NAME "LZF")
set (LZF_PACKAGE_VERSION "${LZF_VERS_MAJOR}.${LZF_VERS_MINOR}")
set (LZF_PACKAGE_VERSION_MAJOR "${LZF_VERS_MAJOR}.${LZF_VERS_MINOR}")
set (LZF_PACKAGE_VERSION_MINOR "${LZF_VERS_RELEASE}")
set (LZF_PACKAGE_STRING "${LZF_PACKAGE_NAME} ${LZF_PACKAGE_VERSION}-${LZF_VERS_SUBRELEASE}")
set (LZF_PACKAGE_TARNAME "lzf")
set (LZF_PACKAGE_URL "http://www.hdfgroup.org")
set (LZF_PACKAGE_BUGREPORT "help@hdfgroup.org")

#-----------------------------------------------------------------------------
# Include some macros for reusable code
#-----------------------------------------------------------------------------
include (${LZF_RESOURCES_DIR}/LZFMacros.cmake)

#-----------------------------------------------------------------------------
# Setup output Directories
#-----------------------------------------------------------------------------
if (NOT LZF_EXTERNALLY_CONFIGURED)
  set (CMAKE_RUNTIME_OUTPUT_DIRECTORY
      ${PROJECT_BINARY_DIR}/bin CACHE PATH "Single Directory for all Executables."
  )
  set (CMAKE_LIBRARY_OUTPUT_DIRECTORY
      ${PROJECT_BINARY_DIR}/bin CACHE PATH "Single Directory for all Libraries"
  )
  set (CMAKE_ARCHIVE_OUTPUT_DIRECTORY
      ${PROJECT_BINARY_DIR}/bin CACHE PATH "Single Directory for all static libraries."
  )
else ()
  # if we are externally configured, but the project uses old cmake scripts
  # this may not be set
  if (NOT CMAKE_RUNTIME_OUTPUT_DIRECTORY)
    set (CMAKE_RUNTIME_OUTPUT_DIRECTORY ${EXECUTABLE_OUTPUT_PATH})
  endif ()
endif ()

#-----------------------------------------------------------------------------
# Targets built within this project are exported at Install time for use
# by other projects using FindLZF.
#-----------------------------------------------------------------------------
if (NOT LZF_EXPORTED_TARGETS)
  set (LZF_EXPORTED_TARGETS "lzf-targets")
endif ()

#-----------------------------------------------------------------------------
# To include a library in the list exported by the project AT BUILD TIME,
# add it to this variable. This is NOT used by Make Install, but for projects
# which include LZF as a sub-project within their build tree
#-----------------------------------------------------------------------------
set_global_variable (LZF_LIBRARIES_TO_EXPORT "")

#-----------------------------------------------------------------------------
# Run all the CMake configuration tests for our build environment
#-----------------------------------------------------------------------------
include (${LZF_RESOURCES_DIR}/ConfigureChecks.cmake)

#-----------------------------------------------------------------------------
# Option to Build Shared/Static libs, default is static
#-----------------------------------------------------------------------------
option (BUILD_SHARED_LIBS "Build Shared Libraries" OFF)
set (CMAKE_POSITION_INDEPENDENT_CODE ON)

#-----------------------------------------------------------------------------
# When building utility executables that generate other (source) files :
# we make use of the following variables defined in the root CMakeLists.
# Certain systems may add /Debug or /Release to output paths
# and we need to call the executable from inside the CMake configuration
#-----------------------------------------------------------------------------
set (EXE_EXT "")
if (WIN32)
  set (EXE_EXT ".exe")
  add_definitions (-D_BIND_TO_CURRENT_VCLIBS_VERSION=1)
  add_definitions (-D_CRT_SECURE_NO_WARNINGS)
  add_definitions (-D_CRT_NONSTDC_NO_DEPRECATE)
endif ()

if (MSVC)
  set (CMAKE_MFC_FLAG 0)
endif ()

set (MAKE_SYSTEM)
if (CMAKE_BUILD_TOOL MATCHES "make")
  set (MAKE_SYSTEM 1)
endif ()

set (CFG_INIT "/${CMAKE_CFG_INTDIR}")
if (MAKE_SYSTEM)
  set (CFG_INIT "")
endif ()

#-----------------------------------------------------------------------------
# Compiler specific flags : Shouldn't there be compiler tests for these
#-----------------------------------------------------------------------------
if (CMAKE_COMPILER_IS_GNUCC)
  set (CMAKE_C_FLAGS "${CMAKE_ANSI_CFLAGS} ${CMAKE_C_FLAGS} -std=c99 -fomit-frame-pointer -finline-functions -fno-common")
endif ()
if (CMAKE_COMPILER_IS_GNUCXX)
  set (CMAKE_CXX_FLAGS "${CMAKE_ANSI_CFLAGS} ${CMAKE_CXX_FLAGS} -fomit-frame-pointer -finline-functions -fno-common")
endif ()

#-----------------------------------------------------------------------------
# This is in here to help some of the GCC based IDES like Eclipse
# and code blocks parse the compiler errors and warnings better.
#-----------------------------------------------------------------------------
if (CMAKE_COMPILER_IS_GNUCC)
  set (CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -fmessage-length=0")
endif ()
if (CMAKE_COMPILER_IS_GNUCXX)
  set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fmessage-length=0")
endif ()

#-----------------------------------------------------------------------------
# Include user macros
#-----------------------------------------------------------------------------
include (UserMacros.cmake)

#-----------------------------------------------------------------------------
# All libs/tests/examples need the main include directories
#-----------------------------------------------------------------------------
INCLUDE_DIRECTORIES (${LZF_BINARY_DIR} ${LZF_SOURCE_DIR} ${CMAKE_RUNTIME_OUTPUT_DIRECTORY})

#============================================================================
# lzf
#============================================================================

INCLUDE_DIRECTORIES (${CMAKE_BINARY_DIR})

#-----------------------------------------------------------------------------
# Define lzf Library
#-----------------------------------------------------------------------------
set (LZF_PUBLIC_HEADERS
    lzf.h
)
set (LZF_HDRS
    crc32.h
    lzfP.h
)
set (LZF_SRCS
    lzf_c.c
    lzf_d.c
)

add_library (${LZF_LIB_TARGET} STATIC ${LZF_SRCS} ${LZF_HDRS} ${LZF_PUBLIC_HEADERS})

if (MSVC AND CMAKE_CL_64)
  set_target_properties (${LZF_LIB_TARGET} PROPERTIES STATIC_LIBRARY_FLAGS "/machine:x64")
endif ()
TARGET_C_PROPERTIES (${LZF_LIB_TARGET} STATIC " " " ")
target_link_libraries (${LZF_LIB_TARGET} ${LINK_LIBS})
LZF_SET_LIB_OPTIONS (${LZF_LIB_TARGET} ${LZF_LIB_NAME} STATIC)
set_target_properties(${LZF_LIB_TARGET} PROPERTIES
    PUBLIC_HEADER "${LZF_PUBLIC_HEADERS}"
    INTERFACE_INCLUDE_DIRECTORIES "$<INSTALL_INTERFACE:$<INSTALL_PREFIX>/include>"
)
set_global_variable (LZF_LIBRARIES_TO_EXPORT ${LZF_LIB_TARGET})
set (install_targets ${LZF_LIB_TARGET})

if (BUILD_SHARED_LIBS)
  add_library (${LZF_LIBSH_TARGET} SHARED ${LZF_SRCS} ${LZF_HDRS} ${LZF_PUBLIC_HEADERS})

  set (SHARED_LINK_FLAGS " ")
  set_target_properties (${LZF_LIBSH_TARGET} PROPERTIES DEFINE_SYMBOL ${LZF_LIB_CORENAME}_EXPORTS)
  TARGET_C_PROPERTIES (${LZF_LIBSH_TARGET} SHARED " " "${SHARED_LINK_FLAGS}")
  target_link_libraries (${LZF_LIBSH_TARGET} ${LINK_LIBS})
  LZF_SET_LIB_OPTIONS (${LZF_LIBSH_TARGET} ${LZF_LIB_NAME} SHARED)
  set_target_properties(${LZF_LIBSH_TARGET} PROPERTIES
      PUBLIC_HEADER "${LZF_PUBLIC_HEADERS}"
      INTERFACE_INCLUDE_DIRECTORIES "$<INSTALL_INTERFACE:$<INSTALL_PREFIX>/include>"
  )
  set_global_variable (LZF_LIBRARIES_TO_EXPORT "${LZF_LIBRARIES_TO_EXPORT};${LZF_LIBSH_TARGET}")
  set (install_targets ${install_targets} ${LZF_LIBSH_TARGET})
endif ()

#-----------------------------------------------------------------------------
# Dashboard and Testing Settings
#-----------------------------------------------------------------------------
option (BUILD_TESTING "Build LZF Unit Testing" OFF)
if (BUILD_TESTING)
  set (DART_TESTING_TIMEOUT 1200 CACHE INTEGER
       "Timeout in seconds for each test (default 1200=20minutes)")
  enable_testing ()
  include (CTest)

  #-- Adding test for example
  add_executable (lzf lzf.c)
  TARGET_NAMING (lzf STATIC)
  TARGET_C_PROPERTIES (lzf STATIC " " " ")
  target_link_libraries (lzf ${LZF_LIB_TARGET})

  add_test (NAME lzf COMMAND $<TARGET_FILE:lzf>)
endif ()

#-----------------------------------------------------------------------------
# Add Target(s) to CMake Install for import into other projects
#-----------------------------------------------------------------------------

if (LZF_EXPORTED_TARGETS)
  install (
      TARGETS
          ${install_targets}
      EXPORT
          ${LZF_EXPORTED_TARGETS}
      LIBRARY DESTINATION ${LZF_INSTALL_LIB_DIR} COMPONENT libraries
      ARCHIVE DESTINATION ${LZF_INSTALL_LIB_DIR} COMPONENT libraries
      RUNTIME DESTINATION ${LZF_INSTALL_BIN_DIR} COMPONENT libraries
      FRAMEWORK DESTINATION ${LZF_INSTALL_FWRK_DIR} COMPONENT libraries
      PUBLIC_HEADER DESTINATION ${LZF_INSTALL_INCLUDE_DIR} COMPONENT headers
  )
endif ()

#-----------------------------------------------------------------------------
# Generate the config.h file containing user settings needed by compilation
#-----------------------------------------------------------------------------
configure_file (${LZF_RESOURCES_DIR}/config.h.in ${LZF_BINARY_DIR}/config.h @ONLY)

include (CMakePackageConfigHelpers)

#-----------------------------------------------------------------------------
# Add file(s) to CMake Install
#-----------------------------------------------------------------------------
if (NOT LZF_INSTALL_NO_DEVELOPMENT)
  install (
      FILES ${PROJECT_BINARY_DIR}/config.h
      DESTINATION ${LZF_INSTALL_INCLUDE_DIR}
      COMPONENT headers
  )
endif ()

#-----------------------------------------------------------------------------
# Add Target(s) to CMake Install for import into other projects
#-----------------------------------------------------------------------------
if (NOT LZF_EXTERNALLY_CONFIGURED)
  install (
      EXPORT ${LZF_EXPORTED_TARGETS}
      DESTINATION ${LZF_INSTALL_CMAKE_DIR}
      FILE ${LZF_PACKAGE}${LZF_PACKAGE_EXT}-targets.cmake
      COMPONENT configinstall
  )
endif ()

#-----------------------------------------------------------------------------
# Export all exported targets to the build tree for use by parent project
#-----------------------------------------------------------------------------
if (NOT LZF_EXTERNALLY_CONFIGURED)
  export (
      TARGETS ${LZF_LIBRARIES_TO_EXPORT} ${LZF_LIB_DEPENDENCIES}
      FILE ${LZF_PACKAGE}${LZF_PACKAGE_EXT}-targets.cmake
  )
  export (PACKAGE ${LZF_PACKAGE}${LZF_PACKAGE_EXT})
endif ()

#-----------------------------------------------------------------------------
# Set includes needed for build
#-----------------------------------------------------------------------------
set (LZF_INCLUDES_BUILD_TIME
    ${LZF_SRC_DIR} ${LZF_BINARY_DIR}
)

#-----------------------------------------------------------------------------
# Set variables needed for installation
#-----------------------------------------------------------------------------
set (LZF_VERSION_STRING ${LZF_PACKAGE_VERSION})
set (LZF_VERSION_MAJOR  ${LZF_PACKAGE_VERSION_MAJOR})
set (LZF_VERSION_MINOR  ${LZF_PACKAGE_VERSION_MINOR})

#-----------------------------------------------------------------------------
# Configure the lzf-config.cmake file for the build directory
#-----------------------------------------------------------------------------
set(INCLUDE_INSTALL_DIR ${LZF_INSTALL_INCLUDE_DIR} )
set(SHARE_INSTALL_DIR "${CMAKE_CURRENT_BINARY_DIR}/${LZF_INSTALL_CMAKE_DIR}" )
set(CURRENT_BUILD_DIR "${CMAKE_CURRENT_BINARY_DIR}" )
configure_package_config_file (
    ${LZF_RESOURCES_DIR}/lzf-config.cmake.in
    "${LZF_BINARY_DIR}/${LZF_PACKAGE}${LZF_PACKAGE_EXT}-config.cmake"
    INSTALL_DESTINATION "${LZF_INSTALL_CMAKE_DIR}"
    PATH_VARS INCLUDE_INSTALL_DIR SHARE_INSTALL_DIR CURRENT_BUILD_DIR
    INSTALL_PREFIX "${CMAKE_CURRENT_BINARY_DIR}"
)

#-----------------------------------------------------------------------------
# Configure the lzf-config.cmake file for the install directory
#-----------------------------------------------------------------------------
set(INCLUDE_INSTALL_DIR ${LZF_INSTALL_INCLUDE_DIR} )
set(SHARE_INSTALL_DIR "${CMAKE_INSTALL_PREFIX}/${LZF_INSTALL_CMAKE_DIR}" )
set(CURRENT_BUILD_DIR "${CMAKE_INSTALL_PREFIX}" )
configure_package_config_file (
    ${LZF_RESOURCES_DIR}/lzf-config.cmake.in
    "${LZF_BINARY_DIR}${CMAKE_FILES_DIRECTORY}/${LZF_PACKAGE}${LZF_PACKAGE_EXT}-config.cmake"
    INSTALL_DESTINATION "${LZF_INSTALL_CMAKE_DIR}"
    PATH_VARS INCLUDE_INSTALL_DIR SHARE_INSTALL_DIR CURRENT_BUILD_DIR
)
if (NOT LZF_EXTERNALLY_CONFIGURED)
  install (
      FILES ${LZF_BINARY_DIR}${CMAKE_FILES_DIRECTORY}/${LZF_PACKAGE}${LZF_PACKAGE_EXT}-config.cmake
      DESTINATION ${LZF_INSTALL_CMAKE_DIR}
      COMPONENT configinstall
  )
endif ()

#-----------------------------------------------------------------------------
# Configure the LZF-config-version.cmake file for the install directory
#-----------------------------------------------------------------------------
if (NOT LZF_EXTERNALLY_CONFIGURED)
  configure_file (
      ${LZF_RESOURCES_DIR}/lzf-config-version.cmake.in
      ${LZF_BINARY_DIR}${CMAKE_FILES_DIRECTORY}/${LZF_PACKAGE}${LZF_PACKAGE_EXT}-config-version.cmake @ONLY
  )
  install (
      FILES ${LZF_BINARY_DIR}${CMAKE_FILES_DIRECTORY}/${LZF_PACKAGE}${LZF_PACKAGE_EXT}-config-version.cmake
      DESTINATION ${LZF_INSTALL_CMAKE_DIR}
      COMPONENT configinstall
  )
endif ()

#-----------------------------------------------------------------------------
# Add Document File(s) to CMake Install
#-----------------------------------------------------------------------------
if (NOT LZF_EXTERNALLY_CONFIGURED)
  install (
      FILES
          ${LZF_SOURCE_DIR}/LICENSE
          ${LZF_SOURCE_DIR}/README
          ${LZF_SOURCE_DIR}/Changes
          ${LZF_SOURCE_DIR}/RELEASE_HDF.txt
      DESTINATION ${LZF_INSTALL_DATA_DIR}
      COMPONENT documents
  )
endif ()

#-----------------------------------------------------------------------------
# Check for Installation Utilities
#-----------------------------------------------------------------------------
if (WIN32)
  set (PF_ENV_EXT "(x86)")
  find_program (NSIS_EXECUTABLE NSIS.exe PATHS "$ENV{ProgramFiles}\\NSIS" "$ENV{ProgramFiles${PF_ENV_EXT}}\\NSIS")
  if(NOT CPACK_WIX_ROOT)
    file(TO_CMAKE_PATH "$ENV{WIX}" CPACK_WIX_ROOT)
  endif()
  find_program (WIX_EXECUTABLE candle  PATHS "${CPACK_WIX_ROOT}/bin")
endif ()

#-----------------------------------------------------------------------------
# Set the cpack variables
#-----------------------------------------------------------------------------
if (NOT LZF_EXTERNALLY_CONFIGURED)
  set (CPACK_PACKAGE_VENDOR "HDF_Group")
  set (CPACK_PACKAGE_NAME "${LZF_PACKAGE_NAME}")
  if (CDASH_LOCAL)
    set (CPACK_PACKAGE_VERSION "${LZF_PACKAGE_VERSION}")
  else (CDASH_LOCAL)
    set (CPACK_PACKAGE_VERSION "${LZF_PACKAGE_VERSION_STRING}")
  endif (CDASH_LOCAL)
  set (CPACK_PACKAGE_VERSION_MAJOR "${LZF_PACKAGE_VERSION_MAJOR}")
  set (CPACK_PACKAGE_VERSION_MINOR "${LZF_PACKAGE_VERSION_MINOR}")
  set (CPACK_PACKAGE_VERSION_PATCH "")
  set (CPACK_RESOURCE_FILE_LICENSE "${CMAKE_CURRENT_SOURCE_DIR}/README")
  set (CPACK_PACKAGE_DESCRIPTION_FILE "${CMAKE_CURRENT_SOURCE_DIR}/RELEASE_HDF.txt")
  set (CPACK_RESOURCE_FILE_README "${CMAKE_CURRENT_SOURCE_DIR}/RELEASE_HDF.txt")
  set (CPACK_PACKAGE_RELOCATABLE TRUE)
  set (CPACK_PACKAGE_DESCRIPTION_SUMMARY "lzf Installation")
  set (CPACK_PACKAGE_INSTALL_DIRECTORY "${CPACK_PACKAGE_VENDOR}/${CPACK_PACKAGE_NAME}/${CPACK_PACKAGE_VERSION}")

  set (CPACK_GENERATOR "TGZ")
  if (WIN32)
    set (CPACK_GENERATOR "ZIP")

    if (NSIS_EXECUTABLE)
      list (APPEND CPACK_GENERATOR "NSIS")
    endif ()
    # Installers for 32- vs. 64-bit CMake:
    #  - Root install directory (displayed to end user at installer-run time)
    #  - "NSIS package/display name" (text used in the installer GUI)
    #  - Registry key used to store info about the installation
    set (CPACK_NSIS_PACKAGE_NAME "${LZF_PACKAGE_STRING}")
    if (CMAKE_CL_64)
      set (CPACK_NSIS_INSTALL_ROOT "$PROGRAMFILES64")
      set (CPACK_PACKAGE_INSTALL_REGISTRY_KEY "${CPACK_PACKAGE_NAME}-${CPACK_PACKAGE_VERSION} (Win64)")
    else ()
      set (CPACK_NSIS_INSTALL_ROOT "$PROGRAMFILES")
      set (CPACK_PACKAGE_INSTALL_REGISTRY_KEY "${CPACK_PACKAGE_NAME}-${CPACK_PACKAGE_VERSION}")
    endif ()
    # set the install/unistall icon used for the installer itself
    # There is a bug in NSI that does not handle full unix paths properly.
    #set (CPACK_NSIS_MUI_ICON "${LZF_RESOURCES_DIR}\\\\LZF.ico")
    #set (CPACK_NSIS_MUI_UNIICON "${LZF_RESOURCES_DIR}\\\\LZF.ico")
    # set the package header icon for MUI
    #set (CPACK_PACKAGE_ICON "${LZF_RESOURCES_DIR}\\\\LZF.bmp")
    set (CPACK_NSIS_DISPLAY_NAME "${CPACK_NSIS_PACKAGE_NAME}")
    set (CPACK_PACKAGE_INSTALL_DIRECTORY "${CPACK_PACKAGE_VENDOR}\\\\${CPACK_PACKAGE_NAME}\\\\${CPACK_PACKAGE_VERSION}")
    set (CPACK_NSIS_CONTACT "${LZF_PACKAGE_BUGREPORT}")
    set (CPACK_NSIS_MODIFY_PATH ON)

    if (WIX_EXECUTABLE)
      list (APPEND CPACK_GENERATOR "WIX")
    endif ()
#WiX variables
    set (CPACK_WIX_UNINSTALL "1")
# .. variable:: CPACK_WIX_LICENSE_RTF
#  RTF License File
#
#  If CPACK_RESOURCE_FILE_LICENSE has an .rtf extension it is used as-is.
#
#  If CPACK_RESOURCE_FILE_LICENSE has an .txt extension it is implicitly
#  converted to RTF by the WiX Generator.
#  The expected encoding of the .txt file is UTF-8.
#
#  With CPACK_WIX_LICENSE_RTF you can override the license file used by the
#  WiX Generator in case CPACK_RESOURCE_FILE_LICENSE is in an unsupported
#  format or the .txt -> .rtf conversion does not work as expected.
    set (CPACK_RESOURCE_FILE_LICENSE "${CMAKE_CURRENT_SOURCE_DIR}/README")
# .. variable:: CPACK_WIX_PRODUCT_ICON
#  The Icon shown next to the program name in Add/Remove programs.
#    set(CPACK_WIX_PRODUCT_ICON "${LZF_RESOURCES_DIR}\\\\hdf.ico")
#
# .. variable:: CPACK_WIX_UI_BANNER
#
#  The bitmap will appear at the top of all installer pages other than the
#  welcome and completion dialogs.
#
#  If set, this image will replace the default banner image.
#
#  This image must be 493 by 58 pixels.
#
# .. variable:: CPACK_WIX_UI_DIALOG
#
#  Background bitmap used on the welcome and completion dialogs.
#
#  If this variable is set, the installer will replace the default dialog
#  image.
#
#  This image must be 493 by 312 pixels.
#
  elseif (APPLE)
    list (APPEND CPACK_GENERATOR "DragNDrop")
    set (CPACK_COMPONENTS_ALL_IN_ONE_PACKAGE ON)
    set (CPACK_PACKAGING_INSTALL_PREFIX "/${CPACK_PACKAGE_INSTALL_DIRECTORY}")
    #set (CPACK_PACKAGE_ICON "${LZF_RESOURCES_DIR}/hdf.icns")

    option (LZF_PACK_MACOSX_FRAMEWORK  "Package the LZF Library in a Bundle" OFF)
    if (LZF_PACK_MACOSX_FRAMEWORK AND LZF_BUILD_FRAMEWORKS)
      set (CPACK_BUNDLE_NAME "${LZF_PACKAGE_STRING}")
      set (CPACK_BUNDLE_LOCATION "/")    # make sure CMAKE_INSTALL_PREFIX ends in /
      set (CMAKE_INSTALL_PREFIX "/${CPACK_BUNDLE_NAME}.framework/Versions/${CPACK_PACKAGE_VERSION}/${CPACK_PACKAGE_NAME}/")
      set (CPACK_SHORT_VERSION_STRING "${CPACK_PACKAGE_VERSION}")
      #-----------------------------------------------------------------------------
      # Configure the Info.plist file for the install bundle
      #-----------------------------------------------------------------------------
      configure_file (
          ${LZF_RESOURCES_DIR}/CPack.Info.plist.in
          ${LZF_BINARY_DIR}/CMakeFiles/Info.plist @ONLY
      )
      configure_file (
          ${LZF_RESOURCES_DIR}/PkgInfo.in
          ${LZF_BINARY_DIR}/CMakeFiles/PkgInfo @ONLY
      )
      install (
          FILES ${LZF_BINARY_DIR}/CMakeFiles/PkgInfo
          DESTINATION ..
      )
    endif ()
  else (WIN32)
    list (APPEND CPACK_GENERATOR "STGZ")
    set (CPACK_PACKAGING_INSTALL_PREFIX "/${CPACK_PACKAGE_INSTALL_DIRECTORY}")
    set (CPACK_COMPONENTS_ALL_IN_ONE_PACKAGE ON)

    set (CPACK_DEBIAN_PACKAGE_SECTION "Libraries")
    set (CPACK_DEBIAN_PACKAGE_MAINTAINER "${LZF_PACKAGE_BUGREPORT}")

#      list (APPEND CPACK_GENERATOR "RPM")
    set (CPACK_RPM_PACKAGE_RELEASE "1")
    set (CPACK_RPM_COMPONENT_INSTALL ON)
    set (CPACK_RPM_PACKAGE_RELOCATABLE ON)
  endif ()

  # By default, do not warn when built on machines using only VS Express:
  if (NOT DEFINED CMAKE_INSTALL_SYSTEM_RUNTIME_LIBS_NO_WARNINGS)
    set (CMAKE_INSTALL_SYSTEM_RUNTIME_LIBS_NO_WARNINGS ON)
  endif ()
  include (InstallRequiredSystemLibraries)

  set (CPACK_INSTALL_CMAKE_PROJECTS "${LZF_BINARY_DIR};LZF;libraries;/")
  set (CPACK_INSTALL_CMAKE_PROJECTS "${LZF_BINARY_DIR};LZF;headers;/")
  set (CPACK_INSTALL_CMAKE_PROJECTS "${LZF_BINARY_DIR};LZF;configinstall;/")

  set (CPACK_ALL_INSTALL_TYPES Full Developer User)
  set (CPACK_INSTALL_TYPE_FULL_DISPLAY_NAME "Everything")

  set(CPACK_COMPONENTS_ALL libraries headers documents configinstall)

  include (CPack)

  cpack_add_component_group(Runtime)

  cpack_add_component_group(Documents
      EXPANDED
      DESCRIPTION "Release notes for lzf"
  )

  cpack_add_component_group(Development
      EXPANDED
      DESCRIPTION "All of the tools you'll need to develop applications"
  )

  cpack_add_component (libraries
      DISPLAY_NAME "LZF Libraries"
      REQUIRED
      GROUP Runtime
      INSTALL_TYPES Full Developer User
  )
  cpack_add_component (headers
      DISPLAY_NAME "LZF Headers"
      DEPENDS libraries
      GROUP Development
      INSTALL_TYPES Full Developer
  )
  cpack_add_component (documents
      DISPLAY_NAME "LZF Documents"
      GROUP Documents
      INSTALL_TYPES Full Developer
  )
  cpack_add_component (configinstall
      DISPLAY_NAME "LZF CMake files"
      DEPENDS libraries
      GROUP Development
      INSTALL_TYPES Full Developer User
  )

endif ()
