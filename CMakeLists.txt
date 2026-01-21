cmake_minimum_required(VERSION 3.16)
project(EGPythia6 LANGUAGES CXX)
include(GNUInstallDirs)

# Find required packages
find_package(ROOT REQUIRED)

# Option for custom Pythia6 library path
set(PYTHIA_LIB_PATH "" CACHE PATH "Custom path to Pythia6 library directory")

# Find Pythia6 library
if(PYTHIA_LIB_PATH)
  find_library(Pythia6_LIBRARY
    NAMES Pythia6
    PATHS ${PYTHIA_LIB_PATH}
    NO_DEFAULT_PATH
  )
else()
  find_library(Pythia6_LIBRARY
    NAMES Pythia6
  )
endif()

if(NOT Pythia6_LIBRARY)
  message(FATAL_ERROR "Pythia6 library not found")
endif()

# Create imported target for Pythia6
add_library(Pythia6 SHARED IMPORTED)
set_target_properties(Pythia6 PROPERTIES
  IMPORTED_LOCATION ${Pythia6_LIBRARY}
)

include_directories(
  ${ROOT_INCLUDE_DIRS}
  inc/
)

add_library(EGPythia6 SHARED
  src/TMCParticle.cxx
  src/TPythia6.cxx
  src/TPythia6Decayer.cxx)

set_target_properties(EGPythia6 PROPERTIES
    VERSION 6.33.01
    SOVERSION 6.33
)

root_generate_dictionary(
  G__EGPythia6
  TMCParticle.h
  TPythia6.h
  TPythia6Decayer.h
  LINKDEF inc/LinkDef.h
  MODULE EGPythia6
)

target_link_libraries(EGPythia6
  PRIVATE
    Pythia6
    ROOT::Core
    ROOT::EG
    ROOT::Graf
    ROOT::Physics
)

# Installation rules
install(TARGETS EGPythia6
  LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
  ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
  RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
)

install(DIRECTORY inc/
  DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
  FILES_MATCHING PATTERN "*.h"
  PATTERN "LinkDef.h" EXCLUDE
)

install(FILES ${CMAKE_CURRENT_BINARY_DIR}/libEGPythia6.rootmap
              ${CMAKE_CURRENT_BINARY_DIR}/libEGPythia6_rdict.pcm
  DESTINATION ${CMAKE_INSTALL_LIBDIR}
)
