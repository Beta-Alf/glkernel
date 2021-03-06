
set(target debugger)


# Qt5

# good resource: http://www.kdab.com/using-cmake-with-qt-5/

# http://qt-project.org/forums/viewthread/30006/
if(MSVC)
    cmake_policy(SET CMP0020 NEW)
endif()


# External libraries

find_package(GLM REQUIRED)

find_package(Qt5OpenGL  5.1 QUIET)
find_package(Qt5Core    5.1 QUIET)
find_package(Qt5Gui     5.1 QUIET)
find_package(Qt5Widgets 5.1 QUIET)

if (NOT Qt5Core_FOUND)
    message("Example ${target} skipped: Qt5 not found")
    return()
endif()

message(STATUS "App ${target}")

set(CMAKE_AUTOMOC ON)
set(AUTOMOC_MOC_OPTIONS PROPERTIES FOLDER CMakeAutomocTargets)

# Probably works in the next cmake release -> http://www.cmake.org/Bug/view.php?id=13788
# What we do not want is automocs beside the project -> http://www.cmake.org/Bug/view.php?id=13688
set_property(GLOBAL PROPERTY AUTOMOC_FOLDER CMakeAutomocTargets)


if(OPTION_USE_LOOP_PARALLELIZATION)

    find_package(OpenMP QUIET)

    if (NOT OPENMP_FOUND)
        message("Loop parallelization in ${target} skipped: OpenMP not found")
    endif()

endif()


# Includes

include_directories(
    ${GLM_INCLUDE_DIR}
)

include_directories(
    BEFORE
    ${CMAKE_SOURCE_DIR}/source/glkernel/include
    ${CMAKE_CURRENT_SOURCE_DIR}
    ${CMAKE_CURRENT_BINARY_DIR}
)


# Libraries

set(libs
#   glkernel
    Qt5::OpenGL
    Qt5::Core
    Qt5::Gui
    Qt5::Widgets
)


# Compiler definitions

if (OPTION_BUILD_STATIC)
    add_definitions("-D${META_PROJECT_NAME_UPPER}_STATIC")
endif()

add_definitions("-DGLM_FORCE_RADIANS")

if(OPENMP_FOUND)
    set(CMAKE_C_FLAGS   "${CMAKE_C_FLAGS}   ${OpenMP_C_FLAGS}")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${OpenMP_CXX_FLAGS}")
endif()


# Sources

set(sources
    main.cpp
)


# Build executable

add_executable(${target} ${sources})

target_link_libraries(${target} ${libs})

target_compile_options(${target} PRIVATE ${DEFAULT_COMPILE_FLAGS})

set_target_properties(${target}
    PROPERTIES
    LINKER_LANGUAGE              CXX
    FOLDER                      "${IDE_FOLDER}"
    COMPILE_DEFINITIONS_DEBUG   "${DEFAULT_COMPILE_DEFS_DEBUG}"
    COMPILE_DEFINITIONS_RELEASE "${DEFAULT_COMPILE_DEFS_RELEASE}"
    LINK_FLAGS_DEBUG            "${DEFAULT_LINKER_FLAGS_DEBUG}"
    LINK_FLAGS_RELEASE          "${DEFAULT_LINKER_FLAGS_RELEASE}"
    DEBUG_POSTFIX               "d${DEBUG_POSTFIX}")


# Deployment

install(TARGETS ${target}
    RUNTIME DESTINATION ${INSTALL_BIN}
#   LIBRARY DESTINATION ${INSTALL_SHARED}
#   ARCHIVE DESTINATION ${INSTALL_LIB}
)
