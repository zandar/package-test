project(test-package)
cmake_minimum_required(VERSION 2.8)

#config

set(PROJECT_VERSION_MAJOR 0)
set(PROJECT_VERSION_MINOR 1)
set(PROJECT_VERSION_PATCH 0)

#setup
set(CMAKE_MODULE_PATH "${CMAKE_SOURCE_DIR}/cmake")

set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)

if(${CMAKE_SOURCE_DIR} STREQUAL ${CMAKE_BINARY_DIR})
    message( FATAL_ERROR "In-source builds not allowed. Please make a new directory (called a build directory) and run CMake from there. You may need to remove CMakeCache.txt." )
endif()

#Packaging
if(CMAKE_SYSTEM_PROCESSOR STREQUAL "x86_64")
    set(CPACK_DEBIAN_PACKAGE_ARCHITECTURE "amd64")
elseif(CMAKE_SYSTEM_PROCESSOR MATCHES "^i.86$")
    set(CPACK_DEBIAN_PACKAGE_ARCHITECTURE "i386")
elseif(${CMAKE_SYSTEM_PROCESSOR} STREQUAL "arm" OR ${CMAKE_SYSTEM_PROCESSOR} STREQUAL "armv7l")
    set(CPACK_DEBIAN_PACKAGE_ARCHITECTURE "armhf")
else()
    set(CPACK_DEBIAN_PACKAGE_ARCHITECTURE "${CMAKE_SYSTEM_PROCESSOR}")
endif()
	
string(TIMESTAMP CPACK_DEBIAN_TIMESTAMP "%y%m%d%H%M")
	
set(CPACK_GENERATOR "DEB")
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "PPA testing package.")
set(CPACK_PACKAGE_VENDOR "Day One")
set(CPACK_PACKAGE_FILE_NAME "${PROJECT_NAME}_${PROJECT_VERSION_MAJOR}.${PROJECT_VERSION_MINOR}.${PROJECT_VERSION_PATCH}_${CPACK_DEBIAN_PACKAGE_ARCHITECTURE}+${CPACK_DEBIAN_TIMESTAMP}")
set(CPACK_PACKAGE_VERSION_MAJOR ${PROJECT_VERSION_MAJOR})
set(CPACK_PACKAGE_VERSION_MINOR ${PROJECT_VERSION_MINOR})
set(CPACK_PACKAGE_VERSION_PATCH "${PROJECT_VERSION_PATCH}+${CPACK_DEBIAN_TIMESTAMP}")
set(CPACK_DEBIAN_PACKAGE_SHLIBDEPS ON)
set(CPACK_DEBIAN_PACKAGE_SECTION "angee")
set(CPACK_DEBIAN_PACKAGE_MAINTAINER "Michal Vokáč")
set(CPACK_PACKAGE_CONTACT "vokac.m@gmail.com")
set(CPACK_DEBIAN_PACKAGE_DEPENDS "")

add_custom_target(deploy COMMAND scp ${CPACK_PACKAGE_FILE_NAME}.deb aptly-ppa@vps1.meetangee.com:/home/aptly-ppa/.aptly/incoming COMMAND ssh aptly-ppa@vps1.meetangee.com /home/aptly-ppa/bin/process-incoming.sh COMMENT "Uploading ${CPACK_PACKAGE_FILE_NAME} package to PPA")

#this is to hackt this error https://jderobot.org/Varribas-tfm/ARDrone:cmake_advanced
#file(WRITE dummy_main.c "int main(int argc, char** argv){return 0;}")
#add_executable(dummy_main dummy_main.c)
aux_source_directory(./src SRC_LIST)
add_executable(${PROJECT_NAME} ${SRC_LIST})

INSTALL(TARGETS ${PROJECT_NAME} DESTINATION /opt/angee/${PROJECT_NAME})

install(DIRECTORY "${PROJECT_SOURCE_DIR}/src/" DESTINATION /opt/test-package USE_SOURCE_PERMISSIONS)

INCLUDE(CPack)
INCLUDE(InstallRequiredSystemLibraries)
 
