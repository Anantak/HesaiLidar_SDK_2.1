cmake_minimum_required(VERSION 3.8.11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_STANDARD 17)
PROJECT(HESAI_21_DRIVER)

set(WITH_PTCS_USE ON CACHE BOOL "Use Ptcs")
set(FIND_CUDA OFF CACHE BOOL "Find CUDA")  # Manuj : setting CUDA OFF by default

if(FIND_CUDA)
    find_package(CUDA)

if(${CUDA_FOUND})

	project(CommonModules LANGUAGES CXX CUDA)

	#Get CUDA compute capability
	set(OUTPUTFILE ${PROJECT_SOURCE_DIR}/_build/cuda_script) # No suffix required
	set(CUDAFILE   ${PROJECT_SOURCE_DIR}/test/check_cuda.cu)
	execute_process(COMMAND nvcc -lcuda ${CUDAFILE} -o ${OUTPUTFILE})
	execute_process(COMMAND ${OUTPUTFILE}
									RESULT_VARIABLE CUDA_RETURN_CODE
									OUTPUT_VARIABLE CUDA_ARCH_DETECTED)

	if(${CUDA_RETURN_CODE} EQUAL 0)
		set(CUDA_SUCCESS "TRUE")
	else()
		set(CUDA_SUCCESS "FALSE")
	endif()

	string(REPLACE "-arch=sm_" "" CUDA_ARCH_NUMBER "${CUDA_ARCH_DETECTED}")

	if (${CUDA_SUCCESS})
		message(STATUS "----------------------------")
		message(STATUS "------ CUDA detection ------")
		message(STATUS "----------------------------")
		message(STATUS "CUDA Architecture: ${CUDA_ARCH_DETECTED}")
		message(STATUS "CUDA Arch number: ${CUDA_ARCH_NUMBER}")
		message(STATUS "CUDA Version: ${CUDA_VERSION_STRING}")
		message(STATUS "CUDA Path: ${CUDA_TOOLKIT_ROOT_DIR}")
		message(STATUS "CUDA Libararies: ${CUDA_LIBRARIES}")
		message(STATUS "CUDA Include Dir: ${CUDA_INCLUDE_DIRS}")
		message(STATUS "----------- end ------------")
		message(STATUS "----------------------------")
	else()
		message(WARNING ${CUDA_ARCH_DETECTED})
	endif()

  if(NOT DEFINED CMAKE_CUDA_ARCHITECTURES)
    set(CMAKE_CUDA_ARCHITECTURES ${CUDA_ARCH_NUMBER})
  endif()


	set(CUDA_SOURCE_PROPERTY_FORMAT OBJ)
	set(CUDA_SEPARABLE_COMPILATION ON)
	include_directories(${CUDA_INCLUDE_DIRS})
	set(CUDA_PROPAGATE_HOST_FLAGS OFF)
	set(CUDA_NVCC_FLAGS -arch=sm_${CUDA_ARCH_NUMBER};-O3;-G;-g;-std=c++17)
	link_directories($ENV{CUDA_PATH}/lib/x64)
else(${CUDA_FOUND})
	MESSAGE(STATUS "cuda not found!")
endif(${CUDA_FOUND})
endif()


cmake_policy(SET CMP0053 NEW)
set(CMAKE_BUILD_TYPE ON)
set(CMAKE_BUILD_TYPE Release)
add_definitions(-O3)
add_compile_options(-Wall)

add_subdirectory(libhesai)

add_library(hesai_sdk_lib
	driver/hesai_lidar_sdk.cc
)
target_link_libraries(hesai_sdk_lib PUBLIC 
	lidar_lib
	${OTHER_LIB}
)
set_target_properties(hesai_sdk_lib 
	PROPERTIES CXX_STANDARD_REQUIRED TRUE CXX_STANDARD 17
)
target_include_directories(hesai_sdk_lib PUBLIC   
	${OTHER_INCLUDE_DIRS}
	driver
	./
) 

if(FIND_CUDA)
if(${CUDA_FOUND})
	target_compile_definitions(hesai_sdk_lib PUBLIC USE_CUDA=1)
endif(${CUDA_FOUND})
endif()


if(DISENABLE_TEST_CC)

else()

	add_executable(test_hesai_lidar_app
		test/test.cc
	)
	target_link_libraries(test_hesai_lidar_app
		hesai_sdk_lib
	)

	# add_executable(multi_test
	# 	test/multi_test.cc
	# )
	# target_link_libraries(multi_test
	# 	hesai_sdk_lib
	# )

endif()
