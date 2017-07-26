cmake_minimum_required(VERSION 3.0)

include("cmake/HunterGate.cmake")
HunterGate(
  URL "https://github.com/ruslo/hunter/archive/v0.19.44.tar.gz"
  SHA1 "a68f92d1370d738ff058ec1d36a02cf4cdc34af6"
)

project(pico VERSION 0.0.1)

option(PICO_BUILD_APPS "Build applications" OFF)

add_library(pico rnt/picornt.c rnt/picornt.h)

#inclue "picornt.h"
target_include_directories(pico
  INTERFACE
  "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/rnt>"
  )

if(PICO_BUILD_APPS)

  find_package(OpenMP)
  if (OPENMP_FOUND)
    add_executable(pico-train gen/picolrn.c)
    target_compile_definitions(pico-test PICO_HAS_OPENMP=1)    
    install(TARGETS pico-train DESTINATION bin)    
  endif()

  hunter_add_package(OpenCV)
  find_package(OpenCV REQUIRED)

  add_executable(pico-test rnt/sample/sample.c)  
  target_link_libraries(pico-test pico ${OpenCV_LIBS})
  install(TARGETS pico-test DESTINATION bin)
  
endif()

# Installation (https://github.com/forexample/package-example)

set(config_install_dir "lib/cmake/${PROJECT_NAME}")
set(generated_dir "${CMAKE_CURRENT_BINARY_DIR}/generated")
set(version_config "${generated_dir}/${PROJECT_NAME}ConfigVersion.cmake")
set(project_config "${generated_dir}/${PROJECT_NAME}Config.cmake")
set(TARGETS_EXPORT_NAME "${PROJECT_NAME}Targets")
set(namespace "${PROJECT_NAME}::")
set(include_install_dir "include")

include(CMakePackageConfigHelpers)

# Note: PROJECT_VERSION is used as a VERSION
write_basic_package_version_file(
  "${version_config}" COMPATIBILITY SameMajorVersion
  )

# Use variables:
#   * TARGETS_EXPORT_NAME
#   * PROJECT_NAME
configure_package_config_file(
  "${PROJECT_SOURCE_DIR}/cmake/Config.cmake.in"
  "${project_config}"
  INSTALL_DESTINATION "${config_install_dir}"
  )

install(
  FILES "${project_config}" "${version_config}"
  DESTINATION "${config_install_dir}"
  )

install(
  EXPORT "${TARGETS_EXPORT_NAME}"
  NAMESPACE "${namespace}"
  DESTINATION "${config_install_dir}"
  )

install(
  FILES
  rnt/picornt.h  
  DESTINATION "${include_install_dir}/${PROJECT_NAME}"
)

install(
  TARGETS pico
  EXPORT "${TARGETS_EXPORT_NAME}"
  LIBRARY DESTINATION "lib"
  ARCHIVE DESTINATION "lib"
  RUNTIME DESTINATION "bin"
  INCLUDES DESTINATION "include"
  )



