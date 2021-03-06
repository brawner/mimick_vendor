cmake_minimum_required(VERSION 3.5)

project(mimick_vendor)

find_package(ament_cmake REQUIRED)

macro(build_mimick)

  set(cmake_commands)
  set(cmake_configure_args -Wno-dev)
  set(cmake_configure_args -DCMAKE_INSTALL_PREFIX=${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}_install)

  if(WIN32)
    if(DEFINED CMAKE_GENERATOR)
      list(APPEND cmake_configure_args -G ${CMAKE_GENERATOR})
    endif()
    if("${CMAKE_SYSTEM_PROCESSOR}" MATCHES "^(x86_|x86-|AMD|amd|x)64$")
      list(APPEND cmake_configure_args -A x64)
    endif()
  endif()

  if(DEFINED CMAKE_BUILD_TYPE)
    if(WIN32)
      build_command(_build_command CONFIGURATION ${CMAKE_BUILD_TYPE})
      list(APPEND cmake_commands "BUILD_COMMAND ${_build_command}")
    else()
      list(APPEND cmake_configure_args -DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE})
    endif()
  endif()

  if(DEFINED CMAKE_TOOLCHAIN_FILE)
    list(APPEND cmake_configure_args "-DCMAKE_TOOLCHAIN_FILE=${CMAKE_TOOLCHAIN_FILE}")
    if(ANDROID)
      if(DEFINED ANDROID_ABI)
        list(APPEND cmake_configure_args "-DANDROID_ABI=${ANDROID_ABI}")
      endif()
      if(DEFINED ANDROID_CPP_FEATURES)
        list(APPEND cmake_configure_args "-DANDROID_CPP_FEATURES=${ANDROID_CPP_FEATURES}")
      endif()
      if(DEFINED ANDROID_FUNCTION_LEVEL_LINKING)
        list(APPEND cmake_configure_args "-DANDROID_FUNCTION_LEVEL_LINKING=${ANDROID_FUNCTION_LEVEL_LINKING}")
      endif()
      if(DEFINED ANDROID_NATIVE_API_LEVEL)
        list(APPEND cmake_configure_args "-DANDROID_NATIVE_API_LEVEL=${ANDROID_NATIVE_API_LEVEL}")
      endif()
      if(DEFINED ANDROID_NDK)
        list(APPEND cmake_configure_args "-DANDROID_NDK=${ANDROID_NDK}")
      endif()
      if(DEFINED ANDROID_STL)
        list(APPEND cmake_configure_args "-DANDROID_STL=${ANDROID_STL}")
      endif()
      if(DEFINED ANDROID_TOOLCHAIN_NAME)
        list(APPEND cmake_configure_args "-DANDROID_TOOLCHAIN_NAME=${ANDROID_TOOLCHAIN_NAME}")
      endif()
    endif()
  else()
    list(APPEND cmake_configure_args "-DCMAKE_CXX_COMPILER=${CMAKE_CXX_COMPILER}")
  endif()

  include(ExternalProject)
  externalproject_add(mimick-ext
    GIT_REPOSITORY https://github.com/ros2/Mimick.git
    GIT_TAG 1c0387257ae9109c87a15fef6485f58df675d2ac
    TIMEOUT 6000
    ${cmake_commands}
    CMAKE_ARGS
      ${cmake_configure_args}
  )

  # The external project will install to the build folder, but we'll install that on make install.
  install(
    DIRECTORY
      ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}_install/
    DESTINATION
      ${CMAKE_INSTALL_PREFIX}
  )

endmacro()

build_mimick()

ament_export_libraries(mimick)
ament_export_dependencies(mimick)

if(BUILD_TESTING)
  find_package(ament_lint_auto REQUIRED)
  ament_lint_auto_find_test_dependencies()
endif()

# this ensures that the package has an environment hook setting the PATH
ament_package()
