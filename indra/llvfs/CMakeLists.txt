# -*- cmake -*-

project(llvfs)

include(00-Common)
include(LLCommon)
include(UnixInstall)
include(PrebuiltPlugin)

include_directories(
    ${LLCOMMON_INCLUDE_DIRS}
    )

set(llvfs_SOURCE_FILES
    lldir.cpp
    lldiriterator.cpp
    lllfsthread.cpp
    llpidlock.cpp
    llvfile.cpp
    llvfs.cpp
    llvfsthread.cpp
    )

set(llvfs_HEADER_FILES
    CMakeLists.txt

    lldir.h
    lldiriterator.h
    lllfsthread.h
    llpidlock.h
    llvfile.h
    llvfs.h
    llvfsthread.h
    )

if (DARWIN)
  LIST(APPEND llvfs_SOURCE_FILES lldir_mac.cpp)
  LIST(APPEND llvfs_HEADER_FILES lldir_mac.h)
endif (DARWIN)

if (LINUX)
  LIST(APPEND llvfs_SOURCE_FILES lldir_linux.cpp)
  LIST(APPEND llvfs_HEADER_FILES lldir_linux.h)

  if (INSTALL)
    set_source_files_properties(lldir_linux.cpp
                                PROPERTIES COMPILE_FLAGS
                                "-DAPP_RO_DATA_DIR=\\\"${APP_SHARE_DIR}\\\""
                                )
  endif (INSTALL)
endif (LINUX)

if (WINDOWS)
  LIST(APPEND llvfs_SOURCE_FILES lldir_win32.cpp)
  LIST(APPEND llvfs_HEADER_FILES lldir_win32.h)
endif (WINDOWS)

set_source_files_properties(${llvfs_HEADER_FILES}
                            PROPERTIES HEADER_FILE_ONLY TRUE)

list(APPEND llvfs_SOURCE_FILES ${llvfs_HEADER_FILES})

add_library (llvfs ${llvfs_SOURCE_FILES})
add_dependencies(llvfs prepare)

target_link_libraries(llvfs
    ${Boost_FILESYSTEM_LIBRARY}
    ${Boost_SYSTEM_LIBRARY}
    )

if (DARWIN)
  include(CMakeFindFrameworks)
  find_library(CARBON_LIBRARY Carbon)
  target_link_libraries(llvfs ${CARBON_LIBRARY})
endif (DARWIN)

if (SKIP_BUILDING_PLUGIN)
    add_definitions(-DUSE_PREBUILT_PLUGIN=1)
endif (SKIP_BUILDING_PLUGIN)
