###################################################################################################
#                                                                                                 #
# This file is part of BLASFEO.                                                                   #
#                                                                                                 #
# BLASFEO -- BLAS For Embedded Optimization.                                                      #
# Copyright (C) 2019 by Gianluca Frison.                                                          #
# Developed at IMTEK (University of Freiburg) under the supervision of Moritz Diehl.              #
# All rights reserved.                                                                            #
#                                                                                                 #
# The 2-Clause BSD License                                                                        #
#                                                                                                 #
# Redistribution and use in source and binary forms, with or without                              #
# modification, are permitted provided that the following conditions are met:                     #
#                                                                                                 #
# 1. Redistributions of source code must retain the above copyright notice, this                  #
#    list of conditions and the following disclaimer.                                             #
# 2. Redistributions in binary form must reproduce the above copyright notice,                    #
#    this list of conditions and the following disclaimer in the documentation                    #
#    and/or other materials provided with the distribution.                                       #
#                                                                                                 #
# THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS" AND                 #
# ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED                   #
# WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE                          #
# DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT OWNER OR CONTRIBUTORS BE LIABLE FOR                 #
# ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES                  #
# (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES;                    #
# LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND                     #
# ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT                      #
# (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS                   #
# SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.                                    #
#                                                                                                 #
# Author: Gianluca Frison, gianluca.frison (at) imtek.uni-freiburg.de                             #
#                                                                                                 #
###################################################################################################

cmake_minimum_required(VERSION 2.8.11)

cmake_policy(SET CMP0048 NEW)
project(blasfeo LANGUAGES C ASM VERSION 0.1)


# Target architecture
# set(TARGET X64_AUTOMATIC CACHE STRING "Target architecture")
# set(TARGET X64_INTEL_HASWELL CACHE STRING "Target architecture")
# set(TARGET X64_INTEL_SANDY_BRIDGE CACHE STRING "Target architecture")
# set(TARGET X64_INTEL_CORE CACHE STRING "Target architecture")
# set(TARGET X64_AMD_BULLDOZER CACHE STRING "Target architecture")
# set(TARGET ARMV8A_ARM_CORTEX_A57 CACHE STRING "Target architecture")
# set(TARGET ARMV8A_ARM_CORTEX_A53 CACHE STRING "Target architecture")
# set(TARGET ARMV7A_ARM_CORTEX_A15 CACHE STRING "Target architecture")
# set(TARGET ARMV7A_ARM_CORTEX_A9 CACHE STRING "Target architecture")
# set(TARGET ARMV7A_ARM_CORTEX_A7 CACHE STRING "Target architecture")
set(TARGET GENERIC)

# Linear Algebra backend
set(LA HIGH_PERFORMANCE CACHE STRING "Linear algebra backend")
# set(LA REFERENCE CACHE STRING "Linear algebra backend")
# set(LA BLAS CACHE STRING "Linear algebra backend")


# BLAS and LAPACK version (for LA=BLAS in BLASFEO or for BLASFEO_BENCHMARS=ON)
set(EXTERNAL_BLAS 0 CACHE STRING "Reference blas to use")
# set(EXTERNAL_BLAS OPENBLAS CACHE STRING "Reference blas to use")
# set(EXTERNAL_BLAS NETLIB CACHE STRING "Reference blas to use")
# set(EXTERNAL_BLAS MKL CACHE STRING "Reference blas to use")
# set(EXTERNAL_BLAS BLIS CACHE STRING "Reference blas to use")
# set(EXTERNAL_BLAS ATLAS CACHE STRING "Reference blas to use")


# Stack buffer size (malloc for larger)
set(K_MAX_STACK 300 CACHE STRING "Maximum k value using stack memory")

# Options
# set(BLASFEO_TESTING ON CACHE BOOL "Tests enabled")
set(BLASFEO_TESTING OFF CACHE BOOL "Tests disabled")

# set(BLASFEO_BENCHMARKS ON CACHE BOOL "Benchmarks enabled")
set(BLASFEO_BENCHMARKS OFF CACHE BOOL "Benchmarks disabled")

set(BLASFEO_EXAMPLES ON CACHE BOOL "Examples enabled")
# set(BLASFEO_EXAMPLES OFF CACHE BOOL "Examples disabled")

# Use C99 extension to math library
set(USE_C99_MATH ON CACHE BOOL "Use C99 extension to math library")

# Compile auxiliary functions with external dependencies
# (for memory allocation and printing)
set(EXT_DEP ON CACHE BOOL "Compile external dependencies in BLASFEO")

# Skip BLASFEO utils for unsupported systems and for Realease build
if(CMAKE_SYSTEM MATCHES "dSpace" OR CMAKE_BUILD_TYPE MATCHES "Release")
	set(BLASFEO_TESTING OFF CACHE BOOL "Examples disabled" FORCE)
	set(BLASFEO_BENCHMARKS OFF CACHE BOOL "Benchmarks disabled" FORCE)
	set(BLASFEO_EXAMPLES OFF CACHE BOOL "Examples disabled" FORCE)
endif()

# C Compiler
if(CMAKE_C_COMPILER_ID MATCHES MSVC)
	if(NOT ${TARGET} MATCHES GENERIC)
		message( FATAL_ERROR "MSVC compiler only supported for TARGET=GENERIC")
	endif()
endif()

# Macro level (code size vs performance in assembly kernels): 0 (no macro),
# 1 (all macro but gemm kernel), 2 (all macro)
set(MACRO_LEVEL 0)

# enable runtine checks
set(RUNTIME_CHECKS OFF)

# testing
if(BLASFEO_TESTING MATCHES ON)
	set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -DTESTING_MODE=1")
endif()

if(BLASFEO_BENCHMARKS MATCHES ON)
	set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -DBENCHMARKS_MODE=1")
endif()

# Specify the linear algebra package to use
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -DLA_${LA}")


set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -DK_MAX_STACK=${K_MAX_STACK}")

#
if(${RUNTIME_CHECKS})
	set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -DDIM_CHECK")
endif()

#
if(${USE_C99_MATH})
	set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -DUSE_C99_MATH")
endif()

#
if(${EXT_DEP})
	set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -DEXT_DEP")
endif()

#
if(${MACRO_LEVEL} MATCHES 1)
	set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -DMACRO_LEVEL=1")
