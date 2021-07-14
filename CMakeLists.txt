project(git-clang-format-cmake)
cmake_minimum_required(VERSION 3.0)

find_package(Git REQUIRED)
find_package(PythonInterp REQUIRED)

list(APPEND CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR})

set(GCF_CLANGFORMAT_STYLE "file" CACHE STRING
    "Parameter pass to clang-format -style=<here>")
set(GCF_IGNORE_LIST "" CACHE STRING
    "Semi colon separated list of directories to ignore")
set(GCF_CLANGFORMAT_MINIMAL_VERSION "0.0.0" CACHE STRING
    "The minimal required version of clang-format")
set(GCF_FORCE_OVERWRITE OFF CACHE BOOL
    "If true, always overwrite pre-commit hook and script")
set(GCF_SCRIPT ${CMAKE_CURRENT_SOURCE_DIR}/git-cmake-format.py)

find_package(ClangFormat ${GCF_CLANGFORMAT_MINIMAL_VERSION} REQUIRED)

execute_process(COMMAND ${GIT_EXECUTABLE} rev-parse --git-path hooks
  WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
  OUTPUT_VARIABLE GCF_GIT_HOOKS_DIR
  OUTPUT_STRIP_TRAILING_WHITESPACE)
get_filename_component(GCF_GIT_HOOKS_DIR ${GCF_GIT_HOOKS_DIR} REALPATH BASE_DIR ${CMAKE_SOURCE_DIR})
if(NOT GCF_GIT_HOOKS_DIR)
  message(WARNING "Not in a git repository")
else()
  set(GCF_GIT_DIR_SCRIPT ${GCF_GIT_HOOKS_DIR}/git-cmake-format.py)
  if((NOT GCF_FORCE_OVERWRITE) AND (EXISTS ${GCF_GIT_HOOKS_DIR}/pre-commit))
    message(STATUS "custom hook is already installed, skipping it (overwrite with GCF_FORCE_OVERWRITE=ON)")
  else()
    message(STATUS "Installing git-cmake-format.py and pre-commit hook into ${GCF_GIT_HOOKS_DIR}")

    configure_file(
      ${GCF_SCRIPT} ${GCF_GIT_DIR_SCRIPT}
      COPYONLY)

    configure_file(
      ${CMAKE_CURRENT_SOURCE_DIR}/git-pre-commit-hook
      ${GCF_GIT_HOOKS_DIR}/pre-commit
      @ONLY)
  endif()
  unset(GCF_SCRIPT)
  unset(GCF_GIT_HOOKS_DIR)

  add_custom_target(format
    ${PYTHON_EXECUTABLE} ${GCF_GIT_DIR_SCRIPT}
    --cmake ${GIT_EXECUTABLE}
    ${ClangFormat_EXECUTABLE} -style=${GCF_CLANGFORMAT_STYLE}
    -ignore=${GCF_IGNORE_LIST}
    WORKING_DIRECTORY ${CMAKE_SOURCE_DIR})
  unset(GCF_GIT_DIR_SCRIPT)
endif()
