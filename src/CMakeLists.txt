include(CheckIncludeFiles)
CHECK_INCLUDE_FILES(sys/filio.h HAVE_SYS_FILIO_H)
IF(HAVE_SYS_FILIO_H)
  ADD_DEFINITIONS(-DHAVE_SYS_FILIO_H)
ENDIF(HAVE_SYS_FILIO_H)

INCLUDE(CheckCXXSourceCompiles)
CHECK_CXX_SOURCE_COMPILES("
# include <stdlib.h>
# include <sys/mman.h>
int main() { void *f = 0; return mlock(f,8); }
" MLOCK_TAKES_VOID)
if(NOT MLOCK_TAKES_VOID)
  MESSAGE(STATUS "mlock(2) does not take a void *")
  ADD_DEFINITIONS(-DMLOCK_NOT_VOID_PTR)
endif(NOT MLOCK_TAKES_VOID)

if(DEVELOPER_MODE)
  add_definitions(-DQCA_PLUGIN_PATH="${CMAKE_BINARY_DIR}/lib/${QCA_LIB_NAME}")
else()
  if(USE_RELATIVE_PATHS)
    add_definitions(-DQCA_PLUGIN_PATH="${QCA_PREFIX_INSTALL_DIR}/${QCA_PLUGINS_INSTALL_DIR}")
  else()
    add_definitions(-DQCA_PLUGIN_PATH="${QCA_PLUGINS_INSTALL_DIR}")
  endif()
endif()

# base source files


SET( nonmoc_SOURCES
	qca_tools.cpp
	qca_plugin.cpp
	qca_textfilter.cpp
	qca_basic.cpp
	support/logger.cpp
)

SET( moc_SOURCES
	qca_cert.cpp
	qca_core.cpp
	qca_default.cpp
	qca_keystore.cpp
	qca_publickey.cpp
	qca_safetimer.cpp
	qca_securelayer.cpp
	qca_securemessage.cpp
	support/qpipe.cpp
	support/console.cpp
	support/synchronizer.cpp
	support/dirwatch.cpp
	support/syncthread.cpp
)

IF (WIN32)
  SET( nonmoc_SOURCES ${nonmoc_SOURCES} qca_systemstore_win.cpp )
elseif(APPLE)
  set( nonmoc_SOURCES ${nonmoc_SOURCES} qca_systemstore_mac.cpp)
else()
  SET( nonmoc_SOURCES ${nonmoc_SOURCES} qca_systemstore_flatfile.cpp )
endif()

# Support files
#SET( qca_HEADERS ${qca_HEADERS} support/dirwatch/dirwatch_p.h )

# Botan tools
SET( botan_BASE botantools/botan )


INCLUDE_DIRECTORIES(${QT_QTCORE_INCLUDE_DIR} support ${botan_BASE} )

ADD_DEFINITIONS(
	-DBOTAN_TYPES_QT
	-DBOTAN_NO_INIT_H
	-DBOTAN_NO_CONF_H
	-DBOTAN_TOOLS_ONLY
	-DBOTAN_MINIMAL_BIGINT
)

ADD_DEFINITIONS(
  -DBOTAN_MP_WORD_BITS=32
  -DBOTAN_KARAT_MUL_THRESHOLD=12
  -DBOTAN_KARAT_SQR_THRESHOLD=12
  -DBOTAN_EXT_MUTEX_QT
)

if(UNIX OR OS2)
   ADD_DEFINITIONS( -DBOTAN_EXT_ALLOC_MMAP)
endif(UNIX OR OS2)

SET( botan_SOURCES
	${botan_BASE}/util.cpp
	${botan_BASE}/exceptn.cpp
	${botan_BASE}/mutex.cpp
	${botan_BASE}/mux_qt/mux_qt.cpp
	${botan_BASE}/charset.cpp
	${botan_BASE}/defalloc.cpp
	${botan_BASE}/mp_comba.cpp
	${botan_BASE}/mp_mul.cpp
	${botan_BASE}/mp_shift.cpp
	${botan_BASE}/mp_misc.cpp
	${botan_BASE}/divide.cpp
	${botan_BASE}/big_base.cpp
	${botan_BASE}/big_code.cpp
	${botan_BASE}/big_io.cpp
	${botan_BASE}/big_ops2.cpp
	${botan_BASE}/big_ops3.cpp
	${botan_BASE}/bit_ops.cpp
	${botan_BASE}/libstate.cpp
	${botan_BASE}/mem_pool.cpp
	${botan_BASE}/modules.cpp
	${botan_BASE}/mp_asm.cpp
	${botan_BASE}/mp_mulop.cpp
	${botan_BASE}/parsing.cpp
)

IF (UNIX OR OS2)
  SET( botan_SOURCES ${botan_SOURCES} ${botan_BASE}/ml_unix/mlock.cpp)
  SET( botan_SOURCES ${botan_SOURCES} ${botan_BASE}/alloc_mmap/mmap_mem.cpp)
ENDIF (UNIX OR OS2)

IF(WIN32)
  SET( botan_SOURCES ${botan_SOURCES} ${botan_BASE}/ml_win32/mlock.cpp)
ENDIF(WIN32)

MY_AUTOMOC( moc_SOURCES )
SET( SOURCES ${SOURCES} ${moc_SOURCES} ${nonmoc_SOURCES} )

SET( SOURCES ${SOURCES} ${botan_SOURCES})

qt4_wrap_cpp( SOURCES "${qca_INCLUDEDIR}/QtCrypto/qca_core.h")
qt4_wrap_cpp( SOURCES "${qca_INCLUDEDIR}/QtCrypto/qca_cert.h")
qt4_wrap_cpp( SOURCES "${qca_INCLUDEDIR}/QtCrypto/qca_keystore.h")
qt4_wrap_cpp( SOURCES "${qca_INCLUDEDIR}/QtCrypto/qcaprovider.h")
qt4_wrap_cpp( SOURCES "${qca_INCLUDEDIR}/QtCrypto/qca_publickey.h")
qt4_wrap_cpp( SOURCES "${qca_INCLUDEDIR}/QtCrypto/qca_securelayer.h")
qt4_wrap_cpp( SOURCES "${qca_INCLUDEDIR}/QtCrypto/qca_securemessage.h")
qt4_wrap_cpp( SOURCES "${qca_INCLUDEDIR}/QtCrypto/qca_support.h")
qt4_wrap_cpp( SOURCES "${qca_INCLUDEDIR}/QtCrypto/qpipe.h")
qt4_wrap_cpp( SOURCES "qca_safeobj.h")
qt4_wrap_cpp( SOURCES "${qca_INCLUDEDIR}/QtCrypto/qca_safetimer.h")

add_library(${QCA_LIB_NAME} ${SOURCES}  ${public_HEADERS})
TARGET_LINK_LIBRARIES(${QCA_LIB_NAME} ${QT_QTCORE_LIBRARY})

if(WIN32)
	TARGET_LINK_LIBRARIES(${QCA_LIB_NAME} crypt32 ws2_32)
endif(WIN32)

if(OS2)
  set_target_properties(${QCA_LIB_NAME} PROPERTIES
    OS2_DEF_VENDOR "$ENV{VENDOR}"
    OS2_DEF_PATCH "${QCA_LIB_PATCH_VERSION}"
    OS2_DEF_VERSION "${QCA_LIB_VERSION_STRING}")
endif()

if(APPLE)
  set(COREFOUNDATION_LIBRARY "-framework CoreFoundation")
  set(COREFOUNDATION_LIBRARY_SECURITY "-framework Security")
  TARGET_LINK_LIBRARIES(${QCA_LIB_NAME} ${COREFOUNDATION_LIBRARY} ${COREFOUNDATION_LIBRARY_SECURITY})

  if(OSX_FRAMEWORK AND NOT USE_RELATIVE_PATHS)
    set_target_properties(${QCA_LIB_NAME} PROPERTIES
                          INSTALL_NAME_DIR "${QCA_LIBRARY_INSTALL_DIR}"
    )
  endif()
endif(APPLE)

if(NOT ANDROID)
  set_target_properties(${QCA_LIB_NAME} PROPERTIES
                        VERSION ${QCA_LIB_MAJOR_VERSION}.${QCA_LIB_MINOR_VERSION}.${QCA_LIB_PATCH_VERSION}
                        SOVERSION ${QCA_LIB_MAJOR_VERSION}
                        )
endif()

set_target_properties(${QCA_LIB_NAME} PROPERTIES
                      DEFINE_SYMBOL QCA_MAKEDLL
                      PUBLIC_HEADER "${public_HEADERS}"
                      FRAMEWORK ${OSX_FRAMEWORK}
                      EXPORT_NAME ${QCA_LIB_NAME}
                      )

if(NOT DEVELOPER_MODE)
  # Do not split 'PUBLIC_HEADER ...' line. It means install headers to folder
  # and set this folder as -I flag for imported target.
  # Also EXPORT doesn't actually install any files. It only created a new target.
  install(TARGETS ${QCA_LIB_NAME} EXPORT ${QCA_CONFIG_NAME_BASE}Targets
          LIBRARY DESTINATION "${QCA_LIBRARY_INSTALL_DIR}"
          RUNTIME DESTINATION "${QCA_BINARY_INSTALL_DIR}"
          ARCHIVE DESTINATION "${QCA_LIBRARY_INSTALL_DIR}"
          FRAMEWORK DESTINATION "${QCA_LIBRARY_INSTALL_DIR}"
          PUBLIC_HEADER DESTINATION "${QCA_FULL_INCLUDE_INSTALL_DIR}" INCLUDES DESTINATION "${QCA_FULL_INCLUDE_INSTALL_DIR}"
  )
  install_pdb(${QCA_LIB_NAME} ${QCA_BINARY_INSTALL_DIR})
endif()