endif()
if(${MACRO_LEVEL} MATCHES 2)
	set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -DMACRO_LEVEL=2")
endif()

# Detect the host system
if(${CMAKE_HOST_SYSTEM_NAME} MATCHES "Linux")
	set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -DOS_LINUX")
	set(CMAKE_ASM_FLAGS "${CMAKE_ASM_FLAGS} -DOS_LINUX")
endif()
if(${CMAKE_HOST_SYSTEM_NAME} MATCHES "Darwin")
	set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -DOS_MAC")
	set(CMAKE_ASM_FLAGS "${CMAKE_ASM_FLAGS} -DOS_MAC")
endif()
if(${CMAKE_HOST_SYSTEM_NAME} MATCHES "Windows")
	set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -DOS_WINDOWS")
	set(CMAKE_ASM_FLAGS "${CMAKE_ASM_FLAGS} -DOS_WINDOWS")
endif()

# External BLAS flags
set(C_FLAGS_BLAS_0        "-DEXTERNAL_BLAS_NONE")
set(C_FLAGS_BLAS_OPENBLAS "-DEXTERNAL_BLAS_OPENBLAS")
set(C_FLAGS_BLAS_BLIS     "-DEXTERNAL_BLAS_BLIS")
set(C_FLAGS_BLAS_NETLIB   "-DEXTERNAL_BLAS_NETLIB")
set(C_FLAGS_BLAS_MKL      "-DEXTERNAL_BLAS_MKL -m64")
set(C_FLAGS_BLAS_ATLAS    "-DEXTERNAL_BLAS_ATLAS")

# message(STATUS "Using external BLAS: ${EXTERNAL_BLAS}")
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${C_FLAGS_BLAS_${EXTERNAL_BLAS}}")


# common C flags
if(CMAKE_C_COMPILER_ID MATCHES "GNU" OR CMAKE_C_COMPILER_ID MATCHES "Clang")
	set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -fPIC")
endif()


# architecture-specific C flags
set(C_FLAGS_TARGET_X64_INTEL_HASWELL      "-m64 -mavx -mavx2 -mfma")
set(C_FLAGS_TARGET_X64_INTEL_SANDY_BRIDGE "-m64 -mavx")
set(C_FLAGS_TARGET_X64_INTEL_CORE         "-m64 -msse3")
set(C_FLAGS_TARGET_X64_AMD_BULLDOZER      "-m64 -mavx -mfma")
set(C_FLAGS_TARGET_ARMV8A_ARM_CORTEX_A57  "-march=armv8-a+crc+crypto+simd -mcpu=cortex-a57")
set(C_FLAGS_TARGET_ARMV8A_ARM_CORTEX_A53  "-march=armv8-a+crc+crypto+simd -mcpu=cortex-a53")
set(C_FLAGS_TARGET_ARMV7A_ARM_CORTEX_A15  "-marm -mfloat-abi=hard -mfpu=neon-vfpv4 -mcpu=cortex-a15")
set(C_FLAGS_TARGET_ARMV7A_ARM_CORTEX_A9   "-marm -mfloat-abi=hard -mfpu=neon -mcpu=cortex-a9")
set(C_FLAGS_TARGET_ARMV7A_ARM_CORTEX_A7   "-marm -mfloat-abi=hard -mfpu=neon-vfpv4 -mcpu=cortex-a7")
set(C_FLAGS_TARGET_GENERIC                "")

# architecture-specific assembly flags
set(ASM_FLAGS_TARGET_X64_INTEL_HASWELL      "")
set(ASM_FLAGS_TARGET_X64_INTEL_SANDY_BRIDGE "")
set(ASM_FLAGS_TARGET_X64_INTEL_CORE         "")
set(ASM_FLAGS_TARGET_X64_AMD_BULLDOZER      "")
set(ASM_FLAGS_TARGET_ARMV8A_ARM_CORTEX_A57  "")
set(ASM_FLAGS_TARGET_ARMV8A_ARM_CORTEX_A53  "")
set(ASM_FLAGS_TARGET_ARMV7A_ARM_CORTEX_A15  "-mfpu=neon-vfpv4 -mcpu=cortex-a15")
set(ASM_FLAGS_TARGET_ARMV7A_ARM_CORTEX_A9   "-mfpu=neon -mcpu=cortex-a9")
set(ASM_FLAGS_TARGET_ARMV7A_ARM_CORTEX_A7   "-mfpu=neon-vfpv4 -mcpu=cortex-a7")
set(ASM_FLAGS_TARGET_GENERIC                "")


# if(${TARGET} MATCHES X64_AUTOMATIC)
# 	include(${PROJECT_SOURCE_DIR}/cmake/X64AutomaticTargetSelection.cmake)
# 	X64AutomaticTargetSelection()
# 	# message(STATUS "Detected target ${TARGET}")
# else()
# 	include(${PROJECT_SOURCE_DIR}/cmake/TestSingleTarget.cmake)
# 	TestSingleTarget()
# 	# message(STATUS "Compiling for target: ${TARGET}")
# endif()



if(${TARGET} MATCHES X64_INTEL_HASWELL)
  set(TARGET_NEED_FEATURE_AVX2 1)
  set(TARGET_NEED_FEATURE_FMA  1)
endif()

if(${TARGET} MATCHES X64_INTEL_SANDY_BRIDGE)
  set(TARGET_NEED_FEATURE_AVX 1)
endif()

if(${TARGET} MATCHES X64_INTEL_CORE)
  set(TARGET_NEED_FEATURE_SSE3 1)
endif()

if(${TARGET} MATCHES X64_AMD_BULLDOZER)
  set(TARGET_NEED_FEATURE_AVX 1)
  set(TARGET_NEED_FEATURE_FMA 1)
endif()

if(${TARGET} MATCHES ARMV8A_ARM_CORTEX_A57)
  set(TARGET_NEED_FEATURE_VFPv4 1)
  set(TARGET_NEED_FEATURE_NEONv2 1)
endif()

if(${TARGET} MATCHES ARMV8A_ARM_CORTEX_A53)
  set(TARGET_NEED_FEATURE_VFPv4 1)
  set(TARGET_NEED_FEATURE_NEONv2 1)
endif()

