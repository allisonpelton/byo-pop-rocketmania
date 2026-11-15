cmake_minimum_required(VERSION 3.20)
# The game (games/rocketmania), the SexyAppFramework port it runs on (framework/, platform/) and the libraries they
# use (third_party/), for Windows. Generated from byo-pop's top-level CMakeLists.txt by its tools/publish.py.
project(byo-pop-rocketmania C CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)
# the exe and the DLLs it loads land in the build folder itself
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR})

# One exe that carries the C++ runtime it uses (libc++ and libunwind linked in, so SDL2.dll is the only DLL
# beside it), without the functions nothing calls and, in a Release build, without its symbol table. The code
# is compiled the same either way; only what is linked changes.
if(MINGW)
  add_compile_options(-ffunction-sections -fdata-sections)
  add_link_options(-static-libstdc++ -static-libgcc -Wl,--gc-sections $<$<CONFIG:Release>:-s>)
endif()

if(NOT SDL2_DIR AND DEFINED ENV{SDL2_DIR})
  set(SDL2_DIR "$ENV{SDL2_DIR}")
endif()
find_package(SDL2 REQUIRED CONFIG)
set(PORT_SDL SDL2::SDL2 SDL2::SDL2main)

add_subdirectory(third_party)

# Shared by every game: the portable SexyAppFramework (framework/) and the platform layer (platform/),
# built together as the sexyframework library (framework/CMakeLists.txt).
add_subdirectory(framework)

# Native builds run from the build folder: SDL2.dll and the llvm-mingw runtime DLLs go next to the exe
get_filename_component(PORT_COMPILER_DIR "${CMAKE_CXX_COMPILER}" DIRECTORY)
file(GLOB PORT_RUNTIME_DLLS "${PORT_COMPILER_DIR}/../i686-w64-mingw32/bin/*.dll")

add_subdirectory(games/rocketmania)
