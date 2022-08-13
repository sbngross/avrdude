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

set(GEN_TEXI_SCRIPT "gen_texi.sh")
set(GEN_DOC_SCRIPT "gen_doc.sh")

set(DOC_NAME ${PROJECT_NAME})

set(MAIN_TEXI "${DOC_NAME}.texi")

configure_file(${GEN_TEXI_SCRIPT}.in ${GEN_TEXI_SCRIPT}
		FILE_PERMISSIONS
		OWNER_EXECUTE OWNER_READ
		@ONLY
	)

configure_file(${GEN_DOC_SCRIPT}.in ${GEN_DOC_SCRIPT}
		FILE_PERMISSIONS
		OWNER_EXECUTE OWNER_READ
		@ONLY
	)

foreach(TEXI ${GEN_TEXI_FILES_LIST})

	add_custom_command(OUTPUT ${TEXI}
		COMMAND ${CMAKE_CURRENT_BINARY_DIR}/${GEN_TEXI_SCRIPT} ${TEXI}
		)

endforeach()

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