if(${TARGET} MATCHES ARMV7A_ARM_CORTEX_A15)
  set(TARGET_NEED_FEATURE_VFPv3 1)
  set(TARGET_NEED_FEATURE_NEON 1)
endif()

if(${TARGET} MATCHES ARMV7A_ARM_CORTEX_A7)
  set(TARGET_NEED_FEATURE_VFPv3 1)
  set(TARGET_NEED_FEATURE_NEON 1)
endif()

if(${TARGET} MATCHES ARMV7A_ARM_CORTEX_A9)
  set(TARGET_NEED_FEATURE_VFPv3 1)
  set(TARGET_NEED_FEATURE_NEON 1)
endif()


# Create the target.h file with the proper target definition
configure_file(${PROJECT_SOURCE_DIR}/blasfeo_target.h.in
	${CMAKE_CURRENT_SOURCE_DIR}/include/blasfeo_target.h @ONLY)


# Append the appropriate flags based on the architecture and compiler
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -DTARGET_${TARGET}")
set(CMAKE_ASM_FLAGS "${CMAKE_ASM_FLAGS} -DTARGET_${TARGET}")

if(CMAKE_C_COMPILER_ID MATCHES "GNU" OR CMAKE_C_COMPILER_ID MATCHES "Clang")
	set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${C_FLAGS_TARGET_${TARGET}}")
	set(CMAKE_ASM_FLAGS "${CMAKE_ASM_FLAGS} ${ASM_FLAGS_TARGET_${TARGET}}")
endif()


# source files

file(GLOB CMN_SRC
	${PROJECT_SOURCE_DIR}/auxiliary/blasfeo_processor_features.c
	${PROJECT_SOURCE_DIR}/auxiliary/blasfeo_stdlib.c
	)

