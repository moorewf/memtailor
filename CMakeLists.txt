cmake_minimum_required(VERSION 3.12)

project(memtailor VERSION 1.0 LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

add_library(memtailor
  src/memtailor/Arena.cpp
  src/memtailor/BufferPool.cpp
  src/memtailor/MemoryBlocks.cpp
  src/memtailor/Arena.h
  src/memtailor/ArenaVector.h
  src/memtailor/BufferPool.h
  src/memtailor/MemoryBlocks.h
  )

target_include_directories(memtailor
  PUBLIC
    src
  )

install(TARGETS memtailor ARCHIVE DESTINATION lib)
install(FILES include/memtailor.h DESTINATION include)
install(DIRECTORY src/memtailor/
  DESTINATION include/memtailor
  FILES_MATCHING PATTERN "*.h"
  PATTERN stdinc.h EXCLUDE
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
  endif()

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

