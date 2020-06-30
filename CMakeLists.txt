# Try to find boost library
# if cannot find boost, download the percific version

cmake_minimum_required(VERSION 3.14.0)

project(BOOST_LIBRARY)

# variables
# custom definitions
set(BOOST_MAJOR_VERSION 1 CACHE INTERNAL "Boost major version number, default is 1")
set(BOOST_MINOR_VERSION 73 CACHE INTERNAL "Boost minor version number, default is 73")
set(BOOST_PATCH_VERSION 0 CACHE INTERNAL "Boost patch version number, default is 0")

option(DOWNLOAD_BOOST "Download and compile boost" OFF)
option(TEST_FLAG "Test this cmake file" OFF)

# set(BOOST_COMPONENTS "system filesystem" CACHE STRING "Required boost components, default is \"system filesystem\"")
# boost temporary directories
set(BOOST_INSTALL_DIR ${CMAKE_BINARY_DIR}/boost_binary CACHE PATH "Boost install directory")
set(BOOST_DOWNLOAD_DIR ${CMAKE_BINARY_DIR}/boost_download CACHE PATH "Boost download directory")

message(STATUS "Try to find boost, version number ${BOOST_MAJOR_VERSION}.${BOOST_MINOR_VERSION}.${BOOST_PATCH_VERSION}")

macro (TRY_FIND_BOOST_USING_FIND_PACKAGE)
  set(USE_BOOST_DIR false)
  set(USE_BOOST_ROOT false)

  # check global variable
  if ((NOT DEFINED Boost_DIR) AND (NOT DEFINED BOOST_ROOT))
    message(STATUS "First, check \"Boost_DIR\" variables. If you already have a boost, please set the Boost_DIR variable to boost cmake directory")
    message(STATUS "For example, ~/boost_binary/lib/cmake/Boost-1.72.0")
  elseif (DEFINED Boost_DIR)
    message(STATUS "Your Boost_DIR is set to ${Boost_DIR}, try to find boost packages")
    set(USE_BOOST_DIR true)
  elseif (DEFINED BOOST_ROOT)
    message(STATUS "Your BOOST_ROOT is set to ${BOOST_ROOT}, try to find boost packages")
    set(USE_BOOST_ROOT true)
  endif()

  # try to find packages
  find_package(Boost ${BOOST_MAJOR_VERSION}.${BOOST_MINOR_VERSION}.${BOOST_PATCH_VERSION}) # COMPONENTS ${BOOST_COMPONENTS_LIST})

  if (NOT Boost_FOUND)
    if (${USE_BOOST_ROOT})
      message(STATUS "Please check your Boost_ROOT definition, Example: ~/boost_binary/lib/cmake/Boost-1.72.0")
    elseif (${USE_BOOST_DIR})
      message(STATUS "Please check your Boost_DIR definition, Example: ~/boost_binary/lib/cmake/Boost-1.72.0")
    else ()
      message(STATUS "Cannot find boost package for ${BOOST_MAJOR_VERSION}.${BOOST_MINOR_VERSION}.${BOOST_PATCH_VERSION} using find_package")
    endif()
  endif()
endmacro()


macro (DOWNLOAD_AND_COMPILE_BOOST)
  if (NOT DOWNLOAD_BOOST)
    message(STATUS "Cannot find boost library, If you want to download boost and compile it, set -DDOWNLOAD_BOOST:BOOL=ON, and run cmake again^_^")
    return ()
  endif()

  # if boost has already built
  find_package(Boost ${BOOST_MAJOR_VERSION}.${BOOST_MINOR_VERSION}.${BOOST_PATCH_VERSION} PATHS ${BOOST_INSTALL_DIR}/lib/cmake/Boost-${BOOST_MAJOR_VERSION}.${BOOST_MINOR_VERSION}.${BOOST_PATCH_VERSION} NO_DEFAULT_PATH)
  if (Boost_FOUND)
    message(STATUS "Boost has been built. Use the built version at ${BOOST_INSTALL_DIR}/lib/cmake/Boost-${BOOST_MAJOR_VERSION}.${BOOST_MINOR_VERSION}.${BOOST_PATCH_VERSION}")
    return ()
  endif()

  message(STATUS "We will download boost to ${BOOST_DOWNLOAD_DIR} and compile in this directory")
  message(STATUS "We will install boost to ${BOOST_INSTALL_DIR}")

  if (WIN32)
      set(BOOST_CONFIGURE_COMMAND "bootstrap.bat")
  else ()
      set(BOOST_CONFIGURE_COMMAND "sh bootstrap.sh")
  endif()

  include(ExternalProject)
  ExternalProject_Add(
      Boost
      URL https://downloads.sourceforge.net/project/boost/boost/${BOOST_MAJOR_VERSION}.${BOOST_MINOR_VERSION}.${BOOST_PATCH_VERSION}/boost_${BOOST_MAJOR_VERSION}_${BOOST_MINOR_VERSION}_${BOOST_PATCH_VERSION}.zip
      DOWNLOAD_DIR ${BOOST_DOWNLOAD_DIR}
      UPDATE_COMMAND ""
      CONFIGURE_COMMAND ${BOOST_CONFIGURE_COMMAND}
      BUILD_COMMAND b2 --prefix=${BOOST_INSTALL_DIR} --build-type=complete -d+2 install
      BUILD_IN_SOURCE true
      INSTALL_COMMAND ""
      INSTALL_DIR ${BOOST_INSTALL_DIR}
  )

  find_package(Boost ${BOOST_MAJOR_VERSION}.${BOOST_MINOR_VERSION}.${BOOST_PATCH_VERSION}) # COMPONENTS ${BOOST_COMPONENTS_LIST})
endmacro() 

if (NOT Boost_FOUND)
  TRY_FIND_BOOST_USING_FIND_PACKAGE() 
endif()

if (NOT Boost_FOUND)
  DOWNLOAD_AND_COMPILE_BOOST()
endif()


# ------------------------------------ TEST START ---------------------------------
if (${TEST_FLAG})

  message(STATUS "Start configure TEST CODE!")

  # write a test cpp file
  set(CPP_CODE "#include <iostream>

int main()
{
  std::cout << \"Hello, world!\" << std::endl\;
  return 0\;
}
")

  FILE(WRITE main.cpp ${CPP_CODE})

  add_executable(${PROJECT_NAME} main.cpp)
endif()
# ------------------------------------ TEST END ---------------------------------