if(${LA} MATCHES HIGH_PERFORMANCE)

	if(${TARGET} MATCHES X64_INTEL_HASWELL)

		file(GLOB AUX_SRC
			${PROJECT_SOURCE_DIR}/auxiliary/d_aux_lib4.c
			${PROJECT_SOURCE_DIR}/auxiliary/s_aux_lib8.c
			${PROJECT_SOURCE_DIR}/auxiliary/m_aux_lib48.c
			)

		file(GLOB KERNEL_SRC
			${PROJECT_SOURCE_DIR}/kernel/avx2/kernel_dgemm_12x4_lib4.S
			${PROJECT_SOURCE_DIR}/kernel/avx2/kernel_dgemm_8x8_lib4.S
			${PROJECT_SOURCE_DIR}/kernel/avx2/kernel_dgemm_8x4_lib4.S
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_dgemm_8x4_lib4.c
#			${PROJECT_SOURCE_DIR}/kernel/avx/kernel_dgemm_10xX_lib4.S
#			${PROJECT_SOURCE_DIR}/kernel/avx/kernel_dgemm_8x2_lib4.S
#			${PROJECT_SOURCE_DIR}/kernel/avx/kernel_dgemm_6xX_lib4.S
#			${PROJECT_SOURCE_DIR}/kernel/avx/kernel_dgemm_4x2_lib4.S
			${PROJECT_SOURCE_DIR}/kernel/avx2/kernel_dgemm_4x4_lib4.S
			${PROJECT_SOURCE_DIR}/kernel/avx/kernel_dgemm_diag_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/avx2/kernel_dgemv_8_lib4.S
			${PROJECT_SOURCE_DIR}/kernel/avx/kernel_dgemv_4_lib4.S
			${PROJECT_SOURCE_DIR}/kernel/avx2/kernel_dsymv_6_lib4.S
			${PROJECT_SOURCE_DIR}/kernel/avx2/kernel_dgetrf_pivot_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/avx/kernel_dgeqrf_4_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/avx2/kernel_dgebp_lib4.S
			${PROJECT_SOURCE_DIR}/kernel/avx2/kernel_dgelqf_4_lib4.S
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_dgemv_4_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/avx/kernel_dgecp_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/avx2/kernel_dgetr_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/avx/kernel_dpack_lib4.S

			${PROJECT_SOURCE_DIR}/kernel/avx2/kernel_sgemm_24x4_lib8.S
			${PROJECT_SOURCE_DIR}/kernel/avx2/kernel_sgemm_16x4_lib8.S
			${PROJECT_SOURCE_DIR}/kernel/avx2/kernel_sgemm_8x8_lib8.S
			${PROJECT_SOURCE_DIR}/kernel/avx2/kernel_sgemm_8x4_lib8.S
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_sgemm_8x4_lib8.c
			${PROJECT_SOURCE_DIR}/kernel/avx/kernel_sgemm_diag_lib8.c
			${PROJECT_SOURCE_DIR}/kernel/avx/kernel_sgemv_8_lib8.S
			${PROJECT_SOURCE_DIR}/kernel/avx/kernel_sgemv_4_lib8.S
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_ssymv_4_lib8.c
			${PROJECT_SOURCE_DIR}/kernel/avx/kernel_sgead_lib8.S
			${PROJECT_SOURCE_DIR}/kernel/avx/kernel_sgetr_lib8.S
			${PROJECT_SOURCE_DIR}/kernel/avx/kernel_sgecpsc_lib8.S

			${PROJECT_SOURCE_DIR}/kernel/kernel_align_x64.S
			)

		file(GLOB BLAS_SRC
			${PROJECT_SOURCE_DIR}/blasfeo_api/d_blas1_lib4.c
			${PROJECT_SOURCE_DIR}/blasfeo_api/d_blas2_lib4.c
			${PROJECT_SOURCE_DIR}/blasfeo_api/d_blas2_diag_lib.c
			${PROJECT_SOURCE_DIR}/blasfeo_api/d_blas3_lib4.c
			${PROJECT_SOURCE_DIR}/blasfeo_api/d_blas3_diag_lib4.c
			${PROJECT_SOURCE_DIR}/blasfeo_api/d_lapack_lib4.c

			${PROJECT_SOURCE_DIR}/blasfeo_api/s_blas1_lib8.c
			${PROJECT_SOURCE_DIR}/blasfeo_api/s_blas2_lib8.c
			${PROJECT_SOURCE_DIR}/blasfeo_api/s_blas2_diag_lib.c
			${PROJECT_SOURCE_DIR}/blasfeo_api/s_blas3_lib8.c
			${PROJECT_SOURCE_DIR}/blasfeo_api/s_blas3_diag_lib8.c
			${PROJECT_SOURCE_DIR}/blasfeo_api/s_lapack_lib8.c
			)

	endif(${TARGET} MATCHES X64_INTEL_HASWELL)

	if(${TARGET} MATCHES X64_INTEL_SANDY_BRIDGE)

		file(GLOB AUX_SRC
			${PROJECT_SOURCE_DIR}/auxiliary/d_aux_lib4.c
			${PROJECT_SOURCE_DIR}/auxiliary/s_aux_lib8.c
			${PROJECT_SOURCE_DIR}/auxiliary/m_aux_lib48.c
			)

		file(GLOB KERNEL_SRC
			${PROJECT_SOURCE_DIR}/kernel/avx/kernel_dgemm_12x4_lib4.S
#			${PROJECT_SOURCE_DIR}/kernel/avx/kernel_dgemm_10xX_lib4.S
			${PROJECT_SOURCE_DIR}/kernel/avx/kernel_dgemm_8x4_lib4.S
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_dgemm_8x4_lib4.c
#			${PROJECT_SOURCE_DIR}/kernel/avx/kernel_dgemm_8x2_lib4.S
#			${PROJECT_SOURCE_DIR}/kernel/avx/kernel_dgemm_6xX_lib4.S
			${PROJECT_SOURCE_DIR}/kernel/avx/kernel_dgemm_4x4_lib4.S
#			${PROJECT_SOURCE_DIR}/kernel/avx/kernel_dgemm_4x2_lib4.S
			${PROJECT_SOURCE_DIR}/kernel/avx/kernel_dgemm_diag_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/avx/kernel_dgemv_12_lib4.S
			${PROJECT_SOURCE_DIR}/kernel/avx/kernel_dgemv_8_lib4.S
			${PROJECT_SOURCE_DIR}/kernel/avx/kernel_dsymv_6_lib4.S
			${PROJECT_SOURCE_DIR}/kernel/avx/kernel_dgemv_4_lib4.S
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_dgemv_4_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/avx/kernel_dgetrf_pivot_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/avx/kernel_dgeqrf_4_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/avx/kernel_dgebp_lib4.S
			${PROJECT_SOURCE_DIR}/kernel/avx/kernel_dgelqf_4_lib4.S
			${PROJECT_SOURCE_DIR}/kernel/avx/kernel_dpack_lib4.S

			${PROJECT_SOURCE_DIR}/kernel/avx/kernel_sgemm_16x4_lib8.S
			${PROJECT_SOURCE_DIR}/kernel/avx/kernel_sgemm_8x8_lib8.S
			${PROJECT_SOURCE_DIR}/kernel/avx/kernel_sgemm_8x4_lib8.S
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_sgemm_8x4_lib8.c
			${PROJECT_SOURCE_DIR}/kernel/avx/kernel_sgemm_diag_lib8.c
			${PROJECT_SOURCE_DIR}/kernel/avx/kernel_sgemv_8_lib8.S
			${PROJECT_SOURCE_DIR}/kernel/avx/kernel_sgemv_4_lib8.S
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_ssymv_4_lib8.c
			${PROJECT_SOURCE_DIR}/kernel/avx/kernel_sgead_lib8.S
			${PROJECT_SOURCE_DIR}/kernel/avx/kernel_sgecpsc_lib8.S
			${PROJECT_SOURCE_DIR}/kernel/avx/kernel_sgetr_lib8.S
			${PROJECT_SOURCE_DIR}/kernel/avx/kernel_dgecp_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/avx/kernel_dgetr_lib4.c

			${PROJECT_SOURCE_DIR}/kernel/kernel_align_x64.S
			)

		file(GLOB BLAS_SRC
			${PROJECT_SOURCE_DIR}/blasfeo_api/d_blas1_lib4.c
			${PROJECT_SOURCE_DIR}/blasfeo_api/d_blas2_lib4.c
			${PROJECT_SOURCE_DIR}/blasfeo_api/d_blas2_diag_lib.c
			${PROJECT_SOURCE_DIR}/blasfeo_api/d_blas3_lib4.c
			${PROJECT_SOURCE_DIR}/blasfeo_api/d_blas3_diag_lib4.c
			${PROJECT_SOURCE_DIR}/blasfeo_api/d_lapack_lib4.c

			${PROJECT_SOURCE_DIR}/blasfeo_api/s_blas1_lib8.c
			${PROJECT_SOURCE_DIR}/blasfeo_api/s_blas2_lib8.c
			${PROJECT_SOURCE_DIR}/blasfeo_api/s_blas2_diag_lib.c
			${PROJECT_SOURCE_DIR}/blasfeo_api/s_blas3_lib8.c
			${PROJECT_SOURCE_DIR}/blasfeo_api/s_blas3_diag_lib8.c
			${PROJECT_SOURCE_DIR}/blasfeo_api/s_lapack_lib8.c
			)

	endif(${TARGET} MATCHES X64_INTEL_SANDY_BRIDGE)

	if(${TARGET} MATCHES X64_INTEL_CORE)

		file(GLOB AUX_SRC
			${PROJECT_SOURCE_DIR}/auxiliary/d_aux_lib4.c
			${PROJECT_SOURCE_DIR}/auxiliary/s_aux_lib4.c
			${PROJECT_SOURCE_DIR}/auxiliary/m_aux_lib44.c
			)

		file(GLOB KERNEL_SRC
			${PROJECT_SOURCE_DIR}/kernel/sse3/kernel_dgemm_4x4_lib4.S
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_dgemm_4x4_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_dgemm_diag_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_dgemv_4_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_dsymv_4_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_dgetrf_pivot_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_dgeqrf_4_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_dgecp_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_dgetr_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_dpack_lib4.c

			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_sgemm_4x4_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_sgemm_diag_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_sgemv_4_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_ssymv_4_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_sgetrf_pivot_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_sgecp_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_sgetr_lib4.c

			${PROJECT_SOURCE_DIR}/kernel/kernel_align_x64.S
			)

		file(GLOB BLAS_SRC
			${PROJECT_SOURCE_DIR}/blasfeo_api/d_blas1_lib4.c
			${PROJECT_SOURCE_DIR}/blasfeo_api/d_blas2_lib4.c
			${PROJECT_SOURCE_DIR}/blasfeo_api/d_blas2_diag_lib.c
			${PROJECT_SOURCE_DIR}/blasfeo_api/d_blas3_lib4.c
			${PROJECT_SOURCE_DIR}/blasfeo_api/d_blas3_diag_lib4.c
			${PROJECT_SOURCE_DIR}/blasfeo_api/d_lapack_lib4.c

			${PROJECT_SOURCE_DIR}/blasfeo_api/s_blas1_lib4.c
			${PROJECT_SOURCE_DIR}/blasfeo_api/s_blas2_lib4.c
			${PROJECT_SOURCE_DIR}/blasfeo_api/s_blas2_diag_lib.c
			${PROJECT_SOURCE_DIR}/blasfeo_api/s_blas3_lib4.c
			${PROJECT_SOURCE_DIR}/blasfeo_api/s_blas3_diag_lib4.c
			${PROJECT_SOURCE_DIR}/blasfeo_api/s_lapack_lib4.c
			)

	endif(${TARGET} MATCHES X64_INTEL_CORE)

	if(${TARGET} MATCHES X64_AMD_BULLDOZER)

		file(GLOB AUX_SRC
			${PROJECT_SOURCE_DIR}/auxiliary/d_aux_lib4.c
			${PROJECT_SOURCE_DIR}/auxiliary/s_aux_lib4.c
			${PROJECT_SOURCE_DIR}/auxiliary/m_aux_lib44.c
			)

		file(GLOB KERNEL_SRC
			${PROJECT_SOURCE_DIR}/kernel/fma/kernel_dgemm_4x4_lib4.S
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_dgemm_4x4_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_dgemm_diag_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_dgemv_4_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_dsymv_4_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_dgetrf_pivot_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_dgeqrf_4_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_dpack_lib4.c

			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_sgemm_4x4_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_sgemm_diag_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_sgemv_4_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_ssymv_4_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_sgetrf_pivot_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_sgecp_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_dgecp_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_dgetr_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_sgetr_lib4.c

			${PROJECT_SOURCE_DIR}/kernel/kernel_align_x64.S
			)

		file(GLOB BLAS_SRC
			${PROJECT_SOURCE_DIR}/blasfeo_api/d_blas1_lib4.c
			${PROJECT_SOURCE_DIR}/blasfeo_api/d_blas2_lib4.c
			${PROJECT_SOURCE_DIR}/blasfeo_api/d_blas2_diag_lib.c
			${PROJECT_SOURCE_DIR}/blasfeo_api/d_blas3_lib4.c
			${PROJECT_SOURCE_DIR}/blasfeo_api/d_blas3_diag_lib4.c
			${PROJECT_SOURCE_DIR}/blasfeo_api/d_lapack_lib4.c

			${PROJECT_SOURCE_DIR}/blasfeo_api/s_blas1_lib4.c
			${PROJECT_SOURCE_DIR}/blasfeo_api/s_blas2_lib4.c
			${PROJECT_SOURCE_DIR}/blasfeo_api/s_blas2_diag_lib.c
			${PROJECT_SOURCE_DIR}/blasfeo_api/s_blas3_lib4.c
			${PROJECT_SOURCE_DIR}/blasfeo_api/s_blas3_diag_lib4.c
			${PROJECT_SOURCE_DIR}/blasfeo_api/s_lapack_lib4.c
			)

	endif(${TARGET} MATCHES X64_AMD_BULLDOZER)

	if(${TARGET} MATCHES ARMV8A_ARM_CORTEX_A57)

		file(GLOB AUX_SRC
			${PROJECT_SOURCE_DIR}/auxiliary/d_aux_lib4.c
			${PROJECT_SOURCE_DIR}/auxiliary/s_aux_lib4.c
			${PROJECT_SOURCE_DIR}/auxiliary/m_aux_lib44.c
			)

		file(GLOB KERNEL_SRC
			${PROJECT_SOURCE_DIR}/kernel/armv8a/kernel_dgemm_8x4_lib4.S
			${PROJECT_SOURCE_DIR}/kernel/armv8a/kernel_dgemm_4x4_lib4.S
			${PROJECT_SOURCE_DIR}/kernel/armv8a/kernel_dpack_lib4.S

			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_dgemm_4x4_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_dgemm_diag_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_dgemv_4_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_dsymv_4_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_dgetrf_pivot_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_dgeqrf_4_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_dpack_lib4.c

			${PROJECT_SOURCE_DIR}/kernel/armv8a/kernel_sgemm_16x4_lib4.S
			${PROJECT_SOURCE_DIR}/kernel/armv8a/kernel_sgemm_12x4_lib4.S
			${PROJECT_SOURCE_DIR}/kernel/armv8a/kernel_sgemm_8x8_lib4.S
			${PROJECT_SOURCE_DIR}/kernel/armv8a/kernel_sgemm_8x4_lib4.S
			${PROJECT_SOURCE_DIR}/kernel/armv8a/kernel_sgemm_4x4_lib4.S
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_sgemm_4x4_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_sgemm_diag_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_sgemv_4_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_ssymv_4_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_sgetrf_pivot_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_sgecp_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_sgetr_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_dgecp_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_dgetr_lib4.c

			${PROJECT_SOURCE_DIR}/kernel/kernel_align_generic.c
			)

		file(GLOB BLAS_SRC
			${PROJECT_SOURCE_DIR}/blasfeo_api/d_blas1_lib4.c
			${PROJECT_SOURCE_DIR}/blasfeo_api/d_blas2_lib4.c
			${PROJECT_SOURCE_DIR}/blasfeo_api/d_blas2_diag_lib.c
			${PROJECT_SOURCE_DIR}/blasfeo_api/d_blas3_lib4.c
			${PROJECT_SOURCE_DIR}/blasfeo_api/d_blas3_diag_lib4.c
			${PROJECT_SOURCE_DIR}/blasfeo_api/d_lapack_lib4.c

			${PROJECT_SOURCE_DIR}/blasfeo_api/s_blas1_lib4.c
			${PROJECT_SOURCE_DIR}/blasfeo_api/s_blas2_lib4.c
			${PROJECT_SOURCE_DIR}/blasfeo_api/s_blas2_diag_lib.c
			${PROJECT_SOURCE_DIR}/blasfeo_api/s_blas3_lib4.c
			${PROJECT_SOURCE_DIR}/blasfeo_api/s_blas3_diag_lib4.c
			${PROJECT_SOURCE_DIR}/blasfeo_api/s_lapack_lib4.c
			)

	endif(${TARGET} MATCHES ARMV8A_ARM_CORTEX_A57)

	if(${TARGET} MATCHES ARMV8A_ARM_CORTEX_A53)

		file(GLOB AUX_SRC
			${PROJECT_SOURCE_DIR}/auxiliary/d_aux_lib4.c
			${PROJECT_SOURCE_DIR}/auxiliary/s_aux_lib4.c
			${PROJECT_SOURCE_DIR}/auxiliary/m_aux_lib44.c
			)

		file(GLOB KERNEL_SRC

			${PROJECT_SOURCE_DIR}/kernel/armv8a/kernel_dgemm_12x4_lib4.S
			${PROJECT_SOURCE_DIR}/kernel/armv8a/kernel_dgemm_8x4_lib4.S
			${PROJECT_SOURCE_DIR}/kernel/armv8a/kernel_dgemm_4x4_lib4.S
			${PROJECT_SOURCE_DIR}/kernel/armv8a/kernel_dpack_lib4.S


			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_dgemm_4x4_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_dgemm_diag_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_dgemv_4_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_dsymv_4_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_dgetrf_pivot_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_dgeqrf_4_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_dpack_lib4.c

			${PROJECT_SOURCE_DIR}/kernel/armv8a/kernel_sgemm_16x4_lib4.S
			${PROJECT_SOURCE_DIR}/kernel/armv8a/kernel_sgemm_12x4_lib4.S
			${PROJECT_SOURCE_DIR}/kernel/armv8a/kernel_sgemm_8x8_lib4.S
			${PROJECT_SOURCE_DIR}/kernel/armv8a/kernel_sgemm_8x4_lib4.S
			${PROJECT_SOURCE_DIR}/kernel/armv8a/kernel_sgemm_4x4_lib4.S
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_sgemm_4x4_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_sgemm_diag_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_sgemv_4_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_ssymv_4_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_sgetrf_pivot_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_sgecp_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_sgetr_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_dgecp_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_dgetr_lib4.c

			${PROJECT_SOURCE_DIR}/kernel/kernel_align_generic.c
			)

		file(GLOB BLAS_SRC
			${PROJECT_SOURCE_DIR}/blasfeo_api/d_blas1_lib4.c
			${PROJECT_SOURCE_DIR}/blasfeo_api/d_blas2_lib4.c
			${PROJECT_SOURCE_DIR}/blasfeo_api/d_blas2_diag_lib.c
			${PROJECT_SOURCE_DIR}/blasfeo_api/d_blas3_lib4.c
			${PROJECT_SOURCE_DIR}/blasfeo_api/d_blas3_diag_lib4.c
			${PROJECT_SOURCE_DIR}/blasfeo_api/d_lapack_lib4.c

			${PROJECT_SOURCE_DIR}/blasfeo_api/s_blas1_lib4.c
			${PROJECT_SOURCE_DIR}/blasfeo_api/s_blas2_lib4.c
			${PROJECT_SOURCE_DIR}/blasfeo_api/s_blas2_diag_lib.c
			${PROJECT_SOURCE_DIR}/blasfeo_api/s_blas3_lib4.c
			${PROJECT_SOURCE_DIR}/blasfeo_api/s_blas3_diag_lib4.c
			${PROJECT_SOURCE_DIR}/blasfeo_api/s_lapack_lib4.c
			)

	endif(${TARGET} MATCHES ARMV8A_ARM_CORTEX_A53)

	if(${TARGET} MATCHES ARMV7A_ARM_CORTEX_A15)

		file(GLOB AUX_SRC
			${PROJECT_SOURCE_DIR}/auxiliary/d_aux_lib4.c
			${PROJECT_SOURCE_DIR}/auxiliary/s_aux_lib4.c
			${PROJECT_SOURCE_DIR}/auxiliary/m_aux_lib44.c
			)

		file(GLOB KERNEL_SRC
			${PROJECT_SOURCE_DIR}/kernel/armv7a/kernel_dgemm_4x4_lib4.S
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_dgemm_4x4_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_dgemm_diag_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_dgemv_4_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_dsymv_4_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_dgetrf_pivot_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_dgeqrf_4_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_dpack_lib4.c

			${PROJECT_SOURCE_DIR}/kernel/armv7a/kernel_sgemm_12x4_lib4.S
			${PROJECT_SOURCE_DIR}/kernel/armv7a/kernel_sgemm_8x4_lib4.S
			${PROJECT_SOURCE_DIR}/kernel/armv7a/kernel_sgemm_4x4_lib4.S
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_sgemm_4x4_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_sgemm_diag_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_sgemv_4_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_ssymv_4_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_sgetrf_pivot_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_sgecp_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_sgetr_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_dgecp_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_dgetr_lib4.c

			${PROJECT_SOURCE_DIR}/kernel/kernel_align_generic.c
			)

		file(GLOB BLAS_SRC
			${PROJECT_SOURCE_DIR}/blasfeo_api/d_blas1_lib4.c
			${PROJECT_SOURCE_DIR}/blasfeo_api/d_blas2_lib4.c
			${PROJECT_SOURCE_DIR}/blasfeo_api/d_blas2_diag_lib.c
			${PROJECT_SOURCE_DIR}/blasfeo_api/d_blas3_lib4.c
			${PROJECT_SOURCE_DIR}/blasfeo_api/d_blas3_diag_lib4.c
			${PROJECT_SOURCE_DIR}/blasfeo_api/d_lapack_lib4.c

			${PROJECT_SOURCE_DIR}/blasfeo_api/s_blas1_lib4.c
			${PROJECT_SOURCE_DIR}/blasfeo_api/s_blas2_lib4.c
			${PROJECT_SOURCE_DIR}/blasfeo_api/s_blas2_diag_lib.c
			${PROJECT_SOURCE_DIR}/blasfeo_api/s_blas3_lib4.c
			${PROJECT_SOURCE_DIR}/blasfeo_api/s_blas3_diag_lib4.c
			${PROJECT_SOURCE_DIR}/blasfeo_api/s_lapack_lib4.c
			)

	endif(${TARGET} MATCHES ARMV7A_ARM_CORTEX_A15)

	if(${TARGET} MATCHES ARMV7A_ARM_CORTEX_A7)

		file(GLOB AUX_SRC
			${PROJECT_SOURCE_DIR}/auxiliary/d_aux_lib4.c
			${PROJECT_SOURCE_DIR}/auxiliary/s_aux_lib4.c
			${PROJECT_SOURCE_DIR}/auxiliary/m_aux_lib44.c
			)

		file(GLOB KERNEL_SRC
			${PROJECT_SOURCE_DIR}/kernel/armv7a/kernel_dgemm_4x4_lib4.S
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_dgemm_4x4_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_dgemm_diag_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_dgemv_4_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_dsymv_4_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_dgetrf_pivot_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_dgeqrf_4_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_dpack_lib4.c

			${PROJECT_SOURCE_DIR}/kernel/armv7a/kernel_sgemm_8x4_lib4.S
			${PROJECT_SOURCE_DIR}/kernel/armv7a/kernel_sgemm_4x4_lib4.S
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_sgemm_4x4_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_sgemm_diag_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_sgemv_4_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_ssymv_4_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_sgetrf_pivot_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_sgecp_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_sgetr_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_dgecp_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_dgetr_lib4.c

			${PROJECT_SOURCE_DIR}/kernel/kernel_align_generic.c
			)

		file(GLOB BLAS_SRC
			${PROJECT_SOURCE_DIR}/blasfeo_api/d_blas1_lib4.c
			${PROJECT_SOURCE_DIR}/blasfeo_api/d_blas2_lib4.c
			${PROJECT_SOURCE_DIR}/blasfeo_api/d_blas2_diag_lib.c
			${PROJECT_SOURCE_DIR}/blasfeo_api/d_blas3_lib4.c
			${PROJECT_SOURCE_DIR}/blasfeo_api/d_blas3_diag_lib4.c
			${PROJECT_SOURCE_DIR}/blasfeo_api/d_lapack_lib4.c

			${PROJECT_SOURCE_DIR}/blasfeo_api/s_blas1_lib4.c
			${PROJECT_SOURCE_DIR}/blasfeo_api/s_blas2_lib4.c
			${PROJECT_SOURCE_DIR}/blasfeo_api/s_blas2_diag_lib.c
			${PROJECT_SOURCE_DIR}/blasfeo_api/s_blas3_lib4.c
			${PROJECT_SOURCE_DIR}/blasfeo_api/s_blas3_diag_lib4.c
			${PROJECT_SOURCE_DIR}/blasfeo_api/s_lapack_lib4.c
			)

	endif(${TARGET} MATCHES ARMV7A_ARM_CORTEX_A7)


	if(${TARGET} MATCHES ARMV7A_ARM_CORTEX_A9)

		file(GLOB AUX_SRC
			${PROJECT_SOURCE_DIR}/auxiliary/d_aux_lib4.c
			${PROJECT_SOURCE_DIR}/auxiliary/s_aux_lib4.c
			${PROJECT_SOURCE_DIR}/auxiliary/m_aux_lib44.c
			)

		file(GLOB KERNEL_SRC
			${PROJECT_SOURCE_DIR}/kernel/armv7a/kernel_dgemm_4x4_lib4.S
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_dgemm_4x4_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_dgemm_diag_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_dgemv_4_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_dsymv_4_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_dgetrf_pivot_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_dgeqrf_4_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_dpack_lib4.c

			${PROJECT_SOURCE_DIR}/kernel/armv7a/kernel_sgemm_8x4_lib4.S
			${PROJECT_SOURCE_DIR}/kernel/armv7a/kernel_sgemm_4x4_lib4.S
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_sgemm_4x4_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_sgemm_diag_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_sgemv_4_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_ssymv_4_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_sgetrf_pivot_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_sgecp_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_sgetr_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_dgecp_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_dgetr_lib4.c

			${PROJECT_SOURCE_DIR}/kernel/kernel_align_generic.c
			)

		file(GLOB BLAS_SRC
			${PROJECT_SOURCE_DIR}/blasfeo_api/d_blas1_lib4.c
			${PROJECT_SOURCE_DIR}/blasfeo_api/d_blas2_lib4.c
			${PROJECT_SOURCE_DIR}/blasfeo_api/d_blas2_diag_lib.c
			${PROJECT_SOURCE_DIR}/blasfeo_api/d_blas3_lib4.c
			${PROJECT_SOURCE_DIR}/blasfeo_api/d_blas3_diag_lib4.c
			${PROJECT_SOURCE_DIR}/blasfeo_api/d_lapack_lib4.c

			${PROJECT_SOURCE_DIR}/blasfeo_api/s_blas1_lib4.c
			${PROJECT_SOURCE_DIR}/blasfeo_api/s_blas2_lib4.c
			${PROJECT_SOURCE_DIR}/blasfeo_api/s_blas2_diag_lib.c
			${PROJECT_SOURCE_DIR}/blasfeo_api/s_blas3_lib4.c
			${PROJECT_SOURCE_DIR}/blasfeo_api/s_blas3_diag_lib4.c
			${PROJECT_SOURCE_DIR}/blasfeo_api/s_lapack_lib4.c
			)

	endif(${TARGET} MATCHES ARMV7A_ARM_CORTEX_A9)

	if(${TARGET} MATCHES GENERIC)

		file(GLOB AUX_SRC
			${PROJECT_SOURCE_DIR}/auxiliary/d_aux_lib4.c
			${PROJECT_SOURCE_DIR}/auxiliary/s_aux_lib4.c
			${PROJECT_SOURCE_DIR}/auxiliary/m_aux_lib44.c
			)

		file(GLOB KERNEL_SRC
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_dgemm_4x4_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_dgemm_diag_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_dgemv_4_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_dsymv_4_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_dgetrf_pivot_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_dgeqrf_4_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_dgecp_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_dgetr_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_dpack_lib4.c

			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_sgemm_4x4_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_sgemm_diag_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_sgemv_4_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_ssymv_4_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_sgetrf_pivot_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_sgecp_lib4.c
			${PROJECT_SOURCE_DIR}/kernel/generic/kernel_sgetr_lib4.c

			${PROJECT_SOURCE_DIR}/kernel/kernel_align_generic.c
			)

		file(GLOB BLAS_SRC
			${PROJECT_SOURCE_DIR}/blasfeo_api/d_blas1_lib4.c
			${PROJECT_SOURCE_DIR}/blasfeo_api/d_blas2_lib4.c
			${PROJECT_SOURCE_DIR}/blasfeo_api/d_blas2_diag_lib.c
			${PROJECT_SOURCE_DIR}/blasfeo_api/d_blas3_lib4.c
			${PROJECT_SOURCE_DIR}/blasfeo_api/d_blas3_diag_lib4.c
			${PROJECT_SOURCE_DIR}/blasfeo_api/d_lapack_lib4.c

			${PROJECT_SOURCE_DIR}/blasfeo_api/s_blas1_lib4.c
			${PROJECT_SOURCE_DIR}/blasfeo_api/s_blas2_lib4.c
			${PROJECT_SOURCE_DIR}/blasfeo_api/s_blas2_diag_lib.c
			${PROJECT_SOURCE_DIR}/blasfeo_api/s_blas3_lib4.c
			${PROJECT_SOURCE_DIR}/blasfeo_api/s_blas3_diag_lib4.c
			${PROJECT_SOURCE_DIR}/blasfeo_api/s_lapack_lib4.c
			)

	endif(${TARGET} MATCHES GENERIC)

