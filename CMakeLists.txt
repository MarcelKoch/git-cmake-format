project(git-clang-format-cmake)
cmake_minimum_required(VERSION 2.8)

find_package(Git REQUIRED)
find_package(PythonInterp REQUIRED)

list(APPEND CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR})

set(GCF_CLANGFORMAT_STYLE "file" CACHE STRING
    "Parameter pass to clang-format -style=<here>")
set(GCF_IGNORE_LIST "" CACHE STRING
    "Semi colon separated list of directories to ignore")
set(GCF_CLANGFORMAT_MINIMAL_VERSION "0.0.0" CACHE STRING
    "The minimal required version of clang-format")
set(GCF_SCRIPT ${CMAKE_CURRENT_SOURCE_DIR}/git-cmake-format.py)

find_package(ClangFormat ${GCF_CLANGFORMAT_MINIMAL_VERSION} REQUIRED)

execute_process(COMMAND ${GIT_EXECUTABLE} rev-parse --show-toplevel
  WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
  OUTPUT_VARIABLE GCF_GIT_ROOT
  OUTPUT_STRIP_TRAILING_WHITESPACE)
if(NOT GCF_GIT_ROOT)
  message(WARNING "Not in a git repository")
else()
  configure_file(
    ${CMAKE_CURRENT_SOURCE_DIR}/git-pre-commit-hook
    ${GCF_GIT_ROOT}/.git/hooks/pre-commit
    @ONLY)
  unset(GCF_GIT_ROOT)

  add_custom_target(format
    ${PYTHON_EXECUTABLE} ${GCF_SCRIPT}
    --cmake ${GIT_EXECUTABLE}
    ${ClangFormat_EXECUTABLE} -style=${GCF_CLANGFORMAT_STYLE}
    -ignore=${GCF_IGNORE_LIST}
    WORKING_DIRECTORY ${CMAKE_SOURCE_DIR})
  unset(GCF_SCRIPT)
endif()
