cmake_minimum_required(VERSION 3.8)
if(COMMAND cmake_policy)
  cmake_policy(SET CMP0003 NEW)
endif(COMMAND cmake_policy)

project(blah2)
include(CMakePrintHelpers)

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Werror")

find_package(Threads REQUIRED)
find_path(RAPIDJSON_INCLUDE_DIRS "rapidjson/allocators.h")
find_package(asio REQUIRED)
find_package(ryml CONFIG REQUIRED)
find_package(httplib CONFIG REQUIRED)
find_package(Armadillo CONFIG REQUIRED)
find_package(Catch2 CONFIG REQUIRED)

set(PROJECT_ROOT "${PROJECT_SOURCE_DIR}")
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${PROJECT_ROOT}/bin")

# include from top-level src dir
include_directories(src)

# SDRplay library setup (keep existing)
add_library(sdrplay /usr/local/include/sdrplay_api.h)
set_target_properties(sdrplay PROPERTIES LINKER_LANGUAGE C)
target_link_libraries(sdrplay PUBLIC /usr/local/lib/libsdrplay_api.so.3.15)

# Main executable - RSPduo only (remove unused capture sources)
add_executable(blah2
  src/blah2.cpp
  src/capture/Capture.cpp
  src/capture/Source.cpp
  src/capture/rspduo/RspDuo.cpp
  src/process/ambiguity/Ambiguity.cpp
  src/process/clutter/WienerHopf.cpp
  src/process/detection/CfarDetector1D.cpp
  src/process/detection/Centroid.cpp
  src/process/detection/Interpolate.cpp
  src/process/tracker/Tracker.cpp
  src/process/spectrum/SpectrumAnalyser.cpp
  src/process/meta/HammingNumber.cpp
  src/process/utility/Socket.cpp
  src/data/IqData.cpp
  src/data/Map.cpp
  src/data/Detection.cpp
  src/data/Track.cpp
  src/data/meta/Timing.cpp
)

target_link_libraries(blah2 PRIVATE 
  Threads::Threads
  asio::asio
  ryml::ryml
  httplib::httplib
  armadillo
  fftw3
  fftw3_threads
  sdrplay
)
target_include_directories(blah2 PRIVATE RAPIDJSON_INCLUDE_DIRS "rapidjson/allocators.h")

# Essential unit tests only (remove unused ones)
add_executable(testAmbiguity
  test/unit/process/ambiguity/TestAmbiguity.cpp
  src/data/IqData.cpp
  src/data/Map.cpp
  src/process/ambiguity/Ambiguity.cpp
  src/process/meta/HammingNumber.cpp
)
target_link_libraries(testAmbiguity PRIVATE 
  Catch2::Catch2WithMain 
  fftw3 
  fftw3_threads
)