else(${LA} MATCHES HIGH_PERFORMANCE) # REFERENCE BLAS

	file(GLOB AUX_SRC
		${PROJECT_SOURCE_DIR}/auxiliary/d_aux_lib.c
		${PROJECT_SOURCE_DIR}/auxiliary/s_aux_lib.c
		${PROJECT_SOURCE_DIR}/auxiliary/m_aux_lib.c
		)

	file(GLOB BLAS_SRC
		${PROJECT_SOURCE_DIR}/blasfeo_api/d_blas1_lib.c
		${PROJECT_SOURCE_DIR}/blasfeo_api/d_blas2_lib.c
		${PROJECT_SOURCE_DIR}/blasfeo_api/d_blas2_diag_lib.c
		${PROJECT_SOURCE_DIR}/blasfeo_api/d_blas3_lib.c
		${PROJECT_SOURCE_DIR}/blasfeo_api/d_blas3_diag_lib.c
		${PROJECT_SOURCE_DIR}/blasfeo_api/d_lapack_lib.c

		${PROJECT_SOURCE_DIR}/blasfeo_api/s_blas1_lib.c
		${PROJECT_SOURCE_DIR}/blasfeo_api/s_blas2_lib.c
		${PROJECT_SOURCE_DIR}/blasfeo_api/s_blas2_diag_lib.c
		${PROJECT_SOURCE_DIR}/blasfeo_api/s_blas3_lib.c
		${PROJECT_SOURCE_DIR}/blasfeo_api/s_blas3_diag_lib.c
		${PROJECT_SOURCE_DIR}/blasfeo_api/s_lapack_lib.c
		)

