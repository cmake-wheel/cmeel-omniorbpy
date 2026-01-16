cmake_minimum_required(VERSION 3.30)

set(NAME omniorbpy)
set(_NAME omniORBpy)
set(VERSION 4.3.4)
set(EXT tar.bz2)
set(SHA256 a709c3c77b9c6b08616e1c9e12a5a9b9d5ccc1f2dcf6f647f205018d77f819a7)

project("cmeel-${NAME}" VERSION "${VERSION}")

find_package(Python3 REQUIRED COMPONENTS Interpreter)

execute_process(COMMAND "${Python3_EXECUTABLE}" "-m" "cmeel" "cmake"
                OUTPUT_VARIABLE CMEEL_PREFIX)
string(STRIP "${CMEEL_PREFIX}" CMEEL_PREFIX)
message(STATUS "cmeel prefix: ${CMEEL_PREFIX}.")

if(APPLE)
  set(ORIGIN "@loader_path")
  set(BUILD "")
else()
  set(ORIGIN "\$\$ORIGIN")
  set(BUILD "--build=${CMAKE_HOST_SYSTEM_PROCESSOR}-unknown-linux-gnu")
endif()

include(ExternalProject)
ExternalProject_Add(
  ${NAME}
  URL "http://downloads.sourceforge.net/project/omniorb/${_NAME}/${_NAME}-${VERSION}/${_NAME}-${VERSION}.${EXT}"
  URL_HASH "SHA256=${SHA256}"
  DOWNLOAD_EXTRACT_TIMESTAMP ON
  BUILD_IN_SOURCE ON
  CONFIGURE_COMMAND
    ${CMAKE_COMMAND} "-E" "env" "LDFLAGS=-Wl,-rpath,'${ORIGIN}'" "--"
    "./configure" "--prefix=${CMAKE_INSTALL_PREFIX}" "${BUILD}"
    "--with-omniorb=${CMEEL_PREFIX}"
  BUILD_COMMAND "make"
  # TEST_COMMAND "make" "check"
  INSTALL_COMMAND "make" "install")

# dummy file for install target
install(FILES "README.md" DESTINATION "share/cmeel-${NAME}/")
