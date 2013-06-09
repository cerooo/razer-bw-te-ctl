cmake_minimum_required (VERSION 2.8.5)
project (razer-bw-te-ctl C)
set (project_VERSION "1.0")

find_package (PkgConfig REQUIRED)
pkg_check_modules (dependencies REQUIRED libusb-1.0)
include_directories (${dependencies_INCLUDE_DIRS})

include (GNUInstallDirs)
configure_file (${PROJECT_SOURCE_DIR}/config.h.in
	${PROJECT_BINARY_DIR}/config.h)
include_directories (${PROJECT_BINARY_DIR})

add_executable (${PROJECT_NAME} ${PROJECT_NAME}.c)
target_link_libraries (${PROJECT_NAME} ${dependencies_LIBRARIES})
install (TARGETS ${PROJECT_NAME} DESTINATION ${CMAKE_INSTALL_BINDIR})

find_program (HELP2MAN_EXECUTABLE help2man)
if (NOT HELP2MAN_EXECUTABLE)
	message (FATAL_ERROR "help2man not found")
endif ()

foreach (page ${PROJECT_NAME})
	set (page_output "${PROJECT_BINARY_DIR}/${page}.1")
	list (APPEND project_MAN_PAGES "${page_output}")
	add_custom_command (OUTPUT ${page_output}
		COMMAND ${HELP2MAN_EXECUTABLE} -N
			"${PROJECT_BINARY_DIR}/${page}" -o ${page_output}
		DEPENDS ${PROJECT_NAME}
		COMMENT "Generating man page for ${page}" VERBATIM)
endforeach ()

add_custom_target (docs ALL DEPENDS ${project_MAN_PAGES})

foreach (page ${project_MAN_PAGES})
	string (REGEX MATCH "\\.([0-9])" manpage_suffix "${page}")
	install (FILES "${page}"
		DESTINATION "${CMAKE_INSTALL_MANDIR}/man${CMAKE_MATCH_1}")
endforeach ()

set (CPACK_PACKAGE_DESCRIPTION_SUMMARY
	"Razer BlackWidow Tournament Edition control utility")
set (CPACK_PACKAGE_VERSION ${project_VERSION})
set (CPACK_PACKAGE_VENDOR "Premysl Janouch")
set (CPACK_PACKAGE_CONTACT "Přemysl Janouch <p.janouch@gmail.com>")
set (CPACK_RESOURCE_FILE_LICENSE "${PROJECT_SOURCE_DIR}/LICENSE")

set (CPACK_GENERATOR "TGZ;ZIP")
set (CPACK_PACKAGE_FILE_NAME
	"${PROJECT_NAME}-${project_VERSION}-${CMAKE_SYSTEM_NAME}-${CMAKE_SYSTEM_PROCESSOR}")

set (CPACK_SOURCE_GENERATOR "TGZ;ZIP")
set (CPACK_SOURCE_IGNORE_FILES "/\\\\.git;/build;/CMakeLists.txt.user")
set (CPACK_SOURCE_PACKAGE_FILE_NAME "${PROJECT_NAME}-${project_VERSION}")

set (CPACK_SET_DESTDIR TRUE)
include (CPack)
