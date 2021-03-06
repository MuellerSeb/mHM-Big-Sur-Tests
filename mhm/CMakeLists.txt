# mHM cmake script
cmake_minimum_required(VERSION 3.14)
# additional cmake-modules
list(APPEND CMAKE_MODULE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/cmake/cmake-modules)
# get version and date from files (version.txt and version_date.txt)
include(version)
get_version(MHM_VER MHM_VER_DEV MHM_DATE)
# create the project
project(mhm
  VERSION ${MHM_VER}
  DESCRIPTION "The mesoscale Hydrological Model"
  HOMEPAGE_URL "https://www.ufz.de/mhm"
  LANGUAGES Fortran
)
# add full version and date to pre-processor flags (qoutes need in before hand)
add_compile_definitions(MHMVERSION='${MHM_VER_DEV}' MHMDATE='${MHM_DATE}')
# Compile
add_subdirectory(./src)  # the lib folder containing mhm_lib
option(BUILD_MHM_DRIVER "Build mHM with executable driver." ON)
if(BUILD_MHM_DRIVER)
  add_executable(mhm ./src/mHM/mhm_driver.f90)
  target_link_libraries(mhm PUBLIC mhm_lib)
  if(CMAKE_Fortran_COMPILER_ID MATCHES "GNU" AND CMAKE_WITH_GPROF)
    target_link_options(mhm PUBLIC -pg)
  endif()
  # add install option
  install(TARGETS mhm DESTINATION bin)
endif()

# setup coverage with GNU
if(CMAKE_Fortran_COMPILER_ID MATCHES "GNU" AND CMAKE_WITH_COVERAGE)
  include(CodeCoverage)
  append_coverage_compiler_flags()
  SETUP_TARGET_FOR_COVERAGE_LCOV(
    NAME mhm_coverage_CI
    EXECUTABLE ../CI-scripts/run_cmake_coverage.sh
    DEPENDENCIES mhm mhm_lib
    EXCLUDE src/lib/*
    GENHTML_ARGS -t "mHM coverage" --html-prolog ../doc/html_files/cov_header.prolog
  )
endif()
# automatically enable testing (OFF by default)
option(BUILD_TESTING "Build with pfUnit tests." OFF)
include(CTest)
# add pfunit test folder
if(BUILD_TESTING)
  add_subdirectory(./src/tests)
endif()

# With this, paths are added to the INSTALL_RPATH, and via the second command also to the build.
if (CMAKE_BUILD_MODULE_SYSTEM_INDEPENDENT)
  set_target_properties(mhm
    PROPERTIES
    INSTALL_RPATH "${CMAKE_Fortran_IMPLICIT_LINK_DIRECTORIES}"
    BUILD_WITH_INSTALL_RPATH ON
  )
endif()
