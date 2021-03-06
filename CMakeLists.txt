cmake_minimum_required(VERSION 3.13)
if (DEFINED MCU)
	# Setting up the toolchain
	set(CMAKE_TOOLCHAIN_FILE "${CMAKE_SOURCE_DIR}/cmake/cortex.cmake")
endif()
# Project information
project(Ublox
	VERSION 3.2.0
	DESCRIPTION "UBLOX sensor driver"
	LANGUAGES CXX
)
if (DEFINED MCU)
	# Grab the processor and set up definitions and compile options
	include(${CMAKE_SOURCE_DIR}/cmake/config_mcu.cmake)
	configMcu(${MCU})
	include(FetchContent)
	# Fetch core
	FetchContent_Declare(
		core
		GIT_REPOSITORY 	https://github.com/bolderflight/core.git
		GIT_TAG v2.0.5
	)
	FetchContent_MakeAvailable(core)
	# Fetch units
	FetchContent_Declare(
		units
		GIT_REPOSITORY 	https://github.com/bolderflight/units.git
		GIT_TAG v3.2.0
	)
	FetchContent_MakeAvailable(units)
	# Fetch gnss
	FetchContent_Declare(
		gnss
		GIT_REPOSITORY 	https://github.com/bolderflight/gnss.git
		GIT_TAG v2.4.0
	)
	FetchContent_MakeAvailable(gnss)
	# Add the library target
	add_library(ublox
		src/ublox/ublox.cc
		include/ublox/ublox.h
	)
	# Link libraries
	target_link_libraries(ublox
		PUBLIC
			core
			units
			gnss
	)
	# Setup include directories 
	target_include_directories(ublox PUBLIC 
		$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
		$<INSTALL_INTERFACE:include>
	)
endif()

# Example and test if this project is built separately
if (PROJECT_NAME STREQUAL CMAKE_PROJECT_NAME)
	if (DEFINED MCU)
		# Add the spi example target
		add_executable(uart_example examples/uart_example.cc)
		# Add the includes
		target_include_directories(uart_example PUBLIC 
			$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
			$<INSTALL_INTERFACE:include>
		)
		# Link libraries to the example target
		target_link_libraries(uart_example
			PRIVATE 
				ublox
				units
		)
		# Add hex and upload targets
		include(${CMAKE_SOURCE_DIR}/cmake/flash_mcu.cmake)
		FlashMcu(uart_example ${MCU})
	endif()
endif()
