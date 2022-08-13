set(GEN_TEXI_FILES_LIST
	version.texi
	parts.texi
	programmers.texi
	programmer_types.texi
	)

set(DOC_TYPES_LIST
	pdf
	dvi
	ps
	html
	)

if(${CMAKE_BUILD_TYPE} EQUAL "Debug")
	set(SCRIPT_DEBUG "TRUE")
endif()

set(GEN_TEXI_SCRIPT "gen_texi.sh")
set(GEN_DOC_SCRIPT "gen_doc.sh")

set(DOC_NAME ${PROJECT_NAME})
# TODO: find and set actual executables
set(MAIN_EXE avrdude)
set(AWK_BIN awk)
set(MAKEINFO_BIN makeinfo)

set(MAIN_TEXI "${DOC_NAME}.texi")
set(STYLE_SHEET "${DOC_NAME}.css")
set(COMMENTS "parts_comments.txt")

configure_file(${GEN_TEXI_SCRIPT}.in tmp/${GEN_TEXI_SCRIPT} @ONLY)
file(COPY ${CMAKE_CURRENT_BINARY_DIR}/tmp/${GEN_TEXI_SCRIPT}
		DESTINATION ${CMAKE_CURRENT_BINARY_DIR}
		FILE_PERMISSIONS
		OWNER_EXECUTE OWNER_READ
	)

configure_file(${GEN_DOC_SCRIPT}.in tmp/${GEN_DOC_SCRIPT} @ONLY)
file(COPY ${CMAKE_CURRENT_BINARY_DIR}/tmp/${GEN_DOC_SCRIPT}
		DESTINATION ${CMAKE_CURRENT_BINARY_DIR}
		FILE_PERMISSIONS
		OWNER_EXECUTE OWNER_READ
	)

file(COPY ${MAIN_TEXI} DESTINATION ${CMAKE_CURRENT_BINARY_DIR})
file(COPY ${STYLE_SHEET} DESTINATION ${CMAKE_CURRENT_BINARY_DIR})
file(COPY ${COMMENTS} DESTINATION ${CMAKE_CURRENT_BINARY_DIR})

# temporary solution till we figured out how to create texi files in a proper way
file(COPY generator DESTINATION ${CMAKE_CURRENT_BINARY_DIR})

foreach(TEXI ${GEN_TEXI_FILES_LIST})

	add_custom_target("${TEXI}_target" DEPENDS ${TEXI})

	add_custom_command(OUTPUT ${TEXI}
		COMMAND ${CMAKE_CURRENT_BINARY_DIR}/${GEN_TEXI_SCRIPT} ${TEXI}
		# TODO: add dependency on main
		)

endforeach()

list(TRANSFORM GEN_TEXI_FILES_LIST APPEND "_target")
foreach(DOC_TYPE ${DOC_TYPES_LIST})

	set(DOC_FILE "${DOC_NAME}.${DOC_TYPE}")

	add_custom_target("doc_${DOC_TYPE}_target" DEPENDS ${DOC_FILE})

	add_custom_command(OUTPUT ${DOC_FILE}
		COMMAND ${CMAKE_CURRENT_BINARY_DIR}/${GEN_DOC_SCRIPT} ${DOC_TYPE}
		MAIN_DEPENDENCY ${MAIN_TEXI}
		DEPENDS ${GEN_TEXI_FILES_LIST}
		)

endforeach()

list(TRANSFORM DOC_TYPES_LIST PREPEND "doc_")
list(TRANSFORM DOC_TYPES_LIST APPEND "_target")
add_custom_target("doc_target"
	ALL
	DEPENDS ${DOC_TYPES_LIST}
	)
