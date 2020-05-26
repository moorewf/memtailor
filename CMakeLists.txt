cmake_minimum_required(VERSION 3.12)

set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

project(memtailor VERSION 1.0 LANGUAGES CXX)

find_package(Threads 2.1 REQUIRED QUIET)

add_compile_options(
  -DPACKAGE_NAME="${PROJECT_NAME}"
  -DPACKAGE_TARNAME="${PROJECT_NAME}"
  -DPACKAGE_VERSION="${PROJECT_VERSION}"
  -DPACKAGE_STRING="${PROJECT_NAME} ${PROJECT_VERSION}"
  -DPACKAGE_BUGREPORT=""
  -DPACKAGE_URL=""
  -DPACKAGE="${PROJECT_NAME}"
  -DVERSION="${PROJECT_VERSION}"
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
target_include_directories(memtailor PUBLIC
  $<BUILD_INTERFACE:${CMAKE_SOURCE_DIR}/src>
  $<INSTALL_INTERFACE:include>
  )

set_target_properties(memtailor PROPERTIES PUBLIC_HEADER src/memtailor.h)
install(TARGETS memtailor LIBRARY DESTINATION lib)
install(DIRECTORY src/memtailor
  DESTINATION include
  FILES_MATCHING PATTERN "*.h"
  )
install(FILES README.md license.txt
  DESTINATION licenses/memtailor
  )

include(CTest)
if(BUILD_TESTING)
  add_executable(memtailor-gtests
    src/test/ArenaTest.cpp
    src/test/BufferPoolTest.cpp
    src/test/MemoryBlocksTest.cpp
    src/test/testMain.cpp
    )

  ################################
  # add gtest testing ############
  ################################
  find_package(GTest)
  include(GoogleTest)
  gtest_discover_tests(memtailor-gtests)

  if(GTEST_FOUND)
    target_link_libraries(memtailor-gtests memtailor GTest::GTest GTest::Main)
    target_include_directories(memtailor-gtests PRIVATE ${GTEST_INCLUDE_DIR})
  else()
    include(FetchContent)
    FetchContent_Declare(googletest
      GIT_REPOSITORY https://github.com/google/googletest.git
      GIT_TAG        release-1.10.0
      )
    FetchContent_MakeAvailable(googletest)

    target_link_libraries(memtailor-gtests memtailor gtest)
    target_include_directories(memtailor-gtests PRIVATE
      ${googletest_SOURCE_DIR}/googletest/include
      ${googletest_SOURCE_DIR}/googletest/src
      ${googletest_SOURCE_DIR}/googletest
      )
  endif()
endif()