endif(${LA} MATCHES HIGH_PERFORMANCE)

if(${EXT_DEP})

	file(GLOB EXT_SRC
		${PROJECT_SOURCE_DIR}/auxiliary/d_aux_ext_dep_lib.c
		${PROJECT_SOURCE_DIR}/auxiliary/d_aux_ext_dep_lib4.c
		${PROJECT_SOURCE_DIR}/auxiliary/s_aux_ext_dep_lib.c
		${PROJECT_SOURCE_DIR}/auxiliary/s_aux_ext_dep_lib4.c
		${PROJECT_SOURCE_DIR}/auxiliary/v_aux_ext_dep_lib.c
		${PROJECT_SOURCE_DIR}/auxiliary/i_aux_ext_dep_lib.c
		${PROJECT_SOURCE_DIR}/auxiliary/timing.c
		)

endif()


if(BLASFEO_TESTING MATCHES ON)

	file(GLOB AUX_REF_SRC
		${PROJECT_SOURCE_DIR}/auxiliary/d_aux_libref.c
		${PROJECT_SOURCE_DIR}/auxiliary/s_aux_libref.c
		${PROJECT_SOURCE_DIR}/auxiliary/d_aux_ext_dep_libref.c
		${PROJECT_SOURCE_DIR}/auxiliary/s_aux_ext_dep_libref.c
		${PROJECT_SOURCE_DIR}/blasfeo_api/d_blas3_libref.c
		)

	set(BLASFEO_REF_SRC ${AUX_REF_SRC})
	add_library(blasfeo_ref ${BLASFEO_REF_SRC})
	set_target_properties(blasfeo_ref PROPERTIES LINKER_LANGUAGE C)
