#
# Preamble
#

cmake_minimum_required(VERSION 3.15)
project(SocketIO VERSION 1.6.0 LANGUAGES CXX)

#
# Create Target
#

# Add the library
add_library (SocketIO STATIC)
add_library(SocketIO::SocketIO ALIAS SocketIO)

# Add sources to target
target_sources(SocketIO
	PRIVATE
		include/SocketIO/sio_client.h
		include/SocketIO/sio_message.h
		include/SocketIO/sio_socket.h
        include/SocketIO/Internal/sio_client_impl.h
        include/SocketIO/Internal/sio_packet.h
        src/internal/sio_client_impl.cpp
        src/internal/sio_packet.cpp
        src/sio_client.cpp
        src/sio_socket.cpp
)

# Postfix debug files
set_target_properties(SocketIO PROPERTIES DEBUG_POSTFIX "d")

#
# Usage Requirements
#

# Add compiler features and definitions
target_compile_features(SocketIO PRIVATE cxx_std_11)
target_compile_definitions(SocketIO PRIVATE
    # These will force ASIO to compile without Boost
    BOOST_DATE_TIME_NO_LIB
    BOOST_REGEX_NO_LIB
    ASIO_STANDALONE
    # These will force WebsocketPP to compile with C++11
    _WEBSOCKETPP_CPP11_STL_
    _WEBSOCKETPP_CPP11_FUNCTIONAL_
)

# Specify include directories to use when compiling a given target
target_include_directories(SocketIO
	PUBLIC
		$<INSTALL_INTERFACE:include>
		$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
	PRIVATE
		${CMAKE_CURRENT_SOURCE_DIR}/src
        ${CMAKE_CURRENT_SOURCE_DIR}/lib/websocketpp 
        ${CMAKE_CURRENT_SOURCE_DIR}/lib/rapidjson/include
        ${CMAKE_CURRENT_SOURCE_DIR}/lib/asio/asio/include
)

# Link dependencies (None)

#
# Install and Export
#

# Set the install destination for cmake files
set(INSTALL_CONFIGDIR ${CMAKE_INSTALL_LIBDIR}/cmake/SocketIO)

# Install include directory
include(GNUInstallDirs)
install(DIRECTORY include/ DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})

# Specify the name of the export set the targets belong to
install(TARGETS SocketIO
	EXPORT SocketIOTargets
	ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}/$<CONFIG>
	LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}/$<CONFIG>
	RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}/$<CONFIG>
	INCLUDES DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
)

# Export the targets to a script
install(EXPORT SocketIOTargets
	FILE SocketIOTargets.cmake
	NAMESPACE SocketIO::
	DESTINATION ${INSTALL_CONFIGDIR}
)

# Generate a config file that includes the exports
include(CMakePackageConfigHelpers)
configure_package_config_file(
	${CMAKE_CURRENT_LIST_DIR}/cmake/SocketIOConfig.cmake.in
	${CMAKE_CURRENT_BINARY_DIR}/cmake/SocketIOConfig.cmake
	INSTALL_DESTINATION ${INSTALL_CONFIGDIR}
	NO_SET_AND_CHECK_MACRO
	NO_CHECK_REQUIRED_COMPONENTS_MACRO
)

# Generate the version file for the config file
write_basic_package_version_file(
	${CMAKE_CURRENT_BINARY_DIR}/cmake/SocketIOConfigVersion.cmake
	VERSION ${PROJECT_VERSION}
	COMPATIBILITY SameMajorVersion
)

# Install the Config, ConfigVersion and custom Find Module
install(FILES
	#${CMAKE_CURRENT_LIST_DIR}/FindSocketIO.cmake
	${CMAKE_CURRENT_BINARY_DIR}/cmake/SocketIOConfig.cmake
	${CMAKE_CURRENT_BINARY_DIR}/cmake/SocketIOConfigVersion.cmake
	DESTINATION ${INSTALL_CONFIGDIR}
)

# Make this project available from the build directory
export(EXPORT SocketIOTargets
	FILE ${CMAKE_CURRENT_BINARY_DIR}/SocketIOTargets.cmake
	NAMESPACE SocketIO::
)

#
# Testing
#

#if(BUILD_TESTS)
#	enable_testing()
#	add_subdirectory(test)
#endif()