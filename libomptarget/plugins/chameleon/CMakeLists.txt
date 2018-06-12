##===----------------------------------------------------------------------===##
#
# Build a plugin for Chameleon offloading.
#
##===----------------------------------------------------------------------===##
if (NOT(CMAKE_SYSTEM_PROCESSOR MATCHES "(x86_64)|(ppc64le)$" AND CMAKE_SYSTEM_NAME MATCHES "Linux"))
  libomptarget_say("Not building Chameleon offloading plugin: only support Chameleon in Linux x86_64 or ppc64le hosts.")
  return()
elseif (NOT LIBOMPTARGET_DEP_LIBELF_FOUND)
  libomptarget_say("Not building Chameleon offloading plugin: libelf dependency not found.")
  return()
#elseif(NOT LIBOMPTARGET_DEP_CHAMELEON_FOUND)
#  libomptarget_say("Not building Chameleon offloading plugin: Chameleon library not found in system.")
#  return()
endif()

libomptarget_say("Building Chameleon offloading plugin.")

#set(additional_libs "")
#set(additional_libs ${LIBOMPTARGET_DEP_CHAMELEON_LIBRARIES} ${additional_libs})

set(tmachine_name "chameleon")
set(tmachine_libname "chameleon")
set(tmachine_triple "chameleon-pc-linux-gnu")

# Define the suffix for the runtime messaging dumps.
add_definitions("-DTARGET_NAME=${tmachine_name}")
# Define macro with the ELF ID for this target.
add_definitions("-DTARGET_ELF_ID=62")

#if(LIBOMPTARGET_CMAKE_BUILD_TYPE MATCHES debug)   
#endif()

#include_directories(${LIBOMPTARGET_DEP_CHAMELEON_INCLUDE_DIRS})
include_directories(${LIBOMPTARGET_DEP_LIBFFI_INCLUDE_DIR})
include_directories(${LIBOMPTARGET_DEP_LIBELF_INCLUDE_DIRS})

add_library("omptarget.rtl.${tmachine_libname}" SHARED ${CMAKE_CURRENT_SOURCE_DIR}/src/rtl.cpp)

# Install plugin under the lib destination folder.
install(TARGETS "omptarget.rtl.${tmachine_libname}" LIBRARY DESTINATION lib${OPENMP_LIBDIR_SUFFIX})

target_link_libraries(
  "omptarget.rtl.${tmachine_libname}"
  ${LIBOMPTARGET_DEP_LIBFFI_LIBRARIES} 
  ${LIBOMPTARGET_DEP_LIBELF_LIBRARIES}
  dl
  "-Wl,--version-script=${CMAKE_CURRENT_SOURCE_DIR}/../exports")

# Report to the parent scope that we are building a plugin.
set(LIBOMPTARGET_SYSTEM_TARGETS "${LIBOMPTARGET_SYSTEM_TARGETS} ${tmachine_triple}" PARENT_SCOPE)
