cmake_minimum_required(VERSION 3.12)

set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

project(memtailor VERSION 1.0 LANGUAGES CXX)

find_package(Threads 2.1 REQUIRED QUIET)




include(CheckIncludeFiles)
check_include_files("stdlib.h;stdarg.h;string.h;float.h" STDC_HEADERS)
check_include_files(sys/types.h	HAVE_SYS_TYPES_H)
check_include_files(sys/stat.h	HAVE_SYS_STAT_H)
check_include_files(stdlib.h	HAVE_STDLIB_H)
check_include_files(string.h	HAVE_STRING_H)
check_include_files(memory.h	HAVE_MEMORY_H)
check_include_files(strings.h	HAVE_STRINGS_H)
check_include_files(inttypes.h	HAVE_INTTYPES_H)
check_include_files(stdint.h	HAVE_STDINT_H)
check_include_files(unistd.h	HAVE_UNISTD_H)
check_include_files(dlfcn.h	HAVE_DLFCN_H)

# set_property(TARGET memtailor PROPERTY CXX_STANDARD 11)
add_compile_options(
  -g -O2
  -DPACKAGE_NAME="${PROJECT_NAME}"
  -DPACKAGE_TARNAME="${PROJECT_NAME}"
  -DPACKAGE_VERSION="${PROJECT_VERSION}"
  -DPACKAGE_STRING="${PROJECT_NAME} ${PROJECT_VERSION}"
  -DPACKAGE_BUGREPORT=""
  -DPACKAGE_URL=""
  -DPACKAGE="${PROJECT_NAME}"
  -DVERSION="${PROJECT_VERSION}"
  # TODO
  -DLT_OBJDIR=".libs/"
)

add_library(memtailor STATIC
  src/memtailor.h
  src/memtailor.cpp
  src/memtailor/Arena.cpp
  src/memtailor/BufferPool.cpp
  src/memtailor/MemoryBlocks.cpp
  src/memtailor/Arena.h
  src/memtailor/ArenaVector.h
  src/memtailor/BufferPool.h
  src/memtailor/MemoryBlocks.h
  src/memtailor/stdinc.h
  )

target_link_libraries(memtailor Threads::Threads)

target_include_directories(memtailor
  PUBLIC
    src
  )

install(TARGETS memtailor ARCHIVE DESTINATION lib)
install(FILES src/memtailor.h DESTINATION include)
install(DIRECTORY src/memtailor/
  DESTINATION include/memtailor
  FILES_MATCHING PATTERN "*.h"
#  PATTERN stdinc.h EXCLUDE
  )
install(FILES README.md license.txt
  DESTINATION licenses/memtailor
  )


option(PACKAGE_TESTS "Build the tests" ON)
if(PACKAGE_TESTS)
  enable_testing()
  add_executable(memtailor-gtests
    src/test/ArenaTest.cpp
    src/test/BufferPoolTest.cpp
    src/test/MemoryBlocksTest.cpp
    src/test/gtestInclude.cpp
    src/test/testMain.cpp
    )
  target_link_libraries(memtailor-gtests memtailor)

################################
# add gtest testing ############
################################

include(FetchContent)

FetchContent_Declare(
  googletest
  GIT_REPOSITORY https://github.com/google/googletest.git
  GIT_TAG        release-1.8.0
)

FetchContent_GetProperties(googletest)
if(NOT googletest_POPULATED)
  FetchContent_Populate(googletest)
endif()

target_include_directories(memtailor-gtests
  PRIVATE
  ${googletest_SOURCE_DIR}/googletest/include
  ${googletest_SOURCE_DIR}/googletest/src
  ${googletest_SOURCE_DIR}/googletest
  )

add_test(NAME memtailor-gtests
      COMMAND memtailor-gtests
  )
endif()
