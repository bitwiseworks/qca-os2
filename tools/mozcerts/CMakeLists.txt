project(mozcerts)

set(mozcerts_bin_SRCS main.cpp)  

add_executable(${MOZCERTS_NAME} ${mozcerts_bin_SRCS})

target_link_qca_libraries(${MOZCERTS_NAME})

if(OS2)
  set_target_properties(${MOZCERTS_NAME} PROPERTIES
    OS2_DEF_VENDOR "$ENV{VENDOR}"
    OS2_DEF_PATCH "${QCA_LIB_PATCH_VERSION}"
    OS2_DEF_VERSION "${QCA_LIB_VERSION_STRING}")
endif()

if(NOT DEVELOPER_MODE)
  install(TARGETS ${MOZCERTS_NAME} DESTINATION "${QCA_BINARY_INSTALL_DIR}")
  install_pdb(${MOZCERTS_NAME} ${QCA_BINARY_INSTALL_DIR})
endif()
