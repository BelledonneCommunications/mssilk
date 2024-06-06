############################################################################
# CMakeLists.txt
# Copyright (C) 2014-2023  Belledonne Communications, Grenoble France
#
############################################################################
#
# This program is free software; you can redistribute it and/or
# modify it under the terms of the GNU General Public License
# as published by the Free Software Foundation; either version 2
# of the License, or (at your option) any later version.
#
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with this program; if not, write to the Free Software
# Foundation, Inc., 51 Franklin Street, Fifth Floor, Boston, MA  02110-1301, USA.
#
############################################################################

cmake_minimum_required(VERSION 3.0)

project(MSSilk VERSION 1.1.1 LANGUAGES C)

set(CMAKE_LIBRARY_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/lib/mediastreamer2/plugins")

include(GNUInstallDirs)

if(NOT CMAKE_INSTALL_RPATH AND CMAKE_INSTALL_PREFIX)
	set(CMAKE_INSTALL_RPATH ${CMAKE_INSTALL_FULL_LIBDIR})
	message(STATUS "Setting install rpath to ${CMAKE_INSTALL_RPATH}")
endif()

find_package(Mediastreamer2 5.3.0 REQUIRED)

set(FLAVOUR "FIX")
file(GLOB SILK_SOURCE_FILES "sdk/SILK_SDK_SRC_v1.0.9/SILK_SDK_SRC_${FLAVOUR}_v1.0.9/src/*.c")

# turn off warnings for the sdk compilation since we won't fix them
set_source_files_properties(${SILK_SOURCE_FILES} PROPERTIES COMPILE_FLAGS -w)

set(SOURCE_FILES
	${SILK_SOURCE_FILES}
	silk_dec.c
	silk_enc.c
)

if(BUILD_SHARED_LIBS)
	if(IOS)
		add_library(mssilk SHARED ${SOURCE_FILES})
	else()
		add_library(mssilk MODULE ${SOURCE_FILES})
	endif()
else()
	add_library(mssilk STATIC ${SOURCE_FILES})
endif()
target_compile_definitions(mssilk PRIVATE "VERSION=\"${PROJECT_VERSION}\"")
target_include_directories(mssilk PRIVATE "sdk/SILK_SDK_SRC_v1.0.9/SILK_SDK_SRC_${FLAVOUR}_v1.0.9/interface")
target_link_libraries(mssilk PRIVATE ${Mediastreamer2_TARGET})
if(BUILD_SHARED_LIBS)
	if(IOS)
		set_target_properties(mssilk PROPERTIES
			FRAMEWORK TRUE
			MACOSX_FRAMEWORK_IDENTIFIER org.linphone.mssilk
  			MACOSX_FRAMEWORK_INFO_PLIST "${PROJECT_SOURCE_DIR}/build/osx/Info.plist.in"
		) 
	endif()
	if(MSVC)
		install(FILES $<TARGET_PDB_FILE:mssilk>
			DESTINATION ${CMAKE_INSTALL_BINDIR}
			PERMISSIONS OWNER_READ OWNER_WRITE OWNER_EXECUTE GROUP_READ GROUP_EXECUTE WORLD_READ WORLD_EXECUTE
			CONFIGURATIONS Debug RelWithDebInfo
		)
		set_target_properties(mssilk PROPERTIES PREFIX "lib")
	endif()
endif()


install(TARGETS mssilk
	RUNTIME DESTINATION ${Mediastreamer2_PLUGINS_DIR}
	LIBRARY DESTINATION ${Mediastreamer2_PLUGINS_DIR}
	ARCHIVE DESTINATION ${Mediastreamer2_PLUGINS_DIR}
	FRAMEWORK DESTINATION Frameworks
	PERMISSIONS OWNER_READ OWNER_WRITE OWNER_EXECUTE GROUP_READ GROUP_EXECUTE WORLD_READ WORLD_EXECUTE
)


# CPack settings
string(TOLOWER "${PROJECT_NAME}" LOWERCASE_PROJECT_NAME)
set(CPACK_SOURCE_GENERATOR "TGZ")
set(CPACK_SOURCE_PACKAGE_FILE_NAME "${LOWERCASE_PROJECT_NAME}-${PROJECT_VERSION}")
set(CPACK_SOURCE_IGNORE_FILES
	"^${CMAKE_BINARY_DIR}"
	"/\\\\..+"
)

include(CPack)
