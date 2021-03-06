add_custom_target(coordgen_support ALL)

if(RDK_BUILD_COORDGEN_SUPPORT)
    if(NOT DEFINED MAEPARSER_DIR)
      set(MAEPARSER_DIR "${CMAKE_CURRENT_SOURCE_DIR}/maeparser")
    endif()
    if(NOT EXISTS "${MAEPARSER_DIR}/MaeParser.hpp")
        set(SHA "83368293dcc0eb07562dadfb7728b8d18d23a6cb")
        downloadAndCheckMD5("https://codeload.github.com/schrodinger/maeparser/tar.gz/${SHA}"
              "${CMAKE_CURRENT_SOURCE_DIR}/master.tar.gz"
              "32c0c3b315bba49fbf4c41a07aa58528")
        execute_process(COMMAND ${CMAKE_COMMAND} -E tar zxf
          ${CMAKE_CURRENT_SOURCE_DIR}/master.tar.gz
          WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR})
        file(RENAME "maeparser-${SHA}" "${MAEPARSER_DIR}")
    else()
      message("-- Found MAEParser source in ${MAEPARSER_DIR}")
    endif()

    file(GLOB MAESOURCES "${MAEPARSER_DIR}/*.cpp")
    rdkit_library(maeparser ${MAESOURCES} SHARED )
    if((MSVC AND RDK_INSTALL_DLLS_MSVC) OR ((NOT MSVC) AND WIN32))
      set_target_properties(maeparser PROPERTIES WINDOWS_EXPORT_ALL_SYMBOLS TRUE)
    endif()
    install(TARGETS maeparser DESTINATION ${RDKit_LibDir})

    if(NOT DEFINED COORDGEN_DIR)
      set(COORDGEN_DIR "${CMAKE_CURRENT_SOURCE_DIR}/coordgenlibs")
    endif()
    if(NOT EXISTS "${COORDGEN_DIR}/sketcherMinimizer.h")
        set(SHA "ede3191bf1c1dde53f70ca086014ff9ec7768c55")
        downloadAndCheckMD5("https://codeload.github.com/schrodinger/coordgenlibs/tar.gz/${SHA}"
              "${CMAKE_CURRENT_SOURCE_DIR}/master.tar.gz"
              "") # we aren't testing the md5 here because this code is still changing quickly
        execute_process(COMMAND ${CMAKE_COMMAND} -E tar zxf
          ${CMAKE_CURRENT_SOURCE_DIR}/master.tar.gz
          WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR})
        file(RENAME "coordgenlibs-${SHA}" "${COORDGEN_DIR}")
    else()
      message("-- Found coordgenlibs source in ${COORDGEN_DIR}")
    endif()

    file(GLOB CGSOURCES "${COORDGEN_DIR}/*.cpp")
    rdkit_library(coordgenlib ${CGSOURCES} SHARED LINK_LIBRARIES maeparser)
    if((MSVC AND RDK_INSTALL_DLLS_MSVC) OR ((NOT MSVC) AND WIN32))
      set_target_properties(coordgenlib PROPERTIES WINDOWS_EXPORT_ALL_SYMBOLS TRUE)
    endif()
    install(TARGETS coordgenlib DESTINATION ${RDKit_LibDir})

    set(MAE_FILES ${COORDGEN_DIR}/templates.mae )
    install(FILES ${MAE_FILES}
            DESTINATION ${RDKit_ShareDir}/Data
            COMPONENT data )

    include_directories(${CMAKE_CURRENT_SOURCE_DIR})
    set(RDK_COORDGEN_LIBS MolAlign coordgenlib maeparser CACHE STRING "the external libraries" FORCE)
    rdkit_headers(CoordGen.h DEST GraphMol)

    if(RDK_BUILD_PYTHON_WRAPPERS)
      add_subdirectory(Wrap)
    endif(RDK_BUILD_PYTHON_WRAPPERS)

    rdkit_test(testCoordGen test.cpp
      LINK_LIBRARIES
      ${RDK_COORDGEN_LIBS} Depictor
      FileParsers SmilesParse SubstructMatch GraphMol
      RDGeneral DataStructs RDGeneral RDGeometryLib
      ${RDKit_THREAD_LIBS})
else (RDK_BUILD_COORDGEN_SUPPORT)
    set(RDK_COORDGEN_LIBS  CACHE STRING "the external libraries" FORCE)
endif(RDK_BUILD_COORDGEN_SUPPORT)
