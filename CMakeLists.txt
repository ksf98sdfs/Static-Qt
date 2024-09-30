cmake_minimum_required( VERSION 3.30.3 )

set( library_name Static-Qt-Test )

project( ${library_name} CXX )

set( CMAKE_INCLUDE_CURRENT_DIR ON )
set( CMAKE_CXX_STANDARD 17 )
set( CMAKE_CXX_STANDARD_REQUIRED ON )
set( CMAKE_CXX_EXTENSIONS OFF )

# Enable static linkage of runtime on MSVC
if( MSVC )
  set( CMAKE_MSVC_RUNTIME_LIBRARY "MultiThreaded" )
endif()

# Enable static linking of OpenSSL
set( OPENSSL_USE_STATIC_LIBS TRUE )
if( MSVC )
	set( OPENSSL_MSVC_STATIC_RT TRUE )
endif()

# Hide the console window in visual studio projects
if( MSVC )
	set( CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} /SUBSYSTEM:WINDOWS /ENTRY:mainCRTStartup" )
	set( CMAKE_EXE_LINKER_FLAGS_RELEASE "${CMAKE_EXE_LINKER_FLAGS} /SUBSYSTEM:WINDOWS /ENTRY:mainCRTStartup" )
endif()

# Locate OpenSSL's libraries and header files.
find_package( OpenSSL REQUIRED )

message(STATUS "OpenSSL found: ${OPENSSL_FOUND}")
message(STATUS "Using OpenSSL version: ${OPENSSL_VERSION}")
message(STATUS "The OpenSSL include directory: ${OPENSSL_INCLUDE_DIR}")
message(STATUS "The OpenSSL crypto library: ${OPENSSL_CRYPTO_LIBRARY}")
message(STATUS "The OpenSSL SSL library: ${OPENSSL_SSL_LIBRARIES}")

file( GLOB SRC_FILES ${PROJECT_SOURCE_DIR}/src/*.cpp ${PROJECT_SOURCE_DIR}/src/*.h ${PROJECT_SOURCE_DIR}/src/*.hpp ${PROJECT_SOURCE_DIR}/src/*.qrc )

if( CMAKE_SYSTEM_NAME STREQUAL "Windows" )
	add_executable( ${library_name} WIN32 ${SRC_FILES} )
elseif( CMAKE_SYSTEM_NAME STREQUAL "Linux" )
	add_executable( ${library_name} ${SRC_FILES} )
elseif( CMAKE_SYSTEM_NAME STREQUAL "Darwin" )
	add_executable( ${library_name} ${SRC_FILES} )
endif()

# Set the correct Visual Studio startup project.
if ( MSVC )
	set_property( DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR} PROPERTY VS_STARTUP_PROJECT ${library_name} )
endif()

# Link against the Qt libraries, and add required include directories and compile definitions.
if( CMAKE_SYSTEM_NAME STREQUAL "Windows" )
	target_link_libraries( ${library_name} PRIVATE Qt5::Core Qt5::Gui Qt5::Network Qt5::Widgets OpenSSL::Crypto OpenSSL::SSL crypt32.lib ws2_32.lib )
elseif( CMAKE_SYSTEM_NAME STREQUAL "Linux" )
	target_link_libraries( ${library_name} PRIVATE OpenSSL::Crypto OpenSSL::SSL )
elseif( CMAKE_SYSTEM_NAME STREQUAL "Darwin" )
	target_link_libraries( ${library_name} PRIVATE Qt5::Core Qt5::Gui Qt5::Network Qt5::Widgets OpenSSL::Crypto OpenSSL::SSL )
endif()

# Include ${CMAKE_CURRENT_BINARY_DIR} for the generated header.
target_include_directories( ${library_name} PRIVATE ${CMAKE_CURRENT_BINARY_DIR} )
