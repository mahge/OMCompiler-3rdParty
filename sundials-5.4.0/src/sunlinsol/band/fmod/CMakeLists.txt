# ----------------------------------------------------------------------
# Programmer(s): Cody J. Balos @ LLNL
# ----------------------------------------------------------------------
# SUNDIALS Copyright Start
# Copyright (c) 2002-2020, Lawrence Livermore National Security
# and Southern Methodist University.
# All rights reserved.
#
# See the top-level LICENSE and NOTICE files for details.
#
# SPDX-License-Identifier: BSD-3-Clause
# SUNDIALS Copyright End
# ----------------------------------------------------------------------
# CMakeLists.txt file for the F2003 band SUNLinearSolver object library
# ----------------------------------------------------------------------

set(sunlinsolband_SOURCES fsunlinsol_band_mod.f90 fsunlinsol_band_mod.c)

if(SUNDIALS_BUILD_STATIC_LIBS)
  sundials_add_f2003_interface_library(sundials_fsunlinsolband_mod_static STATIC_OBJECT
    ${sunlinsolband_SOURCES}
  )
  add_dependencies(sundials_fsunlinsolband_mod_static_obj sundials_fgeneric_static_obj)
  target_link_libraries(sundials_fsunlinsolband_mod_static PUBLIC sundials_fsunmatrixband_mod_static)
  set_target_properties(sundials_fsunlinsolband_mod_static
    PROPERTIES OUTPUT_NAME sundials_fsunlinsolband_mod CLEAN_DIRECT_OUTPUT 1)
  install(TARGETS sundials_fsunlinsolband_mod_static DESTINATION ${CMAKE_INSTALL_LIBDIR})
endif(SUNDIALS_BUILD_STATIC_LIBS)

if(SUNDIALS_BUILD_SHARED_LIBS)
  sundials_add_f2003_interface_library(sundials_fsunlinsolband_mod_shared SHARED_OBJECT
    ${sunlinsolband_SOURCES}
  )
  add_dependencies(sundials_fsunlinsolband_mod_shared_obj sundials_fgeneric_shared_obj)
  target_link_libraries(sundials_fsunlinsolband_mod_shared PUBLIC sundials_fsunmatrixband_mod_shared)
  set_target_properties(sundials_fsunlinsolband_mod_shared
    PROPERTIES OUTPUT_NAME sundials_fsunlinsolband_mod CLEAN_DIRECT_OUTPUT 1)
  set_target_properties(sundials_fsunlinsolband_mod_shared
    PROPERTIES VERSION ${nveclib_VERSION} SOVERSION ${nveclib_SOVERSION})
  install(TARGETS sundials_fsunlinsolband_mod_shared DESTINATION ${CMAKE_INSTALL_LIBDIR})
endif(SUNDIALS_BUILD_SHARED_LIBS)

message(STATUS "Added SUNLINSOL_BAND F2003 interface")
