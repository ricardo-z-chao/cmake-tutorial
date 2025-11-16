cmake_minimum_required(VERSION 3.15)
set(gcc_like_cxx "$<COMPILE_LANG_AND_ID:CXX,ARMClang,AppleClang,Clang,GNU,LCC>")
set(msvc_cxx "$<COMPILE_LANG_AND_ID:CXX,MSVC>")
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY "${PROJECT_BINARY_DIR}")
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY "${PROJECT_BINARY_DIR}")
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${PROJECT_BINARY_DIR}")
option(BUILD_SHARED_LIBS "Build using shared libraries" ON)
project(Tutorial VERSION 1.0)
add_executable(Tutorial tutorial.cxx)
configure_file(TutorialConfig.h.in TutorialConfig.h)
target_include_directories(Tutorial PUBLIC "${PROJECT_BINARY_DIR}")
add_library(tutorial_compiler_flags INTERFACE)
target_compile_features(tutorial_compiler_flags INTERFACE cxx_std_11)
target_compile_options(
  tutorial_compiler_flags
  INTERFACE
    "$<${gcc_like_cxx}:$<BUILD_INTERFACE:-Wall;-Wextra;-Wshadow;-Wformat=2;-Wunused>>"
    "$<${msvc_cxx}:$<BUILD_INTERFACE:-W3>>")
add_subdirectory(MathFunctions)
target_link_libraries(Tutorial PUBLIC MathFunctions)

include(CTest)
if(BUILD_TESTING)
  add_test(NAME Runs COMMAND Tutorial 25)
  add_test(NAME Usage COMMAND Tutorial)
  set_tests_properties(Usage PROPERTIES PASS_REGULAR_EXPRESSION
                                        "Usage:.*number")
  add_test(NAME StandardUse COMMAND Tutorial 4)
  set_tests_properties(StandardUse PROPERTIES PASS_REGULAR_EXPRESSION "4 is 2")

  function(do_test target arg result)
    add_test(NAME Comp${arg} COMMAND ${target} ${arg})
    set_tests_properties(Comp${arg} PROPERTIES PASS_REGULAR_EXPRESSION
                                               ${result})
  endfunction()

  # do a bunch of result based tests
  do_test(Tutorial 4 "4 is 2")
  do_test(Tutorial 9 "9 is 3")
  do_test(Tutorial 5 "5 is 2.236")
  do_test(Tutorial 7 "7 is 2.645")
  do_test(Tutorial 25 "25 is 5")
  do_test(Tutorial -25 "-25 is (-nan|nan|0)")
  do_test(Tutorial 0.0001 "0.0001 is 0.01")
endif()

include(InstallRequiredSystemLibraries)
set(CPACK_RESOURCE_FILE_LICENSE "${CMAKE_CURRENT_SOURCE_DIR}/LICENSE")
set(CPACK_PACKAGE_VERSION_MAJOR "${Tutorial_VERSION_MAJOR}")
set(CPACK_PACKAGE_VERSION_MINOR "${Tutorial_VERSION_MINOR}")
set(CPACK_GENERATOR "ZIP")
set(CPACK_SOURCE_GENERATOR "ZIP")
include(CPack)

include(CMakePackageConfigHelpers)
install(
  EXPORT MathFunctionsTargets
  FILE MathFunctionsTargets.cmake
  DESTINATION lib/cmake/MathFunctions)
export(EXPORT MathFunctionsTargets
       FILE "${CMAKE_CURRENT_BINARY_DIR}/MathFunctionsTargets.cmake")
configure_package_config_file(
  ${CMAKE_CURRENT_SOURCE_DIR}/config.cmake.in
  "${CMAKE_CURRENT_BINARY_DIR}/MathFunctionsConfig.cmake"
  INSTALL_DESTINATION "lib/cmake/MathFunctions"
  NO_SET_AND_CHECK_MACRO NO_CHECK_REQUIRED_COMPONENTS_MACRO)
write_basic_package_version_file(
  "${CMAKE_CURRENT_BINARY_DIR}/MathFunctionsConfigVersion.cmake"
  VERSION "${Tutorial_VERSION_MAJOR}.${Tutorial_VERSION_MINOR}"
  COMPATIBILITY AnyNewerVersion)
install(FILES ${CMAKE_CURRENT_BINARY_DIR}/MathFunctionsConfig.cmake
              ${CMAKE_CURRENT_BINARY_DIR}/MathFunctionsConfigVersion.cmake
        DESTINATION lib/cmake/MathFunctions)