endif()

set(BLASFEO_SRC ${CMN_SRC} ${AUX_SRC} ${KERNEL_SRC} ${BLAS_SRC} ${EXT_SRC})

# add library
add_library(blasfeo ${BLASFEO_SRC})
if(EMBEDDED_TARGET MATCHES XILINX_NONE_ELF )
	target_link_libraries(blasfeo PUBLIC -Wl,--start-group xil c gcc -Wl,--end-group)
endif()


target_include_directories(blasfeo
	PUBLIC
		$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>)

if(USE_C99_MATH)
	target_compile_definitions(blasfeo PUBLIC USE_C99_MATH)
endif()

install(TARGETS blasfeo EXPORT blasfeoConfig
	LIBRARY DESTINATION lib
	ARCHIVE DESTINATION lib
	RUNTIME DESTINATION bin)


install(EXPORT blasfeoConfig DESTINATION "share/blasfeo/cmake")

file(GLOB_RECURSE BLASFEO_HEADERS "include/*.h")
install(FILES ${BLASFEO_HEADERS} DESTINATION "include/blasfeo/")

# tests
if(BLASFEO_TESTING MATCHES ON)
	add_subdirectory(tests)
endif()

# benchmarks
if(BLASFEO_BENCHMARKS MATCHES ON)
	add_subdirectory(benchmarks)
endif()

# examples
if(BLASFEO_EXAMPLES MATCHES ON)
	add_subdirectory(examples)
endif()
