project(qcatool)

set(qcatool_moc_SRCS main.cpp)

MY_AUTOMOC( qcatool_moc_SRCS )

set(qcatool_bin_SRCS ${qcatool_moc_SRCS})

add_executable(${QCA_TOOL_NAME} ${qcatool_bin_SRCS})

target_link_qca_libraries(${QCA_TOOL_NAME})

if(OS2)
  set_target_properties(${QCA_TOOL_NAME} PROPERTIES
    OS2_DEF_VENDOR "$ENV{VENDOR}"
    OS2_DEF_PATCH "${QCA_LIB_PATCH_VERSION}"
    OS2_DEF_VERSION "${QCA_LIB_VERSION_STRING}")
endif()

if(NOT DEVELOPER_MODE)
  install(TARGETS ${QCA_TOOL_NAME} DESTINATION "${QCA_BINARY_INSTALL_DIR}")
  install_pdb(${QCA_TOOL_NAME} ${QCA_BINARY_INSTALL_DIR})
endif()
