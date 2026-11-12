cmake_minimum_required(VERSION 3.20)
# byo-pop-rocketmania: one game of byo-pop (https://github.com/allisonpelton/byo-pop), generated from its top-level
# CMakeLists.txt by tools/publish.py. The game's files come from the user's own install (assets/rocketmania/).
# Native builds only: the browser version is built from byo-pop, which has the site.
project(byo-pop-rocketmania C CXX)
if(EMSCRIPTEN)
  message(FATAL_ERROR "This repository builds the native game only; build the web version from byo-pop")
endif()

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)
# Every game and tool lands in the build folder itself (build/native/bejeweled.exe, build/web/rulestest.js)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR})
# A build shares the machine with the user and several sessions, and Ninja would run a job per core (20 here) at
# normal priority: at most 6 compiles and 2 links run at once (the user, 2026-11-01). A web link's wasm-opt has its own
# cap (cmake/ByopopWeb.cmake).
set_property(GLOBAL APPEND PROPERTY JOB_POOLS byopop_compile=6 byopop_link=2)
set(CMAKE_JOB_POOL_COMPILE byopop_compile)
set(CMAKE_JOB_POOL_LINK byopop_link)

if(EMSCRIPTEN)
  set(SDL2_LIBRARIES "")
  add_library(sdl2_port INTERFACE)
  target_compile_options(sdl2_port INTERFACE "-sUSE_SDL=2")
  target_link_options(sdl2_port INTERFACE "-sUSE_SDL=2")
  set(PORT_SDL sdl2_port)
else()
  if(NOT SDL2_DIR AND DEFINED ENV{SDL2_DIR})
    set(SDL2_DIR "$ENV{SDL2_DIR}")
  endif()
  find_package(SDL2 REQUIRED CONFIG)
  set(PORT_SDL SDL2::SDL2 SDL2::SDL2main)
endif()

add_subdirectory(third_party)
# Shared by every game: the portable SexyAppFramework (framework/) and the platform layer (platform/),
# built together as the sexyframework library (framework/CMakeLists.txt).
add_subdirectory(framework)

if(NOT EMSCRIPTEN)
  # Native builds run from the build folder: SDL2.dll and the llvm-mingw runtime DLLs go next to the exe
  get_filename_component(PORT_COMPILER_DIR "${CMAKE_CXX_COMPILER}" DIRECTORY)
  file(GLOB PORT_RUNTIME_DLLS "${PORT_COMPILER_DIR}/../i686-w64-mingw32/bin/*.dll")
endif()

option(BUILD_ROCKETMANIA "Build the rocketmania port" ON)
if(BUILD_ROCKETMANIA)
  add_subdirectory(games/rocketmania)
endif()
