# ---------------------------------------------------------------
# Programmer(s): Slaven Peles and Cody J. Balos @ LLNL
# ---------------------------------------------------------------
# SUNDIALS Copyright Start
# Copyright (c) 2002-2020, Lawrence Livermore National Security
# and Southern Methodist University.
# All rights reserved.
#
# See the top-level LICENSE and NOTICE files for details.
#
# SPDX-License-Identifier: BSD-3-Clause
# SUNDIALS Copyright End
# ---------------------------------------------------------------
# CMakeLists.txt file for IDA raja examples
# ---------------------------------------------------------------

# Example lists are tuples "name\;type" where the type is
# 'develop' for examples excluded from 'make test' in releases

# Examples using SUNDIALS linear solvers
set(IDA_examples
  "idaHeat2D_kry_p_mpiraja\;1\;4\;develop"
  )

if(MPI_C_COMPILER)
  # use MPI wrapper as the compiler
  set(CMAKE_C_COMPILER ${MPI_C_COMPILER})
  set(CMAKE_CXX_COMPILER ${MPI_CXX_COMPILER})
endif()

include_directories(${MPI_CXX_INCLUDE_PATH})

# Specify libraries to link against (through the target that was used to
# generate them) based on the value of the variable LINK_LIBRARY_TYPE
if(LINK_LIBRARY_TYPE MATCHES "static")
  set(IDA_LIB sundials_ida_static)
  set(NVECS_LIB sundials_nvecmpiplusx_static sundials_nveccudaraja_static)
else()
  set(IDA_LIB sundials_ida_shared)
  set(NVECS_LIB sundials_nvecmpiplusx_shared sundials_nveccudaraja_shared)
endif()

# Set-up linker flags and link libraries
set(SUNDIALS_LIBS ${IDA_LIB} ${NVECS_LIB} ${EXTRA_LINK_LIBS})


# Add the build and install targets for each IDA example
foreach(example_tuple ${IDA_examples})

  # parse the example tuple
  list(GET example_tuple 0 example)
  list(GET example_tuple 1 number_of_nodes)
  list(GET example_tuple 2 number_of_tasks)
  list(GET example_tuple 3 example_type)

  # add example source files
  add_executable(${example} ${example}.cu)

  set_target_properties(${example} PROPERTIES FOLDER "Examples")

  # add example to regression tests
  sundials_add_test(${example} ${example}
    MPI_NPROCS ${number_of_tasks}
    ANSWER_DIR ${CMAKE_CURRENT_SOURCE_DIR}
    ANSWER_FILE ${example}.out
    EXAMPLE_TYPE ${example_type})

  # libraries to link against
  target_link_libraries(${example} PRIVATE ${SUNDIALS_LIBS} ${MPI_CXX_LIBRARIES})

  # install example source and out files
  if(SUNDIALS_EXAMPLES_INSTALL)
    install(FILES ${example}.cu ${example}.out
      DESTINATION ${SUNDIALS_EXAMPLES_INSTALL_PATH}/ida/mpiraja)
  endif()

endforeach(example_tuple ${IDA_examples})


# create Makfile and CMakeLists.txt for examples
if(SUNDIALS_EXAMPLES_INSTALL)

  # Install the README file
  install(FILES README DESTINATION ${SUNDIALS_EXAMPLES_INSTALL_PATH}/ida/mpiraja)

  # Prepare substitution variables for Makefile and/or CMakeLists templates
  set(SOLVER "IDA")
  set(SOLVER_LIB "sundials_ida")

  examples2string(IDA_examples EXAMPLES)

  # Regardless of the platform we're on, we will generate and install
  # CMakeLists.txt file for building the examples. This file  can then
  # be used as a template for the user's own programs.

  # generate CMakelists.txt in the binary directory
  configure_file(
    ${PROJECT_SOURCE_DIR}/examples/templates/cmakelists_parallel_RAJA_ex.in
    ${PROJECT_BINARY_DIR}/examples/ida/mpiraja/CMakeLists.txt
    @ONLY
    )

  # install CMakelists.txt
  install(
    FILES ${PROJECT_BINARY_DIR}/examples/ida/mpiraja/CMakeLists.txt
    DESTINATION ${SUNDIALS_EXAMPLES_INSTALL_PATH}/ida/mpiraja
    )

  # On UNIX-type platforms, we also  generate and install a makefile for
  # building the examples. This makefile can then be used as a template
  # for the user's own programs.

  if(UNIX)
    # generate Makefile and place it in the binary dir
    configure_file(
      ${PROJECT_SOURCE_DIR}/examples/templates/makefile_parallel_RAJA_ex.in
      ${PROJECT_BINARY_DIR}/examples/ida/mpiraja/Makefile_ex
      @ONLY
      )
    # install the configured Makefile_ex as Makefile
    install(
      FILES ${PROJECT_BINARY_DIR}/examples/ida/mpiraja/Makefile_ex
      DESTINATION ${SUNDIALS_EXAMPLES_INSTALL_PATH}/ida/mpiraja
      RENAME Makefile
      )
  endif(UNIX)

  # add test_install target
  sundials_add_test_install(ida mpiraja)

endif(SUNDIALS_EXAMPLES_INSTALL)
