cmake_minimum_required(VERSION 3.5)
project(osqp_interface)

if(NOT CMAKE_CXX_STANDARD)
  set(CMAKE_CXX_STANDARD 14)
endif()

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Ofast)
endif()

find_package(ament_cmake_auto REQUIRED)
ament_auto_find_build_dependencies()

find_package(Eigen3 REQUIRED)

# find_package(osqp REQUIRED)
# get_target_property(OSQP_INCLUDE_DIR osqp::osqpstatic INTERFACE_INCLUDE_DIRECTORIES)

set(EXTERNAL_INCLUDE_DIRS
  "${EIGEN3_INCLUDE_DIR}"
  "${OSQP_INCLUDE_DIR}"
  )

ament_auto_add_library(osqp_interface
  src/osqp_interface.cpp
  src/csc_matrix_conv.cpp
  include/osqp_interface/osqp_interface.h
  include/osqp_interface/csc_matrix_conv.h
  )
# target_link_libraries(osqp_interface osqp::osqpstatic)
ament_target_dependencies(osqp_interface osqp)
# target_link_libraries(osqp_interface osqp_vendor::osqp)

target_include_directories(osqp_interface PUBLIC "${EXTERNAL_INCLUDE_DIRS}")
# needed so clients of this package don't need to worry about includes of this package
ament_export_include_directories("${EXTERNAL_INCLUDE_DIRS}")

ament_auto_package()

## https://stackoverflow.com/questions/32183975/how-to-print-all-the-properties-of-a-target-in-cmake/56738858#56738858
## https://stackoverflow.com/a/56738858/3743145

## Get all properties that cmake supports
execute_process(COMMAND cmake --help-property-list OUTPUT_VARIABLE CMAKE_PROPERTY_LIST)
## Convert command output into a CMake list
STRING(REGEX REPLACE ";" "\\\\;" CMAKE_PROPERTY_LIST "${CMAKE_PROPERTY_LIST}")
STRING(REGEX REPLACE "\n" ";" CMAKE_PROPERTY_LIST "${CMAKE_PROPERTY_LIST}")

list(REMOVE_DUPLICATES CMAKE_PROPERTY_LIST)

function(print_target_properties tgt)
    if(NOT TARGET ${tgt})
      message("There is no target named '${tgt}'")
      return()
    endif()

    foreach (prop ${CMAKE_PROPERTY_LIST})
        string(REPLACE "<CONFIG>" "${CMAKE_BUILD_TYPE}" prop ${prop})
        get_target_property(propval ${tgt} ${prop})
        if (propval)
            get_target_property(propval ${tgt} ${prop})
            message ("${tgt} ${prop} = ${propval}")
        endif()
    endforeach(prop)
  endfunction(print_target_properties)

print_target_properties(osqp)

  function(dump_cmake_variables)
    get_cmake_property(_variableNames VARIABLES)
    list (SORT _variableNames)
    foreach (_variableName ${_variableNames})
        if (ARGV0)
            unset(MATCHED)
            string(REGEX MATCH ${ARGV0} MATCHED ${_variableName})
            if (NOT MATCHED)
                continue()
            endif()
        endif()
        message(STATUS "${_variableName}=${${_variableName}}")
    endforeach()
endfunction()

dump_cmake_variables(osqp_vendor)
